---
title: SSD Wear Leveling
aliases:
  - Wear Leveling
  - 웨어 레벨링
tags:
  - concept
  - ssd
  - ftl
  - wear-leveling
  - ssd-validation
type: concept
status: growing
domain: SSD FTL
created: 2026-02-21
updated: 2026-07-15
source:
source_type:
reliability:
related_projects:
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# SSD Wear Leveling

## 한 줄 정의

- Wear Leveling은 NAND block의 erase/program 마모가 특정 block에 몰리지 않도록 erase count를 가능한 균등하게 분산하는 SSD 수명 관리 기법이다.

## 이 개념의 층위

- 위치: 컨트롤러 / FTL / NAND / 검증
- 누구의 동작인가: device / firmware
- 입력: block erase count, valid page 분포, hot/cold data, free block 압력, GC policy
- 출력: erase count 분산, valid data migration, WAF 증가 가능성, endurance 개선 가능성

## 왜 중요한가

- NAND block은 P/E cycle 수명 한계가 있다.
- 일부 block만 과도하게 erase되면 전체 SSD 수명과 reliability에 문제가 생길 수 있다.
- Wear Leveling은 수명에는 유리하지만, valid page를 추가로 옮기면 WAF와 latency 비용이 생길 수 있다.
- SSD validation에서는 성능 지표와 endurance 관련 지표를 trade-off로 봐야 한다.

## 핵심 동작 / 원리

1. FTL이 block별 erase count와 data placement 상태를 추적한다.
2. 많이 마모된 block과 덜 마모된 block의 차이를 줄이도록 data를 재배치한다.
3. static wear leveling은 오래 움직이지 않는 cold data까지 옮겨 erase 기회를 분산할 수 있다.
4. 이 과정에서 additional migration이 발생해 WAF 비용이 생긴다.

## 대표 시나리오

- hot data가 특정 logical range에 몰리면 일부 physical block이 더 자주 erase될 수 있다.
- enterprise SSD에서는 장기 endurance와 DWPD 관점에서 wear 분산이 중요하다.
- GC victim selection이 immediate reclaim만 보면 WAF는 낮아질 수 있지만 wear spread가 커질 수 있다.

## 무엇과 헷갈리기 쉬운가

- [[SSD Garbage Collection]] 와의 차이:
  - GC는 free block 회수, Wear Leveling은 erase count 분산이 핵심 목적이다.
- [[SSD Trim]] 와의 차이:
  - TRIM은 invalidation 정보를 전달하는 host command이고, Wear Leveling은 device 내부 수명 관리 정책이다.
- 자주 하는 오해:
  - wear_std가 낮다고 항상 좋은 정책이라고 말하면 안 된다. WAF, moved valid pages, QoS 비용도 함께 봐야 한다.

## 관련 개념

- 상위 개념:
  - [[FTL]]
- 인접 개념:
  - [[SSD Garbage Collection]]
  - [[Write Amplification Factor]]
  - [[Data Temperature]]
- 결과적으로 연결되는 것:
  - endurance
  - DWPD
  - retention

## 검증 / 실무 관점

- 실제 제품 검증에서 확인할 포인트:
  - workload 장기 실행 후 erase count 분산
  - WAF와 wear_std의 trade-off
  - static WL이 성능 비용을 얼마나 만드는지
- 어떤 로그/메트릭/명령으로 볼 수 있는가:
  - white-box model: wear_avg, wear_std, wear_max, moved valid pages
  - real SSD: vendor telemetry나 NVMe SMART/health log가 필요할 수 있음
- 실무에서 왜 문제가 되는가:
  - 수명 개선을 위해 너무 공격적으로 데이터를 옮기면 성능과 QoS가 나빠질 수 있다.

## 내 프로젝트와 연결

- 연결되는 프로젝트:
  - [[SSD FTL-GC White-box Validation Lab]]
- 연결되는 실험/결과:
  - `C:\Users\nei11\venv\venv\GC\docs\wear_leveling_tuning_findings.md`
  - `C:\Users\nei11\venv\venv\GC\docs\portfolio_gc_evidence.md`
- 자소서/면접에서 설명할 때 쓸 수 있는 문장:
  - Wear Leveling은 endurance를 위해 필요하지만 WAF와 migration 비용을 만들 수 있어, 검증에서는 wear_std 하나가 아니라 WAF와 latency proxy까지 함께 봐야 한다고 이해하고 있습니다.

## 한계 / 예외 / 조건

- 실제 SSD의 wear metric은 firmware/vendor telemetry에 의존하므로 외부 black-box fio만으로 직접 확인하기 어렵다.
- simulator의 erase count는 개념 검증용 proxy이며 실제 NAND health와 동일하지 않다.

## 출처 / 참고

- [[SSD FTL-GC White-box Validation Lab]]
- [[SSD Garbage Collection]]
