---
layout: post
title: "Terraform: IaC를 도구를 이용한 GKE 생성하기"
categories: [Terraform]
tags: [Terraform]
---



Terraform은 강력한 Infrastructure as Code(IaC) 도구로, 클라우드 서비스의 생성, 관리, 업데이트를 자동화하는 코드를 작성할 수 있게 해줍니다. 

이는 우리가 Public Cloud에서 사용하는 인프라를 코드 기반으로 효율적으로 생성, 관리할 수 있게 돕습니다.

<br>

"이걸 왜 사용하는가?" 의문이 들 수 있습니다. 저도 가끔 `Low Code`, `No Code`를 외치는 시대에서 IaC는 모순이지 않은가 생각하지만, 결국에는 인프라 관리의 편의성을 위해 Terraform을 사용하고 있습니다.

<br>

Terraform을 사용하는 이유가 많겠지만, 무엇보다 제가 사용하는 가장 큰 이유는 리소스를 생성하거나 삭제하는 것이 매우 간단하다는 점 입니다.

보통 개인 Public Cloud계정을 이용해서 테스트 및 공부를 하고는 하는데, 사용하고 나서 나의 작고 소중한 돈을 아끼기 위해서 생성한 리소스를 삭제합니다.

이 과정에서 놓치는 리소스도 있고, 항상 사용 중인 리소스가 있는지 Billing을 확인합니다. 🥲

<br>

이럴 때, Terraform은 엄청나게 편리합니다.

Terraform으로 생성된 리소스는 `terraform destroy` 명령어 한방으로 전부 삭제 할 수 있으니 편하지 않을 수 없습니다.

<br>

서론이 길었네요. 이제부터 나오는 내용은, 제가 대표적으로 Terraform을 사용해서 생성, 삭제를 반복하는 GKE입니다.

이를 보고, 한분이라도 GKE를 쉽게 생성, 삭제하였으면 좋겠다는 생각이 들어 공유합니다.

<br>

### gcloud auth

Terraform을 이용해서 GCP 리소스를 생성, 삭제 하기 위해서는 GCP에서 제공하는 인증방식을 사용해야 합니다.

이 중 저는 gcloud를 이용해서 사용합니다.

<br>

gcloud는, GCP의 리소스를 관리하는데 사용되는 강력한 도구로, 터미널에서 GCP의 다양한 기능을 사용할 수 있습니다.

설치법은 [GCP docs](https://cloud.google.com/sdk/docs/install?hl=ko)에 친절하게 나와있으니 참고하세요~

<br>

그 중에서도 [gcloud auth](https://cloud.google.com/sdk/gcloud/reference/auth)를 사용할 건데, 사용법은 간단합니다.

```shell 
gcloud auth login --brief
```

위의 명령어를 Terminal에 입력하면, google 로그인을 통해서 간단하게 인증이 완료됩니다.



```shell
gcloud auth list
```

정상적으로 등록되었는지 확인을 위해서는 list 명령어를 사용하면 됩니다.



```shell
gcloud auth revoke test@gmail.com
```

만약 사용하고 있는 계정이 많거나, 다른 이유로 인증에서 제거하고 싶으면 revoke를 사용하세요. 

(저 같은 경우, 회사계정과 분리하고 싶어서 가끔 사용합니다. 그럴리는 없겠지만, 회사계정을 이용해서 회사에서 사용하는 리소스를 제거할 수도 있으니...)

<br>

```shell
gcloud auth application-default login
```

마지막으로 위의 명령어를 입력하면, Terraform같은 application 등에서 google auth에 등록된 내용을 default로 사용하도록 설정합니다.



위의 내용대로 gcloud auth를 등록하면 Terraform을 이용해서 GCP에 접근할 준비가 완료 됩니다.

<br>

### Terraform 설치하기

Terraform 설치하는 방법은 간단합니다.

저같은 경우, Mac 유저로 brew를 통해서 간단하게 설치합니다.

```shell
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```



Mac 외에 환경에서 Terraform을 설치하는 방법은 [Terraform Docs: Install Terraform](https://developer.hashicorp.com/terraform/downloads)을 참고해주세요.

<br>

```shell
$ terraform --version
Terraform v1.4.6
on darwin_arm64
```

정상적으로 설치되었는지 확인하기 위해서 위와 같이 테스트 해보세요.

<br>



### GKE 생성하기

`GKE` or `Terraform`을 조금 만져보신 분들은, 설정할 내용이 많다는 것을 느낄 것 입니다.

GKE를 생성하기 위해 VPC, Subnet 등을 추가로 생성하고 이를 사용합니다. 

이에 대한 과정을 [BESPIN GLOBAL: Terraform 을 사용하여 GKE 생성하기](https://www.bespinglobal.com/terraform-gke/) 에서 잘 설명하고 있습니다.

<br>

저는 조금 다르게 module 방법을 이용하려고 합니다.

module을 직접 생성하는 것은 힘들지만, 이미 생성되어 있는 module을 가져다 쓰는건 간단합니다.

다행히도, 이러한 모듈을 [Google Cloud and HashiCorp](https://github.com/terraform-google-modules)에서 제공해주고 있습니다.

> 괜히 힘들게 살지 말고, 거인에 어깨에 올라타자...

<br>

[Github Repo](https://github.com/terraform-google-modules/terraform-google-kubernetes-engine)를 Clone 받아서 필요한 module을 가져오기만 하면 됩니다.

저는 modules 중 [private-cluster](https://github.com/terraform-google-modules/terraform-google-kubernetes-engine/tree/master/modules/private-cluster)를 사용했습니다. 본인의 필요에 따라 modules를 선택하셔도 괜찮습니다.

해당 파일을 개인의 선호에 따라 새로운 폴더에 구성하였습니다.



```shell
$ tree
.
├── gke
│   ├── gke.tf
│   └── variables.tf
└── module
    └── google-kubernetes-engine
        ├── README.md
        ├── cluster.tf
        ├── dns.tf
        ├── firewall.tf
        ├── main.tf
        ├── masq.tf
        ├── networks.tf
        ├── outputs.tf
        ├── sa.tf
        ├── scripts
        │   └── delete-default-resource.sh
        ├── variables.tf
        ├── variables_defaults.tf
        └── versions.tf
```

`gke` 라는 폴더와 `module` 폴더로 분리하였으며, `module` 폴더안에, `google-kubernetes-engine`이라는 이름으로 이전에  다운 받은 private-cluster를 넣었습니다.

<br>



이후, module은 건들지 않고, gke 폴더에서 작업합니다.

기본적으로 `gke.tf`와, `variables.tf`를 생성하여 사용합니다.

```shell
# gke.tf
module "cluster" {
  source = "../module/google-kubernetes-engine"

  ip_range_pods     = ""
  ip_range_services = ""
  name              = "<생성할 cluster-name을 입력하세요>"
  network           = ""
  project_id        = "<project-id를 입력하세요>"
  subnetwork        = ""
  region            = "asia-northeast3"
  zones             = ["asia-northeast3-a"]
  regional          = false
  node_pools = [
    {
      name         = "pool-1"
      machine_type = "e2-standard-2"
      autoscaling  = false
      node_count   = 2
      disk_size_gb = 10
      disk_type    = "pd-balanced"
      auto_upgrade = true
    },
  ]
}
```

<br>

```shell
# variables.tf
provider "google" {
  project = "<project-id를 입력하세요>"
  region  = "asia-northeast3"
}

terraform {
  backend "gcs" {
    bucket = "terraform정보를 담을 bucket이름"
    prefix = "terraform-gke"
  }
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = ">= 4.51.0, < 4.65.0"
    }
  }
}
```



위의 내용은 제가 기본적으로 사용하는 내용이지, Terraform에서 GKE를 생성하기 위한 필수 조건이 아닙니다.

사용하고 싶은 내용으로 수정하셔서 사용하셔도 됩니다.

특히, `variables.tf`에 backend부분은 gcs가 아닌 local로 사용하셔도 좋습니다.

<br>



### Terraform을 이용해서 리소스 생성하기

```shell
terraforms/gke
```

먼저 모든 명령어는 이전에 생성한 gke 폴더에서 진행됩니다.



이제 생성만 하면 완료됩니다.

```shell
terraform init
```

위의 명령어를 통해서, terraform을 초기화 해줍니다. `init` 명령어의 경우 멱등성을 보장하여 여러먼 입력해도 문제가 생기지 않습니다.

<br>

```shell
terraform plan
```

다음은 `plan` 명령어입니다. Terraform을 사용하게 되면, 가장 많이 사용하는 명령어인 것 같습니다.

정의한 tf 파일들로 생성, 변경, 삭제되는 리소스들을 표시합니다.

<br>

```shell
terraform apply
```

마지막으로 실제 GCP에 리소스를 생성하는 과정입니다.

생성, 변경, 삭제되는 리소스를 확인하고 `yes`를 입력하면 작업이 진행됩니다.

<br>

명령어를 입력하면 실제 GCP에서 리소스를 생성 중임을 확인 할 수 있습니다.

```shell
Apply complete! Resources: 9 added, 0 changed, 0 destroyed.
```

위와 같이 terminal에서 terraform 메시지가 보이면 작업이 완료된 상태입니다.

이제 원하는 대로 수정해서 Terraform을 사용하시면 됩니다.

<br>



```shell
terraform destroy
```

마지막으로 작고 소중한 돈을 아끼고자... 리소스를 사용하지 않는다면 삭제해주세요.
