---
title: SSD 허브
aliases:
  - SSD Hub
  - SSD 공부 허브
tags:
  - hub
  - ssd
  - ssd-validation
  - portfolio
type: hub
status: growing
domain: SSD
created: 2026-02-21
updated: 2026-07-15
related_projects:
  - ssd-mini-lab
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - product-engineering
  - test-engineering
---

# SSD 허브

SSD를 구조, 성능, 제품, 검증 관점으로 연결하는 메인 허브.

이 허브의 목적은 SSD 지식을 단순히 모아두는 것이 아니라, **공부한 개념 → 실험/검증 관점 → 포트폴리오/면접 문장**으로 이어지게 만드는 것이다.

## 먼저 읽을 것

1. [[SSD 기본 원리와 구조]]
2. [[개발자를 위한 SSD(Coding for SSD) - Part 6 A Summary - What every programmer should know about solid-state drives]]
3. [[SSD Mini Lab 프로젝트 허브]]
4. [[SSD FTL-GC White-box Validation Lab]]

## 핵심 개념 지도

### 기초 구조

- [[SSD 기본 원리와 구조]]
- [[SSD 인터페이스]]
- [[SSD 통신 프로토콜]]
- [[클라이언트 SSD]]
- [[엔터프라이즈 SSD]]

### 내부 동작

- [[개발자를 위한 SSD(Coding for SSD) - Part 3 페이지 & 블록 & FTL(Flash Translation Layer)]]
- [[SSD Garbage Collection]]
- [[SSD Wear Leveling]]
- [[SSD Trim]]
- [[SSD 전력 손실 보호 원리]]
- [[Write Amplification Factor]]
- [[Data Temperature]]
- [[PVB Metadata Model]]

### 성능과 QoS

- [[개발자를 위한 SSD(Coding for SSD) - Part 2 SSD의 아키텍쳐와 벤치마킹]]
- [[개발자를 위한 SSD(Coding for SSD) - Part 4 고급 기능과 내부 병렬 처리]]
- [[개발자를 위한 SSD(Coding for SSD) - Part 5 접근 방법과 시스템 최적화]]
- [[왜 평균 IOPS만 보면 안 되는가]]
- [[SSD Mini Lab Portfolio Evidence]]

### 검증 / 실무

- [[SSD 단계적 검증 테스트 방법 및 저장장치]]
- [[OCP-OPF-Testing-and-Validation]]
- [[Open Source SSD Validation in Practice Updates from Meta's Qualification Framework]]
- [[SSD Mini Lab Stage 1 회고]]
- [[SSD Mini Lab Portfolio Evidence]]
- [[Request Timing MVP]]
- [[Temperature-Aware GC Core Findings]]

### 제품 / 시장 / 회사 이해

- [[Fadu PCIe 3.0 SSD 브로셔]]
- [[Fadu PCIe 4.0 SSD 브로셔]]
- [[엔터프라이즈 스토리지 계층화와 AI Workload]]

## 추천 읽기 순서

1. [[SSD 기본 원리와 구조]]
2. [[개발자를 위한 SSD(Coding for SSD) - Part 3 페이지 & 블록 & FTL(Flash Translation Layer)]]
3. [[SSD Garbage Collection]]
4. [[SSD Wear Leveling]]
5. [[SSD Trim]]
6. [[개발자를 위한 SSD(Coding for SSD) - Part 2 SSD의 아키텍쳐와 벤치마킹]]
7. [[SSD Mini Lab 프로젝트 허브]]
8. [[왜 평균 IOPS만 보면 안 되는가]]
9. [[SSD FTL-GC White-box Validation Lab]]
10. [[Request Timing MVP]]

## 이 주제의 핵심 질문

- SSD는 HDD와 비교해서 왜 빠르며, 그 빠름은 어떤 조건에서 흔들리는가?
- FTL, GC, Wear Leveling, TRIM은 성능과 수명에 어떤 영향을 주는가?
- SSD validation에서는 평균 성능보다 어떤 지표를 함께 봐야 하는가?
- host, OS, filesystem, cache, interface, device 내부 동작 중 어느 계층의 영향인지 어떻게 구분할 수 있는가?
- 제품 브로셔나 스펙에 적힌 성능 수치를 실제 검증 관점으로 어떻게 해석해야 하는가?

## 제품 / 실무 / 검증 연결

- 관련 제품:
  - [[클라이언트 SSD]]
  - [[엔터프라이즈 SSD]]
  - [[Fadu PCIe 3.0 SSD 브로셔]]
  - [[Fadu PCIe 4.0 SSD 브로셔]]
- 관련 테스트 항목:
  - sequential read/write
  - random read/write
  - queue depth sweep
  - p99/p99.9 latency
  - run-to-run variation
  - direct I/O vs buffered I/O
  - WAF / GC pause / wear trade-off
  - PVB metadata correction cost
- 관련 직무 키워드:
  - SSD validation
  - product engineering
  - firmware validation
  - QoS
  - fio
  - workload
  - telemetry
  - SMART/NVMe health context
  - reproducibility

## 프로젝트 연결

- 프로젝트 허브:
  - [[SSD Mini Lab 프로젝트 허브]]
  - [[SSD FTL-GC White-box Validation Lab]]
- black-box 측정 track:
  - [[SSD Mini Lab Stage 1 회고]]
  - [[SSD Mini Lab Portfolio Evidence]]
  - [[왜 평균 IOPS만 보면 안 되는가]]
  - [[fio]]
  - [[Queue Depth]]
  - [[p99 latency]]
  - [[SSD QoS]]
- white-box 내부 모델 track:
  - [[Request Timing MVP]]
  - [[Temperature-Aware GC Core Findings]]
  - [[SSD Garbage Collection]]
  - [[SSD Wear Leveling]]
  - [[SSD Trim]]
  - [[Write Amplification Factor]]
  - [[GC Pause]]
  - [[Data Temperature]]
  - [[PVB Metadata Model]]
- 추후 포트폴리오/자소서로 연결 가능한 노트:
  - [[SSD 단계적 검증 테스트 방법 및 저장장치]]
  - [[OCP-OPF-Testing-and-Validation]]
  - [[Open Source SSD Validation in Practice Updates from Meta's Qualification Framework]]

## 노트 타입별 분류

### 프로젝트 허브

- [[SSD Mini Lab 프로젝트 허브]]
- [[SSD FTL-GC White-box Validation Lab]]

### 개념 노트

- [[SSD 기본 원리와 구조]]
- [[SSD 인터페이스]]
- [[SSD 통신 프로토콜]]
- [[SSD Garbage Collection]]
- [[SSD Wear Leveling]]
- [[SSD Trim]]
- [[SSD 전력 손실 보호 원리]]
- [[Write Amplification Factor]]
- [[Data Temperature]]
- [[PVB Metadata Model]]

### 문서 / 자료 스크랩

- [[개발자를 위한 SSD(Coding for SSD) - Part 2 SSD의 아키텍쳐와 벤치마킹]]
- [[개발자를 위한 SSD(Coding for SSD) - Part 3 페이지 & 블록 & FTL(Flash Translation Layer)]]
- [[개발자를 위한 SSD(Coding for SSD) - Part 4 고급 기능과 내부 병렬 처리]]
- [[개발자를 위한 SSD(Coding for SSD) - Part 5 접근 방법과 시스템 최적화]]
- [[개발자를 위한 SSD(Coding for SSD) - Part 6 A Summary - What every programmer should know about solid-state drives]]
- [[Fadu PCIe 3.0 SSD 브로셔]]
- [[Fadu PCIe 4.0 SSD 브로셔]]
- [[OCP-OPF-Testing-and-Validation]]
- [[Open Source SSD Validation in Practice Updates from Meta's Qualification Framework]]

### 검증 포인트 / 실험 리포트 / 회고

- [[왜 평균 IOPS만 보면 안 되는가]]
- [[SSD Mini Lab Stage 1 회고]]
- [[SSD Mini Lab Portfolio Evidence]]
- [[Request Timing MVP]]
- [[Temperature-Aware GC Core Findings]]

## 아직 비어 있거나 보강할 노트

- 아직 없는 필수 노트:
  - 현재 SSD 허브 기준 핵심 후보는 1차 생성 완료
- 보강이 필요한 노트:
  - [[SSD 통신 프로토콜]]
  - [[SSD]]
  - [[SSD 인터페이스]]

## 업데이트 로그

- 2026-07-15:
  - PARA branch 구조 테스트용으로 허브 노트 재정리.
  - SSD 노트를 개념, 문서/자료, 검증 포인트, 프로젝트 회고 흐름으로 재분류.
  - [[SSD Mini Lab 프로젝트 허브]], [[SSD FTL-GC White-box Validation Lab]]를 프로젝트 branch로 추가.
  - [[SSD Mini Lab Portfolio Evidence]], [[Request Timing MVP]], [[Temperature-Aware GC Core Findings]]를 evidence branch로 추가.
  - [[fio]], [[Queue Depth]], [[p99 latency]], [[SSD QoS]], [[Write Amplification Factor]], [[GC Pause]], [[Data Temperature]], [[PVB Metadata Model]] 핵심 후보 노트 생성.
  - [[External SSD Product Validation]], [[NVMe SMART Telemetry]] 제품형 검증/telemetry 노트 생성.

