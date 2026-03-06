[출처1](https://horong01.tistory.com/entry/SSD%EC%9D%98-%EA%B8%B0%EB%B3%B8-%EC%9B%90%EB%A6%AC%EC%99%80-%EA%B5%AC%EC%A1%B0)
[출처2](https://blog.naver.com/ohmydata00/221183027428)

## SSD?
SSD(Solid State Drive). 하드 디스크 드라이브와 동일한 형태로 개발된 대용량 플래시 메모리. NAND Flash Memory 반도체를 이용해 정보를 저장한다. 

## SSD 구조
![[Pasted image 20260305164445.png]]
PC와 연결되는 인터페이스, 데이터 저장용 메모리, 인터페이스와 메모리 사이 데이터 교환 작업을 제어하는 컨트롤러, 외부 장치와 SSD 사이 처리 속도 차이를 줄여주는 Buffer 메모리로 구성되어 있다. 
### NAND Flash Memory
비휘발성 메모리. 전원이 꺼져도 데이터가 사라지지 않는다. 이는 데이터를 장기간 보관 가능하게 해준다. Cell이라는 기본 저장 단위로 구성되며 셀 하나는 1비트 또는 그 이상의 데이터를 저장할 수 있다. 여기에는 SLC(Single-Level Cell), MLC(Multi-Level Cell), TLC(Triple-Level Cell), QLC(Quad-Level Cell)이 있다. 각 기술은 속도와 내구성, 비용 등에서 차이가 있으며 SLC는 가장 빠르고 내구성이 뛰어나지만 용량당 비용이 높다. 반면 QLC는 더 많은 데이터를 저장할 수 있지만 속도와 내구성이 상대적으로 낮다. 
### 컨트롤러
데이터의 읽기, 쓰기, 삭제 등의 모든 작업을 관리하며 SSD의 성능을 결정짓는 중요한 요소 중 하나. 컨트롤러는 Wear Leveling, Garbage Collection, [[Project(목표와 데드라인)/취준/반도체 공부/SSD/SSD Trim.md|TRIM]]과 같은 다양한 알고리즘을 사용해 플래시 메모리의 수명을 연장하고 성능을 최적화 한다. 
- [[Project(목표와 데드라인)/취준/반도체 공부/SSD/SSD Wear Leveling.md|Wear Leveling]]: NAND Flash Memory는 쓰기와 삭제가 반복되면서 특정 셀의 내구성이 떨어지기 때문에 이를 균등하게 분배하는 기술
- [[Project(목표와 데드라인)/취준/반도체 공부/SSD/SSD Garbage Collection.md|Garbage Collection]]: 셀 내 불필요한 데이터 청소하는 기술
- [[Project(목표와 데드라인)/취준/반도체 공부/SSD/SSD Trim.md|TRIM]]: 해당 데이터 완전 삭제 기술

### DRAM 캐시
SSD가 데이터를 임시로 저장하는 고속 메모리 공간. 데이터의 접근 속도를 크게 향상시키는 요인. 

## SSD 작동 원리
인터페이스를 통해 데이터를 저장하고자 하는 경우 컨트롤러는 플래시 메모리의 ㅇ
### 데이터 쓰기
