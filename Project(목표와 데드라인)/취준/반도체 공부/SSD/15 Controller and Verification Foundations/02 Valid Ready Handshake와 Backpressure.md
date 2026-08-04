---
title: Valid Ready Handshake와 Backpressure
aliases:
  - Valid Ready Handshake
  - Ready Valid Handshake
  - Handshake와 Backpressure
tags:
  - concept
  - ssd
  - controller
  - handshake
  - backpressure
  - axi
  - digital-design
  - verification
type: learning-module
status: growing
domain: SSD Controller Validation
created: 2026-08-04
updated: 2026-08-04
source_type: official-protocol-spec
reliability: source-checked-learning-note
related_projects:
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - rtl-verification
---

# Valid Ready Handshake와 Backpressure

## 한 문장 정의

Valid/ready handshake는 sender가 “보낼 정보가 유효하다”고 알리고 receiver가 “지금 받을 수 있다”고 알렸을 때만 clock edge에서 한 번의 transfer를 성립시키는 흐름 제어 방식이다.

## 쉬운 비유

Sender가 택배 상자를 들고 있고 receiver가 창고 문을 연다고 생각해보자.

- `valid=1`: 전달할 상자와 송장이 준비됐다.
- `ready=1`: 창고에 받을 공간과 처리 능력이 있다.
- `valid && ready=1`: 이번 clock edge에서 인계가 성립한다.
- `valid=1, ready=0`: 상자를 버리거나 바꾸지 말고 기다린다.

Receiver가 바쁘면 `ready`를 내릴 수 있다. 이 정지 신호가 upstream으로 전파되는 것이 backpressure다.

## 신호의 주체

| 신호 | 누가 만든다 | 뜻 |
| --- | --- | --- |
| `valid` | sender/source | payload가 현재 유효하다 |
| `ready` | receiver/destination | 이번 transfer를 받아들일 수 있다 |
| `payload` | sender/source | data, command, address, metadata 등 |
| `fire` | 양쪽이 해석 | `valid && ready`가 clock edge에서 참인 transfer event |

```systemverilog
wire fire = valid && ready;
```

`fire`는 표준 신호 이름이 아니라 RTL에서 handshake 성립 조건을 읽기 쉽게 붙이는 관용적 이름이다.

## 네 가지 조합

| valid | ready | clock edge에서의 의미 |
| --- | --- | --- |
| 0 | 0 | 보낼 정보가 없고 receiver도 수용을 약속하지 않음 |
| 0 | 1 | receiver는 받을 수 있지만 sender가 보낼 것이 없음 |
| 1 | 0 | sender는 유효한 payload를 유지하며 기다림 |
| 1 | 1 | 한 건의 transfer 성립 |

`0, 0`의 정확한 내부 사유는 신호만으로 알 수 없다. Receiver가 실제로 바쁜지, 단순히 ready를 필요할 때만 올리는 설계인지는 구현에 달려 있다.

## 반드시 지켜야 할 핵심 규칙

Arm AXI 규칙을 기초로 보면 다음이 중요하다.

1. Sender는 `ready`가 올라올 때까지 기다린 뒤 `valid`를 올리는 방식으로 의존하면 안 된다.
2. Receiver는 `valid`를 본 뒤 `ready`를 올릴 수도 있고, 미리 `ready`를 올려둘 수도 있다.
3. `valid=1, ready=0`으로 stall된 동안 sender는 valid를 유지하고 payload를 바꾸지 않아야 한다.
4. 상태와 counter는 `valid`만이 아니라 실제 transfer인 `valid && ready`를 기준으로 갱신해야 한다.

첫 번째 규칙은 deadlock을 피하기 위한 핵심이다.

```text
잘못된 sender: ready가 오기 전에는 valid를 올리지 않음
잘못된 receiver: valid가 오기 전에는 ready를 올리지 않음
결과: 둘 다 영원히 0인 deadlock 가능
```

Receiver가 valid를 기다리는 것은 AXI에서 허용되지만, sender까지 ready를 기다리면 안 된다.

## Cycle로 읽기

다음은 payload `A`가 두 cycle 동안 막혔다가 전달되는 예다.

| Cycle | valid | ready | payload | 결과 |
| --- | ---: | ---: | --- | --- |
| 0 | 0 | 1 | - | transfer 없음 |
| 1 | 1 | 0 | A | A를 유지하며 stall |
| 2 | 1 | 0 | A | A를 유지하며 stall |
| 3 | 1 | 1 | A | edge에서 A transfer |
| 4 | 0 | 1 | - | 다음 payload 대기 |

핵심은 Cycle 1~2에서 `A`가 사라지거나 `B`로 바뀌지 않는 것이다.

### 연속 전송

Sender와 receiver가 매 cycle 준비되어 있으면 `valid=1`, `ready=1`을 유지하면서 매 clock edge마다 서로 다른 payload를 전달할 수 있다.

| Cycle | valid | ready | payload | transfer |
| --- | ---: | ---: | --- | --- |
| 0 | 1 | 1 | A | A |
| 1 | 1 | 1 | B | B |
| 2 | 1 | 1 | C | C |

Handshake는 반드시 매 transfer 사이에 valid나 ready를 0으로 내리라는 방식이 아니다.

## Backpressure란

Backpressure는 downstream이 현재 더 받을 수 없다는 사실을 upstream에 전달해 overflow와 데이터 손실을 막는 흐름 제어다.

```text
NAND scheduler busy
  -> internal request FIFO가 차오름
  -> FIFO ready=0
  -> command decode stage stall
  -> upstream queue residence time 증가
  -> host-visible latency와 tail latency 증가 가능
```

Backpressure 자체는 오류가 아니다. 유한한 buffer가 있는 시스템에서 정상적인 보호 장치다. 문제는 다음과 같은 경우다.

- 너무 오래 지속되어 progress가 사라진다.
- 한 traffic class에만 계속 전파되어 starvation이 생긴다.
- buffer가 full인데도 sender가 payload를 밀어 넣어 overflow가 난다.
- backpressure가 어디서 시작됐는지 telemetry와 trace로 볼 수 없다.

## SSD controller에서의 위치

Ready/valid와 유사한 내부 흐름 제어는 다음 경계에 적용해 생각할 수 있다.

- command fetch -> decode
- decode -> internal command FIFO
- FTL lookup -> allocator
- DMA engine -> write buffer
- scheduler -> NAND channel command interface
- completion generator -> internal completion buffer

그러나 이것은 controller 내부 학습 모델이다. NVMe Submission Queue와 Completion Queue는 host memory의 queue, pointer, doorbell, phase tag로 동작한다. NVMe SQ/CQ entry가 RTL의 `valid`와 `ready` wire를 host에 직접 노출하는 것은 아니다.

NVMe Completion Queue에 free slot이 없으면 controller가 해당 CQ에 status를 더 게시하지 못하고, 연결된 Submission Queue 처리를 멈출 수 있다. 이것은 signal-level ready/valid와 동일한 프로토콜은 아니지만, **downstream capacity 부족이 upstream progress를 제한한다**는 backpressure 관점으로 비교할 수 있다.

## Request/Acknowledge와의 차이

| 방식 | 핵심 의미 | 주의점 |
| --- | --- | --- |
| valid/ready | 한 clock edge의 transfer 수락 | 매 cycle 연속 transfer 가능 |
| request/acknowledge | 요청 발생과 수락/완료를 사건으로 표현 | ack가 acceptance인지 completion인지 정의 필요 |
| busy/done | 장시간 operation의 실행 상태와 완료 | data transfer handshake와 별개일 수 있음 |

`ack`라는 이름만으로 durable completion인지 단순 접수인지 알 수 없다. Interface contract를 확인해야 한다.

## 작은 1-entry elastic buffer

아래 예시는 한 항목을 보관하면서 downstream stall을 upstream backpressure로 전달한다.

```systemverilog
logic        full_q;
logic [31:0] data_q;

assign s_ready = !full_q || m_ready;
assign m_valid = full_q;
assign m_data  = data_q;

always_ff @(posedge clk or negedge rst_n) begin
  if (!rst_n) begin
    full_q <= 1'b0;
  end else if (s_ready) begin
    full_q <= s_valid;
    if (s_valid)
      data_q <= s_data;
  end
end
```

- Buffer가 비어 있으면 `s_ready=1`이다.
- Buffer가 차 있어도 downstream이 같은 cycle에 가져가면 새 payload로 교체할 수 있다.
- Buffer가 차 있고 downstream이 막히면 `s_ready=0`이 되어 upstream을 멈춘다.

이 코드는 single-clock 학습용 예시다. Clock-domain crossing FIFO, reset sequencing, timing closure 문제를 해결하지 않는다.

## Assertion으로 확인할 성질

다음은 이후 assertion 학습에서 다시 다룰 개념적 property다.

```systemverilog
// Stall 중에는 valid와 payload를 유지한다.
assert property (@(posedge clk) disable iff (!rst_n)
  m_valid && !m_ready |=> m_valid && $stable(m_data));
```

추가로 확인할 조건은 다음과 같다.

- input handshake 수 - output handshake 수가 buffer capacity 범위 안에 있는가?
- 같은 payload가 두 번 transfer되지 않는가?
- 받아들인 payload가 사라지지 않는가?
- ready가 계속 주어질 때 pending payload가 결국 나오는가?
- reset 직후 stale valid가 잘못 올라오지 않는가?

## 자주 발생하는 버그

| 버그 | 결과 | 검증 방법 |
| --- | --- | --- |
| ready가 0인데 valid를 한 cycle만 pulse | command/data drop | random stall 주입 후 scoreboard 비교 |
| stall 중 payload 변경 | 다른 command가 전달됨 | stable-payload assertion |
| valid만 보고 pointer 증가 | transfer되지 않은 entry 소비 | pointer update가 fire와 같은지 확인 |
| ready를 buffer capacity와 무관하게 유지 | FIFO overflow | full condition과 occupancy invariant |
| sender와 receiver가 서로 기다림 | deadlock | bounded progress/timeout test |
| reset 중 한쪽만 초기화 | phantom transfer | reset release sequence test |

## QD, FIFO depth와의 차이

- [[Queue Depth|QD]]: host/test 관점에서 outstanding I/O 수를 나타내는 workload 조건이다.
- FIFO depth: 특정 hardware queue가 저장할 수 있는 entry 수다.
- `valid/ready`: 인접한 두 stage 사이에서 한 항목의 이동을 결정하는 signal protocol이다.

QD32라고 해서 controller 내부 FIFO depth가 32라는 뜻은 아니다. 높은 QD는 내부 여러 queue와 resource에 더 많은 pressure를 줄 수 있지만, 실제 mapping은 controller-specific이다.

## 내 프로젝트와 연결

- [[Request Timing MVP]]의 single FIFO는 request 대기를 cost model로 표현하지만 cycle-accurate ready/valid RTL은 아니다.
- [[Resource Contention MVP]]의 resource wait는 downstream busy가 host I/O와 GC에 주는 지연을 관찰하는 상위 모델이다.
- [[SSD QoS]]와 [[p99 latency]]는 backpressure가 길거나 불공정하게 지속될 때 host-visible 결과로 연결될 수 있다.
- [[Durable Request Replay]]의 ACK는 단순 handshake acceptance가 아니라 storage effect와 recovery contract를 고려한 completion 의미로 구분해야 한다.

## 확인 문제

1. `valid=1, ready=0`일 때 sender가 반드시 유지해야 하는 것은 무엇인가?
2. Sender가 ready를 본 뒤에만 valid를 올리면 왜 deadlock 위험이 생기는가?
3. `valid=1, ready=1`이 세 cycle 지속되면 transfer는 몇 번 발생하는가?
4. QD32와 32-entry FIFO는 왜 같은 말이 아닌가?
5. NVMe Completion Queue full과 ready/valid backpressure는 무엇이 같고 무엇이 다른가?
6. `ack`가 request acceptance인지 durable completion인지 어떻게 구분하는가?

## 내 답변

> [!note]
> 표를 보지 않고 signal의 주체와 transfer 조건부터 말해본다. 이후 간단한 waveform을 직접 그린다.

1.
2.
3.
4.
5.
6.

## 공식 출처와 적용 범위

- [Arm AMBA AXI and ACE Protocol Specification, IHI 0022H](https://developer.arm.com/-/media/Arm%20Developer%20Community/PDF/IHI0022H_amba_axi_protocol_spec.pdf): VALID가 READY에 의존하면 안 된다는 규칙, receiver의 READY timing, channel handshake dependency를 기준으로 사용했다.
- [Arm Introduction to AXI](https://developer.arm.com/community/arm-community-blogs/b/soc-design-and-simulation-blog/posts/introduction-to-axi-protocol-understanding-the-axi-interface): AXI channel과 ready/valid 흐름을 쉽게 설명한 Arm 공식 입문 자료다.
- [NVM Express Base Specification 2.1](https://nvmexpress.org/wp-content/uploads/NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf): Submission/Completion Queue pointer, doorbell, CQ flow-control 동작을 확인했다.
- [NVMe over PCIe Transport Specification](https://nvmexpress.org/specification/nvme-over-pcie-transport-specification/): PCIe transport에서 host memory queue와 controller command processing의 경계를 확인했다.
- SSD 내부 module 사이 ready/valid 적용은 학습용 예시이며 특정 상용 controller의 공개되지 않은 RTL 구조를 주장하지 않는다.

## 관련 노트

- [[00 SSD Controller and Verification 학습 지도]]
- [[01 FSM과 SSD 상태 전이]]
- [[Queue Depth]]
- [[SSD 통신 프로토콜]]
- [[Request Timing MVP]]
- [[Resource Contention MVP]]
- [[p99 latency]]
- [[SSD QoS]]
