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

아래의 그림 1은 SSD의 주요 컴포넌트들을 도식화한 것이다.이는 이미 다양한 문서([13](https://tech.kakao.com/posts/327#fn:2) [14](https://tech.kakao.com/posts/327#fn:3) [15](https://tech.kakao.com/posts/327#fn:6))들에서 공개된 내용들이며, 간단히 개요만 표시한 것이다.