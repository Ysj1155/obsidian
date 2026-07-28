---
title: External SSD Data Integrity
aliases:
  - 외장 SSD 데이터 무결성 검증
  - External SSD CRC32C Integrity
  - File-Target Integrity Result
tags:
  - experiment
  - validation
  - ssd
  - data-integrity
  - fio
  - portfolio
type: experiment-report
status: completed
domain: SSD Validation
created: 2026-07-28
updated: 2026-07-28
source: D:\ssd_lab\docs\reports\external_ssd_data_integrity_result.md
source_type: local-report
reliability: black-box-measurement
related_projects:
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - product-engineering
  - test-engineering
---

# External SSD Data Integrity

## 한 줄 결론

- `EXT-DATA-INTEGRITY-001`은 4 GiB file-target을 CRC32C로 write/readback 검증한 correctness MVP이며, integrity verdict는 Pass다.
- 단, synchronized Windows host counter observer는 fio 실행 중 nonzero workload signal을 잡지 못해 evidence status가 Limited로 남았다.

## 실험 목적

- 검증 질문:
  - 새로 쓴 4 GiB 파일을 완전히 다시 읽었을 때 fio CRC32C verification error 없이 통과하는가.
- 왜 이 실험이 필요한가:
  - SSD 검증이 평균 성능만 보는 작업이면 부족하다.
  - 제품형 검증에서는 data path correctness와 evidence completeness도 함께 확인해야 한다.
- 기대한 관찰:
  - write phase와 verify-read phase가 모두 정확히 4 GiB를 처리하고, fio job error가 0이어야 한다.

## 조건 매트릭스

| 항목 | 값 |
|---|---|
| protocol | `EXT-DATA-INTEGRITY-001` |
| evidence unit | `data_integrity_4g_20260727_retry1` |
| target | `E:\validation\ssd_lab_verify_4g_20260727_retry1` |
| file size | 4 GiB |
| block size | 1 MiB |
| queue depth | 4 |
| engine | `windowsaio` |
| direct I/O | `1` |
| verification | `crc32c` |
| fio version | `fio-3.42` |

## Integrity Result

| Phase | fio error | Bytes | BW | p99 | p99.9 | Max |
|---|---:|---:|---:|---:|---:|---:|
| CRC32C write | 0 | 4,294,967,296 | 465.455 MiB/s | 10.289 ms | 11.076 ms | 239.891 ms |
| CRC32C verify read | 0 | 4,294,967,296 | 520.656 MiB/s | 7.832 ms | 15.401 ms | 17.386 ms |

- verify phase는 `verify=crc32c`, `verify_only=1`, `do_verify=1` 상태로 실행됐다.
- fio 3.42는 성공 run에서 explicit `verify_errors` field를 내지 않았지만, analyzer는 verify mode, complete byte count, fio job error 0이 모두 맞을 때만 absence를 허용한다.

## Verdict

| 항목 | Verdict | 의미 |
|---|---|---|
| integrity_verdict | Pass | 4 GiB write/readback CRC32C 검증 통과 |
| `REQ-DATA-009` | Pass | correctness evidence 충족 |
| host observer | Limited | logical-disk counter가 zero-only sample을 냄 |
| `REQ-HOST-OBS-010` | Limited | synchronized observer evidence는 usable signal이 부족함 |
| integrated_run_status | Limited | integrity는 Pass지만 관측 coverage가 제한됨 |

## First-Attempt Finding

- 첫 시도 `data_integrity_4g_20260727`은 write 자체는 4 GiB와 fio error 0을 남겼지만, fio가 만든 파일이 verify 단계에 남아 있지 않았다.
- retry1에서는 runner가 `FileMode.CreateNew`와 `SetLength`로 파일을 먼저 만들고, fio가 전체 파일을 overwrite한 뒤 verify-only read를 수행했다.
- 이 실패 evidence를 버리지 않고 retry 설계에 반영한 점이 중요하다.

## 해석

- 결과가 보여주는 것:
  - 이 Windows/exFAT/USB/file-target 경로에서 하나의 4 GiB 파일 write/readback CRC32C 검증은 통과했다.
  - correctness verdict와 observer coverage verdict를 분리했다.
- 결과가 보여주지 못하는 것:
  - power-loss protection, endurance, NAND/FTL behavior, internal ECC coverage, cable removal recovery는 증명하지 못한다.
  - host logical-disk counter가 zero-only였기 때문에 transient storage activity correlation은 아직 완성되지 않았다.
- 가장 중요한 trade-off:
  - `Pass`와 `Limited`는 서로 모순이 아니다. 데이터 무결성 검증은 Pass일 수 있지만, 관측 장치가 충분한 signal을 못 주면 evidence status는 Limited로 남겨야 한다.

## 포트폴리오 / 면접 포인트

> 성능 측정에 그치지 않고 4 GiB CRC32C write/readback으로 file-target data integrity를 검증했습니다. Integrity verdict는 Pass였지만 Windows logical-disk observer가 nonzero workload signal을 잡지 못했기 때문에 observer coverage는 Limited로 분리했습니다. 이를 통해 검증 결과와 관측 한계를 같은 보고서 안에서 분리해 다뤘습니다.

## 관련 노트

- [[SSD Mini Lab 프로젝트 허브]]
- [[External SSD Product Validation]]
- [[NVMe SMART Telemetry]]
- [[fio]]
- [[SSD QoS]]
