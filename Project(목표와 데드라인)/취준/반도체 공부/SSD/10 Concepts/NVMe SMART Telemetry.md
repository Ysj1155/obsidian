---
title: NVMe SMART Telemetry
aliases:
  - SMART Telemetry
  - NVMe SMART
  - Storage Telemetry
  - SSD Telemetry
tags:
  - validation
  - ssd-validation
  - telemetry
  - nvme
  - smart
type: validation-point
status: growing
domain: SSD Validation
created: 2026-07-15
updated: 2026-07-15
source: D:\ssd_lab\docs\reports\environment_collection_week8.md
source_type: local-report
reliability: personal-experiment
related_projects:
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - product-engineering
  - test-engineering
---

# NVMe SMART Telemetry

## 한 줄 결론

- NVMe SMART Telemetry는 SSD 검증 결과를 해석할 때 device health, temperature, reliability counter 같은 환경/상태 evidence를 보강해주는 관찰 layer이며, 수집 실패나 권한 제한도 검증 결과의 일부로 기록해야 한다.

## 검증 질문

- 무엇을 확인하려는가:
  - fio 결과가 나온 시점의 drive 상태, path, free space, telemetry 가용성을 함께 기록했는가.
- 왜 이 검증이 필요한가:
  - p99 spike나 sustained write 악화를 해석할 때 device temperature, reliability counter, permission boundary가 없으면 내부 원인을 단정할 수 없다.
- 좋은 결과/나쁜 결과를 어떻게 구분하는가:
  - 좋은 결과는 환경 snapshot과 telemetry snapshot이 함께 남아 있고, 수집 불가 항목도 명확히 기록된 상태다.
  - 나쁜 결과는 성능 숫자만 있고 fio version, path, drive state, telemetry availability가 없는 상태다.

## 수집 조건

- workload:
  - telemetry 자체는 workload가 아니라 fio 전후 context 수집 layer다.
- test path:
  - Windows path, WSL path, external SSD path를 명시한다.
- 반복 횟수:
  - 중요한 run마다 env/telemetry snapshot을 남긴다.
- 환경/제약:
  - Windows permission, sandbox account, smartctl availability, USB bridge/enclosure 지원 여부가 수집 가능 범위를 제한할 수 있다.

## 봐야 할 지표

- 환경:
  - fio version
  - Python version
  - Git state
  - OS / WSL state
  - filesystem / logical drive info
- telemetry:
  - SMART/NVMe health availability
  - temperature
  - storage reliability counters
  - physical/logical disk metadata
  - free space
  - test file path and size
- 제한 사항:
  - access denied
  - smartctl unavailable
  - device-level data unavailable

## 현재 프로젝트에서 관찰한 것

- `results/env/latest/`와 `results/telemetry/latest/` 구조를 만들었다.
- fio, Python, Git, WSL 상태, path 정보를 수집할 수 있게 했다.
- storage telemetry recon에서는 일부 Windows disk/volume query가 권한 문제로 막혔다.
- `smartctl`은 현재 command environment에서 사용할 수 없었고 optional smartctl scan은 실행되지 않았다.
- 이 결과는 telemetry 실패가 아니라 현재 permission/tooling boundary를 보여주는 evidence로 해석해야 한다.

## 해석 기준

- telemetry는 성능 결과를 대체하지 않고 해석 context를 보강한다.
- telemetry가 없으면 internal root cause claim을 더 조심해야 한다.
- access denied나 tool unavailable은 조용히 무시하지 않고 report limitation에 남긴다.
- 외장 SSD에서는 SMART/NVMe data가 USB bridge를 통해 제한될 수 있다.
- temperature나 reliability counter가 없으면 thermal throttling, media error, device health 원인은 가설로만 남긴다.

## 내 프로젝트와 연결

- 관련 프로젝트:
  - [[SSD Mini Lab 프로젝트 허브]]
  - [[External SSD Product Validation]]
- 관련 실험/결과물:
  - `D:\ssd_lab\docs\reports\environment_collection_week8.md`
  - `D:\ssd_lab\results\telemetry\latest\`
- 포트폴리오/면접에서 쓸 포인트:
  - 성능 숫자뿐 아니라 fio version, path, device 상태, telemetry 수집 가능/불가능까지 검증 evidence로 남겼다.

## 면접/자소서로 바꿀 수 있는 문장

> SSD 성능 결과를 해석할 때 fio 결과만 보지 않고 환경 snapshot과 storage telemetry snapshot을 함께 남기도록 했습니다. 일부 SMART/NVMe 정보가 권한이나 tooling 때문에 수집되지 않은 경우도 limitation으로 기록해, 내부 원인을 과장하지 않도록 했습니다.

## 한계와 다음 확인

- 아직 단정하면 안 되는 부분:
  - telemetry가 없을 때 p99 spike를 thermal throttling이나 internal GC로 단정하면 안 된다.
- 추가로 확인할 실험:
  - normal user PowerShell에서 storage telemetry 재수집
  - `smartctl` 설치/가용성 확인
  - 외장 SSD USB bridge가 SMART passthrough를 지원하는지 확인
- 더 필요한 telemetry / log / metric:
  - temperature
  - media/data integrity error
  - power-on hours
  - unsafe shutdown count
  - available spare / percentage used
  - reliability counters

## 관련 노트

- [[External SSD Product Validation]]
- [[SSD Mini Lab 프로젝트 허브]]
- [[SSD QoS]]
- [[p99 latency]]
- [[fio]]
