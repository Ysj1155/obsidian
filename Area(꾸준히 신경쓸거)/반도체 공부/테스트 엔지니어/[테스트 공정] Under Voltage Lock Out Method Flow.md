[출처](https://m.blog.naver.com/twonkang00/222617621874?recommendTrackingCode=2)

Input을 인가한 시점부터 Output이 발생하는데 까지 걸린 시간. 이 시간을 재는게 UVLO Filter Timer. 이론적 측정치로는 마이크로 세컨드 단위. 이 범위를 벗어나면 전자기기 켜지는데 10초 이상 걸릴 수 있다.
![[Pasted image 20260128170225.png]]
위 사진을 보면 Rising Time과 Falling Time 사이에 Delay 구간이 있다. 저 range는 회사마다 다르다. 

![[Pasted image 20260128171716.png]]

 VULO Filter Time 테스트 조건
 - VB를 15V에서 8V로 Drop
 - Input 전압은 5V를 유지
 - Output 전압을 체크
 Input 인가될 때 Output도 같이 따라 올라간다. 근데 VB를 Drop 시키면서 HO도 떨어지는데 그 뒤 얼마간의 Delay 시간을 가지다가 Output이 나오지 않게 된다. VB에 변화가 생겼다고 바로 Output이 즉각적으로 변화하는 것을 방지하기 위함이다.

### UVLO Positive / Negative
Filter time때는 Input 전압을 Pulse로 주는게 아니라 0V에서 5V로 인가. 천천히 Sweep한것도 아니고 그냥 바로 올린 케이스. 근데 UVLO에서 Time이 아니라 전압을 재고자 할 때는 Input 전압에 0 to 5V 펄스파를 인가한다.
![[Pasted image 20260128172751.png]]

positive 먼저 보면 VB를 올리는 상황에서 0부터 천천히 sweep시킨다. 어느 순간 Output이 발생하기 시작하는데 그게 V_th 지점이다. HO의 그림이 위처럼 표현됐지만 굉장히 Resolution이 높은 펄스파이다.

![[Pasted image 20260128172917.png]]

