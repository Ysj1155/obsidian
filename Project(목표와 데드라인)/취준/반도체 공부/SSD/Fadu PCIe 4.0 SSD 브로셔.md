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


#### 3.0에 비해 발전한 부분
PCIe 4.0 브로셔는 단순히 인터페이스만 3.0에서 4.0으로 올라간 버전이 아니라, 제품 포지션 자체가 더 전형적인 데이터센터 SSD 쪽으로 정리된 느낌이다. 3.0에서는 M.2와 E1.S를 같이 두며 메인스트림 엔터프라이즈 느낌이 강했는데, 4.0에서는 U.2와 E1.S 조합, 최대 8TB 용량, FC4121 컨트롤러, DELTA 플랫폼을 내세우면서 서버/스토리지 어레이용 라인업이라는 색이 더 짙어졌다.

성능도 단순 대역폭 증가에 그치지 않는다. 읽기는 PCIe 3.0의 3,400 MB/s급에서 PCIe 4.0의 7,050 MB/s급으로 올라가고, 랜덤 읽기도 800KIOPS에서 1,350 KIOPS로 상승했다. 특히 쓰기 쪽은 OP7/OP28을 분리해서 보여주며, over-provisioning 조건에 따라 랜덤 쓰기와 QoS가 얼마나 달라지는지 더 직접적으로 드러낸다. 이건 마케팅 문장이라기보다 validation 입장에서 오히려 질문거리를 더 많이 주는 변화다.

또 4.0 브로셔는 QoS, telemetry, latency monitoring, out-of-band management, multiple namespaces 같은 운영/관리 기능을 3.0 때보다 더 강하게 전면에 내세운다. 그래서 이번 세대는 “빠른 SSD”라기보다 “저지연·고효율·운영 가능성까지 챙긴 데이터센터 SSD”라는 방향으로 발전했다고 보는 게 맞아 보인다. 다만 여전히 이 수치들이 steady-state인지, workload 조건이 무엇인지, QoS 9x 개선이 정확히 어떤 비교 기준인지 같은 부분은 브로셔만으로는 부족해서 추가 확인이 필요하다.