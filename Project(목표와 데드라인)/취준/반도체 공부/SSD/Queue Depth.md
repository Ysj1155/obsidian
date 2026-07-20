---
title: Queue Depth
aliases:
  - QD
  - I/O Queue Depth
  - 큐 깊이
tags:
  - concept
  - ssd
  - ssd-validation
  - qos
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

# Queue Depth

## 한 줄 정의

- Queue Depth는 storage device에 동시에 outstanding 상태로 걸려 있는 I/O 요청의 수를 뜻하며, SSD의 병렬성과 queueing delay를 함께 드러내는 중요한 테스트 조건이다.

## 이 개념의 층위

- 위치: host / driver / protocol / device / 검증
- 누구의 동작인가: host가 요청을 쌓고 device가 처리한다.
- 입력: workload, request size, arrival rate, queue depth 설정
- 출력: IOPS 증가 가능성, latency 증가 가능성, tail latency 악화 가능성

## 왜 중요한가

- SSD는 내부 병렬성을 활용할 수 있기 때문에 QD가 올라가면 IOPS가 증가할 수 있다.
- 하지만 QD가 높아지면 request가 기다리는 시간이 늘어 p99/p99.9 latency가 나빠질 수 있다.
- 검증에서는 QD를 성능을 올리는 knob이 아니라 throughput과 QoS trade-off를 드러내는 조건으로 봐야 한다.

## 핵심 동작 / 원리

1. QD가 낮으면 device에 걸린 요청이 적어 병렬성을 충분히 쓰지 못할 수 있다.
2. QD가 높아지면 더 많은 요청이 동시에 들어가 throughput은 올라갈 수 있다.
3. 동시에 queueing delay가 늘어 일부 요청의 완료 시간이 길어질 수 있다.
4. 평균 IOPS와 p99 latency가 서로 다른 방향으로 움직일 수 있다.

## 대표 시나리오

- `rand_read` QD 1/4/16/32 비교
- `rand_write` QD 1/4/16/32 비교
- `rand_write QD32`처럼 IOPS 이득은 제한적인데 p99가 크게 나빠지는 조건
- sustained workload에서 QD16 조건의 tail behavior 확인

## 무엇과 헷갈리기 쉬운가

- 병렬성 증가와 좋은 조건의 차이:
  - QD 증가로 IOPS가 올라가도 QoS가 좋아졌다는 뜻은 아니다.
- latency 평균과 tail latency의 차이:
  - 평균 latency는 괜찮아도 p99/p99.9가 크게 악화될 수 있다.
- 자주 하는 오해:
  - QD가 높을수록 항상 좋은 조건이라고 보면 안 된다.

## 관련 개념

- 상위 개념:
  - [[fio]]
- 인접 개념:
  - [[p99 latency]]
  - [[SSD QoS]]
- 결과적으로 연결되는 것:
  - [[왜 평균 IOPS만 보면 안 되는가]]
  - [[SSD Mini Lab Portfolio Evidence]]

## 검증 / 실무 관점

- 실제 제품 검증에서 확인할 포인트:
  - QD별 평균 IOPS와 p99/p99.9 latency가 같이 좋아지는가
  - 처리량이 plateau에 도달한 뒤 tail latency만 악화되는 구간이 있는가
  - 반복 측정 CV가 특정 QD에서 커지는가
- 어떤 로그/메트릭/명령으로 볼 수 있는가:
  - fio `iodepth`
  - qd_sweep summary CSV
  - p99 latency plot
  - IOPS vs p99 scatter
- 실무에서 왜 문제가 되는가:
  - product spec의 peak IOPS와 실제 QoS 요구 조건은 다를 수 있다.

## 내 프로젝트와 연결

- 연결되는 프로젝트:
  - [[SSD Mini Lab 프로젝트 허브]]
- 연결되는 실험/결과:
  - [[왜 평균 IOPS만 보면 안 되는가]]
  - [[SSD Mini Lab Portfolio Evidence]]
- 자소서/면접에서 설명할 때 쓸 수 있는 문장:
  - QD sweep을 통해 높은 QD가 항상 좋은 조건이 아니라, 처리량 이득과 p99 latency 비용을 함께 봐야 한다는 점을 확인했습니다.

## 한계 / 예외 / 조건

- 실제 NVMe queue 구조와 fio file-target 조건은 다를 수 있다.
- OS/filesystem/path 영향이 섞이면 QD 효과를 device 내부 병렬성만으로 해석하기 어렵다.

## 출처 / 참고

- [[SSD Mini Lab Portfolio Evidence]]
- [[왜 평균 IOPS만 보면 안 되는가]]
