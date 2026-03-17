---
tags:
  - reference
  - scrap
  - processed
  - ssd
  - validation
  - enterprise-ssd
---
[[[Project(목표와 데드라인)/취준/FADU/20250509162304_FADU_PCIe4.0_SSD_Brochure.pdf|출처]]

Turnkey Storage Solution with FLASH Controller, Customizable Firmware, and SSD Designs.

우리는 컨트롤러도 있고, 펌웨어도 우리 손에 있고, SSD 하드웨어 설계까지 묶어서 고객이 빠르게 SSD를 만들 수 있게 해준다.

FADU’s PCIe 4.0 NVMe SSDs are designed to meet the increasing demands placed on Hyperscaler, Hyper converged, Enterprise, and Edgedata centers. At the heart of FADU’s SSDs is an innovative SSD controller architecture that enables ultra-low and consistent latency while virtually eliminating thermal throttling issues. As a result, FADU SSDs deliver industry leading KIOPS/Watt performance while supporting superior QoS. It consumes up to 30% less power and operates up to twice as fast as other PCIe4.0 SSDs, leading to the industry’s best IOPS/Watt. TheSSDssupportavariety offeatures for moderndata centers, including hardware-based security, advanced telemetry, data path, and power loss protection. FADU’s PCIe 4.0 SSD Platform is based on industry standard specifications including NVMe 1.4b, PCIe 4.0 x 4, and OCP NVMeCloud SSD1.0a.

FADU의 PCIe 4.0 NVMe SSD는 하이퍼스케일러, 하이퍼컨버지드, 엔터프라이즈, 엣지 데이터센터에 점점 더 커지는 요구를 충족하도록 설계되었다.  
FADU SSD의 핵심에는 혁신적인 SSD 컨트롤러 아키텍처가 있으며, 이는 초저지연과 일관된 지연시간을 가능하게 하고 열 스로틀링 문제를 사실상 거의 없애준다.
그 결과, FADU SSD는 업계 최고 수준의 KIOPS/Watt 성능을 제공하면서도 우수한 QoS를 지원한다.  
이 제품은 다른 PCIe 4.0 SSD보다 전력 소모를 최대 30%까지 줄이면서, 최대 2배 더 빠르게 동작할 수 있어, 결과적으로 업계 최고 수준의 IOPS/Watt를 달성한다.
또한 FADU SSD는 현대 데이터센터에 필요한 다양한 기능을 지원하는데, 여기에는  
하드웨어 기반 보안, 고급 텔레메트리, 데이터 경로 보호, 전원 손실 보호가 포함된다.
FADU의 PCIe 4.0 SSD 플랫폼은  
NVMe 1.4b, PCIe 4.0 x4, OCP NVMe Cloud SSD 1.0a와 같은  
업계 표준 규격을 기반으로 한다.

---
![[Pasted image 20260317214115.png]]

### 기본 스펙

- **Storage Platform:** DELTA  
    → 제품 플랫폼 이름이 **DELTA**라는 뜻이다.
    
- **Interface:** PCIe 4.0 x4  
    → PCIe 4세대, 4레인 인터페이스를 사용한다.  
    일반적으로 PCIe 3.0 x4보다 대역폭이 훨씬 크다.
    
- **Specifications:** NVMe 1.4b | OCP NVMe Cloud SSD 1.0a  
    → NVMe 1.4b 규격과 OCP Cloud SSD 1.0a 규격을 따른다.  
    즉, **클라우드/데이터센터 환경 호환성**을 의식한 SSD라는 의미다.
    
- **FLASH Controller:** FADU FC4121  
    → FADU의 FC4121 컨트롤러를 사용한다는 뜻이다.  
    브로셔 관점에서는 “핵심 차별화 포인트가 컨트롤러 아키텍처에 있다”는 메시지를 주려는 부분이다.
    
- **SSD Designs:** E1.S | U.2 Form Factors  
    → 폼팩터는 **E1.S와 U.2** 두 가지다.  
    둘 다 데이터센터에서 많이 쓰이는 형태다.  
    이전 PCIe 3.0 이미지의 M.2/E1.S보다 더 **서버·엔터프라이즈 지향성**이 강해 보인다.
    
- **Capacities:** 1TB | 2TB | 4TB | 8TB  
    → 1, 2, 4, 8TB 용량 제공.  
    이전 예시보다 8TB까지 올라가 있어서, 더 본격적인 엔터프라이즈/데이터센터 라인업 느낌이 난다.
    

---

## 2. 성능 수치 해석

### SSD Performance Up To

- **Sequential Read:** 7,050 MB/s  
    → 순차 읽기 최대 7050MB/s  
    PCIe 4.0 x4 SSD답게 꽤 높은 수치다.  
    거의 인터페이스 상한에 근접하는 수준으로 보인다.
    
- **Sequential Write:** 4,200 MB/s  
    → 순차 쓰기 최대 4200MB/s  
    읽기보다 쓰기가 낮다. 일반적이다.  
    다만 이 수치가 어느 조건인지가 중요하다.
    
- **Random Read:** 1,350 KIOPS  
    → 랜덤 읽기 최대 135만 IOPS  
    매우 높은 편이다.  
    브로셔는 읽기 성능과 효율을 강하게 밀고 있는 것으로 보인다.
    
- **Random Write:**
    
    - **OP1:** 200 KIOPS
        
    - **OP2:** 390 KIOPS
        
    
    → 여기서 중요한 건 **왜 랜덤 쓰기가 두 개냐**다.  
    보통 이런 표기는 **Over-Provisioning(OP)** 조건이 다르다는 뜻일 가능성이 높다.  
    즉, 여유 공간을 더 주면 랜덤 쓰기 성능이 올라간다는 의미로 읽힌다.
    
    대략적으로 보면:
    
    - OP가 적은 조건: 200KIOPS
        
    - OP가 더 큰 조건: 390KIOPS
        
    
    이런 구조라면, 쓰기 성능이 **낸드 관리 방식, GC, 여유 공간, steady-state 상태**에 민감하다는 걸 간접적으로 드러낸다.
    

---

## 3. 전력 해석

### SSD Power Consumption

- **Active: <4.5W**
    
- **Idle: <3.5W**
    

이건 꽤 눈에 띈다.

보통 PCIe 4.0 데이터센터 SSD에서 성능이 높을수록 전력도 커지는데,  
여기서는 **“전력 대비 성능 효율”**을 핵심 메시지로 잡고 있다.

특히 Active가 4.5W 미만이라는 문구는 브로셔상 꽤 공격적인 주장이다.  
다만 반드시 확인해야 할 게 있다.

- Active가 **어떤 workload 기준인지**
    
- 순차 읽기인지, 랜덤 읽기인지, 쓰기인지
    
- QD가 얼마인지
    
- APST/Power State 설정이 어떤지
    
- E1.S와 U.2 둘 다 같은 수치인지
    

즉, 숫자 자체는 인상적이지만 **조건이 빠져 있으면 비교 근거는 약하다.**

---

## 4. Benefits 문구 해석

### 1) Twice the throughput of FADU’s PCIe 3.0 SSDs

- **FADU의 PCIe 3.0 SSD 대비 처리량이 2배**
    
- PCIe 3.0 → 4.0 세대 전환 효과를 강조한 문장이다.
    
- 자연스러운 마케팅 포인트다.  
    다만 여기서의 throughput이 정확히 **seq throughput인지, mixed workload throughput인지**는 불명확하다.
    

### 2) Industry-leading KIOPS/Watt with up to 25% lower power than other PCIe 3.0 SSDs

- **다른 PCIe 3.0 SSD보다 최대 25% 낮은 전력으로 업계 최고 수준의 KIOPS/Watt 제공**
    
- 핵심은 절대 성능보다 **성능/전력 효율**이다.
    
- 즉, FADU는 “더 빠르다”보다 **“전력당 처리량이 좋다”**를 강하게 내세우는 회사라는 인상을 준다.
    

### 3) Consistent, low latency for superior Quality of Service (QoS) up to 9x better than most industry leading SSDs

- **일관되고 낮은 지연시간으로 우수한 QoS를 제공하며, 대부분의 업계 선도 SSD보다 최대 9배 우수**
    
- 이 문장은 상당히 강한 주장이다.
    
- 그런데 여기서 가장 중요한 게 빠져 있다:
    
    - QoS를 무엇으로 정의했는지
        
    - 평균 지연인지
        
    - p99인지
        
    - p99.9인지
        
    - 어떤 workload인지
        
    - read/write mix가 무엇인지
        

즉, **브로셔 문장으로는 인상적이지만 기술 검증 관점에서는 가장 먼저 파고들 포인트**다.

### 4) Leading edge, trusted industry security standards

- **최신 수준의 신뢰할 수 있는 업계 보안 표준 지원**
    
- 좋은 말이지만, 너무 추상적이다.
    
- 실제로는 다음 같은 걸 떠올려야 한다.
    
    - TCG Opal
        
    - Secure Erase / Sanitize
        
    - Firmware Secure Boot
        
    - 암호화 기능
        
    - 인증/키 관리 방식
        
- 결국 “무슨 보안 표준을 어떤 모드에서 지원하느냐”가 중요하다.
    

---

# 5. 너가 전에 정리한 방식대로 다시 구조화하면

## A. 제품 포지션 / 세그먼트

- **PCIe 4.0 x4 + NVMe 1.4b + OCP NVMe Cloud SSD 1.0a**  
    → 명확하게 **데이터센터/클라우드 지향 SSD** 포지션이다.
    
- **E1.S | U.2**  
    → M.2보다 훨씬 더 서버/엔터프라이즈에 가까운 폼팩터 구성이다.
    
- **1TB~8TB**  
    → 엔터프라이즈 메인스트림에서 실사용 가능한 범위.  
    이전 PCIe 3.0 브로셔보다 데이터센터 제품답다.
    

즉, 이건 단순 소비자 SSD가 아니라  
**저지연, 효율, QoS를 강조하는 데이터센터용 SSD 플랫폼**으로 읽는 게 맞다.

---

## B. 성능의 상한(peak) 수준

- **Seq Read 7050 MB/s / Seq Write 4200 MB/s**  
    → PCIe 4.0 x4의 장점을 잘 활용하는 상한 성능
    
- **Random Read 1.35M IOPS**  
    → 읽기 처리량 매우 강함
    
- **Random Write 200K / 390K IOPS**  
    → 랜덤 쓰기는 OP 조건에 따라 차이가 큼  
    → 즉, 쓰기 성능은 **여유 공간, GC, steady-state 조건**의 영향을 크게 받는다고 해석 가능
    

여기서 보이는 패턴은:

- 읽기 성능은 매우 공격적으로 제시
    
- 쓰기 성능은 상대적으로 보수적이거나 조건 의존적
    
- 즉, **컨트롤러 아키텍처 강점은 저지연 읽기/QoS/효율 쪽**에 있을 가능성이 높다
    

---

## C. 전력/효율 주장 방향

이번 브로셔의 핵심은 더 분명하다.

이 제품은 단순히 “최대 성능”보다

- **KIOPS/Watt**
    
- **낮은 전력**
    
- **일관된 지연시간**
    
- **열 문제 억제**
    
- **QoS**
    

를 중심으로 판다.

즉, 메시지는 이거다:

> “피크 벤치 숫자만 높은 SSD가 아니라, 실제 데이터센터 운영에서 전력과 지연을 안정적으로 관리하는 SSD다.”

이건 FADU의 회사 포지션과도 잘 맞는다.

---

# 6. 기술 검증 관점에서 빠진 것들

이전처럼 여기서도 핵심은 같다.  
**브로셔 숫자는 peak claim이라서, 조건이 없으면 해석은 가능해도 검증은 안 된다.**

### 반드시 확인해야 할 것

- **IO size**  
    4K인지, 8K인지, 128K인지
    
- **QD (Queue Depth)**  
    QD1인지 QD32인지 QD128인지
    
- **numjobs**  
    병렬 job 수가 몇 개인지
    
- **read/write mix**  
    100% read인지, mixed인지
    
- **FOB vs steady-state**  
    초기 빈 상태에서 측정한 건지, 충분히 preconditioning 후 측정한 건지
    
- **preconditioning 절차**
    
- **QoS 정의**
    
    - 평균 latency?
        
    - p99?
        
    - p99.9?
        
- **전력 측정 조건**
    
    - Active가 어떤 workload 기준인지
        
- **폼팩터별 차이**
    
    - E1.S와 U.2는 발열/냉각/스로틀링 거동이 다를 수 있음
        
- **OP1 / OP2 정확한 의미**
    
    - Over-Provisioning 비율이 각각 몇 %인지
        

---

# 7. “각 단어 뜻”도 같이 정리

### Peak 조건

가장 잘 나오는 조건에서 측정한 최고치라는 뜻이다.  
실사용 평균이라고 보면 안 된다.

### QD (Queue Depth)

장치에 한 번에 걸어놓은 I/O 요청 개수다.  
QD가 높을수록 SSD는 성능을 더 잘 뽑는 경우가 많다.

### IO size

한 번에 읽거나 쓰는 데이터 블록 크기다.  
예: 4KB, 8KB, 128KB.  
랜덤 IOPS는 보통 4KB 기준이 많고, 순차 MB/s는 128KB나 큰 블록 기준이 많다.

### numjobs

동시에 돌리는 작업 수다.  
같은 QD라도 numjobs 구성이 다르면 성능/지연 특성이 달라질 수 있다.

### Steady-state

SSD가 충분히 쓰이고 GC도 발생하는, 말 그대로 **안정 상태**다.  
브로셔 성능은 종종 이 상태가 아니라 초기 깨끗한 상태에서 더 높게 나온다.

### Preconditioning

SSD를 실제 쓰기 압박 상태와 비슷하게 만들기 위해 미리 충분히 쓰는 절차다.  
steady-state 검증의 전 단계라고 보면 된다.

### QoS

여기서는 보통 **지연시간이 얼마나 일정하게 유지되는가**를 말한다.  
평균값보다 tail latency가 중요하다.

### p99 / p99.9 latency

전체 요청 중 느린 쪽 상위 1%, 0.1% 지연시간을 보는 지표다.  
데이터센터 QoS에서 매우 중요하다.  
평균 latency가 좋아도 p99.9가 나쁘면 실서비스 품질은 나쁠 수 있다.

### Thermal throttling

발열 때문에 SSD가 스스로 성능을 낮추는 현상이다.  
데이터센터 SSD에서는 매우 중요하다.

### OP (Over-Provisioning)

사용자에게 안 보이게 SSD 내부에 여유 공간을 더 두는 것이다.  
보통 쓰기 성능, GC 효율, 내구성, QoS 개선에 도움 된다.

---

# 8. 이 이미지에서 바로 던질 수 있는 문답 포인트

면접이나 기술 대화용으로 바꾸면 이런 질문이 자연스럽다.

- 순차 7050MB/s와 4200MB/s는 **어떤 block size / QD 조건**인가?
    
- 랜덤 읽기 1.35M IOPS는 **4K, QD 몇 기준**인가?
    
- 랜덤 쓰기 **OP1 200K / OP2 390K**에서 OP1과 OP2는 각각 **몇 % OP**인가?
    
- 랜덤 쓰기 수치는 **steady-state 결과인가, FOB 결과인가?**
    
- QoS 9배 우수라는 것은 **어떤 percentile latency 기준**인가?
    
- Active <4.5W는 **read / write / mixed 중 어떤 workload 기준**인가?
    
- E1.S와 U.2에서 **발열 및 throttling 차이**는 어떻게 나는가?
    
- Security standards는 구체적으로 **어떤 표준과 기능**을 의미하는가?

---
![[Pasted image 20260317214258.png]]

# 1. 표 내용 먼저 해석

## 기본 사양

- **Interface**: PCIe 4.0 x4
    
- **NVMe**: NVMe 1.4b
    
- **OCP Compliance**: OCP NVMe Cloud SSD 1.0a
    
- **Controller**: FADU FC4121
    
- **NAND**: SKH V6 128 Layer 4D eTLC
    
- **Form Factor**:
    
    - **U.2 (15mm)**
        
    - **E1.S (5.9 / 9.5 / 15 / 25mm)**
        

즉, PCIe 4.0 세대에서 플랫폼이 더 명확하게 **데이터센터용 U.2 / E1.S**로 정리되어 있다.

---

## 성능 표 구조

표는 **OP7**과 **OP28** 두 조건으로 나뉜다.

- **OP7**: 960 / 1920 / 3840 / 7680 GB
    
- **OP28**: 800 / 1600 / 3200 / 6400 GB
    

이건 사실상 같은 물리 NAND 자원에서  
**Over-Provisioning을 7%로 둘 때와 28%로 둘 때**의 usable capacity와 성능 차이를 보여주는 구조로 읽는 게 맞다.

즉,

- OP를 적게 주면 사용자 용량은 커지고
    
- OP를 많이 주면 사용자 용량은 줄지만 성능, 특히 랜덤 쓰기와 QoS가 좋아진다
    

이 표는 그걸 매우 노골적으로 보여준다.

---

# 2. PCIe 3.0 대비 바뀐 점 핵심

---

## A. “기능 소개형”에서 “성능/QoS 수치형”으로 이동

네가 정리한 3.0 자료에는 이런 것들이 있었다.

- Data-path E2E Protection
    
- Internal RAID
    
- AES-XTS SED
    
- Secure Boot
    
- TCG OPAL 2.01
    
- Multiple Namespace
    
- SMART / Health / Telemetry / OCP Log
    
- Latency Monitoring
    
- NVMe-MI
    
- PLP
    
- 512 / 4096 sector size support
    

즉, 3.0 문서는 **보안/무결성/운영 관리 기능 목록**이 많았다.

반면 이번 4.0 표는 그런 기능이 거의 안 보이고 대신:

- Random/Sequential 성능
    
- Latency
    
- QoS(99.9%)
    
- Power Consumption
    
- Reliability
    
- DWPD / Warranty
    

이런 **정량 표**가 중심이다.

### 해석

이건 기능이 사라졌다고 단정하면 안 된다.  
더 정확히는:

> 4.0 표는 기능 설명 브로셔가 아니라, 제품 스펙시트 요약판에 가깝다.

즉, 3.0에서 강조되던 보안/운영 기능이 **없어졌다기보다 이 페이지에 안 적힌 것**일 가능성이 높다.

---

## B. 성능 정보가 훨씬 구체화됨

3.0에서는 대략 이런 식이었다.

- Seq Read/Write
    
- Random Read/Write
    
- Power
    
- QoS 좋다, 효율 좋다
    

그런데 4.0 표는 훨씬 구체적이다.

### 추가된 구체성

- OP7 / OP28 조건 분리
    
- 용량별 성능 분리
    
- 랜덤 read/write latency 제시
    
- **QoS (99.9%) Random Read / Write latency** 제시
    
- **QD / IO size 조건을 Notes에 일부 명시**
    
- U.2 / E1.S 각각의 active read/write power 제시
    
- MTBF / UBER / Retention / DWPD / Warranty / Operating Temp 정리
    

즉, 4.0에서는 **단순 피크 성능 광고**에서 한 단계 나아가  
**데이터센터 고객이 궁금해할 정량 지표**를 넣기 시작했다.

이건 꽤 중요한 변화다.

---

## C. QoS가 말이 아니라 숫자로 내려옴

3.0에서는 “low latency”, “superior QoS” 같은 말이 중심이었다.  
그런데 4.0 표에서는 아예:

- **QoS (99.9%) Random Read (µs)**
    
- **QoS (99.9%) Random Write (µs)**
    

로 내려왔다.

예를 들면:

- Random Read QoS (99.9%): 85/330, 85/235, 85/200 ...
    
- Random Write QoS (99.9%): 100/1920, 25/740, 25/535 ...
    

이 표기는 문맥상 보통

- **read/write latency distribution의 두 값**
    
- 혹은 **평균/최악 또는 burst/sustained 식 벤더 내부 표기**  
    일 수 있는데, 정확한 정의는 추가 문서가 필요하다.
    

하지만 중요한 건 이거다.

### 의미

3.0에서는 QoS를 마케팅 문구로 팔았다면,  
4.0에서는 **tail latency를 숫자로 제시하기 시작했다.**

이건 데이터센터 SSD답게 한 단계 더 성숙한 표현이다.

---

## D. OP 차이에 따른 랜덤 쓰기 개선이 매우 선명해짐

### Random Write (KIOPS)

예를 들면 최대치 기준:

- **OP7**: 200 KIOPS
    
- **OP28**: 390 KIOPS
    

즉, OP를 많이 주면 랜덤 쓰기가 거의 2배 가까이 오른다.

이건 꽤 중요하다. 왜냐하면 이 표가 스스로 말하고 있기 때문이다.

> 이 SSD의 랜덤 쓰기/QoS는 여유 공간과 FTL/GC 정책의 영향을 강하게 받는다.

이건 오히려 검증 포인트를 잘 던져준다.

3.0에서는 그냥 숫자만 던졌다면,  
4.0에서는 **“조건이 바뀌면 성능이 어떻게 변하는지”**를 보여준다.

---

## E. 지연시간 수치가 직접 들어감

### Random Read Latency

- 70 µs
    

### Random Write Latency

- 20 / 15 / 15 / 15 µs 같은 식
    

여기서 약간 이상한 점도 있다.  
보통 랜덤 write latency가 read보다 더 불리하게 보이는 경우가 흔한데, 표기만 보면 write latency가 더 낮게 읽힌다.  
그래서 이 부분은 문맥 해석을 조심해야 한다.

가능한 해석은:

- 특정 정의의 latency일 수 있음
    
- 평균이 아니라 command completion 특정 구간일 수 있음
    
- read/write 행이 뒤바뀌었을 가능성은 낮지만 완전히 배제는 못 함
    
- 벤더 내부 기준의 표기법일 수 있음
    

즉, 그대로 외우기보다 **정의 확인이 필요하다**.

하지만 분명한 건,  
**4.0 자료는 latency를 “좋다”가 아니라 숫자로 내리기 시작했다**는 점이다.

---

## F. Power 정보가 폼팩터별로 정리됨

3.0에서는 power가 비교적 단순했다면,  
4.0에서는:

- **U.2 Active Write**
    
- **U.2 Active Read**
    
- **E1.S Active Write**
    
- **E1.S Active Read**
    
- **Idle <3.5W**
    

이렇게 분리되었다.

### 해석

이건 꽤 실무적이다.  
왜냐하면 데이터센터 SSD는 폼팩터가 다르면:

- 열 방출 조건
    
- 전력 제한
    
- 스로틀링 가능성
    
- 서버 섀시 공기 흐름
    

이 다 달라지기 때문이다.

즉, 4.0에서는 **폼팩터별 전력 특성**을 적어두며 더 실장 관점에 가까워졌다.

다만 여전히 부족한 건:

- power 측정 workload
    
- steady-state 여부
    
- APST / power state 설정
    
- ambient / airflow 조건
    

이다.

---

## G. 신뢰성 항목이 더 표준적인 데이터센터 지표로 정리됨

새 표에는 다음이 들어갔다.

- **MTBF**: 2.5M hours
    
- **UBER**: 1 sector per 10^17 read
    
- **Retention**: 3 months @ 40°C (EOL)
    
- **DWPD**: 0.85 / 1.0 / 3.0
    
- **Warranty**: 5 years
    
- **Operating Temperature**: 0~70°C
    

이건 3.0 때 네가 정리한 “기능성”과는 결이 다르다.  
이번에는 아예 **고객 조달/평가표에 들어갈 법한 신뢰성 수치**를 붙여놨다.

### 의미

- **UBER**: 읽기 중 복구 불가능 오류율 기준
    
- **Retention @ EOL**: 수명 끝 근처에서 데이터 보존 조건
    
- **DWPD**: 하루에 몇 번 전체 용량을 쓰는 workload까지 보증하는지
    
- **MTBF**: 고장 간 평균 시간
    

즉, 4.0 표는 **제품 기술/검증/고객 평가용 문서 냄새**가 더 강하다.

---

# 3. 3.0에서 보였는데 4.0 표에서는 안 보이는 것들

여기가 중요하다.  
“없어졌다”와 “이 페이지에 안 적혔다”를 구분해야 한다.

이번 표에서 **명시적으로 안 보이는 것들**:

## 보안/무결성

- Data-path E2E Protection (SECDED)
    
- Internal RAID
    
- Self Encrypting Drive (AES-XTS)
    
- Secure Boot
    
- TCG/OPAL 2.01
    

## 운영/관리

- Multiple Namespaces
    
- SMART / Health / Telemetry / OCP log fully support
    
- Latency Monitoring Feature
    
- NVMe-MI 1.0a
    
- PLP
    
- 512 / 4096 sector size support
    

### 해석

이게 의미하는 건 둘 중 하나다.

### 가능성 1

4.0 제품에도 당연히 있는데, 이 표는 성능 요약표라서 생략했을 가능성

이 가능성이 높다.  
특히 OCP NVMe Cloud SSD 1.0a를 적어놨다는 건 운영/로그/클라우드 호환 요구사항을 어느 정도 의식하고 있다는 뜻이기 때문이다.

### 가능성 2

일부 기능의 표기 방식을 바꿨거나, 세부 SKU별 지원 범위가 달라져서 이 페이지에서 뺐을 가능성

이것도 가능하다.  
예를 들어 보안 기능은 별도 security brief로 분리했을 수 있다.

즉, 실무적으로는 이렇게 봐야 한다.

> “4.0에서 그런 기능이 삭제됐다고 결론내리기보다, 이 스펙 표만으로는 확인 불가이므로 별도 spec/security/management 문서를 확인해야 한다.”

---

# 4. 바뀐 점만 압축하면

## 핵심 변화 1: 세대 업그레이드

- PCIe 3.0 x4 → **PCIe 4.0 x4**
    
- 컨트롤러도 **FC4121**
    
- NAND도 **SKH V6 128L 4D eTLC**
    
- 폼팩터도 **U.2 / E1.S 중심**
    

즉, 플랫폼이 더 명확하게 데이터센터 주력 제품으로 정리됨.

---

## 핵심 변화 2: 기능 설명보다 수치 표 중심

3.0은 “무엇을 지원하나” 중심,  
4.0은 “어떤 조건에서 얼마가 나오나” 중심.

---

## 핵심 변화 3: QoS/Latency가 정량화됨

- QoS(99.9%) 수치 직접 표기
    
- latency 수치 직접 표기
    
- 단순 성능 광고보다 데이터센터형 표현으로 이동
    

---

## 핵심 변화 4: OP 영향 공개

- OP7 / OP28 비교
    
- 용량과 성능 trade-off 공개
    
- 특히 random write와 QoS가 OP에 민감함을 드러냄
    

이건 컨트롤러/FTL/GC 이해와 연결되는 포인트다.

---

## 핵심 변화 5: 전력과 신뢰성 항목 정리 강화

- U.2 / E1.S별 전력 제시
    
- MTBF / UBER / DWPD / Retention / Warranty 추가
    
- 고객 평가표 스타일에 가까워짐
    

---

# 5. 네가 면접/분석에서 말할 때 쓸 수 있는 식으로 정리하면

이 표를 보고 이렇게 말할 수 있다.

> PCIe 3.0 자료가 보안, 무결성, 관리 기능 위주의 소개였다면, PCIe 4.0 자료는 성능, QoS, 전력, 신뢰성을 수치화한 스펙 테이블 형태로 발전했습니다. 특히 OP7과 OP28을 나눠 용량-성능 trade-off를 직접 보여주고, 99.9% QoS latency와 폼팩터별 power를 제시한 점에서 데이터센터 고객의 검증 포인트를 더 의식한 문서라고 볼 수 있습니다. 반면 3.0에서 강조되던 E2E protection, internal RAID, SED, secure boot, OPAL, telemetry, PLP 같은 기능은 이번 표에 직접 나타나지 않아서, 삭제 여부보다는 별도 문서에서 지원 범위를 재확인해야 한다고 보입니다.

이건 꽤 괜찮은 해석이다.

---

# 6. 이 표를 보고 바로 던질 질문

실제 검증/면접 관점에서는 이런 질문이 좋다.

- OP7 / OP28의 정확한 정의와 reserved area 비율은?
    
- QoS(99.9%) 행의 **두 개 숫자**는 각각 무엇을 의미하는가?
    
- Random read/write latency의 측정 정의는 평균인가, median인가, tail 기준인가?
    
- QoS 측정 조건이 **QD=1/64, 4KB**라고 적혀 있는데, 이 표기가 정확히 어떤 workload 구성인가?
    
- U.2와 E1.S의 power 수치는 동일 workload / 동일 thermal condition에서 측정된 것인가?
    
- PLP, Telemetry, NVMe-MI, Namespace, OPAL 같은 3.0 기능들은 4.0에서도 동일 지원인가?
    
- UBER, retention, DWPD는 각 capacity / OP mode에 따라 동일하게 적용되는가?
    

---

# 7. 결론

이번 4.0 문서는 3.0 대비 다음 방향으로 바뀌었다고 보면 된다.

- **세대 업그레이드**: PCIe 4.0, FC4121, 128L eTLC
    
- **제품 포지션 강화**: U.2 / E1.S 데이터센터 중심
    
- **기능 나열 → 정량 스펙화**
    
- **QoS/latency/power/reliability를 수치로 제시**
    
- **OP별 성능 차이를 공개해서 실제 운영 조건을 더 의식**
    
- 대신 **3.0의 보안/운영 기능 설명은 이 표에서 빠짐**
    

즉, 한 줄로 말하면:

> 3.0 자료가 “무엇을 지원하는 SSD인가”를 보여줬다면, 4.0 자료는 “데이터센터 조건에서 어느 정도의 성능·QoS·전력·신뢰성을 내는가”를 보여주는 쪽으로 무게중심이 이동했다.

---
![[Pasted image 20260317220741.png]]


---
#### 3.0에 비해 발전한 부분
PCIe 4.0 브로셔는 단순히 인터페이스만 3.0에서 4.0으로 올라간 버전이 아니라, 제품 포지션 자체가 더 전형적인 데이터센터 SSD 쪽으로 정리된 느낌이다. 3.0에서는 M.2와 E1.S를 같이 두며 메인스트림 엔터프라이즈 느낌이 강했는데, 4.0에서는 U.2와 E1.S 조합, 최대 8TB 용량, FC4121 컨트롤러, DELTA 플랫폼을 내세우면서 서버/스토리지 어레이용 라인업이라는 색이 더 짙어졌다.

성능도 단순 대역폭 증가에 그치지 않는다. 읽기는 PCIe 3.0의 3,400 MB/s급에서 PCIe 4.0의 7,050 MB/s급으로 올라가고, 랜덤 읽기도 800KIOPS에서 1,350 KIOPS로 상승했다. 특히 쓰기 쪽은 OP7/OP28을 분리해서 보여주며, over-provisioning 조건에 따라 랜덤 쓰기와 QoS가 얼마나 달라지는지 더 직접적으로 드러낸다. 이건 마케팅 문장이라기보다 validation 입장에서 오히려 질문거리를 더 많이 주는 변화다.

또 4.0 브로셔는 QoS, telemetry, latency monitoring, out-of-band management, multiple namespaces 같은 운영/관리 기능을 3.0 때보다 더 강하게 전면에 내세운다. 그래서 이번 세대는 “빠른 SSD”라기보다 “저지연·고효율·운영 가능성까지 챙긴 데이터센터 SSD”라는 방향으로 발전했다고 보는 게 맞아 보인다. 다만 여전히 이 수치들이 steady-state인지, workload 조건이 무엇인지, QoS 9x 개선이 정확히 어떤 비교 기준인지 같은 부분은 브로셔만으로는 부족해서 추가 확인이 필요하다.