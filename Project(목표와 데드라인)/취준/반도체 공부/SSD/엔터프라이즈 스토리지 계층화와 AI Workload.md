---
title: SNIA - Three Truths About Hard Drives and SSDs
aliases:
  - Three Truths About Hard Drives and SSDs
  - HDD와 SSD의 세 가지 진실
tags:
  - source
  - storage
  - ssd
  - hdd
  - datacenter
  - ai-workload
type: source
status: seed
domain: storage
created:
  "{ date }":
updated:
  "{ date }":
source: https://www.snia.org/blog/2024/three-truths-about-hard-drives-and-ssds
source_type: blog
reliability: mixed
related_projects: []
related_roles: []
---

# SNIA - Three Truths About Hard Drives and SSDs

## 문서 정보
- 출처: SNIA Blog
- 유형: 산업/시장 관점 블로그
- 발행/게시 시점: 2024
- 읽은 날짜: {{2026.04.28}}
- 작성 주체: 표준단체 사이트에 올라온 산업계 글. 작성자는 Seagate 소속으로 보이므로 HDD 산업 관점의 편향 가능성 있음.
- 신뢰도 1차 판단: mixed
- 읽은 이유:
  - 데이터센터에서 SSD와 HDD가 실제로 어떤 역할을 나눠 가지는지 이해하기 위해.
  - SSD가 HDD를 대체하는 흐름인지, 아니면 용도별로 공존하는 구조인지 판단하기 위해.
  - FADU SSD Validation 직무 관점에서 enterprise storage workload를 이해하기 위해.

## 이 문서의 핵심 주장
- SSD와 HDD의 $/TB 격차는 당분간 쉽게 사라지지 않는다.
- NAND 공급과 투자 규모만으로 HDD의 전체 저장 용량을 대체하기는 어렵다.
- 현대 enterprise storage는 단일 매체가 아니라 SSD, HDD, tape, hybrid storage가 workload에 따라 역할을 나누는 계층화 구조다.

## 문서가 실제로 말한 것
- 직접 확인된 사실:
  - 데이터센터 저장장치는 성능 요구와 용량 요구가 다르다.
  - SSD는 latency, IOPS, QoS가 중요한 workload에 강하다.
  - HDD는 $/TB와 대용량 저장에서 여전히 강하다.
  - 모든 데이터를 SSD에 저장하는 all-flash 구조는 비용 측면에서 일반화하기 어렵다.
- 해석이 들어간 부분:
  - SSD가 HDD를 완전히 대체하지 못하고, 둘은 역할 분담 관계로 갈 가능성이 높다.
  - AI 시대에도 HDD의 수요는 유지될 가능성이 있다.
  - SSD는 HDD 전체를 대체하기보다 hot/warm data 영역을 넓혀가며 침투할 가능성이 크다.
- 아직 추정인 부분:
  - AI workload 증가가 HDD와 SSD의 성장 비율에 각각 어느 정도 영향을 줄지.
  - QLC/PLC SSD가 warm/cold data 영역을 얼마나 잠식할지.
  - 데이터센터별 실제 SSD/HDD 사용 비율.

## 중요한 항목 정리

### 1. $/TB와 $/IOPS의 분리
- 무엇:
  - HDD와 SSD를 같은 기준으로 비교하면 판단이 흐려진다.
  - HDD는 $/TB 기준에서 강하고, SSD는 $/IOPS, latency, QoS 기준에서 강하다.
- 왜 중요:
  - “SSD가 더 좋다” 또는 “HDD가 더 싸다” 같은 단순 비교를 피할 수 있다.
  - workload 기준으로 저장매체를 판단해야 한다.
- 이 문서에서의 맥락:
  - 원문은 HDD의 $/TB 우위를 강조하며, 모든 데이터를 SSD로 대체하는 것은 비경제적이라고 본다.
- 검증/실무 포인트:
  - SSD Validation에서는 단순 peak throughput보다 p99/p999 latency, sustained workload, power, thermal throttling을 함께 봐야 한다.
  - 고객 workload가 용량 중심인지 성능 중심인지에 따라 검증 기준이 달라진다.
- 연결되는 개념 노트:
  - [[엔터프라이즈 SSD]]
  - [[SSD 단계적 검증 테스트 방법 및 저장장치]]
  - [[SSD Validation Engineer]]

### 2. Hot / Warm / Cold Data
- 무엇:
  - hot data: 자주 접근되고 latency가 중요한 데이터
  - warm data: 반복 접근 가능성이 있는 중간 데이터
  - cold data: 거의 접근하지 않는 보관/아카이브 데이터
- 왜 중요:
  - SSD와 HDD의 역할 분담은 데이터 온도에 따라 결정된다.
  - AI workload는 과거 cold data를 다시 warm/hot data로 끌어올릴 수 있다.
- 이 문서에서의 맥락:
  - 원문은 HDD가 cold/capacity tier에서 계속 중요하다고 본다.
- 검증/실무 포인트:
  - SSD cache/tier에 어떤 데이터를 올릴지 판단하는 기준이 중요하다.
  - cache miss, warm-up, cache thrashing, workload phase 변화가 성능을 무너뜨릴 수 있다.
- 연결되는 개념 노트:
  - [[엔터프라이즈 SSD]]
  - [[개발자를 위한 SSD(Coding for SSD) - Part 5 접근 방법과 시스템 최적화]]

### 3. AI Workload와 Storage Tiering
- 무엇:
  - AI workload는 데이터를 단순 저장하는 것이 아니라 재학습, fine-tuning, validation, drift 대응에 반복 사용한다.
  - 원래 cold였던 데이터가 다시 warm/hot data가 될 수 있다.
- 왜 중요:
  - AI 시대에는 데이터의 양과 접근 속도 요구가 동시에 증가한다.
  - HDD와 SSD가 둘 다 성장할 가능성이 있다.
- 이 문서에서의 맥락:
  - 원문은 HDD의 지속성을 강조하지만, AI workload가 cold data를 다시 끌어올리는 측면은 깊게 다루지 않는다.
- 검증/실무 포인트:
  - AI workload에서는 DLIO, MLPerf Storage 같은 benchmark가 의미를 가질 수 있다.
  - SSD 단품 성능뿐 아니라 실제 데이터 파이프라인에서 병목이 어디서 생기는지 봐야 한다.
- 연결되는 개념 노트:
  - [[OCP-OPF-Testing-and-Validation]]
  - [[SSD Validation Engineer]]

### 4. Cache / Tiering Policy
- 무엇:
  - SSD는 HDD를 단순 대체하기보다 cache 또는 performance tier로 동작할 수 있다.
  - LRU, LFU, ARC, TinyLFU 같은 정책은 workload에 따라 장단점이 다르다.
- 왜 중요:
  - 좋은 정책은 하나를 고르는 문제가 아니라, 어떤 workload에서 어떤 정책이 무너지는지 보는 문제다.
- 이 문서에서의 맥락:
  - 원문은 tiered architecture를 강조하지만, 구체적인 cache policy까지 깊게 들어가지는 않는다.
- 검증/실무 포인트:
  - cache hit ratio
  - cache miss 시 latency spike
  - working set > cache size일 때 성능 붕괴
  - reuse distance
  - cache thrashing
- 연결되는 개념 노트:
  - [[SSD 단계적 검증 테스트 방법 및 저장장치]]
  - [[개발자를 위한 SSD(Coding for SSD) - Part 5 접근 방법과 시스템 최적화]]

## 내 해석
- 내가 이해한 이 문서의 본질:
  - 이 글은 “SSD가 HDD를 곧 완전히 대체한다”는 서사를 견제하는 글이다.
  - 데이터센터 storage는 단일 매체 경쟁이 아니라 workload별 역할 분담 구조로 봐야 한다.
- 이 문서가 특히 강조하는 관점:
  - HDD는 낡아서 남아 있는 기술이 아니라, 대용량 저장의 경제성 때문에 여전히 필요한 매체다.
  - SSD는 성능이 필요한 영역에서 강하지만, 모든 데이터를 SSD로 저장하는 것은 비용적으로 비현실적이다.
- 다른 자료와 충돌하거나 과장돼 보이는 부분:
  - 작성자가 Seagate 소속이라는 점 때문에 HDD의 지속성을 강조하는 방향의 편향 가능성이 있다.
  - AI workload가 cold data를 다시 warm/hot data로 끌어올리는 흐름은 상대적으로 약하게 다룬다.
  - per-TB TCO는 강조하지만, p99 latency, GPU idle time, rack density, watt당 처리량 같은 성능 기준 TCO는 깊게 다루지 않는다.

## 내 프로젝트 / 직무와 연결
- SSD Validation 관점:
  - SSD Validation은 단품 SSD가 빠른지 보는 데서 끝나지 않는다.
  - 실제 enterprise workload와 storage tier 구조 안에서 성능, 전력, 지연, 무결성이 어떻게 유지되거나 무너지는지를 봐야 한다.
  - 평균 throughput보다 p99/p999 latency, sustained workload, cache miss, thermal throttling, power loss protection이 중요하다.
- Product Engineering 관점:
  - 고객에게 SSD가 왜 필요한지 설명하려면 “빠르다”가 아니라 workload 기준으로 설명해야 한다.
  - DB, metadata, AI staging, VM/container storage처럼 SSD가 필요한 영역과 archive/backup처럼 HDD가 합리적인 영역을 구분해야 한다.
- 내 GC/SSD 공부와의 연결:
  - GC, WAF, wear leveling은 SSD 내부 효율 문제다.
  - 그러나 실제 데이터센터에서는 SSD가 어떤 tier에서 어떤 workload를 받는지도 중요하다.
  - FDP나 workload-aware placement 같은 개념은 이 연결고리로 볼 수 있다.
- 자소서/면접에서 쓸 수 있는 포인트:
  - “엔터프라이즈 SSD는 단순 peak 성능보다 고객 workload에서의 예측 가능한 성능과 QoS가 중요하다고 이해했다.”
  - “데이터센터에서는 HDD와 SSD가 단순 경쟁 관계가 아니라 capacity tier와 performance tier로 역할을 나누며, validation도 이 맥락에서 workload 기반으로 설계되어야 한다.”
  - “fio 기반 SSD mini-lab을 단순 속도 측정이 아니라 workload 조건 변화에 따른 latency, power, stability 비교로 확장하고 싶다.”

## 후속 작업
- 새로 만들 개념 노트:
  - [[Hot Warm Cold Data]]
  - [[Storage Tiering]]
  - [[Cache Thrashing]]
  - [[Working Set]]
  - [[LRU와 LFU]]
  - [[AI Workload와 Storage]]
- 보강이 필요한 기존 노트:
  - [[엔터프라이즈 SSD]]
  - [[SSD 허브]]
  - [[SSD 단계적 검증 테스트 방법 및 저장장치]]
  - [[SSD Validation Engineer]]

## 남은 질문
- AI workload 증가가 실제로 HDD 수요와 SSD 수요를 각각 얼마나 늘릴까?
- QLC SSD는 warm/cold data 영역에서 HDD를 얼마나 대체할 수 있을까?
- 데이터센터에서 SSD cache/tier의 적정 크기는 어떤 방식으로 결정할까?
- Validation에서 cache thrashing이나 tiering failure를 어떻게 재현성 있게 테스트할 수 있을까?