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
  
- 