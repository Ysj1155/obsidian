---
title: SSD 인터페이스
aliases:
  - SSD Interface
  - SATA
  - PCIe
  - M.2
  - U.2
  - E1.S
tags:
  - concept
  - ssd
  - interface
  - sata
  - pcie
type: concept
status: growing
domain: SSD Host Interface
created: 2026-07-30
updated: 2026-07-30
---

# SSD 인터페이스

## 한 줄 정의

SSD 인터페이스는 host와 SSD가 물리적/전기적으로 연결되고 데이터를 주고받는 통로다. 이 통로 위에서 [[SSD 통신 프로토콜]]이 read/write/flush/TRIM 같은 명령을 주고받는다.

## 인터페이스와 프로토콜 구분

| 구분 | 예시 | 의미 |
| --- | --- | --- |
| 인터페이스 / bus | SATA, PCIe, USB | 데이터가 지나가는 물리적/전기적 연결 통로 |
| 프로토콜 | AHCI, NVMe, UASP | host와 device가 command를 주고받는 규칙 |
| 폼팩터 | 2.5 inch, M.2, U.2, E1.S | 장치의 모양, 커넥터, 장착 방식 |

가장 많이 헷갈리는 부분은 M.2와 NVMe다. M.2는 폼팩터이고, NVMe는 protocol이다. 따라서 M.2 SATA SSD도 가능하고, M.2 NVMe SSD도 가능하다.

## SATA

SATA는 오래된 storage interface로, HDD와 SATA SSD에서 널리 쓰였다. 일반적인 2.5 inch SATA SSD는 SATA 케이블과 전원 케이블을 사용하고, protocol은 보통 AHCI를 쓴다.

- 장점: 호환성이 넓고 오래된 시스템에서도 쓰기 쉽다.
- 한계: 대역폭과 queue 구조가 고성능 NVMe SSD에 비해 제한적이다.
- 공부 포인트: SATA SSD의 성능 한계는 NAND가 아니라 SATA/AHCI path에서 먼저 걸릴 수 있다.

## PCIe

PCIe는 CPU/chipset과 주변 장치를 연결하는 고속 bus/interface다. NVMe SSD는 보통 PCIe lane을 사용한다.

PCIe 성능은 세대와 lane 수로 표현한다.

- Gen3 x4
- Gen4 x4
- Gen5 x4

여기서 Gen은 PCIe 세대이고, x4는 lane 4개를 쓴다는 뜻이다. 세대가 올라가거나 lane 수가 늘면 이론적 bandwidth가 커진다. 다만 실제 SSD 성능은 NAND 병렬성, controller, firmware, thermal, workload 조건에 함께 좌우된다.

## USB 외장 SSD

외장 SSD는 내부적으로 SATA SSD나 NVMe SSD를 USB bridge/enclosure에 연결한 형태일 수 있다. 이 경우 host가 보는 path는 USB이고, 내부 drive의 원래 protocol이나 telemetry가 제한될 수 있다.

그래서 외장 SSD 검증에서는 다음을 분리해야 한다.

- SSD 자체 성능
- USB bridge/enclosure 성능
- cable/port 영향
- UASP 지원 여부
- filesystem/cache/path 영향
- SMART/TRIM passthrough 가능 여부

## 폼팩터

| 폼팩터 | 주 사용처 | 주의점 |
| --- | --- | --- |
| 2.5 inch SATA | PC, 구형 서버, 호환성 중심 | SATA/AHCI 성능 한계 |
| M.2 | 노트북, 데스크톱, 일부 서버 | M.2 SATA와 M.2 NVMe를 구분해야 함 |
| U.2 | 서버/데이터센터 | hot-swap, cable/backplane, enterprise SSD에서 흔함 |
| E1.S | 데이터센터 EDSFF 계열 | airflow, thermal, density, power envelope가 중요 |

## 검증 / 실무 관점

- 실제 제품 검증에서 확인할 포인트:
  - 테스트 path가 SATA, PCIe/NVMe, USB 중 무엇인가
  - lane 수와 PCIe generation이 기대와 맞는가
  - 외장 SSD라면 USB bridge가 병목인지 확인했는가
  - 동일 SSD라도 M.2/U.2/E1.S 폼팩터별 thermal condition이 다른가
  - interface 한계와 NAND/FTL 한계를 구분했는가
- 어떤 도구/로그로 볼 수 있는가:
  - OS disk/device 정보
  - `nvme-cli`, `smartctl`, Windows Device Manager
  - [[fio]] 결과의 bandwidth ceiling
  - [[NVMe SMART Telemetry]] 가능 여부

## 무엇과 헷갈리기 쉬운가

- PCIe와 NVMe:
  - PCIe는 연결 통로, NVMe는 storage protocol이다.
- M.2와 NVMe:
  - M.2는 모양/커넥터이고, NVMe는 command protocol이다.
- SATA와 2.5 inch:
  - 2.5 inch는 폼팩터이고, SATA는 interface다.
- 외장 SSD와 내부 NVMe SSD:
  - USB enclosure가 들어가면 queueing, telemetry, TRIM, latency 특성이 달라질 수 있다.

## 내 프로젝트와 연결

- [[SSD Mini Lab 프로젝트 허브]]는 외장 SSD 측정이므로 device 내부 성능뿐 아니라 USB/path/filesystem/cache 영향을 같이 기록해야 한다.
- [[External SSD Product Validation]]에서는 interface path를 검증 조건의 일부로 남겨야 한다.
- FADU 브로셔의 M.2, U.2, E1.S 비교는 단순 성능 숫자뿐 아니라 thermal/power/form-factor 조건까지 함께 읽어야 한다.

## 관련 노트

- [[SSD 통신 프로토콜]]
- [[SSD]]
- [[fio]]
- [[Queue Depth]]
- [[NVMe SMART Telemetry]]
- [[External SSD Product Validation]]
