---

layout: post

title:  "Kubernetes Cronjob을 이용한 대쉬보드 구축"

date:   2022-05-18

categories: 데이터엔지니어

tags: 데이터엔지니어

image: /post_img/k8s.png

---

아마 batch작업하면, Linux의 Crontab 혹은, Airflow를 생각하실 겁니다.

Crontab의 경우, Linux환경이라면 간단하게 shell 명령어로 실행할 수 있어서 매우 구현이 간단합니다.

하지만, 이는 모니터링에 단점이 있습니다. ( log를 txt에 기록하는 방법이 대표 )

Airflow의 경우 지정한 스케쥴에 따라서, 지정한 작업이 실행되고 매우 유용하여 많은 사람들이 즐겨 사용합니다.

특히 Airflow를 사용하면, 작업을 모니터링 할 수 있다는 장점이 있습니다.

모니터링의 장점으로 기업의 테크블로그를 보면, Kubernetes operator를 이용한 Airflow구조를 사용하고 있습니다.

하지만 구현하는 과정이 Crontab보다 훨씬 복잡합니다.

이 두개의 중간에는 오늘 시도해볼 Kubernetes Cronjob이 있습니다. ( 개인적인 의견입니다. )

간단하게 설명하면, Kubernetes에 Crontab이지만, Airflow만큼은 아니지만, 약간의 모니터링을 지원합니다.

지금부터, 간단한 대쉬보드를 만드는 과정을 정리하는 글입니다.

<br>

#### Python 코드 작성

먼저 Kubernetes환경에서, 동작할 코드를 작성해줍니다.

```python
print("hello world")
```

간단하게는 위와 같은 코드가 될 수 있겠습니다. 

저 같은 경우, 제가 진행하고 있는 프로젝트의 대쉬보드를 구현하는 코드를 만들었기 때문에 [코드](https://github.com/Sejong-Talk-With/Semogong_plot/tree/main/make_plot)가 깁니다. 

<br>

#### Docker Image 만들기

Kubernetes는 Image를 관리해주는 오케스트레이션 툴입니다.

위에서 작성한 코드가 이미지화 하여, 이를 이용하면 됩니다.

```dockerfile
#dockerfile
FROM python:3.9.10
ADD make_plot/ .
RUN pip install --trusted-host pypi.python.org -r requirements.txt
ENTRYPOINT ["python", "main.py"]
```

제가 사용한 dockerfile입니다.

간단하게 설명드리면, python image를 베이스로 사용합니다. 버전같은 경우 제가 로컬환경에서 사용한 파이싼 환경과 맞췄습니다.

이후 사용한 폴더를 추가해주고, 라이브러리를 설치해줬습니다. 마지막으로, main.py를 실행하는 dockerfile입니다.

<br>

#### Docker build

이제 작성한 코드를 빌드할 차례입니다.


```shell
docker build --platform amd64 --build-arg DEPENDENCY=build/dependency --tag make_plot:latest .
```

제가 사용한 환경은 m1맥 환경이라, platform을 설정해줘야 정상적으로 작동하더라구요.

```shell
docekr build --tag make_plot:latest
```

만약 환경이 상관없으면 위와 같이 간단하게 build 할 수 있습니다.

<br>

#### Docker image to Dockerhub

이후 이미지화된 이미지를 도커허브로 전송합니다.

도커 로그인이 완료된 상태라고 가정하고 진행합니다.

```shell
docker tag d5dfee69af44 wjdqlsdlsp/make_plot
```

먼저 tag작업을 진행합니다. 중간에 d5dfee69af44는 이미지ID입니다.

<br>

위와 같이 설정이 완료되었다면, push해줍니다.

```shell
docker push wjdqlsdlsp/make_plot
```

이과정 까지 정상적으로 진행했다면, 도커허브에 접속해서 정상적으로 전송됬는지 확인합니다.

<br>

#### Kubernetes 설정

이제 본격적인 Kubernetes를 설정할 차례입니다. 저는 GCP 환경에서 진행했습니다.

yaml을 통해서 제어할 예정이므로 아래와 같은 yaml파일을 설정해줍니다.

제가 설정한 내용은, 정각마다 실행한다는 내용입니다.

```shell
#cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: semogong-dash
spec:
  schedule: "0 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: semogong-dash
            image: wjdqlsdlsp/make_plot:latest
            envFrom:
            - configMapRef:
                name: config-dev
          restartPolicy: OnFailure
```

<br>

#### 실행하기

```shell
kubectl create -f cronjob.yaml
```

이제 위에서 정의한 yaml 파일을 이용하여 쿠버네티스에게 작업을 하도록 시킵니다.

<br>

```shell
☁  ~  kubectl get cronjob
NAME            SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
semogong-dash   0 * * * *   False     0        38m             14d
```

작업이 잘 되나 확인하려면 kubectl get cronjob으로 확인해줍니다. 저는 이전에 실행시킨 작업이라 AGE가 14d라고 되어있네요. 최근 실행된 것은 38분이 지났다는 내용이 보입니다.

<br>

```shell
☁  ~  kubectl get pods
NAME                           READY   STATUS      RESTARTS   AGE
semogong-dash-27547800-td5vl   0/1     Completed   0          159m
semogong-dash-27547860-4xbbh   0/1     Completed   0          99m
semogong-dash-27547920-b7vjn   0/1     Completed   0          39m
```

앞의 모니터링이 일부가능하다고 말한 이유는 위와 같습니다. 현재 max 값을 3으로 설정하여 3개까지의 실행결과를 저장합니다.

STATUS를 보면, Completed라고 나오는데 정상적으로 pod가 실행되고 종료됬다는 내용이고 동일하게 AGE가 표시됩니다.

<br>

#### 삽질한 내용

Cronjob시간 설정에 있어서 로컬타임이 아닌, UTC기준으로 스케쥴링 되더라구요. 이 때문에 시간 맞추는데 힘들었습니다.

시간설정이 힘들다면 [Crontab guru](https://crontab.guru/) 를 통해서 어떻게 표기할 지 정해보세요. 지인이 추천해줬는데 유용하더라구요.

<br>
