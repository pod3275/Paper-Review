# Population Based Training of Neural Networks
- Max Jaderberg, Valentin Dalibard, Simon Osindero, et al. 2017, 64회 인용.
- 모델의 hyperparameter optimization에 관한 논문.
- (약간 다른 제목이지만) 저자가 직접 NIPS 2017에서 발표한 [자료](https://vimeo.com/250399261)가 있음. 근데 NIPS 2017 accepted paper는 아님.
- 후에 발견 : 발표 유투브 [동영상](https://www.youtube.com/watch?v=l-Ga0E9vldg)도 있음.

## Introduction
- 모델의 **hyperparameter**란 학습 과정을 통해 변하지 않는 설정들을 의미함.
  - ex1) 학습에 관련된 hyperparameter로 learning rate, batch size 등.
  - ex2) 모델의 구조에 관련된 hyperparameter로 hidden layer의 node수, activation function의 종류 등.
  
- 모델의 성능을 최대로 끌어 올리기 위해서, 좋은 hyperparameter setting은 필수적임.
  - 기존에는 사람이 손으로 하나하나 다 튜닝해보고 찾아야 했음. (grid search의 일종)
  - 최근 들어 **hyperparameter optimization** method에 대한 많은 연구가 진행 중임.
  
- 기존 hyperparameter optimization method를 저자들은 두 가지로 나눔
  - **Parallel search** : 서로 다른 hyperparameter set으로 설정된 모델을 동시에 학습하여 가장 좋은 것을 선택하기.
  - **Sequential optimization** : 기존의 시행착오를 바탕으로 더 좋은 setting을 찾아보고 학습하여 성능 판단하기.
  - Parallel search는 많은 computational cost가 요구되고, sequential optimization은 많은 시간이 소요된다는 단점이 있음.

- 저자들은 이 parallel search와 sequential search를 연결하는(bridge) 쉽고 좋은 기법을 제안함.
  - ***Population Based Training (PBT)***
  - [Genetic algorithm](http://jeongchul.tistory.com/571)을 기반으로 하고 있음.

## Related Works
### 1. Optimization : Exploitation vs. Exploration
- 최적화 문제에 항상 등장하는 두 전략 exploitation과 exploration.

  ![image](https://user-images.githubusercontent.com/26705935/51182416-70da3d80-1911-11e9-9209-21aa71053d00.png)

- Exploi**t**ation : 지금까지 찾았던 것들 중 bes**t**인 것을 더 탐구해보자.
- Exploration : 새로운 정보를 얻기 위해 지금까지 찾은 것 말고 새로운 것을 탐색해보자.
- Exploitation과 exploration은 trade-off 관계이다.
  - 즉, 둘 다 잘 고려해야한다.
  
### 2. Sequential Optimization : Bayesian Optimization
- Bayesian optimization은 기존에 탐색했던 point들을 이용하여, 더 좋을 것으로 기대되는 point를 찾아서 탐색하는 방법임.

  ![image](https://user-images.githubusercontent.com/26705935/51182769-89972300-1912-11e9-97aa-96a89db47e70.png)

- Hyperparameter optimization을 위한 대표적인 기법임.
- 목표

  ![image](https://user-images.githubusercontent.com/26705935/51182492-bdbe1400-1911-11e9-8ee0-dfa23be5a18e.png)
  
  - x는 hyperparameter set, f(x)는 모델의 최종 accuracy.
  - 딥러닝 모델의 동작은 아무도 모르기 때문에, x에 대한 f(x)는 **black-box function**이다.
  - 이렇게 내부 구조의 동작을 정의할 수 없는 f(x)를 **Gaussian Process**를 통해 확률적인 함수로 approximate한다.
  
- "더 좋을 것으로 기대" 라는 것을 정의하기 위한 함수 : **activation function**

  ![image](https://user-images.githubusercontent.com/26705935/51182746-7e43f780-1912-11e9-9ae6-1255e871f65b.png)
  
  - Exploitation과 exploration을 모두 고려한 함수.
  - Activation function의 argmax 점을 다음으로 search할 점으로 여김.
  
- 성능은 좋음. 근데 **매우 오래 걸림.** 보통 1~3일.

### 3. Parallel search : Random search
- 서로 다른 hyperparameter set으로 설정된 여러 개의 모델을 동시에 돌려서 가장 좋은 것 찾기.

  ![image](https://user-images.githubusercontent.com/26705935/51182858-dda20780-1912-11e9-8bf0-59a282e66faa.png)
  
- Random search, grid search 등.
  - 각 hyperparameter set이 좋은 set인지 나쁜 set인지도 모른 채 학습을 끝까지 진행해야 하기 때문에 낭비임.
  - 이를 보완한 기법인 *Hyperband* : parallel search 기반의 대표적인 hyperparameter optimization 기법.

- 같은 시간동안 돌렸을 때 bayesian optimization보다 성능 좋음. 하지만 **이미 돌려본 것들에 대한 정보를 활용하지 못함.**

## Proposed Method
- ***Population Based Training (PBT)***

![image](https://user-images.githubusercontent.com/26705935/51183079-8d777500-1913-11e9-958e-b26d1f285c6f.png)

1. 몇 개를 한꺼번에 돌릴건지 결정한다. 각 모델 = worker.
2. 각 worker에 대해 hyperparameter set과 모델의 weight을 랜덤하게 설정한다.
3. 어느 step만큼 학습을 진행한다. 기존에 정한 threshold에 다다르면 각 worker에 대해 exploit & explore 적용.
4. **Exploit**
    - 내 모델의 중간 성능이 안좋다 싶으면 좋은 것으로 완전히 대체하자!
    - 2가지 전략
      - **Binary tournament** : 랜덤하게 선택한 모델보다 안좋으면 그걸로 대체.
      - **Truncation selection** : 내 모델 성능이 모든 worker중에 하위 20%면, 상위 20% worker들 중 랜덤하게 하나 골라서 그걸로 대체.
    - 모델의 weight과 hyperparameter set 둘 다 대체함. 좋으면 그대로 감.
5. **Explore**
    - 랜덤하게 만들어버리자!
    - **Exploit이 적용된 모델에 대해서만 적용.**
    - 2가지 전략
      - **Perturb** : 특정 factor 만큼 곱해버리기.
      - **Resample** : 그냥 다 무시하고 새로 뽑기.
6. 모델 학습이 끝날 때까지 3, 4, 5를 반복.
  
- 특징
  - Model selection(weight 학습)과 hyperparameter optimization을 동시에 진행.
  - Exploit은 non-differentiable하고 expensive한 metric에도 사용될 수 있음.
    - 예를 들어, testset에 대한 정확도 뿐만 아니라, 기계 번역의 BLEU score, human normalized performance 등
  - 모든 worker는 explore(새로운 지역 탐색)의 benefit을 나눠 받음.
  
## Experiments
- 3가지 learning problem에 대한 기존 모델에 PBT를 적용
  - RL(Reinforcement Learning), MT(Machine Translation), GAN(Generative Adversarial Networks)
  
### 1. RL (Reinforcement Learning)
- 강화학습의 neural network 구조의 agent를 학습.
  - 강화학습: Expected episodic(임시적인, 중간) reward E를 maximize하는 action의 집합 policy를 찾자.
  
- 3가지 task 및 모델
  - DeepMind Lab, UNREAL (Jaderberg et al., 2016)
  - Atari games, Feudal Networks (Vezhnevets et al., 2017)
  - StarCraft 2, A3C baseline agents (Vinyals et al., 2017)

- 실험 setting
  - Hyperparameter : learning rate, entropy cost 등 4개.
  - Step : RMSProp의 1 step.
  - Baseline : 같은 worker 수로 random search하여 찾은 모델.

- 결과

  ![image](https://user-images.githubusercontent.com/26705935/51258936-748fc200-19ee-11e9-8ade-db88e2edbe0e.png)
  
  - 성능 향상이 있었다. Measure가 hunam normalized performance임.
  - 추가적으로, PBT가 진행되면 될수록 hyperparameter 값의 특정한 변화(계속 내려간다든지)가 나타남.
  
### 2. MT (Machine Translation)
- State of the art 모델인 *Transformer* network (Vaswani et al., 2017, [paper](https://arxiv.org/pdf/1706.03762.pdf)) 를 tuning 하자.

- 실험 setting
  - 데이터 : [WMT 2014](http://www.statmt.org/wmt14/)의 English-to-German parallel data.
  - Hyperparameter : learning rate 및 3개의 dropout rate.
  - Eval measure : BLEU score on WMT newstest2012 dataset.
  - Exploit : binary tournament / Explore : perturb.
  - Baseline : hand tuning 또는 Bayesian Optimization으로 최적화된 모델.
  
- 결과
  
  ![image](https://user-images.githubusercontent.com/26705935/51748683-865d1d80-20f0-11e9-8b8f-af5a41a88b20.png)
  
  - 아쉽지만 SOTA(State-Of-The-Art)는 아님. (기존 Transformer 모델이 너무 커서, 작은 모델로 실험함)

  ![image](https://user-images.githubusercontent.com/26705935/51748770-c9b78c00-20f0-11e9-9b4b-dd91f37f8a22.png)
  
  - learning rate 보면 학습을 하면 할수록 **특정 형태**를 띄는 걸 알 수 있음. (계속 내려감)
  
### 3. GAN
- 위의 실험과 거의 동일, 결과도 동일해서 생략함.
  - 결국 성능 향상은 있었다.
  
## Conclusion
- ***PBT(Population Based Training)*** 이라는 최적화 기법을 제시함.
  - 유전 알고리즘을 기반으로 함.
  - Parallel search와 sequential optimization의 조합.
  - 모델의 weight과 hyperparameter를 동시에 최적화.
  - Hyperparameter의 **adaptive scheduling**이 가능함.
