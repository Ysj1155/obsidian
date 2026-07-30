---
title: LBA LPN PPN
aliases:
  - LBA
  - LPN
  - PPN
  - SSD 주소 변환
tags:
  - concept
  - ssd
  - ftl
  - addressing
type: concept
status: growing
domain: SSD FTL
created: 2026-07-30
updated: 2026-07-30
related_projects:
  - ssd-mini-lab
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# LBA LPN PPN

## 한 줄 정의

LBA는 host가 지정하는 논리 블록 주소, LPN은 FTL이 NAND page 크기에 맞춰 다루는 논리 페이지 번호, PPN은 데이터가 실제로 기록된 물리 페이지 번호다.

## 한눈에 보는 관계

```text
host read/write
  -> LBA 범위
  -> FTL이 LPN 단위로 해석
  -> mapping table에서 PPN 조회
  -> NAND physical page 접근
```

| 용어 | 풀네임 | 누가 주로 보는가 | 의미 |
| --- | --- | --- | --- |
| LBA | Logical Block Address | host, OS, protocol | block device의 논리 주소 |
| LPN | Logical Page Number | FTL firmware/model | NAND page 크기에 맞춘 논리 페이지 번호 |
| PPN | Physical Page Number | FTL, NAND 관리 계층 | 실제 NAND 물리 페이지를 가리키는 식별자 |

## 이 개념의 층위

- 위치: host block layer / storage protocol / [[FTL]] / NAND
- 누구의 동작인가:
  - host는 LBA로 요청한다.
  - FTL은 LBA를 내부 논리 단위로 해석해 physical location에 연결한다.
- 입력: read, write, [[TRIM]]의 logical range
- 출력: mapping lookup/update, NAND read/program 대상

## 핵심 동작 / 원리

1. Host는 파일 이름이나 NAND 주소가 아니라 LBA 범위로 I/O를 보낸다.
2. FTL은 요청 범위를 내부 page 단위인 LPN으로 변환한다.
3. Mapping table은 각 LPN의 최신 데이터가 있는 PPN을 가리킨다.
4. 같은 LPN에 overwrite가 오면 보통 새 PPN에 기록한 뒤 mapping을 바꾼다.
5. 이전 PPN은 invalid가 되고 이후 [[SSD Garbage Collection]]의 회수 대상이 된다.

예를 들어 logical block이 4 KiB이고 NAND page가 16 KiB라면, 한 LPN에 네 개 LBA가 대응할 수 있다. 실제 단위와 매핑 방식은 제품과 모델에 따라 다르므로 이 비율을 일반화하면 안 된다.

## 대표 시나리오

```text
초기 상태: mapping[LPN 10] = PPN 100

LPN 10 overwrite
  -> 새 PPN 245에 data program
  -> mapping[LPN 10] = PPN 245
  -> PPN 100은 invalid
```

- Host 관점: 같은 logical address를 덮어썼다.
- NAND 관점: 다른 physical page에 새로 썼다.
- FTL 관점: authoritative mapping이 바뀌었다.

## 무엇과 헷갈리기 쉬운가

- LBA와 LPN은 항상 1:1이 아니다. logical block size와 FTL/NAND page 단위가 다를 수 있다.
- PPN은 host가 직접 지정하는 주소가 아니다.
- PPN은 학습용 모델에서 단순한 정수로 표현할 수 있지만, 실제 장치에서는 channel, die, plane, block, page 같은 물리 구조와 연결된다.
- 모든 SSD가 단순한 page-level mapping table 하나만 사용하는 것은 아니다. 실제 FTL 구조는 vendor-specific이다.

## 검증 / 실무 관점

- Black-box 검증에서는 LBA 범위와 workload를 통제할 수 있지만 LPN→PPN mapping은 보통 직접 볼 수 없다.
- White-box simulator에서는 mapping 변화, old PPN invalidation, free page 감소를 관찰할 수 있다.
- Mapping update와 data program의 순서가 power loss에도 복구 가능해야 한다.
- 잘못된 mapping, stale read, double allocation은 데이터 무결성 결함으로 이어진다.

## 내 프로젝트와 연결

- [[SSD Mini Lab 프로젝트 허브]]: fio가 특정 파일이나 장치 경로에 I/O를 보내도 관찰 가능한 것은 host-side logical I/O와 결과다.
- [[SSD FTL-GC White-box Validation Lab]]: LPN→PPN mapping과 invalidation을 직접 추적한다.
- [[PVB Metadata Model]]: physical page/block 상태를 관리하는 metadata 관점으로 이어진다.

## 면접용 문장

LBA는 host가 보는 논리 블록 주소이고, FTL은 이를 내부 LPN 단위로 해석해 최신 데이터가 있는 PPN으로 매핑합니다. Overwrite 때는 보통 새 physical page에 기록하고 mapping을 갱신하므로 old page가 invalid가 되며, 이것이 GC와 WAF가 필요한 출발점입니다.

## 관련 노트

- [[SSD]]
- [[FTL]]
- [[TRIM]]
- [[SSD Garbage Collection]]
- [[Write Amplification Factor]]
- [[FTL Metadata Recovery and Bad Block Handling]]
