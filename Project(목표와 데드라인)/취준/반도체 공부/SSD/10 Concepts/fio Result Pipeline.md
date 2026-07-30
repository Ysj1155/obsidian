---
title: fio Result Pipeline
aliases:
  - fio 결과 파이프라인
  - fio JSON CSV Pipeline
tags:
  - concept
  - ssd
  - ssd-validation
  - fio
  - data-pipeline
type: concept
status: growing
domain: SSD Validation
created: 2026-07-30
updated: 2026-07-30
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

# fio Result Pipeline

## 한 줄 정의

fio Result Pipeline은 workload 조건과 raw JSON을 원본으로 보존하고, 필요한 지표를 일관된 단위의 CSV로 정규화한 뒤 plot과 report까지 추적 가능하게 연결하는 검증 데이터 흐름이다.

## 전체 흐름

```text
job/config
  -> fio 실행
  -> raw JSON + 실행 로그
  -> parser / validation
  -> normalized CSV
  -> plot / comparison
  -> report / claim
```

## 각 산출물의 역할

| 단계 | 남기는 것 | 역할 |
| --- | --- | --- |
| 입력 | job file, CLI option, 환경 snapshot | 무엇을 실행했는지 재현 |
| 원본 | fio JSON, stderr, exit code | 가공 전 evidence 보존 |
| 정규화 | CSV row | 조건과 핵심 지표를 비교 가능한 형태로 통일 |
| 분석 | 표, plot, CV 계산 | 조건 간 패턴과 이상값 확인 |
| 해석 | Markdown report | 관찰, 가설, 한계를 분리해 설명 |

## 왜 JSON에서 CSV로 바꾸는가

- JSON은 fio가 제공하는 상세 원본과 nested 구조를 보존하기 좋다.
- CSV는 여러 run을 행 단위로 모아 비교, 필터, 통계, plotting하기 좋다.
- Parser는 단순 형식 변환기가 아니라 metric 이름, 단위, workload 조건을 통일하는 경계다.
- 최종 표만 남기지 않고 raw JSON을 보존해야 parser 오류나 해석 변경을 다시 검증할 수 있다.

## 최소 보존 필드

- 식별: run ID, timestamp, result filename
- 조건: workload, read/write, block size, QD, jobs, runtime, direct mode, test path
- 평균: IOPS, bandwidth
- latency: mean, p95, p99, p99.9, max
- 실행 상태: fio version, exit status, error
- 환경: OS/path/device/bridge 정보와 telemetry snapshot 위치

## Parser에서 특히 조심할 점

1. fio 버전과 output schema 차이를 확인한다.
2. `read`, `write`, `trim` 중 실제 workload에 해당하는 section을 선택한다.
3. `clat_ns`, `clat_us`처럼 latency 단위를 하나로 정규화한다.
4. percentile key를 문자열/부동소수점 표현 때문에 잘못 놓치지 않는다.
5. 여러 job의 값을 합칠 때 IOPS 합계와 latency 분포를 같은 방식으로 단순 평균하지 않는다.
6. 누락값을 0으로 위장하지 않고 unavailable로 남긴다.
7. 파일명에서 추측한 조건과 JSON 내부 조건이 일치하는지 검사한다.

## 대표 시나리오

```text
rand_write_bs4k_qd32_run03.json
  -> workload=rand_write
  -> bs=4KiB
  -> qd=32
  -> iops=...
  -> p99_us=...
  -> p999_us=...
  -> CSV 한 행
```

반복 run이 쌓이면 조건별 평균과 CV를 계산하고, [[Storage Performance Metrics]] 기준으로 throughput과 tail latency를 함께 비교한다.

## 무엇과 헷갈리기 쉬운가

- CSV는 원본이 아니라 분석용 파생물이다.
- Plot은 evidence를 보기 쉽게 만든 것이지 raw result를 대체하지 않는다.
- Parser가 성공했다고 실험이 유효한 것은 아니다. fio error, runtime, path, file size도 확인해야 한다.
- 소수점 단위 변환이나 percentile 누락은 그래프 모양은 그럴듯해도 결론을 망칠 수 있다.

## 검증 / 실무 관점

- 같은 raw JSON에서 같은 CSV가 재생성되는지 확인한다.
- Parser unit test에는 정상, 누락, read/write 혼합, 단위 차이 사례를 넣는다.
- Report의 표/그림이 어떤 CSV와 raw JSON에서 왔는지 역추적 가능해야 한다.
- 결과 파일을 덮어쓰기보다 run ID로 구분해 audit trail을 남긴다.

## 내 프로젝트와 연결

- [[SSD Mini Lab 프로젝트 허브]]에서 fio 실행 결과를 JSON→CSV→plot→report로 이어 붙였다.
- [[왜 평균 IOPS만 보면 안 되는가]]의 판단은 평균 IOPS뿐 아니라 percentile과 반복 CV를 함께 추출했기 때문에 가능했다.
- [[SSD Host Device Path]] 정보가 빠지면 같은 숫자라도 어떤 경로의 결과인지 해석하기 어렵다.

## 면접용 문장

fio를 실행하는 데서 끝내지 않고 raw JSON을 보존한 뒤 조건과 단위를 검증하는 parser로 CSV를 만들고, plot과 report가 원본까지 역추적되도록 결과 파이프라인을 구성했습니다.

## 관련 노트

- [[fio]]
- [[Storage Performance Metrics]]
- [[SSD Host Device Path]]
- [[SSD QoS]]
- [[NVMe SMART Telemetry]]
- [[SSD Mini Lab Portfolio Evidence]]
