---
layout: post
title:  "GCP - Secret-Manager 구현하기"
categories: [GCP, Secret-Manager]
tags: [GCP, Secret-Manager]
---



이전 포스팅에서 [GKE - 워크로드아이덴티티 구성하기](https://wjdqlsdlsp.github.io/gcp/2022-09-17-GKE-%EC%9B%8C%ED%81%AC%EB%A1%9C%EB%93%9C%EC%95%84%EC%9D%B4%EB%8D%B4%ED%8B%B0%ED%8B%B0/)를 통해 IAM 정보를 감추는 시도를 해왔습니다.

하지만 이는 IAM Service Account만 해당하며 다른 정보를 지울 수는 없었습니다.

이에 따라 다른 방법을 이용해서 추가적인 Secret 정보를 지우고자 합니다.



### Secret Manager

사용할 방법은 GCP에서 제공하는 [Secret Manager](https://cloud.google.com/secret-manager?hl=ko)를 사용하여 Secret 정보를 숨기려고합니다.



<p align="center"><img src="/assets/img/post_img/secret1.png"></p>

사용법은 매우 간단합니다. GCP - Secret Manager에 접속하여 보안 비밀 만들기를 진행하시면 됩니다.

<p align="center"><img src="/assets/img/post_img/secret2.png"></p>

이름에는 사용하실 이름을 입력하시고, 보안 비밀값에 숨기고 싶은 시크릿 정보를 입력하여 변수처럼 사용할 수 있습니다.

하지만 이를 Kubernetes에 바로 연결할 수는 없고, 다른 도구의 힘을 빌려야 합니다.



### External Secret Operator

Secret Manager에 등록된 시크릿 정보를 사용하기 위해 Kubernetes 환경에서는 [External Secret Operator](https://external-secrets.io/v0.6.0-rc1/)라는 오픈소스를 사용합니다.

<p align="center"><img src="/assets/img/post_img/secret3.png"></p>

external secret operator에서 설명하는 구조입니다. 즉, 쿠버네티스 안에서 pod형태로 띄어두고, 외부 public cloud로 부터 데이터를 받아오는 형식입니다.

<br>

설치법 또한 어렵지 않습니다.

```shell
helm repo add external-secrets https://charts.external-secrets.io
```

helm-chart형식으로 관리하고 있으며, 이를 위의 명령어를 통해서 repo에 추가합니다.



```shell
helm install external-secrets \
   external-secrets/external-secrets \
    -n external-secrets \
    --create-namespace
```

다음으로 위의 명령어를 통해서 설치하면 exteranl secret operator의 준비가 완료됩니다.

(참고) crd-install 옵션이 있던데, 저의 경우 따로 설정하지 않아도 알아서 설치하였습니다.

<br>

다음으로 각 퍼블릭 클라우드 환경에 맞춰 SecretStore를 설치해야 합니다.

이 과정에서 이전 포스터에서 사용한 [워크로드 아이덴티티 ](https://wjdqlsdlsp.github.io/gcp/2022-09-17-GKE-%EC%9B%8C%ED%81%AC%EB%A1%9C%EB%93%9C%EC%95%84%EC%9D%B4%EB%8D%B4%ED%8B%B0%ED%8B%B0/)방법을 사용합니다.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sa
  namespace: es
  annotations:
    iam.gke.io/gcp-service-account: example-team-a@my-project.iam.gserviceaccount.com
```

<br>

이후 ClusterSecretStore를 구성해야합니다.

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: example
  namespace: external-secrets
spec:
  provider:
    gcpsm:
      projectID: sharp-voyage-345407
      auth:
        workloadIdentity:
          # name of the cluster region
          clusterLocation: asia-northeast3-a
          # name of the GKE cluster
          clusterName: test
          # projectID of the cluster (if omitted defaults to spec.provider.gcpsm.projectID)
          clusterProjectID: sharp-voyage-345407
          # reference the sa from above
          serviceAccountRef:
            name: sa
            namespace: es
```

주석에서 설명하는 내용처럼 기입하면 되고, ServiceAccountRef에는 이전에 생성한 ServiceAccount 정보를 입력합니다.

```shell
$ kubectl get clustersecretstore
NAME      AGE    STATUS   READY
example   105m   Valid    True
```

정상적으로 설치되었나, 확인하려면 위의 명령어를 입력해서 현태 상태가 Ready상태인지 확인하세요.

Ready상태가 되면, external secret를 사용할 준비가 된겁니다.

<br>

### Secret정보 불러오기

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: example
spec:
  refreshInterval: 1h           # rate SecretManager pulls GCPSM
  secretStoreRef:
    kind: ClusterSecretStore
    name: example               # name of the SecretStore (or kind specified)
  target:
    name: secret-to-be-created  # name of the k8s Secret to be created
    creationPolicy: Owner
  data:
  - secretKey: test  # name of the GCPSM secret key
    remoteRef:
      key: test
```

위의 yaml 정의를 통해서 secret-manager에 등록된 secret 정보를 가져올 수 있습니다.

refreshInterval를 통해 갱신을 어느 주기로 할지 정할 수 있습니다. ( 갱신을 너무 자주하면 돈이 많이 나갑니다. )

secretStoreRef의 설정의 경우 위에서 설정한 clusterSecretStore 정보를 기입하면 됩니다.

target의 경우 생성하게 될 시크릿의 이름 및 정책입니다.

data의 경우 secret-manager에 등록한 secret 이름입니다. secretKey 및 key 둘다 이름을 입력해 주면 됩니다.

이렇게 실행하면 secret 정보가 생성됩니다.

<br>

### secret정보 mount 하기

사용법은 secret 정보를 사용하는 것과 동일합니다. (secret정보니까 당연한 말이겠죠...)

제가 사용한 코드는 아래와 같습니다.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: publish
spec:
  template:
    spec:
      containers:
      - name: publish
        image: gcr.io/sharp-voyage-345407/publish:0.3
        command: ['python', 'main.py']
        volumeMounts:
          - name: foo
            mountPath: "/credential"
            readOnly: true
        env:
          - name: GOOGLE_APPLICATION_CREDENTIALS
            value: "/credential/test"
      volumes:
        - name: foo
          secret:
            secretName: secret-to-be-created
      restartPolicy: Never
  backoffLimit: 1
```

이전 포스팅에서 사용한 pub/sub에 데이터를 보내는 이미지를 통해 job을 생성했습니다.

저는 시크릿 정보로 GOOGLE_APPLICATION_CREDENTIALS를 사용했기에, 시크릿 정보 마운트 후, 위와 같이 경로를 변수로 등록했습니다. (이렇게 하면, 해당 컨테이너가 credentials를 사용합니다.)

<br>
