---
title: SSD Garbage Collection
aliases:
  - SSD GC
  - Garbage Collection
  - 가비지 컬렉션
tags:
  - concept
  - ssd
  - ftl
  - garbage-collection
  - ssd-validation
type: concept
status: growing
domain: SSD FTL
created: 2026-02-21
updated: 2026-07-15
source:
source_type:
reliability:
related_projects:
  - ftl-gc-whitebox-lab
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# SSD Garbage Collection

## 한 줄 정의

- SSD Garbage Collection은 invalid page가 많은 block을 골라 valid page만 다른 곳으로 옮긴 뒤 block 단위 erase를 수행해 free block을 회수하는 FTL 내부 관리 동작이다.

## 이 개념의 층위

- 위치: 컨트롤러 / FTL / NAND / 검증
- 누구의 동작인가: device / firmware
- 입력: invalid page 분포, free block 압력, GC threshold, workload update pattern, TRIM 정보
- 출력: free block 회수, valid page migration, block erase, WAF 증가, latency spike 가능성

## 왜 중요한가

- NAND flash는 page 단위 program은 가능하지만 기존 page overwrite는 불가능하고 erase는 block 단위로 수행된다.
- host overwrite가 많아지면 old page는 invalid가 되고, SSD는 언젠가 invalid page를 정리해야 한다.
- GC는 free space를 확보하지만 valid page migration 때문에 write amplification과 tail latency를 만들 수 있다.
- SSD validation에서는 평균 성능뿐 아니라 GC active 조건에서 QoS가 어떻게 흔들리는지 봐야 한다.

## 핵심 동작 / 원리

1. FTL이 victim block 후보를 고른다.
2. victim block 안의 valid page를 다른 free page로 migration한다.
3. mapping / reverse mapping을 새 physical location으로 갱신한다.
4. victim block을 erase하고 free block pool로 돌려보낸다.
5. 이 과정에서 내부 write와 erase가 발생해 WAF, GC pause, p99 latency에 영향을 줄 수 있다.

## 대표 시나리오

- random write / overwrite workload에서 invalid page가 빠르게 쌓인다.
- free block이 부족해지면 foreground I/O 도중 GC가 끼어들 수 있다.
- sustained write에서는 시간이 지나며 GC pressure가 커지고 tail latency가 나빠질 수 있다.
- TRIM이 잘 전달되면 FTL이 invalid data를 더 빨리 알 수 있어 GC reclaim 효율이 좋아질 수 있다.

## 무엇과 헷갈리기 쉬운가

- [[SSD Trim]] 와의 차이:
  - TRIM은 host가 더 이상 필요 없는 logical block을 SSD에 알려주는 명령이고, GC는 SSD 내부에서 실제 block erase와 free block 회수를 수행하는 동작이다.
- [[SSD Wear Leveling]] 와의 차이:
  - GC는 공간 회수 중심이고, Wear Leveling은 erase count를 균등하게 만드는 수명 관리 중심이다. 실제 정책에서는 둘의 목적이 충돌할 수 있다.
- 자주 하는 오해:
  - GC가 항상 background에서만 조용히 돈다고 보면 안 된다.
  - GC 결과만 보고 내부 원인을 단정하면 안 된다. 외부 fio 실험에서는 OS/path/cache 영향도 섞일 수 있다.

## 관련 개념

- 상위 개념:
  - [[FTL]]
- 인접 개념:
  - [[SSD Trim]]
  - [[SSD Wear Leveling]]
  - [[Write Amplification Factor]]
- 결과적으로 연결되는 것:
  - [[GC Pause]]
  - [[p99 latency]]
  - [[SSD QoS]]

## 검증 / 실무 관점

- 실제 제품 검증에서 확인할 포인트:
  - sustained write 중 throughput drop과 p99/p99.9 latency 악화
  - preconditioning 이후 steady-state 성능
  - random write 조건에서 run-to-run variation
- 어떤 로그/메트릭/명령으로 볼 수 있는가:
  - black-box: fio latency percentile, bandwidth/IOPS time series, SMART/NVMe telemetry
  - white-box model: gc_events.csv, moved valid pages, erase count, WAF, GC pause
- 실무에서 왜 문제가 되는가:
  - 평균 IOPS가 좋아도 GC가 일부 request를 크게 지연시키면 QoS 문제가 된다.
- 브로슈어/스펙/테스트 항목에서 어떻게 나타나는가:
  - random write IOPS, sustained write, QoS, endurance, DWPD 같은 항목과 연결된다.

## 내 프로젝트와 연결

- 연결되는 프로젝트:
  - [[SSD FTL-GC White-box Validation Lab]]
  - [[SSD Mini Lab 프로젝트 허브]]
- 연결되는 실험/결과:
  - [[Request Timing MVP]]
  - [[Temperature-Aware GC Core Findings]]
  - [[SSD Mini Lab Stage 1 회고]]
- 자소서/면접에서 설명할 때 쓸 수 있는 문장:
  - SSD GC는 단순한 정리 작업이 아니라 free block 확보, WAF, tail latency, wear balance가 함께 얽히는 검증 포인트라고 이해하고 있습니다.

## 한계 / 예외 / 조건

- GC 동작 방식은 vendor firmware마다 다르며 외부 fio 결과만으로 내부 GC 원인을 직접 증명할 수 없다.
- Python simulator의 GC metric은 내부 동작을 이해하기 위한 simplified model이지 실제 SSD firmware 증거가 아니다.
- 실제 장치에서는 SLC cache, thermal throttling, firmware scheduling, host path 영향도 함께 고려해야 한다.

## 출처 / 참고

- [[SSD FTL-GC White-box Validation Lab]]
- [[SSD Mini Lab 프로젝트 허브]]
- `C:\Users\nei11\venv\venv\GC\docs\portfolio_gc_evidence.md`
