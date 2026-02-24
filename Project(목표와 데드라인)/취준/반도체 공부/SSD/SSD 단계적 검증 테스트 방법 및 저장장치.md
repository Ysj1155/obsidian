[출처](obsidian://open?vault=obsidian_clean&file=Project(%EB%AA%A9%ED%91%9C%EC%99%80%20%EB%8D%B0%EB%93%9C%EB%9D%BC%EC%9D%B8)%2F%EC%B7%A8%EC%A4%80%2FFADU%2Fssd%20validation%20%ED%8A%B9%ED%97%88.pdf)

출원인은 北京忆恒创源科技股份有限公司(메모블레이즈 계열로 알려진 중국 엔터프라이즈 SSD 업체)이고, 공개번호는 CN120808860A, 공개일은 2025-10-17

### 핵심: 전체 용량(full capacity) 기준의 랜덤 쓰기 성능(특히 steady-state 4K random write)을 더 짧은 시간에 근사/재현할 수 있도록, short stroke(사용자 용량 축소)로 테스트 시간을 줄이는 단계적 검증 방법

### 특허의 전제
- SSD 성능 검증에서 진짜 의미있는 랜덤 쓰기 성능을 보려면 빈 디스크가 아니라 사전 처리(preconditioning)를 통해 안정 상태(steady-state)에 도달한 뒤 테스트 해야 된다.
- SSD 용량이 커질수록(엔터프라이즈 SSD 16TB/32TB/64TB급) preconditioning 시간이 길어지고 테그트가 시간/장비 점유 측면에서 비싸진다. 
- 개발 단계(펌웨어/기능이 자주 바뀌는 단계)에서는 이런 긴 테스트가 병목이 된다.
steady-state: 비반복 데이터 쓰기 -> GC 활성화 -> GC와 host write 균형 -> 성능 안정화. 기존 방식이 개발 속도를 따라오지 못하는 문제를 정의한다. 기존 방식이 뭔데?

### 특허의 핵심 아이디어
전체 용량 SSD의 랜덤 쓰기 성능을 목표 성능(target performance)으로 잡고, 사용자 용량을 short stroke로 줄여가며 그 목표 성능과 일치하는 목표 용량(target capacity)을 찾은 뒤, 이후 단계 검증에서는 그 목표 용량만 써서 빠르게 테스트하기. 
원래는 16TB 전체를 계속 예열/테스트해야 했다면 특허 특허 방식은 예를 들어 4TB 수준의 short-stroked 사용자 용량이 전체 용량의 steady-state 랜덤 쓰기 성능과 유사하게 맞아떨어지는 지점을 찾고, 그 이후부터는 그 용량으로만 빠르게 반복 검증하는 방식이다.
이게 빠른 이유는 preconditioning 시간이 사용자 용량에 비례해서 크게 줄어들기 때문이다.

### 절차
#### 단계 A. 기준 성능(목표 성능) 확보
1. 표준 용량(full capacity / standard capacity) SSD를 준비
2. 사전처리(preconditioning) 수행 → steady-state 도달
3. 4K random write 성능을 측정 (IOPS 기준)
4. 이 결과를 목표 성능(target performance) 으로 저장
명세서 예시:
- 순차 쓰기: `bs=128KB`, `iodepth=1024`, `numjobs=1`, 전체 용량 2회
- 랜덤 쓰기: `bs=4KB`, `iodepth=8`, `numjobs=64`, 전체 용량 2회
성능 측정 예시:
- `bs=4KB`, `iodepth=8`, `numjobs=64`, `runtime=600s` 랜덤쓰기 성능 테스트(FIO 사용)
먼저 full-cap steady-state 성능을 ground truth. baseline 없이 축약 테스트를 하는게 아니라 보정(calibration) 단계가 있다.

#### B. 목표 성능과 일치하는 목표 용량 탐색 (캘리브레이션 단계)
1. short stroke로 사용자 용량(current user capacity)을 설정
	 최초값은 예시로 전체 용량의 1/4
2. 해당 사용자 용량에 대해
	 preconditioning → 랜덤쓰기 성능 측정
3. 측정 성능을 목표 성능과 비교
	 같으면: 그 용량을 **목표 용량(target capacity)** 으로 확정
	 다르면: 용량 조절 후 재시도
		 성능이 목표보다 낮으면 → 용량 감소
		 성능이 목표보다 높으면 → 용량 증가
4. 일치했다고 끝내지 않고, **여러 번 반복 측정(예: 6회)** 해서 일관되게 맞는지 확인 후 확정
capacity tuning by performance matching. 기준 성능과의 일치성(consistency)을 조건으로 목표 용량을 찾는 접근.

#### C. 빠른 단계 검증(일상 회귀 / 빈번한 확인)
1. 찾은 목표 용량으로 SSD 사용자 용량 설정
2. 그 용량만 preconditioning
3. 랜덤쓰기 성능 테스트
4. 결과 기록 후 현재 버전 이상 여부 분석
특허는 이걸 통해 기존 시간(시간~일 단위)을 분 단위로 줄일 수 있다고 주장하고, 개발 단계에서 검증 빈도를 늘리고 커버리지를 높일 수 있다고 설명한다. 도면 5/6도 기존 대비 시작부터 steady-state 도달까지 시간 단축을 비교하려는 용도다.
![[Pasted image 20260224224907.png]]

### short stroke를 왜 쓰는가
사용자 용량(논리 주소 범위)을 줄이지만 SSD 내부 동작이 너무 비정상적으로 왜곡되지 않게 해야 한다는점, 특히 명세서에 모든 Die에 접근 가능해야 된다는 취지가 강조된다. JESD218B의 short stroke extrapolation 개념을 참조한다.

Validation 입장에서 가장 위험한 것은 테스트가 빠른 대신 실제 full-cap 조건과 다른 현상을 보고 잘못 판단하는 것. 특허는 이 문제를 피하기 위해 target performance 매칭, short stroke 적용 시 내부 구조 왜곡 최소화(모든 die 접근)를 함께 말한다. 시간 단축이 아니라 대표성(representativeness)을 확보하려는 시도.

용량을 줄이면 L2P(Logical to Physical) 매핑 테이블의 크기도 줄어든다. 엔터프라이즈 SSD는 매핑 테이블이 크기 때문에 컨트롤러 캐시(DRAM) 히트율에 영향을 준다. 어떻게?
-> 용량을 줄여서 테스트할 때, 컨트롤러의 DRAM 캐시 부하가 실제 Full Cap 상황보다 너무 가벼워지지는 않는가에 대한 데이터 분석이 병행되어야 이 방법론의 신뢰성이 완성.

### 특허에서 직접 나온 테스트 조건
명세서 기준으로
Preconditioning 예시 (steady-state 유도)
- 순차쓰기 2회
    - `blocksize=128KB`
    - `iodepth=1024`
    - `numjobs=1`
- 랜덤쓰기 2회
    - `blocksize=4KB`
    - `iodepth=8`
    - `numjobs=64`
성능 측정 예시
- 4KB random write
    - `blocksize=4KB`
    - `iodepth=8`
    - `numjobs=64`
    - `runtime=600`
또한 데이터 제거는 sanitize/erase 명령을 통한 전체 사용자 데이터 삭제로 설명한다.

### 특허에 대한 Validation 관점으로 공부할 내용
1. 기준값 기반 축약 테스트 설계 능력
- 빠르게 하지만, 기준과 연결된 빠른 테스트.
	 full-cap baseline -> 축약 조건 calibration -> 반복 quick test 구조.
2. 성능 대표성(Representative test) 개념
- 그 결과가 실제 제품 위험을 반영하냐
- 오탐/미탐을 얼마나 줄이냐
	- 어떤 metric을 target으로 잡을거냐(평균 IOPS?)
	- 일치 기준은 얼마나 엄격하게 할거냐(±x%?)
	-  한 workload에서 맞으면 다른 workload도 맞는지?
3. 테스트 시간 vs 커버리지 트레이드 오프 설계
- 테스트 1회가 짧아지니 같은 장비 시간 내 더 많은 항목/버전 검증이 가능하다.
4. 표준/규격(JESD218B) 참조 습관
- NVMe spec / NVMe-MI / PCle base 개념 공부 필요
- 내부 시험 규격서?
- 릴리즈 검증 기준서?

### 특허 비판적 읽기
1. 4K random write IOPS 하나만 맞춰도 충분한가?
	 특허의 target은 4K random write 성능(IOPS) 중심. 실무는 평균 IOPS만 같아도 tail latency(P99/P99.9)는 다를 수 있다. GC 주기/진폭이 다르면 QoS가 달라질 수 있다. mixed workload(R70/W30), read disturb 성격, TRIM 유무에서는 달라질 수 있다. 즉, 이 방법은 단계적 빠른 스크리닝에 강하지만 최종 사양 검증 대체로 쓰면 위험하다. 
2. 용량 축소가 내부 동작을 완전히 보존하지는 않는다. 
	왜곡 문제가 있을 수 있다. OP 비율 체감 변화, GC 타이밍, wear distribution, thermal profile (테스트 시간이 줄어들면 발열 양상 자체가 달라짐), FTL mapping pressure, metadata behavior등 고려할 변수들이 존재한다. 즉, full-cap steady-state와 동일이 아니라 특정 지표에서 근사로 이해해야 된다. 
3. Workload의 국한성.
	 특허는 4K Random Write에 집중하고 있다. 하지만 실제 엔터프라이즈 환경에서는 Sequential Write와 Random Write가 섞인 믹스 워크로드(Mixed IO) 상황에서 GC가 훨씬 복잡하게 동작한다. 4K 단독 지표로 찾은 '목표 용량'이 믹스 워크로드에서도 대표성을 갖는지 검증이 필요하다. 
4. Wear Leveling 및 내부 수명 관리 알고리즘
	 특정 구간(Short Stroke 구간)만 계속 쓰게 되면, SSD 내부의 Wear Leveling 알고리즘이 특정 블록에 집중되는 마모를 피하기 위해 데이터를 옮기는 작업을 수행할 수 있다. 이 과정이 단기 테스트(Quick Test)에서는 나타나지 않다가 장기 테스트에서만 튀어나올 수 있는 리스크가 있다.
5. Thermal Throttling의 부재
	 테스트 시간이 짧아지면 SSD의 온도가 정상 상태(Thermal Equilibrium)에 도달하기 전에 테스트가 끝날 수 있다. 온도에 의한 성능 저하(Throttling) 시나리오를 놓칠 수 있다. 


### 직무 활동 해볼거
- FIO 스크립트 자동화:명세서에 나온 Preconditioning 및 측정 조건을 바탕으로 `fio` 스크립트를 짜보고, 파이썬 등으로 성능을 비교하여 용량을 자동으로 조절(Binary Search 방식 등)하는 캘리브레이션 툴의 로직을 설계해 보는 경험은 매우 강력한 포트폴리오가 됩니다.
- JESD218B 규격 비교: 특허에서 언급된 JEDEC 표준의 Short Stroke 개념과 실제 이 특허의 방식이 어떻게 다른지(표준은 외삽법(Extrapolation) 위주, 특허는 성능 매칭 위주) 비교 분석해 보세요.
- 데이터 시각화: 도면 5, 6처럼 실제 테스트 시간 단축 효과를 그래프로 그려보고, 시간 대비 검증 효율(Efficiency per Hour)이라는 지표를 정의.
- OP 상관관계와 DRAM 캐시 영향도 같은 하드웨어/펌웨어 구조적 관점 추가.