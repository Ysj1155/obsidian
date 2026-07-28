---
title: FTL
aliases:
  - Flash Translation Layer
  - 플래시 변환 계층
tags:
  - concept
  - ssd
  - ftl
type: concept
status: seed
domain: SSD
created: 2026-07-28
updated: 2026-07-28
related_projects:
  - ftl-gc-whitebox-lab
  - ssd-mini-lab
---

# FTL

## 한 줄 결론

- FTL은 host가 보는 논리 주소 LPN을 NAND의 물리 위치 PPN으로 매핑하고, overwrite, GC, TRIM, wear leveling, bad block recovery 같은 내부 상태 전이를 관리하는 SSD controller의 핵심 계층이다.

## 핵심 역할

- host logical address를 NAND physical page/block으로 변환한다.
- overwrite를 in-place가 아니라 out-of-place write로 처리한다.
- old physical page를 invalid로 만들고, 새 physical page를 authoritative mapping으로 만든다.
- GC가 valid page를 이동하고 block을 erase할 수 있도록 mapping/reverse mapping을 유지한다.
- TRIM, wear leveling, bad block retirement, metadata recovery의 기준 상태를 제공한다.

## 내 프로젝트와 연결

- [[SSD FTL-GC White-box Validation Lab]]에서는 FTL 상태 전이를 Python simulator로 관찰한다.
- [[SSD Mini Lab 프로젝트 허브]]에서는 실제 장치의 외부 동작만 측정하므로, 내부 FTL 원인을 직접 주장하지 않는다.
- 따라서 FTL은 black-box 결과의 추정 원인이 아니라, white-box simulator에서 따로 검증하는 내부 모델로 다뤄야 한다.

## 관련 노트

- [[SSD Garbage Collection]]
- [[SSD Trim]]
- [[SSD Wear Leveling]]
- [[Write Amplification Factor]]
- [[FTL Metadata Recovery and Bad Block Handling]]
- [[Durable Request Replay]]
- [[Controller Lease and External Fencing]]
