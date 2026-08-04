---
title: FSM과 SSD 상태 전이
aliases:
  - SSD FSM
  - Finite State Machine
  - 유한 상태 기계
tags:
  - concept
  - ssd
  - controller
  - fsm
  - digital-design
  - verification
type: learning-module
status: growing
domain: SSD Controller Validation
created: 2026-08-04
updated: 2026-08-04
source_type: official-vendor-guide
reliability: source-checked-learning-note
related_projects:
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - firmware-validation
  - rtl-verification
---

# FSM과 SSD 상태 전이

## 한 문장 정의

FSM(Finite State Machine)은 현재 상태와 입력을 바탕으로 다음 상태와 출력을 결정해, 여러 cycle에 걸친 동작 순서를 제어하는 디지털 회로 모델이다.

## 쉬운 비유

택배 상태를 생각하면 쉽다.

```text
주문 접수 -> 상품 준비 -> 배송 중 -> 배송 완료
                    \-> 배송 실패 -> 재시도 또는 반송
```

택배가 동시에 `상품 준비`와 `배송 완료`일 수 없듯이, 단순 FSM은 한 시점의 제어 상태를 명확히 표현한다. 중요한 것은 상태 이름보다 **어떤 조건에서 다음 상태로 이동하고, 실패하면 어디로 가는가**다.

## 하드웨어에서의 의미

AMD Vivado 문서는 FSM을 다음 세 부분으로 설명한다.

1. **State register:** clock edge마다 current state를 저장한다.
2. **Next-state function:** current state와 input으로 next state를 계산한다.
3. **Output function:** state 또는 state와 input으로 output을 만든다.

```text
                +---------------------+
input --------> | next-state logic    | ----> next_state
current_state ->|                     |
                +---------------------+
                           |
                           v clock edge
                     +------------+
reset -------------->| state reg  | ----> current_state
                     +------------+
                           |
                           v
                     output logic
```

### Current state와 next state

- `current_state`: 현재 clock cycle에서 register가 보존하고 있는 상태다.
- `next_state`: combinational logic이 계산한 다음 후보 상태다.
- 실제 상태 변경은 보통 다음 active clock edge에서 일어난다.

### Moore와 Mealy

| 구분 | 출력이 의존하는 것 | 특징 |
| --- | --- | --- |
| Moore FSM | current state | 출력 변화가 상태 변화와 맞물려 이해하기 쉽다 |
| Mealy FSM | current state + input | 입력에 더 빠르게 반응할 수 있지만 조합 경로와 glitch를 더 조심한다 |

실제 RTL은 두 스타일을 섞을 수 있다. 이름을 외우는 것보다 output이 register를 거치는지, input 변화가 같은 cycle에 output을 바꾸는지를 읽는 것이 중요하다.

## SSD controller에서의 위치

SSD controller 전체가 거대한 FSM 하나인 것은 아니다. 일반적으로 여러 module이 각자의 상태를 갖고 동시에 진행한다.

- command fetch/control FSM
- DMA descriptor/data transfer FSM
- NAND read/program/erase control FSM
- metadata commit/recovery FSM
- reset/error recovery FSM

다음은 한 write request를 이해하기 위한 **학습용 control flow**다.

```text
RESET
  -> IDLE
  -> FETCH_COMMAND
  -> VALIDATE_COMMAND
  -> TRANSLATE_LPN
  -> RESERVE_RESOURCE
  -> TRANSFER_DATA
  -> PROGRAM_NAND
  -> PERSIST_METADATA
  -> COMPLETE
  -> IDLE
```

이 흐름은 공개된 특정 SSD firmware 구조가 아니다. 실제 구현에서는 여러 request가 pipeline으로 겹치고, data와 metadata 경로가 분리되며, command가 out-of-order로 완료될 수도 있다.

## 상태별 질문

| 상태 | 하는 일 | 기다리는 조건 | 대표 실패 |
| --- | --- | --- | --- |
| FETCH_COMMAND | command 정보를 가져옴 | queue entry / DMA 완료 | malformed command, fetch timeout |
| VALIDATE_COMMAND | opcode, range, 권한 확인 | validation 결과 | invalid LBA, unsupported command |
| TRANSLATE_LPN | logical address와 mapping 확인 | mapping lookup | stale/corrupt metadata |
| RESERVE_RESOURCE | free page, buffer, channel 확보 | allocator/grant | no free page, GC pressure |
| TRANSFER_DATA | host/internal buffer 사이 data 이동 | DMA/handshake 완료 | timeout, short transfer |
| PROGRAM_NAND | 새 physical page program | NAND completion | program failure |
| PERSIST_METADATA | mapping/commit evidence 보존 | durability 조건 | torn metadata, power cut |
| COMPLETE | host-visible completion 생성 | CQ space/response path | lost completion, CQ full |

## Overwrite에서 상태 순서가 중요한 이유

기존 `LPN X -> old PPN`이 있을 때 새 PPN에 데이터를 썼다고 곧바로 모든 작업이 끝난 것은 아니다.

```text
1. new PPN 확보
2. new data program
3. new data 검증 또는 program completion 확인
4. mapping commit
5. old PPN을 더 이상 authoritative하지 않은 상태로 처리
6. 약속한 durability 조건을 만족한 뒤 host completion
```

Power cut이 어느 지점에서 발생해도 recovery가 old 또는 new version 중 하나의 일관된 상태를 선택할 수 있어야 한다. `mapping update`, `old page invalidation`, `host ACK`의 정확한 순서는 metadata protocol과 persistence model에 따라 달라지므로 단순 FSM 그림만으로 정답을 단정하지 않는다.

핵심 invariant는 다음과 같다.

- Host에 성공을 알린 write는 약속한 persistence 범위 안에서 복구 가능해야 한다.
- 하나의 LPN에 대한 authoritative version이 모호해지면 안 된다.
- 실패한 allocation과 program의 resource가 조용히 유실되면 안 된다.

## Reset, timeout, recovery state

### Reset state

Reset은 state register를 알려진 시작점으로 돌린다. Intel의 FSM 설계 지침은 defined power-up state와 reset을 권장하며, illegal state가 발생할 가능성이 있는 설계에서는 safe-state 처리와 asynchronous input synchronization을 별도로 고려해야 한다고 설명한다.

Reset을 받았다고 외부 NAND와 persistent metadata까지 자동으로 초기화되는 것은 아니다. Controller logic reset과 persistent state recovery는 서로 다른 문제다.

### Timeout state

Timeout은 “느리다”가 아니라 **기다림에 허용된 경계를 넘었다**는 제어 사건이다.

```text
WAIT_NAND_DONE
  -> done이면 다음 상태
  -> timer 만료면 TIMEOUT
  -> retry / retire / fail / reset 중 정책 선택
```

Timeout 뒤 늦게 도착한 completion을 그대로 받아들이면 이미 재사용된 request ID나 buffer를 잘못 갱신할 수 있다. Epoch, request ID, ownership 같은 식별 경계가 필요한 이유다.

### Illegal state

State encoding에 속하지 않는 값이나 허용되지 않은 조합에 들어간 상태다. `default: state = RESET`만 써두는 것으로 모든 회복성이 증명되지는 않는다. Synthesis optimization, clock-domain crossing, 다른 register의 일관성, persistent side effect를 함께 봐야 한다.

## 작은 SystemVerilog 뼈대

아래 코드는 구조를 읽기 위한 축약 예시이며 완전한 SSD write engine이 아니다.

```systemverilog
typedef enum logic [2:0] {
  ST_RESET,
  ST_IDLE,
  ST_PROGRAM,
  ST_COMMIT,
  ST_COMPLETE,
  ST_ERROR
} state_t;

state_t state_q, state_d;

always_ff @(posedge clk or negedge rst_n) begin
  if (!rst_n)
    state_q <= ST_RESET;
  else
    state_q <= state_d;
end

always_comb begin
  state_d = state_q;

  unique case (state_q)
    ST_RESET:    state_d = ST_IDLE;
    ST_IDLE:     if (cmd_fire)    state_d = ST_PROGRAM;
    ST_PROGRAM:  if (program_err) state_d = ST_ERROR;
                 else if (program_done) state_d = ST_COMMIT;
    ST_COMMIT:   if (commit_err)  state_d = ST_ERROR;
                 else if (commit_done)  state_d = ST_COMPLETE;
    ST_COMPLETE: if (cpl_fire)    state_d = ST_IDLE;
    ST_ERROR:    if (recovered)   state_d = ST_IDLE;
    default:                       state_d = ST_ERROR;
  endcase
end
```

코드를 볼 때 다음을 확인한다.

- `program_done` 전에 commit으로 갈 수 있는가?
- `commit_done` 전에 completion을 보낼 수 있는가?
- `ST_ERROR`에서 resource와 request ownership을 어떻게 정리하는가?
- `cmd_fire`, `cpl_fire`가 정말 handshake 성립을 의미하는가?
- 영원히 빠져나오지 못하는 상태가 있는가?

## 실패 사례와 검증 포인트

| 실패 사례 | 보이는 증상 | 검증 아이디어 |
| --- | --- | --- |
| 상태 전이 누락 | request hang, timeout | 모든 legal state의 exit 조건 확인 |
| 너무 이른 completion | power cut 뒤 acknowledged data 소실 | ACK 이전 각 지점에 fault injection |
| error에서 resource 미반납 | 시간이 갈수록 free page/buffer 감소 | error 후 resource conservation invariant |
| late completion 수용 | 다른 request의 state 오염 | request ID/epoch mismatch test |
| reset 중 새 요청 수용 | partial initialization | reset 동안 input 차단 assertion |
| illegal state 회복만 하고 side effect 방치 | mapping/reverse map 불일치 | recovery 후 전체 invariant 검사 |

## 내 프로젝트와 연결

- [[FTL]]의 out-of-place write를 단순 함수 호출이 아니라 여러 실패 가능한 상태 전이로 다시 본다.
- [[FTL Metadata Recovery and Bad Block Handling]]의 PREPARE/COMMIT과 power-cut probe는 metadata FSM의 fault transition으로 해석할 수 있다.
- [[Durable Request Replay]]의 request ID와 replay ledger는 reset/timeout 뒤 늦은 completion 및 duplicate request를 구분하는 상태다.
- [[Controller Lease and External Fencing]]은 단일 FSM 바깥에서 mutation authority 자체를 제한한다.

## 확인 문제

1. `current_state`와 `next_state`는 언제 달라지고, 실제 register 값은 언제 바뀌는가?
2. `PROGRAM_NAND -> COMPLETE`로 바로 이동하면 power cut에서 어떤 문제가 생길 수 있는가?
3. Controller reset과 persistent metadata recovery는 왜 같은 일이 아닌가?
4. Timeout 이후 늦게 도착한 completion을 구분하려면 어떤 정보가 필요한가?
5. Moore와 Mealy 중 하나가 항상 더 좋은 설계인가?

## 내 답변

> [!note]
> 위 질문에 먼저 내 말로 답한다. 막힌 문장은 [[00 SSD Controller and Verification 학습 지도|SSD Controller and Verification 학습 지도]]의 기존 프로젝트 연결표를 따라가며 보완한다.

1.
2.
3.
4.
5.

## 공식 출처와 적용 범위

- [AMD Vivado Synthesis UG901 - FSM Description](https://docs.amd.com/r/2022.2-English/ug901-vivado-synthesis/FSM-Description): state register, next-state function, output function과 Moore/Mealy 지원을 확인했다.
- [Intel Quartus Design Recommendations - State Machine HDL Guidelines](https://www.intel.com/content/www/us/en/docs/programmable/683082/25-1/state-machine-hdl-guidelines.html): reset, latch 방지, illegal state와 safe-state 관련 구현 지침을 참고했다.
- [Intel Stratix 10 Configuration Guide - Protecting State Machine Logic](https://www.intel.com/content/www/us/en/docs/programmable/683762/25-1/protecting-state-machine-logic.html): reset release가 불완전할 때 one-hot FSM이 illegal state에 들어갈 수 있는 사례를 참고했다.
- SSD write 상태 흐름과 failure case는 위 공식 FSM 원리를 [[FTL]]과 현재 white-box project에 적용한 학습용 모델이며, 특정 상용 controller 구현을 묘사하지 않는다.

## 관련 노트

- [[00 SSD Controller and Verification 학습 지도]]
- [[02 Valid Ready Handshake와 Backpressure]]
- [[FTL]]
- [[LBA LPN PPN]]
- [[FTL Metadata Recovery and Bad Block Handling]]
- [[Durable Request Replay]]

