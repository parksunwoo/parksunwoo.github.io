---
title: Generative Flow with Invertible 1x1 Convolutions
categories:
  - ML
tags:
  - slowpaper
  - generative  
  - vae
  - normalize
last_modified_at: 2019-05-21T22:00:00-00:00
---

# slow paper
=====================================================================

생성모델에 대한 배경지식들을 정리하고 시작하려고 합니다. 생성모델에 대한 내용은 [해당 블로그](https://lilianweng.github.io/lil-log/2018/10/13/flow-based-deep-generative-models.html)의 내용을 참고하였고 추가로 [야코비안 행렬](https://darkpgmr.tistory.com/132), [VAE](https://ratsgo.github.io/generative%20model/2018/01/27/VAE/) 를 참고하여 작성했습니다

**Background**
==============

**Types of Generative Models**

생성모델에는 GAN, VAE 그리고 Flow-based 생성모델이 있습니다. 각 모델의 특징은 무엇일까요?

*   Generative adversarial networks : GAN은 생성자와 구분자로 구성되어있습니다. 구분자는 가짜 데이터가 포함된 전체데이터에서 실제 데이터를 구분해내는 학습을, 생성자는 실제와 비슷한 데이터를 생성해서 서로간 상호학습을 하는 방식입니다. 구분자와 생성자는 minimax 게임을 하는 플레이어 역할을합니다. minimax 게임은 최소극대화라고 하며 최악의 경우 발생가능한 손실을 최소화 한다는 의사결정 원칙입니다.
*   Variational autoencoders : VAE는 데이터의 log likelihood 를 최적화하는 하한선의 최대값을 찾는 방식입니다. 데이터가 생성되는 과정, 즉 데이터의 확률분포를 학습하기 위한 두 개의 뉴럴 네트워크로 구성되어 있습니다. VAE는 잠재변수 Z를 가정하고 있는데 우선 인코더는 관측된 데이터 x를 받아서 잠재변수 Z를 만들어 냅니다. 디코더는 인코더가 만든 Z를 활용해 x 를 복원해내는 역할을 합니다.
*   Flow-based generative models : 연속적인 역변환을 통해서 생성하는 방식입니다. 데이터의 분포에서 학습하는 방식입니다.

Fig1. _Comparison of three categories of generative models (_[https://lilianweng.github.io/lil-log/2018/10/13/flow-based-deep-generative-models.html](https://lilianweng.github.io/lil-log/2018/10/13/flow-based-deep-generative-models.html))

그림을 보면 GAN과 VAE는 각각 구분자와 인코더 부분이 사다리꼴 모양입니다. 이는 원본 데이터를 압축하는 것을 의미하고, 압축 및 확장하는 과정에서 데이터의 손실이 발생할 수 있습니다. 하지만 Flow-based 생성모델은 그림에서도 볼 수 있듯이 역함수 변환을 통해 데이터의 손실을 줄일 수 있습니다.

**Linear Algebra Basics Recap**

flow-based 생성모델의 핵심 개념인 야코비안 행렬식과 the change of variable rule 에 대한 이해가 필요합니다

**Jacobian Matrix and Determinant**

n차원의 입력 벡터 x를 m차원의 출력벡터로 변환할때, 1차 편미분 행렬을 야코비안 행렬이라 합니다. 그레디언트는 다변수 스칼라 함수에 대한 일차 미분인 반면 야코비언은 다변수 벡터 함수에 대한 일차미분입니다. 어떤 함수의 지역적인 변화특성을 파악할때, 지역적인 함수의 변화를 선형근사할 때 또는 함수의 극대(극소)를 찾을 때 활용합니다.

행렬식은 정방형 행렬에서 모든 요소들을 고려해서 계산되어지는 하나의 수입니다. 행렬식이 0이라면 역변환이 불가능 합니다. (singular 행렬 or 모든값이0일때)

**Change of Variable Theorem**

수학에서 변수변환은 어떤 변수(들)로 나타낸 식을 다른 변수(들)로 바꿔 나타내는 기법입니다. 확률밀도 예측부분에 한해서 설명해보면, 랜덤변수 z가 주어지고 z의 확률밀도를 z ~ π(z) 라고 가정해봅시다. f 라는 함수를 통해서 새로운 변수를 만든다면 x = f(z)가 되고, 함수 f는 역변환이 가능함으로 다시 z = f-1(x)로 표기됩니다.

우리가 알고 싶은건 어떤 새로운 변수의 알지못하는 분포를 어떻게 추론할 수 있을 것인가이며 여기서 핵심은 복잡한 분포를 계산을 쉽게하기 위해 우리가 알수있는 정보들로 치환하는 것입니다.

What is Normalizing Flows?
==========================

여러 머신러닝 문제들에서 적절한 확률분포 예측은 중요한 문제이나 쉽지않은 문제입니다. 역전파를 진행하기 위해선 사후확률 p(**z**)|**x** 와 같은 확률 분포가 계산을 위해 단순해야 합니다. 가우시안 분포가 잠재변수 생성모델에 자주 사용되는 이유이기도 합니다. 실생활의 여러 분포들은 가우시안보다 훨씬 더 복잡하지만 가우시안 분포를 통해 이를 단순화해서 표기하고 문제를 풀어나갑니다.

**Normalizing Flow** (NF) 은 확률분포를 예측하는 파워풀한 방식입니다. NF는 단순한 분포를 연속된 역변환 함수를 적용시켜 복잡한 분포로 변환해나가는 방식입니다. 연쇄적으로 변환하는 과정을 통해, change of variables theorem 에 따라 새로운 변수를 반복적으로 대체해서 최종 타겟 변수의 확률 분포를 얻습니다.

Models with Normalizing Flows
=============================

RealNVP (Real-valued Non-Volume Preserving)  
RealNVP는 일련의 bijective 변환 함수들을 쌓아 normalizing flow를 구현했습니다. affine coupling layer 가 나오는데 이는 입력값의 차원을 두 개 부분으로 나누어 처음부터 d까지 차원은 그대로 두고, 나머지 d+1 부터 D까지 차원은 scale and shift affine 변환을 합니다. 이러한 변환을 하면 1) 역변환이 쉽고 2) 야코비안 행렬식을 쉽게 계산할 수 있습니다. 최종적으로 야코비안 행렬은 lower 삼각행렬 형태가 됩니다. affine coupling layer 에서 처음 d까지 차원은 그대로 두게 되면 모델 학습의 의미가 없기에 임의적으로 순서를 조정하는 작업을 하게됩니다.

NICE (Non-linear Independent Component Estimation)  
NICE는 RealNVP에 앞서 나온 모델로 affine coupling layer에서 scale 부분이 없는게 차이점입니다.

Glow  
아래 (a) 그림은 전체 큰 흐름을 나타내고 있습니다. actnorm은 activation normalization(활성함수 표준화)을 의미하며 배치 표준화와 비슷하지만 스케일과 편향 파라미터를 채널마다 사용해서 어파인 변환을 합니다. 미니 배치사이즈는 1로. 앞선 두개 모델에서 더 나아가 채널 순서에 대한 역순열 연산을 invertible 1x1 conv 로 바꿔서 구조를 단순화했습니다.

fig3. generative flow & multi scale architecture

(b) 그림은 x 라는 이미지를 입력값으로 넣고 squeeze를 통해 공간 해상도를 줄이고 (a) 과정을 K번 반복하고 split하는 전체 과정을 다시 (L-1) 번 반복하는 멀티 스케일 구조를 나타내고 있습니다.

위 사진들은 실제 인물이 아닙니다. 모델을 통해서 생성한 인물 사진입니다. 어딘가에서 봤음직한 사진들이 실제가 아니라고 하니 한편으론 스산한 기분이 드네요.
