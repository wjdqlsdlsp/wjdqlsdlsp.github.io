---
layout: post
title:  "kubernetes 구성하기"
categories: [kubernetes]
tags: [kubernetes]
---

보통 Public Cloud에서 제공하는 EKS, GKE 등의 쿠버네티스 서비스를 사용하기 때문에 쿠버네티스 구성에 대해 생각을 해본적 없다가, 온디맨드에 kubernetes를 구성하는 상황
및 CKA 시험을 목적으로 공부하기 위해서 구축하는 방법을 정리해 봤습니다.

[따라배우는 쿠버네티스 - AWS EC2에 시험 환경 구축하기](https://mud-riddle-377.notion.site/Install-Kubernetes-with-kubeadm-c990d74012a34cefbff6d3f1c7455853)를 참고하여 정리하였습니다.

<p align="center"><img src="/assets/img/post_img/make_kubernetes_1.png"></p>

먼저,온디맨드 대신, AWS인스턴스를 사용하여 진행하였습니다. 사용한 환경은 위와 같습니다.

여기서 조심해야 할 점은, 가상 서버 유형을 t3.medium으로 구성했는데, 이는 Control-plain에서 클러스터 구성 시, `Memory가 1700MB이하일 경우 에러 발생`하기 때문에 여유있게 4GB로 구성하였습니다.

---



### 각 인스턴스 기본 셋팅

가장 먼저 ssh를 이용하여 각 인스턴스에 접속한다.

여기서 pem key 방식을 사용했다면, 아래와 같이 모드를 변경해 준다.

```shell
chmod 600 <pem_key_name>
```



이후 ssh를 통해 접속하면 되는데, 접속 방식은 아래와 같다.

```shell
ssh -i <pem_key_name> ubuntu@<public_ip>
```



접속에 성공했다면, 모든 인스턴스에 대해서 로컬 시간대 및 hostname을 변경해준다.

```shell
sudo -i

rm -f /etc/localtime
ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime

# 바뀐지 확인
date
```

KST로 표기되면 정상적으로 시간대가 변경된 것이다.



```shell
hostnamectl hostname k8s-control-plane
```

이후 노드들을 구분하기 위해 hostname을 지정해준다. 여기서는 `k8s-master`, `k8s-worker1`, `k8s-worker2`로 구성했다.



```shell
vi /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4
172.31.159.93 k8s-control-plane
172.31.31.224 k8s-worker1
172.31.40.253 k8s-worker2
```

이후 exit한 뒤, 재접속하면 hostname이 변경되어 있음을 확인할 수 있다.



```shell
ping -c 2 k8s-control-plane
```

hosts설정이 잘 되었나 확인하기 위해서는, 워커노드에서 ping을 보내서 잘 전송이 되는지 확인한다.



다음으로는 IPv4 포워딩을 위한 iptables bridge 구성한다.

```shell
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sysctl --system
```

---



### <참고> 그 외 확인해야 할 내용들

```shell
cat /etc/os-release

lscpu

free -h

hostname

ip addr

# 방화벽 활성화 여부 - 루트권한 필요 (inactive 필요)
ufw status

# 활성화된 스왑 영역 정보를 출력 (없는 상태)
swapon -s
```

쿠버네티스를 구성할 때, 리소스, ip 및 ufw, 스왑 영역등을 확인하는 명령어 들이다.

---

### 쿠버네티스 엔진 설치하기

쿠버네티스 엔진을 설치할 때 크게 두가지로 나눠지는데, 아래와 같다.

- containerd 엔진 설치
- cri-docker 엔진 설치

Dockershim에 대한 지원을 종료하면서, containerd 엔진을 사용하는 것이 표준으로 자리잡혀있다.

[Amazon EKS는 `Dockershim`에 대한 지원을 종료합니다.](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/dockershim-deprecation.html)

<br>

#### containers 엔진

먼저 containerd 엔진 설치 방법이다.

```shell
apt update
apt install  apt-transport-https ca-certificates curl gnupg lsb-release -y

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

apt update

apt install containerd.io -y

mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```



```shell
vi /etc/containerd/config.toml
```
<p align="center"><img src="/assets/img/post_img/make_kubernetes2.png"></p>

`plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options` 부분의 `SystemCgroup` 을 true로 변경해준다,

```shell
systemctl restart containerd
```



잘 설치가 되었나 확인하려면 아래의 명령들을 통해 테스트한다.

```shell
systemctl status containerd

ls -l  /run/containerd/containerd.sock
```



---

#### cri-docker 엔진

다음은 cri-docker를 설치하는 방법이다.

```shell
apt update
apt install -y docker.io
systemctl enable --now docker
systemctl status docker
```



```shell
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.0/cri-dockerd_0.3.0.3-0.ubuntu-bionic_amd64.deb
dpkg -i cri-dockerd_0.3.0.3-0.ubuntu-bionic_amd64.deb
systemctl status cri-docker
ls /var/run/cri-dock
```

---

### kubeadm 설치하기

kubeadm은 툴박스로, 쿠버네티스 클러스터를 구성해주는 친구다.

[kubernetes docs - kubeadm 설치하기](https://kubernetes.io/ko/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)를 한번 읽어보기를 추천한다.



설치 과정은 아래와 같다.

```shell
apt update
apt install -y apt-transport-https ca-certificates curl

curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg \
  https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] \
  https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

apt update
apt install -y kubelet=1.26.0-00 kubeadm=1.26.0-00 kubectl=1.26.0-00
apt-mark hold kubelet kubeadm kubectl
```

---

### kubernetes 클러스터 생성하기 ( control-plane )

이제 kubeadm을 통한 클러스터를 생성하는 단계인데, `control-plane` 에서만 진행한다.

위의 엔진 설치단계에서 containerd를 설치했는 지, cri-docker를 설치했는지에 따라 sock 경로 및 파일명이 다르니 참고해주자.

```shell
# containerd를 설치했으면
kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket unix:///var/run/containerd/containerd.sock

# cri-docker를 설치했으면
kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket unix:///var/run/cri-dockerd.sock
```

위의 명령어를 실행하면, kubeadm join <public-ip 주소> --token <token 값> ~~~

부분이 있는데, 이를 복사해둔다.

```shell
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```



```shell
vi token.join
```

저장하는 값은, 위에서 복사한 kubeadm join에서  `--cri-socket` 부분을 추가한다. docker로 진행했다면, cri-docker.sock을 입력한다.

```shell
kubeadm join <public-ip 주소> --token <token 값> --discovery-token-ca-cert-hash <hash 값> --cri-socket unix:///var/run/containerd/containerd.sock
```

이렇게 클러스터 구성이 완료되면 우리에게 친근한 `kubectl`을 사용할 수 있게된다.

---

### Calico설치하기

```shell
kubectl get nodes
```

위의 명령어를 실행했을 때, NotReady상태임을 알 수 있는데, 이는 Calico가 없어서 pod들 끼리 통신이 불가능한 상태라 그렇다.



```shell
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/custom-resources.yaml
```

위의 명령어를 통해 calico를 설치하고 잠시 설치가 완료될 때 까지 대기한다.



```shell
kubectl get pods -n calico-system
```

이후, `kubectl get nodes` 명령어를 다시 실행하면, Ready상태일 것이다.



Calico 명령어를 사용한다면 아래의 과정을 따른다.

```shell
curl -L https://github.com/projectcalico/calico/releases/download/v3.24.1/calicoctl-linux-amd64 -o calicoctl
chmod +x calicoctl
mv calicoctl /usr/bin

cat << END > ipipmode.yaml
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: default-ipv4-ippool
spec:
  blockSize: 26
  cidr: 192.168.0.0/16
  ipipMode: Always
  natOutgoing: true
  nodeSelector: all()
  vxlanMode: Never
END

calicoctl apply -f ipipmode.yaml
calicoctl get ippool -o wide
```



이제 워커노드들을 연결해주는 과정이다.

각각의 워커노드에서 이전에, `token.join`에 저장 되어있는 명령어를 입력해준다.

```shell
kubeadm join <public-ip 주소> --token <token 값> --discovery-token-ca-cert-hash <hash 값> --cri-socket unix:///var/run/containerd/containerd.sock
```



```shell
kubectl get nodes -o wide
```

이후 노드들을 조회하면 연결되어 있음을 확인할 수 있다. 이렇게 노드연결까지 완료되었다.



### 그 외 설정하면 좋은 내용

kubectl 명령어를 입력할 때, 자동완성을 하는 설정이다.

```shell
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```



현재 sudo권한에서만 kubectl이 가능한데, 이를 ubuntu 권한에서도 가능하게 하는 설정이다.

```shell
mkdir -p ~ubuntu/.kube
cp -i /etc/kubernetes/admin.conf ~ubuntu/.kube/config
chown -R ubuntu:ubuntu ~ubuntu/.kube

exit

source <(kubectl completion bash); echo "source <(kubectl completion bash)" >> ~/.bashrc
```

<br>
