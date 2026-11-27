---
title: NVMe SMART Telemetry
aliases:
  - SMART Telemetry
  - NVMe SMART
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
updated: 2026-11-27
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

> [!summary]
> 넓은 용어와 공통 해석 원칙은 [[SSD Telemetry]]에서 먼저 보고, 이 노트에서는 NVMe 중심 지표와 SSD Mini Lab의 실제 수집 경험을 다룬다.

## 한 줄 결론

- NVMe SMART Telemetry는 SSD 검증 결과를 해석할 때 device health, temperature, reliability counter 같은 환경/상태 evidence를 보강해주는 관찰 layer이며, 수집 실패나 권한 제한도 검증 결과의 일부로 기록해야 한다.

## SMART 로그는 어디에서 오는가

NVMe 장치에서는 호스트가 **Admin Get Log Page** 명령으로 SMART / Health Information Log(`LID 02h`)를 요청한다. 컨트롤러가 장치 상태와 누적 카운터를 정의된 형식으로 반환한다. 요청을 어느 컨트롤러에 보냈는지, 장치가 namespace별 로그를 지원하는지, 도구 출력이 어느 범위의 값을 보여주는지 확인한다. 주소와 상태가 전달되는 과정은 [[NVMe]]에서 이어서 읽는다. [NVM Express Base Specification](https://nvmexpress.org/specification/nvm-express-base-specification/)

직결 장치에선 OS NVMe 드라이버가 관리 명령을 전달할 수 있다. USB 외장 SSD에서는 USB 저장장치 드라이버와 브리지가 사이에 들어간다. 파일 읽기가 정상이어도 브리지가 해당 관리 명령을 전달하지 않거나 도구가 해당 브리지 방식을 지원하지 않을 수 있다. **읽힌 값이 0인 경우, 필드가 미보고된 경우, 로그 자체를 받지 못한 경우**를 별도 상태로 기록한다.

## 필드별 해석

| 필드 | 읽는 법 | 추가 확인 |
|---|---|---|
| Critical Warning | 비트별 현재 경고 상태다. 예를 들어 `0x09`는 bit 0과 bit 3이 설정돼 spare 부족과 미디어 read-only 상태를 함께 보고한 예다. | 같은 스냅샷의 spare, 임계값, 온도, OS 로그를 확인한다. |
| Composite Temperature | 컨트롤러가 계산한 대표 온도. 원시 로그는 Kelvin이며 도구가 °C로 변환할 수 있다. | 센서 1–8의 위치와 보고 여부는 장치마다 다를 수 있다. 최고 NAND 온도로 바로 치환하기 어렵다. |
| Available Spare / Threshold | 사용 가능한 예비 용량에 대한 정규화된 값과 경고 임계값이다. | 파일시스템 여유 공간이나 정확한 불량 블록 개수와는 다른 값이다. |
| Percentage Used | 사용량과 벤더 모델에 따른 수명 소모 추정치. 100은 추정 내구성 소진을 뜻하며 100을 넘을 수 있다. | 100을 즉시 고장으로 해석하거나 이 값에서 남은 일수를 산출할 근거는 부족하다. |
| Data Units Read / Written | 호스트 데이터 전송량. 원시 카운터 1단위는 1,000 × 512 B이며 규격의 반올림을 적용한다. | NAND에 실제로 프로그램된 바이트 수를 별도로 얻을 수 있어야 WAF를 계산한다. |
| Host Read / Write Commands | 장치가 완료한 호스트 읽기·쓰기 명령의 누적 개수다. | OS 병합·분할 때문에 fio 요청 수와 그대로 일치하지 않을 수 있다. |
| Controller Busy Time | I/O 명령으로 컨트롤러가 바빴던 누적 시간으로 단위는 분이다. | 짧은 구간에서 차분이 0이어도 컨트롤러가 놀았다고 판단할 수 없다. |
| Unsafe Shutdowns | 장치가 정의된 종료 통지를 받기 전에 전원을 잃었다고 집계한 횟수다. | 증가 자체로 데이터 손상이나 PLP 실패를 확정하려면 추가 검사가 필요하다. |
| Media and Data Integrity Errors | 컨트롤러가 감지한 복구되지 않은 무결성 오류 횟수다. | 정상 정정된 오류 수는 [[ECC]]의 별도 관찰 대상이다. |
| Error Information Log Entries | 오류 정보 로그에 기록된 누적 엔트리 수다. | 세부 Error Information Log와 OS 로그에서 상태 코드를 본다. 모든 엔트리를 미디어 오류로 분류하면 안 된다. |

일부 필드가 도구에 이미 TB, °C로 변환돼 표시될 수 있다. 수식에는 **원시 카운터**를 썼는지 확인하고, 보고서에는 도구 이름·버전과 단위를 남긴다. [Microsoft NVMe Health Information Log 필드 정의](https://learn.microsoft.com/en-us/windows/win32/api/nvme/ns-nvme-nvme_health_info_log)

## 전후 스냅샷 실습

1. 시험 직전에 timestamp, 장치 식별자, firmware, 연결 경로와 raw 로그를 기록한다. 파일 대상이 어느 물리 장치에 있는지 확인한다.
2. 동일한 조건으로 fio를 실행한다. 시간대별 지연이나 온도 변화를 비교하려면 시간 기준을 맞추고 샘플 주기를 표시한다.
3. 종료 직후 같은 장치·같은 로그 범위에서 다시 수집한다. 재연결이나 장치 교체가 있었으면 두 값이 연속 카운터인지 확인한다.
4. 누적 카운터는 `after − before`로 비교하고, 온도나 경고는 상태 변화로 읽는다. 60초 동안 원시 Data Units Written이 2,000 증가했다면 호스트 기록량은 약 `2,000 × 512,000 = 1.024 GB`, 평균 약 `17.1 MB/s`다. 다른 프로세스가 같은 범위에 기록한 양도 포함될 수 있다.

| 가상 관찰 | 보고서에 쓸 수 있는 판단 |
|---|---|
| p99 상승과 온도 상승이 같은 구간에 보임 | 시간상 함께 발생했다. 냉각 조건을 바꿔 반복하고 thermal transition 관련 기록이 있는지 확인한다. |
| Error Information Log Entries만 증가 | 새 오류 엔트리가 생겼다. 명령·상태 코드와 OS 로그를 확인해 어떤 오류인지 좁힌다. |
| Media Errors 증가 없음, CRC32C readback 일치 | 해당 스냅샷에서 미복구 오류 증가가 관찰되지 않았고, 시험 대상의 readback은 일치했다. |
| USB 경로에서 로그 요청 실패 | 그 경로에서 상태를 관찰하지 못했다. counter 0으로 입력하지 않고 도구 오류와 연결 경로를 남긴다. |

### 무노트 점검

1. `Critical Warning = 0x09`를 2진수로 풀고 두 경고를 읽는다.
2. Host Data Units Written과 SMART Media Errors만으로 WAF와 정정된 raw bit 수를 각각 구할 수 있는지 설명한다.
3. USB 연결에서 fio는 성공하고 SMART를 읽을 수 없었다. 검증 결과와 관찰 범위를 두 문장으로 써 본다.

> [!answer]- 확인용 해설
> 1. `00001001₂`. bit 0과 3이다. 각각 spare 임계값 미만과 미디어 read-only 상태를 나타낸다.
> 2. WAF에는 NAND 내부 기록량이 추가로 필요하다. Media Errors는 미복구 오류 횟수여서 정정된 raw bit 수는 알 수 없다.
> 3. 예시로 “파일 I/O는 해당 USB 경로에서 완료됐다. NVMe 상태 로그 수집은 실패해 미디어 상태는 관찰하지 못했다.”

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

- [[SSD Telemetry]]
- [[External SSD Product Validation]]
- [[SSD Mini Lab 프로젝트 허브]]
- [[SSD QoS]]
- [[p99 latency]]
- [[fio]]

