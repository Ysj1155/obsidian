---
title: p99 latency
aliases:
  - p99
  - tail latency
  - p99 지연시간
tags:
  - validation
  - ssd-validation
  - qos
  - latency
type: validation-point
status: growing
domain: SSD Validation
created: 2026-07-15
updated: 2026-07-15
source: D:\ssd_lab\docs\reports\qos_tail_latency_review.md
source_type: local-report
reliability: personal-experiment
related_projects:
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - product-engineering
  - test-engineering
---

# p99 latency

## 한 줄 결론

- p99 latency는 전체 요청 중 느린 상위 1%의 완료 시간을 보여주는 tail 지표로, 평균 IOPS가 숨기는 QoS 위험을 드러낸다.

## 검증 질문

- 무엇을 확인하려는가:
  - 평균 성능이 좋아 보이는 조건에서도 일부 요청이 지나치게 늦게 끝나는지 확인한다.
- 왜 이 검증이 필요한가:
  - 사용자 경험과 QoS는 평균보다 tail behavior에 더 민감할 수 있다.
- 좋은 결과/나쁜 결과를 어떻게 구분하는가:
  - 좋은 결과는 평균 IOPS가 높을 뿐 아니라 p99/p99.9와 CV가 안정적인 조건이다.
  - 나쁜 결과는 평균은 좋아도 p99가 크거나 반복 측정 CV가 큰 조건이다.

## 테스트 조건

- workload:
  - random read/write, sequential read/write, sustained workload
- block size:
  - 비교할 때 block size가 같아야 한다.
- queue depth:
  - QD가 높아질수록 p99가 커질 수 있다.
- direct mode:
  - `direct=0`은 cache/path 영향을 받을 수 있다.
- runtime:
  - 짧은 run보다 sustained run에서 tail behavior가 더 잘 드러날 수 있다.
- test path:
  - Windows path, WSL path, 외장 SSD 경로 차이를 분리해야 한다.
- 반복 횟수:
  - p99 CV를 함께 봐야 한다.
- 환경/제약:
  - OS, filesystem, cache, background I/O, device state 영향 가능성

## 봐야 할 지표

- 평균:
  - IOPS
  - bandwidth
  - 평균 latency
- tail latency:
  - p99
  - p99.9
  - max latency
- 변동성/CV:
  - p99 CV
  - p99.9 CV
- 보조 지표:
  - workload 조건
  - QD
  - direct mode
  - path

## 관찰한 결과

- QD sweep에서 `rand_write QD32`는 IOPS 이득이 제한적인데 p99 latency가 크게 증가했다.
- WSL path 비교에서는 read path의 p99/p99.9와 CV가 크게 흔들렸다.
- `direct=0 rand_read`는 p99 CV가 높아 cache/path-state sensitive한 결과로 해석할 수 있었다.

## 해석 기준

- p99가 크다는 것은 일부 요청이 평균보다 훨씬 늦게 끝났다는 뜻이다.
- p99 CV가 높으면 같은 조건을 반복해도 tail behavior가 안정적이지 않다는 뜻이다.
- 서로 다른 block size, path, direct mode를 한 줄 leaderboard처럼 비교하면 안 된다.
- 내부 GC, thermal throttling, SLC cache 같은 device-level 원인은 추가 telemetry 없이 단정하면 안 된다.

## 내 프로젝트와 연결

- 관련 프로젝트:
  - [[SSD Mini Lab 프로젝트 허브]]
- 관련 실험/결과물:
  - [[왜 평균 IOPS만 보면 안 되는가]]
  - [[SSD Mini Lab Portfolio Evidence]]
- 포트폴리오/면접에서 쓸 포인트:
  - 평균 IOPS가 아니라 p99/p99.9와 CV를 함께 봐야 안정적인 검증 결과를 설명할 수 있다.

## 면접/자소서로 바꿀 수 있는 문장

> SSD validation에서는 평균 IOPS만으로 결과를 판단하지 않고 p99와 p99.9 latency를 함께 봤습니다. 일부 조건은 처리량은 좋아 보여도 tail latency와 CV가 흔들렸고, 이를 통해 빠른 조건과 안정적으로 해석 가능한 조건을 구분할 수 있었습니다.

## 한계와 다음 확인

- 아직 단정하면 안 되는 부분:
  - p99 spike의 내부 원인을 GC나 thermal issue로 바로 단정하면 안 된다.
- 추가로 확인할 실험:
  - longer sustained workload
  - telemetry snapshot
  - 같은 workload 조건에서 path/direct mode 분리 반복 측정
- 더 필요한 telemetry / log / metric:
  - SMART/NVMe health
  - temperature
  - time-series latency

## 관련 노트

- [[SSD QoS]]
- [[Queue Depth]]
- [[fio]]
- [[왜 평균 IOPS만 보면 안 되는가]]
