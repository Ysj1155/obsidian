---
title: SSD References Index
aliases:
  - SSD 참고 자료 허브
  - SSD Reference Hub
tags:
  - hub
  - index
  - ssd
  - reference
type: index
status: growing
domain: SSD
created: 2026-08-04
updated: 2026-08-04
source_type: source-index
reliability: mixed-source
related_projects:
  - ssd-mini-lab
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - product-engineering
  - firmware-validation
---

# SSD References Index

## 이 인덱스의 역할

외부 문서, vendor brochure, 발표 요약, 기술 글을 한곳에 모은다. Reference 노트는 근거와 읽기 기록이며, 최종 개념과 프로젝트 결론은 [[10 Concepts]] 및 각 프로젝트 노트에 둔다.

- 상위 허브: [[SSD 허브]]
- 개념 노트: [[10 Concepts]]
- 실제 장치 검증: [[SSD Mini Lab 프로젝트 허브]]
- 내부 모델 검증: [[SSD FTL-GC White-box Validation Lab]]

## 표준·검증 프레임워크

- [[OCP-OPF-Testing-and-Validation]]
  - 공개 repository에서 확인한 범위와 확인되지 않은 framework claim을 분리한다.
- [[Open Source SSD Validation in Practice Updates from Meta's Qualification Framework]]
  - 발표 요약과 공개 repository evidence가 같은 범위인지 주의해서 읽는다.

## Vendor 자료

- [[Fadu PCIe 3.0 SSD 브로셔]]
- [[Fadu PCIe 4.0 SSD 브로셔]]

브로셔의 `industry leading`, `up to`, 세대 비교 수치는 vendor claim이다. Test condition과 독립 측정 근거를 확인하기 전까지 제품 검증 결과로 사용하지 않는다.

## SSD 구조와 프로그래밍 관점

1. [[개발자를 위한 SSD(Coding for SSD) - Part 2 SSD의 아키텍쳐와 벤치마킹]]
2. [[개발자를 위한 SSD(Coding for SSD) - Part 3 페이지 & 블록 & FTL(Flash Translation Layer)]]
3. [[개발자를 위한 SSD(Coding for SSD) - Part 4 고급 기능과 내부 병렬 처리]]
4. [[개발자를 위한 SSD(Coding for SSD) - Part 5 접근 방법과 시스템 최적화]]
5. [[개발자를 위한 SSD(Coding for SSD) - Part 6 A Summary - What every programmer should know about solid-state drives]]

이 시리즈는 SSD 구조를 연결해서 읽는 보조 자료다. 오래된 기술 환경이나 단순화된 설명이 현재 NVMe SSD 전체에 그대로 적용된다고 가정하지 않는다.

## 데이터센터와 Workload

- [[AWS와 Oracle 공개 자료로 배우는 데이터센터 스토리지]]
- [[엔터프라이즈 스토리지 계층화와 AI Workload]]

제품·서비스 문서는 workload requirement와 storage 계층을 이해하는 데 사용하고, 특정 SSD의 내부 구현 증거로 사용하지 않는다.

## Reference를 개념으로 바꾸는 규칙

1. 원문이 직접 말한 사실과 내가 추론한 내용을 분리한다.
2. 핵심 용어는 [[10 Concepts]]의 canonical note에 연결한다.
3. Vendor claim에는 측정 조건과 한계를 붙인다.
4. 프로젝트 결과와 연결할 때 실제 관찰 범위를 넘지 않는다.
5. 읽기만 한 자료는 reference에 두고, 내 말로 설명 가능한 내용만 개념·검증 노트로 승격한다.

## 개념 연결

- 구조: [[SSD]], [[FTL]], [[SSD Garbage Collection]]
- 성능: [[Storage Performance Metrics]], [[Queue Depth]], [[SSD QoS]]
- 제품: [[클라이언트 SSD]], [[엔터프라이즈 SSD]]
- 관찰: [[SSD Telemetry]], [[NVMe SMART Telemetry]]
- Controller: [[00 SSD Controller and Verification 학습 지도]]

## 관련 노트

- [[SSD 허브]]
- [[FADU]]
- [[External SSD Product Validation]]
- [[SSD Mini Lab Portfolio Evidence]]
