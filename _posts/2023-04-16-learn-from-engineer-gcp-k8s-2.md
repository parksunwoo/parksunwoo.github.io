---
title: 엔지니어에게 직접 배우는 GCP, kubernetes, Firebase 환경세팅(2)
categories:
  - Dev
tags:
  - devops
  - gcp
  - kubernetes
last_modified_at: 2023-04-16T09:40:00-00:00
---

첫번째 글에서는 Firebase, GCP IAM, Cloud Run, Cloud Tasks, Cloud DNS, Filestore 를
하나씩 세팅해보면서 준비했고 이번 글에서는 GKE Cluster 세팅을 중점적으로 다뤄보려고 한다.

---
실습해볼 쿠버네티스 환경의 구조는 아래 그림과 같다.

![Kubernetes Architecture](https://www.dropbox.com/s/3ngnh2hcfye46wa/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-16%20%EC%98%A4%EC%A0%84%2012.47.31.png?dl=1)

## GKE Cluster

GKE에서 클러스터를 세팅을 시작해보자

Kubernetes Engine 메뉴에서 상단의 만들기를 선택한다.

![Kubernetes Engine](https://www.dropbox.com/s/9onka81w4ezw0x7/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-15%20%EC%98%A4%ED%9B%84%2010.48.50.png?dl=1)

그럼 아래와 같은 팝업이 나오는데 여기서는 **Standard 구성 을 선택한다**

Autopilot 모드는 핸드오프 환경을 제공하는 최적화된 kubernetes 클러스터로 즉시 사용할 수 있는 간소화된 구성, Google에서 노드를 관리하고 구성한다.

![클러스터 만들기](https://www.dropbox.com/s/6hx9blragd9l9ik/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-15%20%EC%98%A4%ED%9B%84%2010.50.46.png?dl=1)

반면 표준 모드는 모든 옵션 구성이 가능하며 노드를 직접 관리하고 구성한다.

실습에서는 직접 구성에 가까워서 표준 모드의 구성을 선택한다.


클러스터 기본 사항에서는 클러스터 이름과 위치 유형을 선택하면 되고 (실습에서는 system-pool과 us-central1)

노드풀 상세 구성으로 넘어가면

노드 수는 4개를 입력하고 
노드 풀 업그레이드 전략에서는 일시 급증 업그레이드를 선택 후 
최대 일시 급증 개수는 2개 최대 사용불가는 0개로 지정했다.

![클러스터 만들기2](https://www.dropbox.com/s/s0wvnimoz485jbr/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-15%20%EC%98%A4%ED%9B%84%2010.54.27.png?dl=1)

왼쪽에 사이드 메뉴로 노드/네트워킹/보안/메타데이터를 확인할 수 있다.
노드 > 노드 설정 구성에서 머신 구성은 e2-standard-2 (2 vGPU, 8GiB memory)를 선택했다.

빠뜨리면 안되는 것이 메타데이터에서 **Kubernetes 라벨 을 지정해야한다.**

해당 라벨을 사용하여 워크로드가 노드에 예약되는 방식을 관리한다.

실습에서는 키에 nodepoolname 을 값에는 system 을 지정하였다.

여기까지가 system-pool 이라는 노드풀을 구성한 것이다.

추가로 gpu-pool 노드풀을 추가해보겠다.

일시 급증 업그레이드 선택후 최대 일시 급증 개수는 1, 최대 사용 불가는 0 으로 지정했고,

![노드 풀 세부정보](https://www.dropbox.com/s/9zdh27rxa3gb1n4/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-15%20%EC%98%A4%ED%9B%84%2011.02.37.png?dl=1)

노드 설정 구성에서 머신 구성은 NVIDIA T4 GPU 1개로 지정했다.

![노드 설정 구성](https://www.dropbox.com/s/o4yxqtfs0axyzy3/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-15%20%EC%98%A4%ED%9B%84%2011.03.11.png?dl=1)

GPU 노드를 대량으로 사용한다면 클러스터를 생성할때 지정하는 리전을 고려해야한다.

아래 GCP GPU 리전 및 영역 사이트에서는 위치 또는 GPU 모델별로 검색해 사용가능한 플랫폼과

NVIDIA RTX 가상 워크스테이션을 확인할 수 있다.

[GCP GPU 리전 및 영역](https://cloud.google.com/compute/docs/gpus/gpu-regions-zones?hl=ko#gpu_regions_and_zones)

클러스터 생성 이후에는 지역을 변경하기가 어려워서 사용하는 GPU 확보가 용이한 지역을 먼저 확인후 지정하는 것이 좋다.

아래와 같이 특정 머신 유형은 정해진 리전 및 영역에서만 사용할수 있고

![리전 및 영역 제한사항 안내](https://www.dropbox.com/s/ezzb7fasabd2z40/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-15%20%EC%98%A4%ED%9B%84%2011.09.08.png?dl=1)

같은 서울 지역이라도 b와 c에 따라 사용할수 있는 GPU 종류가 다르다.

![asia-northeast3-b, c](https://www.dropbox.com/s/upuc94izc07i3g2/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-15%20%EC%98%A4%ED%9B%84%2011.09.21.png?dl=1)

관련된 이야기가 나왔기에 조금더 이야기해보면 T4 GPU를 사용할 수 있는 리전과 영역을 지정해서 클러스터를 세팅했다 하더라도 할당량이 중요하다. 무슨 이야기냐하면 GCP의 경우 할당량 (All Quotas) 메뉴에 접속하면 아래 화면이 나온다.

![프로젝트 할당량](https://www.dropbox.com/s/wbrz7ehg16znqp9/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-15%20%EC%98%A4%ED%9B%84%2011.14.32.png?dl=1)

가장 위에 있는 asia-east1 위치라면 생성할 수 있는 NVIDIA T4 GPU가 최대 16개이며

asia-south2 위치라면 1개이다. 이 한도는 요청할 수 있다는 것이지 T4 사용 한도만큼 보장하지는 않는다.

실제로 서비스하기 위해서는 할당량을 확보하는 것이 우선해결 되어야한다.

만약에 한도가 부족하다면 현재 보는 할당량 우측 탭이 상향 요청이고 여기서 한도추가 요청을 할 수 있다.

![한도 상향 요청](https://www.dropbox.com/s/e34fb6jth67dsti/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-15%20%EC%98%A4%ED%9B%84%2011.18.12.png?dl=1)

위 화면을 보면 가장 오른쪽 상태에서 승인이 된 것도 있지만 요청된 한도만큼 승인되지않고 일부만 승인된 것도 확인할 수 있다. 원하는 한도를 확보해야한다면 회신이 오기까지 기다릴수도 있지만

사용하고 있는 MSP 업체를 통해 상향 요청건을 전달해 한도를 확보할 수 있다.

클러스터가 생성된 것을 확인할 수 있고 우측 “연결”을 클릭하면 GCP 웹 콘솔에서 클러스터에 연결할 수 있다.

여기서는 테스트 중이라 system-pool 노드풀의 노드 수를 2개로 지정했다.

![클러스터 조회](https://www.dropbox.com/s/7eivcssa7drptl4/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-15%20%EC%98%A4%ED%9B%84%2011.22.00.png?dl=1)

![클러스터에 연결](https://www.dropbox.com/s/esascnxaceztnvi/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-15%20%EC%98%A4%ED%9B%84%2011.23.05.png?dl=1)

명령줄 액세스를 복사한후 CLOUD SHELL에서 실행을 클릭하면 아래 Cloud Shell 승인화면이 나온다.

승인을 누르면 클러스터에 연결되어 기본적인 동작을 수행할 수 있다.

![Cloud shell 승인](https://www.dropbox.com/s/4r2oupciodlwjbu/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-15%20%EC%98%A4%ED%9B%84%2011.24.50.png?dl=1)

gcloud 관련 설정이 되어있지않다면 아래와 같은 에러메시지를 확인할 수 있다.

```
ERROR: (gcloud.container.clusters.get-credentials) You do not currently have an active account selected.
Please run:

  $ gcloud auth login

to obtain new credentials.

If you have already logged in with a different account:

  $ gcloud config set account ACCOUNT

to select an already authenticated account to use.
```

key.json 이라는 GCP IAM service account key를 먼저 생성해볼 것인데

클러스터의 추가구성을 할때에 key.json 을 사용할 것이다

아래 명령어들을 하나씩 입력해보면서 key.json 을 생성해보자.
```
# 구글 로그인연동이 창이 나올것이다 로그인을 하면되고
gcloud auth login 

# 프로젝트 ID를 사용해 gcloud에서 사용할 프로젝트를 지정한다
gcloud config set project $PROJECT 

# test라는 이름의 서비스계정을 생성
gcloud iam service-accounts create test --display-name "test"

# 프로젝트에 생성한 서비스계정을 연결하고 dns 어드민 권한을 부여
gcloud projects add-iam-policy-binding $PROJECT \\
   --member <serviceAccount:test@$PROJECT.iam.gserviceaccount.com> \\
   --role roles/dns.admin

# key.json을 생성
gcloud iam service-accounts keys create key.json \\
--iam-account test@$PROJECT.iam.gserviceaccount.com
```

key.json이 생성되었다면 이후에는 별도 구성해놓은 쉘스크립트를 사용해 아래 작업이 차례로 진행된다.

먼저 daemonset gpu driver를 구성하고

```
kubectl apply -f <https://raw.githubusercontent.com/GoogleCloudPlatform/container-engine-accelerators/master/nvidia-driver-installer/cos/daemonset-preloaded.yaml>
```

`PriorityClass` 를 적용하고

daemonset의 role 을 구성하고, 초기 컨테이너 image를 구성하고,

`istio` 사이트에서 지정한 버전의 파일을 다운로드 및 설치하고

```
curl -L <https://istio.io/downloadIstio> | ISTIO_VERSION=$ISTIO_VERSION TARGET_ARCH=x86_64 sh -

./istio-$ISTIO_VERSION/bin/istioctl install -y
```

cert-manger를 설치한다

```
kubectl apply -f <https://github.com/jetstack/cert-manager/releases/download/v1.5.0/cert-manager.yaml>
```

istio 와 cert-manager 의 secret를 생성한다. key.json 정보를 사용해

```
kubectl create secret generic test-svc-acct \\
   --from-file=./key.json -n istio-system

kubectl create secret generic test-svc-acct \\
   --from-file=./key.json -n cert-manager
```

잠시후 `ClusterIssuer` (letsencrypt) 를 구성한다

생성한 ClusterIssuer를 사용해 `Certificate` 를 구성한다

istio-system 내 `ingressgateway` 를 구성한다

`sharedStorage` 를 구성한다 사용할 스토리지의 용량을 지정하고 nfs 서버 spec 도 지정한다

`sharedContainer` 를 구성한다. 앞서 구성한 sharedStorage 와 함께 개인별로 제공되는 엔드포인트에 접속했을때 연결되는 페이지의 포트와 설정을 세팅한다.

모든 설정이 완료되었다면

아래의 명령어의 결과값으로 EXTERNAL-IP와 CLUSTER-IP가 확인된다

```
kubectl get svc -n istio-system istio-ingressgateway
```

아래의 명령어의 결과값으로 READY 에 True가 확인된다 (https 연결가능)

```
kubectl get certificates -n istio-system
```

아래의 명령어의 결과값으로 나오는 shared-storage 엔드포인트에 접속이 가능하다

```
kubectl get virtualservices -A
```

아래 명령어를 입력하면

```
$ kubectl get pods -A
```

현재 실행중인 pod 정보들을 확인할 수 있다
별도로 띄워둔 worker부터 시작해 cert-manager, istio-system, kube-system, shared-storage 가 확인된다.


![kubectl get pods](https://www.dropbox.com/s/r2hyaoodz5uju75/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-16%20%EC%98%A4%EC%A0%84%208.16.38.png?dl=1)


Cloud Run을 통해 띄운 test-server 도 swagger 페이지가 조회되고 토큰값을 입력해 authorize하면
api들을 call 할 수 있다.

![api server docs](https://www.dropbox.com/s/baxuhzck0wkkypb/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-16%20%EC%98%A4%EC%A0%84%209.37.27.png?dl=1)

