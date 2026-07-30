---
title: Storage Performance Metrics
aliases:
  - SSD 성능 지표
  - Storage Metrics
  - 저장장치 성능 지표
tags:
  - concept
  - ssd
  - ssd-validation
  - performance
  - qos
type: concept
status: growing
domain: SSD Validation
created: 2026-07-30
updated: 2026-07-30
related_projects:
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - product-engineering
  - test-engineering
---

# Storage Performance Metrics

## 한 줄 정의

Storage performance는 IOPS나 bandwidth 하나로 정해지지 않으며, workload 조건 아래에서 처리량, latency 분포, 반복 안정성을 함께 읽어야 한다.

## 핵심 지표 사전

| 지표 | 답하는 질문 | 주의점 |
| --- | --- | --- |
| IOPS | 초당 몇 개의 I/O를 완료했는가 | block size와 workload 없이 비교하면 의미가 약하다 |
| Bandwidth | 초당 몇 byte를 전송했는가 | MB/s와 MiB/s 단위를 구분한다 |
| Mean latency | 평균 요청은 얼마나 걸렸는가 | 일부 매우 느린 요청을 숨길 수 있다 |
| p99 latency | 99%의 요청이 이 시간 이내에 끝났는가 | 나머지 1%와 sample 수를 함께 생각한다 |
| p99.9 latency | 더 극단적인 tail은 어떤가 | 짧은 run에서는 표본이 부족할 수 있다 |
| Max latency | 가장 느린 요청은 얼마였는가 | 단일 outlier에 매우 민감하다 |
| CV | 반복 측정값이 평균 대비 얼마나 흔들리는가 | 평균이 0에 가까우면 해석이 불안정하다 |

## IOPS와 bandwidth의 관계

고정 block size에서 각 I/O가 같은 크기이고 추가 효과를 단순화하면 다음 관계를 생각할 수 있다.

```text
bandwidth ≈ IOPS × block size
```

예를 들어 4 KiB random I/O의 10,000 IOPS와 1 MiB sequential I/O의 1,000 IOPS는 IOPS 숫자만으로 우열을 비교할 수 없다. 전자는 작은 요청 처리 능력, 후자는 큰 데이터 전송량을 더 강하게 반영한다.

## Latency percentile 읽기

- p50: 중앙값. 전형적인 요청의 감각에 가깝다.
- p95: 느린 쪽 5% 경계다.
- p99: 느린 쪽 1%가 시작되는 경계다.
- p99.9: 느린 쪽 0.1%를 보기 위한 더 엄격한 tail 지표다.

p99가 5 ms라는 말은 모든 요청이 5 ms 이내라는 뜻이 아니다. 약 99%가 그 값 이하였다는 뜻이며, 정확한 percentile 계산은 도구의 집계 방식과 표본 수에 영향을 받는다.

## CV와 반복 안정성

변동계수(Coefficient of Variation)는 보통 다음처럼 계산한다.

```text
CV = 표준편차 / 평균
```

- 단위가 다른 지표의 상대 변동성을 비교하는 데 유용하다.
- IOPS CV와 p99 CV는 서로 다른 질문에 답한다.
- 반복 횟수가 적으면 CV 자체도 불안정할 수 있다.
- 평균이 0에 가깝거나 부호가 있는 값에는 단순 적용을 조심한다.

## QD와 함께 읽기

[[Queue Depth]]가 높아지면 device 병렬성을 더 활용해 IOPS나 bandwidth가 증가할 수 있다. 동시에 queue 대기 시간이 늘어 latency와 tail이 악화될 수 있다.

따라서 QD sweep에서는 다음을 같이 본다.

- 처리량이 어디서 포화되는가
- 평균 latency와 p99/p99.9가 언제 급격히 증가하는가
- 처리량 증가분이 latency 비용을 정당화하는가
- 반복 run에서 같은 패턴이 재현되는가

## Baseline의 네 가지 workload

입문 검증에서는 흔히 다음 네 조건으로 동작의 큰 윤곽을 잡는다.

| Workload | 주로 보는 것 |
| --- | --- |
| Sequential read | 연속 읽기 bandwidth와 read path |
| Sequential write | 연속 쓰기 bandwidth와 cache/sustained 변화 |
| Random read | 작은 분산 읽기의 IOPS와 latency |
| Random write | FTL update, cache, GC 압력 아래의 IOPS와 tail |

이 네 가지가 모든 사용 사례를 대표하지는 않는다. Block size, read/write mix, QD, runtime, data set size를 붙여야 실제 workload가 된다.

## 검증 / 실무 관점

- 숫자마다 workload, block size, QD, jobs, runtime, direct mode, path를 함께 적는다.
- 서로 다른 block size나 read/write shape을 하나의 순위표로 섞지 않는다.
- 평균과 percentile, run-to-run CV를 함께 본다.
- Sustained workload에서는 시간에 따른 성능과 tail 변화를 확인한다.
- Telemetry와 error log를 결합하되, 상관관계를 원인으로 단정하지 않는다.

## 내 프로젝트와 연결

- [[SSD Mini Lab 프로젝트 허브]]에서 baseline, QD sweep, direct/buffered, path 비교에 사용한 해석 기준이다.
- [[왜 평균 IOPS만 보면 안 되는가]]는 높은 IOPS와 악화된 p99 latency가 동시에 나타날 수 있음을 정리한다.
- [[fio Result Pipeline]]은 이 지표를 raw JSON에서 일관된 단위로 추출한다.

## 면접용 문장

Storage 성능은 IOPS 하나로 평가하지 않고 workload와 block size, QD를 고정한 상태에서 bandwidth, p99/p99.9 latency, 반복 CV를 함께 봤습니다. 이를 통해 처리량이 증가해도 QoS가 악화되는 조건을 구분했습니다.

## 관련 노트

- [[fio]]
- [[Queue Depth]]
- [[p99 latency]]
- [[SSD QoS]]
- [[fio Result Pipeline]]
- [[SSD Host Device Path]]
- [[SSD Telemetry]]
