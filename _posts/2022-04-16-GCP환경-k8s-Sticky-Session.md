---


layout: post

title:  "GCP-k8s Service배포 및 Sticky Session"

date:   2022-04-16

categories: 데이터엔지니어

tags: 데이터엔지니어

image: /post_img/k8s.png

---

#### 개요
이번 포스팅에는 GCP Kubernetes engine을 이용해서 배포하는 과정을 설명합니다.

<br>

#### GCP Kubernetes engine 구축

먼저 사용할 Public Cloud 서비스는 GCP입니다. 

k8s를 배포하기 위해 AWS와 GCP 중 고민을 했지만, GCP에서 제공해주는 300달러 크레딧을 사용하기 위해서 GCP Kubernetes engine으로 선택했습니다. 

AWS의 Kubernetes서비스인 EKS는 프리티어에 해당하지 않습니다. ( 돈이 없네요... )

<p align="center"><img src="/images/post_img/gcp1.png"></p>

Cloud에서 Kubernetes 환경을 구축하는 방법은 간단합니다. GCP 홈페이지에서 Kubernetes Engine을 검색해서 만들기를 생성하면 원하는 환경을 구축할 수 있습니다.

이를 집접 VM을 통해 구축하면 연결하는 과정이 있어서 복잡하고 시간이 소요되는데 이러한 점을 간단히 해결할 수 있습니다.

저는 GKE Standard를 통해서 구축했습니다.

<p align="center"><img src="/images/post_img/gcp2.png"></p>

제가 변경한 설정은 리전 하나입니다. 서울을 서버로 사용하기 위해서 asia-northeast3로 변경하고 나머지 옵션은 default값으로 설정하였습니다. 

옵션들을 살펴보니, 노드에 대해 어떤 용도로 사용하는지(일반 목적, 컴퓨팅 최적화, 메모리 최적화, GPU) 혹은, 얼마나 많은 리소스를 필요로하는지(vCPU, Memory 등) 을 선택 할 수도 있고, 스토리지를 얼마나 사용하는지, 보안은 어떻게 하는지, 메타데이터 (config)등을 사용할 건지, 많은 옵션이 제공됩니다. 필요에 따라서 원하는 옵션을 설정하여 사용하면 될 것 같습니다.

설정을 완료하고 기다리면 몇분 소요후 완성이 됩니다. (직접 노드를 연결하고 하려면 시간이 소요되는데 Cloud서비스에 감사하네요)

<br>

#### GCP CLI 접속하기

<p align="center"><img src="/images/post_img/gcp3.png"></p>

CLI를 접속하는 방법 또한 간단합니다. 접속하고자하는 쿠버네티스 클러스터에 오른쪽 점세개 있는것을 클릭하고 연결을 클릭합니다.

연결을 클릭하면 위와같이 모달이 뜰텐데, CLOUD SHELL에서 실행을 누르면 GCP 하단에 터미널 창이 나오고,

<p align="center"><img src="/images/post_img/gcp4.png"></p>

```shell
gcloud container clusters get-credentials ....
```

위와같은 코드가 입력되어 있을 겁니다. 이것은 마스터 노드 및 워커 노드들을 사용하기 위해 권한을 부여해주는 코드이니 엔터 누르고 실행해주시면 연결이 됩니다. ( 승인 창이 뜰텐데, 승인 버튼도 눌러주세요 ! )

```shell
kubectl get pods
```

정상적으로 잘 되나 실행하기 위해 위의 코드를 한번 입력해서 테스트도 해줍니다. (파드가 없다고 뜨면 정상입니다.)

<br>

#### yaml 파일 받아오기

저같은 경우는 docker hub에 있는 이미지를 사용하기 위해서 도커허브로 부터 불러올 예정입니다.

```shell
kubectl run semogong --image=[이미지경로] --port 8080 --dry-run -o yaml > [yaml파일이름].yaml
```

제가 사용할 이미지는 spring boot 기반의 이미지이므로 port를 8080으로 미리 설정했습니다. 

--dry-run 코드같은경우, 해당 커맨드가 실행가능한지 확인 할 수 있는 코드인데, 이를 -o옵션을 추가해 yaml파일로 얻을 수 있습니다.

위의 코드를 성공적으로 실행하면, 현재 경로에 yaml 파일이 있음을 확인 할 수 있을겁니다.

이제 이 yaml 파일을 이용해서 서비스를 만들 예정이므로 이를 수정해봅시다.

<br>

#### Deployment 만들기

쿠버네티스를 이용해서 배포하는 방법이 여러가지가 있겠지만, 저는 Deployment를 먼저 정의하고 이를 서비스화 시킬 예정입니다.

이 과정에서 이전 단계에서 얻은 yaml 파일을 vim을 이용해서 수정합니다.

```shell
vi [yaml파일이름].yaml
```

vim환경에 익숙하신분들이 많겠지만, 간단한. 커맨드만 소개해드리면, i 를 입력하면 insert모드로 전환되어 수정할 수 있습니다. 이후 수정이 완료되면 esc를 눌러서 insert 모드를 해제하고, Shift + z z 누르면 저장이 됩니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: semogong
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web 
  template:
    metadata:
      name: semogong-pod
      labels:
        app: web
    spec:
      containers:
      - name: semogong-container
        image: wjdqlsdlsp/semogong
        env:
        - name: {환경변수key}
          value: {환경변수value}
        ports:
        - containerPort: 8080
```

참고하시라고 제가 사용한 yaml 파일을 공유해드립니다. Replicas 3으로 설정해서 총 3개의 파드를 유지할 예정입니다. 

Selector를 잘 생성하셔야 error없이 잘 실행되니 주의해주세요!

<br>

#### Deployment 생성

```shell
kubectl create -f [yaml파일].yaml
```

생성하는 방법은 create -f 커맨드를 통해서 생성합니다. 

```shell
kubectl get pods
kubectl get deployments
```

이후 잘 생성됬나 확인을 위해

위와 같은 커맨드를 입력해서 running 상태인지 확인해보세요!

<br>

#### Service 만들기

```shell
kubectl expose deployment [deployment이름] --name=[서비스이름] --type=LoadBalancer --port 80 --target-port 8080
```

서비스 만드는 방법은 위의 코드를 통해서 생성할 수 있습니다.

저는 포트포워딩을 위해 target-port 및 port 옵션으로 추가했습니다. 타입같은 경우, 웹서비스로 이용할 예정이라 LoadBalancer로 설정했습니다.

```shell
kubectl get service
```

이후 기다리다가 위의 커맨드를 통해서 External-ip로 접속하면 정상적으로 실행이 됩니다.

<br>

#### 방화벽 규칙

만약 위의 단계에서 실행이 안된다면 방화벽 규칙을 수정해야합니다. 

GCP에서 firewall 검색하시면 방화벽 규칙으로 이동할 수 있는데, 사용할 포트를 오픈하면 됩니다. 저같은 경우는 80포트, 8080포트, 3306(mysql)포트를 오픈해서 사용하고 있습니다. ( 방화벽 규칙 생성 후 네트워크 연결도 잊지마세요. )

<br>

#### Sticky Session

웹서비스를 테스팅하던 도중, 세션연결이 다른 노드로 변경되는 문제가 발견되었습니다.

사실 당연한건데 이를 놓치고 있었습니다. AWS에서 로드밸런서를 사용해 보신분들은 아시겠지만, Sticky Session 옵션이 있습니다.

Client ip를 이용해서 설정한 시간동안은 이전에 접속한 노드로만 접속하게 트래픽을 넘겨주는 방법입니다.

찾아보니 당연히 쿠버네테스 설정에도 있더라구요. 

GCP Kubernetes Engine 에서 좌측 사이드바에서 서비스 및 수신을 클릭하면 실행한 서비스가 있습니다. 이름을 클릭하여 세부정보를 볼 수 있는데, 그중 YAML이라고 있는 옵션이 있습니다. 

YAML의 하단을 보면, spec부분의 sessionAffinity라는 옵션이 있습니다.

당연히 설정을 안했기 때문에, None으로 설정되어 있더라구요, 이를 변경해 줍시다.

```yaml
sessionAffinity: ClientIP
sessionAffinityConfig:
	clientIP:
		timeoutSeconds: 10800
type: LoadBalancer
```

sessionAffinity옵션을 ClientIP로 설정하고, Config를 설정해줍니다. timeoustSeconds로 설정한 기간동안은, 트래픽이 동일한 노드로 전송됩니다.

<br>

#### Rolling update

프로젝트를 진행하다보면 당연히 웹사이트를 업데이트할 일이 많습니다.

쿠버네티스는 이를 지원하는 롤링 업데이트가 있습니다. 이를 이용하면 무중단 상태로 업데이트를 진행할 수 있는 아주 좋은 기능이죠

```shell
kubectl set image deployment.v1.apps/[디플로이먼트이름] [container이름]=[변경할 이미지]
#kubectl set image deployment.v1.apps/semogong semogong-container=wjdqlsdlsp/semogong:latest
```

방법은 위의 코드를 이용해서 업그레이드합니다. 제가 사용한 코드를 첨부해드립니다. 위의 yaml파일과 비교해보세요. 

도커 허브에 이미지를 업데이트 한 상황에서 위와 같이 입력하면 새로 올라온 이미지로 Rolling update가 진행됩니다.

<br>

#### Pod 한국 시간대로 변경하기

아무 설정도 하지않았을 경우, Pod (Container)의 시간대는 UTC를 사용합니다.

이를 한국 시간대로 바꾸는 방법이 여러 방법이 있는데, 저는 환경변수로 시간대를 입력해주는 방법을 사용합니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: semogong
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web 
  template:
    metadata:
      name: semogong-pod
      labels:
        app: web
    spec:
      containers:
      - name: semogong-container
        image: wjdqlsdlsp/semogong
        env:
        - name: TZ
          value: Asia/Seoul
        ports:
        - containerPort: 8080
```

이전에 Deployment를 정의하기 위한 yaml 파일입니다. 여기서 환경변수로 값을 넘겨주면 한국 시간대로 설정이됩니다.

spec -> template -> containers -> env 부분을 참고해주세요 !