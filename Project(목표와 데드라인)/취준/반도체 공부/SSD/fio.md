---
title: fio
aliases:
  - Flexible I/O Tester
  - fio benchmark
tags:
  - concept
  - ssd
  - ssd-validation
  - fio
type: concept
status: growing
domain: SSD Validation
created: 2026-07-15
updated: 2026-07-15
source: D:\ssd_lab\README.md
source_type: local-project
reliability: personal-experiment
related_projects:
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - product-engineering
  - test-engineering
---

# fio

## 한 줄 정의

- `fio`는 workload 조건을 명시해 storage I/O를 발생시키고 bandwidth, IOPS, latency percentile 같은 결과를 수집하는 I/O 테스트 도구다.

## 이 개념의 층위

- 위치: test framework / OS / filesystem / storage path / device
- 누구의 동작인가: host / test framework
- 입력: workload, block size, queue depth, direct mode, runtime, test path
- 출력: JSON 결과, bandwidth, IOPS, latency, percentile, error 정보

## 왜 중요한가

- SSD validation에서는 측정 조건을 명확히 남기는 것이 결과 숫자만큼 중요하다.
- `fio`는 workload를 재현 가능한 형태로 정의하고 raw JSON 결과를 남길 수 있다.
- 이후 CSV 파싱, 그래프, QoS 리뷰, 포트폴리오 evidence로 이어지는 검증 흐름의 출발점이 된다.

## 핵심 동작 / 원리

1. job file 또는 command option으로 workload 조건을 정의한다.
2. 지정한 test path에 read/write/random/sequential I/O를 발생시킨다.
3. 결과를 JSON 등 구조화된 형태로 저장한다.
4. parser가 JSON에서 bandwidth, IOPS, p95/p99/p99.9 latency 등을 뽑아 CSV로 만든다.
5. 분석 스크립트가 조건별 비교와 그래프를 만든다.

## 대표 시나리오

- baseline 성능 측정: `seq_read`, `seq_write`, `rand_read`, `rand_write`
- QD sweep: QD 1/4/16/32에서 random read/write 비교
- direct vs buffered: `direct=1`과 `direct=0` 비교
- sustained workload: 120s/300s처럼 runtime을 늘려 tail behavior 확인

## 무엇과 헷갈리기 쉬운가

- benchmark score와의 차이:
  - fio 결과는 조건이 붙은 실험 결과이지, SSD의 절대 성능 순위표가 아니다.
- device 성능과의 차이:
  - fio는 host path에서 관찰한 결과를 보여준다. OS, filesystem, cache, test path, USB bridge 같은 요소가 섞일 수 있다.
- 자주 하는 오해:
  - `direct=1`이면 완전히 순수한 device 성능이라고 단정하면 안 된다.
  - 평균 IOPS만 보고 좋은 조건이라고 판단하면 안 된다.

## 관련 개념

- 상위 개념:
  - [[SSD Mini Lab 프로젝트 허브]]
- 인접 개념:
  - [[Queue Depth]]
  - [[p99 latency]]
  - [[SSD QoS]]
- 결과적으로 연결되는 것:
  - [[SSD Mini Lab Portfolio Evidence]]
  - [[왜 평균 IOPS만 보면 안 되는가]]

## 검증 / 실무 관점

- 실제 제품 검증에서 확인할 포인트:
  - job option과 결과 파일명이 일치하는가
  - workload, block size, QD, direct mode, runtime, path가 보존되는가
  - raw JSON과 CSV summary가 audit 가능하게 남아 있는가
- 어떤 로그/메트릭/명령으로 볼 수 있는가:
  - fio JSON
  - parser output CSV
  - p99/p99.9 latency plot
  - run-to-run CV
- 실무에서 왜 문제가 되는가:
  - 측정 조건을 보존하지 않으면 좋은 숫자도 재현하거나 설명하기 어렵다.

## 내 프로젝트와 연결

- 연결되는 프로젝트:
  - [[SSD Mini Lab 프로젝트 허브]]
- 연결되는 실험/결과:
  - [[SSD Mini Lab Portfolio Evidence]]
  - [[SSD Mini Lab Stage 1 회고]]
- 자소서/면접에서 설명할 때 쓸 수 있는 문장:
  - fio를 단순 benchmark 도구로 쓰는 데서 멈추지 않고, JSON 결과를 CSV/plot/report로 이어지는 검증 workflow로 만들었습니다.

## 한계 / 예외 / 조건

- fio file-target 결과는 OS/filesystem/cache/path 영향을 받을 수 있다.
- 외장 SSD에서는 USB bridge, enclosure, port, cable 영향도 함께 고려해야 한다.
- 결과 해석은 항상 test condition과 함께 해야 한다.

## 출처 / 참고

- [[SSD Mini Lab 프로젝트 허브]]
- [[SSD Mini Lab Portfolio Evidence]]
- `D:\ssd_lab\README.md`
