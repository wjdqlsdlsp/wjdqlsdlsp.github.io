---
layout: post
title: PVC 용량 증축하기
categories: [kubernetes]
tags: [kubernetes]
---

pvc를 pod에 bounding해서 사용하다 보면, pvc의 용량을 다 사용해서, bounding되지 못하는 현상이 나타나고는 합니다.

이를 해결하기 위한 방법은 간단합니다. 바로, **"pvc 용량을 증가시킨다"** 입니다.



저는 처음에 해당 문제를 직면했을 때, 당황했습니다.

pvc의 용량을 증가시키면, 기존에 있던 pvc를 삭제하고 새로운 큰 용량의 pvc를 생성하는 것은 아닐까? 걱정하여 pvc를 스냅샷을 뜨고 진행했었습니다.



하지만 쓸데 없는 행동이였습니다. ( 물론 백업은 중요합니다 ^^ )



<p align="center"><img src="/assets/img/post_img/pvc1.png"></p>

위의 사진은 [kubernetes docs](https://kubernetes.io/ko/docs/concepts/storage/storage-classes/) 에서 캡쳐해온 사진입니다.

kubernetes에서는 볼륨 확장에 대해서 지원하고 있습니다. 이에 대한 조건은 위와 같습니다.

위의 버전을 확인해 보면, 웬만한 Public cloud에서는 1.11 이상의 클러스터 버전을 사용하고 있으면 pvc를 증축할 수 있음을 알 수 있습니다.

현재 k8s의 버전이 1.25까지 나온 점을 고려하면, 대부분의 케이스에서 pvc 확장이 가능 할 것이라고 판단됩니다.

### Step 1. Storage Class 설정하기

pvc 볼륨확장을 이용하기 위해서는, pvc가 storage class에 연결된 구조이여야 합니다.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: true
mountOptions:
  - debug
volumeBindingMode: Immediate
```

storage class를 보면, `allowVolumeExpansion` 설정이 있습니다. 해당 설정을 `true`로 설정하면, pvc의 확장 기능을 사용 가능합니다.

만약 이미 선언한 storage class에 해당 옵션이 없다하더라도, edit을 통해 추가해주시면 됩니다.



### Step 2. PVC edit

다음으로는 pvc를 edit 합니다.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 200Gi
  storageClassName: standard
status:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 200Gi
  phase: Bound
```

`status.capacity.storage` 를 원하는 용량으로 증가시키면 pvc 증축이 완료됩니다.



> 참고: 볼륨 확장 기능을 사용해서 볼륨을 확장할 수 있지만, 볼륨을 축소할 수는 없다.

하지만 위의 사진에서도 나와있지만, scale up은 가능하지만, scale down은 불가합니다.

Cloud를 사용하신다면, 돈과 관련되는 이야기니 필요한 만큼 증축하면 될 것 같습니다.

<br>
