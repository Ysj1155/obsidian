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
capacity tuning by performance matching. 기준 성능과 