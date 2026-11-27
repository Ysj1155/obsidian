---
title: PCIe
aliases:
  - PCI Express
  - PCIe Gen3 Gen4 Gen5
tags:
  - concept
  - pcie
  - nvme
  - ssd-validation
type: concept
domain: Storage Interconnect
created: 2026-11-27
---

# PCIe

[[NVMe]]에서 읽기 명령 하나를 추적했다면, 이제 그 과정에서 **CPU가 SSD의 레지스터를 어떻게 갱신하는지, SSD가 호스트 메모리에 어떻게 접근하는지, Gen4 x4 표기가 어떤 성능 상한을 주는지** 살펴본다. 비교 범위는 SSD 프로젝트와 FADU 자료에서 사용하는 Gen3·Gen4·Gen5다.

## 1. 경로와 링크

```text
CPU / Host memory
       │
Root Complex ─ Root Port ── PCIe link ── SSD Endpoint
                           (중간에 switch가 있을 수 있음)
```

- **Root Complex / Root Port**는 호스트의 메모리·프로세서 쪽과 PCIe 계층을 연결하고, Root Port는 아래쪽 링크의 시작점이 된다.
- **Endpoint**는 SSD처럼 PCIe 트랜잭션을 보내고 받는 장치다. **Switch**가 있으면 여러 장치가 같은 upstream 링크를 공유할 수 있다.
- **Lane**은 한 방향 송신용 차동 쌍과 반대 방향 수신용 차동 쌍을 포함한다. x4는 이 lane 네 개를 묶은 링크 폭이다. 양방향이 동시에 동작할 수 있으므로 한 방향 읽기 상한과 송수신 합계는 구분한다.
- **M.2** 같은 폼팩터 표기는 장착 형태와 커넥터 규격을 설명한다. 실제 전기적 연결과 스토리지 명령을 알려면 슬롯 지원 방식과 장치 사양을 확인한다. [[SSD 인터페이스]]

## 2. 장치 발견, BAR, DMA

펌웨어나 OS는 PCIe **configuration space**를 읽어 장치의 식별 정보와 capability를 확인하고, 메모리 리소스 등을 할당한다. 장치는 버스·디바이스·펑션 번호로 식별할 수 있다. PCIe 기능을 가진 것과 해당 링크에서 그 기능이 실제로 활성화됐는지는 두 단계로 확인한다.

**BAR(Base Address Register)**는 장치가 노출하는 레지스터 등의 주소 공간을 설정하는 데 쓰인다. 호스트 드라이버는 할당된 영역을 매핑해 **MMIO**로 장치 레지스터에 접근한다. NVMe doorbell이 그 예다. **DMA**에서는 장치가 호스트 메모리로 직접 트랜잭션을 발행한다. 호스트는 버퍼 준비, DMA 매핑, 동기화와 완료 처리를 담당한다. IOMMU가 활성화된 구성에서는 장치가 사용하는 주소가 호스트 메모리에 매핑되며 접근이 제한될 수 있다. [Linux DMA API HOWTO](https://docs.kernel.org/core-api/dma-api-howto.html) · [NVMe over PCIe Transport](https://nvmexpress.org/wp-content/uploads/NVMe-over-PCIe-Transport-Specification-1_0-2021.06.02-Ratified-1.pdf)

| NVMe 동작 | PCIe에서 일어나는 일 |
|---|---|
| SQ tail doorbell 갱신 | 호스트가 SSD의 BAR 영역에 MMIO Write를 발행한다. |
| SSD가 호스트 SQ 엔트리 가져오기 | SSD가 Memory Read 요청을 보내고 호스트 쪽에서 읽기 응답을 반환한다. |
| NVMe READ의 사용자 데이터 전송 | SSD가 호스트 데이터 버퍼에 Memory Write를 보낸다. |
| NVMe WRITE의 사용자 데이터 전송 | SSD가 호스트 데이터 버퍼를 Memory Read로 읽어 온다. |
| CQ 엔트리와 완료 알림 | SSD가 CQ 내용을 호스트 메모리에 쓰고, 설정된 경우 MSI/MSI-X 메시지를 보낸다. |

여기서 이름이 가장 헷갈리는 부분은 **NVMe Read가 사용자 데이터 방향을 기준으로는 SSD→호스트지만, 그 데이터를 옮기는 PCIe 트랜잭션은 SSD가 발행한 Memory Write**라는 점이다. PCIe의 Completion TLP는 PCIe 읽기 같은 요청에 대한 응답이다. NVMe의 CQ entry는 저장장치 명령의 완료를 표현하는 별도 데이터다. [NVMe over PCIe Transport](https://nvmexpress.org/wp-content/uploads/NVMe-over-PCIe-Transport-Specification-1_0-2021.06.02-Ratified-1.pdf) · [Linux MSI 드라이버 가이드](https://docs.kernel.org/PCI/msi-howto.html)

## 3. 패킷을 운반하는 세 층

| 층 | NVMe SSD 경로에서 할 일 |
|---|---|
| Transaction Layer | MMIO나 DMA 요청을 TLP로 표현한다. Memory Read 요청에는 Completion 응답이 돌아온다. 일반적인 Memory Write는 posted 요청이므로 같은 방식의 Completion 응답을 요구하지 않는다. 수신 버퍼를 넘치지 않도록 전송 크레딧을 사용한다. |
| Data Link Layer | Gen3–5의 일반적인 링크에서 LCRC와 순서 정보를 사용해 링크 단위의 전송 오류를 확인한다. ACK/NAK 및 replay로 복구 가능한 전송 오류를 처리한다. |
| Physical Layer | 비트를 직렬 신호로 보내고 각 lane을 훈련한다. 링크 속도·폭과 equalization을 맞추고, 정상 통신 상태로 전이한다. |

PCIe 링크 replay는 인접 링크에서 손상된 패킷을 다시 보내는 동작이다. NAND read-retry는 셀을 다시 감지해 데이터를 복구하려는 동작이다. 호스트 fio 지연에서 둘을 구분하려면 링크 오류 로그와 장치 내부 통계의 관찰 지점이 다르다는 점부터 체크한다. [[ECC]] · [[NVMe SMART Telemetry]]

## 4. Link training과 실제 연결 상태

전원이 들어오면 양 끝의 포트가 신호와 lane을 확인하고 속도·폭을 맞춘다. 이 상태 전이를 관리하는 것이 **LTSSM(Link Training and Status State Machine)**이다. 학습용으로는 `Detect → Polling → Configuration → L0` 흐름을 잡고, 속도 변경과 재훈련에는 Recovery가 관여한다고 기억한다. 고속 링크의 equalization은 채널 손실이 있어도 신호를 안정적으로 샘플링하도록 송수신 특성을 조정하는 과정이다.

SSD가 Gen5 x4에 대응해도 호스트 슬롯이나 경로가 더 낮은 속도·폭을 제공하면 실제 연결은 달라질 수 있다. Linux에서 PCI 구성 정보를 조회할 수 있는 경우 `lspci -vv -s <BDF>`의 `LnkCap`과 `LnkSta`를 대조한다. 전자는 장치나 포트가 보고한 최대 성능, 후자는 해당 링크의 현재 상태를 보는 데 쓴다. SSD endpoint와 upstream 포트를 함께 확인하고, switch나 칩셋 뒤에 있다면 공유 상위 링크도 살핀다. [PCI-SIG 링크·컴플라이언스 자료](https://pcisig.com/sites/default/files/files/PCI-SIG%20Cabling%20Webinar_FINAL.pdf) · [Linux PCIe 링크 상태 예시](https://wireless.docs.kernel.org/en/latest/en/users/documentation/aspm.html)

## 5. Gen3·4·5 대역폭 계산

이 세 세대는 NRZ와 128b/130b 인코딩을 사용한다. 아래는 **한 방향의 인코딩 반영 링크 상한**이며 TLP 헤더, 흐름 제어, 기타 오버헤드는 제외한 값이다. `GB/s`는 십진 단위다.

```text
GB/s ≈ 전송률(GT/s/lane) × lane 수 × (128 / 130) ÷ 8
```

| 세대 | 전송률 | x1 | x2 | x4 |
|---|---:|---:|---:|---:|
| Gen3 | 8 GT/s/lane | 0.985 GB/s | 1.969 GB/s | 3.938 GB/s |
| Gen4 | 16 GT/s/lane | 1.969 GB/s | 3.938 GB/s | 7.877 GB/s |
| Gen5 | 32 GT/s/lane | 3.938 GB/s | 7.877 GB/s | 15.754 GB/s |

Gen4 x4의 링크 상한에 해당하는 7.877 GB/s를 4 KiB로 나누면 약 192만 IOPS다. 이 값에는 요청·응답 패킷과 장치 제약을 넣지 않았다. **4 KiB 성능 예측치**로 쓰려면 실제 요청당 오버헤드, latency와 outstanding 수, SSD 내부 병렬성 등을 추가해야 한다. 큰 블록의 순차 전송은 링크 상한을 비교하기 좋고, 작은 블록의 낮은 QD에서는 개별 요청 지연이 더 크게 작용할 수 있다. [PCI-SIG Gen별 속도와 대역폭 표](https://pcisig.com/sites/default/files/files/PCI-SIG-Seamless_Transition_to_PCIe_5.0_in_System_Implementations_FINAL.pdf) · [[External SSD Block Size Sweep]]

## 6. SSD 검증에서 성능 미달을 좁히는 법

1. **경로부터 고정한다.** Native PCIe 직결인지 USB 인클로저인지 정리한다. USB 브리지 뒤의 내부 SSD가 Gen4 x4여도 호스트까지 전송하는 USB 경로의 상한이 따로 있다.
2. **협상된 링크를 기록한다.** Gen4 x4 SSD가 실제로 Gen3 x4 또는 Gen4 x2로 붙으면 링크 상한은 둘 다 약 3.94 GB/s다. 처리량 숫자만으로 두 원인을 구별하기 어렵다.
3. **워크로드를 비교한다.** 큰 블록 순차 읽기로 전송 상한을 보고, 작은 블록 QD sweep으로 latency·IOPS를 본다. OS 캐시가 결과를 부풀리지 않도록 [[SSD Host Device Path]]의 측정 경계를 적용한다.
4. **시간이 맞는 증거를 모은다.** p99 spike·timeout과 함께 PCIe AER(Advanced Error Reporting), OS reset 기록, 온도, SMART 전후 차이를 본다. AER는 PCIe 오류를 보고하는 기능이며 지원·설정과 로그 가용성을 확인한다. 오류 로그가 없다는 사실만으로 링크 원인을 배제하기 어렵다.
5. **전원 관리 가설을 확인한다.** Idle 후 첫 I/O만 지연된다면 PCIe 링크의 ASPM과 NVMe 컨트롤러의 전원 상태 전환을 서로 다른 가설로 기록한다. 어떤 전환이 실제로 발생했는지 경로별 관찰이 필요하다. [Linux PCIe AER 가이드](https://docs.kernel.org/PCI/pcieaer-howto.html) · [Linux ASPM 설명](https://wireless.docs.kernel.org/en/latest/en/users/documentation/aspm.html)

## 7. 무노트 복습

1. Gen4 x2와 Gen3 x4의 링크 상한을 계산하고, 결과 숫자만 봤을 때 무엇을 구분할 수 없는지 말한다.
2. NVMe READ의 payload를 옮기는 PCIe 트랜잭션 종류와 방향을 그린다.
3. `LnkCap = 16 GT/s x4`, `LnkSta = 8 GT/s x4`인 포트에서 어떤 상한을 써야 하는가?
4. p99가 상승할 때 PCIe 링크 오류와 NAND 복호화 지연을 좁히기 위해 어떤 증거를 서로 대조하는가?

> [!answer]- 확인용 해설
> 1. 둘 다 약 3.94 GB/s다. 낮은 전송률인지 적은 lane 수인지는 협상 상태를 조회해야 알 수 있다.
> 2. SSD가 호스트 버퍼에 PCIe Memory Write를 보낸다. SQ를 가져오는 별도의 PCIe Memory Read도 경로에 있다.
> 3. 현재 8 GT/s x4이므로 인코딩 반영 상한은 약 3.94 GB/s다. 실제 페이로드 처리량에는 추가 오버헤드가 든다.
> 4. AER·재훈련·reset 등 링크 근거와 컨트롤러 내부 read-retry·ECC 통계 등 미디어 근거를 지연 시점에 맞춰 대조한다.

다 읽은 뒤 [[Fadu PCIe 3.0 SSD 브로셔]]와 [[Fadu PCIe 4.0 SSD 브로셔]]에서 같은 방향의 sequential throughput을 골라 링크 상한과 비교한다. 비교표에는 제품·lane 폭·읽기/쓰기·block size·QD가 명시됐는지를 같이 적는다.
