[출처1](https://semiconductor.samsung.com/kr/ssd/enterprise-ssd/)
[출처2](https://www.kingspec.com/ko/news/what-is-enterprise-ssd.html)
[출처3](https://phisonblog.com/ko/nand-flash-101-enterprise-vs-client-ssds-2/)

# 엔터프라이즈 SSD(기업용 SSD)
데이터 센터에서 랙 스토리지 또는 애플리케이션 서버로 사용하도록 설계되었습니다. 클라이언트 SSD와 달리 엔터프라이즈 SSD는 하나 이상의 마더보드 PC에서 쓰기 및 읽기 명령을 받을 수 있습니다. 랙 스토리지에 설치된 SSD는 일반적으로 RAID 스토리지로 사용되도록 구성되며, 여러 드라이브가 단일 장치로 작동하여 성능 또는 데이터 중복성을 향상시킵니다.

1. 지구력
	기업용 SSD는 혼합된 수의 읽기 및 쓰기를 수행하는 애플리케이션에 설치될 수 있으며 때로는 하루에 SSD의 전체 콘텐츠를 쓰기도 합니다. 일반적으로 파일 크기가 작은 Windows와 같은 소비자 운영 체제를 실행하는 클라이언트 SSD에서는 이런 일이 발생할 가능성이 거의 없습니다. 그러나 엔터프라이즈 SSD 설치에서 SSD는 매우 큰 데이터베이스 또는 데이터 세트와 함께 사용될 수 있습니다. 따라서 기업용 SSD는 일반적으로 DWPD(드라이브 쓰기 횟수)로 측정되는 훨씬 더 높은 내구성이 필요합니다.
2. SLC 캐시
	 RAID의 SSD는 클라이언트 SSD처럼 SLC 캐시의 이점을 누릴 수 없습니다. 이는 기업의 호스트 컴퓨터가 RAID SSD에서 데이터를 적절하게 스트라이프하려면 예측 가능한 성능이 필요하기 때문입니다. RAID에 있는 하나 이상의 SSD가 오프라인 상태가 되어 SLC 캐시를 TLC NAND 영역에 복사하면 전체 RAID 서버가 일시적으로 중단될 수 있습니다.