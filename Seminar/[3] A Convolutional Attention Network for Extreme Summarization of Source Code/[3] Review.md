# A Convolutional Attention Network for Extreme Summarization of Source Code
- Miltiadis allamanis, Hao Peng and Charles Sutton, ICML 2016, [[pdf]](http://proceedings.mlr.press/v48/allamanis16.pdf)

- Attention 모델
  - 새로 만들어지는 output을 input의 어떤 부분을 집중하여 만들 것인지 판단.
  - 각 input마다 attention weight을 줌으로써 새로 생성되는 output이 input의 어떤 부분을 중요하게 생각했는지를 부여함.
  - 요약문 생성, 번역 등의 task에서 활발히 쓰이고 있다.
  
- Attention 문제
  - attention이 특정 부분에 가중치가 쏠림으로써, 그 부분에 연관된 것만 계속해서 생산해낸다.
  - OOV(Out-Of-Vocabulary, input corpus에서 자주 등장하는 단어의 set인 vocabulary에 없는 단어) 처리를 [UNK]로 하는 문제.
  - 이 논문에서는 두 번째 문제에 대한 해결을 다룬다.
  
- Translation-invariante features
  - 요약, 번역 등의 task에서 input의 중요한 부분은 다른 단어로 대체되는 것이 아니라 **그 단어 자체를 output으로 내도록 해야함.**
  - OOV 또한 의미를 모르는 단어이므로, input 단어 그대로 output으로 내도록하면 처리할 수 있음.
  - 기존 attention 모델은 이러한 작업을 잘 못함.
  
- 논문의 task
  - 주어진 source code 내용에 대한 method name(함수 이름)을 예측.
  - 요약 task의 OOV words에 대한 처리도 가능하도록 함.
  
- 제안 모델
  - Attention mechanism : 기존 attention 모델.
  - **Copy mechansim** : input의 단어를 ouput으로 그대로 복사.
  - Meta mechanism : attention과 copy의 비율을 조절. (그냥 이름만 그럴싸함)
  
- Copy Convolutional Attentional Model
  
  ![image](https://user-images.githubusercontent.com/26705935/42079717-5e73e5cc-7bbb-11e8-9bdb-b97c791df81d.png)
 
  - CNN으로 attention weight 계산 + GRU(Gated Recurrent Unit)로 output 생성.
  - 1, 2, 3 layer : attention feature 계산
  
    ![image](https://user-images.githubusercontent.com/26705935/42080939-18e16b52-7bbf-11e8-8fd0-97d91db1c5e4.png)

    - 1st layer : input 전체 c를 padding + convolution (Kl1)
    - 2nd layer : Convolution + h(t-1)과 element-wise multiplication : 이전 정보
    - 3rd layer : feature layer : L2를 normalized.
    
- 식

  ![image](https://user-images.githubusercontent.com/26705935/42081303-0bc73d1a-7bc0-11e8-8055-b83c3794e0d8.png)
  - attention weight alpha 계산
    - Katt와 convolution 및 softmax로 attention weight인 alpha 계산.
  
  - copy weight k 계산
    - Kcopy와 convolution 및 softmax로 copy weight인 k 계산. k도 일종의 attention weight임.
  
  - lambda : meta-attention
    - attention과 copy 사이 비율을 조절. 이것도 Klambda에 의해 학습되는 parameter.
    - Pos2Voc : c(input)에서 단어를 copy하는 것.
    - ToMap : V(vocabulary)에서 단어를 가져오는 것.
    - lambda가 1에 가까우면 copy를 하겠다. 0에 가까우면 attention으로 예측을 하겠다.
    
- Objective function
  
  ![image](https://user-images.githubusercontent.com/26705935/42081592-cfb4b5d6-7bc0-11e8-9f4a-1e5eefd37c5e.png)

  - mt : target 단어, ht-1 : 이전 hidden state 값, c : input 전체
  - I : binary vector
    - target 단어 mt가 input 중 어딘가(ci)에 있으면 1, 아니면 0
  - u : hyperparameter
    - simple attention model의 결과가 UNK이고 target 단어 mt가 input에 있으면 매우 작은 값 exp(-10), 아니면 1.
    - Copy를 용이하게 학습하기 위함.
  - 즉, input이 proper noun(고유명사, OOV 등)이면 k(copy)의 확률만 보고, 아니면 alpha+k(attention + copy)의 확률을 봄.
  
- 실험
  - 데이터 : Github 오픈 소스의 11개의 java project source code.
  - Measure : F1 score
    - F1 score : classification에서 사용되는 measure. precision과 recall로 계산.
  - 비교 : tf-idf 모델, biRNN attention 모델
  
- 결과
  
  ![image](https://user-images.githubusercontent.com/26705935/42082116-249f820a-7bc2-11e8-8f28-09ca72a704ae.png)
  
  - conv-attention : attention을 CNN으로 계산 + copy가 없음.
  - 제안 모델인 copy-attention이 제일 좋았다.
  
  ![image](https://user-images.githubusercontent.com/26705935/42082190-4cbc1ab4-7bc2-11e8-8dcb-d96cffcef257.png)
  
  - use, browser, cache는 OOV : lambda가 1에 가까워지고, 오로지 k만을 보고, 진한 부분에서 copy를 함으로써 output을 얻어낸다.
  - set, end는 vocabulary 단어 : lambda가 0에 가까워지고, alpha와 k 모두에서 봄으로써 output을 얻어낸다.
  
- 결론 및 토의
  - OOV 혹은 input의 중요한 부분을 **copy 하는 기능**을 attention에 추가 함으로써, OOV에 대한 처리 해결.
    - **번역 task는 copy가 의미 없을텐데, OOV는 어떻게 처리하는가? 아니면 의미가 있나?**
    
  - OOV를 다룬 다른 2017년 논문이 있다.
    - Attention weight 분포 + OOV 두 문제 모두 다룸.
    - Copy라는 방법은 유사하지만, 이 논문은 모든 단어에 대해 alpha(attention) + k(copy) 를 고려하여 output을 냄.
    - 논문 제목은 추후에 업로드 예정.
