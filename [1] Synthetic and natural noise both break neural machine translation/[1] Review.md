# Synthetic and natural noise both break neural machine translation
 - 저자 : Yonatan Bellnkov, Yonatan Bisk
 - 학회 : ICLR 2018
 - 날짜 : 2018.02.02
 - 논문 링크 : [[pdf]](https://arxiv.org/pdf/1711.02173)

## 1. Human robust language processing system
인간은 오타, 스펠링 오류, 철자 누락 등의 에러가 섞인 텍스트에 놀랍도록 robust한 언어 체계 시스템을 가지고 있다. (Rawlinson, 1976) 다음 문장을 보면 이해가 잘 될 것이다.
 
>“Aoccdrnig to a rscheearch at Cmabrigde Uinervtisy, it deosn’t mttaer in waht oredr the ltteers in a wrod are, the olny iprmoetnt tihng is taht the frist and lsat ltteer be at the rghit pclae.”

위의 문장은 단어에 에러가 많이 섞여 있지만, 딱 봤을 때 잘 읽힌다.
>According to a research at Cambridge University, it doesn’t matter in what order the letters in a word are, the only important thing is that the first and last letter be at the right place.) 

하지만 기계 학습 기반의 기계 번역 시스템은 이 문장의 의미를 제대로 해석하지 못한다. 이 문장을 구글 번역기를 이용하여 한국어로 해석하였을 때, 다음과 같은 말도 안되는 문장이 나타난다.
 
![image](https://user-images.githubusercontent.com/26705935/40490565-ce639ea8-5fa6-11e8-92b1-83ff1c3d7921.png)

논문 서론에서는 **인간의 robust한 언어 체계 시스템**의 특징을 설명한다. Saberi & Perrott (1999)에 의하면 robustness 특성은 audio에도 잘 적용된다고 한다. Rayner et al. (2006)은 **noise 환경에서 텍스트를 읽는 능력이 단지 11%밖에 떨어지지 않는다**는 결과를 찾았으며, McCusker et. Al. (1981) 은 **두 문자의 위치가 바뀐 형태는 인간에게 잘 인지되지 않는다**는 것을 확인하였다. 특히 중요한 것은, **인간은 단어의 형태(word shape)에 크게 의존**하며, 단어의 맨 앞 문자와 뒷 문자를 고정시킨다면 그 의미를 쉽게 알아볼 수 있다는 것이다. (Mayall et al., 1997, Reicher, 1969; Pelli et al., 2003)

이러한 인간의 robust한 언어 시스템과는 달리, Neural Machine Translation (NMT) 시스템은 에러가 섞인 데이터에 매우 취약하다. 에러가 섞인 텍스트 데이터를 명시적으로 처리하지 않으면 기계 번역 시스템은 이의 의미를 파악하기 힘들다.
 
그럼에도 불구하고 저자들은 character-based NMT가 중요하다고 얘기한다. 두 가지 이유가 있는데, 사전에 없는 단어들에 대한 long-tailed distribution (긴 꼬리 분포?)을 생성하는 데 도움을 준다고 하며, 큰 word embedding 값을 계산하는 computational cost을 줄여준다고 한다. 첫 번째는 아마 특정 언어의 큰 줄기(전체적인 특징)을 파악할 수 있도록 해준다는 것 같다. 그래서 character-based NMT를 계속하여 사용해야 한다.
 
본 논문에서는 기존 NMT 모델의 robustness를 증가시키기 위한 두 가지 방안을 제시한다. 첫 번째는 **word embedding 표현에 있어서 structure-invariant한 표현법을 제시**하는 것이고, 두 번째는 **noise가 섞인 데이터를 이용하여 adversarial training**을 하는 것이다. 결국에는 자연적인(natural) 혹은 합성한(synthetic) noise가 포함된 텍스트를 training set으로 하여 NMT 모델을 학습시키는 방법이 가장 효과적이었다. **본 논문의 point는 synthetic noise를 생성하는 방식이며, natural noise를 어떻게 최대한 cover할지에 대한 논의를 제기**한다.

## 2. Adversarial example
 Adversarial example은 머신 러닝 모델이 실 세계에서 적용되는 것이 매우 위험하다는 사실을 증명한다. 개념은 input에 아주 작은 변화를 줌으로써 머신 러닝 모델이 이를 알아보지 못하여 전혀 다른 결과를 내놓게 만들고, 이를 통해 머신 러닝의 큰 실패를 유도하는 것이다. 예를 들어, 아래 그림과 같이 인간이 알아보지 못하는 아주 작은 change가 섞인 두 이미지를 classification하였을 때, 두 결과가 완전히 반대 혹은 서로 관련 없게 나타날 수 있다.

![image](https://user-images.githubusercontent.com/26705935/40494041-e95a0776-5fae-11e8-93ea-eab617f6dfdc.png)
> (출처 : https://blog.openai.com/adversarial-example-research/)

 Adversarial example을 이용하여 머신 러닝 모델을 제 기능을 하지 못하도록 하는 것이 adversarial attack이다. Adversarial attack은 adversarial example을 생성하는 과정에서 모델의 parameter에 접근할 수 있냐 없냐에 따라 두 가지 종류로 나뉜다. White-box attack은 모델의 parameter에 접근하여 adversarial example을 생성한다. 반면에 Black-box attack은 parameter에 직접 접근하지 않고도 adversarial example을 생성 및 공격을 시도한다.
 
 본 논문에서는 모델의 parameter에 접근하지 않고, NMT 모델에 대한 adversarial example을 생성하는 black-box 방식을 고안한다. 모델의 parameter, 즉 gradient에 접근하지 않고, 저자들은 합성(synthetic) error 혹은 자연(natural) error를 통해 adversarial example을 생성한다. 또한 생성한 adversarial example을 여러 조합으로 섞은 데이터셋을 training set으로 함으로써 adversarial training을 적용한다. 정확히 얘기하면 ensemble adversarial training이고, 이를 통해 NMT 모델의 robustness를 증가시킨다.
 
## 3. Data and Error
 연구에서 사용한 데이터는 TED 강연의 parallel corpus이다. Parallel한 것은 같은 의미의 말을 다양한 언어로 표현되어 있기에 쓴 말인 것 같다. 데이터 셋의 크기는 아래와 같다. 언어는 프랑스어, 독일어, 체코어로 3가지의 언어로 된 말뭉치를 사용한다.

![image](https://user-images.githubusercontent.com/26705935/40494148-2194268a-5faf-11e8-97e3-2da9031a7246.png)

 다음은 본 논문의 point인, noise가 섞인 텍스트를 생성 방안이다. 저자들은 noise를 크게 자연적인(natural) noise와 합성(synthetic) noise로 나눈다. 자연 noise는 실제로 인간이 만들어내는 noise이고, 합성 noise는 여러 요인들에 의해 이렇게 만들어질 것이다 라고 예측하여 만들어낸 noise이다. 사실 “자연적”으로 만들어진다는 것의 정의가 모호하다고 생각한다. 인간이 “자연”스럽게 만드는 noise는 어떤 방식인가? 결국 keyboard로 입력하는 text라면 합성 noise와 크게 다를 바가 없지 않을까? 하지만 뒤의 결과를 보면 합성 noise는 cover되지만 자연 noise는 cover하지 못하는 현상을 볼 수 있다. 자연 noise에는 어떤 데이터들이 있는지, 그것이 합성 noise와 어떻게 다른 지 실제로 보고 싶은 생각이 들었다.
 
 자연 noise는 기존의 (TED 말뭉치와는 다른)말뭉치에서 발생한, 수정 가능한 error(오타, 스펠링 오류, 문자 누락 등)을 추출한다. 이후 TED 말뭉치 내에서 변환할 수 있는 모든 단어들을 natural noise가 섞인 단어로 변환하였다. 변환 비율은 아래와 같다.

![image](https://user-images.githubusercontent.com/26705935/40494197-3d7c8414-5faf-11e8-8680-46f28b4a7314.png)

합성 noise에 대해 저자들은 이를 만드는 4가지 방식을 제시한다. 각각 맨 앞과 맨 뒷문자를 제외한 두 문자의 위치를 바꾸는 Swap, 앞과 뒤를 제외한 중간 문자의 순서를 바꾸는 Mid, 앞과 뒤 문자를 포함하여 단어 내 모든 문자의 순서를 바꾸는 Rand 그리고 단어 내 한 문자를 키보드 상 가까이 있는 문자로 치환하는 Key이다. 

![image](https://user-images.githubusercontent.com/26705935/40494212-47211246-5faf-11e8-9ddc-f17b40a3f939.png)

 만드는 구조를 보면 썩 괜찮아 보인다. 인간이 발생시키는 noise가 섞인 단어들을 어느정도 cover할 수 있을 것이라고 생각 들었다. 근데 막상 결과를 보면 natural noise를 cover하지 못하는 현상을 볼 수 있다.
