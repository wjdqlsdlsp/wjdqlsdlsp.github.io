---

layout: post

title:  "Github Action을 이용한 CI구축하기"

date:   2022-05-15

categories: 데이터엔지니어

tags: 데이터엔지니어

image: /post_img/git.jpg

---

이것저것 공부하다, 블로그 글을 너무 소홀히 한 것 같아 다시 열심히 정리하려고 다짐했습니다.

오늘 정리할 글은, Github Actions 관련 글입니다.

이전에 포스팅한, [GCP Cloud CLI를 통한 GKE 자동배포 환경만들기](http://localhost:4000/%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%97%94%EC%A7%80%EB%8B%88%EC%96%B4/2022/04/24/GCP-GKE%E1%84%8C%E1%85%A1%E1%84%83%E1%85%A9%E1%86%BC%E1%84%87%E1%85%A2%E1%84%91%E1%85%A9/) 를 통해서 자동 배포하는 환경을 구축 했었습니다.

이는 GCP Cloud CLI + Airflow or Crontab + bash를 이용해서 일정 시간마다, 업데이트 하는 환경입니다.

하지만 이는 여러 단점이 존재했습니다.

- 항상 켜져있는 인스턴스가 필요하다. ( Like AWS or GCP Cloud )
- Github에 업데이트함과 동시에 배포가 되는 구조가 아닌, batch 작업으로 구성되어 있다.

이 외에도 작은 단점들이 있어서 사용하지 않았습니다.

그래서 CI/CD 에 관심을 가지고 공부하다 보니 Jenkins를 사용하면 제가 원하는 작업을 할 수 있더군요.

그래서 유튜브에 Jenkins 강의 등을 통해서 공부하던 중, [생활코딩의 github action](http://localhost:4000/%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%97%94%EC%A7%80%EB%8B%88%EC%96%B4/2022/04/24/GCP-GKE%E1%84%8C%E1%85%A1%E1%84%83%E1%85%A9%E1%86%BC%E1%84%87%E1%85%A2%E1%84%91%E1%85%A9/)를 발견했습니다.

처음에는, git push 같은 명령어 인줄 알았는데 참 신기하더라구요.

지금부터 제가 CI를 구축한 과정을 정리한 내용입니다.

<br>

#### Github Actions란?

Github Actions는 간단하게 말하면, Github 에서 제공하는 CI/CD 툴입니다.

Jenkins와 비교해서 매우 간단하게 환경을 구축 할 수 있다는 장점이 있습니다.

<p align="center"><img src="/images/post_img/action1.png"></p>

CI/CD 환경을 구축하고 싶은 레파지토리를 하나 선택해서 Actions를 눌러보면, 다양한 workflow template이 존재합니다.

위의 worflow template을 이용하면 더욱 더 쉽게 환경을 구축할 수 있게 깃헙이 도와줍니다.

아마 화면에 보이는 템플릿이 저와 다를겁니다. 깃헙이 레파지토리를 분석해서 추천해주는 구조인 것 같습니다.

다양한 내용을 원하면 [Github Actions Docs](https://github.com/features/actions)를 참고하세요 !

yaml파일을 보면, uses 같은 내용이 있는데, 이는 독스를 보면서 천천히 설정하면 됩니다.

<br>

#### Github Actions yaml 파일 정의하기

원하는 템플릿 하나를 설정해서 이를 자신의 환경에 맞게 커스터마이징 하면 됩니다.

저는 자바 스프링부트를 이미지화 해서 도커허브로 전송하고, 

이를 쿠버네티스가 도커허브로부터 pull해서 업데이트 하는 구조로 환경을 구축하기를 원합니다. 

이번 블로그 글에서는, 도커허브로 전송하는 과정 까지 CI라인을 구축할 예정입니다.

<br>

#### 사용 yaml 코드

```yaml
name: gradle and docker

on: 
  push:
    branches:
      - master
  pull_request:
    branches:
      - master
jobs:
  build: 
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK 11
        uses: actions/setup-java@v1
        with:
          java-version: 11
        
      - uses: actions/checkout@v1
      - name: Login to DockerHub Registry
        run: echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
        
      - name: Build docker image and push to hub
        run: |
            VERSIONALL=$(cat argo/deployment.yaml | grep image)
            VERSION_SPLIT=($(echo $VERSIONALL | tr ":" "\n")) 
            VERSION=${VERSION_SPLIT[2]}
            rm -rf src/test
            gradle build
            cp build/libs/semogong-0.0.1-SNAPSHOT.jar target/
            docker build -t semogong-demo:0.1 .
            images_id=$(docker images -qa semogong-demo:0.1)
            docker tag $images_id wjdqlsdlsp/semogong:$VERSION
            docker push wjdqlsdlsp/semogong:$VERSION
```

상당히 복잡해 보이지만, shell 및 이미지 관련 내용만 알면 어렵지 않게 CI환경을 구축할 수 있습니다.

<br>

```yaml
name: gradle and docker

on: 
  push:
    branches:
      - master
  pull_request:
    branches:
      - master
```

먼저 맨 위부터 살펴보면, name정의를 합니다. 이는 github actions의 workflow이름입니다.

이후 on이라는 키워드가 등장하는데, 이는 어떤 상황에서 github actions를 작동하는가 입니다.

제가 설정한내용은, master branches에 push가 되거나, pull_request가 될 때, 작동하라는 내용입니다.

<br>

```yaml
jobs:
  build: 
    runs-on: ubuntu-latest
    steps:
```

이후 jobs를 정의합니다. 먼저 build를 통해서 어떤 환경에서 이를 작동하는지 정의합니다.

이는 도커 이미지 개념과 유사하더라구요, 제가 설정한 내용은 가장 최신의 ubuntu환경에서 빌드하라는 설정입니다.

이후 steps 키워드는 이후 나오는 작업을 차례대로 진행하라는 내용입니다.

<br>

```yaml
    - uses: actions/checkout@v2

     - name: Set up JDK 11
       uses: actions/setup-java@v1
       with:
       	java-version: 11
```

여기서 uses키워드가 존재하는데 자세한 내용은 깃헙 독스를 참고해주세요. 

actions/checkout@v2같은 경우, 깃헙 레파지토리에서 소스코드를 가져오라는 내용인 것 같습니다.

다음 Set up JDK 11는, 제가 사용할 환경을 설치하기 위해서 사용했습니다.

이처럼, 깃헙 독스를 통해 자신이 구현하고자 하는 환경을 맞춰 나가면 됩니다.

<br>

```yaml
      - uses: actions/checkout@v1
      - name: Login to DockerHub Registry
        run: echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
        
      - name: Build docker image and push to hub
        run: |
            VERSIONALL=$(cat argo/deployment.yaml | grep image)
            VERSION_SPLIT=($(echo $VERSIONALL | tr ":" "\n")) 
            VERSION=${VERSION_SPLIT[2]}
            rm -rf src/test
            gradle build
            cp build/libs/semogong-0.0.1-SNAPSHOT.jar target/
            docker build -t semogong-demo:0.1 .
            images_id=$(docker images -qa semogong-demo:0.1)
            docker tag $images_id wjdqlsdlsp/semogong:$VERSION
            docker push wjdqlsdlsp/semogong:$VERSION
```

이후, Docker hub에 접근하기 위해서 로그인하는 내용을 추가해줬습니다. 여기서 run 키워드에는 자신이 동작할 내용을 작성해 주면 됩니다.

여기서 secrets.DOCKER_PASSWORD 가 나오는데, 이는 제가 설정한 환경변수입니다. 깃헙에 공개할 내용이기에, 시크릿한 정보는 위와 같이 숨길 수 있습니다.

<p align="center"><img src="/images/post_img/action2.png"></p>

설정하는 방법은, 깃헙 레파지토리에서 Settings에서 Secrets - Action에서 정의하면됩니다.



마지막으로 Build docker image and push to hub는 제가 작동하고자 하는 내용을 정리해줬습니다.

여러줄을 기입하기 위해서 `|` 를 사용하고 계속해서 작성해주시면 됩니다.

위의 코드같은경우, 제가 사용하는 환경에 맞게 구축한 것이기에, 본인이 작동하고 싶은 환경을 구축하면 될 것 같습니다.

<br>

#### 테스트하기

처음 완료를 누르고 설정하면 빌드가 되는 것을 확인 할 수 있습니다.

이후 제가 설정한 on 조건에 따라서, CI라인이 작동하게 됩니다.

<p align="center"><img src="/images/post_img/action3.png"></p>

제가 구현한 환경을 팀원들이 잘 사용해주고 있네요. 위와같이 초록불이 뜨면 정상적으로 CI가 된 상태입니다.

해당 내용을 클릭하면 세부 정보를 파악할 수 있습니다. 오류가 발생했다면, 어디서 오류가 발생했는지 파악해보세요.

만약 오류가 발생했다면, 설정한 깃헙 이메일로 알림이 가더라구요. 

만약 CI를 수정하고 싶으면, 깃헙 레파지토리에서, .github/worflows를 확인 할 수 있을 겁니다. 이를 수정하면 됩니다.

GCP, AWS관련 내용이 보이는 것 보니, 자동 배포까지 구현할 수 있는 환경이 갖춰진 것 같지만, 

저는 Argo CD를 사용하기 원해서 CI라인만 구현했습니다. ( Argo CD 대쉬보드가 너무 보기 좋더라구요 )

이후 CD 관련 글을 정리해서 포스팅 하도록 하겠습니다.

<br>

#### 삽질한 내용...

Github Actions를 설정 하다가 변수로 많은 고생을 했었습니다.

이미지태그를 시간으로 설정해주고 싶었기에 shell 명령어의 date 와 잘 조합해서 적용하려 했습니다. (지금은 직접 CD라인에서 사용하는 yaml파일을 확인하고 전달해주는 식으로 구현)

이 과정에서 오류를 많이 만났는데, 그 중 하나가 변수관련 내용입니다.

우분투환경에 익숙했기에, .bashrc를 수정도 해보고, export도 해보고, 여러가지 시도했었는데 오류가 나면서 실패했습니다.

이유는, step에 있더라구요, 한 스텝이 지나면 지난 내용은 사라지는 구조인 것 같습니다. 만약 변수를 사용할 일이 있으면, 

저처럼 같은 스텝에 정의해주시면 기존 우분투환경처럼 사용할 수 있습니다. ( 제가 마지막 step에 모든 shell 코드를 나열한 이유랍니다. )

어디서 오류가 났는지 확인하기 위해서는 좋지 않은 방법 같습니다.

<br>



