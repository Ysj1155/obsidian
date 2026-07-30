---
title: SSD Telemetry
aliases:
  - Storage Telemetry
  - SSD 상태 정보
  - SSD 텔레메트리
tags:
  - concept
  - ssd
  - ssd-validation
  - telemetry
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

# SSD Telemetry

## 한 줄 정의

SSD Telemetry는 성능 숫자의 배경을 설명하기 위해 장치가 제공하는 health, temperature, wear, error, power 상태와 event 정보를 수집한 관찰 자료다.

## 이 개념의 층위

- 위치: SSD controller/firmware가 만든 상태 정보와 host tool이 수집한 log
- 누구의 동작인가: device가 counter/log를 제공하고 host가 protocol과 tool을 통해 읽는다.
- 입력: 장치 내부 event, error, 온도, 사용량, power cycle, firmware 상태
- 출력: SMART/health log, NVMe log page, OS reliability counter, vendor telemetry

## 무엇을 볼 수 있는가

| 범주 | 대표 항목 | 해석 질문 |
| --- | --- | --- |
| Health | critical warning, available spare, percentage used | 장치가 health 경고나 wear 상태를 보고하는가 |
| Thermal | composite temperature, sensor 값 | workload 전후 온도 변화가 있었는가 |
| Error | media/data integrity error, error log | 오류 징후가 성능 이상과 함께 나타났는가 |
| Power | power-on hours, power cycle, unsafe shutdown | 전원 이력과 비정상 종료가 있었는가 |
| Usage | host read/write counter, data units read/written | 장치에 누적된 host traffic은 어느 정도인가 |
| Device identity | model, serial, firmware, link 정보 | 같은 장치와 환경을 비교하고 있는가 |

Host write는 host가 SSD에 보낸 write traffic을 뜻한다. NAND 내부에서 GC나 metadata 때문에 추가로 발생한 physical write와는 다르며, 두 값의 비율이 [[Write Amplification Factor]]와 연결된다.

## SMART, NVMe log, vendor telemetry

- SMART는 저장장치의 health/status 정보를 부르는 넓은 표현으로 쓰인다.
- NVMe SSD는 표준화된 SMART/Health Information 및 여러 log page를 제공할 수 있다.
- Vendor/OCP telemetry는 표준 health log보다 더 상세한 debug/event 정보를 제공할 수 있지만 제품과 권한에 따라 접근성이 다르다.
- SATA SMART attribute와 NVMe health field는 구조와 의미가 같지 않으므로 이름만 보고 직접 대응시키지 않는다.

자세한 NVMe 검증 관점과 현재 프로젝트의 수집 결과는 [[NVMe SMART Telemetry]]에 둔다.

## Telemetry availability란

Telemetry availability는 “장치가 정보를 갖고 있는가”뿐 아니라 “현재 경로에서 읽을 수 있는가”까지 포함한다.

- OS 권한이 충분한가
- 수집 도구가 설치되어 있는가
- driver가 명령을 전달하는가
- USB bridge/enclosure가 SMART/NVMe passthrough를 지원하는가
- vendor 전용 정보에 접근할 수 있는가

따라서 `access denied`, `smartctl unavailable`, `bridge passthrough unsupported`도 숨기지 않고 검증 한계로 기록한다.

## 성능 결과와 결합하는 법

```text
workload 전 snapshot
  -> fio run과 timestamp
  -> workload 후 snapshot
  -> latency/throughput 변화와 함께 비교
```

- Temperature 상승과 p99 악화가 같이 나타났다면 관련 가능성을 제시할 수 있다.
- 그러나 snapshot만으로 thermal throttling이 원인이라고 확정할 수는 없다.
- Counter는 누적값일 수 있으므로 전후 delta와 단위를 확인한다.
- 수집 주기가 길면 짧은 spike를 놓칠 수 있다.

## 무엇과 헷갈리기 쉬운가

- Telemetry는 benchmark가 아니다. 장치 상태를 설명하는 보조 evidence다.
- Counter 이름이 같아 보여도 vendor와 protocol마다 단위와 갱신 조건이 다를 수 있다.
- Host write와 NAND write는 같은 값이 아니다.
- Telemetry가 없다는 사실은 문제가 없다는 뜻이 아니라 관찰할 수 없다는 뜻일 수 있다.
- [[SSD 전력 손실 보호 원리|PLP]]는 전력 손실 시 데이터를 보호하는 설계이고, unsafe shutdown counter는 전원 사건을 기록하는 telemetry다.

## 검증 / 실무 관점

- fio 결과와 같은 run ID 또는 timestamp로 snapshot을 연결한다.
- Model, serial, firmware, path를 기록해 다른 장치의 로그가 섞이지 않게 한다.
- Raw output을 보존하고 parser가 모르는 필드는 누락으로 표시한다.
- 성능 이상을 내부 GC, thermal, media error로 단정하기 전에 telemetry와 OS/device log를 교차 확인한다.

## 내 프로젝트와 연결

- [[SSD Mini Lab 프로젝트 허브]]에서 환경 및 telemetry snapshot 수집 구조를 만들었다.
- [[SSD Host Device Path]]의 USB bridge와 권한이 telemetry 가용 범위를 제한할 수 있다.
- [[Storage Performance Metrics]]의 변화와 telemetry를 함께 보면 설명력은 높아지지만, black-box 결과만으로 내부 원인을 확정하지 않는다.

## 면접용 문장

성능 결과와 함께 temperature, health, error counter, firmware와 수집 가능 여부를 snapshot으로 남겼습니다. Telemetry가 제한된 경우도 검증 한계로 기록해, p99 악화의 내부 원인을 근거 없이 단정하지 않았습니다.

## 관련 노트

- [[NVMe SMART Telemetry]]
- [[SSD Host Device Path]]
- [[Storage Performance Metrics]]
- [[SSD QoS]]
- [[Write Amplification Factor]]
- [[SSD 전력 손실 보호 원리]]
- [[External SSD Product Validation]]
