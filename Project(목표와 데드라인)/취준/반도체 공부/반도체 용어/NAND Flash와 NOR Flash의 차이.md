[레퍼런스](https://arstechnica.com/information-technology/2012/06/inside-the-ssd-revolution-how-solid-state-disks-really-work/)

### NAND FLASH
- 반도체의 셀이 직렬 스트링 형태로 연결된 플래시 메모리
	- 그 스트링들이 비트라인/워드라인 구조로 배열되어있다
	- 이 구조 때문에 한 쎌만 찍어서 바로 읽는 게 어렵다
- 용량을 늘리기 쉽고 쓰기 속도가 빠르다
	- 대용량 구현이 쉬워 비용/bit가 낮고, 페이지 단위로 연속 쓰기 처리량(throughput)이 높다
- 저장 단위인 셀(cell)을 수직으로 배열하기 때문에 좁은 면적에서 많은 셀을 만들 수 있어 대용량화 가능.
	- 스트링 구조가 면적 효율이 좋고, 페이지/블록 단위 구조 어레이를 단순하게 만든다.
	- 결과적으로 셀당 트랜지스터/배선 부담이 적다
	- NAND는 구조적으로 집적도가 높고, 특히 3D NAND(수직적층)로 대용량을 구현한다.
- ex) USB 메모리, SSD, SD 카드, 스마트폰 저장장치 등 저장 매체

### NOR FLASH
- 반도체의 셀이 병렬로 연결된 플래시 메모리
	- 각 셀이 비트라인에 직접 연결되어 주소 기반(random access)읽기 쉬운 구조
- 셀 단위 랜덤 엑세스이며 읽기 속도가 빠름
	- 지연시간(latency)이 짧다.
	- 바이트/워드 단위로 바로 읽어 실행([XIP](obsidian://open?vault=obsidian&file=Area(%EA%BE%B8%EC%A4%80%ED%9E%88%20%EC%8B%A0%EA%B2%BD%EC%93%B8%EA%B1%B0)%2F%EB%B0%98%EB%8F%84%EC%B2%B4%20%EA%B3%B5%EB%B6%80%2F%EB%B0%98%EB%8F%84%EC%B2%B4%20%EC%9A%A9%EC%96%B4%2FXIP)) 가능
- 한 셀씩 기록하기 때문에 쓰기 속도가 느림
	- 쓰기/삭제 자체가 느리고 대량 쓰기 처리량이 낮아 쓰기 성능이 불리하다
- 저밀도에 가격이 비쌈
	- 셀당 연결 구조가 커서 면적 효율이 낮아 대용량에 불리
- ex) RAM처럼 실행 가능한 코드 저장

### 두 개 차이
![[Pasted image 20260123201411.png]]

- NAND는 보통 페이지 단위로 읽고/쓰기 때문에 작은 랜덤 읽기 지연이 크고, NOR는 바이트/워드 단위 랜덤 읽기 + [XIP](obsidian://open?vault=obsidian&file=Area(%EA%BE%B8%EC%A4%80%ED%9E%88%20%EC%8B%A0%EA%B2%BD%EC%93%B8%EA%B1%B0)%2F%EB%B0%98%EB%8F%84%EC%B2%B4%20%EA%B3%B5%EB%B6%80%2F%EB%B0%98%EB%8F%84%EC%B2%B4%20%EC%9A%A9%EC%96%B4%2FXIP)가 가능해 읽기에 유리하다.
- 접근 단위(Granularirty)
	- NAND: Read/Program은 Page 단위, Erase는 Block 단위
	- NOR: Read는 Byte/Word 단위, Erase는 Sector/Block 단위
- 삭제(Erase)가 먼저라는 플래시 공통 특성
	- 둘 다 write 전에 반드시 erase가 필요
	- NAND가 블록이 크고 관리가 복잡해서 SSD에서는 FTL이 필수
- ECC/Bad block
	- NAND는 공정상 Bad block이 존재하는 것을 전제로 설계
	- 그래서 ECC가 강하게 필요하고 컨트롤러가 중요하다.
	- NOR는 NAND만큼 불량 블록 전제 설계까지는 필요 x
- XIP(Execute-In-Place)
	- NOR이 왜 살아있는지의 1순위 이유
	- 펌웨어를 그 자리에서 실행 가능. 부팅/임베디드에 중요
- 용도 중심 요약
	- NAND: 저장 장치(대용량 데이터)
	- NOR: 펌웨어/부트(빠른 랜덤 읽기 + 실행)

-0