---

layout: post

title:  "Kubernetes Configmap 환경변수 등록"

date:   2022-05-16

categories: 데이터엔지니어

tags: 데이터엔지니어

image: /post_img/gcp.png

---

인스턴스에서 환경변수를 등록하는 방법은 간단합니다.

우분투 환경의 경우, 

```shell
export A=abc
```

위와 같이, export 명령어를 통해서 임시로 변수를 등록하거나

```shell
vi ~/.bashrc
```

bashrc에 export 명령어를 통해 인스턴스 시작시마다 작동하도록 설정하면 됩니다.

하지만 쿠버네티스 특히, GCP같은 환경에서는 어떻게 해야 할까요?

위와 같은 작업을 master node에서 한다고해도, worker node로 전달이 되지 않습니다.

이를 전달해주는 방법은 간단합니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: DEMO_FAREWELL
      value: "Such a sweet sorrow"
```

위와 같이, yaml파일을 정의할 때, env를 추가해서 pod를 생성과 동시에 환경변수를 넘겨 줄 수 있습니다.

하지만 해당 정보가 민감한 정보이며, 이러한 글을 퍼블릭환경 깃에 업로드 한다고 하면, 보안상 끔찍한 일이 벌어집니다.

이를 해결하기 위해서 Configmap을 사용하면 간단합니다.

<br>

#### Configmap yaml 정의하기

Configmap 또한 yaml 파일로 정의할 수 있습니다.

[arisu1000 님의 블로그](https://arisu1000.tistory.com/27843) 를 참고하여 정리하였습니다.

```yaml
#config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config
  namespace: default
data:
  DB_NAME: admin
  DB_PASSWORD: password
```

위와 같이, yaml파일을 정의합니다.

이제 정의한 파일을 실행시켜줍니다.

```shell
kubectl apply -f config.yaml
```

실행방법은 위와같이, 다른 yaml파일 실행방식과 동일합니다.

<br>

#### Configmap 변수 사용 정의

이렇게 configmap을 구성했다면, 해당 변수를 사용하겠다고 사용할 deployment에 등록해줘야합니다.

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
        image: wjdqlsdlsp/semogong:1.6
        ports:
        - containerPort: 8080
        envFrom:
        - configMapRef:
            name: config
```

이는 제가 현재 진행 중인 프로젝트의 deployment 파일 정의입니다.

configmap관련 정의는 아래에서 3번째 줄부터 보시면 됩니다.

```yaml
envFrom:
- configMapRef:
    name: config
```

위와 같은 정보를 등록하면 환경변수로 사용이 가능합니다.

configmap을 통해 정의했기 때문에 걱정없이 내용을 공유할 수 있습니다.

또한, 여러번 정의하지 않아도 되서 클린코드가 되고 장점이 많은 것 같습니다.

<br>

