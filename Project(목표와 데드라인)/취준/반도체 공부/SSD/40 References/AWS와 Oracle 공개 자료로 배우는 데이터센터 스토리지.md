---
title: AWS와 Oracle 공개 자료로 배우는 데이터센터 스토리지
aliases:
  - 데이터센터 스토리지 학습 가이드
  - Cloud Storage Study Guide
tags:
  - reference
  - study-guide
  - storage
  - cloud-storage
  - aws
  - oracle
  - ssd-validation
type: study-guide
status: ready
domain: Data Center Storage
created: 2026-07-30
updated: 2026-07-30
source_type: official-public-material
reliability: official-documentation
related_projects:
  - ssd-mini-lab
  - ftl-gc-whitebox-lab
related_roles:
  - ssd-validation
  - product-engineering
  - firmware-validation
  - test-engineering
---

# AWS와 Oracle 공개 자료로 배우는 데이터센터 스토리지

## 이 문서의 목적

AWS와 Oracle이 공개한 공식 자료를 이용해 데이터센터 스토리지를 다음 관점으로 공부한다.

- 스토리지 성능은 어떤 계층의 한계로 결정되는가
- IOPS, throughput, latency, queue depth는 어떻게 연결되는가
- 성능 문제의 원인이 volume, host, network, OS 중 어디에 있는지 어떻게 구분하는가
- 같은 스토리지도 초기 상태와 데이터 상태에 따라 왜 다른 성능을 보이는가
- local NVMe의 높은 성능과 remote block storage의 내구성은 어떻게 다른가
- 데이터 손실과 느린 I/O를 어떤 방식으로 검증하는가
- 데이터 이동을 줄이기 위해 compute와 storage를 어떻게 배치하는가

이 문서는 AWS나 Oracle 제품의 기능 목록을 외우기 위한 자료가 아니다. 현재 진행 중인 [[SSD Mini Lab 프로젝트 허브]]와 [[SSD FTL-GC White-box Validation Lab]]을 데이터센터 수준의 검증 문제로 확장하는 것이 목적이다.

---

## 먼저 기억할 전체 모델

클라우드에서 관찰되는 storage 성능은 SSD media 하나로 결정되지 않는다.

```text
Application
    ↓
Filesystem / OS cache
    ↓
Block layer / I/O scheduler
    ↓
Virtual device / NVMe driver
    ↓
Instance storage bandwidth
    ↓
Network / storage protocol
    ↓
Remote block storage service
    ↓
Physical SSD / replication layer
```

대략적인 성능 상한은 다음처럼 생각할 수 있다.

```text
관찰 성능
= min(
    workload가 만드는 I/O demand,
    volume IOPS 한도,
    volume throughput 한도,
    instance IOPS/throughput 한도,
    network 또는 attachment 한도
  )
+ queueing, OS, filesystem, cache, device state의 영향
```

따라서 fio 결과가 낮게 나왔다는 사실만으로 물리 SSD가 느리다고 결론 내릴 수 없다.

---

# 권장 학습 순서

## 1단계. Cloud Block Storage Performance Model

### 먼저 읽을 자료

1. [Amazon EBS I/O characteristics and monitoring](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-io-characteristics.html)
2. [Benchmark Amazon EBS volumes](https://docs.aws.amazon.com/ebs/latest/userguide/benchmark_procedures.html)
3. [OCI Block Volume Performance](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/blockvolumeperformance.htm)
4. [Oracle: How to measure disk performance on OCI Block Volume](https://blogs.oracle.com/cloud-infrastructure/how-to-measure-disk-performance-on-oracle-cloud-block-volume)

### 배워야 할 내용

#### IOPS와 throughput은 별개의 한도다

IOPS는 초당 I/O 요청 수이고 throughput은 초당 전송한 데이터 양이다.

```text
대략적인 throughput = IOPS × I/O size
```

예를 들어 I/O size가 커지면 낮은 IOPS에서도 throughput 한도에 먼저 도달할 수 있다. 따라서 서로 다른 block size의 IOPS를 하나의 leaderboard처럼 비교하면 안 된다.

이 내용은 다음 노트와 직접 연결된다.

- [[Queue Depth]]
- [[p99 latency]]
- [[External SSD Block Size Sweep]]
- [[왜 평균 IOPS만 보면 안 되는가]]

#### Queue Depth는 성능 점수가 아니라 부하 조건이다

QD가 너무 낮으면 storage 병렬성을 충분히 사용하지 못할 수 있다. 반대로 QD가 너무 높으면 처리량 이득 없이 queueing delay와 tail latency만 커질 수 있다.

확인해야 할 것은 최고 IOPS가 아니라 다음 trade-off다.

```text
QD 증가
→ outstanding request 증가
→ device utilization 증가 가능
→ IOPS 증가 가능
→ queueing delay와 tail latency 증가 가능
```

#### Volume 한도와 Instance 한도를 분리한다

Cloud block storage에는 적어도 두 개의 상한이 존재할 수 있다.

- 개별 volume이 제공하는 IOPS와 throughput
- instance가 모든 attached volume에 제공할 수 있는 aggregate bandwidth

여러 volume을 묶어도 instance 한도에 도달하면 성능이 더 이상 증가하지 않는다.

### 읽고 나서 답할 질문

1. 4K random workload는 왜 IOPS 중심으로 보는가?
2. 1M sequential workload는 왜 throughput 한도에 먼저 도달하기 쉬운가?
3. QD를 높였는데 IOPS가 증가하지 않고 p99만 악화된다면 어떤 한도를 의심할 수 있는가?
4. 같은 volume을 더 큰 instance에 연결했을 때 성능이 달라질 수 있는 이유는 무엇인가?
5. raw device 결과와 filesystem 위의 결과가 다른 이유는 무엇인가?

### 내 프로젝트에 적용할 것

- `IOPS × block size`로 관찰 throughput을 계산한다.
- volume/device 한도와 host/path 한도를 분리한 가설을 작성한다.
- QD sweep을 `IOPS`, `p99`, `p99.9`, `CV`의 trade-off로 다시 해석한다.
- block-size sweep에서 throughput knee가 생긴 계층의 후보를 나열한다.

---

## 2단계. Storage Observability와 병목 분리

### 먼저 읽을 자료

1. [Amazon EBS detailed performance statistics](https://docs.aws.amazon.com/ebs/latest/userguide/nvme-detailed-performance-stats.html)
2. [Understanding and monitoring latency for Amazon EBS](https://aws.amazon.com/blogs/storage/understanding-and-monitoring-latency-for-amazon-ebs-volumes-using-amazon-cloudwatch/)
3. [OCI Block Volume Metrics Reference](https://docs.oracle.com/en-us/iaas/Content/Block/References/volumemetrics-reference.htm)

### 배워야 할 내용

AWS EBS는 Nitro 기반 instance에서 다음과 같은 storage 통계를 제공한다.

- read/write operation 수
- read/write byte 수
- read/write에 소비한 시간
- volume의 provisioned IOPS 초과 시간
- volume의 provisioned throughput 초과 시간
- instance의 EBS IOPS 초과 시간
- instance의 EBS throughput 초과 시간
- volume queue length
- read/write latency histogram

핵심은 `느렸다`는 관찰을 다음과 같이 세분화할 수 있다는 점이다.

```text
Latency 증가
├─ volume IOPS 한도 초과
├─ volume throughput 한도 초과
├─ instance EBS 한도 초과
├─ queue length 증가
├─ OS/filesystem/cache 영향
├─ application demand 변화
└─ 관찰하지 못한 storage 내부 요인
```

### Observer Matrix

| 관찰자 | 볼 수 있는 것 | 직접 말할 수 없는 것 |
|---|---|---|
| Application | transaction 시간, timeout, error | storage 내부 원인 |
| fio | IOPS, bandwidth, completion latency | 물리 GC 또는 NAND 상태 |
| OS block layer | queue, utilization, device 대기 | remote service 내부 동작 |
| NVMe/EBS statistics | queue, limit 초과, latency histogram | application 의미와 사용자 영향 |
| Cloud monitoring | 시간에 따른 metric과 alarm | 개별 request의 상세 인과관계 |
| SMART/telemetry | 온도, error, health 상태 | host부터 media까지의 전체 latency |

### 읽고 나서 답할 질문

1. fio p99가 증가했을 때 어떤 추가 metric이 있어야 원인을 좁힐 수 있는가?
2. volume limit 초과와 instance limit 초과는 어떻게 다른가?
3. 1분 평균 metric이 정상이어도 짧은 microburst를 놓칠 수 있는 이유는 무엇인가?
4. latency histogram이 평균 latency보다 유용한 상황은 무엇인가?
5. observer가 nonzero activity를 잡지 못했다면 correctness 결과까지 무효가 되는가?

### 내 프로젝트에 적용할 것

- [[External SSD Data Integrity]]의 `correctness Pass`와 `observer Limited`를 다시 비교한다.
- 각 실험 리포트에 `관찰 계층`과 `관찰하지 못한 계층`을 추가한다.
- 내부 GC, thermal throttling, SLC cache를 단정하기 전에 필요한 telemetry를 명시한다.

---

## 3단계. Snapshot Initialization과 State-dependent Performance

### 먼저 읽을 자료

1. [Initialize Amazon EBS volumes](https://docs.aws.amazon.com/ebs/latest/userguide/initalize-volume.html)
2. [Addressing I/O latency when restoring EBS volumes from snapshots](https://aws.amazon.com/blogs/storage/addressing-i-o-latency-when-restoring-amazon-ebs-volumes-from-ebs-snapshots/)

### 배워야 할 내용

EBS snapshot으로 생성한 volume은 처음 접근하는 block을 backing storage에서 가져와야 한다. 초기화되지 않은 block에 처음 접근하면 latency가 증가하고 성능이 낮아질 수 있다.

```text
Snapshot에서 volume 생성
→ block metadata는 존재
→ 최초 접근 시 실제 block fetch
→ cold-read latency 발생 가능
→ 전체 block 초기화 후 정상 성능
```

이 사례는 storage 상태가 테스트 결과의 일부라는 사실을 잘 보여준다.

상태 조건의 예:

- 처음 생성한 volume인지
- snapshot에서 복구한 volume인지
- 접근한 적 없는 cold block인지
- 이전 workload가 데이터를 채운 상태인지
- cache가 warm 상태인지
- SSD가 preconditioned 또는 steady state인지

### 읽고 나서 답할 질문

1. 같은 fio 명령을 실행했는데 첫 번째 run만 느릴 수 있는 이유는 무엇인가?
2. volume initialization과 SSD preconditioning은 같은 개념인가, 다른 개념인가?
3. cold state를 성능 결함으로 잘못 판단하지 않으려면 무엇을 기록해야 하는가?
4. warm-up run을 분석에서 제외한다면 그 기준은 무엇이어야 하는가?

### 내 프로젝트에 적용할 것

- 테스트 전 device/path/cache 상태를 manifest에 추가한다.
- 첫 실행과 반복 실행을 별도로 비교한다.
- reconnect, idle, sequential fill 이후 결과가 달라지는지 확인한다.
- 아직 다루지 못한 `preconditioning`과 `steady state` 학습으로 연결한다.

---

## 4단계. Local NVMe와 Durable Remote Block Storage

### 먼저 읽을 자료

1. [Oracle OCI: Protecting Data on NVMe Devices](https://docs.oracle.com/en-us/iaas/Content/Compute/References/nvmedeviceinformation.htm)
2. [OCI Block Volume Overview](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/overview.htm)
3. [Oracle: Decide Your Storage Solution](https://docs.oracle.com/en/solutions/oci-best-practices/decide-your-storage-solution.html)

### 배워야 할 내용

#### Local NVMe

- compute instance에 직접 연결된다.
- latency가 낮고 높은 성능을 제공할 수 있다.
- instance 또는 device failure가 데이터 손실로 이어질 수 있다.
- RAID, replication, backup과 복구 책임이 사용자에게 남을 수 있다.

#### Remote Block Volume

- compute와 storage 수명을 분리할 수 있다.
- instance가 종료되어도 volume을 유지할 수 있다.
- storage service가 replication과 repair 기능을 제공할 수 있다.
- network, attachment, instance bandwidth가 I/O path에 포함된다.

### 비교 기준

| 기준 | Local NVMe | Remote Block Volume |
|---|---|---|
| Latency | 일반적으로 더 낮음 | network/service path 포함 |
| 성능 변동 요인 | local device와 host | volume, instance, network, service |
| Instance 종료 | 데이터 손실 가능성 고려 | volume 수명을 분리 가능 |
| Replication | 사용자가 설계 | service가 제공할 수 있음 |
| Backup | 사용자가 설계 | snapshot/backup 기능 사용 가능 |
| Failure domain | device/host와 밀접 | storage service로 분리 |
| 적합한 데이터 | 재생성 가능 데이터, cache, scratch | database, persistent application data |

### 읽고 나서 답할 질문

1. 더 빠른 장치가 항상 더 좋은 storage 선택이 아닌 이유는 무엇인가?
2. local NVMe를 database에 사용할 경우 어떤 보호 장치가 필요한가?
3. RAID는 device failure와 Availability Domain failure를 모두 해결하는가?
4. 재생성할 수 있는 데이터와 반드시 보존해야 하는 데이터는 storage 선택이 어떻게 달라지는가?
5. durability와 availability는 어떻게 다른가?

### White-box Lab과 연결

- device failure와 controller failure를 구분한다.
- spare promotion과 durable data copy를 구분한다.
- local recovery와 외부 replication을 구분한다.
- [[FTL Metadata Recovery and Bad Block Handling]]의 범위와 cloud-level durability의 범위를 비교한다.

---

## 5단계. Storage Fault Injection과 Resilience Validation

### 먼저 읽을 자료

1. [Amazon EBS Latency Injection](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-fis-latency-injection.html)
2. [Test and build application resilience using Amazon EBS latency injection](https://aws.amazon.com/blogs/storage/test-and-build-application-resilience-using-amazon-ebs-latency-injection/)

### 배워야 할 내용

정상 성능 테스트는 storage가 느려졌을 때 application이 어떻게 실패하는지 보여주지 못한다.

Fault injection 실험은 다음 순서를 따른다.

```text
1. 정상 상태 정의
2. 실패 가설 작성
3. 제한된 범위에 fault 주입
4. application, storage, monitoring 관찰
5. guardrail 또는 중단 조건 확인
6. 복구 후 일관성과 데이터 무결성 확인
7. 가설 수용 또는 기각
```

주입할 수 있는 storage failure 예:

- read latency 증가
- write latency 증가
- 일부 I/O만 지연
- I/O pause
- throughput 제한
- volume 또는 path unavailable

### 읽고 나서 답할 질문

1. 평균 latency 3ms 증가가 application timeout에 어떤 영향을 줄 수 있는가?
2. 모든 I/O를 중단하는 것과 일부 I/O를 느리게 하는 실험은 무엇이 다른가?
3. fault가 너무 커서 시스템이 즉시 중단되면 어떤 정보를 얻지 못하는가?
4. latency가 정상으로 돌아온 뒤 어떤 데이터 일관성 검증이 필요한가?
5. 실험을 자동으로 중단할 guardrail은 무엇이어야 하는가?

### White-box Lab과 연결

- [[Durable Request Replay]]: completion 유실과 retry에서 중복 mutation 방지
- [[Controller Lease and External Fencing]]: 오래된 controller의 mutation authority 차단
- [[FTL Metadata Recovery and Bad Block Handling]]: failure 이후 복구 상태 검증
- [[Request Timing Policy Findings]]: 정상 성능뿐 아니라 rare tail cost 관찰

---

## 6단계. Object Storage Durability와 Data Integrity

### 먼저 읽을 자료

1. [Amazon S3 Data Protection](https://docs.aws.amazon.com/AmazonS3/latest/userguide/DataDurability.html)
2. [Checking object integrity in Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/checking-object-integrity.html)

### 배워야 할 내용

Object storage의 내구성은 단일 SSD의 내구성과 다른 계층의 문제다.

S3의 공개 설계에서 볼 수 있는 핵심은 다음과 같다.

- 여러 장치와 Availability Zone에 데이터를 중복 저장
- redundancy 손실을 감지하고 repair
- checksum을 사용한 전송 및 저장 데이터의 무결성 확인
- versioning 등을 통한 accidental overwrite/delete 보호

구분해야 할 failure:

```text
Physical device failure
≠ silent data corruption
≠ network transfer corruption
≠ application이 잘못된 데이터를 정상적으로 기록
≠ 사용자의 accidental delete
≠ Availability Zone failure
```

하나의 checksum이나 replication 방식이 모든 failure를 해결하지 않는다.

### 읽고 나서 답할 질문

1. replication과 checksum은 각각 어떤 failure를 탐지하거나 복구하는가?
2. 정상적으로 복제된 잘못된 데이터는 어떻게 복구할 수 있는가?
3. durability가 높아도 backup 또는 versioning이 필요한 이유는 무엇인가?
4. upload checksum Pass는 장기 보존까지 증명하는가?
5. [[External SSD Data Integrity]]의 CRC32C 실험은 위 failure 중 어디까지 확인했는가?

---

## 7단계. Oracle Exadata와 Compute Near Data

### 먼저 읽을 자료

1. [Oracle Exadata Smart Scan](https://www.oracle.com/database/technologies/exadata/software/smartscan/)
2. [Oracle Exadata Architecture](https://www.oracle.com/br/database/technologies/exadata/architecture/)
3. [Exadata Smart Flash Cache Series](https://blogs.oracle.com/exadata/exadata-technology-series-smart-flash-cache-part-i-a-recap)

### 배워야 할 내용

전통적인 구조에서는 storage에서 많은 데이터를 database server로 보낸 다음, database server가 필요 없는 행과 열을 제거할 수 있다.

Exadata Smart Scan은 가능한 처리 일부를 storage server로 보내고 필요한 데이터만 database server로 돌려보낸다.

```text
일반 구조
Storage → 많은 데이터 전송 → Database가 filter

Compute near data
Storage가 일부 filter/processing → 필요한 데이터만 전송
```

핵심 설계 원리:

- 데이터 이동도 비용이다.
- storage는 단순한 byte 저장소보다 지능적인 계층이 될 수 있다.
- compute를 data 가까이에 두면 network와 host CPU 부하를 줄일 수 있다.
- backup scan과 user query는 cache 가치가 다르다.
- 모든 데이터를 cache에 넣으면 cache pollution이 발생한다.
- DRAM, flash, capacity storage 사이에는 workload-aware tiering이 필요하다.

### 읽고 나서 답할 질문

1. SSD가 빨라도 network로 불필요한 데이터를 많이 보내면 왜 느릴 수 있는가?
2. backup I/O를 일반 user I/O처럼 cache하면 어떤 문제가 생기는가?
3. 단순 LRU보다 workload-aware cache가 유리한 상황은 무엇인가?
4. storage server가 processing을 수행하면 어떤 검증 책임이 추가되는가?
5. 이 구조가 AI dataset filtering 또는 vector search와 연결될 수 있는 이유는 무엇인가?

### 기존 노트와 연결

- [[엔터프라이즈 스토리지 계층화와 AI Workload]]
- [[Data Temperature]]
- [[PVB Metadata Model]]
- [[Temperature-Aware GC Core Findings]]
- [[Resource Contention Quality Experiment]]

---

# 추천 4주 학습 계획

## 1주차: 성능 상한과 Queue

- EBS I/O characteristics 읽기
- OCI Block Volume Performance 읽기
- QD, I/O size, IOPS, throughput 관계를 한 장에 그리기
- 기존 block-size sweep 결과를 cloud block storage 모델로 다시 설명하기

완료 기준:

> 서로 다른 block size에서 왜 IOPS와 bandwidth를 같이 봐야 하는지 노트 없이 설명할 수 있다.

## 2주차: Observer와 상태

- EBS detailed performance statistics 읽기
- Observer Matrix를 내 실험에 적용하기
- snapshot initialization 자료 읽기
- 테스트 전 상태 metadata 목록 만들기

완료 기준:

> p99 latency가 증가했을 때 필요한 추가 관찰값과 아직 단정할 수 없는 원인을 구분할 수 있다.

## 3주차: 내구성과 장애

- OCI local NVMe failure model 읽기
- local NVMe와 remote block volume 비교
- EBS latency injection 읽기
- White-box Lab의 failure를 cloud failure domain과 비교하기

완료 기준:

> device failure, host failure, service failure, application error를 서로 다른 테스트로 설계할 수 있다.

## 4주차: 데이터 이동과 계층화

- Exadata Smart Scan 읽기
- Exadata cache/tiering 구조 읽기
- `move data to compute`와 `move compute to data` 비교
- AI storage workload와 연결한 질문 3개 작성

완료 기준:

> 빠른 SSD를 추가하는 것과 데이터 이동량을 줄이는 것이 왜 서로 다른 최적화인지 설명할 수 있다.

---

# 자료를 읽을 때 사용할 내부화 템플릿

각 자료를 읽은 후 원문을 복사하지 말고 아래 질문에 답한다.

```md
## 자료명

### 이 자료가 해결하려는 문제
-

### 시스템 경계
- Application:
- Host/OS:
- Protocol/network:
- Storage service:
- Physical device:

### 핵심 입력과 출력
- 입력:
- 출력:

### 성능 또는 장애 모델
-

### 관찰 가능한 metric
-

### 자료가 증명한 것
-

### 자료가 증명하지 못한 것
-

### 내 프로젝트와 연결
-

### 내 말로 한 문장
-

### 아직 답하지 못하는 질문
-
```

---

# 최종 자기 점검 질문

다음 질문에 노트 없이 답할 수 있으면 1차 학습이 끝난 것이다.

1. 클라우드 block storage가 local SSD보다 느릴 수 있는 경로상의 이유는 무엇인가?
2. IOPS는 충분한데 throughput 한도에 걸리는 사례를 숫자로 설명할 수 있는가?
3. 높은 QD가 처리량과 tail latency에 서로 다른 영향을 주는 이유는 무엇인가?
4. volume limit와 instance limit를 어떤 metric으로 구분할 수 있는가?
5. snapshot에서 복원한 volume의 첫 접근이 느릴 수 있는 이유는 무엇인가?
6. local NVMe의 낮은 latency를 얻는 대신 사용자가 책임져야 하는 것은 무엇인가?
7. RAID, replication, checksum, backup은 각각 어떤 failure를 다루는가?
8. storage latency injection 실험에 guardrail이 필요한 이유는 무엇인가?
9. Smart Scan이 단순히 더 빠른 SSD를 쓰는 것과 다른 이유는 무엇인가?
10. 현재 SSD Mini Lab에서 cloud storage 관점으로 가장 먼저 보강할 observer는 무엇인가?

---

# 주의할 점

- 공식 자료라도 제품의 장점을 설명하기 위한 문서일 수 있다. 공개된 측정 조건과 한계를 함께 읽는다.
- 오래된 Oracle 블로그의 성능 수치는 현재 제품 한도로 사용하지 않는다. 실험 방법만 참고하고 최신 한도는 공식 문서에서 확인한다.
- cloud service의 내부 물리 구현은 공개 범위가 제한적이므로 문서에 없는 내부 원인을 단정하지 않는다.
- fio write workload를 사용 중인 raw device에 실행하면 데이터가 손상될 수 있다.
- 실제 cloud 실습은 비용이 발생할 수 있으므로 volume, instance, snapshot을 만든 경우 종료 및 삭제 조건을 먼저 정한다.
- 데이터센터 storage 학습의 목표는 제품명을 외우는 것이 아니라 `요구사항 → 조건 → 관찰 → 병목 → 판정 → 복구`의 사고 흐름을 익히는 것이다.

