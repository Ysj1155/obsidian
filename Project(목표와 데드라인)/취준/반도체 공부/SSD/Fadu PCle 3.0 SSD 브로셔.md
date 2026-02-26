[출처](obsidian://open?vault=obsidian_clean&file=Project(%EB%AA%A9%ED%91%9C%EC%99%80%20%EB%8D%B0%EB%93%9C%EB%9D%BC%EC%9D%B8)%2F%EC%B7%A8%EC%A4%80%2FFADU%2F20250509162324_FADU_PCIe3.0_SSD_Brochure.pdf)

Turnkey Storage Solution with FLASH Controller, Customizable Firmware, and SSD Designs.

우리는 컨트롤러도 있고, 펌웨어도 우리 손에 있고, SSD 하드웨어 설계까지 묶어서 고객이 빠르게 SSD를 만들 수 있게 해준다.

FADU’sPCIe 3.0 NVMe SSDsaredesigned to meetthe increasing demands placed on Hyperscaler, Hyper converged, Enterprise, and Edge data centers. At the heart of FADU’s SSDs is an innovative SSD controller architecture that enables ultra-low and consistent latency while virtually eliminating thermal throttling issues. As a result, FADU SSDs deliver industry leading KIOPS/Watt performance while supporting superior QoS. In the industry’s first E1.S form factor SSD, FADU’s PCIe 3.0 SSD consumes up to 30% less power and operates up to twice as fast as other PCIe 3.0 SSDs. Consistent low-latency delivers stable, superior Quality of Service (QoS) at any workload. The SSDs support a variety of features for modern data centers, including hardware-based security, advanced telemetry, virtualization functions, data path, and power loss protection.

AWS/Google/Microsoft 같은 초대형 데이터센터인 Hyperscaler, 스토리지/컴퓨트/네트워크를 소프트웨어로 통합한 인프라(HCI)인 Hyperconverged, 일반 기업 데이터 센터 용도의 Enterprise, 데이터가 생성되는 현장 가까운 소형/분산 데이터센터인 Edge data center를 대상으로 설계. PC용이 아니라 데이터 센터용(엔터프라이즈 SSD) 포지셔닝.
컨트롤러 아키텍쳐가 혁신적이기 때문에 지연시간이 낮고 일관적이다(ultra-low and consistent latency). 열 때문에 성능이 떨어지는 현상(thermal throttling)을 줄였다.
KIOPS/Watt 성능 업계 최고(같은 전력에서 더 많은 IOPS를 낸다고 주장). superior Qos(지연시간 분포(p99/p99.9)) 지원. -> 전력 효율 + Qos가 핵심 가치다. 
업계 최초 E1.S 폼팩터 SSD. PCLE 3.0 SSD 대비 전력 30% 덜 먹고, 속도 2배 빠르다 주장. -> ㅣㅂ교 대상, 측정 조건(워크로드/QD/온도/방열)이 없어 기준 모호.
데이터 센터 기능 체크리스트로 현대 데이터센터에 필요한 기능을 지원한다. hardware-based security(하드웨어 보안), advanced telemetry(상태/로그/모니터링 데이터), virtualization functions(가상화 관련 기능), data path(데이터 경로 최적화/기능), power loss protection(PLP: 전원 꺼져도 데이터/메타데이터 보호). 성능만이 아니라 운영/관리/보안/신뢰성 기능이 있다.
업계 표준인 NVMe 1.3, PCLe 3.0 x4, OPC NVMe Cloud SSD 1.0을 따르면서 데이터센터(특히 클라우드)에서 요구하는 규격/호환성 틀 안에 있다고 주장.
