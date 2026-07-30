---
title: SSD 통신 프로토콜
aliases:
  - storage protocol
  - NVMe
  - AHCI
  - SATA protocol
  - NVMe protocol
tags:
  - concept
  - ssd
  - nvme
  - ahci
  - protocol
type: concept
status: growing
domain: SSD Host Interface
created: 2026-07-30
updated: 2026-07-30
related_projects:
  - ssd-mini-lab
---

# SSD 통신 프로토콜

## 한 줄 정의

SSD 통신 프로토콜은 host가 SSD에 read/write/flush/TRIM 같은 명령을 어떤 queue와 register, completion 규칙으로 전달하고 결과를 받는지 정한 약속이다.

## 인터페이스와 프로토콜의 차이

- [[SSD 인터페이스]]: 물리적/전기적 연결 통로. 예: SATA, PCIe, USB bridge, U.2, M.2, E1.S.
- 통신 프로토콜: 그 통로 위에서 command를 주고받는 규칙. 예: AHCI, NVMe.

즉, PCIe는 도로이고 NVMe는 그 도로 위에서 오가는 명령 체계에 가깝다. SATA도 물리/링크 계층 이름으로 쓰이지만, SSD 맥락에서는 보통 SATA + AHCI 조합으로 많이 묶어 말한다.

## 대표 프로토콜

| 프로토콜 | 주로 쓰는 연결 | 특징 | SSD 관점 |
| --- | --- | --- | --- |
| AHCI | SATA | HDD 시대에 맞춰 설계된 host controller interface | 호환성은 좋지만 queue 구조와 병렬성 활용이 NVMe보다 제한적 |
| NVMe | PCIe | 비휘발성 메모리와 PCIe 병렬성에 맞춘 storage protocol | 많은 queue, 낮은 command overhead, 높은 QD/병렬성 활용에 유리 |
| USB Mass Storage / UASP | USB | 외장 저장장치에서 사용 | USB bridge/enclosure가 SMART, TRIM, queueing, latency에 영향을 줄 수 있음 |

## NVMe 핵심

NVMe는 Non-Volatile Memory Express의 약자다. PCIe 위에서 동작하며 SSD가 가진 내부 병렬성을 더 잘 활용하도록 설계된 protocol이다.

핵심은 queue 구조다.

1. Host가 submission queue에 command를 넣는다.
2. SSD controller가 command를 가져가 처리한다.
3. 완료 결과를 completion queue에 기록한다.
4. Host는 completion을 보고 요청이 끝났음을 안다.

NVMe는 여러 submission/completion queue를 둘 수 있어 multicore CPU와 병렬 I/O에 유리하다. 이 때문에 [[Queue Depth]]와 [[p99 latency]]를 함께 봐야 한다. QD가 올라가면 throughput은 좋아질 수 있지만, queueing delay와 tail latency도 커질 수 있다.

## AHCI 핵심

AHCI는 Advanced Host Controller Interface다. SATA 저장장치를 위한 host controller interface로, HDD와의 호환성과 기존 SATA ecosystem에서 중요했다.

AHCI도 NCQ 같은 queueing 기능을 제공하지만, NVMe처럼 PCIe SSD의 높은 병렬성과 낮은 latency를 목표로 만들어진 구조는 아니다. 그래서 현대 고성능 SSD에서는 PCIe + NVMe 조합이 일반적이다.

## TRIM / discard는 어디에 들어가는가

[[TRIM]]은 host가 “이 logical block은 더 이상 필요 없다”고 알려주는 command/hint다.

- SATA/AHCI 계열에서는 ATA TRIM으로 많이 말한다.
- NVMe 계열에서는 Dataset Management / deallocate 계열 동작으로 연결해 이해할 수 있다.
- OS나 filesystem에서는 discard라는 이름으로 보일 수 있다.

중요한 것은 이름보다 의미다. host가 logical range invalidation hint를 보내고, SSD의 [[FTL]]이 이를 내부 상태에 반영할 수 있어야 한다.

## Flush / FUA / power loss와의 관계

Protocol은 단순히 read/write만 전달하지 않는다. Host가 data persistence를 기대하는 지점을 표현하기 위해 flush 같은 명령도 사용한다.

- write completion이 언제 host에 보고되는가
- volatile write cache가 있는가
- flush 이후 어디까지 persistent하다고 볼 수 있는가
- [[SSD 전력 손실 보호 원리]]가 어떤 범위까지 보장하는가

이 질문은 database, filesystem, power-loss validation에서 중요하다.

## 검증 / 실무 관점

- 실제 제품 검증에서 확인할 포인트:
  - NVMe/SATA/USB bridge 중 어떤 path로 테스트했는가
  - queue depth와 numjobs가 protocol queue 구조와 어떻게 연결되는가
  - flush, direct I/O, buffered I/O 조건이 결과에 어떤 영향을 주는가
  - 외장 SSD에서 UASP/USB bridge가 SMART, TRIM, queueing을 제한하지 않는가
- 어떤 도구/로그로 볼 수 있는가:
  - [[fio]] workload option
  - OS device manager / disk info
  - `nvme-cli`, `smartctl` 같은 도구
  - SMART / health / telemetry log

## 무엇과 헷갈리기 쉬운가

- M.2와 NVMe:
  - M.2는 form factor이고, NVMe는 protocol이다. M.2 SATA SSD도 가능하다.
- PCIe와 NVMe:
  - PCIe는 bus/interface이고, NVMe는 storage protocol이다.
- USB 외장 SSD 결과와 SSD 자체 성능:
  - USB bridge, enclosure, cable, port, OS path 영향이 섞일 수 있다.
- 높은 QD와 좋은 QoS:
  - 높은 QD는 처리량을 올릴 수 있지만 tail latency 비용이 생길 수 있다.

## 내 프로젝트와 연결

- [[SSD Mini Lab 프로젝트 허브]]에서는 외장 SSD를 file-target 경로에서 측정했기 때문에, NVMe 내부 protocol만이 아니라 USB bridge, filesystem, Windows/WSL path 영향까지 같이 봐야 한다.
- [[NVMe SMART Telemetry]] 수집이 제한되면 내부 protocol/telemetry 기반 원인 단정은 조심해야 한다.

## 관련 노트

- [[SSD 인터페이스]]
- [[Queue Depth]]
- [[fio]]
- [[TRIM]]
- [[NVMe SMART Telemetry]]
- [[SSD 전력 손실 보호 원리]]
