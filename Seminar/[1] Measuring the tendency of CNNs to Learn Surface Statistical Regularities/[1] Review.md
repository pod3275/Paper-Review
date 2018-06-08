# [1] Measuring the tendency of CNNs to Learn Surface Statistical Regularities
 - ArXiv:1706.07068v1, Jason Jo, and Yoshua Bengio (Universit´e de Montr´eal), [pdf](https://arxiv.org/pdf/1711.11561.pdf)
 
 - CNN을 공격(제 기능을 하지 못하도록)하는 여러 방법들.
   - adversarial example : noise를 섞어서, 인간은 알아볼 수 있는데 기계학습 모델은 알아보지 못하도록 하는 것
   - universal example : 한 class를 대상으로 하는 adversarial example과는 달리, 전체 dataset에 대해 적용할 수 있는 noise 모델
   
 - noise를 섞는 공격 방법은, noise를 통해 classification의 경계를 이동하는 것이라고 분석한다.
 
 - 기존까진 CNN이 filter를 통해 high-level abstraction을 수행하여 어떤 특정 패턴을 인식하고 이를 통해 classification 등의 task를 수행하는 것이라고 생각되어었으나, 이렇게 되면 adversarial example에 공격당하는 것을 설명하지 못한다.
   - **따라서, 저자들은 CNN의 generalization이 training 및 test dataset에 공통적으로 있는 low level의 무언가(저자에 따르면, superficial cues)에 집중하고, 이를 통해 학습하여 성능을 나타내는 것이라고 말한다.**
   
 - 증명을 위한 실험 진행
   - mapping 함수 F : x(original data) -> x'(noise data)
   - F를 fourier filtering을 이용함 : wave를 여러 파장의 단위 신호의 합으로 나타냄. 신호의 분할.
   - radial masking : 가운데(low frequency) 부분을 제외한 나머지 부분(high frequency)을 날려버리기
   - uniformly random masking : p의 확률로 랜덤하게 살리거나 날려버리기
   
     ![image](https://user-images.githubusercontent.com/26705935/41142287-18d01068-6b30-11e8-8543-d293310a58fa.png)
   
   - 모델 : Preact ResNet 학습 및 인식
   
 - 결과
   - 학습을 어떤 dataset(original, radial, random data)로 하든, radial masking을 적용한 data에 대해 인식률이 너무 낮음.
   
     ![image](https://user-images.githubusercontent.com/26705935/41142258-0145ab6a-6b30-11e8-957f-6e3aebd943bf.png)
   
   - radial masking을 적용한 것 = 이미지에서 high frequency(중요하지 않은 부분)가 사라진 것
     - **중요하지 않다고 생각한 부분(high frequency)이 이미지 인식에 영향을 준다.**
  
 - Discussion
   - 이러한 현상은 CNN 구조상의 잘못인가 혹은 loss function의 잘못인가?
   - CNN의 filter의 크기가 매우 작아서(3*3) 쪼그만 픽셀도 다 feature로 받아들이는 것 아닐까?
