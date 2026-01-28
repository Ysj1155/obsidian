[출처](https://m.blog.naver.com/twonkang00/222617518781?recommendTrackingCode=2)

[누설 전류와 콰이센트 전류](Leakage Current & Quiescent Current)의 측정 방법.

핀 맵에 표기된 이름은 다 다르지만 ILK는 기본적으로 VB, High Side Output, Voltage Suply를 연결하고 BV를 인가했을 때 측정되는 Input 전류 값을 읽는 것이다. VB, HO, VS는 모두 High Side 핀이다.

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


IQ는 Quiscsent Current. 