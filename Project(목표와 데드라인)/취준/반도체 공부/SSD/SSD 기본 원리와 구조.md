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
데이터의 읽기, 쓰기, 삭제 등의 모든 작업을 관리하며 SSD의 성능을 결정짓는 중요한 요소 중 하나. 컨트롤러는 Wear Leveling, Garbage Collection, [TRIM](obsidian://open?vault=obsidian_clean&file=Project(%EB%AA%A9%ED%91%9C%EC%99%80%20%EB%8D%B0%EB%93%9C%EB%9D%BC%EC%9D%B8)%2F%EC%B7%A8%EC%A4%80%2F%EB%B0%98%EB%8F%84%EC%B2%B4%20%EA%B3%B5%EB%B6%80%2FSSD%2FSSD%20Trim)과 같은 다양한 알고리즘을 사용해 플래시 메모리의 수명을 연장하고 성능을 최적화 한다. 

