---
title: Resource Contention Quality Experiment
aliases:
  - PAL-lite Quality Experiment
  - Resource Contention 120-run Matrix
tags:
  - experiment
  - validation
  - ssd
  - ftl
  - garbage-collection
  - latency
type: experiment-report
status: planned
domain: SSD FTL Validation
created: 2026-07-20
updated: 2026-07-20
source: C:\Users\nei11\venv\venv\GC\docs\resource_contention_quality_experiment.md
source_type: local-report
reliability: experiment-design
related_projects:
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# Resource Contention Quality Experiment

## 한 줄 결론

- Resource Contention Quality Experiment는 PAL-lite resource model의 결과가 특정 seed나 단일 workload 착시가 아닌지 확인하기 위해, 4개 scenario x 3개 policy x 5개 seed x 2개 timing mode로 구성한 120-run 검증 설계다.

## 실험 목적

- 검증 질문:
  - serial과 PAL-lite의 latency/resource wait 차이가 workload shape, burst, seed 변화에도 유지되는가.
- 왜 이 실험이 필요한가:
  - [[Resource Contention MVP]]는 모델 확장의 가능성을 보여줬지만, 단일 조건만 보면 우연한 resource mapping이나 overload 착시일 수 있다.
- 기대한 관찰:
  - FTL core 결과는 보존되고, resource contention 지표만 timing mode에 따라 달라져야 한다.

## 조건 매트릭스

| 축 | 값 |
|---|---|
| scenarios | `random_update_stress`, `low_op_pressure`, `mixed_read_write_pressure`, `trim_phase_pressure` |
| policies | `greedy`, `cota`, `bsgc` |
| seeds | 41, 42, 43, 44, 45 |
| timing modes | `serial`, `pal_lite` |
| run count | 120 |
| ops per run | 30,000 |
| one-page arrival | 150 us |
| mixed request arrival | 250 us steady + 50 us burst |
| event mode | GC-focused request events plus full GC events |

## Acceptance Checks

| 체크 | 의미 |
|---|---|
| FTL 결과 보존 | serial/PAL-lite pair에서 WAF, GC count, wear std/spread가 같아야 한다. |
| 방향성 반복 | policy 결론이 최소 4/5 seed에서 같은 방향이어야 한다. |
| overload 분리 | mixed request size와 burst가 persistent overload로 바뀌면 안 된다. |
| 다중 지표 보고 | p95/p99.9, GC-pause p99.9, conditional GC-resource wait, WAF, GC count, wear를 함께 본다. |
| CI 해석 | 신뢰구간은 증명 도구가 아니라 sign crossing을 보는 진단 도구로 사용한다. |

## 해석

- 결과가 보여주는 것:
  - resource contention 가정이 policy별 tail latency와 GC-resource wait 해석에 어떤 영향을 주는지 비교할 수 있다.
  - 같은 simulator state에서 timing model만 바꾸는 sensitivity test가 된다.
- 결과가 보여주지 못하는 것:
  - 실제 SSD firmware scheduler, NVMe path, NAND physics를 대변하지 않는다.
- 가장 중요한 trade-off:
  - policy verdict는 WAF 하나가 아니라 WAF, wear, tail latency, GC-resource wait이 함께 같은 방향으로 설명될 때만 강해진다.

## 내가 공부할 때 볼 순서

1. [[Request Timing MVP]]
2. [[Request Timing Policy Findings]]
3. [[Resource Contention MVP]]
4. 이 노트

## 포트폴리오 / 면접 포인트

> 단일 결과가 아니라 scenario, policy, seed, timing mode를 나눈 120-run matrix로 resource contention 가정의 안정성을 검증하도록 설계했습니다. 특히 WAF/GC/wear가 보존되는지 먼저 확인하고, 그 위에서 p99.9와 GC-resource wait 차이를 해석하는 순서로 검증 품질을 관리했습니다.

## 관련 노트

- [[SSD FTL-GC White-box Validation Lab]]
- [[Resource Contention MVP]]
- [[Request Timing MVP]]
- [[Request Timing Policy Findings]]
- [[GC Pause]]
- [[SSD QoS]]
