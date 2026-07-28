---
title: FTL Metadata Recovery and Bad Block Handling
aliases:
  - Bad Block Recovery MVP
  - PBBT Recovery
  - VALID Data Failure Recovery
  - Page Mirroring Recovery
tags:
  - experiment
  - validation
  - ssd
  - ftl
  - recovery
  - bad-block
  - portfolio
type: experiment-report
status: completed
domain: SSD FTL Validation
created: 2026-07-28
updated: 2026-07-28
source: C:\Users\nei11\venv\venv\GC\docs\persistent_bad_block_recovery_mvp.md
source_type: local-report
reliability: deterministic-simulator
related_projects:
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# FTL Metadata Recovery and Bad Block Handling

## 한 줄 결론

- 이 축은 runtime erase failure, bad-block metadata, sudden VALID-data failure, page mirroring을 bounded simulator에서 검증한 recovery chain이다.
- 핵심 교훈은 spare promotion은 용량을 복구할 뿐이고, 사용자 데이터는 surviving copy나 durable relocation evidence가 있을 때만 복구된다는 점이다.

## 검증 질문

- erase failure나 bad block retirement가 controller restart 뒤에도 안전하게 보존되는가.
- bad-block table, NAND page state, forward/reverse mapping, relocation marker가 commit boundary에 따라 rollback/roll-forward 되는가.
- VALID data를 가진 block이 갑자기 실패했을 때 어떤 LPN은 복구되고 어떤 LPN은 명시적으로 lost 처리되어야 하는가.
- mirror copy를 두면 recoverability가 얼마나 좋아지고, WAF/capacity cost는 어떻게 드러나는가.

## 포함한 MVP

| MVP | 원본 문서 | 결과 | 핵심 의미 |
|---|---|---|---|
| Persistent Bad-Block Recovery | `persistent_bad_block_recovery_mvp.md` | 6/6 PASS | PBBT generation, retired set, promoted spare가 restart 후 보존됨 |
| Two-copy PBBT Torn Write | `pbbt_two_copy_torn_write_mvp.md` | 8/8 PASS | A/B generation/checksum/commit flag로 torn metadata selection 검증 |
| Sudden VALID-data Failure | `valid_data_failure_recovery_mvp.md` | 6/6 PASS | spare는 capacity만 복구하고, data는 surviving evidence가 있어야 복구됨 |
| Bounded Page Mirroring | `page_mirroring_mvp.md` | 6/6 PASS | mirror copy가 recoverability를 높이지만 write/capacity cost를 만든다 |
| Primary+Mirror Atomicity | `mirror_write_atomicity_mvp.md` | 6/6 PASS | mirror PREPARE/COMMIT cut에서 old/old 또는 new/new convergence 검증 |

## Persistent Bad-Block Recovery

`SSD.retire_block()`의 runtime path는 다음 순서로 단순화됐다.

```text
select exact replacement spare
  -> persist PBBT intent
  -> commit next full PBBT generation
  -> apply RETIRED health and exact spare promotion
  -> clear pending transition
```

Power-cut checkpoint별 recovery rule:

| Cut point | Durable state | Recovery |
|---|---|---|
| after PBBT prepare | uncommitted intent + physical failure | media scan commits and rolls forward retirement/promotion |
| after PBBT commit | committed table, old controller health | committed generation replay |
| after health apply | committed table and new health, pending not cleared | idempotent replay and clear pending |

핵심은 uncommitted software intent와 irreversible physical failure를 구분하는 것이다.

## Sudden VALID-data Failure

VALID page를 가진 block이 갑자기 실패하면 affected LPN이 모두 복구되는 것이 아니다.

| Case | Affected | Recovered | Unrecoverable | 의미 |
|---|---:|---:|---:|---|
| single-copy baseline | 4 | 0 | 4 | 단일 copy는 손실을 명시해야 함 |
| programmed uncommitted destination | 4 | 1 | 3 | surviving destination + marker가 있으면 salvage |
| committed destination | 4 | 1 | 3 | committed relocation destination 사용 |
| failed destination source fallback | 1 | 1 | 0 | surviving old source로 fallback |
| one shadow partial recovery | 4 | 1 | 3 | 일부 LPN만 복구 가능 |
| no replacement spare | 4 | 0 | 4 | data loss와 capacity loss 분리 |

해석 기준:

- spare promotion은 failed block을 대체할 capacity를 제공한다.
- spare promotion만으로 lost payload가 복구되지는 않는다.
- durable relocation marker와 surviving programmed page가 있어야 복구 evidence가 된다.
- evidence가 없으면 mapping을 failed media에 남기지 않고 `unrecoverable`로 명시한다.

## Page Mirroring

Bounded page mirror model은 protected LPN에 second copy를 둔다.

| Case | Recovered / affected | Mirror writes | Write amplification |
|---|---:|---:|---:|
| no mirror | 0 / 4 | 0 | 1.000 |
| selective mirror | 2 / 4 | 2 | 1.250 |
| full mirror | 4 / 4 | 8 | 2.000 |
| primary + mirror-domain failure | 0 / 4 | 8 | 2.000 |
| exhausted mirror pool | 0 / 1 | 8 | 1.889 |

- full mirroring은 single-domain failure를 단순하게 복구하지만 protected write마다 추가 program cost가 든다.
- selective mirroring은 recoverability와 WAF 사이의 직접적인 knob가 된다.
- mirror domain 자체도 실패하면 복구 evidence가 사라진다.

## 해석

- 결과가 보여주는 것:
  - metadata recovery와 data recovery는 다르다.
  - PBBT/spare는 device geometry와 capacity safety를 복구한다.
  - 사용자 data는 marker-backed surviving copy나 mirror가 있어야 복구할 수 있다.
- 결과가 보여주지 못하는 것:
  - 실제 NAND payload byte, ECC, retention, read retry, physical program atomicity를 증명하지 않는다.
  - 실제 SSD의 RAID/parity/firmware recovery와 동일하다고 말할 수 없다.
- 가장 중요한 trade-off:
  - recovery guarantee를 높이려면 write amplification, capacity overhead, mirror/persistence ordering complexity가 증가한다.

## 포트폴리오 / 면접 포인트

> FTL simulator에서 bad-block retirement와 restart recovery를 모델링하고, PBBT generation, promoted spare, relocation marker, page mirroring이 어떤 failure boundary에서 rollback 또는 roll-forward되는지 deterministic probe로 검증했습니다. 특히 spare promotion은 capacity recovery이지 data recovery가 아니라는 점을 분리했고, mirror copy를 추가할 때 recoverability와 WAF가 어떻게 교환되는지 정리했습니다.

## 관련 노트

- [[SSD FTL-GC White-box Validation Lab]]
- [[SSD Garbage Collection]]
- [[Write Amplification Factor]]
- [[SSD Wear Leveling]]
- [[Durable Request Replay]]
- [[Controller Lease and External Fencing]]
