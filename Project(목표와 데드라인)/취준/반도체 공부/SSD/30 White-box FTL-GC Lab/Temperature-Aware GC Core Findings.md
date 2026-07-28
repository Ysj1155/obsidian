---
title: Temperature-Aware GC Core Findings
aliases:
  - Temp-Aware GC Core Findings
  - Temperature-Aware GC Findings
tags:
  - experiment
  - validation
  - ssd
  - ftl
  - garbage-collection
  - portfolio
type: experiment-report
status: growing
domain: SSD FTL Validation
created: 2026-07-15
updated: 2026-07-15
source: C:\Users\nei11\venv\venv\GC\docs\temp_aware_core_findings.md
source_type: local-report
reliability: simplified-model
related_projects:
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# Temperature-Aware GC Core Findings

## 한 줄 결론

- `temp_aware`는 update-heavy workload에서 WAF 개선 신호가 있었지만 low OP pressure와 TRIM-hot 조건에서 비용이 커져, 기본 정책이 아니라 feature와 boundary를 더 분석해야 할 실험 후보로 정리했다.

## 실험 목적

- 검증 질문:
  - FTL-observed data temperature를 GC victim score에 넣으면 어떤 workload에서 도움이 되고 어디서 깨지는가?
- 왜 이 실험이 필요한가:
  - hot/cold data 분리는 GC/Wear 정책에서 매력적인 feature지만, invalid 회수 효율보다 우선되면 WAF가 악화될 수 있다.
- 기대한 관찰:
  - update-heavy workload에서는 temperature signal이 도움이 될 수 있지만, low OP pressure에서는 valid migration 비용이 커질 수 있다.

## 조건 매트릭스

| 항목 | 값 |
|---|---|
| result dir | `results/temp_aware_core_jobs3` |
| scenarios | random_update_stress, low_op_pressure, trim_locality_hot, trim_locality_cold, wear_leveling_on |
| policies | greedy, bsgc, re50315, cota, temp_aware |
| seeds | 41, 42, 43 |
| runs | 75/75 |
| 주요 지표 | WAF, wear_std, reclaim rate, pending reclaim, WL moved pages |

## 실행 / 산출물

- 원본 문서: `C:\Users\nei11\venv\venv\GC\docs\temp_aware_core_findings.md`
- 관련 trace: `gc_events.csv`
- 추가 instrumentation:
  - `v_invalid_ratio`
  - `v_observed_temp_ewma`
  - `v_observed_coldness`
  - `score_invalid_ratio`
  - `score_observed_temperature`
  - `score_score`

## 결과 요약

| 시나리오 | 관찰 | 해석 |
|---|---|---|
| random_update_stress | `temp_aware`가 WAF 1등 | update-heavy에서는 temperature signal이 GC 효율에 도움 가능 |
| low_op_pressure | `temp_aware`가 가장 나쁜 WAF | coldness bias가 invalid 회수 효율을 손상시킬 수 있음 |
| trim_locality_cold/hot | reclaim rate는 좋고 pending reclaim은 낮음 | TRIM reclaim에는 신호가 있으나 WAF 비용 존재 |
| wear_leveling_on | bsgc보다 WAF와 WL moved pages 비용 큼 | static WL과 같이 쓰려면 bias 조정 필요 |

## 해석

- 결과가 보여주는 것:
  - data temperature feature는 일부 workload에서 의미 있는 신호가 될 수 있다.
  - 하지만 feature를 victim score에 바로 넣으면 low OP pressure에서 valid migration 비용을 키울 수 있다.
- 결과가 보여주지 못하는 것:
  - `temp_aware`가 전반적으로 더 좋은 정책이라는 결론은 낼 수 없다.
  - 현재 metric만으로 victim 선택 원인을 완전히 설명할 수는 없다.
- 가장 중요한 trade-off:
  - cold data를 고려하는 것과 invalid page 회수 효율 사이의 우선순위가 중요하다.

## 이상치 / 리스크

- 이상하게 보이는 조건:
  - random_update에서는 좋지만 low_op_pressure에서는 나쁜 양면성
- 가능한 원인:
  - coldness bonus가 invalid ratio보다 victim 선택을 더 흔듦
- 아직 증거가 없는 원인:
  - 실제 SSD firmware에서 같은 방식으로 동작한다는 주장
- 다음 debug step:
  - victim trace로 low OP pressure에서 비싼 victim을 고르는 조건 분석
  - invalid ratio gate를 둔 `temp_aware_2stage`와 비교

## 포트폴리오 / 면접 포인트

> Temperature-aware GC policy를 실험하면서 update-heavy workload에서는 WAF 개선 신호를 확인했지만, low OP pressure에서는 valid migration 비용이 커지는 failure mode도 확인했습니다. 그래서 이 정책을 성공으로 포장하지 않고, 어떤 조건에서 feature가 유효하고 어디서 깨지는지 trace와 metric으로 분리해 해석했습니다.

## 관련 노트

- [[SSD FTL-GC White-box Validation Lab]]
- [[SSD Garbage Collection]]
- [[SSD Wear Leveling]]
- [[Data Temperature]]
- [[Request Timing MVP]]
