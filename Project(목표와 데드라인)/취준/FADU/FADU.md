---
title: FADU 준비 허브
aliases:
  - FADU
  - 파두
  - 파두 준비 허브
tags:
  - hub
  - company
  - ssd
  - career
type: hub
status: growing
domain: FADU Career Preparation
created: 2026-08-04
updated: 2026-08-04
source_type: mixed-company-material
reliability: mixed-source
related_projects:
  - ssd-mini-lab
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - product-engineering
  - firmware-validation
  - test-engineering
---

# FADU 준비 허브

## 이 허브의 역할

FADU의 SSD 제품·controller 관련 공개 자료와 SSD Validation 직무 준비 자료를 연결한다. 회사 블로그와 brochure는 vendor source이므로 기술 배경을 배우는 자료로 사용하되, 성능·효율 우위 표현은 독립 검증 결과와 구분한다.

- 상위 허브: [[취준]]
- 기술 허브: [[SSD 허브]]
- 외부 자료 인덱스: [[SSD References Index]]

## 먼저 읽을 것

1. [[SSD Validation Engineer(성능 검증)]]
2. [[ssd_validation_24week_roadmap|SSD Validation Engineer 24주 로드맵]]
3. [[파두 블로그 3]]
4. [[파두 블로그 1]]
5. [[파두 블로그 2]]

## 회사 자료 묶음

### Controller와 architecture

- [[파두 블로그 3]]: controller architecture와 control/data path 관련 vendor 설명
- [[Fadu PCIe 3.0 SSD 브로셔]]
- [[Fadu PCIe 4.0 SSD 브로셔]]

### FDP·WAF·workload

- [[파두 블로그 1]]: FDP와 data placement, WAF 관련 vendor 설명
- [[파두 블로그 2]]: hyperscale workload와 enterprise SSD 동향
- [[Write Amplification Factor]]
- [[Data Temperature]]

### 직무 준비

- [[SSD Validation Engineer(성능 검증)]]
- [[ssd_validation_24week_roadmap|SSD Validation Engineer 24주 로드맵]]
- [[공부할 리스트]]
- [[면접 대비]]

## 현재 프로젝트와 연결

- [[SSD Mini Lab 프로젝트 허브]]: 공개 스펙을 반복 가능한 black-box test condition으로 바꾸는 연습
- [[SSD FTL-GC White-box Validation Lab]]: controller 내부 정책과 failure boundary를 bounded model로 공부
- [[00 SSD Controller and Verification 학습 지도]]: RTL·SoC 교육 내용을 SSD controller validation에 연결

## 읽을 때 주의할 점

- Vendor 자료의 `up to`, `industry leading`, 세대 비교는 test condition을 확인한다.
- 공개 자료로 알 수 없는 내부 scheduler, FTL policy, NAND configuration을 단정하지 않는다.
- 회사 자료에서 얻은 아이디어와 내 프로젝트의 실제 결과를 분리한다.
- 제품·채용 정보는 지원 시점의 공식 홈페이지와 공고로 다시 확인한다.

## 면접으로 바꿀 질문

- SSD Validation에서 성능 숫자뿐 아니라 재현성과 tail latency를 왜 보는가?
- Enterprise workload와 client workload의 요구 차이를 어떻게 설명할 것인가?
- 실제 장치 black-box 관찰과 controller white-box model을 왜 분리했는가?
- Vendor claim을 검증 항목으로 바꿀 때 어떤 조건과 telemetry가 필요한가?

## 관련 노트

- [[취준]]
- [[SSD 허브]]
- [[SSD References Index]]
- [[엔터프라이즈 SSD]]
- [[SSD Mini Lab Portfolio Evidence]]
