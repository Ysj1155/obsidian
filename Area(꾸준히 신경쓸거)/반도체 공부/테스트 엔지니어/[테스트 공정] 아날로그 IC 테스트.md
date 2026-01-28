[출처](https://m.blog.naver.com/twonkang00/222556827666?recommendTrackingCode=2)

아날로그는 Discrete보다 복잡하다. Discrete는 핀이 3개 뿐이다. MOSFET이 디지털이고 Gate, Source, Drain으로 나뉜다. 핀이 3개라 테스트가 편하다. 아날로그는 핀이 많아 테스트가 복잡하다.

VCC, VB에서 전압이 잘 들어가는지, Input, Output Voltage와 Current는 각각 어떻고 동작 속도를 따져보고 펄스파 신호에서 T_on, T_off, T_R, T_F 값은 어떤지 측정해야 된다. Function Pads와 Fusing Pads에서 멀티미터 찍었을 때 정상 동작을 탐지하는 등 측정 해야 될 항목이 많다.

그래서 아날로그 IC를 테스트할 때는 프로그래밍이 필요하다. C언어 반복문 갈기기.

### 검사해야될 아날로그 핀
- VCC, VB: 아날로그 소자에 전압 지속적으로 인가되는 곳. 
- Vin, Vout: 각 소자마다 다르지만 대충 저정도면 이해한다. High Side, Low Side 따로 구분지어서 HIN, HOUT, LIN, LOUT이라고 하는 곳도 있다. Vin과 Vout은 DC 테스트를 할 때 가장 중점적으로 보는 곳이다. 옴의 법칙 V=IR 알면 된다. 
	일단 테스트 공정 엔지니어가 테스트를 진행하면 테스트 개발팀, 회로 설계 팀으로부터 'Test Plan'이라는 Sheet을 받는다. 거기에 Vout 값에 TYP=? Vout=??이런 식으로 정보가 넘어온다. TYP는 타겟 값을 의미한다. 개발팀, 회로 설계 팀이 Test Plan을 짜면서 Vout의 예상 값을 적은 것이다.
	그러면 테스트 엔지니어는 DC 테스트를 했을 때 Vout이 TYP에 적힌 Vout보다 차이가 크면 그에 대한 원인 분석을 한다. 아날로그 IC 주변에는 저항이나 CAP 등 소자가 많다. 우리는 Vin, Vout도 알고 저항고 알기 때문에 옴의 법칙에 따라 인풋/아웃풋 전류값을 안다. 이를 통해 어느 소자에서 발열이 심할 지 예상할 수 있는 것이다.
- GND, N.C: Not Connect로 안쓰는 핀이다. GND는 접지다. GND가 0이 아닌 경우를 Floating Source라고 한다.
- OVI, DVI, HVS: DVI(Dual Volatage Current). 이 3개는 테스트 Resource를 뜻하며 이 소자들을 이용해서 테스트를 진행한다. DVI는 채널이 2개뿐인 소자이며 HVS는 단일 소자이다. OVI는 채널이 8개가 된다. 테스트를 진행하기 앞서 각 아날로그 IC의 핀들에 어떤 테스트 Resource를 쓸지 schematic을 그릴 일이 많은데 