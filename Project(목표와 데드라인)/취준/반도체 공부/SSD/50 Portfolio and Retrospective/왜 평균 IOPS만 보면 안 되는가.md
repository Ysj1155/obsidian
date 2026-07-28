---
title: 왜 평균 IOPS만 보면 안 되는가
aliases:
  - 평균 IOPS의 한계
  - SSD Validation에서 Tail Latency가 중요한 이유
tags:
  - validation
  - ssd
  - ssd-validation
  - fio
  - qos
  - portfolio
type: validation-point
status: growing
domain: SSD Validation
created: 2026-06-07
updated: 2026-06-07
source:
source_type: lab-review
reliability: personal-experiment
related_projects:
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - product-engineering
  - test-engineering
---

# 왜 평균 IOPS만 보면 안 되는가

## 한 줄 결론

- SSD validation에서 좋은 결과는 단순히 평균 IOPS가 높은 결과가 아니라, 조건이 명확하고 반복 가능하며 tail latency까지 설명 가능한 결과다.

## 검증 질문

- 무엇을 확인하려는가:
  - 평균 IOPS만으로 SSD 성능 검증 결과를 판단해도 되는가.
- 왜 이 검증이 필요한가:
  - 평균 IOPS는 처리량의 한 단면만 보여준다.
  - 실제 사용자 경험이나 QoS 관점에서는 일부 요청이 얼마나 늦게 끝나는지가 중요하다.
- 좋은 결과와 나쁜 결과를 어떻게 구분하는가:
  - 좋은 결과는 평균 IOPS, p99/p99.9 latency, CV가 함께 안정적인 조건이다.
  - 나쁜 결과는 평균 IOPS는 좋아 보여도 tail latency가 크게 악화되거나 반복 측정 변동성이 큰 조건이다.

## 테스트 조건

- workload:
  - `rand_write`
  - QD sweep 대상 workload
- queue depth:
  - 특히 QD32 조건에서 tail latency 악화 확인
- direct mode:
  - `direct=0` 조건에서 cache 영향 가능성 확인
- test path:
  - WSL path 비교 조건에서 경로 영향 가능성 확인
- 반복 측정:
  - CV를 통해 run-to-run variation 확인
- 환경/제약:
  - host OS, filesystem, cache, WSL path, 외장 SSD 연결 경로의 영향이 섞일 수 있음

## 봐야 할 지표

- 평균:
  - 평균 IOPS
- tail latency:
  - p99 latency
  - p99.9 latency
- 변동성:
  - CV
  - 반복 측정 간 흔들림
- 보조 지표:
  - workload 조건
  - queue depth
  - direct mode
  - test path

## Mini Lab에서 확인한 것

- QD를 높이면 IOPS가 좋아지는 경우가 있었다.
- 하지만 동시에 p99 latency가 크게 증가하는 조건도 있었다.
- 특히 `rand_write QD32`는 처리량 이득이 제한적인데 p99 latency가 크게 나빠졌다.
- 따라서 최고 IOPS 조건이 항상 좋은 검증 조건은 아니라는 점을 확인했다.
- `direct=0`이나 WSL path 비교처럼 경로와 cache 영향이 섞인 실험에서는 평균 성능이 좋아 보여도 p99 CV가 크게 흔들릴 수 있었다.
- 평균값만 보면 빠른 조건처럼 보이지만, 반복 측정 안정성까지 보면 해석이 달라질 수 있다.

## 해석 기준

- 평균 IOPS가 높다는 것은 처리량이 높다는 뜻이지, 항상 좋은 사용자 경험이나 QoS를 보장한다는 뜻은 아니다.
- p99/p99.9 latency는 일부 느린 요청이 얼마나 나빠지는지 보여준다.
- CV는 같은 조건에서 결과가 얼마나 반복 가능하고 안정적인지 보여준다.
- cache, OS, filesystem, WSL path 같은 요소가 개입하면 평균 성능은 좋아 보여도 SSD 자체 성능으로 바로 해석하면 위험하다.
- 검증 결과는 숫자 자체보다 그 숫자가 나온 조건과 반복 가능성까지 함께 봐야 한다.

## 내 프로젝트와 연결

- 관련 프로젝트:
  - [[SSD Mini Lab Stage 1 회고]]
- 관련 실험:
  - QD sweep
  - `rand_write QD32`
  - direct vs buffered 비교
  - WSL path 비교
  - 반복 측정 CV 분석
- 포트폴리오/면접에서 쓸 포인트:
  - 평균 IOPS만 보지 않고 p99/p99.9 latency와 CV를 함께 해석했다.
  - 최고 처리량 조건이 항상 좋은 검증 조건은 아니라는 점을 실험으로 확인했다.
  - cache와 경로 영향이 섞인 결과를 SSD 자체 성능으로 단정하지 않도록 해석 기준을 세웠다.

## 자기 평가

- 이번 mini-lab을 통해 SSD validation에서 중요한 것은 단순히 빠른 숫자를 찾는 것이 아니라, 그 숫자가 나온 조건을 설명하고 반복 가능성을 확인하는 일이라는 점을 배웠다.
- 평균 IOPS는 보기 쉬운 지표지만, tail latency와 CV를 함께 보지 않으면 실제 QoS나 안정성을 놓칠 수 있다.
- 앞으로는 성능 결과를 정리할 때 평균값, tail latency, 반복 측정 안정성, cache/path 영향 가능성을 함께 기록해야 한다.

## 면접/자소서로 바꿀 수 있는 문장

> SSD mini-lab을 진행하면서 평균 IOPS만으로는 검증 결과를 충분히 설명할 수 없다는 점을 확인했습니다. QD를 높이면 처리량은 증가할 수 있지만 p99 latency가 악화될 수 있고, cache나 경로 영향이 섞이면 평균 성능은 좋아 보여도 반복 측정 안정성은 나빠질 수 있었습니다. 그래서 SSD validation에서는 평균 IOPS뿐 아니라 p99/p99.9 latency와 CV를 함께 보고, 조건이 명확하고 반복 가능한 결과인지 판단해야 한다고 정리했습니다.

## 한계와 다음 확인

- 아직 단정하면 안 되는 부분:
  - 외장 SSD, USB bridge, host OS, filesystem, WSL path 영향이 섞여 있을 수 있다.
  - `direct=0` 결과는 SSD media 자체 성능으로 바로 해석하면 안 된다.
- 추가로 확인할 실험:
  - 더 긴 runtime에서 sustained workload 확인
  - p95/p99/p99.9 중심 QoS 리포트 작성
  - Windows path와 WSL path 차이 재측정
  - 가능한 경우 SMART/NVMe telemetry와 함께 latency 변화 확인

## 관련 노트

- [[SSD Mini Lab Stage 1 회고]]
- [[SSD 단계적 검증 테스트 방법 및 저장장치]]
- [[개발자를 위한 SSD(Coding for SSD) - Part 2 SSD의 아키텍쳐와 벤치마킹]]
