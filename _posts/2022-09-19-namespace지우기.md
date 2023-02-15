---
layout: post
title:  안지워지는 namespace 지우기
categories: [kubernetes]
tags: [kubernetes]
---

쿠버네티스를 사용하다보면, Namespace를 삭제하고 싶은 경우가 생깁니다.

물론 Namespace안에 어떤 리소스도 생성되어 있지 않으면, 문제가 생기지 않지만, 대부분의 경우가 오류가 생깁니다.

아무리 기다려도 Terminating 상태에서 머무는 모습을 본 사람이 많을 것입니다.

원인은 finalizers입니다.

Namespace가 갑자기 사라지는 것을 막기위해 kubernetes에는 finalizers설정이 되어 있습니다.

즉, 안에 있는 모든 리소스들이 사라져야 Namespace를 삭제 할 수 있습니다.

하지만, 리소스들이 맘같지 않습니다. ( 분명 다 삭제햇는데, 왜 Namespace가 삭제안되는거야… )

이런 경우 강제로 Namespace를 삭제하는 방법입니다.

```bash
NAMESPACE=ingress-nginx-controller
kubectl get namespace $NAMESPACE -o json > $NAMESPACE.json
```

NAMESPACE라는 변수에, 지우고 싶은 namepace의 이름을 넣어줍니다.

이후, 해당 네임스페이스에 대한 정보를 json 형태로 저장합니다.

```bash
{
    "apiVersion": "v1",
    "kind": "Namespace",
    "metadata": {
        "creationTimestamp": "2022-01-27T06:37:31Z",
        "deletionTimestamp": "2022-08-08T03:28:10Z",
        "labels": {
            "kubernetes.io/metadata.name": "ingress-nginx-controller"
        },
        "name": "ingress-nginx-controller",
        "resourceVersion": "500281231",
        "uid": "c81a2fse2df-vsd12-vsd23-dcs12-vddsvsdf1"
    },
    "spec": {
        "finalizers": []
    },
    "status": {
        "conditions": [
```

다음으로, 저장한 json파일을 수정해야 합니다.

spec.finalizers의 부분을 위와 같이 설정해줍니다. "finalizers": [] 이렇게!

```bash
kubectl proxy
```

이제 **새로운 터미널창**을 하나 켜서 위의 명령어를 입력해줍니다.

```bash
curl -k -H "Content-Type: application/json" -X PUT --data-binary @$NAMESPACE.json http://127.0.0.1:8001/api/v1/namespaces/$NAMESPACE/finalize
```

다시 원래 작업하던 터미널로 돌아와서 ( 사실 어떤 터미널창에 하든 상관없는데, NAMESPACE 변수를 사용하기 위해 위와같은 번거러운 작업과정을 가졌습니다.. )

위의 명령어를 입력하면 수정한 json 파일에 정의한 내용으로 namepace를 수정합니다.

즉, finalizers가 사라졌으므로 예정되어 있던 delete 작업이 수행될 수 있습니다.
<br>
