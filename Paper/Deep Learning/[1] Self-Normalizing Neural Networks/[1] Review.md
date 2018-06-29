# Self-Normalizing Neural Networks
 - 저자 : Günter Klambauer, Thomas Unterthiner, Andreas Mayr, Sepp Hochreiter
 - 학회 : NIPS 2017
 - 논문 링크 : [[pdf]](http://papers.nips.cc/paper/6698-self-normalizing-neural-networks.pdf)
 - 논문 리뷰에 앞서, 이 논문은 여러 이론을 제시하는데 각 이론의 증명을 담은 appendix가 100여장에 달한다. 리뷰에서는 논문에 기재되어있는 간단한 증명만을 다룬다. 자세한 증명은 [appendix](http://papers.nips.cc/paper/6698-self-normalizing-neural-networks-supplemental.zip)를 참고하면 될 것이다.
 
## 1. Introduction
 - 딥 러닝은 여러 벤치마크에서 새로운 기록을 세우고, 다양한 상업용 application을 개발하는데 도움을 주었다. Vision의 분야에선 CNN이, speech 혹은 NLP 분야에서는 RNN이 뛰어난 성능을 보이는 반면, hidden layer가 많은 deep한 feed-forward neural network(이하 FNN)의 성공 사례는 찾아보기 힘들다. 특히 vision이나 sequential한 task와 관련 없는 challenge에서조차도 gradient boosting, random forest 혹은 SVM등이 우세하고, FNN의 성공적인 사례는 드물다.
 
 - Hidden layer의 수가 많은 deep한 neural network를 학습시키기 위해서는 normalization 기법이 필수적이다. 하지만 저자들은 normalization technique을 적용한 학습은 불안정하다고 말한다. 특히 RNN과 CNN의 경우에는 weight를 공유하는 특성을 가지고 있는 반면, FNN은 perturbation(섭동)에 민감하기 때문에 training error의 variance가 크게 나타나게 된다. 저자들은 deep한 FNN의 성공을 하지 못하는 가장 큰 요인은 결국 이 sensitivity to perturbations라고 주장한다.
 
 - 저자들은 이 perturbation에 robust한 FNN 모델을 제시한다. Self-normalizing neural networks(이하 SNN)은 normalization 기법을 적용한 FNN 모델에 비해 training error의 variance가 낮으며, neuron의 activation을 zero mean과 unit variance로 normalize한다. SNN은 변형된 activation function인 Scaled exponential linear units, 즉 SELU를 통해 이러한 기능을 수행합니다.

## 2. Internal Covariance Shift
 - NN의 hidden layer 수가 높아지면, 즉 deep해지면 Internal Covariance Shift라는 문제가 발생한다. 간단히 말하면 앞의 layer에서의 사소한 변화에 의해 뒤의 layer에서 큰 변화로 이루어질 수 있다는 문제이다. 이에 따라 결과의 분포가 전체적으로 shift될 가능성이 높아지고, 결국 학습을 진행하면 할수록 원하는 값이 아니라 shift된 값을 학습하는 꼴로 나타난다. (아래 그림 참고)
 
   ![image](https://user-images.githubusercontent.com/26705935/41337541-7115dc14-6f2a-11e8-98c0-b1cc011f940f.png)
 
 - 문제점을 해결하기 위해서 각 layer의 output(activation 값들)의 mean과 variance를 조절하는 normalization 기법을 사용한다. 대표적인 normalization 기법인 batch normalization은 layer 사이 사이에 각 layer마다 최종 계산된 activation값들의 분포를 normalize해주는 BN layer를 추가한다. (아래 그림 참고)
 
   ![image](https://user-images.githubusercontent.com/26705935/41337568-8a401808-6f2a-11e8-8b3b-e7452a881c8c.png)
 
 - 하지만 저자들은 이러한 batch normalization을 포함한 다양한 normalization 기법을 사용한 모델을 학습할 때, 불안정한 결과가 나타남에 따라 error의 variance가 높아진다고 주장한다.
 
 ## 3. Notation
 - 저자들이 제시한 SNN 모델을 소개하기 앞서, 논문에서 사용된 notation을 간략히 정리한다. 
  
     ![image](https://user-images.githubusercontent.com/26705935/41337747-fdc05608-6f2a-11e8-9b02-986cf0a65603.png)
  
   - Lower layer로 들어오는 각 activations vector를 x, weight matrix를 W, weight에 의해서 계산된 값을 z, 다음 layer로 들어가는 activation 값을 y라고 표현한다.
   - x와 y의 mean 과 variance는 u, v, 그리고 u~ 및 v~ 이며, weight vector들의 mean과 variance는 오메가와 taw로 표시한다. 
   - g는 low layer의 activations 값들의 분포를 high layer의 activations 값들의 분포로 mapping하는 함수이다.
    
## 4. Self-normalizing Neural Network
 - SNN은 mapping 함수 g의 개념을 통해 이루어진다. mapping 함수 g는 mean과 variance의 범위인 Omega에서 Omega로의 함수이고, 저자들은 stable 하고 attracting한 fixed point를 가지고 있다고 말한다. Fixed point는 (u, v)를 말하는데, g에 따라 mapping된 결과 또한 fixed point가 되는 경우 stable하다고 표현한다.
 
### (1) Scaled Exponential Linear Uints (SELUs)
 - SNN을 구성하는 첫 번째 요소는 activation function인 SELUs이다. ELU activation function에 lambda라는 parameter를 곱한 형태이다.
 
   ![image](https://user-images.githubusercontent.com/26705935/41338024-de582682-6f2b-11e8-85e3-e09ab250bb81.png)
 
 - (참고) 저자들은 Self-normalization을 하기 위한 activation function의 조건으로 다음과 같이 4가지 특징을 말한다.
 
   ![image](https://user-images.githubusercontent.com/26705935/41338011-d58119c4-6f2b-11e8-8ba5-8aef1e7cdc4a.png)

### (2) Weight initialization
 - 두 번째 요소는 weight initialization 이다. 초기화된 weight는 mean 0와 variance 1을 갖는 정규화된 값이여야 한다. Weight의 분포는 학습에 따라서 달라질 수 있는데, 저자들은 normalized 되지 않은 weight에 대해서도 self-normalizing한다는 것을 증명하였다. 이 내용은 후에 다룬다.
 
## 5. Deriving Mapping Function g
 - 다음은 Mapping함수 g의 식을 유도하는 과정이다. 먼저, low layer의 activation 값들인 xi는 서로 모두 independent하다고 가정한다.
 
   ![image](https://user-images.githubusercontent.com/26705935/41338301-bb90e2d2-6f2c-11e8-8143-e81b86b0f2b5.png)
 
 - Z는 x와 weight를 곱한 값인데, x 값들이 모두 independent하기 때문에 z 또한 서로 independent하다. 또한 central limit theorem에 의해서 independent한 x에 의해 만들어지는 z들은, n이 크면 클수록 그 분포가 정규 분포를 따르게 된다. Input의 수인 n은 일반적으로 크기 때문에 z는 정규 분포를 이룬다고 볼 수 있고, 이에 따라 high layer의 activation들의 분포를 나타내는 u~ 와 v~ 는 다음과 같이 적분식을 통해 계산된다.
 
   ![image](https://user-images.githubusercontent.com/26705935/41338335-d84e2286-6f2c-11e8-9a23-7807d5d876fc.png)

 - 이를 쭉 풀면 mapping 함수 g는 다음과 같다.
 
   ![image](https://user-images.githubusercontent.com/26705935/41338373-f44db4b0-6f2c-11e8-980a-4d9f81f11c34.png)
   
## 6. Normalized and unnormalized weights
 - 만일 weight가 normalize 되어있는 경우, 즉 w가 0이고 t가 1일 때, activation의 분포가 fixed point에서 fixed point로 수렴하기 때문에 다음과 같은 값을 얻을 수 있다.
 
   ![image](https://user-images.githubusercontent.com/26705935/41338517-5e26f6e4-6f2d-11e8-89d0-3c3b80511943.png)
   
 - 여기서 (0,1)이 fixed point 혹은 제한된 구역으로 수렴하는지, 즉 stable 하고 attracting 한지를 저자들은 증명하였다. mapping 함수 g의 자코비안 행렬, 즉 미분 값의 norm 값이 1보다 작으면 이는 수축 mapping이고 fixed point (0,1)은 stable하다는 것을 이용하여 증명하였다. 따라서 normalize 된 weight를 가지고 있는 경우, 다음 그럼과 같이 alpha와 lambda 값을 통해서 fixed point로 수렴한다.
   
   ![image](https://user-images.githubusercontent.com/26705935/41338633-baad9fd0-6f2d-11e8-876a-3de0d22fd599.png)
   
 - 학습이 진행됨에 따라 초기 normalized weight는 그 분포가 달라질 수 있다. 저자들은 unnormalized weight에 대해서도 (u,v)가 어떤 구간 내로 수렴한다는 것을 증명하였다. (자세한 증명은 appendix 참고) 따라서 normalize 되지 않은 weight에 대해서도 mapping 함수 g에 의해 특정 범위로 수렴한다.
 
## 7. Properties of SELUs and Initialization
 - SELU의 특성은 다음과 같다. Layer input의 variance가 높은 경우 이를 감쇠시키고, variance가 낮은 경우 이를 증가시킨다. 이를 통해 SNN의 각 layer의 output값의 variance는 범위가 제한된다. Variance가 가정한 범위 밖에 있는 경우에도 이를 감소 및 증가시킨다는 것을 저자들은 증명하였다.
 
 - SNN은 weight initialization이 매우 중요한데, 각 weight는 mean 0와 variance 1/n을 이루는 분포로 초기화되어 있어야한다.
 
 - 논문에는 dropout technique을 SNN에 적용시키는 방안도 언급되었으나(alpha-dropout), 이는 중요하지 않은 부분이기 때문에 skip한다.
 
## 8. Experiment
 - 성능을 위한 실험은 총 3가지로 진행되었으며, 기존의 batch, layer, weight normalization 기법을 사용한 FNN과 성능을 비교함으로써 제안 모델의 성능을 평가하였다.
 
### (1) 121 UCI Machine Learning Repository datasets
 - 다양한 응용 분야의 데이터들에 대해 121개의 classification task dataset을 이용한 실험이다. 결과는 다음과 같다.
 
   ![image](https://user-images.githubusercontent.com/26705935/41339082-f11674f6-6f2e-11e8-95d1-340e16735429.png)
 
 - 각각의 FNN들과 SNN의 결과를 순위로 매겼을 때, 평균 순위인 rank 4를 기준으로 차이 값을 나타낸다. Normalization 기법을 이용한 FNN들에 비해 SNN이 더 좋은 성능을 나타내고 있음을 알 수 있다.
 
 - FNN 이에외도 다른 머신 러닝 기법들과 성능 비교한 결과, dataset의 크기가 작은 경우에는 SVM, random forest가 더 높은 성능을 띄지만, dataset의 크기가 큰 경우에는 SNN이 더 좋은 성능을 나타낸다. 모델의 hyperparameter는 validation set을 기반으로 optimization 하였는데, SNN 모델의 평균 depth가 10.8로, 나머지 normalization 기법을 이용한 FNN 모델의 depth보다 높은 값을 갖는다. 즉, SNN은 더 deep한 모델에서 더 우수한 성능을 나타낸다.
 
### (2) Drug discovery : The Tox21 challenge dataset
 - 12,000개의 화합물 데이터에 대해서 화학 구조를 기반으로, 이 물질이 21개의 독성 중 어느 독성을 띄는 지에 대한 task이다. US NIH에서 선보인 모델인 ensemble of shallow ReLU FNNs은 AUC 0.846을 기록하였고, SNN은 AUC 0.845 ± 0.003 을 나타내며 이에 준하는 성능을 보였다.
 
   ![image](https://user-images.githubusercontent.com/26705935/41339398-af4ccc5e-6f2f-11e8-84c1-2ba1405c1659.png)
   
 - 특히 결과 표를 보면 layer의 개수가 높아질수록(ResNet의 경우, block의 수가 높을수록) 다른 모델들은 성능이 점점 낮아지는데 반해, SNN은 성능이 대체적으로 증가하였고, 8 layer에서 가장 최고의 성능을 보였다.
 
### (3) Astronomy : Prediction of pulsars in the HTRU2 dataset
 - wave signal에서 어떤 plusar인지를 구별해내는 task이다. 마찬가지로 SNN의 AUC 값이 다른 FNN 혹은 머신 러닝 기법들보다 더 높게 나타났다.
 
   ![image](https://user-images.githubusercontent.com/26705935/41339542-0c5487a2-6f30-11e8-9211-1c09a35cb36c.png)
   
## 9. Conclusion
 - 결론적으로, SNN은 다음과 같이 4가지 요소들에 의해서 높은 성능을 나타낼 수 있다.
 
   ![image](https://user-images.githubusercontent.com/26705935/41339599-3656da1e-6f30-11e8-93b3-b50d1b2100b5.png)
   
 - SNN은 각 layer activation 값의 분포가 normalize 되고, activation function인 SELU의 특성에 따라서 gradient가 vanishing되거나 exploding되지 않는다. 또한 많은 hidden layer를 갖는 구조에서도 높은 성능을 나타낸다. 이는 다른 FNN 모델에 비해 더 deep한 구조를 적용할 수 있다는 특징을 보인다.
 
## 10. 고찰
 - 수학적인 수식이 많아서 어려운 논문이었지만, 차근차근 따라 읽으며 이해하니 어느 정도 개념을 잡을 수 있었다. (사실 증명 appendix는 못볼것 같다..)
 - 현재 deep learning 분야에서 CNN, RNN 등의 모델이 많이 사용되고 연구되었지만, feed-forward neural network에 대한 연구는 점점 줄어드는 추세였다. 저자들이 말한 대로 deep한 구조의 모델을 적용시키기 힘들다는 단점 때문일 것이다. 기존의 basic한 FNN의 deep structure에 대한 연구라 흥미로웠다.
