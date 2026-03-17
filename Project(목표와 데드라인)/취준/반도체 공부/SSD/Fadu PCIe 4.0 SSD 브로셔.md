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


---
![[Pasted image 20260317214258.png]]
#### 3.0에 비해 발전한 부분
PCIe 4.0 브로셔는 단순히 인터페이스만 3.0에서 4.0으로 올라간 버전이 아니라, 제품 포지션 자체가 더 전형적인 데이터센터 SSD 쪽으로 정리된 느낌이다. 3.0에서는 M.2와 E1.S를 같이 두며 메인스트림 엔터프라이즈 느낌이 강했는데, 4.0에서는 U.2와 E1.S 조합, 최대 8TB 용량, FC4121 컨트롤러, DELTA 플랫폼을 내세우면서 서버/스토리지 어레이용 라인업이라는 색이 더 짙어졌다.

성능도 단순 대역폭 증가에 그치지 않는다. 읽기는 PCIe 3.0의 3,400 MB/s급에서 PCIe 4.0의 7,050 MB/s급으로 올라가고, 랜덤 읽기도 800KIOPS에서 1,350 KIOPS로 상승했다. 특히 쓰기 쪽은 OP7/OP28을 분리해서 보여주며, over-provisioning 조건에 따라 랜덤 쓰기와 QoS가 얼마나 달라지는지 더 직접적으로 드러낸다. 이건 마케팅 문장이라기보다 validation 입장에서 오히려 질문거리를 더 많이 주는 변화다.

또 4.0 브로셔는 QoS, telemetry, latency monitoring, out-of-band management, multiple namespaces 같은 운영/관리 기능을 3.0 때보다 더 강하게 전면에 내세운다. 그래서 이번 세대는 “빠른 SSD”라기보다 “저지연·고효율·운영 가능성까지 챙긴 데이터센터 SSD”라는 방향으로 발전했다고 보는 게 맞아 보인다. 다만 여전히 이 수치들이 steady-state인지, workload 조건이 무엇인지, QoS 9x 개선이 정확히 어떤 비교 기준인지 같은 부분은 브로셔만으로는 부족해서 추가 확인이 필요하다.