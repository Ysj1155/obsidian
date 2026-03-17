---
tags:
  - reference
  - scrap
  - processed
  - ssd
  - validation
  - enterprise-ssd
---

# Fadu PCIe 4.0 SSD 브로셔

## 문서 정보
- 출처: FADU PCIe 4.0 SSD Brochure
- 제품군: Datacenter SSD / Storage Platform DELTA
- 인터페이스: PCIe 4.0 x4
- 규격: NVMe 1.4b, OCP NVMe Cloud SSD 1.0a
- 컨트롤러: FADU FC4121
- 폼팩터: E1.S, U.2
- 용량: 1TB, 2TB, 4TB, 8TB

---

## 한 줄 요약
FADU의 PCIe 4.0 SSD는 데이터센터용 엔터프라이즈 SSD 플랫폼으로, **저전력·낮은 지연·QoS·관리 기능·보안 기능**을 강점으로 내세운다.

---

## 브로셔가 강조하는 핵심 포인트

### 1. 단순 최대 성능보다 **낮은 지연과 QoS**
브로셔는 단순 대역폭 수치뿐 아니라 **ultra-low and consistent latency**, **superior QoS**를 반복해서 강조한다.  
즉 이 제품은 “최대치가 높은 SSD”라기보다, **데이터센터에서 예측 가능한 응답시간을 유지하는 SSD**라는 메시지에 가깝다.

- 최대 순차 읽기: 7,050 MB/s
- 최대 순차 쓰기: 4,200 MB/s
- 최대 랜덤 읽기: 1,350 KIOPS
- 최대 랜덤 쓰기: OP7 200 KIOPS / OP28 390 KIOPS

### 2. **KIOPS/Watt** 중심의 효율 강조
브로셔는 전력 효율을 강하게 내세운다.  
Active 전력은 <14.5W, Idle은 <3.5W로 표기되어 있고, “industry-leading KIOPS/Watt”를 주요 장점으로 둔다.  
즉 데이터센터 관점에서 성능 자체뿐 아니라 **전력당 처리량**이 핵심 가치다.

### 3. 데이터센터용 관리 기능이 많다
이 SSD는 단순 저장장치가 아니라, **운영/관찰/장애 대응이 가능한 장치**로 보인다.

주요 기능:
- SMART / Health Log / Telemetry Log
- Latency Monitoring
- Out-of-Band Management
- Multiple Namespaces (최대 128)
- Multiple Sector Size Support (512 / 4096)

### 4. 보안/무결성 기능이 별도 축으로 강조된다
보안 기능을 별도 페이지로 뺀 점이 중요하다.

- Data-path E2E Protection (SECDED)
- Internal RAID
- Self Encrypting Drive (AES-XTS)
- Secure Boot
- TCG / TCG OPAL 2.01

### 5. PLP가 명시되어 있다
브로셔는 **Power Loss Protection (PLP)** 를 데이터센터 기능으로 직접 명시한다.  
즉 전원 장애 상황에서도 쓰기 중 데이터 유실을 줄이는 보호 설계가 들어간 제품군이라는 뜻이다.

---

## 페이지 2 스펙표에서 봐야 할 것

### 인터페이스 / 규격
- PCIe 4.0 x4
- NVMe 1.4b
- OCP NVMe Cloud SSD 1.0a

### NAND / 컨트롤러
- Controller: FADU FC4121
- NAND: SKH V6 128 Layer 4D eTLC

### 폼팩터
- U.2 (15mm)
- E1.S (5.9mm / 9.5mm / 15mm / 25mm)

### 성능 수치 해석 시 주의
브로셔 성능 수치는 조건이 함께 적혀 있다.

- Sequential: QD=128, IO Size=128KB
- Random: QD=128, IO Size=4KB
- QoS(99.9%): QD=1/64, IO Size=4KB

즉 이 수치를 볼 때는 **테스트 조건이 무엇인지 같이 봐야** 한다.  
브로셔의 최대 성능 숫자만 외우는 건 의미가 작다.

### Reliability / Endurance
- MTBF: 2.5M hours
- UBER: 1 sector per 10^17 read
- Retention: 3 months @ 40°C (EOL)
- Warranty: 5 years
- DWPD: 0.85 / 1.0 / 3.0

여기서 DWPD가 하나의 고정값이 아니라 조건/모델에 따라 달라진다는 점을 봐야 한다.

---

## 보안 기능 해석

### Data-path E2E Protection (SECDED)
호스트에서 SSD 저장 매체까지 데이터 경로 전체에서 오류를 검출/보정해 **전송 중 데이터 무결성**을 높이는 기능.

### Internal RAID
SSD 내부에서 중복성을 두어 데이터 보호성을 높이는 기능.  
일반 RAID와 완전히 같은 개념이라기보다, **내부 NAND 장애 대응용 보호 계층**으로 봐야 한다.

### AES-XTS SED
암호화를 소프트웨어가 아니라 SSD 내부에서 처리해, **성능 저하를 최소화하면서 저장 데이터 보호**를 노리는 기능.

### Secure Boot
부팅 시 악성 펌웨어/변조 코드 로딩을 막기 위한 기능.

### TCG OPAL 2.01
기업 환경에서 요구되는 저장장치 보안 표준 지원.

---

## 데이터센터 기능 해석

### Multiple Namespaces
하나의 SSD를 논리적으로 여러 공간으로 나눠 쓸 수 있게 하여, 멀티테넌트/분리 운영에 유리하다.

### SMART / Health / Telemetry
운영 중 상태 감시, 장애 분석, 수명 추적에 중요하다.  
즉 “고장 난 뒤 확인”이 아니라 **운영 중 관찰 가능성**을 높이는 기능이다.

### Latency Monitoring
평균 속도보다 tail latency가 더 중요한 환경에서 유용하다.  
QoS와 함께 봐야 한다.

### Out-of-Band Management
NVMe-MI, MCTP over SMBUS, MCTP over PCIe VDM 지원은, SSD를 OS 안쪽에서만 보는 게 아니라 **외부 관리 경로로도 다룰 수 있다**는 의미다.

### PLP
전력 손실 시 쓰기 중 데이터와 메타데이터 보호를 위한 핵심 기능.  
엔터프라이즈 SSD에서 중요한 이유가 여기에 있다.

### 512 / 4096 Sector 지원
플랫폼, OS, 워크로드 호환성 측면에서 중요한 항목이다.

---

## 이 브로셔를 읽고 떠올려야 할 검증 질문

### 성능 / QoS
- 최대 성능 수치는 어떤 QD / IO size 조건에서 측정했는가?
- steady-state에서도 QoS가 유지되는가?
- 랜덤 쓰기 성능이 OP7과 OP28에서 왜 차이나는가?
- tail latency(99.9%)는 부하 증가 시 어떻게 변하는가?

### 전력
- Active Write/Read 전력은 용량별로 어떻게 달라지는가?
- KIOPS/Watt를 실제 테스트에서 어떻게 재현할 것인가?
- 열 누적 시 thermal throttling이 정말 거의 없는가?

### 신뢰성
- PLP는 어떤 시나리오에서 어떻게 검증할 것인가?
- retention 3 months @ 40°C (EOL)의 의미를 어떻게 해석해야 하는가?
- DWPD 0.85 / 1.0 / 3.0은 모델/조건별로 어떻게 나뉘는가?

### 운영 / 관리
- Telemetry log에서 어떤 항목을 실제로 볼 수 있는가?
- Latency monitoring은 호스트 툴과 어떻게 연계되는가?
- OOB management는 실제 서버 환경에서 어떤 장점을 주는가?

### 보안
- Secure Boot 검증은 어떤 실패/변조 시나리오로 볼 수 있는가?
- AES-XTS SED와 TCG OPAL은 실제 배포 환경에서 어떻게 구성되는가?
- Data-path E2E protection은 어떤 종류의 오류를 막는가?

---

## 내 기존 노트와 연결
- [[SSD 허브]]
- [[엔터프라이즈 SSD]]
- [[SSD 전력 손실 보호 원리]]
- [[SSD 단계적 검증 테스트 방법 및 저장장치]]
- [[SSD 기본 원리와 구조]]

추가 예정:
- [[SSD Garbage Collection]]
- [[SSD Wear Leveling]]

---

## 이 브로셔를 통해 얻은 관점
이 문서는 SSD를 “빠른 저장장치”로 설명하지 않는다.  
오히려 데이터센터 SSD를 **QoS, 전력 효율, 모니터링, 보안, 전원 장애 대응**까지 포함한 운영 가능한 시스템 부품으로 설명한다.  
따라서 이 브로셔를 읽을 때는 순차/랜덤 최대치보다, **왜 이런 기능이 데이터센터에서 필요한가**를 중심으로 정리하는 편이 더 가치 있다.

---

## 출처에서 직접 확인할 핵심 숫자
- Interface: PCIe 4.0 x4
- NVMe: 1.4b
- Controller: FC4121
- Max Seq Read: 7,050 MB/s
- Max Seq Write: 4,200 MB/s
- Max Random Read: 1,350 KIOPS
- Max Random Write: OP7 200 KIOPS / OP28 390 KIOPS
- Idle Power: <3.5W
- MTBF: 2.5M hours
- UBER: 1 sector per 10^17 read
- Warranty: 5 years