# Adversarial Examples Are Not Easily Detected: Bypassing Ten Detection Methods

- Nicholas Carlini, David Wagner, 2017.11, 140회 인용.
- 10개의 제안된 adversarial detection method 성능 확인 및 비교.
  - 모든 제안된 기법들이 새로운 loss function에 의한 adversarial attack에 의해 실패할 수 있다.
  - 기존에 얘기했던 것보다 detect는 훨씬 더 어려우며, 각 논문들에서 가정한 adversarial example들의 특성도 실제론 그렇지 않은 경우도 많음.
  
- 기존 10개의 detection method에 대한 공격을 시도.
  1. 주어진 모델에 적용된 defense method에 대한 고려 없이 그냥 classification 모델만 공격.
  2. 제안 공격 : 각 defense에 대한 맞춤형 공격. Adversary가 defense를 속이기 위해 필요한 것들을 캡쳐하도록 하는 attacker-loss function 정의.
  3. Transferrability : 방어 모델의 구조 및 파라미터를 몰라도 수행하도록(black-box attack 가능성)
  
  - 결국 white-box, black-box setting에서 10개의 defense method를 모두 break함.

- Attacker의 3가지 환경에서 detection method의 성능을 평가함.
  1. Zero-Knowledge : Detector의 존재를 아예 모름. C&W attack 사용.
	2. Perfect-Knowledge : Detector 존재 + parameter 접근 가능. Loss-function 선택 (제안)공격.
	3. Limit-Knowledge : Detector 존재 알지만 parameter 접근 불가. 2.의 transferability.
  
## Defense Method (F: classification model, D: detector)
### 1. Adversarial Retraining
(1) : Clean(N class)에 대해 Fbase 학습 -> F에 의해 adv img 생성 -> clean + (adv img, N+1) 데이터로 Fsecured 학습.

(2) : Fsecured 대신 이를 이용하여 새 binary model D를 학습.
- 학습 : MNIST
- 결과
	- Zero : (MNIST) (1),(2) : FGSM, JSMA : 100% 근방. 
	- C&W (1) 98.5%. (2) 98% 
	- (CIFAR10) (1),(2) : 70%.
	- Perfect : (2) : classifier + defender = G 정의, 이에 대한 adv 생성. (1),(2) : attack 성공 100%.
	- Limit : 대체 모델 R1, R2  R1으로 adv 생성  R2 평가. (1),(2) : attack 성공 98%.
- 단점
	- Image input. Input의 차원이 크므로 detector 학습이 오래 걸릴 것이다.

 	![image](https://user-images.githubusercontent.com/26705935/46799258-4a938780-cd8f-11e8-983c-b31e1fe160c5.png)
	- G Model : Classifier, detector 구분되어 있는 NN 모델에 사용됨
---
### 2. Examining Convolution Layers
- 각 conv layer의 feature들로 yes/no를 도출하는 detector 학습.
- Only CIFAR
- 결과
	- 실제 구현 시, detector 학습하는 게 hyperparameter에 매우 sensitive하다.
	- Zero : C&W, 81% positive rate, 28% false positive rate. 원 논문에서 말한 99%는 약한 공격에 대함이었다. 성능 좋지 않음.
	- Perfect : 위의 G 모델에 C&W 사용. 공격 성공.
	- Limited : 위의 transferability. 84%~98% (low and high confidence adv example) attack 가능.
---
### 3. PCA Detection
- PCA는 N dim을 k dim으로 linear transformation하는 방법.

(1) Input Image PCA
- 가정 : Adv image place a higher weight on the larger principal components than clean image.
- MNIST, C&W 공격으로 adv 만들어서 각 principal component의 분포를 봄.
- 원 논문 저자 말이 맞지만, 이건 MNIST가 border pixel이 대부분 0이어서 그럼. CIFAR 데이터에서는 차이가 없다.
- 결국 MNIST용임.

(2) Dimension Reduction
- N dim --> K dim으로 linear transformation하고, 이를 이용하여 FCNN training: first K component만을 보는게 attacker입장에서 제한이 될 수 있음.
- FCNN : 2 hidden layers of 100 units. 26개의 다른 model (K:9~784). 
- 결과(Zero) : 25 dimension 이하는 낮고, 25이상은 97% 이상의 detect accuracy.
- Perfect : 99.5% 공격 성공률.

(3) PCA 사용하여 더 낮은 차원의 PCA 공간으로 mapping하고, 이 값으로 classifier 학습.
- CNN. 25차원 이상에서 99% 이상의 정확도.

(4) Hidden Layer PCA
- Conv layer 다음에 PCA를 적용. 각 Ci는 i번째 conv에 PCA 적용 값으로 학습한 SVM 모델.
- ImageNet에는 되나, MNIST, CIFAR에는 안됨.
	- Zero
		- L-BFGS. 80% true positive rate, 0% false positive rate
		- C&W, MNIST : 62% true positive rate, 37% false positive rate. 92% false positive rate
		- C&W, CIFAR : 56%, 40%, 98% false positive rate
- C&W 공격에 대해서는 잘 안됨.
---
### 4. Distribution Detection
- clean과 adv의 분포를 이용하여 detect. Classical Statistical Method.

(1) Maximum Mean Discrepancy
- MMD : 분포를 감지. But 계산량이 많음.
	- a= MMD(X1, X2)  X1의 1개와 X2의 1개를 서로 shuffle 한 Y1, Y2 --> b= MMD(Y1, Y2)
	- a<b면 다른 distribution이다를 감지하여 shuffle을 reject한다.
- MMD가 정확히 걸러내는가를 보기 위해 X1, X2에 random하게 해서 실험 진행.
	- Zero : 반복. C&W로 100개만 되도 안됨. CIFAR도 안됨.
- 이 방법은 큰 수정이 있지 않는 이상 잘 안 된다.

(2) Kernel Density Estimation
- NN의 마지막 hidden layer output으로 GMM 모델을 사용.

	![image](https://user-images.githubusercontent.com/26705935/46954157-cf9dda00-d0ca-11e8-8bfb-8e86cf380cf7.png)

	- Xt: t label을 가지는 training set, Fn-1(x) : output of final hidden layer x.
	- Input x에 대해서 t label x들까지의 거리를 측정.
	- KDE(x)가 특정 threshold보다 작으면 adversarial, 크면 natural.

- last hidden layer output은 image의 high-level semantic 정보를 담는다. Original image보다 더 잘될 것.
- 결과
	- Zero : C&W, MNIST는 잘되는데 CIFAR는 안됨. (CIFAR에서는 80%이상의 adv가 KDE가 t보다 컸음)
	- Perfect : 새로운 loss function 만들어서 공격했더니 MNIST에서도 공격이 잘 된다.
	- Limited : 공격 성공.
---
### 5. Normalization Detection
(1) Dropout Randomization
- Bayesian Neural Network Uncertainty : NN의 uncertainty를 측정.
	- Randomization in Network의 가정: Random value(dropout)에 대해서도 clean이 성능이 좋아야 됨.

	![image](https://user-images.githubusercontent.com/26705935/46954257-1390df00-d0cb-11e8-8ebb-10aa8ed23693.png)
	
	- Input x를 L번 넣어서 계산. Fr은 dropout을 적용한 network. Fr이 항상 같은 결과면 U(x) = 0.
	- X에 대해서 판단하려면, threshold t를 정해서 L번 시행해서 U(x)가 t보다 크면 adv.
	
- 결국 x를 dropout하고 넣은 결과가 변동이 크면 adv, 아니면 clean이라고 판단.
- 결과
	- MNIST FGSM, JSMA : 96% (false positive rate 1% 이하)
	- Zero : C&W, MNIST : 75%, CIFAR : 95%
	- Perfect : 98% success 공격
	- Limited : MNIST 90%, CIFAR에 대해서는 not broken.
	
(2) Mean Blur
- 3*3으로 blur 해서 classifier에 넣기. Author는 “매우 간단하다” 라고 하고 있음.
- 결과
	- Zero : C&W에서 80%의 adv는 판단 못함. High-confidence adv ex에만 적용 가능.
	- Perfect : CNN에 3*3 conv 추가한 모델을 이용해서 공격. 잘 뚫림.

## 결론
- Dropout Randomization이 CIFAR에서 generation(역공격)하기 너무 어려웠다.
	- Randomization이 미래다.
- MNIST 특성이 CIFAR에도 있는 것은 아니다.
	- 특히 KDE는 M에 잘되고 C에 너무 안됨.
- Detection NN은 너무 충분히 공격 가능하다.
- Raw하게 pixel value를 갖고 하는 건 효과적이지 않다.

## 조언
- 강한 공격 가지고 실험하라. FGSM, JSMA는 하지 말아라.
- White-box attack이 실패함을 보여라.
- False positive, true positive rate을 보여라.
- MNIST 말고 다른 dataset도 해봐라.
- 소스 코드를 공개하라.
