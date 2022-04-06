---

layout: post

title:  "Spring Boot k8s 배포"

date:   2022-04-06

categories: 데이터엔지니어

tags: 데이터엔지니어

image: /post_img/k8s.png

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

# 환경변수 세팅
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
java -jar build/libs/[jar파일]

# java -jar build/libs/semogong-0.0.1-SNAPSHOT.jar
```

위의 코드를 통해서 실행하면 서버가 실행되는 것을 알 수 있습니다. 

이후 서버에 접속하기 위해서는 각자의 퍼블릭 IP주소:8080 를 입력하면 정상적으로 접속 할 수 있음을 알 수 있습니다.

<p align="center"><img src="/images/post_img/spring4.png"></p>

저 같은 경우, 팀원들과 현재 진행중인 프로젝트 메인사이트가 잘 출력되네요.

<br>

#### Docker install

https://myjamong.tistory.com/299 블로그 글을 참고했습니다.

```shell
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

apt가 HTTPS 프로토콜을 통해서 repository를 사용할 수 있도록 패키지를 설치

```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

Docker의 공식 GPG key를 추가

```shell
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

apt source list에 repository를 추가

```shell
sudo apt-get install docker-ce
sudo apt install docker.io

#check
sudo docker ps
```

docker 설치

<br>

#### Dockerfile 생성

```shell
touch Dockerfile
```

저는 touch 명령어를 통해 Dockerfile을 생성하였습니다.

```dockerfile
#Dockerfile
FROM openjdk:11-jre-slim
ADD target/semogong-0.0.1-SNAPSHOT.jar app.jar
ENV JAVA_OPTS=""
ENTRYPOINT ["java","-jar","/app.jar"]
```

이후 Dockerfile ADD에 추가한 것처럼, target폴더에 jar 파일을 copy했습니다.

```shell
mkdir
cp build/libs/semogong-0.0.1-SNAPSHOT.jar target/
```

<br>

#### Docker build

```shell
sudo docker build --tag semogong-demo:0.1 .
```

이제 위에서 생성한 Dockerfile을 빌드합니다. 맨 뒤에 . 을 꼭 입력해주세요.

```shell
sudo docker run -p 8080:8080 -e DB_URL=$DB_URL -e DB_NAME=$DB_NAME -e DB_PASSWORD=$DB_PASSWORD semogong-demo:0.1
```

위와 같이, 이미지를 통해 실행하면 위에서 실행했던 서버와 동일하게 실행됩니다.

<br>

#### Docker hub로 이미지 올리기

<p align="center"><img src="/images/post_img/spring5.png"></p>

저는 Docker hub에 레파지토리를 생성해놨습니다. 각자 원하는 이름으로 생성하면 됩니다.

<p align="center"><img src="/images/post_img/spring6.png"></p>

```shell
sudo docker tag 9f5351093984 wjdqlsdlsp/semogong
```

위의 코드를 통해서 이미지 태그를 설정합니다.

```shell
sudo docker login
```

이후 로그인을 합니다.

```shell
sudo docker push wjdqlsdlsp/semogong
```

이제 docker hub로 push 하면 완료됩니다.

<br>

#### Kubernetes 배포하기

저는 gcp kubernetes engine 환경에서 진행합니다.

```shell
kubectl run semogong --image=wjdqlsdlsp/semogong --port 8080 --dry-run -o yaml > semogong.yaml
```

먼저 docker hub로 push 한 이미지를 이용해서 pods를 운영할 예정인데, 이전에 환경변수등을 변경하기 위해 yaml 파일채로 다운로드합니다.

이후 vi를 통한 yaml에 환경변수값을 추가합니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: semogong
  name: semogong
spec:
  containers:
  - image: wjdqlsdlsp/semogong
    name: semogong
    env:
    - name: 환경변수_name
      value: 환경변수_value
    ports:
    - containerPort: 8080
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

저는 위와같이 구성했습니다. spec - containers - env에 원하는 환경변수를 추가하면됩니다. 

( yaml에서 $환경변수이름 으로 환경변수 넘겨주려고했는데, 노드가 달라서 그런지 안넘어가고 오류가 나더라구요. )

```shell
kubectl create -f semogong.yaml
```

이제 수정한 yaml파일을 실행하면됩니다.

```shell
kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
semogong   1/1     Running   1          27s
```

kubectl get pods를 통해 실행한 pods가 정상적으로 작동하는지 체크해주세요 ( STATUS 확인 )

```shell
kubectl expose pod semogong --name=semogong --type=LoadBalancer --port 80 --target-port 8080
```

이제 해당 pod를 서비스화 시키기위해 expose명령어로 노출시켜줍니다.

```shell
kubectl get service
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP      10.120.0.1     <none>        443/TCP        13m
semogong     LoadBalancer   10.120.7.169   34.64.193.53   80:31347/TCP   93s
```

kubectl get service 명령어로 EXTERNAL-IP를 접속하고 해당 주소로 접속하면 됩니다.

저의 경우 34.64.193.53:80 주소로 접속하면 접속이 되겠네요. ( 80번포트는 생략가능 )

<br>

