---
title: SSD Host Device Path
aliases:
  - Host Device Path
  - SSD I/O Path
  - Storage I/O Path
  - 호스트 장치 경로
tags:
  - concept
  - ssd
  - ssd-validation
  - io-path
type: concept
status: growing
domain: SSD Validation
created: 2026-07-30
updated: 2026-07-30
related_projects:
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - product-engineering
  - test-engineering
---

# SSD Host Device Path

## 한 줄 정의

SSD Host Device Path는 application의 I/O 요청이 OS, filesystem, cache, driver, protocol, interface와 bridge를 거쳐 SSD controller에 도달하고 다시 completion으로 돌아오는 전체 경로다.

## 한눈에 보는 경로

```text
fio / application
  -> system call
  -> filesystem
  -> OS page cache 또는 direct I/O 경로
  -> block layer / storage driver
  -> NVMe·SATA·USB storage protocol
  -> PCIe·SATA·USB interface와 bridge
  -> SSD controller
  -> FTL / NAND
  -> completion 반환
```

## 이 개념의 층위

- 위치: test framework에서 NAND까지 이어지는 end-to-end I/O 경로
- 누구의 동작인가: application, OS, driver, bridge, device가 함께 관여한다.
- 입력: test path, I/O engine, direct mode, block size, QD, read/write 요청
- 출력: host가 관찰한 bandwidth, IOPS, latency, error

## 왜 중요한가

- fio 결과는 NAND만의 성능이 아니라 선택한 경로 전체의 관찰값이다.
- 같은 SSD라도 Windows native path, WSL-mounted path, filesystem, port, cable, enclosure에 따라 결과가 달라질 수 있다.
- 경로를 기록하지 않으면 device 변화와 host-side 변화를 구분하기 어렵다.

## 핵심 구분

| 구분 | 예시 | 결과에 줄 수 있는 영향 |
| --- | --- | --- |
| Test target | file, partition, raw device | filesystem 포함 여부, 안전성, 접근 권한 |
| Cache mode | buffered, direct | page cache 개입, alignment와 지원 제약 |
| OS path | Windows, WSL filesystem, mounted Windows path | syscall과 filesystem 경로 차이 |
| Driver/protocol | NVMe, AHCI/SATA, USB Mass Storage, UASP | queue 처리와 command 전달 방식 |
| Physical path | PCIe lane, SATA link, USB port/cable | link bandwidth와 안정성 |
| Bridge/enclosure | NVMe-to-USB bridge | SMART passthrough, queueing, thermal 영향 |

## `direct=1`의 의미

`direct=1`은 지원되는 환경에서 OS page cache 영향을 줄이는 요청이다. 그러나 다음을 뜻하지는 않는다.

- OS와 driver를 완전히 우회한다.
- filesystem 영향이 모두 사라진다.
- USB bridge나 enclosure 영향이 사라진다.
- 측정값이 곧 NAND media의 순수 성능이다.

따라서 direct와 buffered 결과는 “cache 조건이 다른 두 host path”로 해석하는 편이 정확하다.

## 대표 시나리오

- 외장 SSD의 같은 파일을 같은 fio job으로 측정했는데 USB port를 바꾸자 bandwidth가 달라졌다.
- `direct=0` 결과가 비정상적으로 빠르고 반복 간 p99 변동이 컸다.
- WSL 내부 filesystem과 `/mnt/...` 경로에서 latency 분포가 달랐다.
- USB bridge가 SMART passthrough를 지원하지 않아 성능은 측정되지만 device telemetry는 얻지 못했다.

## 무엇과 헷갈리기 쉬운가

- Interface와 protocol은 같은 말이 아니다. 자세한 구분은 [[SSD 인터페이스]]와 [[SSD 통신 프로토콜]]을 본다.
- File-target test와 raw-device test는 목적과 위험이 다르다. Raw-device write는 데이터를 파괴할 수 있다.
- 빠른 buffered 결과를 SSD 자체의 지속 성능으로 단정하면 안 된다.
- 경로 차이로 생긴 현상을 내부 GC나 thermal throttling으로 바로 귀속하면 안 된다.

## 검증 / 실무 관점

- 결과마다 OS, filesystem, target path, mount 방식, direct mode를 보존한다.
- 외장 SSD라면 model, enclosure/bridge, port, cable, negotiated link를 가능한 범위에서 기록한다.
- 비교 실험에서는 바꾸려는 한 요소 외의 경로 조건을 고정한다.
- 성능 결과와 함께 error log 및 [[SSD Telemetry]] 가용성을 남긴다.

## 내 프로젝트와 연결

- [[fio]]의 `filename`, `direct`, `ioengine`, `iodepth`는 host path를 정의하는 핵심 조건이다.
- [[SSD Mini Lab 프로젝트 허브]]의 direct/buffered 및 WSL path 비교는 device 단독 비교가 아니라 경로 비교다.
- [[NVMe SMART Telemetry]] 수집 가능 여부도 USB bridge와 권한의 영향을 받는다.

## 면접용 문장

fio 결과를 SSD 자체의 숫자로만 보지 않고 OS, filesystem, cache, driver, USB bridge까지 포함한 host-device path의 관찰값으로 해석했습니다. 그래서 path와 direct mode를 결과에 함께 보존하고, 경로가 다른 결과는 별도의 비교군으로 다뤘습니다.

## 관련 노트

- [[fio]]
- [[SSD 인터페이스]]
- [[SSD 통신 프로토콜]]
- [[Storage Performance Metrics]]
- [[SSD Telemetry]]
- [[External SSD Product Validation]]
