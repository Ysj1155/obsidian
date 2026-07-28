---
title: SSD Mini Lab Portfolio Evidence
aliases:
  - SSD Mini Lab 포트폴리오 근거
  - fio 검증 포트폴리오 Evidence
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
updated: 2026-07-15
source: D:\ssd_lab\docs\reports\portfolio_evidence.md
source_type: local-report
reliability: personal-experiment
related_projects:
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - product-engineering
  - test-engineering
---

# SSD Mini Lab Portfolio Evidence

## 한 줄 결론

- 이 프로젝트의 포트폴리오 가치는 많은 그래프가 아니라, `condition -> execution -> parsed result -> graph -> interpretation boundary`로 이어지는 검증 흐름을 만든 데 있다.

## 실험 목적

- 검증 질문:
  - 실제 SSD를 black-box DUT로 두고, fio workload 결과를 어떻게 재현 가능하고 해석 가능한 evidence로 만들 수 있는가?
- 왜 이 실험이 필요한가:
  - 평균 IOPS만으로는 QoS, 반복성, cache/path 영향, sustained behavior를 설명하기 어렵다.
- 기대한 관찰:
  - QD, direct mode, path, runtime 조건에 따라 평균 지표와 tail latency가 다르게 움직인다.

## 조건 매트릭스

| Track | 질문 | 주요 산출물 |
|---|---|---|
| Baseline fio | 첫 성능 profile은 어떤가 | `results/fio_summary.csv`, `docs/reports/baseline_v1.md` |
| Queue-depth sweep | QD가 IOPS와 p99에 어떤 영향을 주는가 | `results/qd_sweep_grouped.csv` |
| QD reproducibility | 반복 측정 안정성은 어떤가 | `results/qd_sweep_reproducibility.csv` |
| Direct vs buffered | OS/filesystem cache 영향은 어느 정도인가 | `results/direct_buffered_comparison.csv` |
| WSL path comparison | path layer가 fio behavior를 바꾸는가 | `results/wsl_path_compare_comparison.csv` |
| QoS review | 평균 IOPS 밖의 위험 조건은 무엇인가 | `results/qos_tail_latency_summary.csv` |
| Sustained workload | runtime이 길어지면 tail behavior가 악화되는가 | `docs/reports/sustained_workload_week10.md` |
| Telemetry recon | 환경/telemetry를 안전하게 수집할 수 있는가 | `results/telemetry/latest/` |

## 실행 / 산출물

- 원본 리포트: `D:\ssd_lab\docs\reports\portfolio_evidence.md`
- 주요 README: `D:\ssd_lab\README.md`
- raw 결과: `D:\ssd_lab\results\`
- 분석 스크립트:
  - `parse_fio_results.py`
  - `analyze_qd_sweep.py`
  - `analyze_qd_reproducibility.py`
  - `analyze_direct_buffered.py`
  - `analyze_qos_tail_latency.py`
  - `analyze_sustained_smoke.py`

## 결과 요약

| 관찰 | 의미 | 연결 노트 |
|---|---|---|
| QD를 높이면 IOPS가 증가할 수 있음 | 병렬성 증가 효과 | [[Queue Depth]] |
| `rand_write QD32`에서 p99가 크게 증가 | 최고 IOPS가 좋은 조건은 아님 | [[왜 평균 IOPS만 보면 안 되는가]] |
| `direct=0`은 좋아 보이는 성능을 낼 수 있음 | OS/filesystem cache 영향 가능성 | [[SSD Mini Lab 프로젝트 허브]] |
| sustained write 300s에서 tail latency 악화 | 짧은 benchmark로 장기 QoS를 단정하면 안 됨 | [[SSD QoS]] |

## 해석

- 결과가 보여주는 것:
  - fio 결과를 raw JSON에서 CSV/plot/report로 이어지는 auditable flow로 만들었다.
  - 평균 성능, tail latency, 반복성, path/cache 영향을 분리해 보려는 검증 관점을 세웠다.
- 결과가 보여주지 못하는 것:
  - 내부 FTL/GC 원인, SLC cache, thermal throttling, firmware scheduling을 직접 증명하지는 못한다.
- 가장 중요한 trade-off:
  - 높은 평균 IOPS와 낮은 p99/p99.9 latency, 반복 안정성은 항상 같이 움직이지 않는다.

## 이상치 / 리스크

- 이상하게 보이는 조건:
  - 평균 성능은 좋지만 p99/p99.9 또는 CV가 흔들리는 조건
- 가능한 원인:
  - queueing delay, OS cache, filesystem path, sustained write pressure
- 아직 증거가 없는 원인:
  - 내부 GC, thermal throttling, SLC cache exhaustion
- 다음 debug step:
  - telemetry snapshot, longer sustained workload, path/mode 별 반복 측정 보강

## 포트폴리오 / 면접 포인트

> fio 기반 SSD mini-lab에서 raw JSON을 보존하고 CSV/plot/report로 이어지는 검증 흐름을 만들었습니다. 평균 IOPS뿐 아니라 p99/p99.9 latency와 반복 측정 CV를 함께 봤고, direct/buffered와 path 차이를 분리해 결과 해석의 경계를 문서화했습니다.

## 관련 노트

- [[SSD Mini Lab 프로젝트 허브]]
- [[왜 평균 IOPS만 보면 안 되는가]]
- [[SSD Mini Lab Stage 1 회고]]
- [[SSD FTL-GC White-box Validation Lab]]
