---
title: DRAM
aliases:
  - Dynamic Random Access Memory
  - DRAM 동작 원리
tags:
  - concept
  - dram
  - memory
  - ssd-controller
type: concept
status: growing
domain: Memory Architecture
created: 2026-11-27
---

# DRAM

**읽고 나서 할 수 있어야 하는 것** — 메모리 컨트롤러가 특정 주소를 읽을 때 어떤 row가 열리고, 어떤 회로가 비트를 감지하고, 데이터가 언제 버스에 나타나는지 설명한다. 같은 주소 크기의 요청이라도 지연 시간이 달라지는 이유를 명령 순서로 재구성한다. 마지막으로 이 모델을 SSD의 FTL 매핑 조회에 적용한다.

## 읽는 순서

1. **1–3절**에서 셀 하나를 이해하고 `ACT → RD → PRE`를 종이에 그린다.
2. **4–6절**에서 같은 bank의 주소 두 개를 골라 지연 시간, refresh, 대역폭을 계산한다.
3. **7–8절**에서 SSD의 주소 변환 경로를 복기하고 DRAM 관련 가설을 세운다.
4. **9절**은 노트를 닫고 푼 뒤 접힌 해설을 펼친다.

## 1. 비트 하나를 어떻게 저장하는가

일반적인 DRAM 셀은 접근 트랜지스터 하나와 저장 커패시터 하나로 구성된다. 이를 **1T1C**라고 부른다. 커패시터의 전압 상태가 비트를 나타내고, 트랜지스터는 커패시터를 bitline에 연결하거나 격리한다.

```text
                  wordline (WL)
                        │ 게이트 제어
bitline (BL) ── 접근 트랜지스터 ── 저장 노드 ── 커패시터
```

- **Wordline**은 같은 row의 접근 트랜지스터를 함께 켠다. row 주소를 해석한 결과가 들어가는 곳이다.
- **Bitline**은 선택된 셀과 sense amplifier 사이에서 전압 변화를 전달하는 회로 배선이다.
- **Sense amplifier**는 아주 작은 전압 차이를 감지해 확실한 논리 레벨까지 증폭하고 유지한다.

커패시터에서 전하가 서서히 누설된다. 데이터가 구분 가능한 상태로 남는 시간을 **retention time**이라고 한다. 전원이 켜져 있어도 일정 주기로 셀 값을 감지하고 복원해야 한다. 이 주기적인 동작이 **refresh**다. 전원이 끊겨 감지와 복원이 멈추면 DRAM에만 남아 있던 데이터를 장기 보존할 수 없다.

DRAM과 SRAM은 둘 다 휘발성이다. SRAM은 전원이 공급되는 동안 피드백 회로가 상태를 유지하므로 주기적 refresh가 필요 없고, 대표적인 SRAM 셀은 6개 트랜지스터를 사용한다. 1T1C DRAM은 셀당 면적을 줄여 큰 용량을 구성하기 좋다. NAND는 전원이 꺼져도 상태를 보존하는 별도의 저장 매체이며, 페이지 읽기·프로그램과 블록 소거 제약이 있다. SSD에는 이런 서로 다른 메모리가 함께 들어갈 수 있다. [[SSD 기본 원리와 구조]]

## 2. 읽기의 전기적 동작

### 2.1 Bitline 준비 → 전하 공유 → 감지 → 복원

1. **Precharged 상태.** 이전 row가 닫혀 있고 bitline과 비교용 쌍이 일반적인 설명 모델에서 약 `VDD/2`로 맞춰져 있다. 두 선의 시작점을 맞추면 셀이 만든 작은 차이를 비교할 수 있다.
2. **Wordline 활성화.** 접근 트랜지스터가 켜지고 셀 커패시터가 bitline에 연결된다. 전하가 공유되어 bitline이 기준 전압보다 조금 높아지거나 낮아진다.
3. **감지 및 증폭.** Sense amplifier가 그 차이를 증폭해 bitline 쌍을 높은 전압과 낮은 전압으로 갈라놓는다. 이때 해당 row의 값이 sense amplifier 배열에 잡힌다. 이 배열이 **row buffer** 역할을 한다.
4. **셀 복원.** Wordline이 열린 동안 증폭된 bitline이 셀 커패시터를 다시 충전하거나 방전한다. 전하 공유 중 달라졌던 셀의 상태가 복원된다.

여기서 **destructive read**라는 말은 셀을 감지하는 과정에서 원래 전압이 교란된다는 뜻이다. 정상 ACT 동작에는 복원이 포함되므로 호스트가 데이터를 읽을 때마다 데이터가 사라지는 현상은 보이지 않는다. 이미 열린 row의 값을 다시 요청하면 열려 있는 sense amplifier에서 데이터를 가져올 수 있다. 회로 도식은 RowClone 논문의 DRAM substrate 설명에서 확인할 수 있다. [RowClone 원 논문](https://doi.org/10.1145/2540708.2540725)

### 2.2 왜 sense amplifier가 필요한가

기생 성분과 누설을 생략한 전하 공유 모델에서 `Ccell`은 셀 용량, `CBL`은 bitline 용량이다. 감지 직전의 전압 차이는 다음처럼 근사할 수 있다.

```text
ΔVbitline ≈ Ccell / (Ccell + CBL) × (Vcell − VDD/2)
```

학습용으로 `Ccell = 1`, `CBL = 9`, `VDD = 1 V`라고 하면, 셀 전압이 1 V일 때 bitline은 약 0.55 V가 된다. 0 V를 담고 있었다면 약 0.45 V다. 셀에 저장된 큰 전압 차이를 bitline에서 그대로 읽는다는 가정으로는 이 동작을 설명할 수 없다. 셀 전하가 더 샜거나, bitline에 연결된 회로의 용량이 더 크면 감지할 차이도 줄어든다. 위 숫자는 특정 칩의 측정값이 아니다.

## 3. 주소 하나가 DRAM 명령으로 바뀌는 과정

### 3.1 주소가 가리키는 계층

| 단위 | 접근에 미치는 영향 |
|---|---|
| Channel | 독립적인 메모리 인터페이스. 채널이 여러 개면 여러 경로로 데이터 전송을 분산할 수 있다. |
| Rank | 같은 채널에서 함께 선택되어 동작하는 DRAM 칩 묶음. 같은 채널에 있는 rank들은 데이터 버스를 공유한다. |
| Bank / bank group | Bank는 row가 열리고 닫히는 상태를 관리하는 단위다. Bank group이 있는 규격은 bank 간 연속 명령에 서로 다른 간격을 적용할 수 있다. |
| Subarray | 칩 내부에서 셀 배열과 지역 bitline·sense amplifier를 구성하는 물리 단위. 이 노트에서는 외부 명령을 이해하기 위해 bank당 열린 row 하나로 모델링한다. |
| Row | ACT로 선택되는 셀의 줄. 같은 wordline에 연결된 셀들이 함께 감지된다. |
| Column | 열린 row에서 READ 또는 WRITE가 대상으로 삼는 부분을 고른다. 외부 데이터 전송은 burst 단위로 이루어진다. |

일반적인 DDR 시스템에서는 메모리 컨트롤러가 요청 주소를 채널·랭크·뱅크·행·열 등으로 해석한다. 실제 주소 비트 배치는 플랫폼에 따라 달라진다. 주소 두 개가 연속이라고 해서 같은 bank의 인접 column이라는 사실까지 알 수는 없다.

### 3.2 대표 명령

| 명령 | 컨트롤러가 지정하는 대상 | DRAM 내부 동작 |
|---|---|---|
| ACT | bank와 row | Wordline을 열어 해당 row를 감지하고 복원한다. |
| RD | 열린 bank와 column | Row buffer의 선택된 부분을 내부 I/O를 거쳐 데이터 핀으로 전송한다. |
| WR | 열린 bank와 column | 들어온 데이터를 선택된 열의 sense amplifier·bitline을 통해 셀에 기록한다. |
| PRE | 닫을 bank 등 | 열린 row를 닫고 bitline을 다음 감지를 위한 기준 상태로 맞춘다. |
| REF | 규격과 refresh 모드에 따른 범위 | 내부 refresh 순서에 따라 셀 값을 감지·복원한다. 일반적인 READ처럼 데이터를 외부로 반환하지 않는다. |

예를 들어 bank 0이 닫힌 상태에서 row 12의 column 4를 읽으면 `ACT B0/R12 → 대기 → RD B0/C4 → 데이터 burst` 순으로 진행한다. 다음 요청이 같은 bank·row의 column 8이면 row를 유지한 상태에서 `RD B0/C8`을 발행할 수 있다. 다음 요청이 bank 0의 row 13이라면 기존 row를 닫고 `ACT B0/R13`을 발행해야 한다.

WRITE도 먼저 대상 row를 열어야 한다. 컨트롤러가 정해진 시점에 데이터를 버스로 보내면 내부 쓰기 회로가 선택된 column의 값을 바꾼다. 셀 값이 충분히 자리 잡을 때까지 기다린 뒤 row를 닫아야 한다. DDR 데이터시트를 볼 때 명령 정의와 AC timing 표를 함께 보는 이유다. [Micron DDR4 부품 및 데이터시트](https://www.micron.com/products/memory/dram-components/ddr4-sdram/part-catalog/part-detail/mt40a4g4sa-062e-ps-f)

## 4. 같은 읽기 요청의 지연 시간이 달라지는 이유

### 4.1 Row hit, closed-row miss, row conflict

| 요청 직전 bank 상태 | 경로 | 첫 데이터까지 필요한 대표 타이밍 |
|---|---|---|
| 목표 row가 열려 있음 | RD | CL |
| bank가 이미 precharged 상태 | ACT → RD | tRCD + CL |
| 같은 bank의 다른 row가 열려 있음 | PRE → ACT → RD | tRP + tRCD + CL |

이 표는 각 명령을 가능한 즉시 발행할 수 있다고 가정한 단순 모델이다. 실제 컨트롤러는 이전 ACT 후 최소 활성 시간, 이전 WR 후 복원 시간, refresh나 버스 점유에 따른 대기 등을 추가로 고려한다.

### 4.2 타이밍을 명령 사이에 놓고 읽기

| 이름 | 어떤 시점 사이인가 | 왜 기다리는가 |
|---|---|---|
| tRCD | ACT → RD/WR가 허용되는 시점 | Row 감지 회로가 column 접근을 수행할 수 있어야 한다. 부품별로 읽기와 쓰기 정의를 확인한다. |
| CL / tCL | RD → 첫 읽기 데이터 | Column 선택 이후 내부 데이터 경로와 외부 전송 타이밍을 맞춘다. CL은 보통 클럭 수로 표기한다. |
| tRAS | ACT → 같은 bank의 PRE | 감지와 셀 복원에 필요한 최소 활성 시간을 확보한다. |
| tRP | PRE → 같은 bank의 다음 ACT | Bitline을 다음 감지에 필요한 상태로 되돌린다. |
| tRC | 같은 bank의 ACT → 다음 ACT | 한 row의 활성 구간과 다음 활성화 준비 구간에 걸친 최소 시간이다. |
| tWR | 마지막 쓰기 데이터 이후 → PRE 허용에 필요한 복원 시간 | 쓰인 셀 값이 자리 잡기 전에 row가 닫히지 않도록 한다. 정확한 기준점은 타이밍 다이어그램에서 찾는다. |
| tCCD | 연속 column 명령 사이의 간격 | 내부 전송과 버스 사용을 조율한다. Bank group 관계에 따라 제약이 다를 수 있다. |
| tRRD / tFAW | 다른 bank에 대한 ACT 간격 / 연속 ACT 수가 제한되는 시간 창 | 여러 row를 빠르게 열 때 생기는 활성화 전력 부담을 제한한다. |

예를 들어 학습용으로 `tRCD = 14 ns`, `CL에 해당하는 시간 = 14 ns`, `tRP = 14 ns`라 두면 첫 데이터까지 row hit는 14 ns, closed-row miss는 28 ns, row conflict는 최소 42 ns다. 이미 열린 row가 최소 활성 시간을 채우지 못했다면 conflict에는 그 남은 시간도 붙는다. 여기에는 컨트롤러 큐 대기, 주소 변환, 데이터 burst 완료까지의 시간은 들어 있지 않다.

### 4.3 Bank가 여러 개여도 모든 지연이 사라지지 않는 이유

Bank 0이 ACT 후 tRCD를 기다리는 동안 bank 1의 요청을 진행할 여지가 있다. 이런 겹침이 **bank-level parallelism**이다. 같은 채널의 데이터 버스는 공유하므로 두 응답이 같은 시점에 그 버스를 점유할 수는 없다. 활성화 간격, 읽기와 쓰기 방향 전환, refresh도 스케줄을 제약한다.

컨트롤러가 열린 row를 유지하면 이어지는 row hit 비용이 줄어든다. 다음 접근이 다른 row로 몰리면 row를 닫는 비용이 드러난다. Row hit를 지나치게 우선하면 먼저 들어온 conflict 요청의 대기가 길어질 수도 있다. 성능을 분석할 때는 **서비스 시간과 큐 대기 시간**을 나눠 보고, 평균과 tail의 변화도 따로 확인한다. [[Queue Depth]] · [[p99 latency]]

## 5. Refresh와 retention

**Restore**는 활성화한 row에서 감지 과정으로 달라진 셀의 상태를 복원하는 동작이다. **Precharge**는 row를 닫고 다음 감지를 위해 bitline을 준비하는 동작이다. **Refresh**는 읽기 요청이 없는 row까지 보존할 수 있게 내부 감지·복원을 주기적으로 수행하는 동작이다.

| 개념 | 뜻 |
|---|---|
| Retention time | 특정 조건에서 한 셀이 값을 구분 가능한 상태로 유지하는 시간. |
| Refresh window | 규격에서 정한 refresh 범위를 한 바퀴 다루는 데 허용되는 시간 범위. |
| tREFI | 표준 refresh 모드에서 refresh 명령의 평균 발행 간격을 나타내는 값. |
| tRFC | REF를 발행한 뒤 규격에서 정한 후속 접근을 허용하기까지 필요한 시간. 영향 범위는 refresh 방식에 따라 달라진다. |
| Self-refresh | 지원되는 저전력 상태에서 DRAM이 전원을 공급받으며 내부적으로 refresh를 관리하는 모드. |

학습용 예로 refresh window 64 ms에 8,192회의 REF가 필요하다고 두면 평균 간격은 `64 ms / 8,192 ≈ 7.8125 µs`다. 각 REF가 가정상 350 ns 동안 해당 rank를 막는다면 단순 시간 비율은 약 4.48%다. 이 값은 **rank가 막혀 있는 시간의 근사치**다. 애플리케이션의 평균 처리량 감소율이나 p99 증가율은 요청이 들어오는 시점, 다른 rank 또는 채널의 작업으로 대기를 숨길 수 있는지, 컨트롤러가 refresh를 어떻게 스케줄링하는지에 따라 달라진다. 실제 tREFI와 tRFC는 부품 밀도, 온도 범위와 refresh 모드를 명시한 표에서 확인한다.

Retention은 셀마다 다르고 온도와 저장 패턴의 영향을 받을 수 있다. Liu 등의 연구는 상용 DDR3 칩을 대상으로 저장 패턴 의존성과 일부 셀의 retention이 시간에 따라 달라지는 현상을 관찰했다. 이 관측은 약한 셀의 분포가 왜 중요한지 보여준다. 해당 실험의 수치를 DDR4·DDR5의 제품 보장값으로 옮겨 쓰려면 별도의 근거가 필요하다. [Liu 외, ISCA 2013 DRAM retention 연구](https://istc-cc.cmu.edu/publications/papers/2013/dram-retention-time-characterization_isca13_abs.shtml)

## 6. SDRAM과 DDR로 넘어가기

SDRAM은 인터페이스가 클럭에 동기화되는 DRAM이다. DDR SDRAM은 데이터 전송에서 한 클럭 주기당 두 번 전송한다. DDR의 전송률을 읽을 때는 **클럭 주파수(MHz), 전송률(MT/s), 데이터 버스 폭(bit), 명령 지연(cycles 또는 ns)**을 각각 확인한다. 실제 데이터는 데이터 스트로브(DQS)에 맞춰 전송된다.

```text
클럭 주기(ns) = 2,000 / 데이터 전송률(MT/s)
CAS 시간(ns) ≈ CL × 클럭 주기(ns)
이론 대역폭(GB/s) = 전송률(MT/s) × 데이터 버스 폭(bit) / 8 / 1,000
```

예로 3,200 MT/s에서 클럭 주기는 0.625 ns이고 CL22가 나타내는 시간은 13.75 ns다. 64-bit 데이터 경로를 가정하면 이론 대역폭은 25.6 GB/s다. 이 수치로 row conflict 처리 시간이나 실제 애플리케이션 대역폭을 바로 알 수 없다. 예를 들어 한 x8 칩의 데이터 핀은 8개이고, 64-bit 폭은 그런 칩을 여러 개 묶어 구성할 수 있는 데이터 경로 폭이다. Micron 부품표에서 실제로 전송률과 클럭 주파수, x8, CL의 표기가 어떻게 나뉘는지 비교해 볼 수 있다. [Micron DDR4 부품 카탈로그](https://www.micron.com/products/memory/dram-components/ddr4-sdram/part-catalog)

[[DDR4와 DDR5]]에서 외부 버스트 길이, bank 및 bank group 구성, 모듈의 채널 구성, refresh 명령, 오류 보호 범위를 비교한다. 여기서 배운 ACT·RD·WR·PRE·REF를 두 세대가 64 B를 읽는 과정에 적용해 본다.

## 7. SSD 컨트롤러에서는 무엇을 저장하는가

SSD의 DRAM 용도는 설계에 따라 달라진다. 대표적인 후보는 **FTL 주소 매핑**, 펌웨어의 작업용 자료구조, 읽기·쓰기 데이터 버퍼다. 어느 구조가 DRAM에 있고 무엇이 SRAM 또는 NAND에 있는지는 제품별로 확인해야 한다.

읽기 경로를 단순화해 보자. 호스트가 NVMe로 LBA를 보내면 FTL이 그 주소에 대응하는 NAND 위치를 찾는다. 매핑이 컨트롤러의 메모리에 있으면 거기서 조회할 수 있다. 필요한 매핑이 메모리에 없고 NAND에 저장된 설계라면, 실제 사용자의 데이터 페이지를 읽기 전에 매핑 페이지를 읽는 비용이 생길 수 있다. 이 구분을 먼저 그려 보면 **FTL 매핑 캐시 miss와 DRAM row miss**가 왜 다른지 설명할 수 있다.

| 현상 | 그 계층에서 찾지 못한 것 | 필요한 추가 작업의 예 |
|---|---|---|
| FTL 매핑 캐시 miss | 현재 컨트롤러 메모리에 있는 주소 매핑 항목 | NAND에서 매핑 페이지를 읽고 항목을 가져온다. 구체적인 경로는 구현에 따른다. |
| DRAM row conflict | 필요한 row가 현재 해당 bank에 열려 있지 않으며 다른 row가 열려 있음 | PRE 후 ACT를 발행하고 column 명령을 내린다. |
| OS page cache miss | 호스트 메모리에 캐시된 파일 데이터 | 하위 스토리지 경로에 I/O가 내려갈 수 있다. |

**계산 예제.** 1 TiB 논리 용량에 4 KiB 단위 매핑, 항목당 4 B의 평평한 전체 매핑 테이블을 가정하면 항목은 `2^40 / 2^12 = 2^28개`, 테이블은 `2^28 × 4 B = 1 GiB`다. 다른 조건이 같고 항목이 8 B라면 2 GiB다. 실제 SSD의 DRAM 용량을 추정하려면 매핑 단위, 계층형 테이블이나 압축 여부, NAND에 둔 매핑의 캐시 비율, 나머지 DRAM 용도를 알아야 한다. [[FTL]] · [[LBA LPN PPN]]

온보드 DRAM 자체는 휘발성이다. SSD가 쓰기 완료를 알려줬을 때 어떤 데이터와 매핑이 전원 장애 후에 남는지는 쓰기 캐시 정책, Flush/FUA 의미, PLP의 보호 범위와 복구 동작을 함께 봐야 한다. DRAM-less NVMe SSD가 사용하는 HMB는 지원되는 경우 호스트 메모리 일부를 쓰는 별도 경로다. 온보드 DRAM과 주소 접근 경로 및 전원 장애 경계가 다르다. [[SSD 전력 손실 보호 원리]] · [[SSD 통신 프로토콜]]

## 8. SSD 검증 실험으로 연결하기

**질문.** Random read의 주소 범위가 넓어졌을 때 p99가 증가한다면 매핑 캐시 사용량이나 miss와 관련이 있을까?

1. 같은 장치와 연결 경로에서 small working set과 large working set을 비교한다. I/O 크기, 읽기 패턴, QD, 테스트 시간, 반복 횟수를 맞춘다. 동일한 사전 준비 절차를 쓰고 warm-up을 별도로 기록한다.
2. 주소 범위별로 IOPS와 지연 시간 분포를 본다. 조건 순서를 바꿔 반복해 순서에 따른 변화도 확인한다.
3. OS 캐시, 장치의 읽기 캐시, 데이터 배치, 온도, 백그라운드 동작을 대안 가설로 둔다. 특히 파일 대상이면 경로 자체의 영향이 섞인다. [[SSD Host Device Path]]
4. 펌웨어나 시뮬레이터에 접근할 수 있다면 mapping hit/miss, mapping-page fetch 횟수, 각각의 대기 시간을 함께 기록한다. 실제 DRAM row hit까지 검증하려면 DRAM 컨트롤러 수준의 카운터 또는 그 동작을 관찰할 수 있는 모델이 추가로 필요하다.

**판정 예시.** “주소 범위를 넓힐 때 이 장치와 조건에서 p99가 증가했다”는 fio 결과로 평가할 수 있다. “FTL 매핑 miss가 늘었다”는 별도 카운터나 trace로 확인할 가설이다. [[NVMe SMART Telemetry]]의 온도나 오류 통계는 대안 가설을 검토하는 데 쓸 수 있다. 일반 SMART 지표에서 DRAM row conflict 수를 바로 읽을 수 있지는 않다.

## 9. 무노트 복습과 종이 실습

1. Cell과 bitline의 전압이 공유되기 전후를 그려라. Sense amplifier가 값을 판단하고 셀에 복원하는 시점은 언제인가?
2. Bank 0은 처음에 닫혀 있다. `B0/R4/C0 → B0/R4/C8 → B0/R5/C0`을 순서대로 처리하는 명령을 적고 각각 hit·miss·conflict로 분류하라.
3. `tRP = 15 ns`, `tRCD = 15 ns`, `CL 시간 = 15 ns`일 때, 직전 row를 즉시 닫을 수 있는 conflict의 첫 데이터까지 걸리는 최소 시간은?
4. 64 ms마다 8,192회의 refresh가 필요할 때 tREFI를 계산하라. 이 값만으로 REF 한 번의 접근 차단 시간을 알 수 있는가?
5. 6,400 MT/s CL32와 3,200 MT/s CL22의 CAS 시간을 각각 계산하라. 이 값으로 두 시스템에서 프로그램 전체의 메모리 지연을 비교할 수 있는가?
6. FTL 매핑 캐시 hit와 DRAM row conflict가 한 요청에서 동시에 성립하는 상황을 설명하라.

> [!answer]- 확인용 해설
> 1. Wordline이 열리면 셀과 bitline 사이에 전하 공유가 일어난다. 그 직후 작은 전압 차이가 생긴다. Sense amplifier가 차이를 키우고, wordline이 열린 동안 bitline이 셀 전압을 복원한다.
> 2. `ACT R4 → RD C0`은 closed-row miss, 이어지는 `RD C8`은 hit, `PRE → ACT R5 → RD C0`은 conflict다. 각 명령 사이에는 해당 타이밍을 지켜야 한다.
> 3. `15 + 15 + 15 = 45 ns`. 즉시 PRE를 발행할 수 있다는 전제가 있어야 한다.
> 4. 약 `7.8125 µs`. 이는 명령 간격이다. 한 번의 차단 시간에는 tRFC와 발행한 refresh 방식이 필요하다.
> 5. 각각 `32 × 0.3125 = 10 ns`, `22 × 0.625 = 13.75 ns`. 전체 지연을 비교하려면 row 상태, 컨트롤러 대기, refresh, 버스 경합 등의 정보가 추가로 필요하다.
> 6. 매핑 항목 자체는 SSD DRAM에 있지만, 그 항목을 담은 row와 다른 row가 같은 bank에 열려 있으면 컨트롤러는 기존 row를 닫고 목표 row를 연다. FTL에서는 hit, DRAM에서는 conflict다.

복습할 때 AI에는 **먼저 내가 그린 명령 순서와 숫자 계산을 보여주고, 빠진 전제나 틀린 명령 한 가지를 물어봐 달라**고 요청한다. 정답을 읽은 뒤에는 주소 순서나 tRCD 값을 바꿔 같은 문제를 다시 푼다.

## 원문 읽기

- 셀, bitline, subarray와 row buffer는 RowClone 논문의 DRAM substrate 부분을 본다. 논문의 복사 기법은 일반적인 DRAM 명령 동작을 먼저 읽은 뒤 살펴본다. [RowClone 공개 기술 보고서](https://research.ece.cmu.edu/safari/tr/CMU-CS-13-108.pdf)
- 부품 하나를 고른 뒤 데이터시트에서 ACT·READ·WRITE·PRE·REF 파형을 찾고 tRCD·tRAS·tRP·tRFC 화살표의 시작점과 끝점을 표시한다. [Micron DDR4 부품 페이지와 데이터시트](https://www.micron.com/products/memory/dram-components/ddr4-sdram/part-catalog/part-detail/mt40a4g4sa-062e-ps-f)
- Refresh가 필요한 이유를 실험으로 보고 싶다면 Liu 등의 DDR3 연구에서 실험 방법, 온도와 패턴에 관한 결과를 읽는다. [DRAM retention 실험 논문](https://istc-cc.cmu.edu/publications/papers/2013/dram-retention-time-characterization_isca13_abs.shtml)
