---
layout: post
title:  "AWS - 웹사이트운영하기 (Django)"
date:   2021-11-13
categories: AWS
tags: AWS
image: /post_img/aws.jpg


---

# 아마존 웹서비스에 내가만든 웹사이트 올리기



매일 할 때마다 까먹어서, 이번에 제대로 정리해봅니다.



제가 사용한 환경은 다음과 같습니다.

- FrontEnd - HTML/CSS/JavaScripts
- BackEnd - Django - Python
- AWS OS - **Ubuntu Server 20.04 LTS (HVM)**



저는 EC2만 사용할 예정이며, S3 등의 사용법은 나중에 올리도록 하겠습니다.



먼저 올리고 싶은 웹사이트를 Github 등을 통해 올려둬야 진행이 가능합니다.

<p align="center"><img src="/images/post_img/my_aws1.PNG"></p>

저 같은 경우는, 프로젝트를 진행하면서 사용하는 Repository가 있기에 이를 이용하여 진행합니다.



#### 1. AWS 접속/인스턴스 시작

AWS 계정 만드는 방법은 생략하도록 하겠습니다.

AWS에 계정을 생성하고 접속하면 다음과같은 대시보드가 나타나는데

<p align="center"><img src="/images/post_img/my_aws2.PNG"></p>

인스턴스 시작 버튼을 통해서 OS를 생성해줍시다.



<p align="center"><img src="/images/post_img/my_aws3.PNG"></p>

클릭하면 다음과 같이 어떤 아마존 머신 이미지를 선택할 것인지 고르라고 합니다. 저희는 앞에서 말한 대로 , 우분투 서버 20.04 LTS를 선택해줍니다.

<p align="center"><img src="/images/post_img/my_aws4.PNG"></p>

이후 화면은 인스턴스 유형 선택인데, 본인의 운영목적에 맞게, 인스턴스 유형을 선택해주면 됩니다. 가격이 다르니, 가격표를 참고하세요! (저는 프리티어 클릭했습니다.)

<p align="center"><img src="/images/post_img/my_aws5.PNG"></p>

이후 

<p align="center"><img src="/images/post_img/my_aws6.PNG"></p>

다음으로 검토창이 뜨는데 여기서, 보안그룹 편집을 클릭해줍니다. (인스턴스 생성이후에도 바꿀수 있습니다.)

<p align="center"><img src="/images/post_img/my_aws7.PNG"></p>

장고는 8000포트를 이용하기 때문에 설정해주고, HTTP 80포트도 같이 열어주면 됩니다.  



<p align="center"><img src="/images/post_img/my_aws8.PNG"></p>

설정을 완료하면, 다음과같이 보안그룹에 포트가 생긴것을 확인할 수 있습니다. (두개씩 생기는건 각각 IPv4, IPv6 이니, 걱정안하셔도됩니다.)

설정을 완료하면 시작을통해 인스턴스를 시작하면됩니다. ( 키 페어 설정하셔야합니다! 그거 유출안되게 조심! )



#### Console 환경

<p align="center"><img src="/images/post_img/my_aws9.PNG"></p>

이후 인스턴스가 생성되면 인스턴스 ID를 클릭합니다. 이후 우측 상단에 연결을 클릭합니다.

<p align="center"><img src="/images/post_img/my_aws10.PNG"></p>

계속 연결을 눌러주시면, 터미널 창이 보여지는데, 리눅스환경에 맞춰서 해주시면됩니다.



```shell
sudo apt-get update
```

먼저 apt-get 업데이트 한번 해주시고 이전에 깃에 올려둔 것을 클론해주시면 됩니다.

```shell
git clone [깃헙 클론 주소]
```

이후 저는 Python3-pip를 설치해주었습니다.

```shell
sudo apt-get install python3-pip -y
```

그 뒤로는 로컬환경과 동일합니다. ( requirments를 생성해두셨다면 )

```shell
pip install -r requirments.txt
```

저는 Mecab를 사용하기 때문에 따로 설치를 해주었는데요, 다음 블로그를 참고해주시면 될 것 같습니다.

[[처음부터 시작하는 EC2] konlpy, mecab 설치하기(ubuntu)](https://yuddomack.tistory.com/entry/%EC%B2%98%EC%9D%8C%EB%B6%80%ED%84%B0-%EC%8B%9C%EC%9E%91%ED%95%98%EB%8A%94-EC2-konlpy-mecab-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0ubuntu)

저도 이 블로그에서 도움을 많이 받았어요! - 혹시 tweepy streamlistener 관련해서 오류나면 tweepy 버전을 3.10.0을 설치해주시면 오류가 안납니다.

장고를 포함해서 다 설치되었다면, 

```shell
python3 manage.py makemigrations
python3 manage.py migrate
python3 manage.py runserver 0.0.0.0:8000
```

를 통해서 실행하시면 됩니다.



추가로, 

- 서버를 계속 켜두고싶다면,

```shell
nohup python3 manage.py runserver 0.0.0.0:8000 &
```

- nohup을 통해 켜진 서버를 다시 끄고싶다면

```shell
ps -ef | grep "python3 manage.py runserver"
```

를통해서 8000포트 관련 번호를 다 kill 해주시면 됩니다.

<p align="center"><img src="/images/post_img/my_aws11.PNG"></p>

```shell
kill 21738
kill 21740
```



- 뒤에 port 번호 8000을 생략하고 싶다면, (리디렉션 방법)

```shell
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8000
```

80포트를 바로 8000포트로 연결해주는 방법입니다.

