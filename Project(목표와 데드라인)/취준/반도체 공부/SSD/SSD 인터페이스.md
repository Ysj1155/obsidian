---
tag: " "
tags: ssd
---
[출처1](https://blog.naver.com/buneed_/223111629562)
[출처2](https://www.corsair.com/kr/ko/explorer/diy-builder/storage/whats-the-difference-between-m2-pcie-sata-and-nvme-for-ssds/?srsltid=AfmBOopLgVGvwebgbxoo_72QphQ0HgzTV1CElXH1AA09f42V92bQyGL4)

### SATA
SSD의 물리적 인터페이스. PCI-Express의 약자. 다양한 주변 기기를 컴퓨터 시스템과 통신할 수 있도록 제어하고 데이터 전송을 담당한다. 예전에는 메인보드 슬롯에 꼽는 카드 형태의 장치에 사용했기 때문에 PCIe는 카드 형태의 폼팩터로 말하기도 하며 이때는 PCLe로 표기한다. 
인터페이스 자체를 뜻할 때도 있다. 이 때는 PCI-E 4.0과 같이 버전이나 PCIe4.0x4처럼 기본 속도에 4배속이 가능하다는 뜻으로 배속을 같이 표기한다. 

### PCIe
SSD를 PC에 연결하기 위한 물리적 연결 방식인 두 가지 주요 종류의 SSD 인터페이스. 
SATA는 직렬 고급 기술 부착의 약자로, 더 오래된 기술이다. PCIe보다 더 폭넓은 호환성을 제공하지만, 성능 면에서는 한계를 보인다. 인터페이스의 이론적 최대 처리량은 약 550MB/s이며, 최신 PCIe 5.0 드라이브는 최대 14,000MB/s를 제공할 수 있다. 재 시중에는 두 가지의 일반적인 물리적 SATA 커넥터가 있다. M.2와 SATA. M.2 드라이브는 소켓에 바로 꽂을 수 있지만, SATA 커넥터를 사용하는 드라이브는 SATA 케이블이 필요하다. 
PCIe; 주변 장치 인터커넥트 익스프레스. 이 인터페이스는 상당히 복잡하기 때문에 가능한 한 PCIe로 줄여서 부른다. PCIe는 최대 4개의 PCIe 레인에서 데이터를 전송할 수 있는 반면, SATA는 단 하나의 레인으로 제한된다. 본질적으로 처리량 측면에서 최신 PCIe 표준이 우위에 있다는 것을 의미한다. PCIe는 세대를 거듭할 때마다 인터페이스가 지속적으로 개선되어 매번 처리량이 거의 두 배로 증가한다. 

| **인터페이스**    | **x1 레인**   | **x2 레인**   | **x4 레인**   |
| ------------ | ----------- | ----------- | ----------- |
| **PCIe 5.0** | 4,000MB/sec | 8,000MB/sec | 초당 16,000MB |
| **PCIe 4.0** | 초당 2,000MB  | 4,000MB/sec | 8,000MB/sec |
| **PCIe 3.0** | 1,000MB/sec | 초당 2,000MB  | 4,000MB/sec |
| **SATA**     | 초당 550MB    | -           | -           |

### 레인(x1/x2/x4), 세대(Gen3/4/5)

### 폼펙터와의 관계: M.2, U.2, E1.S


- SATA SSD는 보통 AHCI 사용
- PCIe SSD는 보통 NVMe 사용