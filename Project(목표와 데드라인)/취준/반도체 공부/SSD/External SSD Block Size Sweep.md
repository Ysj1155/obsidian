---
title: External SSD Block Size Sweep
aliases:
  - 외장 SSD 블록 크기 스윕
  - External SSD 4K 64K 1M Sweep
  - Block Size Throughput Knee
tags:
  - experiment
  - validation
  - ssd
  - ssd-validation
  - fio
  - qos
type: experiment-report
status: completed
domain: SSD Validation
created: 2026-07-28
updated: 2026-07-28
source: D:\ssd_lab\docs\reports\external_ssd_block_size_sweep_result.md
source_type: local-report
reliability: black-box-measurement
related_projects:
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - product-engineering
  - test-engineering
---

# External SSD Block Size Sweep

## 한 줄 결론

- QD32 random read/write 조건에서 4K, 64K, 1M block size를 비교한 결과, 64K가 관찰된 throughput knee였다.
- 1M은 64K 대비 bandwidth 이득이 거의 없었고, p99 latency는 read 13.77x, write 15.69x 커졌다.

## 실험 목적

- 검증 질문:
  - 외장 SSD file-target random I/O에서 block size를 키우면 throughput과 tail latency가 어떻게 바뀌는가.
- 왜 이 실험이 필요한가:
  - IOPS, bandwidth, latency는 block size에 따라 의미가 달라진다.
  - 큰 block size가 항상 좋은 조건인지 확인하려면 throughput gain과 tail cost를 같이 봐야 한다.
- 기대한 관찰:
  - 4K에서 64K로 갈 때 throughput이 증가할 수 있지만, 64K 이후에도 이득이 계속되는지는 별도로 확인해야 한다.

## 조건 매트릭스

| 항목 | 값 |
|---|---|
| target | `E:\validation\ssd_lab_seq_32g` |
| file size | 32 GiB |
| workloads | `randread`, `randwrite` |
| block sizes | 4K, 64K, 1M |
| queue depth | 32 |
| runtime | 30s per phase |
| I/O path | `direct=1`, `windowsaio`, `numjobs=1` |
| repeats | 각 workload/block size마다 3 placements |
| order control | Latin-square style counterbalanced order |

## 핵심 결과

| Workload | Block size | BW MiB/s | BW CV | IOPS | p99 ms | p99.9 ms | Max ms |
|---|---:|---:|---:|---:|---:|---:|---:|
| Read | 4K | 215.511 | 2.24% | 55,170.8 | 0.870 | 1.374 | 23.189 |
| Read | 64K | 303.713 | 0.35% | 4,859.4 | 8.389 | 12.146 | 68.593 |
| Read | 1M | 304.145 | 0.03% | 304.1 | 115.518 | 181.404 | 221.975 |
| Write | 4K | 144.203 | 2.50% | 36,916.1 | 1.173 | 2.051 | 25.954 |
| Write | 64K | 497.109 | 0.15% | 7,953.7 | 4.446 | 5.669 | 46.386 |
| Write | 1M | 489.617 | 0.14% | 489.6 | 69.730 | 98.391 | 127.347 |

## Throughput Knee

| 비교 | Bandwidth 변화 | p99 latency 변화 |
|---|---:|---:|
| Read 64K / 4K | 1.409x | 증가 |
| Read 1M / 64K | 1.001x | 13.77x |
| Write 64K / 4K | 3.447x | 증가 |
| Write 1M / 64K | 0.985x | 15.69x |

- 4K에서 64K로 이동할 때 throughput은 크게 좋아졌다.
- 64K에서 1M으로 이동할 때 throughput plateau에 도달했고, latency cost만 크게 증가했다.
- 이 조건에서는 64K가 throughput/latency balance가 가장 좋은 관찰점이었다.

## 해석

- 결과가 보여주는 것:
  - 같은 QD32라도 block size가 커지면 outstanding data가 커지고, latency 해석이 달라진다.
  - 1M I/O는 bandwidth를 더 올리지 못하면서 tail latency를 크게 악화시켰다.
  - counterbalanced order 덕분에 단순히 “뒤에 실행해서 느려졌다”는 설명을 줄일 수 있었다.
- 결과가 보여주지 못하는 것:
  - 64K가 모든 SSD, 모든 workload에서 최적이라는 뜻은 아니다.
  - USB, host scheduling, exFAT, firmware, cache, thermal, FTL, GC 중 어느 것이 원인인지는 식별하지 못한다.
- 가장 중요한 trade-off:
  - block size 비교에서는 IOPS를 단일 성능 점수로 비교하면 안 된다. I/O 하나가 운반하는 byte 수가 다르기 때문이다.

## 포트폴리오 / 면접 포인트

> 외장 SSD에서 4K, 64K, 1M random I/O를 QD32로 비교하고, 64K가 관찰된 throughput knee임을 확인했습니다. 1M은 64K 대비 bandwidth 이득이 없었지만 p99 latency가 read 13.77x, write 15.69x 증가해, block size 선택도 평균 처리량과 tail latency를 함께 봐야 한다는 검증 포인트로 정리했습니다.

## 관련 노트

- [[SSD Mini Lab 프로젝트 허브]]
- [[External SSD Product Validation]]
- [[fio]]
- [[Queue Depth]]
- [[p99 latency]]
- [[SSD QoS]]
- [[왜 평균 IOPS만 보면 안 되는가]]
