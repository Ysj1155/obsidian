---
title: Request Timing MVP
aliases:
  - GC Request Timing MVP
  - SSD FTL Request Timing MVP
tags:
  - experiment
  - validation
  - ssd
  - ftl
  - garbage-collection
  - timing
  - portfolio
type: experiment-report
status: growing
domain: SSD FTL Validation
created: 2026-07-15
updated: 2026-07-20
source: C:\Users\nei11\venv\venv\GC\docs\request_timing_mvp.md
source_type: local-report
reliability: simplified-model
related_projects:
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# Request Timing MVP

## 한 줄 결론

- 기존 page-level write/TRIM simulator를 `host request -> FTL state transition -> simulated completion time` 흐름으로 확장해, WAF뿐 아니라 p95/p99 latency proxy와 GC pause trade-off를 볼 수 있게 만든 단계다.

## 실험 목적

- 검증 질문:
  - 같은 workload와 seed에서 GC policy가 WAF뿐 아니라 request latency proxy와 GC pause를 어떻게 바꾸는가?
- 왜 이 실험이 필요한가:
  - GC policy trade-off를 WAF 하나만으로 보면 foreground request 지연과 tail behavior를 놓칠 수 있다.
- 기대한 관찰:
  - GC migration/erase가 발생한 request는 non-GC request보다 latency proxy가 커진다.

## 조건 매트릭스

| 항목 | 값 |
|---|---|
| request type | read / write / trim |
| request size | single-page / multi-page |
| timing model | single FIFO |
| latency source | `host_read_us`, `host_prog_us`, `erase_us`, `migrate_read_prog_us` |
| GC pause | moved valid pages + erased blocks 기반 proxy |
| artifact | `request_events.csv`, summary, validation report |
| 제외 | channel/die contention, NVMe/PCIe/OS latency, 실제 SSD latency 예측 |

## 실행 / 산출물

- 원본 문서: `C:\Users\nei11\venv\venv\GC\docs\request_timing_mvp.md`
- 정책 결과 문서: `C:\Users\nei11\venv\venv\GC\docs\request_timing_policy_findings.md`
- 실행 후보: `tools/request_timing_policy_compare.py`
- report: `tools/validation_report.py`
- 산출물:
  - `request_events.csv`
  - `gc_events.csv`
  - `summary.csv`
  - `validation_report.md`

## 결과 요약

| 구현 항목 | 의미 |
|---|---|
| `IORequest` normalization | legacy write/TRIM과 read/multi-page request를 같은 구조로 처리 |
| 비파괴 read | mapping/page/device-write 상태를 바꾸지 않고 read hit/miss 관찰 |
| sequential timing engine | arrival/start/completion/queue wait/latency 계산 |
| GC pause 연결 | 같은 request iteration에서 발생한 GC 비용을 latency에 반영 |
| report/QC integration | percentile, queue wait, pause sanity check 가능 |

## 후속 결과

- [[Request Timing Policy Findings]]에서 24-run follow-up을 통해 `greedy`, `cota`, `bsgc`, `pvb_window`의 WAF, wear, p99.9 latency proxy, metadata cost를 비교했다.
- 핵심 결론은 `cota`는 middle-ground 후보, `bsgc`는 wear-biased control, `pvb_window`는 cost와 seed sensitivity 때문에 일반 후보에서 제외하는 쪽이다.
- 이후 [[Resource Contention MVP]]에서 single FIFO timing을 PAL-lite resource contention 모델로 확장했다.

## 해석

- 결과가 보여주는 것:
  - simplified FTL model에서도 GC policy가 foreground request timing에 어떤 영향을 줄 수 있는지 관찰 가능해졌다.
  - WAF, wear, metadata IO와 p95/p99 latency proxy를 같은 report에서 비교할 수 있다.
- 결과가 보여주지 못하는 것:
  - 실제 SSD latency, NVMe queueing, channel/die parallelism, OS/filesystem latency는 아직 모델링하지 않는다.
- 가장 중요한 trade-off:
  - GC policy가 WAF를 줄여도 GC pause나 tail latency proxy가 나빠질 수 있으므로, 정책 평가는 다중 지표로 해야 한다.

## 이상치 / 리스크

- 이상하게 보이는 조건:
  - WAF는 좋은데 p99/GC pause가 큰 policy
- 가능한 원인:
  - valid migration 비용, low free-space pressure, metadata correction 비용
- 아직 증거가 없는 원인:
  - real hardware contention, firmware scheduling, die/channel-level resource conflict
- 다음 debug step:
  - [[Request Timing Policy Findings]]에서 policy verdict 확인
  - [[Resource Contention MVP]]에서 PAL-lite 추가 결과 확인
  - [[Resource Contention Quality Experiment]]에서 seed/scenario 반복성 확인

## 포트폴리오 / 면접 포인트

> FTL/GC simulator에 request timing layer를 추가해, WAF 중심 비교에서 p95/p99 latency proxy와 GC pause까지 보는 검증 흐름으로 확장했습니다. 실제 SSD latency를 예측한다고 주장하지 않고, 동일한 단순 timing 가정 안에서 policy trade-off를 재현 가능하게 비교하는 것이 목적입니다.

## 관련 노트

- [[SSD FTL-GC White-box Validation Lab]]
- [[Request Timing Policy Findings]]
- [[Resource Contention MVP]]
- [[Resource Contention Quality Experiment]]
- [[SSD Garbage Collection]]
- [[GC Pause]]
- [[Write Amplification Factor]]
- [[Temperature-Aware GC Core Findings]]
