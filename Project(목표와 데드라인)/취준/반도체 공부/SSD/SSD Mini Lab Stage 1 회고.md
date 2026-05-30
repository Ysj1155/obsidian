---
title: SSD Mini Lab Stage 1 회고
aliases:
  - fio 기반 SSD 검증 미니랩 1단계 회고
tags:
  - til
  - ssd
  - ssd-validation
  - fio
  - portfolio
type: review
status: growing
domain: SSD Validation
created: 2026-05-30
updated: 2026-05-30
related_projects:
  - ssd-mini-lab
related_roles:
  - ssd-validation
  - test-engineering
  - product-engineering
---

# SSD Mini Lab Stage 1 회고

## 한 줄 요약

`fio`를 이용해서 SSD 성능을 단순히 측정하는 것을 넘어서, 조건을 정의하고, 결과를 파싱하고, 그래프로 보고, 반복성과 한계를 함께 해석하는 작은 검증 흐름을 만들었다.

관련 프로젝트:

- GitHub: [Ysj1155/ssd-mini-lab](https://github.com/Ysj1155/ssd-mini-lab)
- 로컬 프로젝트: `D:\ssd_lab`
- Stage 1 Review: `D:\ssd_lab\docs\reports\stage1_review.md`

## 이번 단계에서 한 일

| 구분 | 내용 |
|---|---|
| Baseline | `seq_read`, `seq_write`, `rand_read`, `rand_write` 4가지 workload 측정 |
| Parser | fio JSON을 CSV로 변환하는 `parse_fio_results.py` 작성 및 보강 |
| Plot | bandwidth, IOPS, p99 latency, run-to-run variation 그래프 생성 |
| QD sweep | QD 1, 4, 16, 32에서 random read/write 비교 |
| Reproducibility | 반복 측정 결과를 평균, 표준편차, CV로 정리 |
| Direct vs Buffered | `direct=1`과 `direct=0` 비교 |
| Documentation | README, 개별 리포트, Stage 1 Review 작성 |

## 핵심 결과

### 1. Baseline

- `rand_read`는 `rand_write`보다 IOPS가 높고 p99 latency가 낮았다.
- sequential workload는 bandwidth가 안정적으로 나왔지만, p99 latency는 반복 측정 간 변동이 더 컸다.
- 평균 성능만 보면 놓치는 정보가 있으므로 p99 latency를 함께 봐야 한다.

### 2. Queue Depth

- QD가 증가하면 병렬성이 올라가 IOPS가 증가할 수 있다.
- 하지만 QD가 높아질수록 p99 latency도 커질 수 있다.
- `rand_write QD32`는 처리량 이득은 크지 않은데 p99 latency가 크게 증가했다.
- 따라서 최고 IOPS 조건이 항상 좋은 조건은 아니다.

### 3. Reproducibility

- 반복 측정을 통해 단일 run의 우연성을 줄일 수 있었다.
- CV를 사용하면 같은 조건에서 결과가 얼마나 흔들리는지 볼 수 있다.
- tail latency는 평균 throughput보다 더 민감하게 흔들릴 수 있다.

### 4. Direct vs Buffered

- `direct=0`은 `direct=1`보다 더 높은 IOPS와 bandwidth를 보였다.
- 하지만 이것은 SSD media 자체가 더 빨라졌다는 뜻이 아니다.
- Windows filesystem/page cache 영향이 포함되었을 가능성이 크다.
- 검증 목적에서는 `direct=1`이 OS cache 영향을 줄이는 데 더 적합하다.

## 내가 배운 검증 관점

### 테스트 조건도 결과의 일부다

결과 숫자만 저장하면 나중에 해석하기 어렵다.

반드시 같이 남겨야 할 것:

- workload
- block size
- iodepth
- direct mode
- runtime
- run number
- fio version
- test path
- OS/filesystem 영향 가능성

### 평균값만 보면 위험하다

평균 bandwidth나 평균 IOPS는 보기 쉽지만, SSD validation에서는 p95/p99/p99.9 latency가 중요하다.

특히 QoS 관점에서는 "빠른가"보다 "일관되게 안정적인가"가 더 중요할 수 있다.

### 캐시 효과는 결과를 좋아 보이게 만들 수 있다

`direct=0` 결과가 좋아 보여도 이것을 곧바로 SSD 자체 성능으로 해석하면 안 된다.

host OS, filesystem, page cache, write buffering 같은 요소가 개입할 수 있다.

### 파서 버그는 분석 버그가 된다

`rand_read_direct0_run1.json` 같은 파일명을 잘못 파싱하면 workload가 `rand_read_direct0`처럼 갈라진다.

그러면 실제로는 같은 workload를 비교하는 실험인데, 분석에서는 서로 다른 workload처럼 처리될 수 있다.

검증 자동화에서는 파일명 규칙, 메타데이터, 실제 fio job option을 모두 조심해야 한다.

## 면접에서 말할 수 있는 문장

> fio 기반 SSD mini-lab을 만들면서 baseline workload, queue depth sweep, 반복 측정 안정성, direct I/O와 buffered I/O 차이를 분석했습니다. 단순 평균 IOPS보다 p99 latency와 run-to-run variation이 검증 관점에서 중요하다는 것을 확인했고, 특히 buffered I/O 결과는 OS cache 영향 때문에 SSD media 성능으로 바로 해석하면 안 된다는 점을 배웠습니다.

조금 더 기술적으로:

> 처음에는 fio JSON을 CSV로 정리하는 parser를 만들었고, 이후 QD sweep과 direct/buffered 실험을 추가하면서 filename metadata를 `workload`, `run`, `qd`, `direct`로 분리하도록 보강했습니다. 이를 통해 결과 파일명과 fio job option을 함께 보존했고, 조건별 평균, 표준편차, CV, p99 latency를 비교할 수 있게 했습니다.

## 예상 면접 질문

### Q. fio에서 `direct=1`은 왜 쓰나요?

OS page cache 영향을 줄이고 storage path에 더 가까운 결과를 보기 위해 사용한다.

다만 `direct=1`을 사용해도 테스트 경로, filesystem, OS, device 연결 방식에 따라 완전히 순수한 device 성능이라고 단정할 수는 없다.

### Q. buffered I/O 결과가 더 좋게 나왔는데 왜 조심해야 하나요?

`direct=0`에서는 OS page cache나 filesystem buffering이 개입할 수 있다.

그래서 read는 cache hit 영향, write는 write-back/buffering 영향으로 실제 media completion보다 좋아 보일 수 있다.

### Q. QD를 높이면 항상 좋은가요?

아니다.

QD를 높이면 병렬성이 증가해 IOPS가 올라갈 수 있지만, 동시에 queueing delay가 늘어나 p99 latency가 나빠질 수 있다.

### Q. 왜 p99 latency를 봐야 하나요?

평균 latency는 대부분의 요청이 어떤 수준인지 보여주지만, validation에서는 tail latency가 중요하다.

일부 요청이 매우 늦게 끝나는 현상은 평균값에 잘 드러나지 않을 수 있다.

### Q. 반복 측정은 왜 필요한가요?

한 번의 결과는 우연한 시스템 상태, cache 상태, background I/O, SSD 내부 상태의 영향을 받을 수 있다.

반복 측정을 해야 결과가 안정적인 패턴인지 판단할 수 있다.

## 현재 한계

| 한계 | 의미 |
|---|---|
| Windows file-based fio | OS/filesystem 영향이 포함됨 |
| 외장 SSD/USB 경로 | USB bridge, enclosure, cable, port 영향 가능 |
| 반복 3회 | 첫 분석으로는 충분하지만 통계적으로 강한 결론은 어려움 |
| 30초 runtime | sustained workload, thermal throttling, 장기 QoS를 보기에는 짧음 |
| SMART/NVMe telemetry 없음 | 온도, error, throttling, media 상태를 직접 확인하지 못함 |
| raw block device 미사용 | 개인 노트북 안전을 위해 낮은 수준의 device validation은 제외 |

## 다음에 할 일

1. Stage 1 Review를 한글 발표/면접용으로 다시 요약한다.
2. WSL/Linux 환경 정보를 수집하는 스크립트를 만든다.
3. Windows path와 WSL path의 차이를 안전한 범위에서 비교한다.
4. p95/p99/p99.9 중심의 QoS 리포트를 만든다.
5. 장시간 sustained workload 실험은 전용 테스트 SSD가 생기면 진행한다.

## 개인 회고

처음에는 fio를 돌려서 숫자를 얻는 것이 목표처럼 보였다.

하지만 진행하면서 중요한 것은 숫자 자체보다 그 숫자가 나온 조건과 해석이라는 것을 알게 되었다.

특히 direct/buffered 실험에서 `direct=0` 결과가 더 좋아 보였지만, 이것을 그대로 SSD 성능이라고 말하면 위험하다는 점이 인상적이었다.

SSD validation engineer 관점에서는 "얼마나 빠른가"뿐 아니라 "어떤 조건에서 측정했는가", "결과가 반복 가능한가", "어떤 계층의 영향이 섞였는가"를 설명할 수 있어야 한다.

이번 Stage 1은 그 관점을 연습한 첫 번째 사이클이다.