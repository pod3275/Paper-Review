# Unsupervised Machine Translation Using Monolingual Corpora Only
- 저자 : Guillaume Lample, Alexis Conneau, Ludovic Denoyer, Marc'Aurelio Ranzato
- 학회 : ICLR 2018
- 날짜 : 2018.04.13
- 논문 : [[pdf]](https://arxiv.org/pdf/1711.00043.pdf)

## 서론
- 현재까지 deep learning 분야의 많은 연구로 NMT(Neural Machine Translation)의 성능은 가면 갈수록 좋아지고 있다.
- 특히, attention 기법이 나타나면서부터 성능은 껑-충 뛰었다. (attention 모델 : [참고](https://github.com/pod3275/Paper-Review/blob/master/Seminar/%5B3%5D%20A%20Convolutional%20Attention%20Network%20for%20Extreme%20Summarization%20of%20Source%20Code/%5B3%5D%20Review.md))
- 허나 그들은 학습을 위해 많은 양의 parallel corpora(두 언어로 번역되어 있는 평행한 문장 쌍의 두 말뭉치)를 필요로 한다.
- parallel corpora를 만들려면 전문가의 지식이 있어야 하고, 잘 안쓰이는 언어는 종종 존재하지 않기도 한다.
- monolingual corpora(하나의 언어로 되어있는 말뭉치)만으로도 학습할 수 있는 기계 번역 모델을 제시한다.
  - 자세히는, **학습할 수 있는 방법**을 제안한다. (사실, loss function이 곧 모델이긴 하다.)
  
## Notation
- 도메인(언어) : source, target.
- Ws, Wt : source, target 도메인으로 되어있는 단어들.
- Zs, Zt : Ws, Wt의 word embedding 값.
- Encoder, Decoder.
  
  ![image](https://user-images.githubusercontent.com/26705935/42151545-52be727e-7e18-11e8-86a0-e2b0efea1248.png)
  
- 모델

  ![image](https://user-images.githubusercontent.com/26705935/42151648-9ce80914-7e18-11e8-8b4a-0c3e8602d7d8.png)
  
  - Sequence-to-sequence 모델 with attention.
  - encoder : bi-directional LSTM, decoder : LSTM, 3 layers.
  - source encoder = target encoder, source decoder = target decoder.
  
## 제안 기법
- 간단하게 요약하면, 두 개의 서로 다른 언어로 된 문장을 하나의 latent space에 매핑한 후, 이를 연결함으로써 번역이 되도록 함.
- 3가지 단계 : Denoising Auto-encoding, Cross Domain Training, Adversarial Training

### (1) Denoising Auto-encoding
- 어떤 언어로 된 input 문장 x를 latent space에 mapping하는 작업.
- input x에 noise model(C(x))를 추가한 x^를 encode하고, 이를 다시 decode했을 때 나오는 값과 x의 차이가 loss function.
- noise를 섞는 이유
  - more powerful한 feature를 찾기 위함. robust한 representation을 나타내기 위함.
  - 저자들은 "Without noise, model does not learn any useful structure in data." 라고 말함.

![image](https://user-images.githubusercontent.com/26705935/42151866-55554174-7e19-11e8-8ac4-27a18e681d43.png)

- C(x) : noise model : Pwd (word drop rate), k (permutation) 라는 hyperparameter.

### (2) Cross Domain Training
- 두 언어 l1, l2에 대해, l1의 문장을 l2의 문장으로 번역하는(서로 matching 시키는) 작업.
- l1으로 된 문장 x   
  --> 현재의 MT모델 M을 이용하여 l2언어로 번역한 문장 y = M(x)   
  --> noise model을 이용하여 noise를 추가한 C(y) sampled   
  --> C(y)를 다시 l1으로 번역하였을 때 x가 나오도록 함.
  
![image](https://user-images.githubusercontent.com/26705935/42152296-a4d995dc-7e1a-11e8-9529-0042148b4fb3.png)

### (3) Adversarial Training
- input 문장은 언어의 종류에 상관 없이 같은 latent space에 mapping 되어야 한다.
- (1), (2) 만으로는 서로 다른 두 언어 사이 strict한 matching을 하기 어렵다.
- Discriminator 이용
  - 주어진 latent vector가 l1에서 비롯된건지 l2에서 비롯된건지 구분해냄.
  
  - 학습 : ![image](https://user-images.githubusercontent.com/26705935/42152453-18d76950-7e1b-11e8-90b6-4806dc9e3a7a.png)
  
- Encoder는 이러한 Discriminator를 속이는 방향으로 학습됨.

  ![image](https://user-images.githubusercontent.com/26705935/42152486-345532ac-7e1b-11e8-89e6-6d3cce55cae7.png)
  
- 왜? (개인적인 의견)
  - 더 확실한 mapping 및 latent space 상에서의 일치를 위해.
  - 두 언어 사이 경계를 허물기 위해 : 어떤 언어로든 번역 가능하다. starGAN같은 구조.
  
### 
