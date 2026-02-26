[출처](obsidian://open?vault=obsidian_clean&file=Project(%EB%AA%A9%ED%91%9C%EC%99%80%20%EB%8D%B0%EB%93%9C%EB%9D%BC%EC%9D%B8)%2F%EC%B7%A8%EC%A4%80%2FFADU%2F20250509162324_FADU_PCIe3.0_SSD_Brochure.pdf)

Turnkey Storage Solution with FLASH Controller, Customizable Firmware, and SSD Designs.

우리는 컨트롤러도 있고, 펌웨어도 우리 손에 있고, SSD 하드웨어 설계까지 묶어서 고객이 빠르게 SSD를 만들 수 있게 해준다.

FADU’sPCIe 3.0 NVMe SSDsaredesigned to meetthe increasing demands placed on Hyperscaler, Hyper converged, Enterprise, and Edge data centers. At the heart of FADU’s SSDs is an innovative SSD controller architecture that enables ultra-low and consistent latency while virtually eliminating thermal throttling issues. As a result, FADU SSDs deliver industry leading KIOPS/Watt performance while supporting superior QoS. In the industry’s first E1.S form factor SSD, FADU’s PCIe 3.0 SSD consumes up to 30% less power and operates up to twice as fast as other PCIe 3.0 SSDs. Consistent low-latency delivers stable, superior Quality of Service (QoS) at any workload. The SSDs support a variety of features for modern data centers, including hardware-based security, advanced telemetry, virtualization functions, data path, and power loss protection.

AWS/Google/Microsoft 같은 초대형 데이터센터인 Hyperscaler, 스토리지/컴퓨트/네트워크를 소프트웨어로 통합한 인프라(HCI)인 Hyperconverged, 일반 기업 데이터 센터 용도의 Enterprise, 데이터가 생성되는 현장 가까운 소형/분산 데이터센터인 Edge data center를 대상으로 설계. PC용이 아니라 데이터 센터용(엔터프라이즈 SSD) 포지셔닝.
컨트롤러 아키텍쳐가 혁신적이기 때문에 지연시간이 낮고 일관적이다(ultra-low and consistent latency). 열 때문에 성능이 떨어지는 현상(thermal throttling)을 줄였다.
KIOPS/Watt 성능 업계 최고(같은 전력에서 더 많은 IOPS를 낸다고 주장). superior Qos(지연시간 분포(p99/p99.9)) 지원. -> 전력 효율 + Qos가 핵심 가치다. 
업계 최초 E1.S 폼팩터 SSD. PCIe 3.0 SSD 대비 전력 30% 덜 먹고, 속도 2배 빠르다 주장. -> ㅣㅂ교 대상, 측정 조건(워크로드/QD/온도/방열)이 없어 기준 모호.
데이터 센터 기능 체크리스트로 현대 데이터센터에 필요한 기능을 지원한다. hardware-based security(하드웨어 보안), advanced telemetry(상태/로그/모니터링 데이터), virtualization functions(가상화 관련 기능), data path(데이터 경로 최적화/기능), power loss protection(PLP: 전원 꺼져도 데이터/메타데이터 보호). 성능만이 아니라 운영/관리/보안/신뢰성 기능이 있다.
업계 표준인 NVMe 1.3, PCIe 3.0 x4, OPC NVMe Cloud SSD 1.0을 따르면서 데이터센터(특히 클라우드)에서 요구하는 규격/호환성 틀 안에 있다고 주장.

![[Pasted image 20260226192910.png]]

A. 제품 포지션/세그먼트
- PCIe 3.0 x4 + NVMe 1.4 + OCP NVMe Cloud SSD 1.0는 클라우드/데이터센터 호환성을 의식한 엔터프라이즈 SSD 포지션
- 폼팩터: M.2 | E1.S. M.2는 범용/엣지/서버 일부, E1.S는 데이터센터 전용 트렌드(EDSFF).
- 용량 1/2/4TB -> 엔터프라이즈라면 더 큰 용량도 흔한데(예: 8TB+), 여기선 비교적 제한적.  메인스트림 엔터프라이즈에 가깝고, 하이엔드 대용량 라인업 느낌은 약함.
B. 성능의 상한(peak) 수준
- Seq Read 3,400 MB/s / Seq Write 2,400 MB/s -> PCIe 3.0 x4 링크 대역폭 근처까지 뽑는 최대치
- Random Read 800KIOPS / Random Write 95KIOPS -> 읽기 강점, 쓰기 상대적으로 약함. 이 패턴은 쓰기 steady-state/GC 영향을 의심하게 만드는 시그널.
C. 전력/효율 주장 방향
- 성능 단독보다 효율(성능/전력)과 QoS를 핵심 세일즈 포인트로 잡고 있음.

측정 조건이 없음. 
- 성능 수치의 조건: IO size, QD, numjobs, read/write mix
- 상태: resh-out-of-box vs steady-state(Preconditioning)
- QoS의 정의: p99? p99.9? 측정 window?
- 전력 측정 조건: 활성 상태가 read인지 write인지, PS state/APST 설정
- 폼팩터별 차이: M.2 vs E1.S는 열/전력/스로틀링이 완전히 다름
-> “브로셔 수치는 peak 조건이라 QD sweep/steady-state 조건을 분리해 재현하고, 평균뿐 아니라 p99.9 지연으로 QoS를 확인하겠습니다.” 이거 각 단어 뜻이 뭔지 알아야 한다. 

#### 알면 좋은거
공부 목표: 
- NVMe 1.4: Identify/Log pages/Telemetry 관련 확인 포인트
- OCP NVMe Cloud SSD 1.0: 클라우드 운영에 필요한 log/health 요구사항(가능하면 spec 읽기)
- Security standards: TCG Opal, secure boot 같은 것들이 실제로는 설정/모드/제약이 많음  
	→ 어떤 명령/툴/절차로 확인할지 준비

#### 문답
- 위 성능 수치의 IO size/QD/numjobs는?
- 랜덤 write 95K는 steady-state인가? preconditioning 절차는?
- QoS “4x”는 p99.9 기준인가? workload mix는?
- Active power <10W는 read/write 중 어떤 workload 기준인가? PS state는?
- M.2와 E1.S에서 **스로틀링 발생 조건**과 성능 차이는?

![[Pasted image 20260226195150.png]]1) 상단 기본 사양 영역 (제품 정체/호환성)
Interface: PCIe 3.0 x4
- SSD가 호스트(CPU/칩셋)와 통신하는 물리 인터페이스.
- PCIe 3.0 레인 4개(x4)를 사용한다는 뜻.
- 최대 대역폭의 상한을 결정해. (PCIe 3.0 x4면 순차 읽기 3.4GB/s 정도가 상한 근처)
NVMe: NVMe 1.4
- PCIe 위에서 동작하는 스토리지 프로토콜 버전.
- NVMe 1.4라는 건 해당 버전의 명령/로그 페이지/기능을 지원한다는 의미(기능 범위는 구현에 따라 다름).

OCP Compliance: OCP NVMe Cloud SSD 1.0
- 클라우드 데이터센터 운영 관점에서 필요한 요구사항(특정 로그/관리 항목 등)을 맞춘다는 주장.
- 실무에서는 OCP 규격의 어떤 항목까지 지원하냐가 중요해서, 보통 세부 스펙/테스트 리포트가 따로 필요함.

Controller: FADU FC3081
- SSD 컨트롤러(SSD의 두뇌) 모델명.
- 성능/QoS/전력/열/신뢰성 특성이 이 컨트롤러+펌웨어 정책에 크게 좌우됨.

NAND: SKH V6 128 Layer 4D eTLC
- 사용한 NAND 종류/세대.
- eTLC는 일반 TLC를 엔터프라이즈 용도로 맞춘(보통 내구성/성능/QoS를 위한) NAND 라인업을 의미하는 경우가 많음(정확한 정의는 벤더마다 다름).

Form Factor
M.2 22110
- M.2 폼팩터, 22mm 폭 / 110mm 길이.
- 서버나 엣지 장비에 쓰이기도 하지만, 방열/스로틀링이 시스템 설계 영향을 많이 받음.

E1.S (5.9/9.5/15/25mm)
- EDSFF 계열 데이터센터용 폼팩터(E1.S).
- 두께 옵션이 여러 개라는 뜻(방열/PLP 부품 탑재/구성에 영향).
- U.2 플랫폼 최적화. 보통은 서버 섀시/백플레인/운영환경에 맞춘다는 의미