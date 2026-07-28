---
title: Resource Contention MVP
aliases:
  - PAL-lite Resource Contention
  - GC Resource Contention MVP
tags:
  - experiment
  - validation
  - ssd
  - ftl
  - garbage-collection
  - latency
type: experiment-report
status: growing
domain: SSD FTL Validation
created: 2026-07-20
updated: 2026-07-20
source: C:\Users\nei11\venv\venv\GC\docs\resource_contention_mvp.md
source_type: local-report
reliability: simplified-model
related_projects:
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# Resource Contention MVP

## 한 줄 결론

- Resource Contention MVP는 Request + Timing MVP의 단일 FIFO timing 모델을 확장해, channel/die처럼 단순화한 resource가 host I/O와 GC migration/erase에 의해 어떻게 기다림을 만드는지 관찰하는 단계다.

## 실험 목적

- 검증 질문:
  - 같은 FTL 상태 변화와 같은 workload에서, resource를 단일 FIFO로 볼 때와 여러 resource로 나눠 볼 때 tail latency 해석이 어떻게 달라지는가.
- 왜 이 실험이 필요한가:
  - [[Request Timing MVP]]는 GC pause를 request latency proxy에 연결했지만, 모든 일을 하나의 줄에 세우는 모델이었다.
  - 실제 SSD처럼 주장할 수는 없더라도, host request와 GC 작업이 같은 내부 자원을 놓고 충돌할 때 생기는 tail risk를 따로 볼 필요가 있다.
- 기대한 관찰:
  - FTL/GC 핵심 결과인 WAF, GC count, wear는 보존하면서 resource wait과 GC resource wait만 추가로 설명할 수 있어야 한다.

## 조건 매트릭스

| 항목 | 값 |
|---|---|
| timing modes | `serial`, `pal_lite` |
| resource mapping | `block_idx % (channels * dies_per_channel)` |
| 기본 PAL-lite 예 | 4 channels x 2 dies |
| selected one-page interval | 150 us |
| mixed multi-page interval | 250 us steady + 50 us burst |
| 핵심 보존 조건 | WAF, GC count, wear 결과가 serial/PAL-lite pair에서 유지되어야 함 |
| 제외 | 실제 NAND scheduler, NVMe latency, firmware priority, plane/page-type parallelism |

## 실행 / 산출물

- 원본 문서: `C:\Users\nei11\venv\venv\GC\docs\resource_contention_mvp.md`
- 후속 실험 설계: `C:\Users\nei11\venv\venv\GC\docs\resource_contention_quality_experiment.md`
- 연결 노트:
  - [[Request Timing MVP]]
  - [[Resource Contention Quality Experiment]]
  - [[GC Pause]]

## 결과 요약

| 관찰 | 의미 |
|---|---|
| 50 us overload 조건 | serial은 포화되어 초 단위 tail이 생겼고, PAL-lite는 병렬 resource 덕분에 훨씬 낮은 tail을 보였다. |
| 150 us 선택 | serial도 offered load를 따라가면서 contention 차이가 보이는 비교점으로 선택됐다. |
| FTL 결과 보존 | timing layer가 WAF/GC/wear 같은 core FTL 결과를 바꾸지 않는지 확인하는 것이 핵심 QC다. |
| 새 지표 | `resource_wait_us`, `gc_resource_wait_us`, resource별 GC victim/destination 정보를 request event에 남긴다. |

## 해석

- 결과가 보여주는 것:
  - 단순 timing model에서도 resource 충돌 가정을 바꾸면 p95/p99 tail latency 해석이 크게 달라질 수 있다.
  - GC victim migration과 erase가 foreground request와 같은 resource를 잡으면 GC pause가 tail latency로 드러날 수 있다.
- 결과가 보여주지 못하는 것:
  - 실제 SSD의 channel/die scheduling이나 firmware latency를 예측하지는 못한다.
  - USB, OS, filesystem, NVMe queueing 같은 black-box 실측 경로와 직접 합치면 안 된다.
- 가장 중요한 trade-off:
  - WAF와 wear가 같아도 resource contention 가정이 달라지면 tail latency 결론이 달라질 수 있다.

## 이상치 / 리스크

- 이상하게 보이는 조건:
  - WAF/GC count는 같은데 PAL-lite와 serial의 p99.9가 크게 달라지는 경우
  - GC resource wait count는 적지만 p99.9나 max latency에 큰 영향을 주는 경우
- 가능한 원인:
  - 특정 victim block과 host target block이 같은 resource에 매핑되는 충돌
  - burst arrival이 resource wait을 누적시키는 상황
- 아직 증거가 없는 원인:
  - 실제 SSD 내부 scheduler 정책
  - 실제 NAND die/channel 병렬성

## 포트폴리오 / 면접 포인트

> Request timing model을 PAL-lite resource contention 모델로 확장해, GC migration/erase와 host I/O가 같은 내부 resource를 사용할 때 tail latency proxy가 어떻게 달라지는지 검증했습니다. 실제 SSD latency 예측이 아니라, 같은 FTL 결과를 보존한 채 scheduling/resource 가정만 바꿔 보는 white-box sensitivity test로 설명할 수 있습니다.

## 관련 노트

- [[SSD FTL-GC White-box Validation Lab]]
- [[Request Timing MVP]]
- [[Resource Contention Quality Experiment]]
- [[Request Timing Policy Findings]]
- [[GC Pause]]
- [[Write Amplification Factor]]
