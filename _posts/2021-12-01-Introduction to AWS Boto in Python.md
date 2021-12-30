---
layout: post
title:  "Introduction to AWS Boto in Python"
date:   2021-12-26
categories: 데이터엔지니어
tags: 데이터엔지니어
image: /post_img/oop.jpg
---


# Introduction to AWS Boto in Python

  <br>

#### Putting Files in the Cloud!

AWS는 인터넷의 중추를 담당하고 있고, Boto3를 사용하면, laptop의 확장으로 사용할 AWS의 기능을 활용할 수 있습니다. ( 자동 보고서 작성, 감정 분석 수행, 알림 전송 등 )




Python에서 AWS와 상호 작용하기 위해 Boto3라이브러리를 사용합니다.

```python
import boto3

s3 = boto3.client('s3',
                region_name='us-east-1',
                aws_access_key_id=AWS_KEY_ID,
                aws_secret_access_key=AWS_SECRET) 

response = s3.list_buckets()
```

사용법은 위와 같습니다.

먼저 `boto3`를 선언하고, AWS 서비스 이름, 리전, 키 및 시크릿를 통해 클라이언트를 초기화 합니다.

<p align="center"><img src="/images/post_img/boto1.png"></p>

Boto3의 키/시크릿을 생성하기 위해 IAM(Identity Acess Management Service)를 사용합니다.



#### 권한 부여 방법

<p align="center"><img src="/images/post_img/boto2.png"></p>

아마존 IAM 사용자에 다음과 같은 권한을 부여합니다.





#### AWS service

- IAM : 이전에 알아본 것처럼 Identity Acess Management Service 의 약어로, 권한등을 관리합니다.

- S3 : Simple Storage Service로, 파일을 클라우드에 저장할 수 있습니다.

- SNS : Simple Notification Sevice로, 이메일과 문자를 보낼 수 있습니다. (이를 이용하여, 데이터 파이프 라인의 이벤트 및 조건을 기반으로 가입자에게 경고합니다.)

- Comprehend : 텍스트 블록에 대한 감정 분석을 수행합니다.

- Rekognition : 이미지에서 텍스트를 추출하고 사진에서 고양이를 찾습니다.
  
  <br>



#### Diving into buckets

S3를 사용하면, 클라우드에 파일을 넣고, URL을 통해 전 세계 어디서나 액세스 할 수 있습니다.

클라우드 스토리지 관리는 데이터 파이프 라인의 핵심 구성 요소입니다.



#### S3 Components - Buckets

S3의 주요 구성 요소는 버킷 및 객체입니다.

- 버킷은 데스크탑의 폴더와 같습니다. ( 객체는 해당 폴더 내의 파일과 같습니다. )

- 버킷에는 자체 권한 정책이 있습니다.

- 정적 웹 사이트의 폴더 역할을 하도록 구성 할 수 있습니다.

- 로그를 생성할 수 있습니다.



#### S3 Components - Objects

버킷이 수행하는 가장 중요한 것은 객체를 포함하는 것 입니다.

객체는 이미지, 비디오, 파일, CSV, 로그 파일드잉 될 수 있습니다.



#### What can we do with buckets?

- 버킷을 생성

- 버킷을 나열

- 버킷을 삭제



#### 버킷 생성

```python
import boto3
s3 = boto3.client('s3', region_name='us-east-1',
                        aws_acess_key_id=AWS_KEY_ID,
                        aws_secret_access_key=AWS_SECRET)
bucket = s3.create_bucket(Bucket='gid-requests' )
```

이전과 동일하게, 클라이언트를 생성한 뒤, gid-requests 버켓을 생성합니다.  (버킷을 생성할 때, 버킷의 이름은 모두 고유해야합니다. 안그러면 오류 생깁니다.)



#### 버킷 나열

```python
import boto3
s3 = boto3.client('s3', region_name='us-east-1',
                        aws_acess_key_id=AWS_KEY_ID,
                        aws_secret_access_key=AWS_SECRET)
bucket_response = s3.list_buckets()
buckets = bucket_response['Buckets']
print(buckets)
```



#### 버킷 삭제

```python
import boto3
s3 = boto3.client('s3', region_name='us-east-1',
                        aws_acess_key_id=AWS_KEY_ID,
                        aws_secret_access_key=AWS_SECRET)
response = s3.delete_bucket('gid-requests')
```
