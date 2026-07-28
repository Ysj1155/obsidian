---
title: Durable Request Replay
aliases:
  - Request Replay MVP
  - Idempotent Replay
  - Durable Request Ledger
  - Selective ACK Recovery
tags:
  - experiment
  - validation
  - ssd
  - ftl
  - request-durability
  - recovery
type: experiment-report
status: completed
domain: SSD FTL Validation
created: 2026-07-28
updated: 2026-07-28
source: C:\Users\nei11\venv\venv\GC\docs\request_replay_mvp.md
source_type: local-report
reliability: deterministic-simulator
related_projects:
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# Durable Request Replay

## 한 줄 결론

- Durable Request Replay는 reset 이후 host retry가 같은 write를 다시 실행해 중복 storage effect를 만들지 않도록 stable request ID, replay ledger, checkpoint floor, selective ACK를 검증한 request durability 축이다.

## 검증 질문

- write는 commit됐지만 completion이 lost된 경우, 같은 host request retry를 다시 page allocation/mapping update 없이 성공 처리할 수 있는가.
- 오래된 replay ledger entry를 지울 때 delayed retry가 다시 실행되지 않도록 durable floor를 먼저 commit할 수 있는가.
- out-of-order completion에서도 selective tombstone과 bitmap-to-floor collapse로 안전하게 ledger를 압축할 수 있는가.
- session rollover와 fence가 old session의 write/ACK mutation을 막는가.

## 포함한 MVP

| MVP | 원본 문서 | 결과 | 핵심 의미 |
|---|---|---|---|
| Durable Request Identity and Replay | `request_replay_mvp.md` | 6/6 PASS | committed identity는 retry 시 storage effect를 반복하지 않음 |
| Safe Ledger Eviction | `request_checkpoint_eviction_mvp.md` | 6/6 PASS | floor commit 뒤 entry deletion, retry below floor는 fail closed |
| Out-of-order Selective ACK | `out_of_order_selective_ack_mvp.md` | 6/6 PASS | gap retention, selective tombstone, bitmap-to-floor collapse |
| Safe Session Rollover | `request_session_rollover_mvp.md` | 6/6 PASS | durable fence가 old session mutation을 차단 |
| SACK Concurrency/Fencing | `request_selective_ack_concurrency_mvp.md` | 6/6 PASS | generation CAS로 stale writer를 거부 |
| Threaded Adapter | `request_protocol_threaded_adapter_mvp.md` | 6/6 PASS | real-thread adapter가 protocol mutation을 lock boundary 안에 둠 |

## Request Replay Model

`ssd_gc_lab/request_replay.py`의 bounded ledger는 request ID마다 다음 evidence를 보존한다.

- request ID
- operation and LPN fingerprint
- `PREPARED`, `COMMITTED`, `ROLLED_BACK` state
- intended primary destination PPN
- committed mirror version, when protected
- completion send attempt 여부

Protected write의 simplified ordering:

```text
request PREPARE
  -> primary destination program
  -> mirror PREPARE
  -> primary mapping commit
  -> mirror COMMIT
  -> request COMMIT
  -> completion send
```

## Replay Probe 결과

| Case | Result | Duplicate storage effect |
|---|---|---|
| normal write retry | completion replay | no |
| request PREPARE cut | rollback, then one execution | no |
| primary commit cut | request roll-forward, completion replay | no |
| mirror COMMIT cut | request roll-forward, completion replay | no |
| completion sent, then reset | completion replay | no |
| same ID, different LPN | conflict, fail closed | no |

핵심은 “exactly-once completion delivery”가 아니라 “effectively-once storage effect”다.

## Safe Ledger Eviction

Replay ledger를 무한히 키울 수 없으므로 ordered request ID는 cumulative replay floor로 압축한다.

```text
checkpoint PREPARE
  -> durable replay-floor COMMIT
  -> delete matching committed entries
  -> checkpoint clear
```

- floor commit은 compressed tombstone이다.
- `sequence <= evicted_through` retry는 allocator에 들어가기 전 `RequestBelowReplayFloor`로 거부된다.
- entry를 먼저 삭제하고 floor를 나중에 commit하면 duplicate suppression evidence를 잃는다.

## Selective ACK와 Session Fence

- out-of-order completion은 contiguous floor만으로는 안전하게 압축할 수 없다.
- selective tombstone과 bitmap을 사용해 gap을 보존하고, gap이 닫히면 floor로 collapse한다.
- session rollover는 durable fence를 먼저 세워 old session의 write/ACK mutation을 차단한다.
- concurrency test에서는 SACK generation CAS가 stale writer를 거부하고 refreshed writer만 merge한다.

## 해석

- 결과가 보여주는 것:
  - reset 이후 same request retry가 storage side effect를 반복하지 않도록 bounded protocol을 구성했다.
  - ledger eviction은 삭제 자체보다 floor commit ordering이 핵심이다.
  - out-of-order ACK는 selective evidence를 남긴 뒤 안전하게 압축해야 한다.
- 결과가 보여주지 못하는 것:
  - 실제 NVMe command ID persistence나 exactly-once host delivery를 증명하지 않는다.
  - payload equality, real queue semantics, multi-controller coordination은 별도 축이다.
- 가장 중요한 trade-off:
  - retry 안전성을 얻으려면 stable request identity와 durable tombstone/fence authority가 필요하다.

## 포트폴리오 / 면접 포인트

> Reset 이후 host가 같은 write를 다시 보낼 때 중복 write가 발생하지 않도록 durable request ledger와 replay floor를 모델링했습니다. Request PREPARE, primary commit, mirror commit, completion-loss cut을 나눠 6/6 deterministic probe로 검증했고, ledger entry를 삭제할 때는 durable floor를 먼저 commit해 delayed retry가 allocator에 들어가지 못하도록 했습니다.

## 관련 노트

- [[SSD FTL-GC White-box Validation Lab]]
- [[FTL Metadata Recovery and Bad Block Handling]]
- [[Controller Lease and External Fencing]]
- [[GC Pause]]
- [[Write Amplification Factor]]
