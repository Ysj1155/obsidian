---
title: Controller Lease and External Fencing
aliases:
  - Multi-Controller Lease MVP
  - External Monotonic Fencing
  - Anti-Rollback Authority
  - Controller Ownership Validation
tags:
  - experiment
  - validation
  - ssd
  - ftl
  - controller
  - fencing
  - recovery
type: experiment-report
status: completed
domain: SSD FTL Validation
created: 2026-07-28
updated: 2026-07-28
source: C:\Users\nei11\venv\venv\GC\docs\external_fencing_anti_rollback_mvp.md
source_type: local-report
reliability: deterministic-simulator
related_projects:
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - test-engineering
---

# Controller Lease and External Fencing

## 한 줄 결론

- Controller Lease and External Fencing은 여러 controller front end가 같은 SSD state를 다룰 때 stale owner가 write/ACK/fence/rollover를 수행하지 못하게 owner, epoch, lease expiry, external high-water authority를 검증한 ownership protocol 축이다.

## 검증 질문

- 두 controller가 같은 SSD state를 공유할 때 동시에 owner처럼 mutation할 수 없게 막을 수 있는가.
- controller A가 멈춘 뒤 controller B가 takeover할 때 PREPARE/COMMIT cut을 deterministic하게 복구할 수 있는가.
- A가 old epoch로 돌아왔을 때 write, ACK, fence, rollover를 fail closed할 수 있는가.
- 내부적으로 valid한 old whole-controller image도 external high-water epoch와 맞지 않으면 거부할 수 있는가.

## 포함한 MVP

| MVP | 원본 문서 | 결과 | 핵심 의미 |
|---|---|---|---|
| Multi-Controller Lease/Epoch | `controller_lease_epoch_mvp.md` | 6/6 PASS | shared owner/epoch authority가 stale owner mutation 차단 |
| Bounded Lease Expiry | `controller_lease_expiry_mvp.md` | 6/6 PASS | suspicion은 진단일 뿐, exact expiry + CAS만 takeover 허용 |
| External Fencing / Anti-Rollback | `external_fencing_anti_rollback_mvp.md` | 6/6 PASS | external high-water epoch가 old controller image를 fail closed |
| Threaded Adapter | `request_protocol_threaded_adapter_mvp.md` | 6/6 PASS | local thread safety를 adapter boundary 안에 둠 |

## Multi-Controller Lease

Shared durable authority는 다음 상태를 가진다.

```text
owner_controller_id
lease_epoch
lease_generation
pending lease transition
```

Lease mutation boundary:

```text
shared lease gate
  -> validate controller ID + epoch
  -> controller-local lock
  -> write / ACK / session fence / rollover / snapshot
  -> mutation commit
  -> release local lock
-> release lease gate
```

이 구조가 controller-local lock보다 강한 이유는 validation-to-commit gap을 shared gate가 닫기 때문이다.

## Lease/Epoch Probe 결과

| Case | Result |
|---|---|
| A initial acquire and write | A owns epoch 1 and executes |
| B ordinary acquire | rejected; A/epoch 1 preserved |
| B takeover PREPARE cut | rollback to A/epoch 1 |
| B takeover COMMIT cut | roll-forward to B/epoch 2 |
| stale A protocol mutations | write/ACK/fence/rollover all rejected |
| A in-flight commit vs B takeover | A finishes first; epoch 2 commits after |

## Bounded Lease Expiry

Failure detector는 renewal silence를 세 단계로 나눈다.

```text
elapsed < suspicion threshold       -> HEALTHY
elapsed >= threshold, tick < expiry -> SUSPECTED
 tick >= expiry                     -> EXPIRED
```

- `SUSPECTED`는 진단 상태일 뿐 takeover 권한이 아니다.
- `EXPIRED`와 owner/epoch/generation CAS가 맞을 때만 takeover한다.
- renew/takeover boundary에서 renew와 takeover가 같은 boundary state에 대해 동시에 성공하지 않는다.

## External Fencing / Anti-Rollback

External authority는 controller snapshot 안에 포함되지 않는 독립 authority다.

```text
owner_id
high_water_epoch
authority_generation
pending external PREPARE/COMMIT intent
```

External-first ordering:

```text
local lease PREPARE
  -> external high-water PREPARE
  -> external high-water COMMIT
  -> local lease COMMIT
  -> completion
```

- unsafe reverse order는 금지된다.
- external COMMIT 이후에는 local image가 아직 A/1을 담고 있어도 stale A가 external epoch gate를 통과하지 못한다.
- external mode에서는 direct SSD call, bare threaded adapter, bounded-only adapter mutation도 capability gate에서 거부된다.

## External Fencing Probe 결과

| Case | Result |
|---|---|
| bootstrap restart | controller A/1 matches external A/1 |
| external-first takeover | external B/2 and local B/2; stale A and direct SSD bypass rejected |
| external PREPARE cut | both authorities recover to A/1 |
| external COMMIT cut | external B/2 forces local roll-forward to B/2 |
| local COMMIT cut | both B/2 states validate |
| old whole image | controller A/1 image rejected against external B/2 |

## 해석

- 결과가 보여주는 것:
  - controller-local lock만으로는 multi-controller ownership을 설명할 수 없다.
  - shared durable lease gate와 monotonic epoch가 stale owner mutation을 막는다.
  - old controller image가 내부 checksum상 valid해도 external authority와 맞지 않으면 fail closed해야 한다.
- 결과가 보여주지 못하는 것:
  - distributed consensus, hardware persistent CAS, real storage-fabric reservation, physical clock drift bounds를 증명하지 않는다.
  - external authority와 controller image가 함께 rollback되는 공격은 막지 못한다.
- 가장 중요한 trade-off:
  - ownership safety는 local state만으로 닫히지 않는다. controller image 밖의 monotonic authority가 있어야 anti-rollback claim을 할 수 있다.

## 포트폴리오 / 면접 포인트

> Multi-controller 환경에서 stale controller가 다시 write/ACK/fence를 수행하지 못하도록 owner/epoch lease authority를 모델링했습니다. Takeover PREPARE/COMMIT cut, expired lease boundary, stale owner mutation rejection, external high-water anti-rollback을 6/6 deterministic probe로 검증했고, external authority가 controller snapshot 밖에 있어야 old whole-image rollback을 막을 수 있다는 한계를 명시했습니다.

## 관련 노트

- [[SSD FTL-GC White-box Validation Lab]]
- [[Durable Request Replay]]
- [[FTL Metadata Recovery and Bad Block Handling]]
- [[SSD Garbage Collection]]
- [[SSD QoS]]
