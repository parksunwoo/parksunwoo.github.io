---
title: GKE 자동 업그레이드 후 Istio 버전 호환성 문제 해결- 경험과 교훈
categories:
  - Dev
tags:
  - gke upgrade
  - istio
  - k8s
last_modified_at: 2024-03-10T23:30:00-00:00
---

## Istio 버전 업그레이드 후 발생한 슬랙 에러 해결: 문제의 진짜 원인을 쫓는 모험

---

https://parksunwoo.github.io/dev/2023/05/20/a-istio-failure.html

해당 내용의 글이 게시되고 얼마 지나지 않아 한통의 메일을 받게되었다.

메일을 처음 확인했을때 보내주신 분(이하 A님) 이 석연치 않았던 부분을 정확히 지적해줘서 부끄러움은 잠시였고 고마운 마음이 더 컸다.

메일의 원문은 아래와 같다.

---

안녕하세요!

업그레이드 관련한 것이 아닌, 장애 상황의 원인에 대한 의문을 가지게 되어 질문을 드려봅니다.

istio의 업그레이드에 대해 알아보다가 포스트를 읽게 되었습니다. in-place 방식으로 istio 업그레이드한 경험을 볼 수 있어서 좋았어요.

다름이 아니라, 장애 상황에 대해 다시 읽어보다 보니 원인이 확실치는 않지만 범인이 istio 같지 않아서 의견을 남겨봅니다.

(일단 istio가 지원 종료되었다고 뭔가 달라질 것 같지 않았습니다.)

본문에 istiod의 에러 로그에서 *v1beta1.Ingress , *v1beta1.ValidatingWebhookConfiguration 리소스를 찾을 수 없다는 에러를 확인할 수 있습니다.

공교롭게도, 쿠버네티스 v1.22부터 Ingress에 해당하는 `[networking.k8s.io](http://networking.k8s.io/)` API와 ValidatingWebhookConfiguration에 해당하는 `[admissionregistration.k8s.io](http://admissionregistration.k8s.io/)` API는 Beta 버전인 v1beta1은 deprecate되고 GA버전인 v1으로 변경되었습니다. ([GKE 문서 참고](https://disq.us/url?url=https%3A%2F%2Fkubernetes.io%2Fblog%2F2021%2F07%2F14%2Fupcoming-changes-in-kubernetes-1-22%2F%3AP4ESaLW1QhvJIXIIqC_p4SBZE0k&cuid=5888907), [쿠버네티스 문서 참고](https://disq.us/url?url=https%3A%2F%2Fkubernetes.io%2Fblog%2F2021%2F07%2F14%2Fupcoming-changes-in-kubernetes-1-22%2F%3AP4ESaLW1QhvJIXIIqC_p4SBZE0k&cuid=5888907))

또한, 운영환경에서 사용하시던 istio 1.7.4는 소스를 분석해보니 deprecate된 `[networking.k8s.io/v1beta1](http://networking.k8s.io/v1beta1)`, `[admissionregistration.k8s.io/v1beta1](http://admissionregistration.k8s.io/v1beta1)` 을 사용하고 있습니다.

운영환경의 GKE 버전은 이 글에서 확인할 수 없지만, istio 1.7.4버전이 2020년 10월말~11월 릴리즈된 것으로보아 1.19, 1.20 혹은 그 이하 버전으로 생각됩니다. 그리고 해당 버전이 완전히 지원 종료되며 운영환경의 GKE의 자동 업그레이드 등의 이벤트가 있지 않았을까 조심스럽게 추측해봅니다.

장애 발생 전 운영환경 GKE 버전과 그 후 버전을 확인할 수 있을지 모르겠네요.

벌써 반년이 다 되어가는 장애 상황이지만, 원인을 명확히 파악하는 것이 언제 발생할지 모를 또 다른 장애 상황에 대비할 수 있기에 의견을 남겨봅니다.

(혹시 위 내용을 여유되실 때 확인하고 답글 달아주시면 정말 고맙겠습니다. 저도 궁금하네요!)

---

꽤 시일이 지난 과거이지만 운영환경의 GKE 버전을 확인할수 있는 방법을 찾아야했다.

```python
resource.type="gke_cluster"
protoPayload.metadata.operationType=~"(UPDATE_CLUSTER|UPGRADE_MASTER)"
resource.labels.cluster_name="CLUSTER_NAME"
```

GCP 상에 로그탐색기에 쿼리를 실행하니 클러스터 업그레이드 내역을 조회할수 있었다. 

내역을 조회해 보니 A님이 이야기한 부분이 충분히 가능한 이야기라 생각되고 istio 지원종료가 원인이 아닐수 있겠다는 생각이 들었다

이슈가 발생한 23년 5월 18일 20시 이전에 gke 업그레이드 로그가 있는지 확인하였을때 ([링크](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-upgrades?hl=ko#check-update-log))

5월 18일 19시 52분에 1.22.17-gke.7500 에서 1.23.17-gke.1700 로 클러스터 업그레이드가 진행되었음을 확인하였다.

```python
metadata: {

	currentMasterVersion: "1.23.17-gke.1700"
	operationType: "UPGRADE_MASTER"
	previousMasterVersion: "1.22.17-gke.7500"
	
}
```

참고로 5월 18일 이전 Kubernetes 부 버전 업그레이드는 4월 11일에

1.21.14-gke.18800 에서 1.22.17-gke.5400 로 클러스터 업그레이드가 진행되었다.

```python
metadata: {

	currentMasterVersion: "1.22.17-gke.5400"
	operationType: "UPGRADE_MASTER"
	previousMasterVersion: "1.21.14-gke.18800"
	
}
```

위의 단서들로 유추해본 이슈당일의 진행과정은 다음과 같았다.

-----------------------

(1) gke 버전이 자동업그레이드

(2) gke 자동 업그레이드 영향으로 istio 버전 호환성이 맞지않아 istio가 정상동작하지 않음

(3) istiod의 에러 로그에서 *v1beta1.Ingress , *v1beta1.ValidatingWebhookConfiguration 리소스를 찾을 수 없다는 에러발생

(4) istio를 1.7.4에서 1.13.3 으로 업그레이드 해 gke 버전과 호환성을 맞춰서 해결

로 생각해볼수 있었다.

하지만 한 가지 이상한 부분은 istio 1.7 버전의 Supported Kubernetes Versions 은 1.16, 1.17, 1.18 이었다.

1.7이 오래된 버전이라 commit 기록을 통해서 해당 정보를 찾을 수 있었다.

![https://github.com/istio/istio.io/commit/778b410748c985b6f7949044ebd51c519109efbb](https://prod-files-secure.s3.us-west-2.amazonaws.com/d31c1c0b-3aec-4ebc-9889-bf72ef09cf61/4c04d7ca-fd36-4f9d-b47a-b227aa0b0b0d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-03-10_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_10.56.09.png)

https://github.com/istio/istio.io/commit/778b410748c985b6f7949044ebd51c519109efbb

istio 1.7 버전에서 호환가능한 gke 버전이 1.22(?) 까지였는데 1.23으로 자동업그레이드 되면서 문제가 발생했고

istio 1.13 버전으로 업그레이드 하면서 gke 1.23 버전과 호환이 되었다고 생각하는게 맞을까?

![https://github.com/istio/istio.io/commit/f0658ddc6899cc6ab7048b6f1825552f7693de48](https://prod-files-secure.s3.us-west-2.amazonaws.com/d31c1c0b-3aec-4ebc-9889-bf72ef09cf61/d510e272-699d-4a34-87e7-4ddc079fc47a/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-03-10_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_10.57.25.png)

https://github.com/istio/istio.io/commit/f0658ddc6899cc6ab7048b6f1825552f7693de48

위 추가적인 내용을 정리해 A 님께 답장 메일을 보냈다.

그리고 며칠후 다시 A 님으로 부터 회신이 왔다.

A님의 메일 원문은 아래와 같다.

---

안녕하세요 박선우님 A라고 합니다.

먼저 답변 주셔서 너무 감사드리며, 아래 추가적은 제 생각과 분석을 정리했습니다.

두서없는 장황한 글이지만 천천히 봐주시면 감사하겠습니다.

일단은 5월18일자로 GKE 자동 업데이트가 19시52분에 있었다는 점, 블로그 포스트에 올리신 Slack의 알림이 20시1분에 왔다는 것이 확인되었고,

업데이트가 분명히 영향을 준 것으로 보여집니다.

먼저 분석해주신 4단계에 대해서 제 의견을 추가해보겠습니다.

***(1) gke 버전이 자동업그레이드***

> 실제로 5/18에 업데이트가 있었음, 시간상 맞아떨어짐

> 추가로 궁금한 점은 4월의 1.22.17-gke.5400 → 1.22.17-gke.7500 GKE 패치 버전 변경은 업그레이드가 아닌 것인지 궁금해집니다.

***(2) gke 자동 업그레이드 영향으로 istio 버전 호환성이 맞지않아 istio가 정상동작하지 않음***

> istio는 버전 변경 없음

> GKE 자동 업그레이드

> istio가 업데이트된 GKE 클러스터와 통신하는 과정에서 문제가 발생한 것으로 예상

***(3) istiod의 에러 로그에서 *v1beta1.Ingress , *v1beta1.ValidatingWebhookConfiguration 리소스를 찾을 수 없다는 에러발생***

> GKE 업데이트 영향으로 일부 API가 v1beta1 -> v1 으로 전환

> istio의 1.7.4 소스 코드를 확인하면, ValidatingWebhookConfiguration 및 Ingress 관련 코드에 모두 **v1beta1 API**를 사용 중

***(4) istio를 1.7.4에서 1.13.3 으로 업그레이드 해 gke 버전과 호환성을 맞춰서 해결***

> istio 1.13.3 소스 코드를 확인하면, ValidatingWebhookConfiguration 및 Ingress 관련 코드에를 사용 중

**v1 API**

> 업데이트된 GKE와 istio가 사용하는 API 간에 문제 없음

그리고 추가로 의문을 제기해주신 부분에 대해서 제 생각은 다음과 같습니다.

istio 1.7 버전의 Supported Kubernetes Versions 은1.16, 1.17, 1.18 이었습니다. ([링크](https://github.com/istio/istio.io/commit/778b410748c985b6f7949044ebd51c519109efbb))

istio 1.7 버전에서 호환가능한 gke 버전이 1.22(?) 까지였는데 ...

링크를 주신 대로 istio 1.7버전의 k8s 공식 호환 버전은 1.16~1.18이 맞는 것으로 보입니다.

다만 어디까지나 **"공식"** 이며, 1.22 버전의 API Deprecate 이전까지는 아마도 호환성을 공식으로 보장하지 않을 뿐이지 동작에는 문제가 없었을 것으로 생각합니다.

아래 버전 지원 표에서 음영 처리 한 부분을 보시면, istio 1.10 버전에 갑자기 **하위 버전들이 아닌 k8s 1.22 호환성이 "테스트는 되었지만 지원하지 않는 버전" 에 추가된 것**을 확인하실 수 있습니다.

실제로 k8s 1.22버전이 2021년 4월에 공개된 것으로 볼 때, 2021년 5월 istio 1.10버전에서 1.22 버전에 대한 호환성을 준비하는 버전이 1.10, 그리고 1.11부터 공식 지원이 이루어지지 않았을까 추측합니다.

1.9버전까지는 소스 코드에서 v1beta1 API를 사용하고 있으며, k8s 1.20까지 공식 지원하는 것으로 봐서 API의 큰 변경이 일어나기 직전인 1.21까지는 비공식적으로는 동작에 문제가 없었을 것 같습니다.

요약하면,

- ~ istio 1.9 :: v1beta1 API 사용, k8s 1.22 미발표로 아직 API deprecation에 대한 대응이 없음
- istio 1.10 :: API deprecation 에 대한 준비가 이루어졌으며, k8s 1.22를 비공식으로는 지원
- istio 1.11 :: k8s 1.22 출시 및 공식 지원

그래서... 요약하면 아마도 문제 발생 전에 박선우님의 클러스터의 istio 1.7.4 는 공식 지원되지는 않지만 
k8s 1.21 까지는 동작하는 상태로 예상해봅니다.

https://mail.google.com/mail/u/0?ui=2&ik=5363f94599&attid=0.1&permmsgid=msg-f:1786324580502379117&th=18ca4cbdda649a6d&view=fimg&fur=ip&sz=s0-l75-ft&attbid=ANGjdJ8RkOmhCMLMBYkFxSyIN4thIN7BEBtUQAbBLxt8q2-PLj6jFihqFndlk2X5VeAtAqCRiutXxw5oiTiZlKEFQM2pYqKPZbPcMePEoGfTMArzokM2yexAvct0k5o&disp=emb&realattid=ii_lqlxk7p80

그럼 여기서 의문이 들 수 있습니다.

**k8s 1.22에서 API Deprecated 라면서 왜 4월11일의 1.21 -> 1.22에 문제가 터지지 않고, 
5월18일 1.22 -> 1.23에서 문제가 터진거지? 시점이 안 맞는데?**

저도 여기서 막혔는데, 한 가지 실마리로 다음을 찾았습니다. (https://cloud.google.com/kubernetes-engine/docs/deprecations/apis-1-23?hl=ko)

위 링크는, Ingress의 경우 1회성으로 1.23까지는 v1beta1 API 를 사용하게 해준다는 GKE의 문서입니다.

아마도... Ingress와 함께 Webhook 쪽도 GKE에 한정하여 유예가 있지 않았을까 추측은 해보지만.. 확실하진 않습니다 ㅠㅠ

제 의견은 여기까지이며, 왜 1.23 업그레이드에서 문제가 발생했는지, 
GKE의 패치버전 1.22.17-gke.**5400** -> 1.22.17-gke.**7500** 은 별도의 업그레이드 정보가 없는지가

아직 풀리지 않은 부분입니다.

혹시나 istio의 코드 분석을 하시려면, 1.7.4 버전에서 에러코드중 하나인  Patch webhook failed  을 검색해보시면 될 것 같습니다.

istio 코드 전체에서 딱 한 줄이라서, 앞뒤 코드를 조금 보시면 어디서 deprecated되는 API를 사용하는지 확인가능합니다.

또 다른 의견과 정보 있으시면 언제나 회신주세요.

즐거운 연말되시고, 새해 복 많이 받으세요.

감사합니다.

---

A 님이 이야기해주신 GKE의 패치버전 업그레이드 정보를 찾기 위해 4월 부터 5월까지 업그레이드 기록을 
모두 조회했고 그 결과는 아래와 같다.

```python
#4/2 19시
metadata: {

currentMasterVersion: "1.21.14-gke.18800"
operationType: "UPGRADE_MASTER"
previousMasterVersion: "1.21.14-gke.18100"

}

#4/11 5시

metadata: {

currentMasterVersion: "1.22.17-gke.5400"
operationType: "UPGRADE_MASTER"
previousMasterVersion: "1.21.14-gke.18800"

}

#4/15 8시

metadata: {

currentMasterVersion: "1.22.17-gke.6100"
operationType: "UPGRADE_MASTER"
previousMasterVersion: "1.22.17-gke.5400"

}

#5/13 4시

metadata: {

currentMasterVersion: "1.22.17-gke.7500"
operationType: "UPGRADE_MASTER"
previousMasterVersion: "1.22.17-gke.6100"

}

#5/18 19시 52분

metadata: {

currentMasterVersion: "1.23.17-gke.1700"
operationType: "UPGRADE_MASTER"
previousMasterVersion: "1.22.17-gke.7500"

}
```

로그에 따르면

GKE의 패치버전 1.22.17-gke.**5400** -> 1.22.17-gke.**7500** 은

4월 15일에 gke.5400 에서 gke.6100 으로 업그레이드 되고 이후

5월 13일에 gke.6100 에서 gke.7500 으로 업그레이드 되었다.

A님이 의견을 주셨던 아래 내용이 맞는지도 확인이 필요했다.

“Ingress와 함께 Webhook 쪽도 GKE에 한정하여 유예가 있지 않았을까 추측”

MutatingWebhookConfiguration 과 ValidatingWebhookConfiguration 은 무엇이고 어떤 역할은 하는지?

- **MutatingWebhookConfiguration:** Pod 스펙을 변경하는 Webhook을 정의합니다.
    - **사용 사례:**
        - Pod에 필요한 sidecar 컨테이너 자동 주입
        - Pod 설정 유효성 검사 및 기본값 설정
        - Pod spec에 대한 사용자 지정 로직 적용
- **ValidatingWebhookConfiguration:** Pod 스펙을 유효성 검사하는 Webhook을 정의합니다.
    - **사용 사례:**
        - Pod spec의 필수 필드 검사
        - Pod spec의 정합성 검사
        - Pod spec에 대한 사용자 지정 유효성 검사 로직 적용

**1.23 버전부터 MutatingWebhookConfiguration과 ValidatingWebhookConfiguration API v1beta1이 지원 중단되었습니다.**

**변경 사항 요약:**

- **v1beta1 API 더 이상 사용 불가능:** 1.23 버전 이상의 GKE 클러스터는 v1beta1 API를 지원하지 않습니다.
- **v1 API 사용 필요:** Webhook을 계속 사용하려면 v1 API로 마이그레이션해야 합니다.
- **마이그레이션 과정:**
    - Webhook 코드 업데이트: v1 API 버전에 맞춰 Webhook 코드를 변경해야 합니다.
    - 매니페스트 업데이트: Webhook 사용을 위한 매니페스트에서 API 버전을 v1beta1에서 v1로 변경해야 합니다.

위 설명에 따르면 1.23부터 v1beta1 API 는 사용 불가이기에 5월 18일에 1.23 버전으로 자동 업그레이드 되면서 호환되지 않는 v1beta1 API 사용으로 이슈가 발생한게 맞게 된다.

그럼 여기서 Kubernetes 버전 업그레이드 주기는 어떻게 되고 Istio와 같은 패키지 버전 관리는 어떻게 효율적으로 할수 있을지 정리가 필요하다

**Kubernetes 버전 업그레이드 주기는**

- **마이너 버전:** 3개월마다 출시됩니다. 버그 수정 및 안정성 향상에 중점을 둡니다.
- **메이저 버전:** 1년마다 출시됩니다. 새로운 기능 추가 및 기존 기능 개선에 중점을 둡니다.

이에 따라 권장하는 **업그레이드 주기는**

- **프로덕션 환경:** 최소 6개월 이상 버전을 유지하고, 지원 종료 3개월 전에 업그레이드하는 것이 좋습니다.
- **테스트 환경:** 최신 버전을 유지하여 새 기능 테스트 및 업그레이드 프로세스 검증

업그레이드 전에 항상 백업을 수행해야하고 패키지 버전 관리를 위한 최적의 방법으로는 3가지를 제시한다

- **버전 호환성 매트릭스 참조:** Kubernetes, Istio 및 기타 사용 중인 패키지의 버전 호환성 매트릭스 확인

- **자동화 도구 활용:** ArgoCD와 같은 자동화 도구를 사용하여 배포 프로세스 자동화 및 버전 관리
- **테스트 환경에서 업그레이드 테스트:** 프로덕션 환경에 적용하기 전에 테스트 환경에서 업그레이드 테스트 수행

위 내용을 참고해 패키지 버전관리를 하는 것에 대해서는 다른 글로 정리해보려고 한다.

---

작성한 블로그 글을 통해 미처 확실히 정리하지 못한 부분을 짚고 넘어갈 수 있어서 좋았고

또한 작성한 글을 통해 누군가와 이야기하고 문제의 원인을 찾아간다는 느낌을 가질수있어 잊지못할 경험이 되었다.