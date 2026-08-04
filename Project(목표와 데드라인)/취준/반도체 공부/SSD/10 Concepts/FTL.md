---
title: FTL
aliases:
  - Flash Translation Layer
  - 플래시 변환 계층
  - Flash Translation Table
tags:
  - concept
  - ssd
  - ftl
type: concept
status: growing
domain: SSD FTL
created: 2026-07-28
updated: 2026-07-30
related_projects:
  - ftl-gc-whitebox-lab
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# FTL

## 한 줄 정의

FTL은 host가 보는 logical address를 NAND의 physical location으로 매핑하고, overwrite, [[TRIM]], [[SSD Garbage Collection]], [[SSD Wear Leveling]], bad block, metadata recovery 같은 내부 상태 전이를 관리하는 SSD controller의 핵심 firmware layer다.

## 이 개념의 층위

- 위치: SSD controller firmware / NAND management layer
- 누구의 동작인가: device 내부 동작. host는 FTL을 직접 보지 못한다.
- 입력: host read/write/trim/flush command, NAND 상태, free block pressure, error 정보
- 출력: mapping update, physical page program/read, invalidation, GC migration, erase, telemetry/event

## 왜 필요한가

Host는 SSD를 연속적인 LBA 공간처럼 본다. 하지만 NAND flash는 다음 제약을 가진다.

- page는 program 단위다.
- 기존 programmed page는 바로 overwrite할 수 없다.
- erase는 page가 아니라 block 단위다.
- block마다 P/E cycle과 error 상태가 다르다.
- bad block이 생길 수 있고, valid data를 보존해야 한다.

FTL은 이 차이를 숨겨서 host에게는 block device처럼 보이게 만든다. 이 추상화 덕분에 OS와 application은 NAND 내부 page/block 제약을 직접 관리하지 않아도 된다.

## 핵심 역할

1. Address translation
   - host LBA/LPN을 NAND PPN으로 변환한다.
2. Out-of-place update
   - overwrite 요청을 기존 위치 덮어쓰기가 아니라 새 free page write로 처리한다.
3. Invalidation
   - 새 write가 authoritative mapping이 되면 old physical page를 invalid로 표시한다.
4. Garbage collection 지원
   - victim block의 valid page를 옮기고 mapping/reverse mapping을 갱신한다.
5. Wear leveling
   - 특정 block만 과하게 erase되지 않도록 erase count 분포를 관리한다.
6. TRIM 반영
   - host가 더 이상 쓰지 않는 logical range를 invalidation hint로 받아 GC 효율을 높일 수 있다.
7. Bad block / error handling
   - NAND error나 bad block을 감지하고 더 이상 쓰지 않도록 retire/remap할 수 있다.
8. Metadata durability
   - mapping table과 block state가 power loss 이후에도 복구 가능하도록 관리한다.

## 기본 상태 전이

```text
host write LPN X
  -> free PPN Y 선택
  -> NAND program Y
  -> mapping[X] = Y
  -> old PPN invalid 처리
  -> 필요하면 GC / wear leveling / metadata persist
```

```text
host read LPN X
  -> mapping[X] 조회
  -> PPN Y read
  -> ECC / data path check
  -> host에 data 반환
```

```text
host TRIM LPN X
  -> mapping[X]가 있으면 invalid 처리 후보
  -> 이후 GC에서 valid migration 대상에서 제외 가능
```

## 매핑 방식의 큰 분류

| 방식 | 아이디어 | 장점 | 비용/한계 |
| --- | --- | --- | --- |
| page-level mapping | LPN 단위로 PPN을 관리 | random write와 fine-grained update에 유리 | mapping table이 큼 |
| block-level mapping | 큰 block 단위로 mapping | metadata가 작음 | random update에 불리 |
| hybrid/log-block mapping | block mapping에 log/update block을 섞음 | metadata와 성능 사이 절충 | 구현 복잡도와 merge 비용 |

현대 SSD의 실제 FTL은 제품별로 복잡하고 vendor-specific이다. 따라서 외부 fio 결과만 보고 특정 mapping 방식을 단정하면 안 된다.

## GC와의 관계

[[SSD Garbage Collection]]은 FTL의 mapping 정보를 바탕으로 valid page와 invalid page를 구분한다. victim block을 erase하려면 valid page를 다른 free page로 옮기고, mapping을 새 위치로 바꿔야 한다.

그래서 GC는 단순 청소가 아니라 FTL metadata update를 동반하는 상태 전이다. 이때 valid page migration 때문에 [[Write Amplification Factor]]가 증가하고, foreground request와 겹치면 [[GC Pause]]나 tail latency가 커질 수 있다.

## TRIM과의 관계

[[TRIM]]은 host가 더 이상 필요 없는 logical range를 알려주는 hint다. FTL이 이 정보를 잘 반영하면 GC가 불필요한 valid page migration을 줄일 수 있다.

하지만 TRIM은 즉시 NAND erase를 보장하지 않는다. FTL이 언제, 어떻게 반영할지는 OS, filesystem, interface, firmware policy, workload에 따라 달라진다.

## Wear leveling과의 관계

[[SSD Wear Leveling]]은 block erase count가 한쪽으로 몰리지 않도록 관리한다. GC는 invalid page가 많은 block을 고르고 싶어 하지만, wear leveling은 erase count가 낮거나 높은 block을 고려해야 할 수 있다.

따라서 좋은 FTL policy는 WAF 하나만 낮추는 것이 아니라 WAF, wear spread, latency, metadata cost, recovery safety를 함께 본다.

## 검증 / 실무 관점

- black-box SSD에서 직접 볼 수 있는 것:
  - IOPS, bandwidth, latency, p99/p99.9, SMART/telemetry 일부
  - sustained workload에서의 성능 변화
  - TRIM 전후 성능 변화
- black-box에서 단정하면 안 되는 것:
  - 내부 mapping 방식
  - GC victim policy
  - SLC cache size와 정확한 folding 시점
  - thermal throttling, GC, firmware scheduling 중 무엇이 원인인지
- white-box model에서 관찰할 수 있는 것:
  - mapping table 변화
  - invalid page 분포
  - GC victim selection
  - WAF, wear_std, reclaim lag, metadata recovery

## 내 프로젝트와 연결

- [[SSD FTL-GC White-box Validation Lab]]에서는 FTL 상태 전이를 Python simulator로 관찰한다.
- [[SSD Mini Lab 프로젝트 허브]]에서는 실제 장치의 외부 동작만 측정하므로, 내부 FTL 원인을 직접 주장하지 않는다.
- 이 둘을 연결할 때는 “black-box에서 현상을 관찰하고, white-box model로 가능한 내부 메커니즘을 공부한다”로 표현하는 것이 안전하다.

## 면접용 문장

FTL은 host가 보는 LBA 공간과 NAND의 page/block 제약 사이를 이어주는 SSD firmware layer입니다. Host overwrite를 out-of-place write로 처리하고 mapping을 갱신하며, TRIM, GC, wear leveling, bad block handling을 통해 성능과 내구성, 복구 가능성을 관리합니다. 다만 실제 제품의 FTL 정책은 vendor-specific이므로 외부 benchmark만으로 내부 원인을 단정하지 않도록 조심해야 합니다.

## 관련 노트

- [[SSD]]
- [[LBA LPN PPN]]
- [[SSD 기본 원리와 구조]]
- [[TRIM]]
- [[SSD Garbage Collection]]
- [[SSD Wear Leveling]]
- [[Write Amplification Factor]]
- [[GC Pause]]
- [[PVB Metadata Model]]
- [[FTL Metadata Recovery and Bad Block Handling]]
- [[Durable Request Replay]]
- [[Controller Lease and External Fencing]]

