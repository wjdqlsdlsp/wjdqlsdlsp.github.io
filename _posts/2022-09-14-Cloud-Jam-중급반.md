---
layout: post
title:  Cloud jam 중급반(kubernetes) - 몰랐던 내용들
categories: [kubernetes]
tags: [kubernetes]
---



본 포스트는, Google Cloud에서 진행한 Cloud Jam 중급반(GKE)을 통해 배운 몰랐던 내용들을 정리했습니다.

<br>

#### Pod

**pod는 kubernetes를 이루는 가장 작은 단위이다.**

>  당연히 컨테이너인 줄 알았는데, GCP에서는 파드와 컨테이너를 동일 선 상에서 보는 듯하다. ( 파드가 곧 컨테이너고 컨테이너는 파드다 라고 설명하네요. 이건 사람에 따라 다를 것 같습니다. )



<br>

**pod는 스스로 selfhealing 할 수 없는 객체이다.**

따라서 pod를 효율적으로 배포하기 위해서는 아래와 같은 Controller objects 를 응용하는 것이 좋다.

- Deployment
- StatefulSet
- DeamonSet
- Job

> 항상 위에서 설명하는 리소스들을 사용해와서, Pod가 selfhealing을 할 수 없는 줄 몰랐습니다.

<br>

#### yaml 파일 이용하기

kubernetes의 리소스를 생성하는 방법은 크게 2가지로 나눠집니다.

- cli를 통한 생성

```shell
kubectl run nginx -n namespace image=nginx:latest
```



- yaml 사용

```shell
kubectl create -f nginx_deployment.yaml -n namespace .
```

<br>

선택은 자유지만, yaml파일을 통해서 리소스를 생성하는 것을 권장합니다.

만약 cli를 통해서 직접 생성한다면, cli에 인자로 입력해야하는 설정값이 많아지며, 일회성이기에 리소스 관리가 힘들어집니다.

(물론, kubernetes에 리소스가 남아서 명령어를 통해 조회하면, yaml 소스를 볼 수는 있습니다.)



Cloud Jam 강사분은, cli를 통한 입력은 간단한 작업에서만 사용하라고 권장하고 있습니다.

<br>



#### namespace

네임스페이스를 지정할 때, 가장 좋은 방법은 cli로 입력 받는 방법입니다.

```bash
kubectl -n demo apply -f mypod.yaml
```

만약 yaml 파일에서 지정하면 이후에 수정하기가 힘들어 집니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: demo
```

문제가 생기는 것은 아니지만, **less flexible**이라고 설명

<br>

위에 내용을 보고, **"yaml 파일에 namespace를 정의하고 cli에서 -n 인자로 추가 입력받으면 오버라이딩되서 상관없지 않을까?"** 라고 생각해서 테스트해봤습니다.



```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: test
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

사용한 yaml 파일은 kubernetes에서 거의 교보재처럼 사용하는 nginx deployment에 namespace만 추가했습니다.



이후 `kubectl create -f nginx-deployment -n test-other` 명령어를 통해 생성하려고 시도했습니다.

```shell
$ kubectl create -f nginx_deployment.yml -n test-other

error: the namespace from the provided object "test" does not match the namespace "test-other". You must pass '--namespace=test' to perform this operation.
```

결과는 위와 같이, 아예 생성이 안되었습니다.



> 결론: namespace는 cli에서 입력받자. less flexible를 피하자!
<br>
