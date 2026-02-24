[출처](https://m.blog.naver.com/twonkang00/222617518781?recommendTrackingCode=2)

[누설 전류와 콰이센트 전류](Leakage Current & Quiescent Current)의 측정 방법.

핀 맵에 표기된 이름은 다 다르지만 ILK는 기본적으로 VB, High Side Output, Voltage Suply를 연결하고 BV를 인가했을 때 측정되는 Input 전류 값을 읽는 것이다. VB, HO, VS는 모두 High Side 핀이다.

## ILK
ILK의 타입은 여러가지가 있지만 뒤에 백~천 단위의 숫자가 붙은 경우는 BV를 인가했을 때 나오는 누설 전류 값을 의미한다.

ex) ILK1200: High Side Pin에서 1250V (50V 정도 마진)를 인가했을 때, Input current를 측정. 처음부터 0V -> 1250V를 인가하면 소자가 망가지니 step by step으로 인가해야 된다.
 ```
 set_vi (HVS, 0, 0)
 set_vi (HVS, 100, 0)
 set_vi (HVS, 200, 0)
 .
 .
 .
 .
 set_vi (HVS, 1250, 0)
 if (V>1245)
 {
 measure->site[1]
 }
 ```
 이런 느낌으로 점진적 전압 인가 하면서 조건문을 활용해서 위 조건을 충족하면 측정 값을 각 site별로 뿌려주고 아니면 데이터 로그 상에서 ILK1200 항목은 Fail로 처리하게 한다.
 Fail이 뜨면 이를 디버깅 하는게 테스트 엔지니어의 업무 중 하나이다.

- 뒤에 수백~수천 단위가 나오면 BV(Breakdown Volatage)를 재는것
- 한번에 목표 전압을 인가하는게 아니라 단계별로 전압 인가
- High Side Pin 쪽에서 측정.


## IQ
IQ는 Quiscsent Current. 어떤 핀에서 측정하는지에 따라 이름이 바뀌는데 IQCC(IQDD) / IQBS가 있다. 둘 다 IC에 전압을 인가하는 곳이다. VCC, VB는 꾸준히 전압이 들어오는 핀으로 인풋이 아니라 전자 기기의 전원 코드와 비슷한 기능을 하는 핀이다.

IQCC는 VCC 단자를 측정할 때 얻는 Input Current 값을 재는 것이다. 이 때 세팅할 조건은 VB, VS 사이에서 15V가 인가되어야 하고 High Side Input 전압은 0V여야 한다. 

만약 조건이 세팅 되어있는데 IQBS가 Test recommendation에서 벗어났으면 이것을 컨트롤하는 Relay 소자의 문제일 수도 있고 또 다른 타입의 이슈일 수도 있다. 위 조건이 제대로 들어갔는지 측정해 봤는데 그 이유도 아니면 디버깅 하기 쉬어진다는데 아직 왜 그런지까지는 내가 모르겠다.

### IP
I는 current 전류이고 P는 pulse이다. IP 역시 전원 핀이 VCC냐 VB냐에 따라 측정할 핀이 달라지는데 IP가 의미하는 것은 오퍼레이팅 전류이다. 
![[Pasted image 20260128162702.png]]
위 오실로스코프처럼 깔끔하게 나오면 이상적이다. 하지만 IP 항목이 Fail나면 이 pulse파가 이상하게 측정 될 수 있다. IP 항목을 측정하기 위한 조건은 High Input 핀에 0 to 5V 펄스파를 인가해야 된다. 그러면 DVM을 이용해 High 핀과 GND를 연결해서 전압을 측정하면 2.5V를 기대할 수 있다. 자체 저항 값 때문에 지저분한 값이 나올 수 있지만 아무튼 중간 값이 기대 값이다. 
만약 측정 값이 2.5V가 아니라면 오실로스코프로 펄스파를 측정해야 된다. IC 자체의 불량이거나 IP 테스트 항목을 컨트롤 하는데 사용되는 Relay 소자의 문제가 대표적이다.