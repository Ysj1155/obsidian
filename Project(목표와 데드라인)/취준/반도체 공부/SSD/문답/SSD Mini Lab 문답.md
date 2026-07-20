---
title: SSD Mini Lab 문답
aliases:
  - SSD Mini Lab Q&A
  - fio 기반 SSD 검증 문답
tags:
  - qna
  - study
  - ssd
  - ssd-validation
  - fio
type: qna
status: seed
domain: SSD Validation
created: 2026-07-20
updated: 2026-07-20
related_projects:
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - product-engineering
  - test-engineering
---

# SSD Mini Lab 문답

## 이 문답의 목적

- `D:\ssd_lab`에서 진행한 black-box SSD 검증 프로젝트를 내 언어로 설명한다.
- fio, QD, p99/p99.9, CV, direct/buffered, WSL path, telemetry, 외장 SSD 검증 계획을 질문으로 풀어본다.
- 연결되는 프로젝트:
  - [[SSD Mini Lab 프로젝트 허브]]

## 핵심 질문

### Q1. 이 프로젝트는 정확히 무엇을 검증하는 프로젝트인가?

내 답변: 내 기억 상으로는 1TB 외장 SSD를 사용하는 프로젝트로, 가상 시나리오를 작성해서 random R/W등 여러 조건과 스펙이 맞는지 반복 실험을 통해 신뢰성 판단하고 지표 정리하는 프로젝트. 검증 과정에서 남는 데이터들이 무엇이고 그 데이터들을 통해 무엇을 배울 수 있는지 공부가 아직 필요하다 생각한다.  


정리된 답변:


연결 노트:
- [[SSD Mini Lab 프로젝트 허브]]
- [[SSD Mini Lab Portfolio Evidence]]

### Q2. fio를 단순 벤치마크가 아니라 검증 도구로 쓰려면 무엇을 남겨야 하는가?

내 답변: fio가 뭐


정리된 답변:


연결 노트:
- [[fio]]
- [[SSD Mini Lab Portfolio Evidence]]

### Q3. Queue Depth를 높이면 왜 IOPS와 p99 latency가 동시에 변할 수 있는가?

내 답변:


정리된 답변:


연결 노트:
- [[Queue Depth]]
- [[p99 latency]]
- [[왜 평균 IOPS만 보면 안 되는가]]

### Q4. 왜 평균 IOPS만 보면 안 되는가?

내 답변:


정리된 답변:


연결 노트:
- [[왜 평균 IOPS만 보면 안 되는가]]
- [[SSD QoS]]

### Q5. direct=0 결과가 좋아 보여도 왜 조심해야 하는가?

내 답변:


정리된 답변:


연결 노트:
- [[fio]]
- [[SSD QoS]]

### Q6. WSL path 비교 결과는 어떤 해석 경계를 가져야 하는가?

내 답변:


정리된 답변:


연결 노트:
- [[SSD Mini Lab Portfolio Evidence]]
- [[p99 latency]]

### Q7. 외장 SSD를 DUT로 검증할 때 먼저 확인해야 할 것은 무엇인가?

내 답변:


정리된 답변:


연결 노트:
- [[External SSD Product Validation]]
- [[NVMe SMART Telemetry]]

### Q8. telemetry가 없으면 어떤 주장을 조심해야 하는가?

내 답변:


정리된 답변:


연결 노트:
- [[NVMe SMART Telemetry]]
- [[SSD QoS]]

## 아직 헷갈리는 것

-
-

## 면접식으로 바꾼 답변

>

## 다음 질문 후보

-
-
