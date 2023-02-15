---
layout: post
title:  "AWS RDS 구축하기"
categories: [AWS]
tags: [AWS]
---


<p align="center"><img src="/assets/img/post_img/rds.png"></p>


RDS는 AWS에서 운영하는 관계형 데이터베이스 서비스입니다.

이를 이용해서 클라우드 환경에서 데이터 베이스를 운영할 수 있습니다.

<br>

#### RDS 선택하기
<p align="center"><img src="/assets/img/post_img/rds1.png"></p>

AWS에서 제공하는 RDS는 위와같이 6개가 있습니다.

Aurora는 AWS에서 만든 데이터 베이스로, 매우 좋은 성능과 높은 가용성을 자랑합니다. ( 하지만 비쌉니다. 본인의 상황을 잘 알아보고 선택하세요. )

이번 실습에서는 MySQL를 사용합니다. - 프리티어를 제공하므로, 무료로 사용할 수 있습니다.

<p align="center"><img src="/assets/img/post_img/rds2.png"></p>

저처럼 프리티어를 사용하실 분은, 템플릿에서 프리티어를 설정하면 알아서 세팅이 됩니다.

<p align="center"><img src="/assets/img/post_img/rds3.png"></p>

이후 설정칸이 나오는데, 이부분은 중요하니 마스터 사용자 이름 및 마스터 암호를 잘 설정하고 기억해주세요.

이후 데이터 베이스 연결을 위해서 사용합니다.

<p align="center"><img src="/assets/img/post_img/rds4.png"></p>

스토리지 설정 전에, DB인스턴스 클래스 부분이 나오는데, 프리티어는 db.t2.micro인스턴스만 사용가능합니다. 프리티어가 아니라면, 원하시는 인스턴스를 사용하시면 됩니다. ( 단, 성능이 좋으면 가격이 비쌉니다. )

스토리지에서는 스토리지 유형, 용량, 자동 조정 설정등이 나오는데, gp2가 가장 대중적으로 사용됩니다. io1 나 io2의 경우 높은 전송속도를 요구하는 상황에서 사용하면 적합합니다.

스토리지 자동 조정 활성화를 설정하면, 활당된 스토리지가 부족하다고 판단되면 자동으로 크기를 증가시킵니다.

<p align="center"><img src="/assets/img/post_img/rds5.png"></p>

가용성 및 내구성 설정도 프리티어에서는 단일 AZ만 지원하기 때문에 해당되지 않습니다.

이후 연결 설정이 나오는데, VPC 및 서브넷 그룹, 퍼블릭액세스, 보안그룹등을 세팅할 수 있습니다.

설정을 변경하고자 하는 분은 변경해주세요! ( 본 실습에서는 간단한 데이터베이스만 운영하기에 변경하지 않습니다. )

저는 제 로컬 컴퓨터에서 RDS에 데이터를 저장할 예정이기 때문에 퍼블릭 액세스 설정을 예로 설정했습니다

<p align="center"><img src="/assets/img/post_img/rds6.png"></p>

데이터베이스 인증은, 위와 같이 3개의 셋팅이 있습니다.

보안에 맞게 설정하면 됩니다. ( 저는 가장 간단한 암호 인증만 사용했습니다. )

이후 보안을 위해 IAM 및 Kerberos인증방법도 사용해보세요.

<p align="center"><img src="/assets/img/post_img/rds9.png"></p>

추가구성을 통해 초기 데이터베이스 이름을 설정해야합니다. 이후 연결할 때, 데이터베이스 이름을 입력받기 때문에 꼭 설정하셔야합니다.

이후 아래 백업 및 스냅샷은 상황에 맞게 설정해 주시면 됩니다. 설정이 끝났으면 생성을 눌러주세요.

데이터베이스 생성을 하고 기다리면 데이터 베이스가 실행됩니다. ( 다소 시간이 걸리니 기다려주세요. )

<p align="center"><img src="/assets/img/post_img/rds7.png"></p>

데이터베이스가 생성되면, 엔드포인트 부분을 통해서 rds에 접속할 수 있습니다.

<p align="center"><img src="/assets/img/post_img/rds8.png"></p>

본인이 사용하는 툴을 사용해서 데이터 베이스에 연결하면 완료됩니다. ( 저는 sqleletron 사용했습니다. )

Name에 본인이 구별할 수 있는 데이터베이스이름, Server Address에 이전에 확인한 데이터베이스 엔드포인트 및 3306 포트 (일반적으로 RDS는 3306포트를 이용해서 통신합니다.)

User 및 Password에 본인이 RDS생성할 때 사용한 정보를 입력해주고 연결하면 됩니다.

initial Database/Keyspace에 해당하는 정보는 위에서 추가설정에서 설정한 데이터베이스 이름입니다. 참고해주세요.

<p align="center"><img src="/assets/img/post_img/rds10.png"></p>

잘 작동하나 테스트해보니 데이터베이스가 잘 작동함을 알 수 있습니다.

사용을 안하시면 데이터베이스를 삭제해주세요. ! (데이터 베이스를 이후에 사용할 일이 없다면, 스냅샷까지 삭제해주세요 소액이지만 과금이 될 수 있습니다.)

<br>
<br>
