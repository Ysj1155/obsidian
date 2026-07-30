---
title: SSD
aliases:
  - Solid State Drive
  - 솔리드 스테이트 드라이브
  - NAND SSD
tags:
  - concept
  - ssd
  - storage
type: concept
status: growing
domain: SSD
created: 2026-07-30
updated: 2026-07-30
related_projects:
  - ssd-mini-lab
  - ftl-gc-whitebox-lab
---

# SSD

## 한 줄 정의

SSD는 NAND flash 같은 비휘발성 반도체 메모리에 데이터를 저장하고, controller와 firmware가 host I/O를 NAND 동작으로 변환해 처리하는 storage device다.

## 이 개념의 층위

- 위치: host system / storage interface / controller / NAND flash
- 누구의 동작인가: host는 logical I/O를 보내고, SSD controller가 내부 NAND 동작으로 변환한다.
- 입력: read/write/flush/TRIM 같은 host command
- 출력: data persistence, IOPS, bandwidth, latency, telemetry, error status

## 핵심 구성 요소

| 구성 요소 | 역할 | 공부 포인트 |
| --- | --- | --- |
| NAND flash | 실제 데이터를 저장하는 비휘발성 매체 | page program, block erase, P/E cycle, SLC/MLC/TLC/QLC |
| Controller | host command를 처리하고 NAND를 제어하는 SoC | queue 처리, ECC, FTL, scheduling, security, telemetry |
| Firmware | controller 위에서 동작하는 정책/상태 관리 로직 | [[FTL]], [[SSD Garbage Collection]], [[TRIM]], [[SSD Wear Leveling]] |
| DRAM / HMB | mapping table, cache, buffer 등에 쓰일 수 있는 memory | DRAM-less SSD, HMB, power loss risk |
| Interface | host와 SSD가 물리적으로 연결되는 통로 | [[SSD 인터페이스]], SATA, PCIe, U.2, M.2, E1.S |
| Protocol | host가 SSD에 명령을 주고 결과를 받는 규칙 | [[SSD 통신 프로토콜]], AHCI, NVMe |

## 왜 HDD와 다르게 봐야 하는가

HDD는 mechanical seek와 rotation이 성능의 큰 축이지만, SSD는 NAND flash의 병렬성, erase-before-write 제약, controller firmware 정책이 성능과 QoS를 좌우한다.

SSD는 기존 page를 바로 덮어쓸 수 없다. 새 page에 out-of-place write를 하고, old page는 invalid 상태가 된다. 이후 [[SSD Garbage Collection]]이 valid page를 옮기고 block erase를 수행해 free block을 만든다. 이 과정 때문에 [[Write Amplification Factor]], [[p99 latency]], [[SSD QoS]]가 중요해진다.

## 읽기/쓰기/삭제의 기본 모델

1. Host는 LBA 기준으로 read/write command를 보낸다.
2. [[SSD 통신 프로토콜]]은 command queue와 completion 방식으로 요청을 전달한다.
3. Controller의 [[FTL]]은 LBA/LPN을 NAND physical page 위치로 변환한다.
4. Read는 mapping을 따라 page를 읽고 ECC/데이터 경로 검사를 거쳐 host에 반환한다.
5. Write는 새 free page에 program하고 mapping을 갱신한다.
6. Delete 자체는 host 파일시스템 관점의 변화이고, [[TRIM]]이 전달되어야 SSD가 invalidation hint를 받을 수 있다.
7. 내부적으로 free block이 부족해지면 [[SSD Garbage Collection]]과 [[SSD Wear Leveling]]이 개입한다.

## 성능을 볼 때의 핵심 축

- bandwidth: 큰 sequential I/O에서 초당 얼마나 많은 bytes를 옮기는가
- IOPS: 작은 random I/O를 초당 몇 개 처리하는가
- latency: 각 요청이 완료되기까지 얼마나 걸리는가
- tail latency: p99/p99.9 요청이 얼마나 늦게 끝나는가
- QoS: latency가 얼마나 일관적으로 유지되는가
- endurance: NAND가 얼마나 많은 write/erase를 견디는가
- power/thermal: 전력과 온도 조건에서 성능이 얼마나 유지되는가
- telemetry: SMART/health/log로 상태를 설명할 수 있는가

## 무엇과 헷갈리기 쉬운가

- interface와 protocol:
  - PCIe/SATA는 연결 통로이고, NVMe/AHCI는 명령 규칙이다.
- SSD 성능과 NAND 성능:
  - SSD 성능은 NAND만이 아니라 controller, firmware, queue, cache, thermal, host path 영향을 함께 받는다.
- 평균 IOPS와 좋은 SSD:
  - 평균 IOPS가 높아도 p99 latency, 반복 안정성, sustained 성능이 나쁘면 검증 관점에서는 좋은 조건이 아닐 수 있다.
- TRIM과 secure erase:
  - TRIM은 invalidation hint이지 보안 삭제 보장이 아니다.

## 검증 / 실무 관점

- 실제 제품 검증에서 확인할 포인트:
  - workload별 throughput/IOPS/latency/p99/p99.9
  - QD 변화에 따른 throughput-QoS trade-off
  - sustained write에서 성능이 언제, 왜 흔들리는지
  - SMART/telemetry로 온도, media error, unsafe shutdown, wear 상태를 설명할 수 있는지
  - filesystem/cache/path/USB bridge 같은 host-side 영향을 분리했는지
- 어떤 도구/로그로 볼 수 있는가:
  - [[fio]] JSON/CSV/plot
  - OS disk info
  - SMART / NVMe health log
  - vendor/OCP telemetry

## 내 프로젝트와 연결

- [[SSD Mini Lab 프로젝트 허브]]: 실제 외장 SSD를 black-box로 측정해 평균 IOPS, p99 latency, CV를 비교한다.
- [[SSD FTL-GC White-box Validation Lab]]: FTL/GC/TRIM/WL을 단순화한 simulator로 내부 상태 전이를 관찰한다.
- 이 둘을 섞어 단정하지 않는 것이 중요하다. black-box 실험은 외부 현상이고, white-box simulator는 내부 원리를 공부하기 위한 모델이다.

## 관련 노트

- [[SSD 기본 원리와 구조]]
- [[LBA LPN PPN]]
- [[SSD Host Device Path]]
- [[Storage Performance Metrics]]
- [[SSD Telemetry]]
- [[SSD 인터페이스]]
- [[SSD 통신 프로토콜]]
- [[FTL]]
- [[TRIM]]
- [[SSD Garbage Collection]]
- [[Write Amplification Factor]]
- [[SSD QoS]]

