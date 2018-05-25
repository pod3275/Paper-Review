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
 
 ## 4. 첫 번째 실험
첫 번째 실험은 Vanilla text로 학습하고, noise text를 test set으로 하여 번역의 성능을 판별한다. 판별 기준은 BLEU score인데, BLEU score는 기계 학습이 번역한 텍스트가 인간이 만들어 내는 것과 얼마나 유사한지를 기준으로 기계 학습의 성능을 측정하는 도표이다.

모델은 총 3가지의 서로 다른 NMT 모델을 사용하였다. Char2char 모델(Lee et al., 2017)은 attention 기법을 이용한sequence-to-sequence model으로, 복잡한 encoder 및 standard recurrent decoder를 가진다. Nematus 모델(Sennrich et al., 2017)은 WMT라는 기계 번역 워크샵에서 가장 우수한 성능을 보인 모델이다. 마지막으로 charCNN 모델은 word representation 기반의 character cNN 모델이다. 이는 형태학적으로 풍부한 특성을 가진 언어에서 좋은 성능을 보인다고 한다. (Kim et al., 2015; Belinkov & Glass, 2016; Costa-juss`a & Fonollosa, 2016; Sajjad et al., 2017)

3가지 모델 및 위에서 설명한 데이터를 이용하여 한 실험의 결과는 당연히 아래 표와 같이 잘 안 나온다. 이것은 논문이 작성된 이유이기도 하다.

![image](https://user-images.githubusercontent.com/26705935/40530594-5aec8f78-6034-11e8-82cc-66cb918270f1.png)

특히 독일 사람은 noise가 섞인 text를 보고 robust한 system에 의해 그 의미를 잘 이해하는 반면, 3가지 NMT 모델은 모두 말이 되지 않는 번역 결과를 내놓는다. 이로써 기존의 기계 번역 시스템은 noise가 섞인 text에 매우 취약하다는 것을 보인다.

단순히 spell check를 한 후에 checking된 text를 번역하면 성공할 수 있지 않나? 라는 의문을 가진 사람들을 위해 저자는 친절하게 한 번 더 실험한다. Noise text를 google의 spell-checker로 spelling check를 하고, 그 text로 번역 성능을 평가한다. Spelling check를 했을 때, 안 한 noise text보다 BLEU 점수가 5점 가량 상승하지만, 여전히 vanilla text를 번역할 때보다 성능이 낮게 나타난다. 표는 생략하겠다.

## 5. 제안 기법

기존 NMT 모델의 취약함 + spell checker로도 cover되지 않음을 보인 저자들은 자신들만의 방식을 제안한다.(사실 자신들만의 방식 이라기엔 기술적인 contribution은 별로 없다.) 

첫 번째로 새로운 word embedding 표현법을 제안한다. 기존의 word embedding값을 얻는 방식은, 단어 내의 문자의 순서에 예민하다. 그래서 문자의 순서를 바꾸는 swap, middle, random인 합성 noise에 취약한 모습을 보였다. 저자들은 새로이 구조적으로 불변하는 단어 표현법(structure invariant word representation)을 제안한다. 말은 거창하지만, 그냥 단순히 단어의 embedding 값으로 단어를 구성하는 각 문자의 embedding 값의 average를 쓰자는 것이다. (meanChar 모델이라고 저자들은 명명하였다.) 이렇게 되면 당연히 문자간의 순서에 independent해진다. 그럼에도 불구하고 다른 noise (keybord noise 혹은 natural noise)는 cover되지 않고, 낮은 성능을 보인다. 아쉽지만 이 방법은 accept되지 않는다.

두 번째는 학습 데이터에 noise가 섞인 text를 이용하자는 것이다. 이렇게 하면 noise가 섞인 text도 충분히 cover되어 높은 성능을 보일 것이다. 어찌 보면 당연한 소리다. 실험은 charCNN NMT 모델만을 이용하여 하였고, 아래 표와 같이 당연하게도 특정 종류의 noise로 학습한 모델은 그 종류의 noise에 대해 견고한 결과를 보인다. 하지만 한 종류의 noise로 학습한 모델은 다른 종류의 noise를 cover하지 못한다. 따라서 저자들은 여러 종류의 noise를 섞인 데이터로 모델을 학습시켰고, 결과적으로 random, keyboard, natural noise를 섞은 데이터로 학습한 것이 두루두루 좋은 성능을 나타내는 것을 볼 수 있다.

![image](https://user-images.githubusercontent.com/26705935/40530635-84953a78-6034-11e8-9c94-fbc021a4f1cb.png)

