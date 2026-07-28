[출처1](https://phisonblog.com/ko/nand-flash-101-enterprise-vs-client-ssds-2/)

# 클라이언트 SSD
클라이언트 SSD(소비자 SSD)는 태블릿, 노트북, 데스크톱 PC처럼 단일 사용자/단일 시스템 중심 환경의 기본 저장 장치로 쓰이는 SSD다. 엔터프라이즈 SSD보다 burst 성능, 전력, 가격, 일반 사용자 체감 성능을 더 중시하는 경우가 많지만, 제품군별 차이가 크다.

1. 지구력
	클라이언트 workload는 일반적으로 엔터프라이즈 DB/로그/스토리지 서버보다 write pressure가 낮은 경우가 많다. 그래서 보증/내구성은 보통 TBW 중심으로 제시된다. 다만 영상 편집, 게임 설치, 개발 빌드, 로컬 AI 데이터셋처럼 쓰기량이 큰 개인 workload도 있으므로 너무 가볍다고 단정하면 안 된다.
2. SLC 캐시
	많은 클라이언트 SSD는 TLC 또는 QLC NAND를 사용하고, 일부 영역을 pseudo-SLC cache처럼 운용해 짧은 burst write 성능을 높인다. cache 크기와 방식은 제품마다 다르다.
	pseudo-SLC cache는 짧은 쓰기 burst를 빠르게 받아 사용자 체감 성능을 높이는 데 유리하다. 다만 cache가 소진되거나 background folding이 발생하면 sustained write 성능과 tail latency가 달라질 수 있다.
3. 안전장치
	클라이언트 SSD는 엔터프라이즈 SSD처럼 강한 PLP를 제공하지 않는 경우가 많다. 하지만 모든 제품이 동일한 것은 아니며, controller/firmware 수준의 basic protection과 capacitor 기반 full PLP는 구분해야 한다.
4. 보증
	클라이언트 SSD 보증은 보통 기간과 TBW 조건을 함께 본다. 기간은 제품군에 따라 3년 또는 5년 등 다양하므로 “최대 1년”처럼 일반화하면 안 된다.
