---
title: PVB Metadata Model
aliases:
  - PVB
  - Page Validity Bitmap
  - Lazy Gecko-lite Metadata Model
tags:
  - concept
  - ssd
  - ftl
  - metadata
  - garbage-collection
  - ssd-validation
type: concept
status: growing
domain: SSD FTL
created: 2026-07-15
updated: 2026-07-15
source: C:\Users\nei11\venv\venv\GC\docs\lazy_gecko_lite.md
source_type: local-report
reliability: simplified-model
related_projects:
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# PVB Metadata Model

## 한 줄 정의

- PVB Metadata Model은 FTL이 block/page valid 상태를 항상 완벽히 아는 것이 아니라, Page Validity Bitmap과 lazy correction을 통해 stale metadata와 correction cost를 관찰하는 모델이다.

## 이 개념의 층위

- 위치: FTL / metadata / GC victim selection / 검증
- 누구의 동작인가: device / firmware model
- 입력: invalidation event, pending invalid page, reverse-map metadata, GC candidate block
- 출력: PVB invalid count, pending invalid count, correction event, reverse-map read/write cost

## 왜 중요한가

- 기존 simulator는 GC policy가 valid/invalid 상태를 완벽히 안다고 가정했다.
- 실제 flash-resident page-mapping FTL에서는 mapping table 대부분이 flash에 있고 RAM에는 일부 metadata만 있을 수 있다.
- 그러면 victim selection 시점의 metadata가 stale할 수 있고, 정확도를 높이려면 reverse-map correction 비용을 지불해야 한다.
- PVB 모델은 victim 품질과 metadata IO 비용 사이의 trade-off를 보이게 한다.

## 핵심 동작 / 원리

1. `exact` mode에서는 simulator가 block valid/invalid 상태를 정확히 안다고 본다.
2. `lazy_gecko` mode에서는 invalidation을 PVB에 즉시 반영하지 않고 pending으로 둔다.
3. GC victim을 고르는 시점에 일부 후보 block의 pending invalid를 correction한다.
4. correction은 victim 품질을 높일 수 있지만 reverse-map metadata read/write 비용을 만든다.
5. `pvb_window`는 top-K 후보만 correction해 전체 scan과 fallback 사이의 절충을 본다.

## 대표 시나리오

- PVB가 아직 invalid signal을 모르기 때문에 `pvb_greedy`가 exact greedy fallback에 의존하는 구간
- `pvb_window`가 top-K 후보만 correction해 metadata cost를 제한하는 구간
- decision-boundary gate가 correction 대상 후보를 줄여 metadata read를 줄이는 구간

## 무엇과 헷갈리기 쉬운가

- exact metadata와의 차이:
  - exact는 ground truth를 바로 아는 가정이고, PVB는 stale metadata와 correction cost를 명시적으로 본다.
- policy success와의 차이:
  - PVB를 넣었다고 더 좋은 policy가 된다는 뜻은 아니다. 비용과 failure mode를 관찰하기 위한 모델이다.
- 자주 하는 오해:
  - PVB 결과를 실제 Lazy Gecko 구현 검증으로 말하면 안 된다. 현재는 simplified observation model이다.

## 관련 개념

- 상위 개념:
  - [[FTL]]
- 인접 개념:
  - [[SSD Garbage Collection]]
  - [[Write Amplification Factor]]
  - [[GC Pause]]
- 결과적으로 연결되는 것:
  - metadata IO cost
  - victim selection quality
  - [[Request Timing MVP]]

## 검증 / 실무 관점

- 실제 제품 검증에서 확인할 포인트:
  - stale metadata가 victim selection을 얼마나 흔드는가
  - correction window가 victim 품질을 올리는 대신 metadata IO를 얼마나 쓰는가
  - fallback exact에 의존하는 비율이 얼마나 되는가
- 어떤 로그/메트릭/명령으로 볼 수 있는가:
  - lazy_gecko_pending_invalid_pages
  - lazy_gecko_reverse_map_reads/writes
  - pre_pvb_exact_best_match
  - pvb_window_reverse_map_reads
  - pvb_window_corrected_pages
- 실무에서 왜 문제가 되는가:
  - metadata 현실성을 무시하면 GC policy 비교가 지나치게 이상적인 조건이 될 수 있다.

## 내 프로젝트와 연결

- 연결되는 프로젝트:
  - [[SSD FTL-GC White-box Validation Lab]]
- 연결되는 실험/결과:
  - [[Request Timing MVP]]
  - [[SSD Garbage Collection]]
- 자소서/면접에서 설명할 때 쓸 수 있는 문장:
  - GC policy가 완벽한 metadata를 항상 볼 수 있다는 가정을 완화하고, PVB stale metadata와 correction cost가 victim selection에 미치는 영향을 trace로 관찰했습니다.

## 한계 / 예외 / 조건

- 논문 Lazy Gecko를 그대로 복제한 구현이 아니라 핵심 아이디어를 단순화한 관찰 모델이다.
- 실제 firmware metadata layout, cache hit/miss, crash consistency까지 포함하지 않는다.
- PVB tuning 결과는 범용 default policy 성공이 아니라 workload-conditional evidence로 해석해야 한다.

## 출처 / 참고

- [[SSD FTL-GC White-box Validation Lab]]
- `C:\Users\nei11\venv\venv\GC\docs\lazy_gecko_lite.md`
