---
layout: post
title:  "gcr 이미지 푸쉬"
categories: [gcr, Docker]
tags: [gcr, Docker]
---


본 포스트는 제가 보려고 만든 글로, 내용이 다소 빈약합니다.

<br>


```shell
docker build -t <image_name>:<image_version> .
```

dockerfile이 동일한 경로에 있다고 가정했을 때, dockerfile을 입력하지 않고 `.` 입력해서 사용할 수 있습니다.


<br>


```shell
docker build --platform linux/amd64 -t <image_name>:<image_version> .
```

만약 Macbook M1 apple slicon칩을 사용한다면?

위와 같이, `--platform`을 설정해줄 수 있습니다. 설정하지 않는다면, 다른 linux 체제에서 이미지를 불러올 때 환경 에러가 날 수 있습니다.

<br>

```shell
docker images
```

이후 docker build가 성공되었다면, `docker iamges` 명령어를 통해 조회할 수 있습니다.

<br>

```shell
docker tag <image_id> gcr.io/<project_id>/<repo_name>
```

`docker iamges`를 통해 알아낸 image_id를 tag 명령어를 통해서 태깅해 줍니다.

<br>

```shell
docker push gcr.io/<project_id>/<repo_name>:<image_version>
```

마지막으로 태깅한 이미지를 push 합니다.
<br>
