[출처](https://m.blog.naver.com/twonkang00/222574810879?recommendTrackingCode=2)

테스트 아이템이 중요하다. 테스트 아이템은 개발팀에서 반도체가 정상적으로 동작하는지 확인하기 위한 지표라고 생각해두자.

일반적으로 생각하는 IC칩을 보면 PMIC가 있다. 하나의 소자에 많은 PIN들이 있어서 각 PIN들은 제각각의 기능을 한다. VCC, VB는 소자에 전력을 공급하고 VIN, VOUT은 말 그대로 Input 전압, output 전압이 나오는 쪽이다. 
기본적으로 Input 전압을 인가하면 Output 전압은 소자 내에서 다양한 매커니즘을 거쳐 파형이 출력된다. 
테스트 엔지니어는 15V를 인가하는데 왜 Output이 3.3V로 이론상 나와야 되는데 실측 값이 왜 다른 값이 나올까? 하고 궁리하는게 테스트 엔지니어다. 

![[Pasted image 20260128142543.png]]

예시 사진으로
A: VB(VCC와 같은 역할)
B: HO(High Side Output)
C: VS
D: LO
E: HIN
F: NC(No Connection)
G: LIN
H: NC
라고 가정해보자.

VB, VCC는 IC에 지속적으로 전력을 공급하는 PIN이다. 15V를 인가하고 있다고 가정한다. HIN, LIN, HO, LO는 인풋과 아웃풋으로 사용되는 PIN이다. H,L은 High Side / Low Side이다. 하나의 IC 내에 2개의 인풋이 있는 것이다.

- ILK: 누설 전류. IC에 대한 BV(Breakdown Voltage; IC가 버틸 수 있는 최대 전압 값)를 측정하고자 전압을 인가하는 과정에서 발생하는 전류. 수백nA가 나오는게 보통이다. 예를 들어 어떤 IC의 BV가 1200V라고 할 때 이에 근접한 전압 값을 인가해준다. 그 때 HO, VB, VS 핀에서 전류를 쟀을 때 나오는 값이 ILK가 된다. 데이터 시트 or 테스트 플랜에 *TYP*라고 적힌 심볼이 있다. 개발 쪽 테스트 엔지니어가 시트를 제작 할 때 예상 ILK를 적은것이다. 이 TYP를 기준으로 min, max 값이 같이 기입되는데 만약 측정 시 이 범위를 벗어난다면 소자가 불량이거나 플래너에 잘못 기입된거라고 파악한다.
- IQBS: Quiescent current. VB 핀에서 측정한 콰이센트 전류다. 콰이센트 전류는 VCC, VB에서 전압이 인가되있는 상태에서 Input 핀에 전압을 인가했을 때 VB 핀에서 측정되는 전류 값이다. 이 전류는 제품을 빠르게 동작시키기 위한 *대기 전류*가 그 역할이다. IQBS를 재려면 VB 핀에 측정하고 IQCC를 재려면 VCC 핀에서 측정하면 된다. 핀만 다를 뿐 측정을 위한 조건은 둘  다 동일하게 Input 핀에 전압을 인가할 때 이다. IQBS와 IQCC는 ILK보다 높은 전류 값을 가지며 보통 수백 uA 전류값을 가진다.