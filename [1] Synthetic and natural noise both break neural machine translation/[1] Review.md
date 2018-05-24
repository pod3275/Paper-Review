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

