# On detecting adversarial perturbations

- Jan Hendrik Metzen, Tim Genewein, Volker Fischer, Bastian Bischoff, 2017년 2월, 118회 인용.

## Introduction
- 기존의 방어 기법들
  - Goodfellow : Adversarial training : training data에 adversarial image도 포함.
  - Zheng : Objective function 식을 통해 특정 clean image와 이에 해당하는 adversarial image의 output이 비슷해지도록 학습함.
  - Papernot : Defensive distillation : FGSM과 L-BFGS에서는 잘 되나, C&W에서 안됨.
  
## Proposed Method
- DeepNN에 detector subnetwork를 추가. 이를 학습하여 주어진 input에 대한 adversarial detection 수행.

![image](https://user-images.githubusercontent.com/26705935/47154382-2569c080-d31d-11e8-837b-fad706e2285f.png)

  - Inputs from intermediate feature representations of the model.
  - Classification network + Detector.
  
- 단계
  1. 먼저 classification network를 clean image에 대해 학습.
  2. 동시에 각 data에 대한 adversarial image 생성 -> 후에 dynamic하게 생성함.
  3. Origin image : 0, adversarial image : 1 label의 output, classification network의 weight을 freeze하고 detector만 학습.
  
- 질문
  - **Gradient-based attack 기법들은 classification network 뿐만 아니라 detector에 대해서도 gradient를 계산함으로써, 속일 수 있다.**
  - Classification model과 detector를 동시에 속이는 공격 기법이 dynamic adversary 공격. 
    - FGSM과 비슷한 수식에, (1-sigma)Jcls + (sigma)Jdet
  - 이에 대한 defense를 제안.
    - **dynamic adversary training : 즉각적으로 adversarial이면 확률 0.5, 0.5로 맞춘다? 로 adversarial image를 생성해서, 이를 training.**

## 실험 1. CIFAR10
- Classification Model 학습
  - 32-layer ResNet
  - 45000 training data + 100 epochs + SGD with momentum(=0.9), learning rate=0.1(after 41 epochs: 0.01, after 61 epochs: 0.001)
  - 5000 validation data : Clean accuracy : 91.3%
  - 이후 중간 단계의 feature를 input으로 받는 detector를 붙여서 학습함.
  
- Detector 학습 
  - Detector : conv->mp->conv->mp->conv->conv->gap
  - 20 epochs, 45000 training data, learning rate=0.0001 (classification model에 비해 많이 낮음. 의미 없을 수도.)
  - FGSM, Iterative FGSM 공격에 대한 epsilon : 1, 2, 3, 4
  
- 실험 결과

  ![image](https://user-images.githubusercontent.com/26705935/47154826-57c7ed80-d31e-11e8-8395-bb349e85725f.png)
  
  - 예측 성능이 낮은(강한) 공격에 대해서는 detect가 잘 됨.
    - 예측 성능 30% 이하인 공격에 대해 80%, 10% 공격에 대해 90%의 detection.
    - 기존 공격 기법들의 강한 정도 : FGSM < Iterative FGSM < DeepFool
    
  - Detector의 위치에 따라 2<1<3<0<4 (classification model의 layer) 순으로 detect 성능이 좋음.
    - FGSM, Iterative FGSM에 대해서는 2가 제일 좋고, DeepFool은 4가 제일 좋음.
    - 공격 방법마다 붙여야 되는 것 자체가 다르니까, 모두에 대한 공격에서 좋은 건 아니라는 것.
    - 성능이 가장 낮았던 case는 0.77정도. (향후 연구에서 이거보단 좋아야 할 듯.)
    
- epsilon(hyperparameter) setting 실험 및 결과

  ![image](https://user-images.githubusercontent.com/26705935/47155121-0d933c00-d31f-11e8-9bff-a4a8045fbaea.png)
  
  - 큰 epsilon의 공격에 의해 생성된 adversarial example로 학습하고, 작은 epsilon에 의한 adversarial example로 test하는 경우는 성능이 낮음.
	- 공격의 epsilon은 크면 클수록 classification model 성능이 떨어져서 detect가 더 잘 됨.
	- 따라서 classification model 기준으로 30% 아래의 accuracy를 만족 + 가장 작은 epsilon을 선택하는 것이 좋음.

- A로 학습하고 B를 test하는 실험 및 결과 (A, B는 공격 방법)
  - Transferrability 측정 실험.
  
  ![image](https://user-images.githubusercontent.com/26705935/47155365-82ff0c80-d31f-11e8-84d5-3e879fabc093.png)
  
  - Iterative FGSM에 의힌 adversarial image로 학습하고, FGSM에 의한 adversarial image로 test 하는건 잘 되지만, 반대는 성능이 낮음.
	- 센 공격기법에 의한 adv example로 학습해야 함. (예를 들어 DeepFool)

## 실험 2. 10-class ImageNet
- Class를 10개로 줄인 이유
  - Computational power를 줄이기 + 너무 비슷한 class로 간단화 하는 adversary를 피하기 위함.
  
- Classification model
  - Pre-trained VGG16 + 10개 class만 선택하도록 하는 layer 추가.
  
- Detector
  - Classification model의 4번째 pooling layer에 붙임.
  - 3*3 196개의 feature maps. 데이터 수는 명시 X.
  - 500 epochs + Adam + lr=0.0001

- 결과

  ![image](https://user-images.githubusercontent.com/26705935/47155681-37992e00-d320-11e8-8723-2cdf18aeccff.png)
  
  - Detectability : 85%이상, Iterative(l2) 공격에 대한 detection이 아예 안됨.(0.5)
  
  ![image](https://user-images.githubusercontent.com/26705935/47155719-5697c000-d320-11e8-9646-da2d564e90e9.png)
  
## Discussion
- Adversarial image와 clean image는 아주 작은 차이를 갖는데, 왜 이렇게 감지가 잘 되는가?
  - Detector는 특정 데이터가, 데이터 manifold 중심에서 인근 클래스 경계 방향으로 약간 벗어나는 입력을 감지한다.
  - Adversarial 이미지는 데이터 manifold 중심으로부터 특정 방향으로만 뻗어 있다.

## 결론
- Noise를 덜 규칙적으로 만드는 공격 방법을 만들어라.

## (생각해본) 단점
- CIFAR10, 10-class subset of ImageNet에 대해서만 test : class 수가 낮음.
- Imagenet의 Iterative l2 공격이면 detect 안 됨.
- 학습되는 이미지의 공격 방법, e에 따라서 결과가 다름. (의미 없을 수도 있음)
- (단점일려나) detector 또한 conv, pool의 CNN 형태 : output 직전 값으로 한 건 아님.
- CNN 학습에 필요한 데이터 수가 많을 것 : 학습을 위해 adversarial image 수가 많이 요구된다.


 
