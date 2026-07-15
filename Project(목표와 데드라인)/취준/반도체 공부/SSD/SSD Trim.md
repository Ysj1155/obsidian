---
title: SSD Trim
aliases:
  - TRIM
  - SSD TRIM
tags:
  - concept
  - ssd
  - ftl
  - trim
  - ssd-validation
type: concept
status: growing
domain: SSD FTL
created: 2026-02-21
updated: 2026-07-15
source: https://www.kingspec.com/ko/news/what-is-ssd-trim.html
source_type: blog
reliability: medium
related_projects:
  - ftl-gc-whitebox-lab
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# SSD Trim

## 한 줄 정의

- TRIM은 OS가 더 이상 사용하지 않는 logical block 정보를 SSD에 알려, SSD가 해당 data를 invalid로 처리하고 이후 GC에서 더 효율적으로 회수할 수 있게 하는 명령이다.

## 이 개념의 층위

- 위치: OS / 파일시스템 / 인터페이스 / FTL / 검증
- 누구의 동작인가: host가 알려주고 device가 반영한다.
- 입력: 파일 삭제, volume discard, logical block invalidation 정보
- 출력: FTL invalid page marking, GC reclaim 효율 개선 가능성, write amplification 감소 가능성

## 왜 중요한가

- OS에서 파일을 지워도 SSD 내부는 그 physical page가 더 이상 필요 없는지 바로 알 수 없다.
- TRIM이 없으면 SSD는 이미 host 관점에서 삭제된 data도 valid라고 생각해 GC 때 불필요하게 옮길 수 있다.
- TRIM은 GC 효율, WAF, sustained write 성능, 장기 QoS와 연결된다.

## 핵심 동작 / 원리

1. 사용자가 파일을 삭제하거나 파일시스템이 block을 해제한다.
2. OS/file system이 해당 logical block이 더 이상 필요 없다는 정보를 SSD에 전달한다.
3. SSD FTL은 해당 LPN mapping을 invalid 처리할 수 있다.
4. 이후 GC가 해당 page를 valid migration 대상에서 제외할 수 있다.
5. 결과적으로 free block 회수와 WAF 측면에서 유리해질 수 있다.

## 대표 시나리오

- 대량 파일 삭제 후 sustained write 성능이 달라질 수 있다.
- VM image, build cache, dataset처럼 쓰고 지우는 작업이 반복될 때 TRIM 전달 여부가 중요해질 수 있다.
- white-box model에서는 TRIM locality와 reclaim lag를 관찰할 수 있다.

## 무엇과 헷갈리기 쉬운가

- [[SSD Garbage Collection]] 와의 차이:
  - TRIM은 “이 logical data는 더 이상 필요 없다”는 host-side hint이고, GC는 실제 block erase와 free block 회수를 수행하는 device-side 동작이다.
- secure erase와의 차이:
  - TRIM은 보통 성능/관리 목적의 invalidation hint이지, 데이터 보안 삭제를 보장하는 명령으로 단정하면 안 된다.
- 자주 하는 오해:
  - TRIM이 실행됐다고 즉시 NAND block이 erase되는 것은 아니다. 실제 회수는 GC 시점과 firmware 정책에 달려 있다.

## 관련 개념

- 상위 개념:
  - [[FTL]]
- 인접 개념:
  - [[SSD Garbage Collection]]
  - [[SSD Wear Leveling]]
- 결과적으로 연결되는 것:
  - [[Write Amplification Factor]]
  - [[SSD QoS]]

## 검증 / 실무 관점

- 실제 제품 검증에서 확인할 포인트:
  - TRIM 전달 전후 sustained write 또는 random write 성능 변화
  - 삭제/trim 이후 GC reclaim 지연
  - TRIM locality가 GC 효율에 주는 영향
- 어떤 로그/메트릭/명령으로 볼 수 있는가:
  - real SSD: OS discard 설정, filesystem mount option, fio trim workload, telemetry
  - white-box model: trim_events.csv, trim_gc_lag.csv, pending reclaim, reclaim rate
- 실무에서 왜 문제가 되는가:
  - TRIM이 없거나 늦게 반영되면 SSD가 불필요한 valid data를 옮겨 WAF와 tail latency가 나빠질 수 있다.

## 내 프로젝트와 연결

- 연결되는 프로젝트:
  - [[SSD FTL-GC White-box Validation Lab]]
  - [[SSD Mini Lab 프로젝트 허브]]
- 연결되는 실험/결과:
  - `C:\Users\nei11\venv\venv\GC\docs\trim_gc_validation_evidence.md`
  - `C:\Users\nei11\venv\venv\GC\docs\trim_evidence_matrix.md`
- 자소서/면접에서 설명할 때 쓸 수 있는 문장:
  - TRIM은 파일 삭제와 SSD 내부 GC 사이를 연결하는 hint로, 검증에서는 TRIM이 실제 reclaim과 WAF 개선으로 이어지는지 조건과 지표로 확인해야 한다고 이해하고 있습니다.

## 한계 / 예외 / 조건

- TRIM 지원 여부와 동작 방식은 OS, filesystem, interface, drive firmware에 따라 달라질 수 있다.
- TRIM이 성능을 항상 즉시 개선한다고 단정하면 안 된다.
- 외부 fio 실험만으로 TRIM 내부 반영 시점과 GC 동작을 완전히 증명하기는 어렵다.

## 출처 / 참고

- [KingSpec: What is SSD TRIM](https://www.kingspec.com/ko/news/what-is-ssd-trim.html)
- [[SSD Garbage Collection]]
- [[SSD FTL-GC White-box Validation Lab]]
