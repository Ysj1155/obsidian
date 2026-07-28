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
updated: 2026-07-28
related_projects:
  - ssd-mini-lab
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - product-engineering
  - firmware-validation
  - test-engineering
---

# SSD 허브

SSD를 구조, 성능, 제품 검증, FTL/GC 내부 모델 관점으로 연결하는 메인 허브.

이 허브의 목적은 SSD 지식을 단순히 모으는 것이 아니라, 개념 노트, 실험 리포트, 검증 포인트, 포트폴리오 문장으로 이어지는 branch를 만드는 것이다.

## 먼저 읽을 것

1. [[SSD 기본 원리와 구조]]
2. [[개발자를 위한 SSD(Coding for SSD) - Part 6 A Summary - What every programmer should know about solid-state drives]]
3. [[SSD Mini Lab 프로젝트 허브]]
4. [[SSD FTL-GC White-box Validation Lab]]
5. [[왜 평균 IOPS만 보면 안 되는가]]

## 프로젝트 허브

- [[SSD Mini Lab 프로젝트 허브]]
  - 실제 SSD를 fio로 측정하는 black-box validation track.
  - 핵심 산출물: [[SSD Mini Lab Portfolio Evidence]], [[External SSD Product Validation]], [[External SSD Block Size Sweep]], [[External SSD Data Integrity]], [[재현되지 않은 가설도 검증 결과다]], [[왜 평균 IOPS만 보면 안 되는가]]
- [[SSD FTL-GC White-box Validation Lab]]
  - FTL/GC/TRIM/wear/resource contention, recovery, request durability, controller ownership을 Python simulator로 관찰하는 white-box validation track.
  - 핵심 산출물: [[Request Timing MVP]], [[Request Timing Policy Findings]], [[Resource Contention MVP]], [[Resource Contention Quality Experiment]], [[FTL Metadata Recovery and Bad Block Handling]], [[Durable Request Replay]], [[Controller Lease and External Fencing]]

## 개념 지도

### 기초 구조

- [[SSD]]
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
- [[fio]]
- [[Queue Depth]]
- [[p99 latency]]
- [[SSD QoS]]
- [[GC Pause]]
- [[왜 평균 IOPS만 보면 안 되는가]]

### 검증 / 실무

- SSD 단계별 검증 테스트 방법 및 저장장치
- [[OCP-OPF-Testing-and-Validation]]
- [[Open Source SSD Validation in Practice Updates from Meta's Qualification Framework]]
- [[SSD Mini Lab Stage 1 회고]]
- [[SSD Mini Lab Portfolio Evidence]]
- [[External SSD Product Validation]]
- [[External SSD Block Size Sweep]]
- [[External SSD Data Integrity]]
- [[재현되지 않은 가설도 검증 결과다]]
- [[NVMe SMART Telemetry]]
- [[Request Timing MVP]]
- [[Request Timing Policy Findings]]
- [[Resource Contention MVP]]
- [[Resource Contention Quality Experiment]]
- [[Temperature-Aware GC Core Findings]]
- [[FTL Metadata Recovery and Bad Block Handling]]
- [[Durable Request Replay]]
- [[Controller Lease and External Fencing]]

### 제품 / 시장 / 회사 이해

- [[Fadu PCIe 3.0 SSD 브로셔]]
- [[Fadu PCIe 4.0 SSD 브로셔]]
- [[엔터프라이즈 스토리지 계층화와 AI Workload]]

## 공부 순서

### Black-box 실측 track

1. [[fio]]
2. [[Queue Depth]]
3. [[p99 latency]]
4. [[SSD QoS]]
5. [[SSD Mini Lab 프로젝트 허브]]
6. [[SSD Mini Lab Portfolio Evidence]]
7. [[External SSD Product Validation]]
8. [[External SSD Block Size Sweep]]
9. [[External SSD Data Integrity]]
10. [[재현되지 않은 가설도 검증 결과다]]
11. [[왜 평균 IOPS만 보면 안 되는가]]

### White-box FTL/GC track

1. [[SSD Garbage Collection]]
2. [[SSD Wear Leveling]]
3. [[SSD Trim]]
4. [[Write Amplification Factor]]
5. [[GC Pause]]
6. [[Request Timing MVP]]
7. [[Request Timing Policy Findings]]
8. [[Resource Contention MVP]]
9. [[Resource Contention Quality Experiment]]
10. [[Temperature-Aware GC Core Findings]]
11. [[FTL Metadata Recovery and Bad Block Handling]]
12. [[Durable Request Replay]]
13. [[Controller Lease and External Fencing]]

## 핵심 질문

- 평균 IOPS가 좋아 보여도 p99/p99.9 latency와 CV를 함께 보면 결론이 달라지는가?
- QD를 올렸을 때 throughput gain과 tail latency cost가 균형을 이루는가?
- block size를 키웠을 때 bandwidth gain과 outstanding data / tail latency cost를 함께 설명할 수 있는가?
- data integrity Pass와 observer Limited를 분리해서 말할 수 있는가?
- direct/buffered, WSL path, filesystem, cache, USB bridge 영향을 device-level claim과 어떻게 분리할 수 있는가?
- GC policy를 WAF 하나로 평가하지 않고 wear, GC pause, metadata cost, resource wait까지 함께 설명할 수 있는가?
- spare promotion, data recovery, mirror recovery를 구분해서 설명할 수 있는가?
- request retry와 stale controller mutation을 fail-closed protocol로 막는 경계를 설명할 수 있는가?
- black-box fio 결과와 white-box FTL/GC 모델 결과를 섞지 않고, 서로 보완적인 evidence로 말할 수 있는가?

## 포트폴리오 문장 후보

> SSD mini-lab에서는 fio 기반 black-box 측정으로 QD, p99/p99.9 latency, 반복 CV, block-size throughput knee, CRC32C data integrity, observer limitation을 분석했고, FTL-GC white-box lab에서는 GC policy가 WAF, wear, GC pause, resource contention에 미치는 영향과 bad-block recovery, request replay, controller fencing 같은 failure boundary를 simulator trace로 검증했습니다. 두 track을 분리해 실제 장치 관찰 결과와 내부 모델 기반 해석의 한계를 구분했습니다.

## 문답 / 자기 언어화

- [[SSD 문답 허브]]
- [[SSD Mini Lab 문답]]
- [[FTL-GC White-box 문답]]
- [[SSD 검증 공통 문답]]

## 아직 보강할 것

- [[FTL]] 개념 노트 생성
- [[FTL Metadata Recovery and Bad Block Handling]], [[Durable Request Replay]], [[Controller Lease and External Fencing]]를 문답 노트로 소화
- SSD 단계별 검증 테스트 방법 및 저장장치 정리 상태 점검
- [[NVMe SMART Telemetry]]를 external SSD 실측 결과와 더 직접 연결
- regression profile 실행 결과가 생기면 [[External SSD Product Validation]] 갱신
- quorum/consensus, physical clock authority, real hardware monotonic counter는 white-box lab의 범위 밖으로 유지

## 업데이트 로그

- 2026-07-28:
  - [[FTL Metadata Recovery and Bad Block Handling]], [[Durable Request Replay]], [[Controller Lease and External Fencing]]를 white-box validation branch에 추가.
  - [[SSD FTL-GC White-box Validation Lab]]을 최신 `C:\Users\nei11\venv\venv\GC` 상태에 맞춰 recovery/protocol 축까지 확장.
  - [[External SSD Block Size Sweep]], [[External SSD Data Integrity]], [[재현되지 않은 가설도 검증 결과다]]를 black-box validation branch에 추가.
  - [[External SSD Product Validation]]과 [[SSD Mini Lab 프로젝트 허브]]를 최신 `D:\ssd_lab` 상태에 맞춰 갱신.
- 2026-07-20:
  - 메인 허브를 읽을 수 있는 한국어 구조로 재정리.
  - [[External SSD Product Validation]]을 실제 QD sweep repeat=3 및 sustained write 결과 중심으로 갱신.
  - [[Resource Contention MVP]], [[Resource Contention Quality Experiment]], [[Request Timing Policy Findings]]를 white-box branch에 추가.
- 2026-07-15:
  - PARA branch 구조 테스트용으로 SSD 허브 재정리.
  - [[SSD Mini Lab 프로젝트 허브]], [[SSD FTL-GC White-box Validation Lab]]를 프로젝트 branch로 추가.
  - [[fio]], [[Queue Depth]], [[p99 latency]], [[SSD QoS]], [[Write Amplification Factor]], [[GC Pause]], [[Data Temperature]], [[PVB Metadata Model]] 개념/검증 노트 생성.
