---
title: DDR4와 DDR5
aliases:
  - DDR4
  - DDR5
  - DDR4 DDR5 동작 구조 비교
tags:
  - concept
  - dram
  - ddr
  - memory
type: concept
domain: Memory Architecture
created: 2026-11-27
---

# DDR4와 DDR5

[[DRAM]]에서 배운 셀, row buffer, ACT·READ·WRITE·PRE·REF를 DDR4와 DDR5에도 적용할 수 있다. 이 노트에서는 **컨트롤러가 64 B를 읽는 동안 칩 안팎에서 어떤 일이 일어나는지** 따라간다. 그 경로를 이해하면 DDR5에서 subchannel을 나누고 burst를 늘린 이유, bank group과 refresh가 성능에 영향을 주는 이유를 설명할 수 있다.

## 1. 비교할 층부터 정하기

| 층 | DDR4 | DDR5 | 관찰할 변화 |
|---|---|---|---|
| 셀과 배열 | DRAM 셀, 감지·복원, row buffer | 같은 동작 모델 | 같은 bank 안의 row conflict에는 두 세대 모두 row를 닫고 여는 과정이 필요하다. |
| 칩 조직 | 예로 16 Gb x8 부품은 4 bank group × 4 bank | 비교 예시의 16 Gb x8은 8 bank group × 4 bank | 여러 bank에 요청을 분산할 여지가 달라진다. 밀도와 x4/x8/x16 조직별로 확인한다. |
| 칩 데이터 경로 | 8n prefetch와 기본 BL8 | 16n prefetch와 기본 BL16 | 한 column 명령이 외부 핀으로 내보내는 기본 전송 횟수가 달라진다. |
| 일반 비ECC UDIMM | 모듈의 64-bit 데이터 경로 한 개 | 모듈당 32-bit 데이터 경로 두 개 | DDR5는 두 subchannel에 독립적으로 명령을 스케줄링할 수 있다. |
| Refresh | All-bank refresh | All-bank 및 same-bank refresh | Refresh로 접근이 제한되는 bank 범위를 조정할 수 있다. |
| 오류 정정 | DDR4 세대의 ECC DIMM에서 시스템 ECC를 구성할 수 있음 | 칩 내부 on-die ECC와 별도로 시스템 ECC를 구성할 수 있음 | 두 보호 방식이 오류를 감지·정정하는 위치가 다르다. |

부품 비교는 Micron의 16 Gb x8 예시를 기준으로 삼았다. 모듈 구성과 칩 부품표는 각각 따로 읽는다. [Micron DDR4·DDR5 장치 비교](https://jp.micron.com/content/dam/micron/global/public/products/white-paper/ddr5-more-than-a-generational-update-wp.pdf) · [Micron DDR5 클라이언트 모듈 자료](https://www.micron.com/content/dam/micron/global/public/products/white-paper/ddr5-key-module-features-wp-client.pdf)

## 2. 64 B READ가 실제로 지나가는 경로

### 2.1 두 세대의 공통 순서

1. 메모리 컨트롤러가 요청 주소를 채널 또는 subchannel·rank·bank·row·column으로 나눈다. 각 주소 비트의 배치는 플랫폼이 정한다.
2. 대상 row가 열려 있으면 column 명령을 발행할 수 있다. 닫혀 있으면 ACT 후 tRCD를 기다린다. 다른 row가 열려 있으면 PRE와 tRP가 먼저 필요하다.
3. READ가 열린 row의 column을 선택한다. 내부에서 병렬로 가져온 데이터를 I/O 회로가 여러 번의 외부 전송으로 나누어 보낸다.
4. DRAM이 데이터 핀 **DQ**로 비트를 보내고 데이터 스트로브 **DQS**로 샘플링 기준을 제공한다. WRITE에서는 컨트롤러가 DQ와 DQS를 보내는 쪽이다. 명령·주소는 별도의 **CA 경로**를 사용한다.
5. 이후 요청에 맞춰 같은 row를 유지하거나, 필요한 타이밍을 지킨 뒤 PRE로 닫는다.

### 2.2 폭과 burst가 함께 바뀐다

```text
전송 payload(B) = 한 번에 쓰는 데이터 폭(bit) × burst 전송 횟수 ÷ 8

DDR4 64-bit 경로 × BL8  ÷ 8 = 64 B
DDR5 32-bit 경로 × BL16 ÷ 8 = 64 B   (subchannel 하나)
```

두 식은 각각 64 B를 만든다. DDR5에서는 다른 subchannel에서도 별도 요청 하나를 진행할 여지가 생긴다. 주소가 어느 subchannel에 매핑되는지, 두 경로 모두에 보낼 요청이 있는지가 중요하다. 같은 subchannel에 몰린 두 요청은 그 경로의 데이터 버스를 공유한다. 모듈 안의 두 subchannel은 CPU 보드에서 DIMM 슬롯을 여러 메모리 채널에 연결하는 구성과 구분해서 봐야 한다. [Micron DDR5 클라이언트 모듈 자료](https://www.micron.com/content/dam/micron/global/public/products/white-paper/ddr5-key-module-features-wp-client.pdf)

비ECC 모듈의 단순한 x8 칩 예시를 손으로 계산해 보자. DDR4 x8 칩 하나는 BL8에서 8 B를 보낸다. 같은 rank의 x8 칩 8개가 같이 보내면 64 B다. DDR5 x8 칩 하나는 BL16에서 16 B를 보내고, subchannel의 x8 칩 4개가 합쳐서 64 B를 보낸다. **x8은 칩의 데이터 핀 수**, **BL은 데이터 핀당 해당 burst에서 전송하는 횟수**다.

### 2.3 Prefetch가 하는 일

8n·16n prefetch는 column 접근 한 번으로 외부 핀당 여러 비트분의 데이터를 내부에서 병렬로 준비하는 구조를 가리킨다. I/O 회로는 그 비트를 시간 순서로 내보낸다. DDR5에서 전송률을 올릴 때 외부 인터페이스와 내부 배열의 속도를 똑같이 올릴 필요가 없는 이유 중 하나다. 칩의 prefetch 깊이와 ACT로 열리는 row 전체의 크기는 별개의 단위다. [Micron DDR4·DDR5 장치 비교](https://jp.micron.com/content/dam/micron/global/public/products/white-paper/ddr5-more-than-a-generational-update-wp.pdf)

## 3. 지연 시간과 대역폭을 따로 계산하기

DDR은 클럭 한 주기에 두 번 데이터를 전송한다. `tCK(ns) = 2000 / MT/s`이고, CAS의 시간 환산값은 `CL × tCK`다. 아래는 모듈의 64-bit 총 payload 폭을 기준으로 계산한 예시다. 두 프로파일은 계산을 위한 입력이다.

| 항목 | DDR4-3200 CL22 | DDR5-5600 CL46 |
|---|---:|---:|
| tCK | 0.625 ns | 약 0.357 ns |
| READ → 첫 데이터의 CAS 시간 | 13.75 ns | 약 16.43 ns |
| 이론 대역폭, 총 64-bit 경로 | 25.6 GB/s | 44.8 GB/s |
| 64 B의 데이터 버스 점유 시간 근사, BL / 전송률 | 2.50 ns | 약 2.86 ns, subchannel 하나 |

DDR5 예시는 첫 데이터까지의 CAS 시간이 더 길다. 동시에 두 subchannel을 채우면 이론상 더 많은 데이터를 초당 보낼 수 있다. 실제 요청에는 큐 대기, row 열기·닫기, refresh, 버스 방향 전환 등이 붙는다. 특히 작은 요청을 하나씩 보내는 작업과 독립적인 요청을 많이 보내는 작업에서는 같은 메모리를 써도 서로 다른 결과가 나올 수 있다.

데이터시트에서 CL 옆에 **tRCD·tRP·tRAS·tRFC**도 적어 본다. 사이클 값과 최소 ns 조건이 함께 있는 항목은 데이터시트의 환산 규칙을 확인한다. 예를 들어 최소 14 ns와 최소 8 cycles가 동시에 요구되고 tCK가 0.625 ns라면 `max(ceil(14 / 0.625), 8) = 23 cycles`가 필요하다. [Micron DDR4 부품 데이터시트](https://www.micron.com/products/memory/dram-components/ddr4-sdram/part-catalog/part-detail/mt40a4g4sa-062e-ps-f) · [Micron DDR5 코어 데이터시트 및 부품 부록](https://www.micron.com/products/memory/dram-components/ddr5-sdram/part-catalog/part-detail/mt60b2g8rz-64b-d)

## 4. Bank group과 refresh가 바꾸는 스케줄

### 4.1 Bank가 많아져도 주소가 한 bank에 몰리면

같은 row를 이어서 읽으면 ACT를 반복할 필요가 없다. 다른 bank에 독립적인 요청이 있으면 한 bank가 기다리는 동안 다른 명령을 진행할 수 있다. 서로 다른 bank라도 같은 bank group에 있으면 일부 내부 경로를 공유하므로 더 긴 명령 간격이 적용될 수 있다.

- **tCCD_S / tCCD_L** 계열은 연속 column 명령 간격을 읽는 데 쓰인다. 일반적인 비교에서는 다른 group과 같은 group을 구분하며, 정확한 READ·WRITE 조합별 이름과 적용 조건은 규격과 부품표를 본다.
- **tRRD / tFAW**는 여러 bank에 ACT를 몰아 발행하는 경우에도 제약이 남는 이유다. 예를 들어 다수 bank가 모두 닫혀 있어도 모든 bank를 같은 시점에 열 수는 없다.
- **Row hit 비율과 bank 분산**은 따로 기록한다. 같은 bank의 서로 다른 row를 번갈아 접근하는 패턴은 bank 수를 늘려도 그 bank의 PRE·ACT 비용이 남는다.

### 4.2 DDR5 same-bank refresh

All-bank refresh인 `REFab`는 대상 rank에서 모든 관련 bank를 준비시키고, 명령이 요구하는 기간 동안 해당 bank들에 정상 접근을 재개할 수 없게 한다. DDR5의 `REFsb`는 **각 bank group에서 같은 번호의 bank 하나씩**을 대상으로 삼는다. 예를 들어 group 0부터 7까지 각각 bank 2를 고르는 식이다. 이 예에서는 32개 중 8개가 선택된다.

다른 bank의 작업을 계속할 수 있는 여지가 생기지만, 실제 스케줄에는 refresh 모드, 대상 bank의 precharged 조건, 미대상 bank의 ACT 간격 제약, 전체 refresh 의무가 남는다. 요청이 refresh 대상에 몰릴수록 대기가 더 잘 드러날 수 있다. 실측을 해석할 때는 tRFCab·tRFCsb와 tREFI를 해당 부품의 밀도 및 모드와 함께 기록한다. [Micron DDR5 SDRAM New Features](https://assets.micron.com/adobe/assets/urn%3Aaaid%3Aaem%3A5ea148c8-e3fe-489e-8489-99b1b9cdcd3c/renditions/original/as/ddr5-new-features-white-paper.pdf)

## 5. 모듈과 신호, 부팅할 때 필요한 것

| 용어 | 하는 일 | 공부할 때 구분할 것 |
|---|---|---|
| SPD / SPD Hub | 모듈 조직과 지원 타이밍 등의 설정 정보를 제공한다. DDR5 모듈에서 SPD Hub는 모듈 관리 경로의 일부다. | SPD에 적힌 지원 프로파일과 부팅 후 실제 적용된 속도·타이밍을 구분한다. |
| PMIC | 일반적인 DDR5 DIMM에 배치되어 모듈 내 전압을 조절한다. | 부품의 코어·I/O 전압과 모듈이 외부에서 공급받는 전원을 구분한다. 대표적인 코어·I/O 전압은 DDR4 1.2 V, DDR5 1.1 V다. |
| Training | 부팅 중 컨트롤러와 DRAM 사이의 전송 타이밍과 샘플링 여유를 맞춘다. | DQ·DQS 정렬, 기준 전압과 타이밍 여유를 조정하는 작업이다. 플랫폼에 따라 실제 절차가 달라진다. |
| ODT / DFE | ODT는 종단으로 반사를 줄이고, DDR5의 DQ 수신기에서 쓰는 DFE는 이전 심볼에 따른 신호 왜곡을 줄이는 데 쓰인다. | 전송률이 높을수록 같은 시간 내에 신호를 판독할 여유가 줄어든다. 이 문제는 셀 retention과 다른 지점에서 발생한다. |

UDIMM 기준 모듈 설명을 읽을 때, 칩 내부 동작과 보드 레벨 부품을 같은 층에 놓지 않는다. 예를 들어 PMIC와 SPD Hub는 DIMM 설명에서, `ACT`와 `tRFC`는 DRAM 부품 데이터시트에서 먼저 찾는다. [Micron DDR5 클라이언트 모듈 자료](https://www.micron.com/content/dam/micron/global/public/products/white-paper/ddr5-key-module-features-wp-client.pdf) · [Micron DDR5 장치 기능](https://jp.micron.com/content/dam/micron/global/public/products/white-paper/ddr5-more-than-a-generational-update-wp.pdf)

## 6. On-die ECC와 시스템 ECC

```text
DRAM 셀 배열 → 칩 내부 on-die ECC → DRAM I/O 핀 → 모듈·보드 배선 → 메모리 컨트롤러
                  칩 내부 코드워드 처리                         시스템 ECC 검사 지점의 예
```

**On-die ECC**는 DRAM 칩 내부에 저장된 데이터의 오류를 내부 회로가 감지·정정하는 방식이다. Micron 문서는 128 data bits와 8 check bits를 묶는 단일 비트 정정(SEC) 예를 설명한다. 정정이 끝난 값이 핀에 나타나므로, 핀 이후 전송 경로에서 생긴 오류는 그 내부 검사로 다룰 수 없다. 이 SEC 설명에 시스템 수준의 이중 비트 오류 검출까지 포함시켜 해석하지 않는다. [Micron DDR5 SDRAM New Features](https://assets.micron.com/adobe/assets/urn%3Aaaid%3Aaem%3A5ea148c8-e3fe-489e-8489-99b1b9cdcd3c/renditions/original/as/ddr5-new-features-white-paper.pdf)

**시스템 ECC**는 메모리 컨트롤러가 데이터와 별도 검사 비트를 저장하고 읽을 때 검사하는 구성이다. DIMM 유형, 컨트롤러, 플랫폼 설정이 지원해야 하며 실제 보호 범위와 오류 보고 방식은 구현에 따른다. 시스템 로그에서 오류가 보고됐는지 볼 때는 어느 층에 카운터가 있고 컨트롤러가 무엇을 노출하는지 확인한다. NAND 내부 ECC는 SSD 컨트롤러와 NAND 데이터 보호의 별도 사례다. [[SSD]] · [[NVMe SMART Telemetry]]

## 7. SSD 프로젝트에 적용할 질문

호스트 DIMM에서의 64-bit와 두 subchannel 비교를 SSD 온보드 메모리에 그대로 적용할 수는 없다. SSD 쪽에서는 해당 컨트롤러에 연결된 DRAM의 **실제 종류, 데이터 폭, 채널 수, 어떤 자료구조를 저장하는지**를 먼저 확인한다. 온보드 메모리의 매핑 조회가 의심되면 다음처럼 증거를 구분한다.

| 관찰한 결과 | 다음에 필요한 관찰 |
|---|---|
| Random read p99 증가 | fio 조건과 반복성, 호스트·USB·OS 경로, 장치 상태를 확인한다. |
| FTL 매핑 접근 지연 증가 | 매핑 hit/miss 카운터, NAND 매핑 페이지 fetch, 컨트롤러 trace를 확인한다. |
| DRAM row conflict나 refresh 대기 증가 | DRAM 컨트롤러 카운터나 명령 trace, 또는 명령 타이밍을 볼 수 있는 모델을 확인한다. |

연습으로 [[External SSD Block Size Sweep]]의 4 KiB I/O 하나를 고르고 **호스트 요청 크기, SSD 내부 매핑 조회, DRAM burst payload**를 각 단계에 적어 본다. 어느 단계에서 몇 번의 DRAM 접근이 일어나는지는 펌웨어와 하드웨어 설계를 알아야 정할 수 있다. [[FTL]] · [[SSD Host Device Path]]

## 8. 노트를 닫고 풀기

1. DDR4는 64-bit, DDR5는 subchannel당 32-bit인데 어떻게 둘 다 READ 하나로 64 B를 만들 수 있는가?
2. 두 요청이 DDR5의 서로 다른 subchannel에 갈 때와 같은 subchannel에 갈 때의 제약은 어떻게 다른가?
3. DDR4-3200 CL22와 DDR5-5600 CL46의 CAS 시간을 계산하라. 계산한 값에 어떤 대기 시간들이 빠져 있는가?
4. 8개 bank group에 group당 4개 bank가 있을 때 bank 번호 2를 대상으로 한 `REFsb`는 어느 bank들을 고르는가?
5. DRAM이 데이터를 내보낸 뒤 보드 배선에서 비트가 바뀌었다. 어떤 계층에서 이 오류를 검출할 수 있는가?
6. 읽을 L2P 매핑이 온보드 DRAM에 있지만 요청이 느리다. 매핑 캐시 miss와 row conflict를 어떻게 가려낼 것인가?

> [!answer]- 확인용 해설
> 1. `64 × 8 / 8 = 64 B`, `32 × 16 / 8 = 64 B`. 폭이 반이 되고 기본 전송 횟수가 두 배가 된다.
> 2. 서로 다른 subchannel에서는 각자의 데이터 경로에 전송을 스케줄할 수 있다. 같은 subchannel에서는 같은 버스의 전송 시간과 명령 제약을 나눠 쓴다.
> 3. 각각 13.75 ns와 약 16.43 ns다. Row를 닫고 여는 시간, 큐 대기, refresh, burst 완료까지의 시간 등이 빠져 있다.
> 4. 각 group의 bank 2, 총 8개를 선택한다. 대상 외 bank에도 데이터시트에 명시된 명령 타이밍 제약이 남는다.
> 5. 오류가 발생한 지점 이후에서 검사하는 시스템 ECC가 지원·활성화돼 있다면 그 코드의 보호 범위에서 검출할 수 있다. On-die ECC 검사는 칩 내부 출력 이전에 수행된다.
> 6. FTL의 hit/miss와 매핑용 NAND fetch를 먼저 확인한다. 매핑이 메모리에 있었는지와 해당 DRAM row가 열려 있었는지는 별개의 상태이므로 후자는 컨트롤러 카운터나 명령 trace로 확인한다.

## 원문 읽기 순서

1. 모듈 자료에서 두 subchannel 그림을 보고 `32 × 16 ÷ 8`을 각 경로에 적어 본다. [Micron DDR5 클라이언트 모듈 자료](https://www.micron.com/content/dam/micron/global/public/products/white-paper/ddr5-key-module-features-wp-client.pdf)
2. 장치 자료에서 표의 같은 밀도와 x8 조직만 골라 bank group 수, bank 수, prefetch, burst를 비교한다. [Micron DDR4·DDR5 장치 비교](https://jp.micron.com/content/dam/micron/global/public/products/white-paper/ddr5-more-than-a-generational-update-wp.pdf)
3. 실제 부품 데이터시트에서 ACT→READ→첫 데이터의 파형을 옮겨 그리고, 가능하면 `REFab`와 `REFsb`의 타이밍 제약도 표시한다. [DDR4 데이터시트 진입점](https://www.micron.com/products/memory/dram-components/ddr4-sdram/part-catalog/part-detail/mt40a4g4sa-062e-ps-f) · [DDR5 데이터시트 진입점](https://www.micron.com/products/memory/dram-components/ddr5-sdram/part-catalog/part-detail/mt60b2g8rz-64b-d)
