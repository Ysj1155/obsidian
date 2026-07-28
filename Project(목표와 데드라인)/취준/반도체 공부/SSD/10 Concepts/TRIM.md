---
title: TRIM
aliases:
  - SSD TRIM
  - discard
  - deallocate
tags:
  - concept
  - ssd
  - ftl
  - trim
  - gc
type: concept
status: seed
domain: SSD FTL
created: 2026-07-28
updated: 2026-07-28
related_projects:
  - ftl-gc-whitebox-lab
  - ssd-mini-lab
---

# TRIM

## 한 줄 정의

TRIM은 host가 SSD에게 “이 logical block은 더 이상 유효한 데이터가 아니다”라고 알려주는 invalidation hint다.

## 왜 필요한가

파일을 삭제해도 SSD는 그 순간 NAND 안의 어떤 physical page가 버려져도 되는지 자동으로 알 수 없다. OS와 파일시스템 입장에서는 파일이 지워졌지만, SSD 내부 FTL 입장에서는 기존 LPN mapping이 아직 valid data처럼 보일 수 있다.

TRIM이 전달되면 FTL은 해당 logical block을 invalid로 표시할 수 있고, 이후 [[SSD Garbage Collection]]이 valid page migration을 줄일 수 있다. 이 때문에 TRIM은 [[Write Amplification Factor]], sustained write 성능, 장기 [[SSD QoS]]와 연결된다.

## 머릿속 모델

1. 사용자가 파일을 삭제하거나 filesystem이 block을 해제한다.
2. OS/filesystem이 SSD에 TRIM, discard, deallocate 계열 명령을 보낸다.
3. SSD의 [[FTL]]은 해당 LPN이 더 이상 필요 없다는 정보를 받는다.
4. FTL은 mapping이나 page state를 invalid로 처리할 수 있다.
5. 나중에 GC가 block을 고를 때, TRIM된 page는 옮기지 않아도 되는 후보가 된다.

## 반드시 구분할 것

- TRIM은 삭제 명령 자체가 아니라 host-side hint다.
- TRIM은 즉시 NAND erase를 보장하지 않는다.
- TRIM은 secure erase가 아니다.
- TRIM이 성능을 항상 즉시 개선한다고 단정하면 안 된다.
- TRIM 효과는 OS, filesystem, interface, drive firmware, workload, free space, OP 조건에 따라 달라진다.

## GC와의 관계

TRIM은 “이 LBA는 더 이상 필요 없다”는 정보를 준다. GC는 그 정보를 이용해 실제 block erase와 free block 회수를 수행한다.

즉, TRIM은 invalid data를 더 잘 알려주는 신호이고, GC는 그 신호를 바탕으로 physical space를 회수하는 내부 작업이다.

## 검증 질문

- TRIM이 실제로 drive까지 전달되는가?
- TRIM 이후 random/sustained write 성능이 달라지는가?
- TRIM된 logical range가 GC migration 대상에서 제외되는가?
- TRIM locality가 reclaim 효율에 영향을 주는가?
- TRIM이 늦게 반영될 때 WAF나 tail latency가 나빠지는가?

## 내 프로젝트와 연결

- 자세한 검증 노트: [[SSD Trim]]
- 내부 모델 관찰: [[SSD FTL-GC White-box Validation Lab]]
- 연결 개념: [[FTL]], [[SSD Garbage Collection]], [[Write Amplification Factor]], [[SSD QoS]]

## 면접용 문장

TRIM은 파일 삭제와 SSD 내부 GC 사이를 연결하는 host-side invalidation hint입니다. TRIM이 잘 전달되면 FTL이 불필요한 valid migration을 줄일 수 있어 WAF와 sustained write 안정성에 도움이 될 수 있지만, 실제 효과는 OS, filesystem, firmware, workload 조건에 따라 검증해야 합니다.

## 더 읽을 순서

1. [[FTL]]
2. [[SSD Garbage Collection]]
3. [[Write Amplification Factor]]
4. [[SSD Trim]]
5. [[Temperature-Aware GC Core Findings]]
