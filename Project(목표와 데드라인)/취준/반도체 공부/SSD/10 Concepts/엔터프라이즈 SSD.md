[출처1](https://semiconductor.samsung.com/kr/ssd/enterprise-ssd/)
[출처2](https://www.kingspec.com/ko/news/what-is-enterprise-ssd.html)
[출처3](https://phisonblog.com/ko/nand-flash-101-enterprise-vs-client-ssds-2/)

# 엔터프라이즈 SSD(기업용 SSD)
엔터프라이즈 SSD는 데이터센터, 서버, 스토리지 어레이처럼 높은 내구성, 일관된 성능, 관리성, 전원 장애 대응이 중요한 환경을 대상으로 설계되는 SSD다. 일부 제품은 dual-port, namespace, telemetry, PLP 같은 기능을 제공하지만, 모든 엔터프라이즈 SSD가 같은 기능을 갖는 것은 아니므로 SKU와 스펙 확인이 필요하다. 랙 스토리지에서는 RAID나 분산 스토리지 구성의 일부로 사용될 수 있다.

1. 지구력
	기업용 SSD는 혼합된 수의 읽기 및 쓰기를 수행하는 애플리케이션에 설치될 수 있으며 때로는 하루에 SSD의 전체 콘텐츠를 쓰기도 합니다. 일반적으로 파일 크기가 작은 Windows와 같은 소비자 운영 체제를 실행하는 클라이언트 SSD에서는 이런 일이 발생할 가능성이 거의 없습니다. 그러나 엔터프라이즈 SSD 설치에서 SSD는 매우 큰 데이터베이스 또는 데이터 세트와 함께 사용될 수 있습니다. 따라서 기업용 SSD는 일반적으로 DWPD(드라이브 쓰기 횟수)로 측정되는 훨씬 더 높은 내구성이 필요합니다.
2. SLC 캐시
	엔터프라이즈 SSD에서는 클라이언트 SSD처럼 큰 pseudo-SLC cache에 의존한 burst 성능보다 sustained write, QoS, tail latency 안정성이 더 중요하게 취급되는 경우가 많다. 다만 “엔터프라이즈 SSD는 SLC 캐시를 쓰지 않는다”처럼 일반화하면 안 되고, 제품별 캐싱/펌웨어 정책을 확인해야 한다.
3. 안전장치
	많은 엔터프라이즈 SSD는 Power Loss Protection(PLP)을 제공한다. 보통 커패시터 등으로 전원 장애 시 in-flight data나 FTL metadata를 보호하려는 구조지만, 보호 범위는 제품마다 다르다. 사용자 데이터, DRAM cache, FTL metadata, atomicity 보장 범위는 스펙과 전원 차단 테스트로 확인해야 한다.
4. 보증
	기업용 SSD는 보통 DWPD/TBW 같은 endurance 조건과 함께 보증 기간이 제시된다. 5년 보증이 흔하지만 제품군마다 다르다.
5. 추가 기능
	 단일 데이터 센터에는 수백 또는 수천 개의 엔터프라이즈 SSD가 포함될 수 있습니다. 따라서 엔터프라이즈 SSD에는 종종 펌웨어에 추가 기능이 추가되어 데이터 센터를 감독하는 IT 관리자가 보다 쉽게 구성하고 상태를 보고할 수 있습니다.

##### 엔터프라이즈 SSD에서 캐시 의존도를 조심해서 봐야 하는 이유
엔터프라이즈 환경에서는 일관된 지연시간과 sustained 성능이 중요하다. pseudo-SLC cache에 크게 의존하면 cache flush나 cache exhaustion 이후 지연시간/처리량이 달라질 수 있으므로, 제품 검증에서는 burst 구간과 steady-state 구간을 분리해서 봐야 한다. 다만 캐시 사용 여부와 방식은 제품별 구현이다.

##### 모든 엔터프라이즈 SSD가 RAID 구성과 호환됩니까?
많은 엔터프라이즈 SSD는 RAID나 분산 스토리지 환경에서 예측 가능한 성능을 목표로 하지만, 호환성은 컨트롤러, 펌웨어, 폼팩터, dual-port 지원, namespace 기능, host stack에 따라 달라진다. 특정 벤더 제품 설명을 전체 엔터프라이즈 SSD 특성으로 일반화하면 안 된다.

##### 엔터프라이즈 SSD는 데이터 센터 관리에 어떻게 도움이 되나요?
엔터프라이즈 드라이브에는 원격 분석, 스마트 모니터링, 펌웨어 수준 제어 기능이 포함되어 있습니다. 이를 통해 IT 관리자는 수천 대의 드라이브 성능, 상태 및 수명 주기 상태를 추적하여 유지 관리를 간소화하고 다운타임을 최소화할 수 있습니다.