---

layout: post

title:  "AWS - Spring Boot 서버 배포하기"

date:   2022-04-04

categories: 데이터엔지니어

tags: 데이터엔지니어

image: /post_img/aws.jpg

---

#### EC2 구축하기

먼저 운영할 서버의 필요에 따라 사양을 정하고 EC2 인스턴스를 선택합니다.

저는 여기서 Ubuntu Sever 20.04 LTS를 선택하였고, 프리티어로 진행하기 위해 t2.micro로 진행하였습니다.

<p align="center"><img src="/images/post_img/spring1.png"></p>

이후 보안 설정을 제외하고 모든 설정은 기본값으로 설정하였습니다.

<p align="center"><img src="/images/post_img/spring2.png"></p>

보안 설정같은 경우, 이후 서버가 정상적으로 작동하는지 확인하기 의해 모든 트래픽에 개방하였습니다. ( 절대 이러지 마세요. 필요한 포트만 개방하는 것을 추천드립니다. )

이후 키페어를 선택하고 EC2 환경을 구축하였습니다.

<br>

#### EC2 Console

이후 EC2 인스턴스에 접속한 뒤, 환경을 세팅하였습니다.  

<br>

#### Java install

Spring Boots에 맡게 자바를 다운로드 해줍니다.

```shell
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install openjdk-11-jdk
```

저는 사용하는 jdk에 맞게 jdk-11을 install 했습니다.

<br>

#### 환경변수 등록

```shell
sudo vi ~/.bashrc
```

이후 영구적인 환경변수를 등록하기 위해 bashrc 파일을 수정하였습니다. 



```shell
export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))
export PATH=$PATH:$JAVA_HOME/bin
export DB_URL=DB엔드포인트
export DB_NAME=DB_USER
export DB_PASSWORD=DB_PASSWORD
```

위와 같이 환경변수로 5개를 추가해 줬습니다. 위의 2개의 코드는 java를 환경변수에 등록하는 코드입니다.

아래 3개의 코드는, 사용중인 DB에 정상적으로 연결하기 위해 엔드포인트, 유저네임, 패스워드를 환경변수로 저장했습니다.

```shell
source ~/.bashrc
```

환경변수 등록을 완료하였으면, 위의 코드로 bashrc를 실행해줍니다.

<br>

#### Install Gradle

Spring Boots를 빌드하는 방법은 크게 2가지가 있는데 저는 그 중 더 많이 사용하는 Gradle 방식을 사용합니다.

```shell
VERSION=7.1.1
wget https://services.gradle.org/distributions/gradle-${VERSION}-bin.zip -P /tmp
```

위의 코드를 통해 gradle을 먼저 다운로드 합니다.

spring boots에서 7.1.1버전을 따로 설정하지 않으면 오류가 나더라구요. 그래서 저는 버전을 설정했습니다.

```shell
sudo apt-get install unzip
sudo unzip -d /opt/gradle /tmp/gradle-${VERSION}-bin.zip
sudo ln -s /opt/gradle/gradle-${VERSION} /opt/gradle/latest
```

이후 unzip을 사용하기 위해 다운로드하고, 다운받은 gradle을 계속해서 설치하였습니다.

```shell
sudo vi /etc/profile.d/gradle.sh
```

위에서 환경변수를 한 것처럼, gradle에 변수를 추가하기 위해 vi 로 수정하였습니다.

```shell
export GRADLE_HOME=/opt/gradle/latest
export PATH=${GRADLE_HOME}/bin:${PATH}
```

추가한 코드는 위와같습니다.

```shell
sudo chmod +x /etc/profile.d/gradle.sh
source /etc/profile.d/gradle.sh
```

이후 chmod를 통해서 모드를 변경하고, 실행시켰습니다.

<br>

#### Build

이제 모든 준비를 완료하고 gradle을 통한 build를 할 시간입니다.

준비된 spring boots 폴더에 접속하고 이를 빌드해줍니다.

```shell
gradle build
```

처음 빌드를 할 때는 다소 시간이 소요됩니다. 

이후 빌드가 완료되면, build/libs/ 폴더에 jar 파일이 생성됩니다.

<p align="center"><img src="/images/post_img/spring3.png"></p>

위와 같이 2개의 jar이 보일텐데, 이중 plain이 아닌 SNAPSHOT.jar를 실행시킬 예정입니다.

( 당연히 각자 본인의 설정에 따라 파일이름이 다릅니다. )

```shell
java -jar build/libs/semogong-0.0.1-SNAPSHOT.jar
```

위의 코드를 통해서 실행하면 서버가 실행되는 것을 알 수 있습니다. 

이후 서버에 접속하기 위해서는 각자의 퍼블릭 IP주소:8080 를 입력하면 정상적으로 접속 할 수 있음을 알 수 있습니다.

<p align="center"><img src="/images/post_img/spring4.png"></p>

저 같은 경우, 팀원들과 현재 진행중인 프로젝트 메인사이트가 잘 출력되네요.

이후 저는 이를 이미지화 시키고, docker hub에 저장한 다음, 쿠버네티스로 배포하는 것에 도전할 예정입니다. !