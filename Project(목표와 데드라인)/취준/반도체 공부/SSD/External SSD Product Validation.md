---
title: External SSD Product Validation
aliases:
  - 외장 SSD 제품 검증
  - External SSD DUT Validation
  - External SSD Black-box Validation
tags:
  - experiment
  - validation
  - ssd
  - ssd-validation
  - fio
  - portfolio
type: experiment-report
status: growing
domain: SSD Validation
created: 2026-07-15
updated: 2026-07-28
source: D:\ssd_lab\docs\reports\external_ssd_product_validation.md
source_type: local-report
reliability: black-box-measurement
related_projects:
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - product-engineering
  - test-engineering
---

# External SSD Product Validation

## 한 줄 결론

- 외장 SSD를 `external_ssd_dut_01`이라는 black-box DUT로 두고, fio file-target 실험을 requirement, condition, runner/observer, raw evidence, analysis, verdict로 추적한 제품형 검증 결과다.
- 현재 프로젝트의 핵심은 최고 benchmark 숫자가 아니라 QD, sustained workload, mixed workload, block-size, data integrity, host observation limitation을 분리해 설명하는 validation system으로 성장했다는 점이다.

## 현재 상태

- QD sweep, sustained QD16/QD32, state reproducibility, 32 GiB sequential pilot, mixed workload controls, idle-ramp challenge, block-size mapping, 4 GiB CRC32C data integrity가 완료됐다.
- compact regression profile은 설계/계약 단계까지 완료됐고, 새 workload를 실행하는 단계는 아니다.
- Korean portfolio material은 사용자가 명시적으로 요청할 때까지 보류 상태다.

## DUT 범위

| 항목 | 값 |
|---|---|
| DUT label | `external_ssd_dut_01` |
| vendor/model | SanDisk Extreme SSD |
| connection | external SSD over USB path |
| filesystem | exFAT |
| main test target | `E:\validation\ssd_lab_fio_testfile` |
| large working-set target | `E:\validation\ssd_lab_seq_32g` |
| fio version | `fio-3.42` |
| test model | black-box file-target fio validation |
| repo path | `D:\ssd_lab` |
| out of scope | raw physical-drive destructive test, firmware modification, direct NAND/FTL/GC tracing, power-loss validation |

## 검증 흐름

```text
requirement -> condition -> runner / observer -> raw evidence -> analysis -> verdict
```

| 단계 | 이 프로젝트에서의 의미 |
|---|---|
| requirement | 무엇을 검증할지 `REQ-*`로 정의한다. |
| condition | workload, QD, block size, runtime, target path를 고정한다. |
| runner | fio를 실행하고 raw JSON/log/runner manifest를 남긴다. |
| observer | 실행 전후 또는 실행 중 read-only 환경/telemetry/counter evidence를 남긴다. |
| raw evidence | fio JSON, one-second logs, manifests를 보존한다. |
| analysis | CSV summary, window summary, paired comparison, verdict file을 만든다. |
| verdict | Pass, Observation, Limited, not reproduced를 분리한다. |

## 실행 Coverage

| 영역 | 대표 test / result | 상태 | Obsidian branch |
|---|---|---|---|
| QD sweep | 4K randread/randwrite QD 1/4/16/32 repeat=3 | Complete | [[왜 평균 IOPS만 보면 안 되는가]] |
| sustained QoS | QD16/QD32 120s/300s read/write | Complete | [[p99 latency]], [[SSD QoS]] |
| mixed workload | ABBA/BAAB, mixed-ratio sessions | Complete, hypothesis not reproduced | [[재현되지 않은 가설도 검증 결과다]] |
| idle ramp | 0/60/300s mirrored idle-duration test | Complete, no clear association | [[재현되지 않은 가설도 검증 결과다]] |
| block-size mapping | 4K/64K/1M QD32 randread/randwrite | Complete | [[External SSD Block Size Sweep]] |
| data integrity | 4 GiB CRC32C write/readback | Pass | [[External SSD Data Integrity]] |
| host observer | synchronized logical-disk counters | Limited | [[External SSD Data Integrity]], [[NVMe SMART Telemetry]] |
| regression profile | compact requirement-based profile | Contract complete | 이 노트의 다음 단계 섹션 |

## QD Sweep 핵심 결과

| workload | QD | avg IOPS | IOPS CV | p99 us | p99 CV | p99.9 us | p99.9 CV |
|---|---:|---:|---:|---:|---:|---:|---:|
| rand_read | 1 | 3959.11 | 0.016 | 350.21 | 0.031 | 547.50 | 0.023 |
| rand_read | 4 | 16382.92 | 0.004 | 389.80 | 0.006 | 531.80 | 0.022 |
| rand_read | 16 | 50430.60 | 0.006 | 703.15 | 0.007 | 976.21 | 0.005 |
| rand_read | 32 | 65959.01 | 0.002 | 1144.15 | 0.008 | 1591.98 | 0.006 |
| rand_write | 1 | 16053.86 | 0.008 | 92.33 | 0.034 | 142.34 | 0.025 |
| rand_write | 4 | 48123.72 | 0.052 | 129.02 | 0.187 | 242.35 | 0.166 |
| rand_write | 16 | 49210.83 | 0.014 | 398.00 | 0.016 | 583.00 | 0.032 |
| rand_write | 32 | 49410.50 | 0.021 | 1067.69 | 0.431 | 1600.17 | 0.456 |

- random read는 QD에 따라 scale했고 반복성도 비교적 안정적이었다.
- random write는 QD16 부근에서 throughput이 포화됐고, QD32는 평균 이득이 거의 없으면서 tail CV가 크게 나빠졌다.
- 이 결과는 [[왜 평균 IOPS만 보면 안 되는가]]의 핵심 근거다.

## Sustained / State 해석

- QD32 300s original session의 late-run degradation은 traced session에서 재현되지 않았다.
- QD16 120s는 낮은 throughput regime이 두 session에서 반복됐지만, severe p99.9 inflation은 고정 패키지처럼 함께 움직이지 않았다.
- state-reproducibility study에서는 baseline-conditioning-post sequence를 세 independently initiated session으로 바꿨고, bandwidth delta가 -6.37%, -7.59%, +23.33%로 섞였다.
- 가장 방어 가능한 결론은 deterministic internal mechanism이 아니라 session-level black-box variability다.

## Mixed / Idle 결과

- ABBA에서는 second mixed phase가 나빠졌지만, BAAB에서는 방향이 뒤집혔다.
- mixed-ratio Session 1과 Session 2는 ratio rank, cycle rank, position rank, Phase 5 direction, ramp pattern을 재현하지 못했다.
- idle-ramp test는 0/60/300초 pre-probe idle을 분리했지만 prior 37-53초 ramp를 재현하지 못했다.
- 이 결과들은 성능 가설을 지지하지 않지만, 검증 evidence로는 가치가 크다. 자세한 해석은 [[재현되지 않은 가설도 검증 결과다]]에 정리했다.

## Block Size 결과

- QD32 random read/write에서 64K가 observed throughput knee였다.
- 1M은 64K 대비 bandwidth 이득이 없었고, p99 latency는 read 13.77x, write 15.69x 증가했다.
- 이 결과는 block size를 평균 throughput만으로 선택하면 안 된다는 근거다.
- 자세한 수치와 해석은 [[External SSD Block Size Sweep]]에 정리했다.

## Data Integrity / Observer 결과

- 4 GiB CRC32C write/readback은 정확히 4,294,967,296 bytes를 처리했고 fio error 0으로 통과했다.
- integrity verdict는 Pass, `REQ-DATA-009`도 Pass다.
- synchronized Windows logical-disk observer는 실행됐지만 zero-only samples만 나와 host-observer evidence는 Limited다.
- 이 분리는 correctness verdict와 observability coverage를 섞지 않는 좋은 예다. 자세한 내용은 [[External SSD Data Integrity]]에 정리했다.

## Requirement Verdict 요약

| 분류 | Verdict | 의미 |
|---|---|---|
| Performance sweep | Observation / Pass evidence | 외부 spec threshold가 없어 관찰값 중심 |
| QoS metrics | Pass | p99/p99.9가 별도로 보고됨 |
| Repeatability | Pass | repeat/CV/cross-session evidence 존재 |
| Sustained | Pass | time-window 비교 존재 |
| Mixed / idle hypotheses | evidence Pass, hypothesis not reproduced | 실험 완성도와 성능 방향을 분리 |
| Block size | Pass | counterbalanced 4K/64K/1M evidence complete |
| Data integrity | Pass | CRC32C write/readback complete |
| Host observer | Limited | counter collector는 zero-only signal |
| Telemetry | Limited | SMART/reliability/fsutil coverage 제한 |
| Traceability | Pass for traced runs | runner/observer/manifests linked |
| Limitation | Pass | USB/Windows/exFAT/file-target/root-cause boundary 명시 |

## 다음 단계

- broad performance sweep은 일단 멈추는 것이 맞다.
- 다음 방향은 compact regression profile이다.
- regression profile의 기본 구성:
  - 4K random read/write QD1/QD16
  - 64K random read/write QD32
  - 120s sustained QoS condition
  - 4 GiB CRC32C file-integrity condition
- hard threshold는 아직 이르다. 현재는 `threshold_mode: observation`으로 두고, independent session이 충분해진 뒤 mature band를 정의해야 한다.

## 해석 한계

- 이 프로젝트는 내부 FTL, NAND, GC, firmware 원인을 증명하지 않는다.
- USB bridge, enclosure, host controller, Windows, exFAT, fio file-target effects가 모두 포함된 black-box 결과다.
- SMART/reliability counters는 현재 Windows/USB path에서 제한적이다.
- direct power measurement나 forced power-loss validation은 수행하지 않았다.

## 포트폴리오 / 면접 포인트

> 외장 SSD를 black-box DUT로 두고, fio 실험을 requirement, condition, runner/observer, raw evidence, analysis, verdict로 추적 가능한 검증 체계로 만들었습니다. 평균 IOPS뿐 아니라 p99/p99.9 latency, 반복 CV, sustained window, block-size knee, data integrity, host-observer limitation을 분리해 보고했고, 성능 가설이 재현되지 않은 경우에도 evidence Pass와 hypothesis verdict를 분리했습니다.

## 관련 노트

- [[SSD Mini Lab 프로젝트 허브]]
- [[SSD Mini Lab Portfolio Evidence]]
- [[External SSD Block Size Sweep]]
- [[External SSD Data Integrity]]
- [[재현되지 않은 가설도 검증 결과다]]
- [[왜 평균 IOPS만 보면 안 되는가]]
- [[fio]]
- [[Queue Depth]]
- [[p99 latency]]
- [[SSD QoS]]
- [[NVMe SMART Telemetry]]
