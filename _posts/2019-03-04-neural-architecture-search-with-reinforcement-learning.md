---
title: Neural Architecture Search with Reinforcement Learning
categories:
  - ML
tags:
  - slowpaper
  - nas  
  - rnn
  - reinforce
last_modified_at: 2019-03-04T22:00:00-00:00
---

# slow paper
=====================================================================

강화학습을 활용해서 최적의 모델 아키텍쳐를 찾아보자!

뉴럴네트워크의 성공에도 불구하고 모델의 디자인을 설계하는 일은 어려운일입니다. 이 논문에서는 RNN 네트워크를 사용해서 모델의 파라미터 또는 구조를 생성하고 강화학습을 통해 기대 정확도를 최대로 만드는 아키텍쳐를 생성합니다.

우선 강화학습은 기계학습의 한 영역으로 행동심리학에서 영감을 받았습니다. 어떤 환경 안에서 정의된 에이전트가 현재의 상태를 인식하여, 선택 가능한 행동들 중 보상을 최대화하는 행동 혹은 행동 순서를 선택하는 방법입니다.

**Introduction**  
딥뉴럴 네트워크의 성공으로 feature 디자인에서 architecture 디자인으로 패러다임이 변화하고 있습니다. 하지만 아키텍쳐 디자인은 전문지식이 필요하고 탐색하는데 많은 시간이 소요됩니다.

figure1. [Barret Zoph](https://arxiv.org/search/cs?searchtype=author&query=Zoph%2C+B), “Neural Architecture Search with Reinforcement Learning”

하이퍼파라미터의 특성들 예를들면, 필터의 갯수, 스트라이드의 길이/ 너비로 한정지었기 때문에 문자열을 생성하는 RNN을 컨트롤러로 사용하였습니다. NAS의 큰 흐름은 아래와 같습니다.

1) 컨트롤러에서 p의 확률로 A라는 샘플 아키텍쳐를 생성하면  
2) 자식 네트워크에서는 A 아키텍쳐를 훈련시켜 정확도를 구합니다.  
3) 정확도를 리워드 신호로 사용. policy gradient를 계산해서 컨트롤러를 업데이트 시키는 과정을 반복합니다.  
4) 과정이 반복되면 더 높은 확률로 높은 정확도를 보이는 아키텍쳐를 찾을수 있습니다.

NAS는 실험을 통해 이미지인식(CIFAR-10)과, 자연어처리 모델링(Penn Treebank)에서 좋은 성과를 보였습니다.

**Related work  
**하이퍼파라미터 최적화 연구는 네트워크의 구조와 연결성을 지정하는 가변 길이를 생성하는데 한계가 있었습니다. 현대 신경 진화 알고리즘 연구는 search-based 방식이다보니 탐색속도가 느리고 잘 작동하기 위해선 휴리스틱방법이 요구되었습니다.  
컨트롤러에서 뉴럴 아키텍쳐 탐색은 이전 예측값들을 조건값으로 하이퍼파라미터를 한 번에 하나씩 예측하는 auto-regressive 한 구조입니다.

**Methods**  
이 논문의 핵심은 skip connection을 구성하여 모델의 복잡도를 높인 것과 파라미터 서버 접근방식을 사용해서 훈련 속도를 높인 부분입니다.

**3.1 Generate Model with a Controller Recurrent Neural Network**

figure2. [Barret Zoph](https://arxiv.org/search/cs?searchtype=author&query=Zoph%2C+B), “Neural Architecture Search with Reinforcement Learning”

컨트롤러를 통해 CNN모델에서 사용하는 하이퍼파라미터들을 생성합니다.  
한개 레이어에서 사용할 필터, 스트라이드 값을 예측하고 이를 반복합니다. 하이퍼파라미터 예측시에 softmax classifier 를 거친값이 다음 스텝의 인풋값으로 들어갑니다. 컨트롤러 RNN이 아키텍쳐를 생성하면 생성된 아키텍쳐의 뉴럴 네트워크를 짓고 훈련을 시킵니다. 미리 확보해둔 검증 데이터셋으로 네트워크의 정확도를 측정합니다. 컨트롤러 RNN의 파라미터 세타c 는 정확도의 기대값을 최대화하기 위해 최적화 됩니다.

**3.2 Training with Reinforce**

eq(1)

컨트롤러의 토큰 리스트 a\_1:T 는 자식 네트워크 구조로 디자인됩니다(이때 T는 파라미터의 수 \* 레이어의 수) 자식 네트워크는 정확도를 출력하고 이 정확도를 강화학습의 리워드 R로 사용해서 컨트롤러를 강화학습 훈련시킵니다. 여기서 리워드 R이 미분 불가능한 이유에 대해서 생각해보면, RNN Controller의 매 시점마다 softmax classifier를 통해 ‘자식 네트워크의 하이퍼파라미터’들이 나오는데, 이는 argmax로 정해져서 나온 값이라 검증데이터 셋의 정확도 계산에는 직접적으로 명시가 되지 않기 때문입니다. 또한 정확도 계산은 매우 높은 분산을 가지므로 분산을 낮추기위해 이전 아키텍쳐 정확도 지수이동평균값인 b를 baseline으로 사용해서 보완합니다.

**Accelerate Training with Parallelism and Asynchronous Updates  
**앞서 말한 자식 네트워크는 우리가 일반적으로 이야기하는 하나의 모델을 뜻합니다. 따라서 여러 컨트롤러 \* 여러 자식 네트워크는 수 많은 네트워크를 나타내고(13,000~ 15,000 개) 훈련속도를 높이기 위해 고안된 방법이 파라미터-서버 구조입니다.

figure3. Distributed training for Neural Architecture Search

S개의 파라미터 서버가 있고 이 서버와 연결된 K개의 복제된 컨트롤러에 공유된 파라미터 값이 저장됩니다. 각각의 컨트롤러는 다시 m개의 자식 네트워크를 복제해서 병렬로 훈련을 시킵니다. 자식 네트워크의 정확도는 파라미터 서버에 보낼 세타c 에 대한 그래디언트들을 계산하기 위해 기록됩니다. 구글에서 800 GPU로 2 -3주 시간이 소요되었다 하니 모델 전체의 크기가 엄청나게 큰 것을 짐작할 수 있습니다.

**3.3 Increase Architecture Complexity with Skip Connection and Other Layer Types**  
여러개의 레이어층이 쌓이면 소실문제가 발생하기 때문에 skip connection을 추가해서 탐색 공간을 넓히려고 합니다. 레이어 마다 앵커포인트를 더해서 이전 레이어들 중 어떤 레이어를 현재의 레이어 입력값으로 할지를 결정합니다.

**3.4 Generate Recurrent Cell Architectures  
**지금까지 Neural Architecture 탐색을 CNN에 적용해보았다면 아래에선 RNN에 적용하려고 해보았습니다. 기본적인 RNN Cell 계산식입니다  
h\_t = tanh(W\_1 \* x\_t + W\_2 \* h\_t-1)

RNN 그리고 LSTM은 입력값으로 x\_t와 h\_t-1 을, 출력값으로 h\_t를 갖는 트리구조로 나타낼 수 있습니다. RNN 컨트롤러에서는 트리노드들의 결합방식(addition, elementwise multiplication) 과 활성화함수(tanh, sigmoid)를 선택할 수 있습니다.

figure5.

그림의 좌측부터 (a),(b),(c ) 라 할 때 그림 (b)의 Tree index 0 은 add와 tanh를 결합한 것이고 Tree index 1은 elementwise multiplication 과 ReLu 를 결합한 것입니다. 그림 (c )에서도 해당 부분을 볼수있습니다. 그림 (b)의 Cell indices 의 왼쪽 1부분이 의미하는 것은 다음 메모리구조 C\_t와 연결되는것은 Tree index 1 이며 오른쪽 0부분은 h\_t 를 구할때 사용되는 것이 Tree index 0 이라는 것입니다. 그림 (b)의 Tree index 2 는 Tree0과 Tree1의 결합방식을 나타내는 것으로 그림에선 elementwise multiplication와 sigmoid의 결합이 됩니다.

**Conclusion**

기존 sota 모델과 비교했을 때 약간의 성능감소가 있지만 더 작은 파라미터 사용만으로 구현이 되었고 RNN Cell의 경우 오히려 더 좋은 성능을 보였다고 합니다. language 모델링에서 만들어진 RNN cell 은 API로 추가되어 tf.contrib.rnn.NASCell 로 사용할 수 있습니다.

[

tf.contrib.rnn.NASCell | TensorFlow
-----------------------------------

### CheckpointableObjectGraph.CheckpointableObject.SlotVariableReference

www.tensorflow.org

](https://www.tensorflow.org/api_docs/python/tf/contrib/rnn/NASCell)

아래는 mnist 데이터셋에 위 논문의 모델을 사용해 CNN 모델 구조를 찾아본 블로그와 github 사이트입니다.

[https://lab.wallarm.com/the-first-step-by-step-guide-for-implementing-neural-architecture-search-with-reinforcement-99ade71b3d28](https://lab.wallarm.com/the-first-step-by-step-guide-for-implementing-neural-architecture-search-with-reinforcement-99ade71b3d28)  
[https://github.com/wallarm/nascell-automl](https://github.com/wallarm/nascell-automl)
