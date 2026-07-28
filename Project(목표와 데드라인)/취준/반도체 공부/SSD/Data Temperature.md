---
title: Data Temperature
aliases:
  - FTL-observed Data Temperature
  - Hot Cold Data Temperature
  - 데이터 온도
tags:
  - concept
  - ssd
  - ftl
  - garbage-collection
  - ssd-validation
type: concept
status: growing
domain: SSD FTL
created: 2026-07-15
updated: 2026-07-15
source: C:\Users\nei11\venv\venv\GC\docs\data_temperature_model.md
source_type: local-report
reliability: simplified-model
related_projects:
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# Data Temperature

## 한 줄 정의

- Data Temperature는 logical data가 얼마나 자주 다시 write되는지를 기준으로 hot/warm/cold 성향을 추정하는 metadata이며, GC victim selection이나 data placement 판단에 쓰일 수 있다.

## 이 개념의 층위

- 위치: FTL / metadata / GC policy / 검증
- 누구의 동작인가: device / firmware model
- 입력: LPN write history, revisit interval, host write count
- 출력: hot/warm/cold classification, block observed temperature, GC score feature

## 왜 중요한가

- 자주 바뀌는 hot data와 오래 유지되는 cold data를 섞어 두면 GC와 Wear Leveling 비용이 커질 수 있다.
- FTL이 관찰한 data temperature는 workload oracle label이 아니라 device 내부 관찰 기반 feature라는 점에서 의미가 있다.
- 하지만 temperature signal을 잘못 쓰면 invalid page 회수 효율보다 coldness bias가 앞서 WAF가 악화될 수 있다.

## 핵심 동작 / 원리

1. host write마다 LPN별 write history를 갱신한다.
2. 같은 LPN이 짧은 간격으로 다시 write될수록 hot score가 올라간다.
3. LPN을 hot/warm/cold로 분류한다.
4. GC/WL migration은 host write가 아니므로 temperature를 새로 올리지 않고 metadata를 보존한다.
5. block-level observed temperature를 GC victim score feature로 사용할 수 있다.

## 대표 시나리오

- random update workload에서 hot LPN이 반복적으로 write된다.
- TRIM locality가 hot/cold data와 섞이면 reclaim 효율이 달라질 수 있다.
- `temp_aware` 정책은 update-heavy에서 WAF 개선 신호가 있었지만 low OP pressure에서는 비용이 커졌다.

## 무엇과 헷갈리기 쉬운가

- workload oracle label과의 차이:
  - 실험 generator의 hot/cold 설정이 아니라 FTL이 write history로 관찰한 metadata다.
- COTA coldness와의 차이:
  - COTA의 coldness proxy와 달리 LPN revisit interval 기반 observed temperature를 사용한다.
- 자주 하는 오해:
  - data temperature feature가 항상 GC policy를 좋게 만든다고 보면 안 된다.

## 관련 개념

- 상위 개념:
  - [[FTL]]
- 인접 개념:
  - [[SSD Garbage Collection]]
  - [[SSD Wear Leveling]]
  - [[Write Amplification Factor]]
- 결과적으로 연결되는 것:
  - [[Temperature-Aware GC Core Findings]]

## 검증 / 실무 관점

- 실제 제품 검증에서 확인할 포인트:
  - temperature feature가 WAF를 낮추는 조건과 깨지는 조건
  - hot/cold classification이 GC victim 품질에 미치는 영향
  - wear-leveling과 결합했을 때 비용 증가 여부
- 어떤 로그/메트릭/명령으로 볼 수 있는가:
  - temperature_hot_lpn_ratio
  - temperature_warm_lpn_ratio
  - temperature_cold_lpn_ratio
  - observed_temp_ewma
  - gc_events.csv victim temperature fields
- 실무에서 왜 문제가 되는가:
  - feature가 좋아 보여도 low OP pressure 같은 조건에서 failure mode가 생길 수 있다.

## 내 프로젝트와 연결

- 연결되는 프로젝트:
  - [[SSD FTL-GC White-box Validation Lab]]
- 연결되는 실험/결과:
  - [[Temperature-Aware GC Core Findings]]
- 자소서/면접에서 설명할 때 쓸 수 있는 문장:
  - workload oracle label이 아니라 FTL이 관찰한 write history 기반 temperature feature를 GC policy에 연결하고, 이 feature가 어떤 조건에서 유효하고 어디서 깨지는지 분석했습니다.

## 한계 / 예외 / 조건

- 현재 모델은 write temperature만 보며 read hotness는 포함하지 않는다.
- 실제 SSD stream, host hint, NVMe directive와는 다르다.
- simplified model의 feature이므로 실제 firmware 동작으로 확대하면 안 된다.

## 출처 / 참고

- [[Temperature-Aware GC Core Findings]]
- `C:\Users\nei11\venv\venv\GC\docs\data_temperature_model.md`
