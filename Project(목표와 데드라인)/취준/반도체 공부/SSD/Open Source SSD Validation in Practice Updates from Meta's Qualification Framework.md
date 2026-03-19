---
aliases:
  - "Open Source SSD Validation in Practice: Updates from Meta's Qualification Framework"
tags:
  - ssd
  - validation
  - reference
---
[출처](https://www.youtube.com/watch?v=-w6hqG_TvJw)

# 1. 테스트 아키텍쳐
## 왜 Pytest와 python인가? 
Meta는 거대한 데이터 센터 환경에서 수만 개의 SSD를 관리한다. 과거에는 제조사마다 제각각인 전용 validation 도구를 사용했지만, Meta는 이를 Pytest 기반의 파이썬 프레임 워크로 통합했다.
- 확장성: 수천개의 테스트 케이스를 구조화하고 병렬로 실행하기 적합
- Fixture 기능: 테스트 전후에 SSD의 상태를 초기화하거나 로그를 수집하는 작업을 쉽게 관리하기 위해.
- 생태계: pytest는 널리 사용하는 도구여서 새로운 엔지니어나 SSD 벤더사가 프레임워크를 배우는 비용이 적기 때문. 
- 실행 속도 보다는 개발 속도와 코드의 가독성이 더 중요하기 때문. 하드웨어 자체의 성능 측정은 내부 엔진이 담당하고, 엔지니어는 테스트 시나리를 빠르게 작성하고 수정하는데 집중하기 위함. 
## 하드웨어 추상화 계층(HAL)
Meta는 삼성, 하닉, 마이크론 등 수많은 제조사(Vendor)의 SSD를 사용한다. 하지만 제조사마다 사용하는 도구나 세부 명령어(Vendor Unique Commands)가 조금씩 다르다. HAL은 이런 복잡한 차이점을 중간에서 가려주는 역할. 
- 표준화: 테스트 스크립트는 "SSD의 로그를 가져와"라는 표준 명령만 내림
- 어댑터 역할: HAL이 각 제조사에 맞는 구체적인 명령어(NVMe 표준 혹은 제조사 전용 명령어)로 번역해서 전달.
엔지니어는 특정 제조사에 종속되지 않고 공통된 하나의 테스트 코드만 작성하면 된다.
### 테스트 실행 흐름(Execution Flow)
스크립트의 실행 순서
	1. 발견(Discovery): 현재 서버에 꽃힌 SSD가 뭐냐
	2. 준비(Setup): 테스트를 위해 SSD의 데이터를 지우거나(Sanitize) 특정 상태로 만들기
	3. 실행(Test Loop): 실제 읽기/쓰기나 에러 주입 시나리오 돌리기
	4. 수집(Log Collection): HAL을 통해 SSD 내부의 로그(Telemetry)를 긁어모으기
4번 로그 수집에서 왜 실패했는지 밝히기 위해 데이터 분석하는게 중요하다.
### 테스트 유닛의 독립성(Test Isolation)
수천 개의 SSD를 테스트할 때, 앞선 테스트가 다음 테스트에 영향을 주면 안된다. Meta의 프레임워크는 각 테스트 케이스가 실행되기 전후로 SSD를 특정 상태(Clean State)로 되돌리는 Setup & Tear down 로직이 정교하게 짜여져 있다. 
### 로그 파싱 및 시각화(Log Parsing & JSON)
SSD에서 쏟아지는 Raw 데이터는 사람이 읽기 힘들다. 이 아키텍쳐에는 로그를 반드시 JSON 형식으로 변환해 데이터베이스에 저장하고 이를 대시보드로 시각화하는 파이프라인이 포함되어있다. 엔지니어는 텍스트 로그가 아니라 그래프를 보며 문제 진단이 가능하다. 

# 2. 핵심 검증 시나리오(Key Scenarios)
### 에러 주입(Error Injection)
데이터 센터애는 수만 개의 SSD가 동시에 돌아가기 때문에 확률적으로 반드시 하드웨어 에러나 갑작스러운 전원 차단이 발생한다. Meta의 검증 스크립트는 이를 인위적으로 만들어낸다. 
- 정전 시나리오(Sudden Power Loss): 데이터를 쓰고 있는 도중 전원을 갑자기 차단. 다시 켰을 때 이전에 기록된 데이터가 깨지지 않았는지(Data Integrity), SSD가 벽돌 안되고 정상적으로 부팅 되는지 확인.
- 비트 에러 유도(Bit Error): SSD 내부의 낸드 플래시 메모리에 고의로 에러 일으키기. 이때 SSD가 자체 정정 기능(ECC)을 통해 데이터를 잘 복구는지, 혹은 다잉 메시지 보고하는지 검증.
### FDP(Flexible Data Placement); 효율적인 데이터 배치
- 

# 3. 업계 표준과 협업(Ecosystem)