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
updated: 2026-07-15
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

- `fio` 기반으로 실제 SSD를 black-box DUT처럼 다루며, workload 조건, 결과 파싱, p99/p99.9 latency, 반복 측정 안정성, 해석 한계를 함께 정리하는 외부 성능 검증 프로젝트.

## 이 프로젝트의 목적

- 실제 SSD를 내부 firmware 관점이 아니라 외부 제품 검증 관점에서 본다.
- 목표는 최고 벤치마크 숫자를 찾는 것이 아니라, 반복 가능한 검증 흐름을 만드는 것이다.
- 핵심 흐름은 `condition -> execution -> parsed result -> graph -> interpretation boundary`이다.
- [[SSD FTL-GC White-box Validation Lab]]과 병렬로 두되, 이 프로젝트는 실제 장치의 black-box 측정 track으로 유지한다.

## 원본 위치와 핵심 문서

- 로컬 프로젝트: `D:\ssd_lab`
- README: `D:\ssd_lab\README.md`
- 포트폴리오 근거: `D:\ssd_lab\docs\reports\portfolio_evidence.md`
- Obsidian 요약: [[SSD Mini Lab Portfolio Evidence]]
- Stage 1 회고: `D:\ssd_lab\docs\reports\stage1_review.md`
- QoS 리뷰: `D:\ssd_lab\docs\reports\qos_tail_latency_review.md`
- Sustained workload: `D:\ssd_lab\docs\reports\sustained_workload_week10.md`
- 외장 SSD 검증 계획: `D:\ssd_lab\docs\reports\external_ssd_product_validation.md`

## 핵심 검증 질문

- 같은 SSD라도 workload, QD, direct mode, path에 따라 결과가 어떻게 달라지는가?
- 평균 IOPS가 좋아 보여도 p99/p99.9 latency와 CV까지 보면 같은 결론이 유지되는가?
- `direct=1`과 `direct=0` 차이를 SSD media 성능으로 오해하지 않으려면 어떤 해석 경계가 필요한가?
- Windows path, WSL path, filesystem, cache, 외장 SSD 연결 경로의 영향을 어떻게 분리해서 설명할 수 있는가?
- sustained workload에서 짧은 run과 긴 run의 tail latency가 어떻게 달라지는가?

## 프로젝트 흐름

1. Baseline workload 실행
   - `seq_read`, `seq_write`, `rand_read`, `rand_write`
   - 결과: `results/fio_summary.csv`, `results/plots/`, `docs/reports/baseline_v1.md`
2. Queue-depth sweep
   - 4K random read/write, QD 1/4/16/32
   - 결과: `results/qd_sweep_grouped.csv`, `results/qd_sweep_plots/`
3. 반복 측정 안정성
   - run-to-run variation과 CV 확인
   - 결과: `results/qd_sweep_reproducibility.csv`
4. Direct vs buffered
   - `direct=1`과 `direct=0` 비교
   - 결과: `results/direct_buffered_comparison.csv`
5. WSL path 비교
   - WSL native ext4와 `/mnt/d` Windows-mounted path 비교
   - 결과: `results/wsl_path_compare_comparison.csv`
6. QoS / tail latency review
   - p99, p99.9, CV를 기준으로 위험 조건 선별
   - 결과: `results/qos_tail_latency_summary.csv`
7. Sustained workload
   - 120s/300s 장시간 smoke로 tail behavior 확인
   - 결과: `docs/reports/sustained_workload_week10.md`
8. External SSD product validation 준비
   - DUT profile, requirement matrix, execution runbook, product validation report template

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
- 조건 메타데이터:
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

- QD를 높이면 IOPS가 좋아질 수 있다.
- 하지만 `rand_write QD32`처럼 처리량 이득은 제한적인데 p99 latency가 크게 악화되는 조건이 있었다.
- 따라서 최고 IOPS 조건이 항상 좋은 검증 조건은 아니다.
- 관련 노트: [[왜 평균 IOPS만 보면 안 되는가]]

### Direct vs buffered

- `direct=0`은 `direct=1`보다 좋아 보이는 평균 성능을 낼 수 있다.
- 하지만 이 결과는 SSD media 자체가 빨라졌다는 뜻이 아니라 OS/filesystem cache 영향이 섞였을 가능성이 크다.
- 검증 보고서에서는 observed behavior와 device-level claim을 분리해야 한다.

### Sustained workload

- 짧은 run에서는 보이지 않던 tail latency 악화가 긴 runtime에서 드러날 수 있다.
- 300s write 조건은 120s write 조건보다 평균 IOPS와 p99/p99.9 latency가 더 나쁜 방향으로 변했다.
- 이것도 내부 원인을 단정하기보다 현재 OS/path/file-target 조건에서 관찰된 결과로 표현해야 한다.

## 내 프로젝트와 연결

- 직접 연결되는 노트:
  - [[SSD Mini Lab Stage 1 회고]]
  - [[왜 평균 IOPS만 보면 안 되는가]]
  - [[SSD 허브]]
- 내부 모델 track과 연결:
  - [[SSD FTL-GC White-box Validation Lab]]
- 앞으로 만들면 좋은 노트:
  - [[fio]]
  - [[Queue Depth]]
  - [[p99 latency]]
  - [[SSD QoS]]
  - [[External SSD Product Validation]]

## 포트폴리오 문장

> fio 기반 SSD mini-lab을 만들면서 workload 조건을 정의하고, fio JSON을 CSV로 파싱하고, p99/p99.9 latency와 반복 측정 CV를 함께 분석했습니다. 평균 IOPS만으로 성능을 판단하지 않고, direct/buffered, WSL path, sustained workload처럼 결과 해석을 흔들 수 있는 조건을 분리해 검증 보고서에 반영했습니다.

## 면접에서 강조할 점

- 단순히 fio를 돌린 것이 아니라 조건, 결과, 그래프, 해석 한계를 한 흐름으로 만들었다.
- 평균 IOPS보다 p99/p99.9 latency와 CV를 함께 본다.
- cache/path 영향이 섞인 결과를 SSD 자체 성능으로 과장하지 않는다.
- black-box DUT 검증과 white-box FTL/GC 모델링을 구분해서 설명할 수 있다.

## 아직 보강할 것

- [[External SSD Product Validation]] 노트 생성
- [[fio]] 개념 노트 생성
- [[Queue Depth]] 개념 노트 생성
- [[p99 latency]] 검증 포인트 노트 생성
- [[SSD Mini Lab Portfolio Evidence]] 생성 완료

## 업데이트 로그

- 2026-07-15:
  - `D:\ssd_lab` 스윕 결과를 바탕으로 프로젝트 허브 생성.
  - 외부 black-box SSD 검증 track으로 정리.
  - [[SSD 허브]], [[왜 평균 IOPS만 보면 안 되는가]], [[SSD FTL-GC White-box Validation Lab]]과 연결.
  - [[SSD Mini Lab Portfolio Evidence]]를 evidence branch로 추가.


