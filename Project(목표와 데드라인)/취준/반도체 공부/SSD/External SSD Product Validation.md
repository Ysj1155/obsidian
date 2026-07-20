---
title: External SSD Product Validation
aliases:
  - 외장 SSD 제품 검증
  - External SSD DUT Validation
  - External SSD Black-box Validation
tags:
  - experiment
  - validation
  - ssd
  - ssd-validation
  - fio
  - portfolio
type: experiment-report
status: seed
domain: SSD Validation
created: 2026-07-15
updated: 2026-07-15
source: D:\ssd_lab\docs\reports\external_ssd_product_validation.md
source_type: local-report
reliability: draft-plan
related_projects:
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - product-engineering
  - test-engineering
---

# External SSD Product Validation

## 한 줄 결론

- External SSD Product Validation은 외장 SSD를 black-box DUT로 정의하고, fio 기반 QD sweep, sustained workload, p99/p99.9 latency, 반복성, 환경/telemetry snapshot을 요구사항과 연결하는 제품형 검증 계획이다.

## 실험 목적

- 검증 질문:
  - 실제 외장 SSD를 제품처럼 다룰 때 어떤 조건, evidence, verdict가 있어야 검증 결과를 설명할 수 있는가?
- 왜 이 실험이 필요한가:
  - 외장 SSD 결과는 SSD media뿐 아니라 USB bridge, enclosure, host controller, filesystem, OS cache, path 영향을 함께 받는다.
- 기대한 관찰:
  - QD별 random read/write 성능, sustained write/read behavior, p99/p99.9 latency, 반복 측정 CV, telemetry 가능/불가능 범위를 분리해 보고한다.

## DUT 범위

| 항목 | 값 |
|---|---|
| DUT label | `external_ssd_dut_01` |
| connection | external SSD, likely USB path |
| test model | black-box file-target fio validation |
| repo path | `D:\ssd_lab` |
| environment snapshot | `results/env/latest/` |
| telemetry snapshot | `results/telemetry/latest/` |
| out of scope | raw physical-drive destructive test, firmware modification, direct NAND/FTL/GC tracing |

## 조건 매트릭스

| Test case | Workload | Runtime | Repeats | 연결 요구사항 |
|---|---|---:|---:|---|
| `EXT-PERF-RR-QD-SWEEP` | randread 4k QD 1/4/16/32 | 60s | 3 | REQ-PERF-001, REQ-QOS-001, REQ-REPRO-001 |
| `EXT-PERF-RW-QD-SWEEP` | randwrite 4k QD 1/4/16/32 | 60s | 3 | REQ-PERF-002, REQ-QOS-001, REQ-REPRO-001 |
| `EXT-SUST-WRITE-120S` | randwrite 4k QD16 | 120s | 3 | REQ-SUST-001, REQ-QOS-001 |
| `EXT-SUST-WRITE-300S` | randwrite 4k QD16 | 300s | 3 | REQ-SUST-001, REQ-SUST-002 |
| `EXT-SUST-READ-120S` | randread 4k QD16 | 120s | 3 | REQ-SUST-003, REQ-QOS-001 |

## 실행 / 산출물

- DUT profile: `D:\ssd_lab\docs\reports\external_ssd_dut_profile.md`
- requirement matrix: `D:\ssd_lab\docs\reports\external_ssd_requirement_matrix.md`
- execution runbook: `D:\ssd_lab\docs\reports\external_ssd_execution_runbook.md`
- product validation report: `D:\ssd_lab\docs\reports\external_ssd_product_validation.md`
- config matrix: `D:\ssd_lab\configs\external_ssd_validation_matrix.yaml`
- 예상 결과:
  - fio JSON outputs
  - parsed CSV summary
  - QD plots
  - sustained time-series CSV/plots
  - environment snapshot
  - telemetry snapshot

## 실행 전 체크리스트

- 외장 SSD drive letter를 확인한다.
- test file path가 반드시 외장 SSD를 가리키는지 확인한다.
- raw physical drive path를 사용하지 않는다.
- internal OS drive를 실수로 target하지 않는다.
- free space가 충분한지 확인한다.
- 환경 snapshot을 수집한다.
- 가능하면 storage telemetry snapshot을 수집한다.
- fio smoke run 후 JSON의 `filename` field가 외장 SSD path인지 확인한다.

## 결과 요약 방식

| 관점 | 봐야 할 것 |
|---|---|
| 평균 성능 | bandwidth, IOPS |
| tail latency | p99, p99.9, max latency |
| 반복성 | repeat count, CV, run-to-run variation |
| sustained behavior | first/middle/last window, 120s vs 300s ratio |
| 환경 | OS, fio version, path, filesystem, free space |
| telemetry | SMART/NVMe health, temperature, reliability counter availability |
| limitation | 외장 path와 device-level 원인 구분 |

## 해석

- 결과가 보여주는 것:
  - 외장 SSD를 특정 host/path 조건에서 관찰한 black-box product behavior.
  - p99/p99.9, sustained behavior, 반복 안정성, 환경 context가 갖춰진 검증 evidence.
- 결과가 보여주지 못하는 것:
  - 내부 NAND, FTL, GC root cause.
  - USB bridge와 device media의 영향을 완전히 분리한 결론.
  - direct power validation. 별도 측정 장비 없이는 power claim을 하면 안 된다.
- 가장 중요한 trade-off:
  - 제품형 검증에서는 성능 숫자뿐 아니라 path, telemetry, limitation, verdict wording이 함께 있어야 한다.

## Verdict Vocabulary

- `Pass`: 필요한 evidence가 있고 정의된 rule을 만족한다.
- `Observation`: evidence는 있지만 mature threshold가 아직 없다.
- `Limited`: 테스트는 됐지만 권한, 환경, tooling 때문에 해석이 제한된다.
- `Blocked`: 테스트를 못 했거나 critical evidence가 없다.

## 이상치 / 리스크

- 이상하게 보이는 조건:
  - 높은 평균 IOPS와 높은 p99/p99.9가 함께 나타나는 조건
  - 120s보다 300s에서 write tail latency가 크게 악화되는 조건
  - telemetry가 없어 원인 추정만 가능한 조건
- 가능한 원인:
  - USB bridge/enclosure, OS cache, filesystem path, thermal behavior, sustained write pressure
- 아직 증거가 없는 원인:
  - internal GC, SLC cache exhaustion, firmware throttling
- 다음 debug step:
  - environment/telemetry snapshot 확보
  - test path 재확인
  - short smoke -> QD sweep -> sustained 순서로 단계적 실행

## 포트폴리오 / 면접 포인트

> 외장 SSD를 black-box DUT로 정의하고, fio 기반 QD sweep과 sustained workload를 요구사항 matrix에 연결했습니다. p99/p99.9 latency, 반복 측정 CV, 환경/telemetry snapshot, 해석 한계를 함께 남겨 제품형 검증 보고서 구조로 정리했습니다.

## 관련 노트

- [[SSD Mini Lab 프로젝트 허브]]
- [[fio]]
- [[Queue Depth]]
- [[p99 latency]]
- [[SSD QoS]]
- [[NVMe SMART Telemetry]]
- [[SSD Mini Lab Portfolio Evidence]]
