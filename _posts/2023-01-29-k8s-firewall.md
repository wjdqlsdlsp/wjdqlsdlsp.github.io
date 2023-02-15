---
layout: post
title:  "kubernetes API 통신 관련 내용 정리"
categories: [kubernetes]
tags: [kubernetes]
---



최근 워크스테이션에 쿠버네티스를 셋팅하고 있는데, 이 과정에서 쿠버네티스가 어떤 방식으로 통신하고 있는지 자세히 들여다 보았다.

이전에는 퍼블릭 클라우드에서 매니징 해주는 상태에서 쿠버네티스를 사용했기에 쿠버네티스가 어떤 포트를 이용하고 어떤방식으로 통신하고 있는지 생각해 본 적이 없다.



## 쿠버네티스가 사용하는 포트들

먼저 쿠버네티스가 어떤 포트를 통해서 통신하고 있는지는 [쿠버네티스 Docs](https://kubernetes.io/ko/docs/reference/ports-and-protocols/)에 잘 나와있다.



### 컨트롤 플레인

| 프로토콜 | 방향     | 포트 범위 | 용도                     | 사용 주체            |
| -------- | -------- | --------- | ------------------------ | -------------------- |
| TCP      | 인바운드 | 6443      | 쿠버네티스 API 서버      | 전부                 |
| TCP      | 인바운드 | 2379-2380 | etcd 서버 클라이언트 API | kube-apiserver, etcd |
| TCP      | 인바운드 | 10250     | Kubelet API              | Self, 컨트롤 플레인  |
| TCP      | 인바운드 | 10259     | kube-scheduler           | Self                 |
| TCP      | 인바운드 | 10257     | kube-controller-manager  | Self                 |



### 워커 노드

| 프로토콜 | 방향     | 포트 범위   | 용도            | 사용 주체           |
| -------- | -------- | ----------- | --------------- | ------------------- |
| TCP      | 인바운드 | 10250       | Kubelet API     | Self, 컨트롤 플레인 |
| TCP      | 인바운드 | 30000-32767 | NodePort 서비스 | 전부                |



Docs에 나와있는 표 부분을 가져왔다. 당연히 컨트롤 플레인과 워커노드가 사용하는 포트가 살짝 다르다.

<br>

여기서 참고를 위해서 Docs에 나와있는 [쿠버네티스 컴포넌트](https://kubernetes.io/ko/docs/concepts/overview/components/)를 가져왔다.

<p align="center"><img src="https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg"></p>

제일 먼저, 모든 컴포넌트들이 Control Plane의 API와 연결이 되어 있는데, 이 과정에서 쿠버네티스 API는 6443의 포트를 이용해서 데이터를 받아들이는 듯 하다.

Control Plane에서 나머지 etcd, kubelet, kube-scheduler, kube-controller-manager는 각각의 포트들을 사용한다.

> (참고) Control Plane 내부에도 Kubelet이 존재합니다.

그리고 워커노드와 컨트롤 플레인이 통신할 때는, Kubelet을 통해서 통신을 하는데 이 과정에서 10250 포트를 사용한다.

30000 ~ 32767의 포트는 노드포트 서비스라고 나와 있다.



---



## GCP에서는 어떻게 사용하고 있을까?

위를 통해서 특정 포트만 열어주면 쿠버네티스를 사용할 수 있다는 것을 알아냈다.

그래서 궁금해진 것이 "GKE에서는 어떻게 운영하고 있을까?" 이였으며, 몇가지 테스트해보았다.

<p align="center"><img src="/assets/img/post_img/default-gcp.png"></p>

위는 GKE를 생성했을 때, GCP에서 자동으로 생성해주는 VPC의 방화벽이다.

추가적인 설명을 잠깐 하면, `10.116.0.0/14` 의 CIDR는 클러스터 포드 IPv4 범위(기본값) 라고 표기되어있는 CIDR인데 이 영역에서 쿠버네티스 리소스들의 endpoint ip가 할당된다.

`10.128.0.0/9` 는 GKE에서 컨트롤플레인 및 노드가 할당된다.



GKE에서는 컨트롤 플레인을 GCP에서 운영하고, 워커노드를 노드풀 형식으로 관리하고 있는데, 컨트롤 플레인을 제어영역이라고 부른다. 이러한 이유때문에, 워커노드들을 각각의 VM 인스턴스로 생성하는 것 보다 GKE의 가격이 비싸다.

<br>

조금 더 자세히 살펴보면, 먼저 워커노드 및 컨트롤 플레인에 해당하는 CIDR에서 모든 인바운드 포트를 오픈해두었다. 이렇게 되면 위에서 확인한 모든 포트가 오픈되어 있기에 사용하는데 문제가 없다.



여기서 특이하게 위에서 보지 못한 10255 포트가 나오는데, Read-Only Kubelet API로 사용한다고 한다. 어떻게 사용되는지는 조금 더 찾아봐야 할 듯 하다.

**하지만 VPC에서 확인되는 방화벽은 GKE를 최초 생성할 때만 적용되는 듯 하다. 그래서 크게 의미 없는 듯 하다.**

> VPC에 있는 모든 방화벽 규칙을 삭제해도 정상적으로 동작한다.

<br>

워커 노드에 대해 코드로 정의된 내용을 보다 보면, 아래와 같이 10250포트를 사용함을 알 수 있다.

```yaml
  daemonEndpoints:
    kubeletEndpoint:
      Port: 10250
```



또한 GKE에서 endpoints를 조회해보면, `metrics-server`라는 이름으로 10250을 사용하는 것을 알 수 있다.

```shell
$ kubectl get endpoints -n kube-system metrics-server
NAME                     ENDPOINTS                                            AGE
metrics-server           10.32.0.5:10250                                      8m29s
```



GKE에서는 따로 컨트롤플레인에 대한 내용을 제공하지 않아서 이에 대한 포트는 확인 할 수 없는 듯 하다.

```shell
$ kubectl get endpoints -n kube-system calico-typha
NAME           ENDPOINTS                           AGE
calico-typha   10.178.0.21:5473,10.178.0.22:5473   2m21s
```

여기서 몇가지 설정을 변경하면 calico가 존재함을 알 수 있는데, 이 또한 알아두면 좋을 듯 하다.



```shell
$ kubectl get endpoints kubernetes
NAME         ENDPOINTS         AGE
kubernetes   10.178.0.20:443   38m
```

또한, 우리가 입력하는 `kubectl` 같은 명령어는 https를 이용하는 듯하다. ( http 방식 or gRPC 방식을 이용한다고 함 )



**뭔가 확인하고 싶은 내용들을 확인하지 못해서 아쉽다. 결론은, GKE에서는 VPC에서 모든 포트를 열어두고 있는 듯 하며 필요에 따라 NetworkPolicy등을 이용해서 제어하면 될 듯 하다.**



## 몇가지 추가 테스트

이대로 마무리하기 아쉬워서 직접 VM 인스턴스에 쿠버네티스를 띄어서 연결해보았다. ( 이 과정에서는 AWS를 이용했다. )

```shell
root@k8s-control-plane:~# kubectl get endpoints
NAME         ENDPOINTS           AGE
kubernetes   172.31.177.8:6443   5m19s
```

1.26 버전의 쿠버네티스에 containerd기반 노드들로 진행했는데, 위와같이 기대했던 6443포트가 있음을 확인했다.



```shell
root@k8s-control-plane:~# kubectl get pods -n kube-system
NAME                                        READY   STATUS    RESTARTS   AGE
coredns-787d4945fb-6c8fb                    1/1     Running   0          8m
coredns-787d4945fb-hmf5f                    1/1     Running   0          8m
etcd-k8s-control-plane                      1/1     Running   0          8m14s
kube-apiserver-k8s-control-plane            1/1     Running   0          8m12s
kube-controller-manager-k8s-control-plane   1/1     Running   0          8m12s
kube-proxy-ddxrj                            1/1     Running   0          8m
kube-proxy-n55x5                            1/1     Running   0          3m41s
kube-scheduler-k8s-control-plane            1/1     Running   0          8m12s
```

이제 컨트롤플레인이 조회가 되기에, etcd, apiserver, controller-manager, scheduler들의 내용을 확인해봤다.



```yaml
#etcd
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /health?exclude=NOSPACE&serializable=true
        port: 2381
        scheme: HTTP
```



```yaml
# apiserver
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 172.31.177.8
        path: /livez
        port: 6443
        scheme: HTTPS
```



```yaml
# controller-manager
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10257
        scheme: HTTPS
```



```yaml
# scheduler
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10259
        scheme: HTTPS
```

당연한 거지만, Docs에 나와있는 것 처럼, 해당 포트들을 사용하고 있다.

endpoints에서는 조회가 안되지만, pod들의 yaml파일을 열어보면 위와 같이 livenessProbe가 정의되어 있었다.

apiserver를 제외하고 나머지는 `127.0.0.1` 로 설정되어있는 것 보면, 보안을 극대화하고 싶은신 분은, 컨트롤 플레인의 6443포트만, 워커노드의 10250포트 및 사용하는 앱에 따른 포트만 오픈해주면 될 듯 하다.



### 마무리

이전 까지 매니지드 서비스만 이용해와서 한번도 생각해 본 적 없었는데, 공부하면서 조금 찾아보니 쿠버네티스랑 조금 더 친해진 듯 하다.

또한, 매니지드 서비스를 사용하는 것이 이렇게 편리한 일 임을 다시 생각하게 되었다.
