---
title: TimbreTron- A WaveNet(CycleGAN(CQT(Audio))) Pipeline for Musical Timbre Transfer
categories:
  - ML
tags:
  - slowpaper
  - timbre  
  - rainbowgram
  - wavenet
  - cqt
last_modified_at: 2019-05-21T22:00:00-00:00
---

# slow paper
=====================================================================

소리 속 악기를 바꾸는 실험!

푸리에 변환 설명은 해당 [블로그](https://darkpgmr.tistory.com/171)를, 웨이브넷 설명은 [딥마인드 페이지](https://deepmind.com/blog/wavenet-generative-model-raw-audio/)를, 추가로 해당 [슬라이드](https://www.slideshare.net/TaesuKim3/pr12128-timbretron-a-wavenetcyclegancqtaudio-pipeline-for-musical-timbre-transfer)를 참고했습니다.

Yotube : TimbreTron: A WaveNet(CycleGAN(CQT(Audio))) Pipeline for Musical Timbre Transfer

**1\. Introduction**

주기 : 파동이 한번 진동하는데 걸리는 시간, 또는 그 길이.  
주파수 : 1초 동안의 진동 횟수, 주파수와 주기는 서로 역수 관계

Timbre 란?

소리의 3요소를 구성하는 건 높이, 크기, 음색입니다. 여기서 음색은 악기 고유의 특성을 나타내며 높이와 크기가 동일하더라도 구분되어집니다. 모든 악기에서 생성된 소리는 동시에 발생하는 많은 진동을 생성하고 속도가 가장 느린 진동을 기본 주파수라고 하며 가장 큰 소리입니다. 우리가 음악톤을 귀로 식별하기 위해선 3가지 이상의 고조파를 필요로 합니다. 그러나 고품질의 오디오 재생에는 밀도가 높은 음색을 유지하기 위해 기본적으로 7 개의 단순 고조파가 필요합니다.

이번 논문에서는 높이,박자, 크기를 유지한 채로 음악 샘플에서 악기를 바꾸는 실험을 시도했습니다. TimbreTron은 이미지 도메인에서 스타일을 전환하는 방식을 시간과 주파수로 표현된 음악신호에 적용한 방식입니다. 웨이브넷을 사용해서 고품질의 파형을 생성했습니다. 음색을 모델링하거나 변환하는 작업은 다른 소리를 실험하거나 다양한 악기들로 작업을 하는 음악인들에게 중요한 작업이나 어려운 작업이기도 합니다.

TimbreTron 이 음색을 변환하는 과정은 3 단계입니다. 먼저 악기1의 파형에서 CQT 스펙트로그램을 계산하고 phase 정보를 버리고 magnitude 값만 이용, 이미지를 만듭니다. 다음으로 CycleGAN을 사용해서 악기2의 CQT 스펙트로그램으로 변환합니다. 생성된 악기2의 CQT 스펙트로그램에 웨이브넷을 이용하여 악기2의 파형으로 복원합니다.(phase 정보포함)

**2\. Background  
**2.1 Time- Frequency Analysis

시간-주파수 분석은 신호의 주파수를 시간의 변화에 따라 어떻게 측정할지에 대한 방법입니다. 푸리에 변환에 대해 먼저 알아보아야합니다. 푸리에변환은 임의의 입력 신호를 다양한 주파수를 갖는 주기함수들의 합으로 분해하여 표현하는 것입니다. 각각의 주기함수 성분들은 고유의 주파수와 강도를 가지고 있으며 이들을 모두 합치면 원본 신호가 됩니다.

푸리에 변환의 페이즈(phase)  
위상. 반복되는 파형의 한 주기에서 첫 시작점의 각도 혹은 어느 한 순간의 위치라고 정의합니다. 파형(wave)의 시점이 어디인지가 페이즈가 되고 페이즈는 원본 신호를 주기 신호로 분해했을 때 각 주기성분의 시점이 어디인지를 나타내는 요소가 됩니다. 푸리에 계수는 주기함수 성분의 강도(amplitude)를 나타내는 스펙트럼 정보와 시점을 조절하는 페이즈(phase) 정보가 함께 포함되어 있습니다.

Figure2. STFT — CQT — Rainbowgram

Short Time Fourier Transform  
시간 주파수 분석에서 가장 많이 사용되는 방법입니다.

eq (1)

위 식은 STFT 계산식으로 시간 도메인 신호인 x\[n\] , 시간 스텝 m 그리고 진동수 w\_k 를 입력값으로 합니다. 마스킹 된 신호 x\[n\] w\[n-m\]의 이산 푸리에 변환으로서 해석 될 수 있습니다.

Constant Q Transform  
또 다른 시간 주파수 분석 방법으로 상이한 주파수 대역들 사이의 기하학적 분리를 고려했습니다. 그림2를 보면 STFT 와 달리 y축이 로그 주파수를 나타내어 STFT와 비교했을때 고주파에서도 더 많은 정보를 나타내고 있습니다.

Rainbowgram  
CQT 를 색을 사용해서 시각화 한 것으로 CQT 에서 보이지않는 미세한 음색 특성을 강조합니다. 동일한 레인보우그램이나 피아노와 플루트의 모습이 다른 것은 같은 음이라도 악기마다 다른 것을 의미합니다.

2.2 Waveform reconstruction from spectrograms파형을 복원할때에는 강도magnitude 와 위상phase 정보가 모두 필요합니다. 하지만 우리는 앞서 모델링을 할때 위상정보를 버렸기 때문에 강도 정보에서 위상정보를 생성하는 Griffin-Lim 알고리즘을 사용합니다. 이 알고리즘은 프로세스 전체과정에서 강도 값을 일정하게 유지한 채로 STFT 및 역 STFT 연산이 수렴될 때까지 무작위로 위상 값을 추측하는 과정을 반복합니다.

2.3 WaveNet  
웨이브 넷은 auto-regressive 생성모델로 고품질의 오디오 파형을 생성합니다. dilated causal convolution 을 여러층 쌓은 형태입니다. causal filter 는 모델을 학습하는데 있어서 미래시점 데이터를 사용하지 않도록 마스킹 해주는 기능을 가지고 있으며, dilated convolution은 훨씬 이전 시점의 데이터를 고려하고 수용영역을 효율적으로 증가시키기 위한 방법입니다. 웨이브넷의 한계는 파형을 생성하는데 비용이 많이 든다는 것입니다.

2.4 GAN and CycleGAN  
GAN 은 구별자와 생성자로 구성되어 있으며 서로 min-max 게임을 하는 플레이어가 됩니다. 쉽게 말하면 구별자는 샘플 데이터에서 진짜 데이터를 구별하려고 계속 시도하고 한편에서 생성자는 구별자가 판별하기 힘든 가짜 데이터를 만들어내는 것입니다. CycleGAN은 매칭되는 데이터 쌍없이 두개 도메인의 관계를 학습하는 비지도 도메인 전이구조입니다.

**3\. Music processing with constant-q-transform representation**

3.1 CQT for music representation  
CQT 는 특별히 음악 오디오 신호를 처리하는데 적합합니다. STFT 와 달리 낮은 주파수대역에선 더 높은 주파수 해상도를 갖고, 이는 첼로와 트럼본과 같이 낮은 레지스터 악기의 음질 해상도를 향상시키게 됩니다. 높은 주파수대역에선 더 높은 시간 해상도를 갖는데 이는 리듬의 미세한 타이밍을 복원하는데 이점이 있습니다.

3.2 Waveform Reconstruction from CQT representation using conditional wavenet

40개 층의 조건부 웨이브넷을 사용하고 입력 데이터는 CQT 와 파형을 쌍으로 해서 훈련시켰습니다. 웨이브넷을 오디오 샘플특성에 맞춰 재구축 했습니다.

Beam search  
타겟 CQT를 더 잘 일치시키기 위해 웨이브넷 생성시 빔 서치를 사용했습니다. 합성된 오디오 파형의 빔 검색은 다음 두 단계를 번갈아 수행합니다.  
1) n 개의 단계( n=2048) 에 대한 기존의 각 후보 파형에 대한 자동회귀 웨이브넷 을 수행  
2) 웨이브폼의 CQT 스펙트로그램과 타겟 CQT 스펙트로그램 사이 제곱오차가 큰 경우 해당 파형을 가지치기

Reverse generation  
타악기의 경우 처음 음이 갑자기 높았다가 음이 점점 낮아지는데 이러한 부분을 모델을 통해 생성하기에는 어려움이 있습니다. 따라서 거꾸로 뒷부분부터 생성을 시작해서 이러한 문제를 해결했습니다.

4\. Timbre transfer with CycleGAN on CQT representation

CycleGAN을 그대로 사용한 것은 아니고 checkeboard artifacts를 제거하기 위해 deconvolution 연산을 최근접 이웃 보간법과 regular convolution으로 대체했습니다. 이미지에서와는 달리 full 스펙트로그램 구분자를 사용, 초기에는 full 스펙트로그램 구분자가 강력해서 반대에 있는 생성자가 자라나지 못하고 죽어버리는 문제가 있어 GP(Gradient Penalty)를 적용해서 균형을 맞추려 했습니다. 또한 Identity 로스를 적용해서 원래 CycleGAN의 색 구성을 유지하려 했습니다.

항상 느끼는 것이지만 세상에 없던 새로운 것을 만들어내는 것이 아니라  
기존 모델을 잘 분석해서 어떤 부분으로 바꾸어볼 것인지 아이디어를 내는게 중요한 것 같습니다. 여러 실험을 통해서 원본 악기소리와 유사하면서도 또 다른 소리를 만들어내는게 신기합니다.
