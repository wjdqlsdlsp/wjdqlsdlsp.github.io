---

layout: post

title:  "Docker 기본기"

date:   2022-04-22

categories: 데이터엔지니어

tags: 데이터엔지니어

image: /post_img/docker.png

---

# Docker 기본기 익히기

<br>

#### Docker 간단한 컨테이너 만들기

```shell
☁  ~  docker run ubuntu:20.04

Unable to find image 'ubuntu:20.04' locally
20.04: Pulling from library/ubuntu
185e8a4c1005: Already exists
Digest: sha256:9101220a875cee98b016668342c489ff0674f247f6ca20dfc91b91c0f28581ae
Status: Downloaded newer image for ubuntu:20.04
```

run 명령어를 통해서 간단하게 ubuntu 환경을 만들 수 있습니다.

local 환경에 이미지가 없음을 파악하고, 해당 이미지를 pull해서 실행시킵니다.

<br>

```shell
☁  ~  docker ps

CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

하지만 docker ps 를 통해 실행중인 컨테이너를 확인하면 아무것도 없는 것을 알 수 있습니다.

docker 컨테이너가 실행되고 정상적으로 종료되었기 때문에 아무것도 없는 상황입니다.

<br>

```shell
☁  ~  docker run --rm -it ubuntu:20.04 /bin/sh

# ls
bin  boot  dev	etc  home  lib	media  mnt  opt  proc  root  run  sbin	srv  sys  tmp  usr  var
```

위의 코드로 실행하면, ubuntu shell로 접속한 상황임을 알 수 있습니다.

접속된 상황에서 ls 명령어등 우분투 명령어를 실행하면 다음과 같이 ubuntu환경에서 명령어가 사용됩니다.

위에서 사용한 옵션을 살펴보면

`--rm` : 먼저 rm 옵션은 컨테이너가 종료됨과 동시에 컨테이너를 삭제하겠다는 의미입니다.

`-it` : -i 와 -t 옵션을 한번에 입력한 것으로, 터미널 입력을 위한 옵션입니다.

<br>

```shell
docker run --rm -p 1234:6379 redis
```

다음은 port설정방법입니다. 이를 테스트 하기 위해서 redis 컨테이너를 사용해보도록 하겠습니다.

```shell
telnet localhost 1234

set hello world
+OK
get hello
$5
world
quit
```

위의 코드를 실행하고 telnet 명령어로 접속을 하면 정상적으로 연결되는것을 확인 할 수 있습니다.

<br>

```shell
docker stop [container id or container name]
```

stop 명령어를 통해 컨테이너를 중지할 수 있습니다.

<br>

```shell
docker rm [container id or container name]
```

rm명령어를 통해 삭제할 수 있습니다

<br>

```shell
docker pull [이미지]
```

이미지를 pull 하는 방법은 위와 같습니다.

<br>

```shell
docker rmi [image id or image name]
```

이미지 삭제하는방법은 rmi 명령어를 사용합니다.

<br>

```shell
docker network create [network name]
```

위의 명령어를 통해 가상의 네트워크를 생성 할 수 있습니다.

이후 --network 옵션을 통해서 동일 네트워크로 묶을 수 있습니다.

<br>

```shell
docker stop mysql
docker rm mysql
docker run -d -p 3306:3306 \
  -e MYSQL_ALLOW_EMPTY_PASSWORD=true \
  --network=app-network \
  --name mysql \
  -v /my/own/datadir:/var/lib/mysql \
  mysql:5.7
```

 위와 같이 -v 옵션을 통해서, 볼륨을 지정할 수 있습니다.

<br>

#### docker-compose

이전에 사용했던 코드들을 yaml파일을 통해서 간단하게 docker 운영 가능

```yaml
version: '2'
services:
  db:
    image: mariadb:10.5
    volumes:
      - ./mysql:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: wordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
  wordpress:
    image: wordpress:latest
    volumes:
      - ./wp:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
```

[subicura 블로그](https://subicura.com/)에서 가져온 yaml 코드 예시입니다. ( subicura님 블로그 가면 잘 설명되어 있습니다. )

