---
title: Show, Attend, and Tell with Pytorch
categories:
  - ML
tags:
  - image captioning
  - encoder
  - decoder
last_modified_at: 2018-12-29T09:00:00-00:00
---



소개 및 구현

2018년 초,  
[Show, Attend and Tell: Neural Image Caption Generation with Visual Attention](https://arxiv.org/abs/1502.03044) 논문을 읽고 Tensorflow 코드로 구현된 건 있었지만 Pytorch 코드는 없어서 잠시 구현을 해보려 했었다

CNN-RNN 을 연결한 Image Captioning 코드들은 찾을수있었는데, RNN에 Attention을 더하는 부분에서 멈추어있었다. 오랜 과제를 해결한다는 마음으로 이제는 참고할 코드와 조금은 나아진 머신러닝에 대한 지식으로 정리를 해보려한다

시작하기에 앞서  
image\_captioning 코드는 [이곳](https://github.com/yunjey/pytorch-tutorial/tree/master/tutorials/03-advanced/image_captioning)을  
Show, Attend and Tell 구현된 Pytorch 코드와 본문내용은 [이곳](https://github.com/sgrvinod/a-PyTorch-Tutorial-to-Image-Captioning)을 참고해서 정리했음을 밝힌다.

Image Captioning with Attention
-------------------------------

입력값으로 이미지가 주어졌을때 이미지를 변환해서 자연어 설명으로 변환하는 것을 Image Captioning 이라고 한다. 이미지 인코더 부분은 CNN 으로 디코더 부분은 RNN (LSTM) 으로 구성되어있다.  
위 논문이 이전 논문([Show and Tell: A Neural Image Caption Generator](https://arxiv.org/abs/1411.4555)) 과 차이나는 부분은 모델이 어디를 볼지를 스스로 학습한다는 것이다. 그러니깐 설명문의 한단어, 한단어를 만들때 이미지에서 강조되는 부분도 달라진다는 것이다. 이것이 가능하게 된건 Attention 메커니즘 덕분이다.

Fig1. captions generated examples ([https://github.com/sgrvinod/a-PyTorch-Tutorial-to-Image-Captioning](https://github.com/sgrvinod/a-PyTorch-Tutorial-to-Image-Captioning))

Attention 은 모델이 인코딩 부분에서 관련되어있다고 생각하는 특정 부분을 선택할 수 있는것으로 image captioning 부분에서 특정 pixel을 다른 pixel들보다 더 중요하게 고려할 수 있고, 기계번역과 같은 seq2seq 부분에선 특정 단어를 다른 단어보다 더 중요하게 고려할 수 있다

Encoder
-------

인코더는 입력된 3 color 채널의 이미지를 더 작은 이미지의 학습된 채널로 인코딩한다. 여기서 더 작은 인코딩된 이미지는 원본 이미지의 특징들만 잘 요약된 요약본이라고 볼수있다. 인코딩으로 사용되는 건 CNN 모델로, 이미 훈련된 모델 — 여기서는 ResNet-101 — 를 선택했다. 이미 훈련되있는 모델을 활용해서 자신의 task 에 맞게 튜닝해서 사용하면 시간을 아낄 수 있음은 물론 더좋은 성능을 기대할 수 있다.

Fig2. Encoder

원본 이미지(3, H, W)에서 인코더 모델을 통해 (2048, 14, 14) 로 인코딩 결과가 나온다. 논문에선 VGGnet을 사용했고 마지막 레이어가 linear 로 되어있어 수정이 필요하다

Decoder, Attention
------------------

디코더의 역할은 인코딩되어 온 이미지를 보고 단어단위로 이미지 설명문을 생성하는 것이다. 순서대로 작업되어야 하므로 RNN 모델이 사용되고 여기서는 LSTM 을 사용했다. Attention이 없는 경우에는 이미지 전체의 pixel을 단순평균해서 사용하지만 Attention이 있는 경우에는 디코더가 sequence의 각기 다른 지점에 따라 이미지의 다른부분을 바라보게 할 수 있다.

Fig3. Decoder

한 남자가 공을 들고 있다 는 문장에서 “공" 이라는 단어를 생성할 때 디코더가 사진에서 공부분을 바라보게 할 수 있다. 단순평균보다 이미지 전체의 가중된 평균치를 사용함으로써 ( 중요하다고 판단되는 부분을 강조) 더 나은 성능을 가져온다.

Fig4. model structure

인코더가 인코딩된 이미지를 생성하기 시작하면 LSTM 디코더에 넣기위해 인코딩을 초기 hidden state “h” 와 cell state “c”로 변환한다.  
디코더 단계마다,  
인코딩 이미지와 이전 hidden state 는 Attention Network에서 각 pixel별 가중치를 생성하기 위해 사용된다  
이전에 생성된 단어와 인코딩의 가중평균은 LSTM 디코더에 들어가 다음 단어를 생성한다.

Input to model
--------------

#이미지  
미리훈련된 인코더를 사용하기에 인코더 모델에 맞게 이미지를 가공할 필요가 있다. 픽셀 값은 0과 1 사이어야 해서 이미지를 아래 값을 사용해서 평균화하는 작업이 필요하다.

```
mean = \[0.485, 0.456, 0.406\]  
std = \[0.229, 0.224, 0.225\]
```

Pytorch는 NCHW 규격이므로 모델에 입력값으로 들어가는 이미지는  
Float 타입의 `N, 3, 256, 256` 차원형태이다. 여기서 N은 배치 사이즈를 나타낸다.

#캡션  
캡션 (인쇄물에 실린 삽화,사진의 설명문)은 출력물임 동시에 다음 단어를 생성하기위해 디코더에 들어가는 입력값이다.

`<start> a man holds a football <end>`

첫번째 단어를 생성하기 위해선 0번째 단어 <start>가 필요하고 디코더가 캡션의 마지막을 예측하고 디코딩을 중지하기 위해선 역시 <end>가 있어야한다. 또한 캡션의 길이는 각각 다르기 때문에 길이를 같게 만들기 위해선

`<start> a man holds a football <end> <pad> <pad> <pad>....`

<pad> 토큰이 필요하다. 코퍼스(단어뭉치)의 각 단어에 대한 색인 매핑(단어사전)인 vocab을 생성한다.

`9876 1 5 120 1 5406 9877 9878 9878 9878....`

캡션은 Int 타입의 `N, L` 차원형태이다. 여기서는 L은 패딩된 길이를 나타낸다.

\# 캡션길이  
시작과 마지막 토큰이 포함되므로 실제길이 + 2 가 된다  
캡션길이는 동적그래프를 생성하기 위해 중요하다. 동적그래프에 대한 내용은 아래 Loss Function 부분에서 추가적으로 설명하려고 한다. 캡션길이는 Int 타입의 `N` 차원 형태이다

Model Code
----------

Loss Function
-------------

연속된 단어를 생성하기 때문에 우리는 CrossEntropyLoss를 사용한다.  
전체 시퀀스를 생성하는 과정에서 모델이 모든 픽셀을 다룰 수 있기를 원한다. 따라서 모든 timestep 에서 픽셀 가중치의 합과 1사이의 차이를 최소화 하려고 한다. 우리는 padded된 부분의 loss 는 계산할 필요가 없다. pads 부분을 제거하는 가장 쉬운 방법은 `[pack_padded_sequence()](https://pytorch.org/docs/master/nn.html#torch.nn.utils.rnn.pack_padded_sequence)` 를 사용하는 것이다.

Fig5. description of pack\_padded\_sequence()

Conclusion
----------

어텐션 접근방식으로 MS COCO, Flickr8k and Flickr30k 데이터셋에서 SOTA를 달성했다. 학습된 어텐션 메커니즘은 모델의 생성과정을 좀 더 해석가능성이 있도록 하고, 학습된 alignments는 인간의 직감과 유사함을 설명했다.
