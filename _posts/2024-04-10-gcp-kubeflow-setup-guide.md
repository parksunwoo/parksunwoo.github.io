---
title: Google Cloud Platform에서 Kubeflow 구축하기- 상세 가이드
categories:
  - Dev
tags:
  - gke
  - kubeflow
  - guide
last_modified_at: 2024-04-10T22:10:00-00:00
---

[Kubeflow on Google Cloud](https://googlecloudplatform.github.io/kubeflow-gke-docs/dev/docs/)

아래의 내용은 해당 문서를 바탕으로

실제로 GCP 상에 GKE를 셋업하고 Kubeflow를 구현하는 것까지 진행해본다.

docs에는 나와있지 않은 조금더 상세한 설명을 목표로 한다.

---

# **GCP 프로젝트 설정**

가장 처음 할일은 프로젝트를 구성하는 일. 관리 클러스터와 Kubeflow 클러스터를 서로 다른 google cloud 프로젝트에 구성해도 되고 같은 프로젝트 에 구성도 가능

프로젝트에 청구가 활성화되어있어야 한다. (GKE를 구성하고 Kubeflow를 구성하는 것은 과금이 시작된다)

![gcp-project-1](/assets/images/gcp-project-1.png)

터미널에서 `gcloud auth login` 를 입력하면 아래 액세스 창이 나온다.

허용을 누르고 터미널로 다시 돌아오면

![gcp-project-2](/assets/images/gcp-project-2.png)

로그인 된 계정과 현재 프로젝트를 확인할 수 있다.

간단히 `gcloud components update`  명령어로 현재 google cloud 의 컴포넌트를 업데이트 할수 있다.

![gcp-project-3](/assets/images/gcp-project-3.png)

`gcloud services enable`  명령어로 이후과정에 필요한 API 를 활성화시킨다.

![gcp-project-4](/assets/images/gcp-project-4.png)

활성화 완료되면 아래와 같은 메시지를 확인할 수 있다

Operation "operations/acat.p2-558684478102-2a5c1f42-f578-45d2-a777-c6375e4eb479" finished successfully.

# **OAuth 클라이언트 설정**

IAP(ID 인식 프록시) 를 설정하는 것에 대한 문서이다.

GKE에서 실행되는 애플리케이션에 대한 액세스를 관리, HTTPS로 액세스하는 애플리케이션에 대한 중앙 권한 부여 계층을 설정하여 애플리케이션 수준 액세스 제어 모델을 채택할 수 있다.

프로덕션 배포 또는 민감한 데이터에 액세스 하는 경우 권장

Oauth 동의화면 메뉴에서 docs를 따라 설정 및 저장하고

![oauth-client-1](/assets/images/oauth-client-1.png)

사용자 인증 정보 메뉴에서

**애플리케이션 유형에서** **웹 애플리케이션을** 선택, 생성된 클라이언트 ID를 복사한뒤

아래 화면과 같이 승인된 리디렉션 URI에 CLIENT_ID 부분에 입력후 저장한다.

```
https://iap.googleapis.com/v1/oauth/clientIds/<CLIENT_ID>:handleRedirect
```

여기까지 프로젝트 설정 및 OAuth 설정을 진행하였고 이후 관리 클러스터 배포 및 Kubeflow 클러스터 배포를 진행해보겠다.

# **관리 클러스터 배포**

docs에 따라 gcloud 구성요소 설치, 관리 클러스터를 설치한 Github 리포지토리 복제,

환경변수 구성까지 진행한다.

환경변수 구성시 

`LOCATION` 부분은 클러스터에 GPU 노드를 추가한다고 가정했을때

`gcloud compute accelerator-types list`

해당 명령어로 사용할 수 있는 GPU 와 Zone 정보를 미리 체크해서 정하는 것이 좋다.

GPU 노드 추가 설정부분은 docs 부분에서도 클러스터 셋업 이후 마지막 부분에 나오는데 보통 Kubeflow를 구축한다면 GPU를 같이 사용할 것이라 이 부분을 꼭 고려해서 작업하는 것이 도움된다.

[Add GPU nodes to your cluster](https://googlecloudplatform.github.io/kubeflow-gke-docs/dev/docs/customizing/#add-gpu-nodes-to-your-cluster)

환경변수 구성한 이후 `bash [kpt-set.sh](http://kpt-set.sh/)` 를 수행하면

아래 캡쳐화면 처럼 셋팅된 값들이 출력된다.

![manage-cluster-1](/assets/images/manage-cluster-1.png)

이제 관리 클러스터를 배포할 시간이다.

`make create-cluster`  명령어 수행하면 도커이미지를 pull 하고

클러스터 배포가 시작된다. 동시에 cloud console 화면에서도 클러스터 생성을 확인할수 있다.

![manage-cluster-2](/assets/images/manage-cluster-2.png)

![manage-cluster-3](/assets/images/manage-cluster-3.png)

# **Kubeflow 클러스터 배포하기**

이번에는 관리 클러스터와는 다르게 git repo 기준으로 
`kubeflow-distribution/kubeflow` 로 이동해서 작업한다.

잊지말아야할 것은 

env.sh 에서 가장 아래 부분에 앞서 설정했던 사용자 인증정보 화면의 클라이언트 ID와 클라이언트 보안 비밀번호를 추가설정하고

export CLIENT_ID=<Your CLIENT_ID>
export CLIENT_SECRET=<Your CLIENT_SECRET>

환경변수 설정을 완료하였다면 

`source env.sh` 를 실행후 

`bash ./kpt-set.sh` 를 실행한다.

그러면 설정한 환경변수 기준으로 셋업 작업이 시작되고 RUNNING, PASS 가 반복되며 진행된다

가장 마지막에는 “enabled”를 확인할수있다.

혹시 중간에 환경변수 설정을 수정한다면 kpt-set.sh 를 실행해 수정된 사항을 재반영해야한다.

![kubeflow-cluster-1](/assets/images/kubeflow-cluster-1.png)

`make apply-kcc` 명령어로 ConfigConnectorContext 를 적용하고

`make apply` 명령어로 kubeflow 클러스터 배포를 시작한다.

![kubeflow-cluster-2](/assets/images/kubeflow-cluster-2.png)

kubectl -n kubeflow get all

![kubeflow-cluster-3](/assets/images/kubeflow-cluster-3.png)

`gcloud projects add-iam-policy-binding "${KF_PROJECT}" --member=user:<EMAIL> --role=roles/iap.httpsResourceAccessor`

<EMAIL> 부분에 실제 사용할 이메일을 기입하고 명령어를 실행하면 등록되어있는 멤버와 해당하는 role을 확인할 수 있다.

이후 

`kubectl -n istio-system get ingress` 명령어를 실행하면

아래와 같이 접속할수있는 HOSTS 가 나오고 해당 HOSTS 를 복사해 접속한다


| NAME           | HOSTS                                                     | ADDRESS       | PORTS | AGE   |
|----------------|-----------------------------------------------------------|---------------|-------|-------|
| envoy-ingress  | your-kubeflow-name.endpoints.your-gcp-project.cloud.goog  | 34.102.232.34 | 80    | 5d13h |


간단히 구글 이메일 인증화면을 볼 수 있고 위에서 입력한 <EMAIL>로 구글 로그인을 하면 드디어 

Kubeflow 화면으로 이동한다

![kubeflow-cluster-4](/assets/images/kubeflow-cluster-4.png)



![kubeflow-cluster-5](/assets/images/kubeflow-cluster-5.png)

혹시나 하는 마음으로 등록하지 않은 이메일로 해당 Kubeflow 에 접속을 시도하면

아래와 같이 접속제한 화면을 확인할 수 있다.


![kubeflow-cluster-6](/assets/images/kubeflow-cluster-6.png)

kubeflow 에서 mnist 예제를 수행해보면 

Notebook 메뉴에서 name 을 입력하고 노트북을 생성하면 


![kubeflow-1](/assets/images/kubeflow-1.png)

Activity에서 workspace 생성 및 도커이미지를 pulling 하는 로그를 확인할 수 있다


![kubeflow-2](/assets/images/kubeflow-2.png)

GKE 화면 kubeflow-dev 클러스터에서도 default 노드풀과는 별개의 노드풀을 생성 하고 

노트북 생성시 입력한 리소스 자원에 맞춰 노드를 생성하는 것을 확인할 수 있다


![kubeflow-3](/assets/images/kubeflow-3.png)

셋업 작업이 완료되면 이전 Notebooks 메뉴에서 신규생성된 notebook을 확인하고 오른쪽 CONNECT 버튼이 활성화 됨을 확인할 수 있다.


![kubeflow-4](/assets/images/kubeflow-4.png)



![kubeflow-5](/assets/images/kubeflow-5.png)