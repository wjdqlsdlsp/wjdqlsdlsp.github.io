---
layout: post
title:  "lambda container 배포하기"
tags: [AWS]
categories: [AWS]
---

AWS의 대표적인 serverless 서비스인 lambda에는 흔히 우리가 아는 코드로 배포하는 방식과 컨테이너로 배포하는 방식이 있다.

<p align="center"><img src="/assets/img/post_img/lambda-container-1.png"></p>

[AWS 2020:reivnet에서 새로 공개](https://aws.amazon.com/ko/blogs/korea/new-for-aws-lambda-container-image-support/)되고 현재는 많은 사람들이 사용하고 있다.

lambda container이미지를 이용한 서비스에 대한 내용은 발표한 영상들이 많이 있어서 참고하면 좋을 듯하다.

- [AWS Lambda 컨테이너 이미지 서비스 활용하기 – 김태수:: AWS Builders Online Series](https://www.youtube.com/watch?v=tTg9Lp7Sqok&ab_channel=AmazonWebServicesKorea)
- [컨테이너 AWS Lambda Container를 활용한 서버리스 아키텍처 배포 및 성능 향상 - 이민재, Superb AI](https://www.youtube.com/watch?v=FTei6vum5kE&ab_channel=AWS%ED%95%9C%EA%B5%AD%EC%82%AC%EC%9A%A9%EC%9E%90%EB%AA%A8%EC%9E%84-AWSKRUG)

lambda의 장점은 가져가고, 이미지 기반으로 동작한다는 장점까지 챙길 수 있는 좋은 선택인 것 같다.



간단하게 사용법을 공유해보자 한다.



## 이미지 빌드

이미 많은 사람들이 도커에 익숙해서 오히려 코드 기반의 lambda를 사용하는 것보다 편할 수도 있다.

```shell
$ tree
.
├── Dockerfile
└── main.py
```

최대한 간단하게 구성하고 싶어서 파일은 두개만 사용했다.



먼저 파이썬 파일은 단순하게 인자로 받는 event와 context를 출력한다.

성공적으로 종료될 경우, statusCode: 200을 반환하는 매우 간단한 함수이다.

```python
def handler(event, context):
    print("event: ", event)
    print("context: ", context)

    return {'statusCode': 200}
```



다음으로 Dockerfile이다.

```dockerfile
FROM amazon/aws-lambda-python:3.8

COPY main.py /var/task/

CMD ["main.handler"]
```

도커파일도 간단하게 작성했다. 여기서 웬만하면 이미지는 aws에서 제공하는 것을 추천한다고 한다.

그리고 동작할 함수 파일의 경로가 `/var/task` 로 설정되어 있어야 한다. 물론 바꿀수는 있지만 default를 추구해서 냅뒀다.

마지막으로 CMD명령으로 파일명.함수명을 입력해주면 준비가 완료된다.



이후 단계는 도커이미지를 빌드하면 된다.

```shell
docker build --platform linux/arm64/v8 -t <이미지이름>:<태그> .
```

나는 m1유저라 따로 platform을 지정해줬는데, 에러가 안난다면 그대로 진행해도 될 듯 하다.





## 테스트

여기서 lambda container의 장점이 나오는데, 잘 동작하는지 테스트하고 싶으면 아래와 같이 진행하면 된다.

```shell
docker run -p 9000:8080 <이미지이름>:<태그>
```

로컬에서 도커를 실행시켜서 빌드한 파일을 실행한다.



```shell
curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d '{"talk": "Hello World"}'
```

그리고 다른 터미널창을 또 열어서 위와 같이 post를 보내본다.



```shell
START RequestId: a597fb7c-1e97-495b-87fb-e490e48a23de Version: $LATEST
event:  {'talk': 'Hello World'}
context:  <__main__.LambdaContext object at 0x400d38ba30>
END RequestId: a597fb7c-1e97-495b-87fb-e490e48a23de
REPORT RequestId: a597fb7c-1e97-495b-87fb-e490e48a23de	Init Duration: 3.02 ms	Duration: 675.96 ms	Billed Duration: 676 ms	Memory Size: 3008 MB	Max Memory Used: 3008 MB
```

그럼 서버에서 위와같이 응답을 보낸 것을 확인 할 수 있다. 만약 Error가 발생하면 이를 고쳐주고 다시 빌드하면 된다.

이에 대한 내용은 [AWS Docs](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/images-test.html)에 잘 나와있다.



## ECR에 이미지 push

lambda에서 컨테이너 기반으로 동작하려면 이미지를 가져오는 공간이 필요한데, ECR을 이용한다.

당연히 이 단계에서 aws config가 정의되어 있어야 동작하니 설정이 안되어있다면 설정을 진행한다.

이 또한 AWS Docs에 잘 설명되어 있다. [구성 및 자격 증명 파일 설정](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-configure-files.html)



설정이 완료되었다면, ECR을 생성해 준다.

ECR생성의 경우 콘솔에서 진행했으며, 프라이빗으로 설정했고, 리포지토리 이름만 지정하고 모두 디폴트값으로 설정했다.

<p align="center"><img src="/assets/img/post_img/lambda-container-2.png"></p>



ECR이 생성완료되었으면 이제 이미지를 push할 차례인데, 아래의 과정을 따르면 된다.


```shell
docker tag <이미지이름>:<태그> <계정-ID>.dkr.ecr.ap-northeast-2.amazonaws.com/<리포지토리이름>
```



```shell
aws ecr get-login-password --region <리전이름> | docker login --username AWS --password-stdin <계정-ID>.dkr.ecr.ap-northeast-2.amazonaws.com
```



```shell
docker push <계정-ID>.dkr.ecr.ap-northeast-2.amazonaws.com/<리포지토리이름>
```

위의 과정들을 정상적으로 완료하면 ECR에 이미지가 푸쉬되었을 것이다.



## lambda 함수 만들기

<p align="center"><img src="/assets/img/post_img/lambda-container-3.png"></p>



이제 마지막으로 lambda 함수를 만들면 되는데, 이전에 푸쉬한 ECR 이미지를 선택해주면 된다.

이 과정에서 권한 문제로 생성이 안될 수도 있는데, 해당 람다에게 적절한 권한을 주어지면 생성이 되니 당황하지 말자.

만약 `Entrypoint` 및 `CMD` 등을 수정하고 싶으면 오버라딩 되는 듯하다.



이제 우리가 이전에 사용했던 lambda와 동일하게 동작한다.



## 정리

AWS의 lambda를 컨테이너 이미지로 사용하는 방법을 간단하게 알아봤는데, 이제 "이를 왜 사용하는가?" 이다. 내가 처음 들었던 의문 중 하나이다.

AWS에는 ECS라는 컨테이너 기반의 온디맨드도 있고, Fargate라는 컨테이너 기반의 서버리스도 존재한다.

[AWS에서는 이에 대해 용도에 따라 적합한 서비스를 사용](https://aws.amazon.com/ko/blogs/korea/how-to-choose-aws-container-services/)하는 것이 가격적이나, 성능적이나 좋다고 한다.



간단하게 정리하면

ECS - 대규모 환경에서 사용하기를 권장

Fargate - ECS의 서버리스로, 고객이 컨테이너 배포 및 운영은 AWS에 맡기고, 실행하는 경우

Lambda Container - Lambda의 요청 시에만 코드를 실행하는 경우



즉, 트리거 조건이 있을 경우, lambda를 사용하는게 적합할 듯 하다.
