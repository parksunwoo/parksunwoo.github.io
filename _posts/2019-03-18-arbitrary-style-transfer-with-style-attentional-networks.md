---
title: Arbitrary Style Transfer with Style-Attentional Networks
categories:
  - ML
tags:
  - slowpaper
  - transfer  
  - style
  - attention
last_modified_at: 2019-03-18T22:00:00-00:00
---


# slow paper
=====================================================================

일상의 평범한 사진이 새로운 예술작품이 되는순간!

아래 글의 모든 그림과 수식은 Arbitrary Style Transfer with Style-Attentional Networks 논문을 참고했고, Style Transfer 관련 설명 부분은 [Lunit Tech Blog](https://blog.lunit.io/2017/04/27/style-transfer/) 를 참고했음을 미리 밝힙니다.

1.  **Introduction**

**Arbitrary style transfer 란?**

콘텐츠 이미지(사물이나 사람)에 다른 스타일 이미지(화가 고유의 그림체)를 입혀서 새로운 이미지를 생성해내는 걸 말합니다. 콘텐츠 이미지의 원래 구조를 최대한 유지하면서 스타일을 입히는 부분에서 트레이드 오프관계가 있어왔고 새로운 이미지를 생성하는데 오랜 시간이 걸린다는 문제가 있었습니다.

Style transfer 연구는 크게 두 분류로 나눌수 있습니다.

첫번째는 Image -net 등의 데이터로 미리 학습된 네트워크를 이용하는 방법으로, 콘텐츠 이미지와 스타일 이미지를 네트워크에 통과시킬 때 나온 각각의 피쳐맵을 저장하고, 새롭게 생성될 영상의 피쳐맵이 이미 저장된 피쳐맵 2개와 비슷한 특성을 가지도록 영상을 최적화하는 방법입니다.

두번째는 스타일 트랜스퍼 네트워크를 학습시키는 방법입니다. 서로 다른 두 도메인(실제 사진과 화가의 작품)의 영상들이 주어졌을 때 한 도메인에서 다른 도메인으로 바꿔주도록 학습 시킵니다.

이전 연구들은 Global 스타일과 Local 스타일 패턴을 모두 고려하기가 힘들었습니다. 콘텐츠 이미지의 형태를 유지하려면 스타일 이미지를 살려내기가 어렵고 스타일 이미지에 중심을 두면 본래의 콘텐츠 이미지가 사라져버렸기 때문입니다. 로스함수도 스타일 로스와 콘텐츠 로스 두개의 결합이기에 두 로스간 트레이드 오프 관계가 있었습니다.

**Self Attention mechanism**

이미지생성과 기계번역에 사용되는 방법으로 이미지(시퀀스) 각 부분과 다른 부분과의 관계를 계산해서 가중치를 구합니다. 이 논문에선 스타일 어텐션 네트워크가 콘텐츠의 특성과 스타일 특성 사이의 관계를 구합니다.

*   style-attentional network(SANet)와 디코더 그리고 identity 로스를 사용한 feed forward 네트워크 방식의 학습방법을 사용했습니다
*   다른 방식에 비해 Global, Local 스타일 패턴 균형을 유지한채 높은 품질의 스타일 이미지를 생성합니다
*   identity 로스를 추가함으로써 Global, Local 스타일 패턴을 살리고, 콘텐츠 형태를 유지하면서 스타일을 입히는데 성공했습니다.

**3.1 Network Architecture**

입력값으로 콘텐츠 이미지 Ic, 임의의 스타일 이미지 Is 가 인코더 역할을 하는 훈련된 VGG19 네트워크를 통과합니다. Global 스타일 패턴과 Local 스타일을 적절히 결합시키기 위해서 서로 다른 레이어층에서 나온 피쳐맵을 결합합니다.

콘텐츠 이미지와 스타일 이미지는 VGG 인코더를 지나 각각 Fc, Fs 피쳐맵이 됩니다. 생성된 피쳐맵은 SANet 모듈을 거쳐 피쳐맵 Fcs 로 변환됩니다.

eq(1)

다시 1 X 1 Conv를 통과한 Fcs 와 Fc와의 element wise sum 값으로 Fcsc가 나옵니다.위 그림에서 보라색으로 표시한 Fc가 위와 아래에서 conv를 통과한 값과 만나는 부분은 skip connection을 의미합니다.

eq(2)

두 개의 SANet을 통과한 피쳐맵을 결합하는데 relu\_5\_1 을 통과한 피쳐맵 Fcsc를 upsampling을 해서 relu\_4\_1 을 통과한 피쳐맵과 결합합니다. upsampling을 하는 이유는 아래 VGGNet 구조에서 보라색으로 표시한 첫번째 Conv와 두번째 Conv 사이에 Max-pooling layer 가 있어 서로 다른 크기를 맞춰주기 위함입니다.

**3.2 Style-Attentional Network for Style Feature Embedding**

아래 그림은 SANet 모듈을 사용해서 스타일 피쳐 임베딩을 하는 것을 나타냅니다. 인코더를 통과한 콘텐츠 피쳐맵 Fc 와 스타일 피쳐맵 Fs 는 두 피쳐맵 사이의 어텐션을 계산하기 위해 정규화 및 두 개의 피쳐 공간 f, g를 거쳐 변환됩니다. 학습을 통해 두개 피쳐맵 사이의 관계를 맵핑함으로써 콘텐츠 이미지의 각 위치에 적절히 local 스타일 패턴을 임베딩할 수 있습니다.

**3.3 Full Objective**

eq(3)

eq(3)은 콘텐츠 로스 Lc와 스타일 로스 Ls 그리고 identitiy 로스 L\_identity로 이루어져있습니다. 람다는 각기 다른 로스의 weight 를 나타냅니다. 아래는 Identity 로스에 대한 그림입니다. 입력값으로 동일한 이미지 쌍 Ic와 Ic 를 모델에 넣었을 때 나온 값과 Icc 간 차이를 계산해서 L\_identity를 구합니다.

앞서 말한것처럼 콘텐츠 로스와 스타일 로스는 콘텐츠 이미지의 구조와 스타일 패턴중 어떤 것에 무게를 둘지에 따라 트레이드 오프 관계를 갖는다고 했습니다. 두 로스와 달리 identity 로스는 스타일 특성의 차이가 없는 동일한 입력 이미지로부터 계산된 값입니다. 따라서 스타일 특성을 변화시키기보다 콘텐츠 이미지 구조를 유지하는데 더 무게를 두게됩니다.

identity 로스를 사용해서 콘텐츠 이미지의 구조를 유지시킨 부분과 어텐션을 적용해서 효율을 높인 부분이 크게 다른 점으로 보입니다. 기존 다른 모델들과의 비교부분을 보면 좀 더 작품스타일과 가까워보이는 것 같습니다. 브라우저 상에서 이미지들을 변환하고 확인할 수 있는 링크를 남깁니다.

[

Arbitrary Style Transfer in the Browser
---------------------------------------

### This is an implementation of an arbitrary style transfer algorithm running purely in the browser using TensorFlow.js…

reiinakano.github.io

](https://reiinakano.github.io/arbitrary-image-stylization-tfjs/)
