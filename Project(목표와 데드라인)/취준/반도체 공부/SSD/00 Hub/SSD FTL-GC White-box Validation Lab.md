---
title: SSD FTL-GC White-box Validation Lab
aliases:
  - GC Lab
  - FTL GC White-box Lab
  - SSD GC Simulator Project
  - White-box SSD Validation Lab
tags:
  - hub
  - project
  - ssd
  - ftl
  - garbage-collection
  - ssd-validation
  - recovery
  - portfolio
type: project
status: growing
domain: SSD FTL Validation
created: 2026-07-15
updated: 2026-07-28
source: C:\Users\nei11\venv\venv\GC
source_type: local-project
reliability: deterministic-simulator
related_projects:
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# SSD FTL-GC White-box Validation Lab

## 한 줄 요약

- SSD controller 내부의 FTL, GC, TRIM, metadata durability, request durability, failure recovery, controller ownership을 Python simulator로 단순화해 관찰하는 white-box validation 프로젝트.

## 이 프로젝트의 목적

- 실제 SSD firmware라고 주장하는 것이 아니라, 내부 상태 전이와 실패 경계를 관찰 가능한 모델로 만들어 검증 관점을 훈련한다.
- 핵심은 새 GC policy 하나가 아니라, 같은 workload와 seed에서 policy, metadata model, recovery protocol이 어떤 비용과 한계를 만드는지 trace와 invariant로 설명하는 것이다.
- [[SSD Mini Lab 프로젝트 허브]]와 병렬로 두되, 이 프로젝트는 내부 white-box model track으로 유지한다.

## 현재 정체성

| 구분 | 내용 |
|---|---|
| 프로젝트 유형 | bounded deterministic simulator |
| 핵심 대상 | page-mapped FTL, GC, TRIM, metadata durability, request protocol, recovery boundary |
| 주요 evidence | deterministic probe, manifest, summary CSV, trace, strict invariant, pytest |
| 현재 suite 상태 | handoff 기준 warning-strict full suite 387 passed |
| 하지 않는 주장 | 실제 SSD firmware, NVMe 성능 예측, physical NAND reliability proof |

## 원본 위치와 핵심 문서

- 로컬 프로젝트: `C:\Users\nei11\venv\venv\GC`
- README: `C:\Users\nei11\venv\venv\GC\readme.md`
- Handoff: `C:\Users\nei11\venv\venv\GC\docs\next_chat_handoff.md`
- Resource Contention: `docs\resource_contention_quality_experiment.md`
- PVB decision boundary: `docs\case_study_pvb_decision_boundary.md`
- Translation payload timing: `docs\translation_payload_timing_findings.md`
- Persistent PBBT: `docs\persistent_bad_block_recovery_mvp.md`
- Valid-data failure recovery: `docs\valid_data_failure_recovery_mvp.md`
- Page mirroring: `docs\page_mirroring_mvp.md`
- Request replay: `docs\request_replay_mvp.md`
- Request checkpoint eviction: `docs\request_checkpoint_eviction_mvp.md`
- Controller lease/epoch: `docs\controller_lease_epoch_mvp.md`
- External fencing: `docs\external_fencing_anti_rollback_mvp.md`

## 핵심 검증 질문

- GC victim selection policy가 WAF, wear, TRIM reclaim, metadata IO 사이에서 어떤 trade-off를 만드는가?
- stale metadata와 correction 비용이 victim 품질에 어떤 영향을 주는가?
- PAL-lite resource model에서 GC migration/erase와 host I/O의 resource 충돌이 tail latency proxy에 어떻게 나타나는가?
- bad block, torn metadata, sudden VALID-data failure 뒤 어떤 state는 rollback되고 어떤 state는 roll-forward되는가?
- spare promotion과 data recovery를 구분해서 설명할 수 있는가?
- host retry, lost completion, replay ledger eviction이 중복 write 없이 처리되는가?
- multi-controller ownership에서 stale owner나 old controller image가 mutation authority를 되찾지 못하게 막을 수 있는가?

## 주요 branch

| Branch | 노트 | 역할 |
|---|---|---|
| GC policy / timing | [[Request Timing MVP]], [[Request Timing Policy Findings]] | WAF, wear, p99/p99.9 latency proxy, GC pause trade-off |
| Resource contention | [[Resource Contention MVP]], [[Resource Contention Quality Experiment]] | serial vs PAL-lite, resource wait, GC-resource wait |
| Temperature / PVB | [[Temperature-Aware GC Core Findings]], [[PVB Metadata Model]] | data temperature, stale PVB metadata, correction cost |
| Recovery / bad block | [[FTL Metadata Recovery and Bad Block Handling]] | PBBT, valid-data failure, page mirroring, spare/data recovery 분리 |
| Request durability | [[Durable Request Replay]] | stable request ID, replay ledger, checkpoint floor, selective ACK |
| Controller ownership | [[Controller Lease and External Fencing]] | lease/epoch, bounded expiry, external anti-rollback authority |

## 내부 모델 범위

- 포함:
  - page-mapped out-of-place write
  - `LPN -> PPN` mapping과 reverse mapping
  - overwrite, TRIM, GC, valid-page migration, block erase
  - wear leveling proxy
  - Lazy Gecko-lite / PVB metadata model
  - request timing, PAL-lite resource contention
  - deterministic program/erase fault, block retirement, hidden spare promotion
  - persistent bad-block table, two-copy metadata, relocation marker
  - page mirroring, primary+mirror acknowledgement atomicity
  - request replay, checkpoint eviction, selective ACK, session rollover
  - multi-controller lease/epoch, bounded expiry, external fencing authority
- 제외:
  - 실제 SSD firmware 구현 주장
  - 실제 NVMe/PCIe command path, OS/filesystem, vendor firmware scheduler
  - 실제 NAND cell degradation, ECC, retention, read disturb 물리 정확도
  - hardware monotonic counter, network quorum/consensus, storage-fabric reservation semantics
  - 실제 SSD latency 예측

## 봐야 할 지표 / evidence

- WAF
- GC count
- moved valid pages
- wear std / wear max
- metadata read/write IO
- p95/p99/p99.9 latency proxy
- resource wait / GC resource wait
- PBBT generation, retired/promoted counts
- recovered/unrecoverable LPN count
- mirror writes / mirror WAF
- request ledger state, replay floor, selective tombstone
- owner ID, lease epoch, lease generation, pending transition
- strict invariant / QC findings
- deterministic probe PASS count

## 공부 순서

1. [[SSD Garbage Collection]]
2. [[Write Amplification Factor]]
3. [[Request Timing MVP]]
4. [[Request Timing Policy Findings]]
5. [[Resource Contention MVP]]
6. [[Resource Contention Quality Experiment]]
7. [[Temperature-Aware GC Core Findings]]
8. [[PVB Metadata Model]]
9. [[FTL Metadata Recovery and Bad Block Handling]]
10. [[Durable Request Replay]]
11. [[Controller Lease and External Fencing]]

## 최신 완료 축 요약

### Recovery / Bad Block

- Persistent PBBT recovery는 erase failure retirement, PBBT generation, promoted spare, restart recovery를 6/6 deterministic probe로 검증했다.
- Sudden VALID-data failure는 spare promotion이 capacity만 복구하고, user data는 surviving relocation evidence나 mirror가 있을 때만 복구된다는 점을 분리했다.
- Page mirroring은 full mirror 4/4 recovery at WAF 2.0, selective mirror 2/4 recovery at WAF 1.25를 보여줬다.

### Request Durability

- Durable request replay는 completion lost/reset 뒤 same request retry가 duplicate storage effect를 만들지 않는지 확인했다.
- checkpoint eviction은 floor commit을 entry deletion보다 먼저 수행해 delayed retry가 allocator에 들어가지 못하게 했다.
- selective ACK와 session fence는 out-of-order completion과 session rollover에서 stale mutation을 막는다.

### Controller Ownership

- Lease/epoch authority는 두 controller front end가 같은 SSD state를 공유할 때 stale owner mutation을 막는다.
- bounded lease expiry는 suspicion과 authority를 분리한다. suspicion은 진단일 뿐 takeover 권한이 아니다.
- external fencing은 controller image 밖의 high-water epoch로 old whole-controller image rollback을 fail closed한다.

## 포트폴리오 문장

> SSD FTL/GC simulator에서 GC policy와 tail latency trade-off를 비교하는 단계에서 더 나아가, bad-block retirement, metadata recovery, page mirroring, request replay, controller lease/fencing 같은 failure boundary를 deterministic probe로 검증했습니다. 실제 SSD firmware라고 주장하지 않고, PREPARE/COMMIT ordering, rollback/roll-forward, fail-closed invariant를 명시해 내부 상태 전이가 왜 안전한지 설명하는 white-box validation lab으로 정리했습니다.

## 면접에서 강조할 점

- 실제 firmware가 아니라 bounded deterministic simulator임을 먼저 말한다.
- WAF 같은 efficiency metric과 recovery/fencing 같은 correctness boundary를 분리한다.
- spare promotion은 data recovery가 아니라 capacity recovery라고 설명한다.
- request replay는 exactly-once completion이 아니라 effectively-once storage effect라고 설명한다.
- external fencing은 controller image 밖의 monotonic authority가 있어야 anti-rollback claim이 가능하다고 말한다.
- [[SSD Mini Lab 프로젝트 허브]]와 연결해 black-box 측정과 white-box 내부 모델을 섞지 않는다.

## 아직 보강할 것

- `docs/portfolio_gc_evidence.md`를 별도 포트폴리오 evidence 노트로 요약
- [[FTL]] 개념 노트 생성
- `project_internalization_plan` 성격의 “내 언어로 설명하기” 문답 노트 생성
- quorum/consensus, physical clock authority, real hardware monotonic counter는 현재 범위 밖으로 유지

## 업데이트 로그

- 2026-07-28:
  - 최신 `C:\Users\nei11\venv\venv\GC` 상태를 반영해 recovery, request durability, controller fencing branch를 추가.
  - [[FTL Metadata Recovery and Bad Block Handling]], [[Durable Request Replay]], [[Controller Lease and External Fencing]] 연결.
  - 프로젝트 정체성을 GC policy 비교에서 white-box FTL/GC + failure/recovery/protocol validation lab으로 확장.
- 2026-07-20:
  - [[Request Timing Policy Findings]], [[Resource Contention MVP]], [[Resource Contention Quality Experiment]]를 white-box branch에 추가.
  - Request timing 이후 policy verdict와 resource contention 검증 흐름을 허브에 반영.
- 2026-07-15:
  - `C:\Users\nei11\venv\venv\GC` 스윕 결과를 바탕으로 프로젝트 허브 생성.
  - 내부 white-box FTL/GC validation track으로 정리.
