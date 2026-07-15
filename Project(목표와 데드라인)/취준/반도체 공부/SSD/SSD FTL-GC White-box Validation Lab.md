---
title: SSD FTL-GC White-box Validation Lab
aliases:
  - GC Lab
  - FTL GC White-box Lab
  - SSD GC Simulator Project
tags:
  - hub
  - project
  - ssd
  - ftl
  - garbage-collection
  - ssd-validation
  - portfolio
type: project
status: growing
domain: SSD FTL Validation
created: 2026-07-15
updated: 2026-07-15
source: C:\Users\nei11\venv\venv\GC
source_type: local-project
reliability: simplified-model
related_projects:
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# SSD FTL-GC White-box Validation Lab

## 한 줄 요약

- SSD controller 내부의 FTL, GC, TRIM, wear, metadata, request timing 흐름을 Python simulator로 단순화하고, 같은 workload와 seed에서 정책 trade-off를 검증하는 white-box validation 프로젝트.

## 이 프로젝트의 목적

- 실제 SSD firmware라고 주장하는 것이 아니라, 내부 동작을 관찰 가능한 모델로 만들어 검증 관점을 훈련한다.
- 핵심은 새 GC 알고리즘 하나를 주장하는 것이 아니라, FTL 상태 변화와 GC/TRIM/wear/metadata 비용을 trace와 metric으로 설명하는 것이다.
- [[SSD Mini Lab 프로젝트 허브]]와 병렬로 두되, 이 프로젝트는 내부 white-box model track으로 유지한다.

## 원본 위치와 핵심 문서

- 로컬 프로젝트: `C:\Users\nei11\venv\venv\GC`
- README: `C:\Users\nei11\venv\venv\GC\readme.md`
- Handoff: `C:\Users\nei11\venv\venv\GC\docs\next_chat_handoff.md`
- Request + Timing MVP: `C:\Users\nei11\venv\venv\GC\docs\request_timing_mvp.md`
- Request + Timing Obsidian 요약: [[Request Timing MVP]]
- Portfolio evidence: `C:\Users\nei11\venv\venv\GC\docs\portfolio_gc_evidence.md`
- Temperature-aware findings: `C:\Users\nei11\venv\venv\GC\docs\temp_aware_core_findings.md`
- Temperature-aware Obsidian 요약: [[Temperature-Aware GC Core Findings]]
- Lazy Gecko-lite: `C:\Users\nei11\venv\venv\GC\docs\lazy_gecko_lite.md`

## 핵심 검증 질문

- GC victim selection policy가 WAF, wear, TRIM reclaim, metadata IO 사이에서 어떤 trade-off를 만드는가?
- 같은 workload와 seed에서 policy별 차이가 반복 가능한가?
- `greedy`, `bsgc`, `re50315`, `cota`, `temp_aware_2stage`, `pvb_window`는 어떤 조건에서 강하고 약한가?
- data temperature, PVB stale metadata, lazy correction 비용은 victim 선택 품질에 어떤 영향을 주는가?
- host request timing을 붙였을 때 WAF뿐 아니라 p95/p99 latency proxy와 GC pause trade-off도 설명할 수 있는가?

## 내부 모델 범위

- 포함:
  - page-mapped out-of-place write
  - `VALID` / `INVALID` / `FREE` page state
  - `LPN -> PPN` mapping과 reverse mapping
  - overwrite, TRIM, GC, valid-page migration, block erase
  - wear leveling proxy
  - data temperature metadata
  - Lazy Gecko-lite / PVB metadata model
  - request timing MVP
- 제외:
  - 실제 SSD firmware 구현 주장
  - NVMe/PCIe/fio 실측 결과와 직접 합치기
  - 실제 SSD latency 예측
  - channel/die/plane 병렬성
  - read disturb, retention, ECC, page type 같은 물리 세부사항

## 주요 검증 축

### GC policy trade-off

- `greedy`: WAF 기준선으로 강하지만 wear balance와 TRIM reclaim은 약할 수 있음
- `bsgc`: wear와 TRIM reclaim 쪽의 안정적인 balanced baseline
- `re50315`: wear balance가 강하지만 WAF 비용이 있음
- `cota`: handcrafted middle-ground control
- `temp_aware`: update-heavy에서는 좋은 신호가 있으나 low OP pressure에서 비용이 커질 수 있음
- `temp_aware_2stage`: invalid ratio gate 뒤 temperature로 tie-break하는 보수적 후보

### TRIM / reclaim 검증

- TRIM은 host가 더 이상 쓰지 않는 LPN을 FTL에 알려주는 입력이다.
- 중요한 것은 TRIM 이벤트 자체가 아니라 invalidation 이후 GC reclaim까지의 lag와 locality다.
- 관련 evidence:
  - `docs/trim_gc_validation_evidence.md`
  - `docs/trim_evidence_matrix.md`

### Wear-leveling

- wear는 실제 NAND health telemetry가 아니라 erase count 기반 proxy다.
- static wear-leveling은 low-wear block의 valid data를 이동하고 erase count spread를 줄이는 비용을 기록한다.
- 핵심은 wear 개선과 WAF/moved-valid 비용을 함께 보는 것이다.

### Data temperature

- workload oracle label이 아니라 LPN write 재방문 간격 기반 `observed_temp_ewma`를 사용한다.
- [[Temperature-Aware GC Core Findings]]에서 `temp_aware`는 update-heavy workload에서 WAF 개선 신호가 있었지만, low OP pressure에서는 비싼 victim을 고르는 문제가 있었다.
- 이 결과는 좋은 성공담보다 “어떤 조건에서 feature가 깨지는가”를 설명하는 검증 포인트로 가치가 크다.

### Lazy Gecko-lite / PVB metadata

- PVB는 flash-resident FTL에서 invalidation metadata가 늦게 반영되는 상황을 단순화한 모델이다.
- `pvb_window`는 GC 직전 top-K 후보만 correction해서 metadata read 비용과 victim 품질을 함께 본다.
- 결론은 PVB가 항상 우월하다는 것이 아니라, correction 비용과 victim 품질 trade-off를 관찰 가능하게 만들었다는 점이다.

### Request + Timing MVP

- [[Request Timing MVP]]에서 기존 page-level write/TRIM simulator를 `host request -> FTL state transition -> simulated completion time` 흐름으로 확장했다.
- Python 실행 시간과 simulated NAND latency를 분리한다.
- p50/p95/p99 latency proxy, queue wait, GC pause, throughput proxy를 report에 남긴다.
- 이 단계의 목적은 실제 SSD latency 예측이 아니라 같은 단순 timing 가정 아래에서 policy/workload trade-off를 비교하는 것이다.

## 봐야 할 지표

- WAF
- GC count
- moved valid pages
- wear average / wear std / wear max
- TRIM reclaim rate
- pending reclaim
- metadata read/write IO
- PVB stale invalidation
- p50/p95/p99 latency proxy
- GC pause request rate
- queue wait
- throughput proxy
- invariant / QC 결과

## 내 프로젝트와 연결

- 직접 연결되는 노트:
  - [[SSD 허브]]
  - [[SSD Mini Lab 프로젝트 허브]]
  - [[SSD Garbage Collection]]
  - [[SSD Wear Leveling]]
  - [[SSD Trim]]
  - [[Request Timing MVP]]
  - [[Temperature-Aware GC Core Findings]]
- 앞으로 만들면 좋은 노트:
  - [[FTL]]
  - [[Write Amplification Factor]]
  - [[GC Pause]]
  - [[PVB Metadata Model]]
  - [[Data Temperature]]
  - [[Request Timing MVP]]

## 포트폴리오 문장

> SSD controller 내부의 FTL/GC/TRIM lifecycle을 Python simulator로 모델링하고, 여러 GC policy를 같은 workload와 seed에서 비교했습니다. WAF 하나만 보지 않고 wear, TRIM reclaim, metadata IO, p95/p99 latency proxy, GC pause를 함께 분석하면서, 정책이 어떤 조건에서 강하고 어디서 깨지는지 검증 관점으로 정리했습니다.

## 면접에서 강조할 점

- 실제 SSD firmware라고 과장하지 않는다.
- simplified model의 scope와 non-goal을 먼저 말한다.
- WAF, wear, metadata, latency proxy를 함께 보는 trade-off 관점을 강조한다.
- 실패하거나 default 승격하지 않은 정책도 boundary evidence로 설명한다.
- [[SSD Mini Lab 프로젝트 허브]]와 연결해 “외부 black-box 측정”과 “내부 white-box 모델링”을 구분할 수 있다고 말한다.

## 아직 보강할 것

- [[FTL]] 개념 노트 생성
- [[Write Amplification Factor]] 개념 노트 생성
- [[PVB Metadata Model]] 검증 포인트 노트 생성
- [[Request Timing MVP]] review 노트 생성
- [[Temperature-Aware GC Core Findings]] review 노트 생성
- `docs/portfolio_gc_evidence.md`를 별도 포트폴리오 evidence 노트로 요약

## 업데이트 로그

- 2026-07-15:
  - `C:\Users\nei11\venv\venv\GC` 스윕 결과를 바탕으로 프로젝트 허브 생성.
  - 내부 white-box FTL/GC validation track으로 정리.
  - [[SSD 허브]], [[SSD Mini Lab 프로젝트 허브]]와 연결.
  - [[Request Timing MVP]], [[Temperature-Aware GC Core Findings]]를 evidence branch로 추가.


