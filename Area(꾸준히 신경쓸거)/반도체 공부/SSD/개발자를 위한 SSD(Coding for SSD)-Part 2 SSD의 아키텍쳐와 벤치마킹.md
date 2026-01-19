---
aliases:
---
[카카오 테크](https://tech.kakao.com/posts/327)

![[Pasted image 20260119143804.png]]

# 1. SSD의 구조
## 1.1. NAND 플래시 메모리 셀
SSD(Solid-State Drive)는 플래시 메모리를 기반으로한 저장 매체이다. 비트들은 Floating-Gate 트랜지스터로 구성된 셀에 저장된다.SSD는 모든 컴포넌트가 전기 장치이며, HDD와 같은 기계 장치를 가지고 있지 않다.

Floating-Gate 트랜지스터에 전압이 가해지면서 셀의 비트가 쓰여지거나 읽혀지게 된다.트랜지스터는 NOR 플래시 메모리와 NAND 플래시 메모리 두 종류가 있는데(여기에서는 NAND의 NOR 플래시 메모리의 차이에 대해서 깊이 있게 살펴보지는 않을 것이다),이 게시물에서는 많은 제조사들이 채택하고 있는NAND 플래시 메모리에 대해서만 살펴보도록 하겠다. 

NAND 플래시 메모리 모듈의 중요한 속성은 수명이 제한적(Wearing-off)이라는 것이다.트랜지스터는 셀에 전자를 저장하면서 쓰기를 하게 되는데,매번 P/E(Program & Erase, Program은 쓰기를 의미) 사이클마다 일부 전자가 오류로 인해서 트랜지스터에 갇히게 된다.그리고 이렇게 갇힌 전자들이 쌓여서 일정 수준을 넘어서게 되면, 그 셀은 사용 불가능한 상태가 되는 것이다.

> ##### 수명 제한
> 
> 각 셀은 최대 P/E 사이클을 가지는데, 이 사이클을 넘어서면 결함 셀로 간주된다.NAND 플래시 메모리는 수명 제한을 가지는데, 이 수명 제한은 NAND 플래시 메모리의 타입(SLC, MLC, TLC)에 따라서 조금씩 차이가 있다[1](https://tech.kakao.com/posts/327#fn:31).

최근 연구에 따르면, NAND 플래시 메모리 칩에 높은 열이 가해지면 각 셀에 갇혀있던(프로그램되어 있던) 전자들이 사라진다는 것이 확인되었다[2](https://tech.kakao.com/posts/327#fn:14) [3](https://tech.kakao.com/posts/327#fn:51).또한 아직 연구중이며 언제 시중에 이런 제품이 출시될지 모르지만, SSD의 수명을 상당히 높일 수 있는 방법도 있다.현재 시중에 출시되는 SSD의 메모리 셀 타입은:

- SLC (Single level cell), SLC 트랜지스터는 단 하나의 비트만 저장할 수 있지만 상대적으로 긴 수명을 가짐
- MLC (Multiple level cell), MLC 트랜지스터는 2 비트를 저장할 수 있으며, SLC에 비해서 상대적으로 레이턴시(응답 속도)가 높고 짧은 수명을 가짐
- TLC (Triple-level cell), TLC 트랜지스터는 3 비트를 저장할 수 있지만, MLC보다 레이턴시가 높고 수명이 더 짧음

> ##### 메모리 셀 타입
> 
> SSD는 플래시 메모리 기반의 저장 매체이다. 비트는 셀에 저장되며, 메모리 셀은 3가지 타입이 있다. SLC는 1비트, MLC는 2비트 그리고 TLC는 3비트를 저장할 수 있다.

표1은 각 NAND 플래시 셀 타입의 상세한 정보를 보여주고 있다. 비교를 위해서 HDD와 메인 메모리(RAM) 그리고 L1/L2 캐시도 같이 포함하였다.
![[Pasted image 20260119144146.png]]
표1: 다른 메모리 컴포넌트와 NAND 플래시 메모리의 특성 및 레이턴시 비교

- Notes
    - `*` metric is not applicable for that type of memory
- Sources
    - P/E cycles [4](https://tech.kakao.com/posts/327#fn:20)
    - SLC/MLC latencies [5](https://tech.kakao.com/posts/327#fn:1)
    - TLC latencies [6](https://tech.kakao.com/posts/327#fn:23)
    - Hard disk drive latencies [7](https://tech.kakao.com/posts/327#fn:18) [8](https://tech.kakao.com/posts/327#fn:19) [9](https://tech.kakao.com/posts/327#fn:25)
    - RAM latencies [10](https://tech.kakao.com/posts/327#fn:30) [11](https://tech.kakao.com/posts/327#fn:52)
    - L1 and L2 cache latencies [11](https://tech.kakao.com/posts/327#fn:52)

동일 량의 트랜지스터로 더 많은 비트들을 저장할 수 있다면, 당연히 제조 단가는 낮아지게 된다.SLC 타입의 SSD는 MLC 타입보다 신뢰성이 높고 더 오랜 시간 사용할 수 있지만, 제조 비용이 크다.그래서 대 부분의 SSD는 MLC 또는 TLC 타입이며, 엔터프라이즈급의 SSD만 SLC를 사용한다.워크로드에 맞게 적절한 메모리 타입을 선정하는 것이 중요한데, 여기서 워크로드라 함은 얼마나 SSD에 데이터가 자주 기록되는지를 의미한다.예를 들어서 쓰기 빈도가 아주 높다면 SLC 타입의 SSD가 최선이며,(동영상 스트리밍을 위한 저장소와 같이) 쓰기는 많지 않지만 읽기가 매우 많다면 TLC가 가장 적절한 선택이 될 것이다.게다가 실제 서비스 수준의 워크로드로 수행된 벤치마크의 결과에 의하면, 실제 TLC 타입의 메모리 수명은 크게 문제되지 않는 것으로 보인다 [12](https://tech.kakao.com/posts/327#fn:36).

> ##### NAND 플래시 페이지와 블록
> 
> 셀들은 Block으로 그룹핑 되어 있으며, Block들은 다시 Plane으로 그룹핑되어 있다.SSD를 읽고 쓰기시에 최소 접근 단위는 Page라고 하며, Page는 개별로 삭제(Erase)될 수 없고 반드시 삭제는 Block 단위로만 수행될 수 있다.NAND 플래시 메모리의 Page 크기는 2KB, 4KB, 8KB, 16KB로 다양한데, 대 부분 SSD의 Block은 128개 또는 256개의 Page를 가진다.이는 SSD 제조사별로 Block의 크기가 256KB에서 4MB까지 다양하다는 것을 의미한다.예를 들어서 삼성 SSD 840 EVO는 2048KB의 Block 크기를 가지며 각 Block은 256개의 8KB 페이지를 가진다.Page와 Block의 접근 방법에 대해서는 섹션 3.1에서 자세히 살펴보도록 하겠다.

## 1.2. SSD의 구성

아래의 그림 1은 SSD의 주요 components를 도식화한 것이다.
![[Pasted image 20260119145133.png]]
사용자의 요청은 호스트 인터페이스를 통해서 유입되는데,이 글을 작성하는 시점에는 가장 일반적인 인터페이스 방식은 ATA(SATA)와 PCI Express(PCIe) 타입이다.SSD 컨트롤러에 장착된 프로세서가 명령을 받아서 플래시 컨트롤러로 전달하게 된다.SSD는 자체적으로 보드에 내장된 메모리(RAM)을 가지는데, 일반적으로 이 메모리는 맵핑 정보를 저장하거나 캐시 용도로 사용된다.상세한 맵핑 정책에 대해서는 섹션 4에서는 살펴보도록 하겠다.SSD는 여러 개의 Channel을 통해서 서로 NAND 플래시 메모리 패키지로 구성된다.Channel에 대한 상세한 내용은 섹션 6에서 살펴보도록 하겠다.
아래의 그림 2와 3은 실생활에서 사용되는 SSD가 어떻게 생겼는지를 보여주고 있다.그림 2는 2013년 8월에 출시된 512GB 삼성 840 Pro SSD이다. 기판에서 보이듯이, 주요 컴포넌트는:
- 1 SATA 3.0 인터페이스
- 1 SSD 컨트롤러 (삼성 MDX S4LN021X01-8030)
- 1 RAM 모듈 (256 MB DDR2 삼성 K4P4G324EB-FGC2)
- 8 MLC NAND-플래시 모듈, 각 모듈은 64 GB (삼성 K9PHGY8U7A-CCK0)
![[Pasted image 20260119152120.png]]
그림 3은 2013년 후반기에 출시된 마이크론(Micron) P420m Enterprise PCIe 이며, 주요 컴포넌트는:
- 8 레인(lane) PCI Express 2.0 인터페이스
- 1 SSD 컨트롤러
- 1 RAM 모듈 (DRAM DDR3)
- 64 MLC NAND-플래시 모듈(32 채널), 각 모듈은 32GB (마이크론 31C12NQ314 25nm)
전체 메모리 공간은 2048GB이지만, 프로비저닝(over-provisioning) 영역을 제외하고 1.4TB 사용 가능.
![[Pasted image 20260119152138.png]]

## 1.3. SSD의 생산 공정
많은 SSD 제조사는 SSD 생산을 위해서 SMT(Surface-Mount Technology)를 사용하는데,SMT 생산 과정에서는 전자 부품들이 회로 기판(PCBs)위에 직접 장착된다. SMT 생산 라인은 기계들이 줄지어 있고, 각 기계들이 부품을 기판위에 놓거나 놓여진 부품들을 납땜하는 등의 각 작업들을 담당하게 된다. 전체 생산 공정에서 여러번의 품질 체크 과정도 수행된다. 
[RAM & SSD & 메인 보드는 어떻게 만들어 지는가?](https://www.youtube.com/watch?v=3s7KG6QwUeQ)

## 2. 벤치마킹과 성능 메트릭
### 2.1. 기본 벤치마킹
아래의 표2는 여러 SSD 드라이브들의 랜덤과 시퀀셜 워크로드에서 스루풋을 보여주고 있다.비교를 위해서 2008년과 2013년에 출시된 SSD를 HDD와 메모리(RAM)과 함께 나열해 보았다.
![[Pasted image 20260119152640.png]]
Notes
- metric is not applicable for that storage solution
- measured with 2 MB chunks, not 4 KB
Metrics
- MB/s: Megabytes per Second
- KIOPS: Kilo IOPS, i.e 1000 Input/Output Operations Per Second
Sources
- Samsung 64 GB
- Intel X25-M
- Samsung SSD 840 EVO
- Micron P420M
- Western Digital Black 4 TB
- Corsair Vengeance DDR3 RAM
호스트 인터페이스는 성능을 결정하는 중요한 요소중 하나이다. 최근에 출시되는 SSD의 일반적인 인터페이스는 SATA 3.0 또는 PCI Express 3.0이다. SATA 3.0 인터페이스는 6 Gbit/s 정도의 데이터 전송량을 낼 수 있는데, 이는 초당 550MB 정도의 성능이다. 그리고 PCIe 3.0 인터페이스에서는 레인(lane)당 8 GT/s(GT/s 는 초당 전송가능한 Giga를 의미, Gigatransfers/second) 데이터 전송이 가능한데,이는 대략 1 GB/s 정도이다. PCIe 3.0 인터페이스는 최소 1개 이상의 레인(lane)을 가지고 있는데, 4개 레인을 가진 SSD는 SATA 3.0 인터페이스보다 8배나 빠른 4 GB/s 전송 속도를 낼 수 있다. 일부 엔터프라이즈 SSD중에는 SAS (Serial Attached SCSI) 인터페이스를 장착한 제품들도 있는데, 이들은 12 GBit/s정도의 전송 속도를 제공하지만 실제 SAS 인터페이스를 장착한 제품은 많지 않다. 
최근 출시되는 대 부분의 SSD들은 SATA 3.0의 최대 전송 속도인 550MB/s를 초과할 정도의 성능을 가지고 있으므로, 현재는 대부분 인터페이스가 병목 지점인 경우가 많다. PCI Express 3.0이나 SAS 인터페이스를 장착한 SSD는 엄청난 성능 향상을 제공할 것이다. SATA보다 빠른 PCI Express 와 SAS 인터페이스 제조사에서 주로 사용되는 호스트 인터페이스는SATA 3.0 (550 MB/s)과 PCI Express 3.0 (레인당 1 GB/s, 다중 채널 사용)이다. SAS(Serial Attached SCSI) 또한 엔터프라이즈 SSD에 사용되는데, PCIe와 SAS 인터페이스는 SATA보다 훨씬 빠른 성능을 제공하지만, 그만큼 가격도 비싼 편이다.

### 2.2. 프리 컨디셔닝 (Pre-conditioning)
> If you torture the data long enough, it will confess.
> 
> — Ronald Coase

SSD 제조사가 제공하는 제품의 데이터 시트에는 놀라울 정도의 성능 메트릭들로 채워져 있다.마케팅 효과를 위해서 제조사들은 항상 반짝이는 수치들을 보여주기 위해서 방법을 찾아내는 것처럼 보인다.그 수치들이 진정으로 뭔가를 의미하는지 모르겠지만, 실제 데이터 시트의 수치와 운영 시스템에서 사용되는 제품의 성능 예측은 다른 문제이다.
Marc Bevand가 작성한 문서[27](https://tech.kakao.com/posts/327#fn:66)는 일반적인 SSD 벤치마크의 결함에 대해서 언급하고 있는데,이 문서에서 Marc Bevand는 SSD의 LBA(Logical Block Addressing) 크기 설정에 대한 언급없이 랜덤 쓰기 성능을 명시하거나Queue depth를 1로 설정하고 테스트한 결과를 레포팅하는 것은 적절하지 못하다고 이야기하고 있다.또한 이 이외에도 벤치마크 도구의 버그나 잘못된 사용으로 인한 케이스들도 다양하다.
SSD의 성능을 정확하게 예측하는 것은 어려운 부분이다.많은 하드웨어 리뷰 블로그들이 10여분 정도의 테스트 실행 후, 그 결과를 두고 충분히 신뢰성 있다라고 이야기하고 있다.그러나 SSD는 지속적인 랜덤 쓰기(SSD의 전체 공간이 30분에서 3시간 정도 저장할 수 있는 수준의 워크로드)가 발생하는 테스트 환경에서만 성능 저하를 보여준다.그래서 어느정도의 쓰기 부하를 발생(이를 “pre-conditioning” [28](https://tech.kakao.com/posts/327#fn:50)라고 함)시킨 후, 벤치마크가 좀 더 신뢰성 있는 테스트라고 볼 수 있는 것이다.아래의 그림 7은 [StorageReview.com](http://storagereview.com/) [16](https://tech.kakao.com/posts/327#fn:26)으로부터 가져온 것인데, 다양한 SSD에서 “pre-conditioning”의 효과를 잘 보여주고 있다.좀더 명확한 성능 저하는 30분 정도 지난 이후 시점부터 나타나는데,이때부터 모든 SSD 드라이브에 대해서 스루풋은 떨어지고 레이턴시(응답 속도)는 커지는 것을 확인할 수 있다.그리고 4시간정도 경과한 후에는 성능이 더 떨어져서 최하 수준으로 수렴하는 것을 확인할 수 있다.
