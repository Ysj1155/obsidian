[출처](obsidian://open?vault=obsidian_clean&file=Project(%EB%AA%A9%ED%91%9C%EC%99%80%20%EB%8D%B0%EB%93%9C%EB%9D%BC%EC%9D%B8)%2F%EC%B7%A8%EC%A4%80%2FFADU%2Fssd%20validation%20%ED%8A%B9%ED%97%88.pdf)

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
