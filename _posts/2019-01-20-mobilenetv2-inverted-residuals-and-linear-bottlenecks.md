---
title: MobileNetV2- Inverted Residuals and Linear Bottlenecks
categories:
  - ML
tags:
  - slowpaper
  - mobilenet
  - residuals
  - bottlenecks
last_modified_at: 2019-01-20T23:00:00-00:00
---

# slow paper
=====================================================================

신경망은 많은 분야에서 놀라운 성과를 가져왔지만 더 높은 정확도를 가져오기 위한 과정은 필연적으로 네트워크의 더 높은 계산비용을 가져왔습니다.  
MobileNetV2 에선 모바일 기기에서 사용가능하도록 메모리와 연산량을 감소시켰으나 기존 모델의 정확도를 유지하도록 하였습니다. 아래 본문은 MobileNetV2 를 이해하는데 필요한 주요개념에 대해 설명해보았습니다.  
해당 자료들을 참고하였음을 미리 알립니다.

[Mobile Vision Learning — Xecption and MobileNet  
](https://docs.google.com/presentation/d/1TMVFiC9RxE-f9kQPsU2KMAv5I0izFgDhTBtTtllhve8/edit#slide=id.p1)[Mobile Vision Learning — How cross channel pooling causes infomation loss in CNN?](https://docs.google.com/presentation/d/1wuq_ARC3HOCMTEOCifbqouqfRoaeBdaEYJobSDXzycE/edit#slide=id.p1)  
[MobileNetversion2](http://machinethink.net/blog/mobilenet-v2/)

#depthwise separable convolution  
계산복잡도와 사이즈를 줄이면서도 conv 층의 표현력(expressiveness) 을 유지하기 위해 고안된 방법입니다.  
depthwise convolution + point wise convolution 으로 분리할 수 있습니다.  
보통의 convolution 연산이 input(3\*3\*3) -> output(1) 으로 한번에 수행할때,  
depthwise convolution은 intput(3\*3 + 3\*3 + 3\*3) -> output(1 + 1 + 1)  
pointwise convolution은 input(1 + 1 + 1) -> output (1)  
나누어서 연산이 수행됩니다

Figure1. depthwise separable convolution ([https://tvm.ai/2017/08/22/Optimize-Deep-Learning-GPU-Operators-with-TVM-A-Depthwise-Convolution-Example.html](https://tvm.ai/2017/08/22/Optimize-Deep-Learning-GPU-Operators-with-TVM-A-Depthwise-Convolution-Example.html))

MobilenetV2에서 k=3 (3 \*3 depthwise seperable convolutions) 을 사용한다고 할 때 연산비용은 기존대비 8~9배 정도 감소됩니다! (약간의 정확도 감소)

그럼 어떻게 이런 방법이 가능한지에 대한 의문이 생깁니다  
Xception의 핵심가설 The mapping of cross-channels correlation and spatial correlation can be entirely decoupled 을 살펴보면

— cross-channels correlation : conv 층에 입력되는 이미지들의 유사한정도  
— spatial correlation : conv 필터와 입력 채널 사이의 상관도, 필터가 이미지의 특성을 잘 포착하는 정도

conv 층에 입력되는 이미지들의 유사한정도와 conv 필터가 이미지의 특성을 잘 포착하는 정도는 서로 상관이 없다는 것으로 결국, 입력 채널을 변형(압축)해서 넣어도 성능에는 크게 영향이 없다는 것으로 해석 될 수 있습니다.

#linear bottlenecks #inverted residuals  
입력 채널 정보를 activation space에 전부 담지 못하는 현상이 발생하는데 이는 Relu함수의 비선형에 의한 정보손실로 인해 발생합니다 (0이하의 값을 가진 경우 0으로 맵핑) 또한 연산량을 줄이기 위해서 1\*1 conv 로 과도하게 압축하다보니 역시 정보손실이 생깁니다.

1\*1\*L 을 conv filter 의 개수, M 을 linear transform W의 row 개수라고 할때,

M < L : 출력값이 필터보다 작은 channel pooling의 경우는 정보손실발생  
M ≥ L: 출력값이 필터보다 크거나 같은 channel expansion의 경우는  
정보가 보존됩니다

Figure2. Examples of ReLU transformations of low-dimensional manifolds embedded in higher-dimensional spaces ([https://arxiv.org/pdf/1801.04381.pdf](https://arxiv.org/pdf/1801.04381.pdf))

하지만 M < L 인 경우에 X의 manifold 차원수가 충분히 작으면 (X가 sparse) M<L의 1\*1 conv 필터를 가지고도 X의 정보를 보존할 수 있습니다.

input manifold 를 충분히 담지 못하는 space에서 relu함수를 수행하면 정보손실이 발생하지만 차원수가 충분히 큰 space에서 relu함수를 사용하면 정보가 손실될 가능성이 작습니다. 쉽게 설명하면, 저차원 데이터에 conv 층을 적용하면 많은 정보를 추출할 수 없습니다. 따라서 더 많은 데이터를 추출하기 위해 저차원의 압축된 데이터 압축을 먼저 풀고(확장) -> conv 층을 적용 -> projection 층을 통해 데이터를 다시 압축시킵니다

Figure3. inverted residuals ([http://machinethink.net/blog/mobilenet-v2/](http://machinethink.net/blog/mobilenet-v2/))

마지막 relu 함수를 지나기 전에 channel expansion하여 input manifold를 충분히 큰 space에 담아 놓고 relu함수를 사용하면 정보손실을 최소화 할 수 있습니다. 여기서 더 나가 linear bottleneck을 마지막에 두지말고 앞으로 가져와서 expansion -> projection 구조 & shortcut connection 을 사용합니다

모델의 크기가 커지면 파라미터의 수와 연산비용이 커지는 건 당연한 부분이지만 이러한 trade-off 관계를 깨고 모바일기기에서도 사용이 가능한 모델을 만들기 위해 연산비용을 효율화 시킬 포인트를 찾았습니다. 비슷한 성능을 내면서도 더 높은 효율성을 갖게 되었다는 데 의의가 있습니다.
