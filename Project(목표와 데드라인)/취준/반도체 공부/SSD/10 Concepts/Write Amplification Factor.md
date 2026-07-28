---
title: Write Amplification Factor
aliases:
  - WAF
  - Write Amplification
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
source: C:\Users\nei11\venv\venv\GC\docs\portfolio_gc_evidence.md
source_type: local-report
reliability: simplified-model
related_projects:
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# Write Amplification Factor

## 한 줄 정의

- Write Amplification Factor, WAF는 host가 요청한 write 양에 비해 SSD 내부 NAND에 실제로 더 많이 기록된 비율을 나타내는 지표다.

## 이 개념의 층위

- 위치: FTL / NAND / 검증
- 누구의 동작인가: device / firmware
- 입력: host write, overwrite, GC migration, TRIM, wear-leveling migration
- 출력: 내부 device write 증가, endurance 비용, 성능/QoS 비용 가능성

## 왜 중요한가

- SSD는 out-of-place write와 block erase 제약 때문에 host write보다 더 많은 내부 write가 발생할 수 있다.
- WAF가 높으면 NAND 마모, GC 비용, latency spike 가능성이 커진다.
- GC policy를 평가할 때 WAF는 중요한 지표지만, WAF 하나만으로 좋은 정책을 정하면 wear balance나 QoS를 놓칠 수 있다.

## 핵심 동작 / 원리

1. host가 logical page write를 요청한다.
2. SSD는 새 free physical page에 data를 기록한다.
3. 기존 page는 invalid가 된다.
4. GC가 valid page migration을 수행하면 host가 직접 요청하지 않은 내부 write가 추가된다.
5. 내부 write / host write 비율이 WAF로 나타난다.

## 대표 시나리오

- update-heavy workload에서 invalid page와 GC가 많이 생겨 WAF가 높아진다.
- TRIM이 잘 반영되면 불필요한 valid migration이 줄어 WAF가 낮아질 수 있다.
- Wear Leveling은 erase count 균등화에는 도움이 되지만 WAF를 올릴 수 있다.

## 무엇과 헷갈리기 쉬운가

- IOPS와의 차이:
  - IOPS는 외부 처리량이고, WAF는 내부 write 비용이다.
- wear_std와의 차이:
  - WAF는 write 증폭, wear_std는 erase count 분산 정도다.
- 자주 하는 오해:
  - WAF가 가장 낮은 정책이 항상 좋은 정책은 아니다. wear, latency, metadata cost를 함께 봐야 한다.

## 관련 개념

- 상위 개념:
  - [[SSD Garbage Collection]]
- 인접 개념:
  - [[SSD Wear Leveling]]
  - [[SSD Trim]]
  - [[GC Pause]]
- 결과적으로 연결되는 것:
  - endurance
  - QoS
  - policy trade-off

## 검증 / 실무 관점

- 실제 제품 검증에서 확인할 포인트:
  - update-heavy 조건에서 WAF가 어떻게 달라지는가
  - WAF 감소가 wear_std 또는 latency cost를 만들지는 않는가
  - TRIM locality와 reclaim 효율이 WAF에 어떤 영향을 주는가
- 어떤 로그/메트릭/명령으로 볼 수 있는가:
  - white-box model: host writes, device writes, moved valid pages
  - real SSD: vendor telemetry 없이는 직접 WAF 확인이 어려울 수 있음
- 실무에서 왜 문제가 되는가:
  - WAF는 endurance와 sustained write behavior의 중요한 원인이 될 수 있다.

## 내 프로젝트와 연결

- 연결되는 프로젝트:
  - [[SSD FTL-GC White-box Validation Lab]]
- 연결되는 실험/결과:
  - [[Request Timing MVP]]
  - [[Temperature-Aware GC Core Findings]]
  - [[SSD Garbage Collection]]
- 자소서/면접에서 설명할 때 쓸 수 있는 문장:
  - GC policy를 WAF 하나로만 평가하지 않고 wear, TRIM reclaim, metadata cost, latency proxy까지 함께 보며 trade-off를 해석했습니다.

## 한계 / 예외 / 조건

- 실제 SSD의 WAF는 내부 telemetry 없이는 외부 fio만으로 직접 계산하기 어렵다.
- simulator WAF는 모델 내부 metric이므로 실제 NAND 동작과 동일하다고 주장하면 안 된다.

## 출처 / 참고

- [[SSD FTL-GC White-box Validation Lab]]
- [[SSD Garbage Collection]]
- `C:\Users\nei11\venv\venv\GC\docs\portfolio_gc_evidence.md`

