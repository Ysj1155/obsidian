---
title: NVMe
aliases:
  - NVM Express
  - NVMe 명령과 큐
tags:
  - concept
  - nvme
  - ssd-controller
  - ssd-validation
type: concept
domain: Storage Protocol
created: 2026-11-27
---

# NVMe

호스트가 SSD에 4 KiB 읽기를 요청했을 때, SSD는 **어느 논리 블록을 읽을지, 결과를 호스트 메모리 어디에 놓을지, 완료를 어떻게 알릴지**를 알아야 한다. NVMe over PCIe의 큐와 명령을 이 세 질문에 맞춰 따라간다. 상태 로그부터 보고 왔다면 Get Log Page 역시 이 구조 위에서 실행되는 Admin 명령이다. [[NVMe SMART Telemetry]]

## 1. 어떤 규격을 펼쳐야 하는가

| 궁금한 것 | 먼저 볼 규격 |
|---|---|
| Controller, namespace, Admin 명령, 로그, 상태 코드의 공통 구조 | NVMe Base |
| Read·Write·Flush, 논리 블록과 NVM 명령의 의미 | NVM Command Set |
| PCIe register, 큐 doorbell, 메모리 기반 전송과 인터럽트 | NVMe over PCIe Transport |

NVMe 명령은 PCIe 이외의 transport에서도 사용할 수 있다. 이 노트의 MMIO doorbell과 호스트 메모리 큐 설명은 **일반적인 로컬 NVMe over PCIe 경로**에 적용한다. [NVM Express 규격 구성](https://nvmexpress.org/specification/nvm-express-base-specification/) · [NVMe over PCIe Transport 1.0](https://nvmexpress.org/wp-content/uploads/NVMe-over-PCIe-Transport-Specification-1_0-2021.06.02-Ratified-1.pdf)

## 2. Controller와 namespace

**Controller**는 호스트 명령과 완료 처리를 제공하는 엔티티다. **Namespace**는 호스트가 논리 블록 단위로 접근하는 저장 공간이며, 명령은 NSID로 대상을 식별할 수 있다. 같은 LBA 번호라도 namespace가 다르면 다른 논리 공간의 주소다. 파티션은 보통 OS가 namespace의 블록 공간 안에서 나눈 구획이다.

Read·Write 명령에서 `SLBA`는 시작 LBA, `NLB`는 0부터 세는 블록 수 필드다. `SLBA=200, NLB=7`이라면 LBA 200–207의 8개 블록이다. LBA 데이터 크기가 512 B라면 4 KiB, 4 KiB라면 32 KiB다. 실제 LBA 데이터 크기는 Identify Namespace에서 active LBA format을 확인한다. NAND 물리 주소는 SSD의 [[FTL]]이 해석한다. [[LBA LPN PPN]]

## 3. 부팅 시 큐가 생기는 순서

1. OS가 PCIe 장치를 발견하고 컨트롤러 BAR를 통해 레지스터를 접근한다.
2. 호스트 드라이버가 Admin SQ와 CQ에 쓸 메모리를 확보하고 큐 크기와 주소를 구성한다. 컨트롤러를 활성화하고 준비 상태를 확인한다.
3. Admin 큐로 Identify, Get Log Page, Set Features 등을 실행한다. 큐 개수를 구성한 뒤 통상 I/O CQ를 먼저 만들고, 그 CQ에 연결되는 I/O SQ를 만든다.
4. 이후 데이터 경로의 Read·Write 명령은 I/O 큐를 통해 처리된다. 컨트롤러의 최대 큐 크기나 개수와 드라이버가 실제 사용하는 값은 따로 확인한다.

Admin은 장치 관리, I/O는 데이터 접근을 맡는다. **Admin 명령 성공이 데이터 경로까지 정상이라는 판정으로 곧바로 이어지지는 않는다.** [NVMe over PCIe Transport](https://nvmexpress.org/wp-content/uploads/NVMe-over-PCIe-Transport-Specification-1_0-2021.06.02-Ratified-1.pdf)

## 4. 4 KiB READ를 한 요청으로 추적하기

```text
호스트: SQ 메모리에 명령 작성 → SQ tail doorbell 갱신
SSD:    SQ 엔트리 DMA 읽기 → FTL 조회와 NAND 읽기 → 데이터 DMA 쓰기
SSD:    CQ 엔트리 DMA 쓰기 → 필요하면 인터럽트
호스트: CQ 엔트리 확인 및 I/O 완료 처리 → CQ head doorbell 갱신
```

1. **명령 작성.** 호스트가 SQ 엔트리에 opcode·CID·NSID·SLBA·NLB를 쓰고, 데이터가 도착할 호스트 메모리 위치를 데이터 포인터로 지정한다. PRP는 메모리 페이지 기준의 물리 메모리 위치 정보를 전달하고, 지원되는 경우 SGL은 주소·길이 등으로 버퍼 구간을 기술한다. DMA 관점의 주소는 IOMMU 매핑을 거칠 수 있다.
2. **제출.** 드라이버가 명령 내용을 컨트롤러에 보이게 한 후 MMIO로 SQ tail doorbell을 갱신한다. Doorbell에는 새 꼬리 위치가 들어간다. 명령 본문과 payload는 메모리를 통해 전달된다.
3. **실행.** 컨트롤러가 SQ에서 명령을 가져온다. Read라면 실제 데이터를 호스트 버퍼에 DMA로 쓰고, Write라면 그 버퍼에서 데이터를 DMA로 읽는다. NAND·캐시·내부 병렬성의 구체적 사용 방식은 구현에 따른다.
4. **완료.** 컨트롤러는 CQ에 SQID·CID·SQ Head 정보·상태 코드 등을 적는다. 인터럽트가 설정된 경로에서는 호스트에 알릴 수 있고, 호스트가 polling으로 확인하는 구성도 있다.
5. **회수.** 호스트는 phase tag로 해당 슬롯의 엔트리가 새 것인지 구분하고 SQID·CID로 원래 요청을 찾는다. 소비한 CQ 위치를 head doorbell로 알리면 컨트롤러가 슬롯을 재사용할 수 있다.

SQ에서 명령을 가져갔다는 사실과 명령 처리가 끝났다는 사실은 서로 다른 이벤트다. 여러 요청이 있을 때 완료 순서가 제출 순서와 다를 수 있으므로, 드라이버는 큐 위치 하나만 보고 원본 요청을 찾지 않는다. [NVMe over PCIe Transport](https://nvmexpress.org/wp-content/uploads/NVMe-over-PCIe-Transport-Specification-1_0-2021.06.02-Ratified-1.pdf) · [Linux NVMe PCI endpoint 구현 설명](https://www.kernel.org/doc/html/v6.14/nvme/nvme-pci-endpoint-target.html)

## 5. Queue Depth와 완료 시간

`fio iodepth`는 job·I/O 엔진에서 목표로 삼는 outstanding I/O의 깊이다. OS 블록 큐, 드라이버의 큐 매핑, 실제 명령 병합·분할, device 큐에서 관찰한 깊이는 같은 값이라고 가정할 수 없다. 조건을 볼 때는 `ioengine`, `numjobs`, fio의 실제 depth 분포, OS와 장치가 노출하는 지표의 관찰 지점을 같이 적는다. [[fio]] · [[Queue Depth]]

정상 상태에서 같은 경계를 기준으로 보면 평균 outstanding 수는 대략 `완료율 × 평균 체류 시간`이다. 예를 들어 100,000 IOPS에 평균 200 µs면 평균 20개다. 이 계산에 p99를 대입하면 평균 outstanding을 얻을 수 없다. QD를 높였는데 IOPS가 그대로이고 p99만 늘었다면 병렬성의 추가 이득이 작아졌다는 관찰은 가능하다. 병목 위치는 OS·컨트롤러·NAND·[[PCIe]] 중 어느 곳인지 추가로 확인한다.

## 6. 완료와 영속성, 그리고 오류

| 신호나 명령 | 해석할 때 물을 것 |
|---|---|
| CQ status success | 장치가 이 명령의 완료를 성공으로 보고했다. 시험이 요구한 전원 장애 시점과 영속성 조건은 별도로 확인한다. |
| Flush | 해당 명령의 범위와 순서에 맞춰 데이터·메타데이터를 비휘발성 매체에 반영하도록 요청한다. 호스트 측 캐시 및 파일시스템에서 실제로 어떤 명령이 내려가는지도 영향을 준다. |
| Write의 FUA | 그 쓰기의 데이터·메타데이터가 비휘발성 매체에 반영된 뒤 완료하도록 요구한다. 다른 명령에 대한 일반적인 순서 보장으로 확대하지 않는다. |
| NVMe status error | 컨트롤러의 완료 상태다. 상태 유형과 코드, 해당 명령, Error Information Log를 함께 본다. |
| Timeout / reset | 호스트가 기한 안에 완료를 받지 못한 상황일 수 있다. OS·driver·PCIe 오류와 함께 보고, 기존 명령이 어디까지 처리됐는지 추가 증거를 확인한다. |

TRIM을 연결해 보면 OS의 discard 요청이 NVMe 경로에서 Dataset Management의 Deallocate로 내려갈 수 있다. 논리 범위를 더 이상 사용하지 않는다는 통지이며, 기존 물리 페이지의 즉시 소거나 보안 삭제 완료를 증명하려면 다른 근거가 필요하다. [[TRIM]] · [[SSD 전력 손실 보호 원리]] · [[SSD 통신 프로토콜]]

## 7. 무노트 복습

1. 네가 쓴 Read 명령을 SSD가 어떻게 발견하고 데이터는 어디에 쓰는지 순서대로 그린다.
2. `SLBA=50, NLB=3`이고 LBA 데이터가 4 KiB면 실제 읽는 범위와 크기는 얼마인가?
3. CQ 완료가 같은 순서로 돌아오지 않아도 어떤 필드로 요청을 찾는가?
4. Write가 성공했다는 보고와 전원 손실 후 데이터가 살아 있다는 관찰은 어떻게 검증하는가?

> [!answer]- 확인용 해설
> 1. SQ 기록 → tail doorbell → 컨트롤러가 SQ 가져오기 → 데이터 버퍼와 CQ에 DMA → 호스트가 완료 처리.
> 2. LBA 50–53, 4개 블록이므로 16 KiB다.
> 3. SQID와 CID로 찾는다. Phase tag는 CQ 슬롯이 새 엔트리인지 확인하는 데 쓴다.
> 4. 완료 상태와 Flush/FUA 조건, 캐시 정책을 기록하고 전원 복구 후 해당 주소의 데이터·메타데이터를 확인한다.
