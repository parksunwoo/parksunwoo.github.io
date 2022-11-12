---
title: BAM - Bottleneck Attention Module
categories:
  - ML
tags:
  - slowpaper
  - bam
  - attention
  - bottlenecks
last_modified_at: 2019-01-20T23:00:00-00:00
---

# slow paper
================================================

어텐션 모듈을 잘 활용해서 더 높은 성능을 끌어올릴 수 있을까?

figure1. BAM integrated with a general CNN architecture

위 그림을 자세히보면 BAM을 통해 생성된 어텐션 맵이 기존 피쳐맵에 적용되면서 이후 타겟의 특징에 더 집중되는 것을 볼 수 있습니다.

해당논문은 deep neural network 에서 어텐션을 효과적으로 사용하는 것에 중점을 두고 있습니다. 어텐션 맵을 생성할때에 채널뿐 아니라 공간적인 부분도 고려해서 말입니다. 특징들을 추출해내는 다운샘플링 과정인 bottleneck 부분에 어텐션 모듈이 위치할때 가장 효율적임을 다양한 데이터셋 실험을 통해 증명했습니다.

논문개요, 모델구조, 관련연구들 순서로 설명해보려 합니다

**Introduction**

딥러닝 부문에서 연구자들은 성능향상을 위해 다양한 시도들을 하고있었습니다. 예를 들면 optimizer 설계, adversarial training 제안, task-specific 한 메타 아키텍쳐 설계 등 입니다. 성능향상을 위한 본질적인 접근은 모델 핵심구조를 잘 설계하는 것일 겁니다. 모델 각각의 구조설계에 따라 AlexNet, VGGNet, GoogLeNet, ResNet, DenseNet 이 제안되었습니다. 모델구조 핵심은 더 많은 층을 쌓는 것입니다. 또한 같은 층에서도 더 다양한 특징들을 추출한다면 성능도 더 높아질겁니다.

figure1. AlexNet vs VGGNet

3D 피쳐맵이 주어졌을때, BAM은 중요한 부분을 강조한 3d 어텐션 맵을 생성할 수 있습니다. 이 과정에서 2개의 큰 흐름(spatial and channel)으로 나누어 진행됩니다.

**Bottleneck Attention Module**

입력값으로 주어진 피쳐맵을 F 라고 할때, BAM은 3D 어텐션 맵 M(F)를 입력 원본인 F와의 element-wise multiplication 을 더한 것으로 나타냅니다.  
재정의된 피쳐맵 F’ 는 아래와 같습니다.

eq. (1)

오른쪽 첫번째 항 F를 왼쪽으로 이항시켜서 바라보면 재정의된 F’와 원본 F의 차이가 원본 F와 BAM 함수를 multiplication 한것과 같음을 알 수 있습니다. 논문에서 어텐션을 적용할때에 채널 어텐션 부분과 Mc(F)와 공간 어텐션 부분 Ms(F)를 각각 구분해서 계산하였습니다. 최종적으로 계산된 어텐션 맵입니다

eq. (2)

\-Channel attention branch

각 채널별로 피쳐맵을 aggregate해서 (Global avg pool)을 사용해서 공간 정보는 삭제하고 채널의 정보만을 보려고 한 부분이 figure2에서 상단 부분입니다. 파라미터의 overhead를 줄이기 위해 본래 채널을 감소비율 r만큼 감소시킵니다 : C/r

figure2. model architecture

\-Spatial attention branch

다른 공간위치에서 정보들을 추출하는 spatial 어텐션 맵 Ms(F) 입니다. 주요정보를 추출하는게 핵심이고 여기서는 채널의 크기를 줄임(C/r)과 동시에 수용범위를 확장시키기 위해 dilated convolution을 사용했습니다 figure2 하단부분 — 필터의 간격을 넓혀 필터 전체 크기를 키우고 넓은 범위에서 보는 효과 — 1\*1 conv 를 사용해 채널의 정보는 사라지고 spatial 부분의 정보만 남게되었습니다.

\-Combine two attention branches

2개부분에서 각각의 어텐션 Mc(F)와 Ms(F)를 얻었다면 이를 결합해서 3D 어텐션 맵 M(F)를 출력합니다. 2개의 어텐션 맵은 서로 크기가 다르기에 우리는 element-wise summation을 사용해서 크기를 맞추고 0에서 1사이 범위를 가지는 시그모이드함수를 통과시켰습니다. 최종적으로 구해진 어텐션 맵 M(F)를 eq(1) 처럼 원본 F에 더해 F’를 구합니다

**Related Work**

*   Cross-modal attention  
    어텐션 메커니즘은 서로 상이한 정보들에서 자주 사용되는 기법입니다. Visual question answering (VQA)에서 이미지와 자연어 질문이 주어질 때, 주어지는 질문에 따라 이미지에서 처리되는 부분도 달라지므로 dynamically changing task의 하나로 볼 수 있습니다. 또 다른 방식으로 사용되는 bi-directional inferring에선 텍스트와 이미지 모두에서 어텐션 맵을 생성합니다.
*   Adaptive modules  
    입력값에 따라 출력값을 바꾸는 adpative modules을 사용한 연구들이 있습니다. Dynamic Filter Network에서는 입력값의 특징에 기반해서 convolutional 특성이 생성되는 것을 제안했으며 Spatial Transformer Network에선 입력 특성을 사용해서 하이퍼파라미터 affine 변환을 생성했습니다.

어텐션 모듈의 효율적인 위치가 네트워크의 bottleneck 부분(pooling 이전) 임을 여러 실험을 통해 찾았습니다. 흥미롭게도 계층적 추론과정이 인간이 지각하는 과정과 비슷함을 발견했습니다. 기존 CNN모델에 BAM모듈을 더하는것만으로 성능향상을 가져온다는데 의의가 있습니다
