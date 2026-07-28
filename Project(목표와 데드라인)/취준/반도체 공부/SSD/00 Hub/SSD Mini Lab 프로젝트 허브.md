---
title: SSD Mini Lab 프로젝트 허브
aliases:
  - SSD Mini Lab
  - fio 기반 SSD 검증 미니랩
  - External SSD Black-box Validation Lab
tags:
  - hub
  - project
  - ssd
  - ssd-validation
  - fio
  - portfolio
type: project
status: growing
domain: SSD Validation
created: 2026-07-15
updated: 2026-07-28
source: D:\ssd_lab
source_type: local-project
reliability: personal-experiment
related_projects:
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - product-engineering
  - test-engineering
---

# SSD Mini Lab 프로젝트 허브

## 한 줄 요약

- `fio` 기반으로 실제 외장 SSD를 black-box DUT처럼 다루며, 성능 숫자뿐 아니라 반복성, tail latency, data integrity, observer limitation, requirement verdict를 함께 정리하는 검증 프로젝트.

## 이 프로젝트의 목적

- 실제 SSD를 내부 firmware 관점이 아니라 외부 제품 검증 관점에서 본다.
- 목표는 최고 benchmark 숫자를 찾는 것이 아니라, 반복 가능한 검증 흐름과 해석 가능한 evidence를 만드는 것이다.
- 핵심 흐름은 `requirement -> condition -> runner / observer -> raw evidence -> analysis -> verdict`다.
- [[SSD FTL-GC White-box Validation Lab]]과 병렬로 두되, 이 프로젝트는 실제 장치의 black-box 측정 track으로 유지한다.

## 원본 위치와 핵심 문서

- 로컬 프로젝트: `D:\ssd_lab`
- README: `D:\ssd_lab\README.md`
- 요구사항 matrix: `D:\ssd_lab\docs\reports\external_ssd_requirement_matrix.md`
- 현재 결과 해석: `D:\ssd_lab\docs\reports\external_ssd_product_validation.md`
- roadmap: `D:\ssd_lab\docs\reports\external_ssd_project_roadmap.md`
- regression profile: `D:\ssd_lab\docs\reports\external_ssd_regression_profile.md`
- 내부화 계획: `D:\ssd_lab\docs\reports\project_internalization_plan.md`

## 핵심 branch

| Branch | 노트 | 역할 |
|---|---|---|
| 전체 제품 검증 | [[External SSD Product Validation]] | external SSD DUT 검증의 부모 노트 |
| QD / tail latency | [[왜 평균 IOPS만 보면 안 되는가]] | QD32 write처럼 평균과 tail이 갈라지는 사례 |
| block size | [[External SSD Block Size Sweep]] | 4K/64K/1M QD32 throughput knee와 latency cost |
| data integrity | [[External SSD Data Integrity]] | 4 GiB CRC32C write/readback correctness MVP |
| negative result | [[재현되지 않은 가설도 검증 결과다]] | evidence Pass와 hypothesis not reproduced 분리 |
| telemetry / observer | [[NVMe SMART Telemetry]] | telemetry 가능/불가능, Limited evidence 해석 |
| 포트폴리오 근거 | [[SSD Mini Lab Portfolio Evidence]] | 프로젝트 evidence map |

## 핵심 검증 질문

- 같은 SSD라도 workload, QD, block size, direct mode, path에 따라 결과가 어떻게 달라지는가?
- 평균 IOPS가 좋아 보여도 p99/p99.9 latency와 CV까지 보면 같은 결론을 유지할 수 있는가?
- 성능 가설이 재현되지 않았을 때, 그것을 실패가 아니라 검증 evidence로 설명할 수 있는가?
- file-target CRC32C integrity Pass와 host observer Limited를 분리해서 말할 수 있는가?
- Windows, USB, exFAT, fio file-target 경로의 영향을 device-level claim과 어떻게 분리할 수 있는가?

## 프로젝트 흐름

1. Baseline workload
   - `seq_read`, `seq_write`, `rand_read`, `rand_write`
2. Queue-depth sweep
   - 4K random read/write, QD 1/4/16/32
3. Repeatability / QoS review
   - repeat=3, CV, p99/p99.9, max latency
4. Direct vs buffered / WSL path
   - OS, filesystem, cache, path effect boundary 확인
5. Sustained workload
   - QD16/QD32 120s/300s read/write, time-window evidence
6. State reproducibility / mixed workload
   - reconnect-start, ABBA/BAAB, mixed ratio, idle ramp
7. Block-size mapping
   - 4K/64K/1M QD32 random read/write
8. Data integrity
   - 4 GiB CRC32C write/readback Pass, observer Limited
9. Regression profile
   - broad sweep 대신 compact requirement-based regression으로 축소

## 봐야 할 지표

- 평균 처리량:
  - bandwidth
  - IOPS
- tail latency:
  - p99 latency
  - p99.9 latency
  - max latency
- 반복 안정성:
  - 표준편차
  - CV
  - run-to-run variation
- evidence 품질:
  - runner manifest
  - observer manifest
  - raw fio JSON
  - parsed CSV
  - integrated run manifest
  - Pass / Observation / Limited / Blocked
- 조건 metadata:
  - workload
  - block size
  - queue depth
  - direct mode
  - runtime
  - test path
  - fio version
  - OS/filesystem/cache 영향 가능성

## 핵심 결과와 해석 포인트

### QD와 tail latency

- QD를 높이면 IOPS가 좋아지는 경우가 있다.
- 하지만 rand_write QD32처럼 처리량 이득은 제한적인데 p99/p99.9 latency와 CV가 크게 나빠지는 조건도 있었다.
- 따라서 최고 IOPS 조건이 항상 좋은 검증 조건은 아니다.

### Block size

- [[External SSD Block Size Sweep]]에서 QD32 random I/O의 관찰된 throughput knee는 64K였다.
- 1M은 bandwidth 이득 없이 p99 latency만 read 13.77x, write 15.69x 증가시켰다.
- block size 비교에서는 IOPS를 단일 점수처럼 보면 안 된다.

### Data integrity

- [[External SSD Data Integrity]]에서 4 GiB CRC32C write/readback은 Pass였다.
- 하지만 synchronized Windows host counter observer는 nonzero activity sample을 잡지 못해 Limited로 남았다.
- correctness verdict와 observability coverage를 분리하는 것이 핵심이다.

### 재현되지 않은 가설

- mixed workload, ratio sweep, idle ramp에서 성능 방향은 반복 재현되지 않았다.
- 이 결과는 실패가 아니라 “그 가설을 지금 evidence로는 주장하면 안 된다”는 검증 결과다.
- 자세한 회고는 [[재현되지 않은 가설도 검증 결과다]]에 정리했다.

## 포트폴리오 문장

> fio 기반 SSD mini-lab에서 외장 SSD를 black-box DUT로 정의하고, 요구사항부터 raw evidence, 분석 CSV, verdict까지 추적 가능한 검증 흐름을 만들었습니다. 평균 IOPS뿐 아니라 p99/p99.9 latency, 반복 CV, block-size throughput knee, CRC32C data integrity, observer limitation을 함께 보고했고, 재현되지 않은 성능 가설은 `not reproduced`로 낮춰 해석했습니다.

## 면접에서 강조할 점

- 단순히 fio를 돌린 것이 아니라 조건, 실행, evidence, 분석, 판정을 연결했다.
- 성능 숫자와 evidence completeness를 분리했다.
- Pass와 Limited를 섞지 않았다.
- 재현되지 않은 가설도 버리지 않고 해석 경계를 남겼다.
- black-box DUT 검증과 white-box FTL/GC 모델링을 구분해서 설명할 수 있다.

## 아직 보강할 것

- `project_internalization_plan.md`를 바탕으로 “내 언어로 설명하기” 문답 노트 생성
- regression profile 실행 결과가 생기면 [[External SSD Product Validation]] 갱신
- host observer가 nonzero signal을 잡는 방법을 찾으면 [[External SSD Data Integrity]]와 [[NVMe SMART Telemetry]] 갱신
- 최종 포트폴리오는 모든 실험이 아니라 3-4개 case study로 압축

## 업데이트 로그

- 2026-07-28:
  - [[External SSD Block Size Sweep]], [[External SSD Data Integrity]], [[재현되지 않은 가설도 검증 결과다]] branch 추가.
  - 깨진 한글 허브를 복원하고 최신 `D:\ssd_lab` 흐름 반영.
- 2026-07-20:
  - 외장 SSD 실측 결과 흐름을 반영.
  - [[External SSD Product Validation]]을 핵심 evidence로 승격.
- 2026-07-15:
  - `D:\ssd_lab` 스윕 결과를 바탕으로 프로젝트 허브 생성.
  - 외부 black-box SSD 검증 track으로 정리.
