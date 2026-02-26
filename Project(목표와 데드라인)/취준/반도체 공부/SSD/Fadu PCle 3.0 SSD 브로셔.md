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

---
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
---
![[Pasted image 20260226195150.png]]

1) 상단 기본 사양 영역 (제품 정체/호환성)
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

2) 성능 영역: E1.S and M.2 OP7 Performance
P7 뜻
- Over-Provisioning 7% (여유 공간을 약 7% 두는 구성).
- OP는 랜덤 쓰기 성능, QoS, 수명, GC 부담에 큰 영향을 줌.
- 즉 이 표의 성능은 OP7 조건에서의 수치라고 이해하는 게 안전함.
열(Column) 구조
- M.2: 960GB, 960GB(동일 용량이 두 칸인 건 모델/구성(예: 다른 SKU) 차이를 표현했을 가능성)
- E1.S: 1,920GB, 3,840GB
Capacity (GB)
- SSD 용량 표기. 성능/전력/내구성은 용량과 함께 달라질 수 있음(병렬성, NAND 다이 수, OP 구성 등).
Sequential Read (MB/s) / Sequential Write (MB/s)
- 순차 읽기/쓰기 대역폭.
- Notes에 조건이 붙어 있음:
    - Queue Depth = 128
    - IO Size = 128KB
- 의미: 링크/컨트롤러를 최대한 활용하는 최대치(peak) 조건에 가까움.
- 표에서:
    - Read 3,400 MB/s로 PCIe 3.0 x4 상한 근처
    - Write는 M.2는 1,500, E1.S는 2,400으로 차이가 큼 → 내부 구성/병렬성/전력/열/펌웨어 정책 차이를 의심해볼 포인트.
Random Read (KIOPS) / Random Write (KIOPS)
- 4KB 랜덤 I/O 성능을 IOPS(초당 처리 개수)로 표시. KIOPS = 천 IOPS.
- Notes 조건:
    - Queue Depth = 128
    - IO Size = 4KB
- 표에서:
    - Random Read는 모두 800K로 동일
    - Random Write는 M.2 30K, E1.S 75K/95K로 차이가 큼  
        → 랜덤 쓰기는 GC/FTL/OP/열/전력 제한 영향을 가장 크게 받아서 폼팩터/용량에 따라 잘 갈림.
- 직무 관점에서 여기서 바로 나오는 질문:
    - 이 랜덤 write 수치는 steady-state가? (브로셔는 말 안 함)
    - preconditioning(사전 쓰기) 조건은? (없음)
Random Read Latency (µs) / Random Write Latency (µs)
- 단일 I/O 지연시간 성격이 강함.
- Notes 조건:
    - Queue Depth = 1
    - IO Size = 4KB
- 표에서:
    - Read 70µs 고정
    - Write는 M.2 20µs, E1.S 15µs
- 주의: 지연시간은 평균인지, 중간값인지가 없어서 숫자만으로는 제한적. 실무에선 percentile(꼬리)이 더 중요함.
QoS (99.9%) Random Read (µs) / QoS (99.9%) Random Write (µs)
- 여기서 QoS(99.9%)는 보통 p99.9 latency 의미로 쓰임(1,000개 중 1개 꼴의 느린 지연).
- Notes 조건:
    - Queue Depth = 1/64
    - IO Size = 4KB
- 표기 150/400 같은 형태는 QD=1일 때 150µs, QD=64일 때 400µs 같은 식으로 읽는 게 자연스러움.
- 특히 Write QoS가:
    - M.2: 200/5300 µs
    - E1.S: 60/2000, 60/1500 µs
- 이게 의미하는 바:
    - 랜덤 쓰기에서는 어떤 상황에서 ms 단위 tail latency 이벤트가 생긴다는 신호일 수 있음(예: GC, mapping update, flush, thermal/power state 변화).
    - E1.S가 M.2보다 QoS가 더 좋다는 메시지를 주려는 구조.

3) Power Consumption (전력)
Active (W)
- 활성 동작 시 전력. 표는 모델/용량에 따라:
    - M.2: <6.0, <7.5
    - E1.S: <9.5, <10.0
- 여기서 중요한 점: Active가 어떤 워크로드(읽기/쓰기/혼합)인지 표에 없음. 실무에서는 이 조건이 없으면 비교가 어렵다.
Idle (W)
- 유휴 전력:
    - M.2: <2.0
    - E1.S: <3.0
- 역시 절전 상태(PS state, APST 등) 조건이 없어서 실제 시스템에서 달라질 수 있음.

4) Reliability (신뢰성)
MTBF (Hour): **2.0M**
- 평균 고장 간격(통계적 지표). 개별 제품이 2M시간 버틴다가 아니라, 모델링 기반 신뢰성 지표로 이해해야 함.
UBER: 1 Sector per 10^17 Read
- 읽기 중 “복구 불가능 오류”의 확률 지표(보통 Uncorrectable Bit Error Rate).
- 데이터센터에서 데이터 무결성 얘기할 때 자주 등장.
Retention: 3 Months @ 40°C (EOL)
- **수명 말기(EOL)** 조건에서 40°C 환경에서 3개월 데이터 유지.
- “EOL에서”라는 조건이 붙는 게 중요함(새 제품 상태가 아니라, 마모된 상태 기준).
5) Warranty (보증)
DWPD: 1.3
- Drive Writes Per Day: 하루에 드라이브 전체 용량을 1.3번 쓰는 수준까지 보증한다는 의미.
Period: 3 Years
- 보증 기간 3년.
Operating Temperature (°C): 0 ~ 70
- 동작 온도 범위.
---
#### 내가 판단하고 공부할거
(1) 성능 공부
- Seq(128KB, QD128) vs Rand(4KB, QD128)의 의미
- QD가 올라가면 왜 성능이 오르고, 왜 latency tail이 생기는지
- 폼팩터/용량(E1.S vs M.2, 960 vs 3840) 차이가 왜 성능/QoS에 반영되는지
 (2) QoS 공부 (가장 직무형)
- QoS(99.9%)가 무엇인지(p99.9)
- 왜 Random Write QoS가 ms로 튈 수 있는지(FTL/GC/flush/열/전력)
- consistent latency를 어떻게 테스트로 증명하는지(Percentile 기반 리포트)
 (3) 전력/열 공부
- Active/Idle 전력 측정 조건이 왜 중요한지
- thermal throttling이 실제로 언제 발생하는지(폼팩터/에어플로우/히트싱크/워크로드)
- KIOPS/Watt 같은 효율 지표를 어떻게 산출하는지
(4) 신뢰성 공부
- UBER/Retention/DWPD가 의미하는 바
- EOL retention 조건이 왜 중요한지(마모 상태에서의 데이터 유지)
---
![[Pasted image 20260226220355.png]]

1) PCIe 3.0 SSD Security Features (보안/무결성)
Data-path E2E Protection (SECDED)
- 호스트 → 컨트롤러 내부 → NAND로 데이터가 이동하는 전체 경로에서 **오류 검출/정정**을 하는 메커니즘.
- SECDED는 보통 Single Error Correction, Double Error Detection(1비트 정정, 2비트 오류 검출) 계열.
- DRAM/버스/내부 데이터 경로에서 생기는 **소프트 에러**(bit flip)가 데이터 훼손으로 이어지는 걸 막는 쪽.
- 데이터센터는 성능도 중요하지만, 결국 무결성(integrity)이 핵심.
- E2E가 “어디부터 어디까지”인지(호스트 메모리까지 포함? SSD 내부만?)
- 오류가 발생했을 때 어떤 로그(telemetry/SMART)에 기록되는지
- 실제로는 벤더 내부 테스트 항목이 많아서, 외부에서 할 수 있는 건 에러 카운터/로그로 추적 가능하냐가 핵심 질문.
Internal RAID
- SSD 내부에서 NAND 다이/플레인/블록 단위로 패리티/리던던시를 두는 구조(일종의 RAID-like).
- 흔히 NAND die failure 같은 상황에서도 데이터를 살리기 위한 내부 보호층.
- NAND는 결함/열화가 생길 수 있으니까, 엔터프라이즈 SSD는 고장 나도 데이터 보호 계층을 둠.
- RAID5/6 같은 의미로 오해하면 안 됨: 외부 RAID와 다르고 내부 구현은 벤더마다 다름.
- 보호 대상이 die failure급인지, page/block error 수준인지
- 어떤 장애를 커버하는지(예: uncorrectable block 증가 시 자동 리빌드?)
- 관련 이벤트/상태가 로그로 보이는지.
Self Encrypting Drive (AES-XTS)
- SSD가 내부적으로 데이터를 암호화해서 저장하는 SED(Self-Encrypting Drive).
- AES-XTS는 저장장치용 표준적인 블록 암호 모드.
- 데이터센터/기업 환경은 분실/폐기/리턴(RMA) 시 데이터 유출 방지가 중요.
- 암호화는 켜져 있는지, 키 관리가 제대로 되는지가 핵심.
- 암호화가 항상 on인지, 옵션인지
- 키는 어디에 저장되고(컨트롤러/보안 영역), 삭제(crypto erase)가 되는지
- 성능 영향이 없다는 말은 보통 하드웨어 가속이라 크지 않다 수준이지, 워크로드에 따라 영향이 0은 아님. 벤치로 확인 가능.
Secure Boot
- SSD 펌웨어가 부팅/로딩될 때 서명 검증 등으로 악성/변조 펌웨어가 실행되지 않도록 하는 체인.
- 펌웨어 변조는 데이터센터 보안에서 큰 이슈(공급망/펌웨어 공격).
- SSD도 하나의 컴퓨터라 부팅 체인이 필요함.
- 서명 체인의 루트(키) 관리 방식
- 펌웨어 업데이트 시 서명 검증 강제 여부
- 실패 시 동작(부팅 차단? read-only?).
TCG/OPAL 2.01
- TCG(Trusted Computing Group)의 OPAL 규격: SED의 관리/정책/잠금/인증 표준.
- 암호화 지원만으로는 부족하고, 운영체제/관리도구에서 표준으로 제어 가능해야 엔터프라이즈에서 씀.
- OPAL에도 프로파일/기능 범위가 있음.
- 실제로는 sedutil 같은 도구로 잠금/해제/PSID revert 등 동작 확인이 필요.
- 단순히 Supports OPAL은 체크박스일 수 있음 → 어떤 툴/OS 조합에서 검증됐는지 질문해야 함.

2) PCIe 3.0 SSD Data Center Features (운영/관리/가용성)
Multiple Namespaces (NS) – Max 16 NS
- 하나의 물리 SSD를 여러 개의 논리 디바이스(네임스페이스)로 나누는 기능.
- 테넌트 분리, 서비스별 분할, 가상화 환경에서 유용.
- 운영/관리 자동화에서 자주 쓰임.
- 실제로 16개까지 생성/attach 가능한지
- 네임스페이스 분할이 성능/QoS에 어떤 영향을 주는지
- 지원하는 NS 관리 명령 범위.
SMART / Health Log / Telemetry Log (OCP log fully support)
- SMART/Health: 기본 상태 지표(온도, 미디어 오류, 사용량 등)
- Telemetry: 더 상세한 내부 상태/이벤트 로그(벤더/표준 로그 페이지)
- OCP log: 클라우드 SSD 규격에서 요구하는 운영/디버깅용 로그 요구사항
- 데이터센터 운영은 고장 나기 전 조기 감지가 핵심.
- 장애 분석(RCA)도 로그 없으면 불가능.
- Fully supports라고 써도 실제로는 어떤 log page를 제공하는지**가 중요.
- 로그 필드 정의서가 있는지
- 호스트에서 nvme-cli로 덤프 가능한지.
Latency Monitoring Feature
- SSD 레벨에서 지연시간을 측정/카운팅/로그로 제공하는 기능(추정).
- 어느 시점에 latency가 튀었는지를 찾기 위한 운영 기능.
- 데이터센터 성능 문제는 평균보다 tail latency spike가 원인인 경우가 많음.
- 병목이 스토리지인지 네트워크인지 앱인지 분리할 때 SSD 측 지표가 도움이 됨.
- 어떤 방식으로 제공? (log page? vendor command? histogram?)
- 시간 해상도/구간(버킷) 정의
- 호스트 도구(nvme-cli)에서 바로 볼 수 있는지.
NVMe-MI 1.0a
- NVMe Management Interface: 대규모 환경에서 SSD를 표준 방식으로 관리/모니터링하기 위한 인터페이스.
- 수천/수만 개 SSD를 운영하는 데이터센터는 표준 관리 경로가 필요.
- 어떤 전송 경로(SMBus/PCIe VDM 등)로 제공되는지
- 실제 BMC/관리 솔루션과 연동 검증이 되었는지.

---

## 5) Power Loss Protection (PLP)

**무슨 기능?**

- 전원 장애가 나도, SSD가 쓰기 중이던 데이터/메타데이터를 보호하는 기능(보통 커패시터 기반).
    

**왜 중요?**

- 데이터센터는 전원 장애/리셋이 실제로 일어남.
    
- PLP가 없으면 파일시스템/DB 깨질 수 있음.
    

**검증/질문 포인트**

- 보호 범위: 사용자 데이터만? FTL 메타데이터까지? in-flight write까지?
    
- 전원 차단 테스트 시 pass 기준(데이터 무결성/재부팅 후 복구 시간/오류 카운터)
    
- 실제 커패시터 탑재 여부(폼팩터별 차이 큼).
    

---

## 6) Multiple Sector Size Support (512 / 4096)

**무슨 기능?**

- 512B 섹터와 4KB 섹터(4Kn)를 지원.
    

**왜 중요?**

- OS/플랫폼/파일시스템/DB에 따라 4Kn이 요구되거나 성능/효율 상 이점이 있음.
    

**검증/질문 포인트**

- 4Kn이 **native**인지(진짜 4K 물리 섹터), 512e(에뮬레이션)인지
    
- 포맷 변경 절차, 호환 OS, 성능 영향
    
- 4Kn에서의 alignment/WA 변화.