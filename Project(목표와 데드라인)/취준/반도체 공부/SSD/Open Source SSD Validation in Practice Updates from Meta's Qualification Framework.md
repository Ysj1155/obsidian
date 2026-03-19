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
	3. 실행(Test Loop)
