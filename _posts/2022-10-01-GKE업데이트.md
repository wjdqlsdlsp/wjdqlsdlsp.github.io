---
layout: post
title:  "GKE - 자동 업데이트"
categories: [GCP, GKE]
tags: [GCP, GKE]
---

현재 사용하고 있는 GKE 환경에서는 다양한 작업이 이뤄지고 있습니다.

그 중에는 Downtime이 있어도 되는 작업도 있지만, 몇몇 작업은 Downtime이 존재하면 안되는 작업이 있습니다.

하지만 GKE에서는 업데이트로 인한 Downtime이 존재하는데, 이를 대처하기 위해 업데이트에 대해 자세히 알아보고자 정리합니다.





### GKE 업데이트

GKE는 Google Kubernetes Engine의 줄임말로, GCP에서 쿠버네티스를 제공하는 관리형 서비스입니다.

쿠버네티스는 주기적으로 버전이 업그레이드 되고있으며, 계속해서 보안 및 새로운 서비스를 제공하려고 노력합니다. (참고 : [Kubernetes - Release Note](https://kubernetes.io/ko/releases/) )

GCP에서 또한, 이러한 쿠버네티스 버전에 맞춰 사용자에게 최신의 버전을 제공 하기 위해 노력하고 있습니다.

이 과정에서 Managed Service답게 버전을 업데이트 해주는 기능을 제공하고 있습니다.



### GKE release channel

현재 GKE는 버전을 총 4개의 다른 channel로 구분하고 있습니다. ( 참고 : [GKE release Note](https://cloud.google.com/kubernetes-engine/docs/release-notes#current_versions) )



<p align="center"><img src="/assets/img/post_img/update1.png"></p>



<br>

#### Static (no channel)

<p align="center"><img src="/assets/img/post_img/update2.png"></p>

Static 버전은 말 그대로 고정된 버전입니다. 이는 고정된 버전을 사용하지만, GCP측에서 보안 및 호환성의 유지가 필요한 경우에는 개입해서 자동 업데이트를 진행합니다.



<br>

#### Rapid

<p align="center"><img src="/assets/img/post_img/update3.png"></p>

Rapid 채널은 말 그대로 신속합니다.

쿠버네티스에서 새로운 버전이 출시되면 1~2일 안에 클러스터를 업데이트합니다. 이는 체험판 방식으로 얼리어댑터들에게 유용한 옵션이라고 소개하고 있습니다.

<br>

#### Regular

<p align="center"><img src="/assets/img/post_img/update4.png"></p>

일반채널입니다. 이는 GCP의 권장 옵션입니다. 새로운 버전이 출시되면 충분한 검증 기간을 가진 ( 1~2주 뒤에 업데이트를 진행합니다. )

<br>

#### Stable

<p align="center"><img src="/assets/img/post_img/update5.png"></p>

Stable은 공개 버전 채널입니다. 이는 GCP에서 공개적으로 일반채널에서 충분히 검증을 맞췄다고 판단되는 버전을 사용합니다.



### 자동 업데이트

Channel에 대해서 알아봤으니 이제, 자동 업데이트에 대해 알아 볼 시간입니다.

위에서 언급한 Managed Service 답게, 자동 업데이트를 제공하고 있습니다. 이는 GCP가 자랑하는 장점 중 하나입니다. ( 여러분은 인프라 신경쓰지말고 사용하세요 )

자동 업데이트는 참 편리한 기능이지만, 현재 GKE에서는 downtime이 발생하면 안되는 작업들이 존재합니다.

이러한 작업이 업데이트로 인해 down되게 되면 피해를 가지게 됩니다. 이를 대처하기 위해 자동 업데이트가 언제 진행되는지 알고자 정리합니다.



### 자동 업데이트 활성화 / 비활성화

기본적으로 자동 업데이트는 Control Plane과 Node pool 두개로 나눠지며 default 값은 활성화 입니다.

단, Control Plane은 자동 업데이트를 비활성화 할 수 없습니다. 그나마 이를 제어할 수 있는 방법이 위에서 소개한 Static Channel 사용입니다.



Node Pool의 경우 간단한게 UI 및 CLI를 통해 비활성화 할 수 있습니다.

<p align="center"><img src="/assets/img/post_img/update6.png"></p>

### 강제로 업데이트 되는 케이스

GKE에서는 보안 및 호환성의 유지가 필요한 경우 Static Channel 이여도 강제로 업데이트가 진행됩니다.

이러한 내용에 대해서는 [Release Note](https://cloud.google.com/kubernetes-engine/docs/release-notes#September_23_2022) 에서 소개해주고 있습니다.

<p align="center"><img src="/assets/img/post_img/update7.png"></p>

위의 예시에서 설명하듯이, no longer available에 해당하는 버전에 대해서는 강제로 업데이트가 진행됩니다.

여기서 주목해야할 부분은 “Control Plane과 Node Pool이 각각 어떻게 되는가?” 입니다.



<br>

#### Control Plane

Control Plane의 자동 업데이트는 위에서 언급한 것처럼 항상 활성화가 되어있습니다.

즉, 위의 사진에서 no longer available에 해당하지 않더라도, 아래 부분에 의해 업데이트가 진행됩니다.

<p align="center"><img src="/assets/img/post_img/update8.png"></p>

<br>

#### Node pool

노드 풀의 경우 Control Plane과 다르게 자동 업그레이드를 활성화/비활성화 할 수 있습니다.

활성화를 했다면, 위에 소개된 내용처럼 Control Plane의 버전에 맞춰 자동 업데이트가 진행 될 것이고, 비활성화를 했으면 진행되지 않을 것입니다.



단, no longer available에 해당하는 경우 이야기가 달라집니다.

<p align="center"><img src="/assets/img/post_img/update9.png"></p>

이 경우, 자동 업그레이드 사용 여부와 상관없이 업데이트를 진행합니다.

<p align="center"><img src="/assets/img/post_img/update10.png"></p>

또한, 노드는 Control Plane보다 버전 3개 이상 뒤쳐지면 안된다는 조건이 존재합니다. 뒤쳐지는 경우 업데이트를 진행하게 됩니다.



### 유지보수기간 제외

Control Plane의 업데이트는 유지보수 기간 제외를 통해서 막을 수 있습니다.

<p align="center"><img src="/assets/img/post_img/update11.png"></p>

하지만 이는 영원히 막는 것이 아닌, 단지 업데이트 하는 시간을 설정 혹은 업데이트하면 안되는 시간을 제한하는 방법입니다.

위에 사진의 경고에서 알 수 있듯이, 32일 동안 최소 48시간의 유지보수를 허용해야 합니다.
<br>
