---
title: GC Pause
aliases:
  - Garbage Collection Pause
  - GC latency pause
  - GC pause latency
tags:
  - validation
  - ssd-validation
  - ftl
  - garbage-collection
  - latency
type: validation-point
status: growing
domain: SSD FTL Validation
created: 2026-07-15
updated: 2026-07-15
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

# GC Pause

## 한 줄 결론

- GC Pause는 foreground request 처리 중 GC migration/erase 비용이 request latency에 붙는 현상을 관찰하기 위한 지표이며, WAF만으로는 보이지 않는 tail latency trade-off를 드러낸다.

## 검증 질문

- 무엇을 확인하려는가:
  - GC가 발생한 request가 non-GC request보다 얼마나 더 늦어지는가.
- 왜 이 검증이 필요한가:
  - GC policy는 WAF를 줄여도 foreground latency를 악화시킬 수 있다.
- 좋은 결과/나쁜 결과를 어떻게 구분하는가:
  - 좋은 결과는 GC pause rate와 pause magnitude가 낮고 p95/p99 latency proxy가 안정적인 조건이다.
  - 나쁜 결과는 WAF는 좋아도 GC pause request가 많거나 p99 proxy가 커지는 조건이다.

## 테스트 조건

- workload:
  - update-heavy, trim locality, low OP pressure
- block size:
  - model에서는 page 단위 request size
- queue depth:
  - MVP에서는 run-level 입력으로 기록하되 timing engine은 single FIFO
- direct mode:
  - 해당 없음. white-box simulator 내부 지표
- runtime:
  - ops count / request stream 길이
- test path:
  - 해당 없음
- 반복 횟수:
  - seed 반복 필요
- 환경/제약:
  - simplified model, 실제 SSD latency 예측 아님

## 봐야 할 지표

- 평균:
  - throughput proxy
  - average latency proxy
- tail latency:
  - p95
  - p99
  - max latency proxy
- 변동성/CV:
  - seed variance
- 에러/로그:
  - timing QC
  - request count consistency
- 보조 지표:
  - GC pause request rate
  - pause p95/max/sum
  - moved valid pages
  - erased blocks
  - WAF

## 관찰한 결과

- Request + Timing MVP에서 GC migration과 erase 비용을 request latency proxy에 연결했다.
- `gc_pause_us = moved_valid_pages * migrate_read_prog_us + erased_blocks * erase_us` 형태로 단순화했다.
- 이 지표를 통해 policy 비교가 WAF에서 p95/p99 latency proxy까지 확장됐다.

## 해석 기준

- GC Pause는 실제 SSD latency가 아니라 동일한 단순 timing 가정 아래에서 policy 차이를 비교하는 proxy다.
- GC pause가 큰 조건은 foreground request가 GC 비용을 함께 부담할 가능성을 보여준다.
- WAF와 GC pause는 관련 있지만 항상 같은 방향으로 움직인다고 단정하면 안 된다.
- 실제 hardware에서는 channel/die contention, firmware scheduling, NVMe queueing이 추가로 필요하다.

## 내 프로젝트와 연결

- 관련 프로젝트:
  - [[SSD FTL-GC White-box Validation Lab]]
- 관련 실험/결과물:
  - [[Request Timing MVP]]
  - [[SSD Garbage Collection]]
  - [[Write Amplification Factor]]
- 포트폴리오/면접에서 쓸 포인트:
  - GC policy를 WAF뿐 아니라 foreground latency proxy와 GC pause 관점으로 확장했다.

## 면접/자소서로 바꿀 수 있는 문장

> FTL/GC simulator에 request timing layer를 추가하면서, GC가 발생한 request의 latency proxy에 migration과 erase 비용을 반영했습니다. 이를 통해 WAF 중심 비교에서 p95/p99 latency proxy와 GC pause trade-off까지 보는 검증 흐름으로 확장했습니다.

## 한계와 다음 확인

- 아직 단정하면 안 되는 부분:
  - 실제 SSD latency 수치로 해석하면 안 된다.
- 추가로 확인할 실험:
  - 72-run policy comparison의 p95/p99/GC-pause trade-off 분석
  - Resource Contention MVP에서 PAL-lite 추가
- 더 필요한 telemetry / log / metric:
  - channel/die resource model
  - request_events.csv
  - gc_events.csv

## 관련 노트

- [[Request Timing MVP]]
- [[SSD Garbage Collection]]
- [[Write Amplification Factor]]
