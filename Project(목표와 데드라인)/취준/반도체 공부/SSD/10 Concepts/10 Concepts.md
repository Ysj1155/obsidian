---
title: SSD 10 Concepts Index
tags:
  - hub
  - index
  - ssd
  - concepts
type: index
status: seed
domain: SSD
created: 2026-07-30
updated: 2026-07-30
---

# 10 Concepts

## 목적

이 폴더는 SSD 공부에서 반복해서 등장하는 개념을 모아두는 지식 베이스다. 프로젝트 결과를 읽기 전에 여기서 용어와 관계를 잡고, 프로젝트 노트에서는 실제 실험/관찰로 들어간다.

## 먼저 읽을 기본 흐름

1. [[SSD]]
2. [[SSD 기본 원리와 구조]]
3. [[SSD 인터페이스]]
4. [[SSD 통신 프로토콜]]
5. [[FTL]]
6. [[TRIM]]
7. [[SSD Garbage Collection]]
8. [[Write Amplification Factor]]
9. [[SSD QoS]]
10. [[p99 latency]]

## Black-box 검증 흐름

- [[fio]]
- [[Queue Depth]]
- [[p99 latency]]
- [[SSD QoS]]
- [[NVMe SMART Telemetry]]

## White-box FTL/GC 흐름

- [[FTL]]
- [[SSD Garbage Collection]]
- [[TRIM]]
- [[SSD Wear Leveling]]
- [[Write Amplification Factor]]
- [[GC Pause]]
- [[Data Temperature]]
- [[PVB Metadata Model]]

## 제품/세그먼트 흐름

- [[클라이언트 SSD]]
- [[엔터프라이즈 SSD]]
- [[SSD 전력 손실 보호 원리]]

## 읽을 때 주의할 점

- 외부 benchmark 결과만 보고 내부 FTL/GC 원인을 단정하지 않는다.
- 브로셔 claim은 측정 조건, workload, QD, thermal condition을 확인하기 전까지 claim으로 남긴다.
- 평균 IOPS, p99/p99.9 latency, CV, telemetry를 같이 본다.
- 같은 단어라도 black-box 실험과 white-box simulator에서 의미가 달라질 수 있다.
