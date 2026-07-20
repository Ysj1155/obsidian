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
status: growing
domain: SSD Validation
created: 2026-07-15
updated: 2026-07-20
source: D:\ssd_lab\docs\reports\external_ssd_product_validation.md
source_type: local-report
reliability: black-box-measurement
related_projects:
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - product-engineering
  - test-engineering
---

# External SSD Product Validation

## 한 줄 결론

- 외장 SSD를 `external_ssd_dut_01`이라는 black-box DUT로 두고 fio QD sweep, repeatability, sustained workload를 요구사항 matrix에 연결한 제품형 검증 결과다.
- 현재까지의 핵심은 random read는 QD 증가에 따라 비교적 안정적으로 scale했고, random write는 QD16 이후 처리량 이득이 제한적인 반면 QD32에서 p99/p99.9 변동성이 크게 나빠졌다는 점이다.

## 실험 목적

- 검증 질문:
  - 외장 SSD를 제품처럼 검증할 때 평균 IOPS, tail latency, 반복성, sustained behavior, 환경/telemetry 한계를 어떻게 함께 보고할 수 있는가.
- 왜 이 실험이 필요한가:
  - 외장 SSD 결과는 NAND/FTL만이 아니라 USB bridge, enclosure, host controller, filesystem, OS cache, test path 영향을 함께 받는다.
- 기대한 관찰:
  - 단순 최고 성능 조건이 아니라, 반복 가능하고 tail latency까지 설명 가능한 조건을 찾는다.

## DUT 범위

| 항목 | 값 |
|---|---|
| DUT label | `external_ssd_dut_01` |
| connection | external SSD over USB path |
| filesystem | exFAT |
| test path | `E:\validation\ssd_lab_fio_testfile` |
| test model | black-box file-target fio validation |
| repo path | `D:\ssd_lab` |
| out of scope | raw physical-drive destructive test, firmware modification, direct NAND/FTL/GC tracing |

## 실행 상태

| Test case | Workload | Runtime | Repeats | 상태 |
|---|---|---:|---:|---|
| `EXT-QD-SMOKE` | randread/randwrite 4k QD 1/4/16/32 | 30s | 1 | Done |
| `EXT-PERF-RR-QD-SWEEP` | randread 4k QD 1/4/16/32 | 30s | 3 | Done |
| `EXT-PERF-RW-QD-SWEEP` | randwrite 4k QD 1/4/16/32 | 30s | 3 | Done |
| `EXT-SUST-WRITE-120S` | randwrite 4k QD16 | 120s | 3 | Done |
| `EXT-SUST-READ-120S` | randread 4k QD16 | 120s | 3 | Done |
| `EXT-SUST-READ-QD32-120S` | randread 4k QD32 | 120s | 3 | Done |
| `EXT-SUST-WRITE-QD32-120S` | randwrite 4k QD32 | 120s | 3 | Done |
| `EXT-SUST-WRITE-QD32-300S` | randwrite 4k QD32 | 300s | 3 | Prepared |

## QD Sweep Repeat=3 핵심 결과

| workload | QD | avg IOPS | IOPS CV | p99 us | p99 CV | p99.9 us | p99.9 CV |
|---|---:|---:|---:|---:|---:|---:|---:|
| rand_read | 1 | 3959.11 | 0.016 | 350.21 | 0.031 | 547.50 | 0.023 |
| rand_read | 4 | 16382.92 | 0.004 | 389.80 | 0.006 | 531.80 | 0.022 |
| rand_read | 16 | 50430.60 | 0.006 | 703.15 | 0.007 | 976.21 | 0.005 |
| rand_read | 32 | 65959.01 | 0.002 | 1144.15 | 0.008 | 1591.98 | 0.006 |
| rand_write | 1 | 16053.86 | 0.008 | 92.33 | 0.034 | 142.34 | 0.025 |
| rand_write | 4 | 48123.72 | 0.052 | 129.02 | 0.187 | 242.35 | 0.166 |
| rand_write | 16 | 49210.83 | 0.014 | 398.00 | 0.016 | 583.00 | 0.032 |
| rand_write | 32 | 49410.50 | 0.021 | 1067.69 | 0.431 | 1600.17 | 0.456 |

## Sustained Write QD16 120s

| 지표 | 값 |
|---|---:|
| avg bandwidth | 162.96 MiB/s |
| bandwidth CV | 0.044 |
| avg IOPS | 41716.65 |
| IOPS CV | 0.044 |
| avg p99 | 516.10 us |
| p99 CV | 0.062 |
| avg p99.9 | 735.91 us |
| p99.9 CV | 0.084 |
| max latency CV | 0.770 |
| last-third IOPS / first-third IOPS | 1.058 |
| last-third avg clat / first-third avg clat | 0.949 |

## 해석

- 결과가 보여주는 것:
  - random read는 QD가 올라갈수록 IOPS가 증가했고 repeat=3 기준으로 변동성도 낮았다.
  - random write는 QD1에서 QD16까지는 처리량이 증가했지만, QD32는 QD16 대비 평균 IOPS 이득이 거의 없었다.
  - rand_write QD32는 p99 CV 0.431, p99.9 CV 0.456으로 tail latency 반복성이 가장 약했다.
  - sustained write QD16 120s에서는 last-third IOPS가 first-third보다 낮아지지 않았고, 평균 completion latency도 악화되지 않았다.
- 결과가 보여주지 못하는 것:
  - 내부 GC, SLC cache exhaustion, firmware throttling 같은 root cause는 직접 증명하지 못한다.
  - USB/exFAT/Windows file-target 조건이 섞인 black-box 결과다.
- 가장 중요한 trade-off:
  - 평균 IOPS가 높은 조건이 검증적으로 좋은 조건은 아니다. 특히 QD32 random write처럼 throughput gain은 제한적인데 tail CV가 크게 커지는 조건은 QoS 후보로 조심해야 한다.

## Evidence 위치

- QD smoke:
  - `D:\ssd_lab\results\external_ssd\qd_sweep_smoke\`
  - `D:\ssd_lab\results\external_ssd_qd_smoke_summary.csv`
  - `D:\ssd_lab\results\external_ssd_qd_smoke_grouped.csv`
- QD repeat=3:
  - `D:\ssd_lab\results\external_ssd\qd_sweep_repeat3\`
  - `D:\ssd_lab\results\external_ssd_qd_repeat3_summary.csv`
  - `D:\ssd_lab\results\external_ssd_qd_repeat3_grouped.csv`
- sustained:
  - `D:\ssd_lab\results\external_ssd\sustained_rand_write_120s_qd16_repeat3\`
  - `D:\ssd_lab\results\external_ssd_sustained_summary.csv`
  - `D:\ssd_lab\results\external_ssd_sustained_timeseries.csv`
  - `D:\ssd_lab\results\external_ssd_sustained_window_summary.csv`
  - `D:\ssd_lab\results\external_ssd_sustained_repeatability.csv`

## Requirement Verdict

| Requirement | Verdict | 메모 |
|---|---|---|
| `REQ-PERF-001` | Pass | random read QD sweep evidence 있음 |
| `REQ-PERF-002` | Pass | random write QD sweep evidence 있음 |
| `REQ-QOS-001` | Pass | p99/p99.9와 CV를 함께 보고함 |
| `REQ-REPRO-001` | Pass | repeat=3 결과와 CV 있음 |
| `REQ-SUST-001` | Pass | sustained 120s 결과 있음 |
| `REQ-SUST-002` | TBD | 300s QD32 write 실행 대기 |
| `REQ-SUST-003` | TBD | sustained read/write 비교 해석 보강 필요 |
| `REQ-ENV-001` | TBD | 환경 snapshot 정리 필요 |
| `REQ-TEL-001` | TBD | telemetry 가능/불가능 범위 정리 필요 |
| `REQ-LIMIT-001` | Pass | black-box limitation 명시 |

## 포트폴리오 / 면접 포인트

> 외장 SSD를 black-box DUT로 정의하고 fio QD sweep과 sustained workload를 수행했습니다. 평균 IOPS만 보지 않고 p99/p99.9 latency와 repeat CV를 함께 비교해, rand_write QD32처럼 처리량 이득은 제한적인데 tail latency 안정성이 나쁜 조건을 분리했습니다. 또한 USB/exFAT/Windows file-target이라는 해석 한계를 명시해 device-level claim과 observed behavior를 구분했습니다.

## 관련 노트

- [[SSD Mini Lab 프로젝트 허브]]
- [[왜 평균 IOPS만 보면 안 되는가]]
- [[fio]]
- [[Queue Depth]]
- [[p99 latency]]
- [[SSD QoS]]
- [[NVMe SMART Telemetry]]
- [[SSD Mini Lab Portfolio Evidence]]
