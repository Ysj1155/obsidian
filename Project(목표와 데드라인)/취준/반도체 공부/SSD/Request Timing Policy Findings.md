---
title: Request Timing Policy Findings
aliases:
  - Request Timing 24-run Findings
  - GC Timing Policy Findings
tags:
  - experiment
  - validation
  - ssd
  - ftl
  - garbage-collection
  - latency
type: experiment-report
status: completed
domain: SSD FTL Validation
created: 2026-07-20
updated: 2026-07-20
source: C:\Users\nei11\venv\venv\GC\docs\request_timing_policy_findings.md
source_type: local-report
reliability: simplified-model
related_projects:
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# Request Timing Policy Findings

## 한 줄 결론

- Request Timing Policy Findings는 24-run follow-up에서 `greedy`, `cota`, `bsgc`, `pvb_window`를 같은 timing 가정 아래 비교한 결과이며, `cota`는 중간형 후보, `bsgc`는 wear-biased control, `pvb_window`는 일반 후보에서 제외하는 쪽으로 정리됐다.

## 실험 목적

- 검증 질문:
  - Request + Timing layer를 붙인 뒤에도 GC policy별 WAF, wear, p99.9 latency proxy, metadata cost trade-off가 설명 가능한가.
- 왜 이 실험이 필요한가:
  - [[Request Timing MVP]]가 기능 구현이라면, 이 노트는 실제 policy verdict를 뽑는 첫 정리다.
- 기대한 관찰:
  - 기존 WAF/GC/wear 결과는 유지하면서 latency proxy와 metadata cost가 policy별 차이를 드러내야 한다.

## 조건 매트릭스

| 항목 | 값 |
|---|---|
| scenarios | `random_update_stress`, `low_op_pressure` |
| policies | `greedy`, `cota`, `bsgc`, `pvb_window` |
| seeds | 41, 42, 43 |
| runs | 24 |
| metadata mode | `lazy_gecko` |
| QC | strict QC, manifest, GC event, request event |
| 한계 | 실제 SSD latency 예측이 아니라 single-FIFO cost assumption 비교 |

## 결과 요약

| policy | 해석 |
|---|---|
| `greedy` | 성능 baseline. 낮은 WAF와 안정적인 request tail, 낮은 Python GC cost가 장점이다. |
| `cota` | middle-ground 후보. wear std를 줄이면서 WAF/p99.9 비용은 상대적으로 작다. |
| `bsgc` | wear-biased control. wear 개선은 강하지만 migration/WAF/tail latency 비용이 크다. |
| `pvb_window` | 일반 후보에서 제외. random-update에서는 greedy와 비슷하지만 Python GC time과 metadata read 비용이 컸고 low-OP에서 seed-sensitive regression이 있었다. |

## 핵심 숫자

| scenario | policy | WAF | wear std | p99.9 latency proxy |
|---|---|---:|---:|---:|
| low OP | greedy | 1.004650 | 0.737004 | 1750 us |
| low OP | cota | 1.007667 | 0.662688 | 1850 us |
| low OP | bsgc | 1.037417 | 0.426032 | 3100 us |
| low OP | pvb_window | 1.010783 | 0.764845 | 2050 us |
| random update | greedy | 1.004645 | 0.840377 | 1900 us |
| random update | cota | 1.006448 | 0.789624 | 2000 us |
| random update | bsgc | 1.012861 | 0.796800 | 2250 us |
| random update | pvb_window | 1.004703 | 0.846420 | 1900 us |

## 해석

- 결과가 보여주는 것:
  - `cota`는 greedy보다 wear를 줄이면서 tail cost가 제한적인 middle-ground로 볼 수 있다.
  - `bsgc`는 wear 개선이 강하지만 그 대가로 WAF와 tail latency proxy가 커진다.
  - `pvb_window`는 아이디어 자체보다 cost와 seed sensitivity를 드러낸 실패/제외 evidence로 가치가 있다.
- 결과가 보여주지 못하는 것:
  - 실제 hardware latency 분포나 firmware scheduling 결과는 아니다.
- 가장 중요한 trade-off:
  - 좋은 policy는 WAF 하나로 정하지 않고, wear 개선의 값어치가 tail latency와 metadata cost를 감당할 만큼 있는지 같이 판단해야 한다.

## Tail 해석 주의

- p99.9는 약 20k request 중 상위 약 20개를 보는 값이다.
- conditional GC-pause percentile은 nonzero event 수가 약 195-233개라 p99.9가 max에 가깝다.
- 따라서 p99.9는 rare worst-case screening에는 좋지만 안정적인 분포 증명으로 과장하면 안 된다.

## 다음 질문

- `cota`와 `bsgc`의 rare high-cost victim이 PAL-lite resource model에서 host I/O와 충돌해 tail regression으로 이어지는가.
- 이 질문은 [[Resource Contention MVP]]와 [[Resource Contention Quality Experiment]]로 이어진다.

## 포트폴리오 / 면접 포인트

> Request timing follow-up에서 GC policy를 WAF만으로 평가하지 않고 wear, p99.9 latency proxy, metadata cost를 함께 비교했습니다. 특히 `pvb_window`처럼 수치 일부는 좋아 보여도 cost와 seed sensitivity 때문에 제외해야 하는 후보를 분리해, 검증에서 중요한 것은 최고 수치가 아니라 반복 가능하고 설명 가능한 trade-off라는 점을 확인했습니다.

## 관련 노트

- [[SSD FTL-GC White-box Validation Lab]]
- [[Request Timing MVP]]
- [[Resource Contention MVP]]
- [[GC Pause]]
- [[Write Amplification Factor]]
- [[PVB Metadata Model]]
