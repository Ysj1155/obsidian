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
updated: 2026-07-20
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

- SSD controller 내부의 FTL, GC, TRIM, wear, metadata, request timing, resource contention 흐름을 Python simulator로 단순화하고, 같은 workload와 seed에서 policy trade-off를 검증하는 white-box validation 프로젝트.

## 이 프로젝트의 목적

- 실제 SSD firmware라고 주장하는 것이 아니라, 내부 동작을 관찰 가능한 모델로 만들어 검증 관점을 훈련한다.
- 핵심은 새 GC 알고리즘 하나를 주장하는 것이 아니라, FTL 상태 변화와 GC/TRIM/wear/metadata/timing 비용을 trace와 metric으로 설명하는 것이다.
- [[SSD Mini Lab 프로젝트 허브]]와 병렬로 두되, 이 프로젝트는 내부 white-box model track으로 유지한다.

## 원본 위치와 핵심 문서

- 로컬 프로젝트: `C:\Users\nei11\venv\venv\GC`
- README: `C:\Users\nei11\venv\venv\GC\README.md`
- Handoff: `C:\Users\nei11\venv\venv\GC\docs\next_chat_handoff.md`
- Request + Timing MVP: `C:\Users\nei11\venv\venv\GC\docs\request_timing_mvp.md`
- Request timing policy findings: `C:\Users\nei11\venv\venv\GC\docs\request_timing_policy_findings.md`
- Resource Contention MVP: `C:\Users\nei11\venv\venv\GC\docs\resource_contention_mvp.md`
- Resource Contention Quality Experiment: `C:\Users\nei11\venv\venv\GC\docs\resource_contention_quality_experiment.md`
- Temperature-aware findings: `C:\Users\nei11\venv\venv\GC\docs\temp_aware_core_findings.md`
- Lazy Gecko-lite: `C:\Users\nei11\venv\venv\GC\docs\lazy_gecko_lite.md`

## 핵심 검증 질문

- GC victim selection policy가 WAF, wear, TRIM reclaim, metadata IO 사이에서 어떤 trade-off를 만드는가?
- 같은 workload와 seed에서 policy별 차이가 반복 가능한가?
- `greedy`, `bsgc`, `cota`, `pvb_window`는 어떤 조건에서 강하고 약한가?
- data temperature, PVB stale metadata, lazy correction 비용은 victim 선택 품질에 어떤 영향을 주는가?
- host request timing을 붙였을 때 WAF뿐 아니라 p95/p99 latency proxy와 GC pause trade-off도 설명할 수 있는가?
- PAL-lite resource model을 붙였을 때 GC migration/erase와 host I/O의 resource 충돌이 tail latency proxy에 어떻게 나타나는가?

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
  - PAL-lite resource contention model
- 제외:
  - 실제 SSD firmware 구현 주장
  - NVMe/PCIe/fio 실측 결과와 직접 합치기
  - 실제 SSD latency 예측
  - 실제 NAND scheduler나 firmware priority
  - read disturb, retention, ECC, page type 같은 물리 세부사항

## 주요 검증 축

### GC policy trade-off

- `greedy`: WAF 기준선으로 강하지만 wear balance와 TRIM reclaim은 약할 수 있음
- `bsgc`: wear와 TRIM reclaim 쪽의 안정적인 balanced baseline
- `cota`: handcrafted middle-ground control. [[Request Timing Policy Findings]]에서는 wear 개선과 제한적인 tail cost 사이의 균형 후보로 정리됨
- `pvb_window`: metadata correction 아이디어를 검증했지만 일반 후보에서는 제외. cost와 seed sensitivity를 보여주는 negative evidence로 가치가 있음

### TRIM / reclaim 검증

- TRIM은 host가 더 이상 쓰지 않는 LPN을 FTL에 알려주는 입력이다.
- 중요한 것은 TRIM 이벤트 자체가 아니라 invalidation 이후 GC reclaim까지의 lag와 locality다.

### Wear-leveling

- wear는 실제 NAND health telemetry가 아니라 erase count 기반 proxy다.
- 핵심은 wear 개선과 WAF/moved-valid 비용을 함께 보는 것이다.

### Data temperature / PVB metadata

- [[Temperature-Aware GC Core Findings]]에서 `temp_aware`는 update-heavy workload에서 WAF 개선 신호가 있었지만, low OP pressure에서는 비싼 victim을 고르는 문제가 있었다.
- [[PVB Metadata Model]]은 correction 비용과 victim 품질 trade-off를 관찰 가능하게 만든다.

### Request + Timing

- [[Request Timing MVP]]에서 기존 page-level write/TRIM simulator를 `host request -> FTL state transition -> simulated completion time` 흐름으로 확장했다.
- [[Request Timing Policy Findings]]에서 24-run follow-up으로 `greedy`, `cota`, `bsgc`, `pvb_window` verdict를 정리했다.

### Resource Contention

- [[Resource Contention MVP]]에서 single FIFO timing 모델을 `serial`과 `pal_lite` timing mode로 확장했다.
- [[Resource Contention Quality Experiment]]는 4개 scenario x 3개 policy x 5개 seed x 2개 timing mode, 총 120-run 설계로 resource contention 해석의 안정성을 검증하려는 단계다.

## 봐야 할 지표

- WAF
- GC count
- moved valid pages
- wear average / wear std / wear max
- TRIM reclaim rate
- pending reclaim
- metadata read/write IO
- PVB stale invalidation
- p50/p95/p99/p99.9 latency proxy
- GC pause request rate
- queue wait
- resource wait
- GC resource wait
- throughput proxy
- invariant / QC 결과

## 내 프로젝트와 연결

- 직접 연결되는 노트:
  - [[SSD 허브]]
  - [[SSD Mini Lab 프로젝트 허브]]
  - [[SSD Garbage Collection]]
  - [[SSD Wear Leveling]]
  - [[SSD Trim]]
  - [[Write Amplification Factor]]
  - [[GC Pause]]
  - [[Data Temperature]]
  - [[PVB Metadata Model]]
  - [[Request Timing MVP]]
  - [[Request Timing Policy Findings]]
  - [[Resource Contention MVP]]
  - [[Resource Contention Quality Experiment]]
  - [[Temperature-Aware GC Core Findings]]

## 공부 순서

1. [[SSD Garbage Collection]]
2. [[Write Amplification Factor]]
3. [[GC Pause]]
4. [[Request Timing MVP]]
5. [[Request Timing Policy Findings]]
6. [[Resource Contention MVP]]
7. [[Resource Contention Quality Experiment]]
8. [[Temperature-Aware GC Core Findings]]
9. [[PVB Metadata Model]]

## 포트폴리오 문장

> SSD controller 내부의 FTL/GC/TRIM lifecycle을 Python simulator로 모델링하고, 여러 GC policy를 같은 workload와 seed에서 비교했습니다. WAF 하나만 보지 않고 wear, TRIM reclaim, metadata IO, p95/p99 latency proxy, GC pause, resource wait을 함께 분석하면서, 정책이 어떤 조건에서 강하고 어디서 깨지는지 검증 관점으로 정리했습니다.

## 면접에서 강조할 점

- 실제 SSD firmware라고 과장하지 않는다.
- simplified model의 scope와 non-goal을 먼저 말한다.
- WAF, wear, metadata, latency proxy, resource contention을 함께 보는 trade-off 관점을 강조한다.
- 실패하거나 default 승격하지 않은 정책도 boundary evidence로 설명한다.
- [[SSD Mini Lab 프로젝트 허브]]와 연결해 “외부 black-box 측정”과 “내부 white-box 모델링”을 구분할 수 있다고 말한다.

## 아직 보강할 것

- [[FTL]] 개념 노트 생성
- `docs/portfolio_gc_evidence.md`를 별도 포트폴리오 evidence 노트로 요약
- [[Resource Contention Quality Experiment]] 실행 결과가 나오면 planned에서 completed로 갱신

## 업데이트 로그

- 2026-07-20:
  - [[Request Timing Policy Findings]], [[Resource Contention MVP]], [[Resource Contention Quality Experiment]]를 white-box branch에 추가.
  - Request timing 이후 policy verdict와 resource contention 검증 흐름을 허브에 반영.
- 2026-07-15:
  - `C:\Users\nei11\venv\venv\GC` 스윕 결과를 바탕으로 프로젝트 허브 생성.
  - 내부 white-box FTL/GC validation track으로 정리.
  - [[SSD 허브]], [[SSD Mini Lab 프로젝트 허브]]와 연결.
