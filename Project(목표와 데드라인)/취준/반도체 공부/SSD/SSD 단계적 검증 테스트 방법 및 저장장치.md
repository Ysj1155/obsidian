[출처](obsidian://open?vault=obsidian_clean&file=Project(%EB%AA%A9%ED%91%9C%EC%99%80%20%EB%8D%B0%EB%93%9C%EB%9D%BC%EC%9D%B8)%2F%EC%B7%A8%EC%A4%80%2FFADU%2Fssd%20validation%20%ED%8A%B9%ED%97%88.pdf)

### 핵심: 전체 용량(full capacity) 기준의 랜덤 쓰기 성능(특히 steady-state 4K random write)을 더 짧은 시간에 근사/재현할 수 있도록, short stroke(사용자 용량 축소)로 테스트 시간을 줄이는 단계적 검증 방법

### 특허의 전제
- SSD 성능 검증에서 진짜 의미있는 랜덤 쓰기 성능을 보려면 빈 디스크가 아니라 사전 처리(preconditioning)를 통해 안정 상태(steady-state)에 도달한 뒤 테스트 해야 된다.
- SSD 용량이 커질수록(엔터프라이즈 SSD 16TB/32TB/64TB급) preconditioning 시간이 길어지고 테그트가 시간/장비 