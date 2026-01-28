[출처](https://m.blog.naver.com/twonkang00/222556827666?recommendTrackingCode=2)

아날로그는 Discrete보다 복잡하다. Discrete는 핀이 3개 뿐이다. MOSFET이 디지털이고 Gate, Source, Drain으로 나뉜다. 핀이 3개라 테스트가 편하다. 아날로그는 핀이 많아 테스트가 복잡하다.

VCC, VB에서 전압이 잘 들어가는지, Input, Output Voltage와 Current는 각각 어떻고 동작 속도를 따져보고 펄스파 신호에서 T_on, T_off, T_R, T_F 값은 어떤지 측정해야 된다. Function Pads와 Fusing Pads에서 멀티미터 찍었을 때 정상 동작을 탐지하는 등 측정 해야 될 항목이 많다.

그래서 아날로그 IC를 테스트할 때는 프로그래밍이 필요하다. C언어 반복문 갈기기.

### 검사해야될 아날로그 핀
- VCC, VB: 아날로그 소자에 전압 지속적으로 인가되는 곳. 
- Vin, Vout: 각 소자마다 다르지만 대충 저정도면 이해한다. High Side, Low Side 따로 구분지어서 HIN, HOUT, LIN, LOUT이라고 하는 곳도 있다. Vin과 Vout은 DC 테스트를 할 때 가장 중점적으로 보는 곳이다. 옴의 법칙 V=IR 알면 된다. 