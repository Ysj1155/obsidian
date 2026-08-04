---
title: SSD Controller and Verification 학습 지도
aliases:
  - Controller and Verification Foundations
  - SSD Controller 검증 기초
  - SSD Controller 학습 허브
tags:
  - hub
  - ssd
  - controller
  - digital-design
  - verification
type: index
status: growing
domain: SSD Controller Validation
created: 2026-08-04
updated: 2026-08-04
source_type: official-spec-and-vendor-guide
reliability: source-checked-learning-note
related_projects:
  - ssd-mini-lab
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - rtl-verification
  - test-engineering
---

# SSD Controller and Verification 학습 지도

## 이 branch의 목적

이 branch는 일반 CS 전체를 다시 공부하는 과정이 아니다. SSD controller 안에서 요청이 **이동하고, 대기하고, 경쟁하고, 실패하고, 복구되는 과정**을 이해하기 위해 필요한 디지털 시스템과 verification 기초를 모은다.

항상 다음 네 질문으로 개념을 읽는다.

1. SSD controller의 어느 경계에 존재하는가?
2. 정상 동작에서는 어떤 상태와 신호가 변하는가?
3. 잘못되면 어떤 데이터 오류나 latency 증상이 생기는가?
4. 어떤 assertion, test, trace, metric으로 확인할 수 있는가?

## 전체 위치

```text
Host software / driver
  -> NVMe Submission Queue와 doorbell
  -> PCIe transport와 controller command fetch
  -> command decode / validation
  -> internal queue / buffer / DMA
  -> FTL address translation
  -> arbitration / NAND scheduler
  -> channel / die / plane
  -> NAND operation
  -> Completion Queue와 interrupt
```

이 그림은 학습용 기능 분해다. 실제 controller의 내부 module 경계, pipeline, SRAM/DRAM 사용법, scheduler 정책은 제품마다 다르다.

NVMe over PCIe에서 host가 memory의 Submission Queue에 command를 쓰고 doorbell을 갱신하면 controller가 command를 가져와 실행하며, 완료 후 Completion Queue entry를 host memory에 기록한다. 따라서 **NVMe SQ/CQ**와 controller 내부의 **RTL FIFO**를 같은 것으로 취급하지 않는다. 둘은 연결되지만 서로 다른 abstraction layer다.

## 학습 순서

### Phase 1: 상태와 흐름 제어

1. [[01 FSM과 SSD 상태 전이]]
2. [[02 Valid Ready Handshake와 Backpressure]]
3. `03 FIFO Buffer Queue와 Backpressure` (예정)

목표: 한 요청이 어느 상태에 있고, producer와 consumer 속도가 다를 때 왜 stall이 필요한지 설명한다.

### Phase 2: 요청 이동과 자원 선택

4. `04 Bus와 Interface Transaction` (예정)
5. `05 DMA와 Descriptor` (예정)
6. `06 Arbitration과 Scheduling` (예정)

목표: host memory의 command/data가 controller로 이동하고, 여러 요청 중 누가 내부 자원을 먼저 쓰는지 설명한다.

### Phase 3: 동시성과 진행 보장

7. `07 Concurrency Parallelism과 Event-driven Model` (예정)
8. `08 Deadlock Starvation Livelock` (예정)

목표: queueing, pipelining, 실제 병렬 자원과 단순 Python thread를 구분하고 starvation이 tail latency와 free-block shortage로 이어지는 과정을 설명한다.

### Phase 4: 검증 구조

9. `09 Testbench Reference Model Scoreboard` (예정)
10. `10 Assertion과 Invariant` (예정)
11. `11 Coverage와 Corner Case` (예정)
12. `12 Reset Timeout Fault Recovery` (예정)

목표: 테스트 개수와 coverage, invariant와 assertion, expected result와 scoreboard를 구분한다.

### Phase 5: SSD 통합

13. `13 SSD Controller Request Pipeline` (예정)
14. `14 Queue and GC Interference Practice` (예정)

목표: 새 simulator를 크게 만드는 것이 아니라 기존 [[Request Timing MVP]]와 [[Resource Contention MVP]]를 다시 읽고, 배운 구조가 현재 model에서 어디까지 구현되었는지 설명한다.

## 기존 Vault와의 연결

| 이번 학습 개념 | 먼저 연결할 기존 노트 | 연결 질문 |
| --- | --- | --- |
| FSM / commit / recovery | [[FTL]], [[Durable Request Replay]] | 언제 state가 durable해지고 ACK 가능한가? |
| FIFO / queue / QD | [[Queue Depth]], [[Request Timing MVP]] | outstanding request와 내부 FIFO depth는 어떻게 다른가? |
| Arbitration / contention | [[Resource Contention MVP]] | host I/O와 GC가 같은 resource를 원하면 누가 기다리는가? |
| Scheduling / tail | [[p99 latency]], [[SSD QoS]] | 평균 처리량이 같아도 어떤 요청이 오래 기다리는가? |
| Invariant / recovery | [[FTL Metadata Recovery and Bad Block Handling]] | restart 뒤에도 반드시 참이어야 하는 조건은 무엇인가? |
| Protocol / host path | [[SSD 통신 프로토콜]], [[SSD Host Device Path]] | host queue부터 NAND까지 어느 층을 관찰하고 있는가? |

## 노트를 공부하는 방법

각 노트는 다음 순서로 사용한다.

1. 한 문장 정의와 쉬운 비유를 읽는다.
2. signal/state 표를 손으로 다시 그린다.
3. SSD 예시에서 정상 흐름을 말로 설명한다.
4. 실패 사례에서 어떤 invariant가 깨지는지 찾는다.
5. 확인 문제의 `내 답변`을 먼저 작성한다.
6. 기존 프로젝트 trace나 test에서 실제 예를 찾는다.

## 출처를 읽는 기준

- **표준/공식 문서 사실:** NVMe queue 동작, AXI VALID/READY 규칙처럼 문서가 직접 정의한 내용이다.
- **vendor 구현 지침:** FPGA synthesis, reset, safe-state 같은 설계 권고다. 모든 ASIC/SSD 구현의 유일한 방식은 아니다.
- **SSD 학습용 적용:** 공개되지 않은 실제 controller 구현을 주장하지 않고, 개념을 이해하기 위해 만든 bounded model이다.
- **내 프로젝트 evidence:** Python simulator와 fio 결과에서 직접 관찰한 범위만 주장한다.

## 현재 진행 상태

- [x] 전체 학습 지도
- [x] [[01 FSM과 SSD 상태 전이]]
- [x] [[02 Valid Ready Handshake와 Backpressure]]
- [ ] FIFO / Buffer / Queue
- [ ] Bus transaction
- [ ] DMA
- [ ] Arbitration / Scheduling
- [ ] Concurrency / event-driven model
- [ ] Deadlock / starvation / livelock
- [ ] Verification basics
- [ ] Assertion / invariant
- [ ] Coverage
- [ ] Reset / timeout / fault recovery
- [ ] SSD controller pipeline 통합
- [ ] 기존 FTL-GC model 적용 실습

## 공식 출처

- [NVM Express Specifications](https://nvmexpress.org/specifications/)
- [NVM Express Base Specification 2.1](https://nvmexpress.org/wp-content/uploads/NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf)
- [NVMe over PCIe Transport Specification](https://nvmexpress.org/specification/nvme-over-pcie-transport-specification/)
- [Arm AMBA AXI and ACE Protocol Specification](https://developer.arm.com/-/media/Arm%20Developer%20Community/PDF/IHI0022H_amba_axi_protocol_spec.pdf)
- [AMD Vivado Synthesis UG901 - FSM Description](https://docs.amd.com/r/2022.2-English/ug901-vivado-synthesis/FSM-Description)
- [Intel Quartus Design Recommendations - State Machine HDL Guidelines](https://www.intel.com/content/www/us/en/docs/programmable/683082/25-1/state-machine-hdl-guidelines.html)

## 관련 노트

- [[SSD 허브]]
- [[10 Concepts]]
- [[SSD FTL-GC White-box Validation Lab]]
- [[SSD Mini Lab 프로젝트 허브]]
- [[SSD 통신 프로토콜]]
- [[SSD Host Device Path]]
