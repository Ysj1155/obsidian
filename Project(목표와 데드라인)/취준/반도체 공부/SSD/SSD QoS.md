---
title: SSD QoS
aliases:
  - SSD Quality of Service
  - SSD QoS 검증
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

# SSD QoS

## 한 줄 결론

- SSD QoS 검증은 평균 처리량이 아니라 tail latency와 반복 안정성을 함께 보며, 사용자가 체감할 수 있는 느린 요청과 해석 위험을 찾는 과정이다.

## 검증 질문

- 무엇을 확인하려는가:
  - SSD가 특정 workload에서 빠를 뿐 아니라 안정적으로 예측 가능한 성능을 내는가.
- 왜 이 검증이 필요한가:
  - 평균 IOPS는 일부 느린 요청, cache/path 영향, 반복 변동성을 숨길 수 있다.
- 좋은 결과/나쁜 결과를 어떻게 구분하는가:
  - 좋은 결과는 평균, p99/p99.9, CV가 모두 설명 가능한 조건이다.
  - 나쁜 결과는 평균은 좋아도 p99/p99.9가 크거나 반복 간 변동성이 큰 조건이다.

## 테스트 조건

- workload:
  - read/write, random/sequential, sustained workload
- block size:
  - 서로 다른 block size는 직접 rank하지 않는다.
- queue depth:
  - 높은 QD는 throughput과 tail latency trade-off를 만든다.
- direct mode:
  - cache influence를 줄이기 위해 `direct=1` 조건을 따로 본다.
- runtime:
  - sustained behavior 확인에는 긴 runtime이 필요하다.
- test path:
  - WSL/native/Windows-mounted/external path를 구분한다.
- 반복 횟수:
  - CV와 repeatability 확인
- 환경/제약:
  - OS/filesystem/cache/device state 영향 가능성

## 봐야 할 지표

- 평균:
  - IOPS
  - bandwidth
- tail latency:
  - p99
  - p99.9
  - max latency
- 변동성/CV:
  - p99 CV
  - p99.9 CV
- 에러/로그:
  - fio error
  - device/OS log
- 보조 지표:
  - path, direct mode, runtime, telemetry

## 관찰한 결과

- 평균 IOPS가 높은 조건이 항상 안정적인 조건은 아니었다.
- `rand_write QD32`는 QoS 관점에서 위험 조건으로 볼 수 있었다.
- buffered 또는 WSL path 관련 결과는 빠르게 보이거나 큰 p99.9를 보일 수 있어 해석 경계가 필요했다.

## 해석 기준

- QoS는 최고 성능보다 안정성과 tail behavior를 설명하는 관점이다.
- p99 magnitude와 p99 CV를 같이 봐야 한다.
- path/cache-sensitive test는 device media 성능으로 바로 해석하지 않는다.
- 서로 다른 workload shape을 하나의 leaderboard로 만들지 않는다.

## 내 프로젝트와 연결

- 관련 프로젝트:
  - [[SSD Mini Lab 프로젝트 허브]]
- 관련 실험/결과물:
  - [[SSD Mini Lab Portfolio Evidence]]
  - [[왜 평균 IOPS만 보면 안 되는가]]
  - [[p99 latency]]
- 포트폴리오/면접에서 쓸 포인트:
  - QoS review를 통해 평균 성능과 tail behavior를 분리해 검증했다.

## 면접/자소서로 바꿀 수 있는 문장

> SSD QoS 관점에서 평균 IOPS뿐 아니라 p99/p99.9 latency와 반복 측정 CV를 함께 분석했습니다. 이를 통해 빠르게 보이는 조건과 실제로 안정적이고 해석 가능한 조건을 구분하려고 했습니다.

## 한계와 다음 확인

- 아직 단정하면 안 되는 부분:
  - QoS 악화의 내부 원인
- 추가로 확인할 실험:
  - sustained workload 확대
  - telemetry와 latency time-series 결합
- 더 필요한 telemetry / log / metric:
  - temperature
  - SMART/NVMe health
  - throttling/error counter

## 관련 노트

- [[p99 latency]]
- [[Queue Depth]]
- [[fio]]
- [[SSD Mini Lab Portfolio Evidence]]
