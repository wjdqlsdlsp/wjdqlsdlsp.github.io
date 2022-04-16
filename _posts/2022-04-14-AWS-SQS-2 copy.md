---


layout: post

title:  "AWS - SQS(boto3) Receive Message and Load"

date:   2022-04-14

categories: 데이터엔지니어

tags: 데이터엔지니어

image: /post_img/sqs.png

---

#### AWS - SQS(boto3) Send Message

[이전 포스팅](https://wjdqlsdlsp.github.io/%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%97%94%EC%A7%80%EB%8B%88%EC%96%B4/2022/04/12/AWS-SQS/)을 통해 Jetson Nano 환경 ( Ubuntu ) 에서 AWS SQS에 데이터를 보내는 것을 구현했습니다. 

지금부터 할 내용은 SQS로 부터 데이터를 받고, 이를 전처리하여 데이터베이스에 로드하는 과정입니다.

 <p align="center"><img src="/images/post_img/jetson.png"></p>

간단하게만 어떤 데이터를 수집하나 소개해드리면, 위의 사진처럼 객체 탐지를하고, 해당 객체의 위치(탐지 시간, 객체 left, top, width, height) 데이터를 수집합니다. 



 <p align="center"><img src="/images/post_img/sqs_queue.png"></p>

젯슨나노 환경으로부터, SQS로 데이터가 잘 전달됨을 확인 했습니다. 

( 테스트를 위해 최대 100번만 데이터를 전송하도록 셋팅했습니다. )

<br>

#### AWS - SQS(boto3) Receive Message

먼저 데이터를 전달 받기 위한 AWS Instance를 셋팅했습니다. ( 프리티어에서 작동하도록 설계 )

- t2.micro ubuntu 20.04 LTS

<br>

#### Update & Download

```shell
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install python3-pip

#install boto3
pip3 install boto3

#install sqlalchemy
pip3 install sqlalchemy

#install pymysql
pip3 install pymysql
```

sqs로 부터 데이터를 받아오기위한 boto3 및 데이터베이스에 로드하기 위한 sql 관련 라이브러리를 설치해줍니다.

<br>

#### credentials 정의

AWS boto3 를 사용하기 위해서는, credentials 를 정의해야합니다. 아마 저와 같은 환경을 구축하셨으면 .aws 폴더 및 credentials 파일이 없을 겁니다. 이를 생성해주면 됩니다.

```shell
mkdir .aws
touch credentials
vi credentials

# insert IAM
[Profile1]
aws_access_key_id=AWS_IAM_KEY
aws_secret_access_key=AWS_IAM_SECRET_KEY

cd ~
```

사용할 IAM 관련 내용을 정의해주면 됩니다. AWS에서 인스턴스 자체에 IAM를 부여했다면, 이과정은 생략해도 됩니다.

<br>

#### RDS 구축

이제 데이터를 적재할 RDS를 생성할 예정입니다. 저는 AWS RDS ( MySQL )를 구축했습니다.

RDS 구축 관련은 저의 이전 [포스팅 글](https://wjdqlsdlsp.github.io/%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%97%94%EC%A7%80%EB%8B%88%EC%96%B4/2022/03/25/AWS-RDS%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0_new/)을 참고해주세요!

RDS 구축이 완료되었다면, 사용해야 할 테이블 스키마를 정의해 줍니다.

```sql
create table TEST(
	date datetime,
	left_value float(8,4),
	top_value float(8,4),
	width_value float(8,4),
	height_value float(8,4),
	lon float(8,4),
	lat float(8,4),
	drone_height float(8,4),
	azimuth float(8,4)
);
```

저는 위와같이 테이블을 정의했습니다.

<br>

#### EC2 환경변수 셋팅

이번엔 환경변수 셋팅 단계입니다. 제가 사용하는 환경변수는 총 5개로 이후 boto3에서 필요로하는 정보들입니다.

```shell
export queue_url=sqs_url
export name=db_user_name
export password=db_password
export end_point=db_endpoint
export database_name=db_name
```

자신이 사용할 환경변수로 셋팅해주시면 됩니다. ( 이후 코드 공유를 위해서 보안상으로 사용한 단계입니다. )

<br>

#### Python 코드 작성

```python
import boto3
import os
import time
from time import strptime
import pymysql
import datetime
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Float, DateTime, Integer

os.environ['AWS_PROFILE'] = "Profile1"
count = 0
sqs = boto3.client('sqs', region_name='ap-northeast-2')
queue_url = os.environ.get("queue_url")
name = os.environ.get("name")
password = os.environ.get("password")
end_point = os.environ.get("end_point")
database_name = os.environ.get("database_name")

engine = create_engine(f'mysql+pymysql://{name}:{password}@{end_point}/{database_name}')

Session = sessionmaker()
Session.configure(bind=engine)
session = Session()
Base = declarative_base()

class TEST(Base):
    __tablename__ = "TEST"
    id = Column(Integer, primary_key=True)
    date = Column(DateTime)
    left_value = Column(Float)
    top_value = Column(Float)
    width_value = Column(Float)
    height_value = Column(Float)
    lon = Column(Float)
    lat = Column(Float)
    drone_height = Column(Float)
    azimuth = Column(Float)

def receive_message():
    # receive message
    response = sqs.receive_message(
            QueueUrl=queue_url,
            AttributeNames=['All']
        )['Messages'][0]
    receiptHandle = response['ReceiptHandle']

    body = response['Body']
    date = time_changer(body.split(" left")[0])
    value_data = body.split()
    left_value = value_data[6]
    top_value = value_data[8]
    width_value = value_data[10]
    height_value = value_data[12]
    insert_data(date = date, left_value=left_value, top_value=top_value, width_value=width_value, height_value=height_value)
    print("insert ok")
    # delete message
    response = sqs.delete_message(
                QueueUrl=queue_url,
                ReceiptHandle=receiptHandle
                )

def insert_data(date, left_value, top_value, width_value, height_value, lon=100, lat=100, drone_height=100, azimute=100):
    data = TEST(
        date=date,
        left_value=left_value,
        top_value=top_value,
        width_value=width_value,
        height_value=height_value,
        lon=lon,
        lat=lat,
        drone_height=drone_height,
        azimuth=azimute
    )
    session.add(data)
    session.commit()

def time_changer(str):
    str = str
    str = str.split()
    time = str[3].split(":")
    date = datetime.datetime(int(str[4]), strptime(str[1],'%b').tm_mon, int(str[2]), int(time[0]), int(time[1]), int(time[2]))
    return date

# 1분이상 메세지가 없으면 종료
while count < 60:
    try:
        receive_message()
        count=0
    except:
        time.sleep(1)
        count+=1

session.close()
```

위의 코드는 제가 사용하는 함수입니다. 필요에 맞춰 수정해서 사용하면 될 듯합니다.

<br>

#### 실행

마지막으로 정의한 python 함수를 ec2 인스턴스에 실행합니다.

```shell
python3 [python파일].py
```

이를 실행하고 정상적으로 데이터 베이스에 적재됬는지 확인합니다.

 <p align="center"><img src="/images/post_img/db_view.png"></p>

확인결과 위와같이 잘 적재됨을 알 수 있네요.

현재 테스트로는 양호하게 작동되지만, 만약 처리해야하는 데이터가 많아지거나 하면 Load balacer를 활성화 하든, kafka를 이용해서 데이터를 처리하든 방법을 생각해야할 것 같습니다.

마지막으로 더이상 AWS를 사용하지 않는다면 지금까지 사용한 인스턴스들을 삭제해줍니다.

<br>