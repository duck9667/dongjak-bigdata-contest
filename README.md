![Python](https://img.shields.io/badge/Python-3.x-blue?logo=python)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter)
![Pandas](https://img.shields.io/badge/Pandas-DataFrame-150458?logo=pandas)
![NumPy](https://img.shields.io/badge/NumPy-Array-013243?logo=numpy)

# Dongjakgu 데이터 분석

동작구 2020 빅데이터 활용 정책 공모전.

## 1. 분석 목적 및 배경

- 목표: 입지 추천
- 배경: 수요에 비해 공급이 적다.

참고 데이터:

- 서울시 고등학교 수(2019-09-29). 출처: https://kess.kedi.re.kr/post/6684877?itemCode=04&menuId=m_02_04_02
- 서울시 청소년 인구 수 통계(2020-02-26). 출처: http://data.seoul.go.kr/dataList/10787/S/2/datasetView.do

## 2. 분석 순서

1. 행정동 단위로 클러스터링
2. 행정동 2차 필터링
3. 상위 집단 대상으로 필터링, 최적의 입지 선정

### 2.1 행정동 단위 클러스터링

참고 데이터:

- 주거 인구(2019-10-28). 출처: https://data.seoul.go.kr/dataList/10727/S/2/datasetView.do
- 현재 고등학교 갯수(2020-04-21). 출처: https://kess.kedi.re.kr/post/6684877?itemCode=04&menuId=m_02_04_02
- 연령별 인구(13-19세, 2020-03-31). 출처: http://27.101.213.4/#

데이터 전처리:

1. raw data 수집
2. 행정동 단위로 aggregation 진행
3. elbow method로 최적의 클러스터 개수 결정

![그림1](https://user-images.githubusercontent.com/33515088/107917442-4c973200-6fab-11eb-95c7-1f5ebc1a7fbf.png)

4. 4개의 클러스터로 나눈 후 각 클러스터의 특징 파악

![그림2](https://user-images.githubusercontent.com/33515088/107917903-0b535200-6fac-11eb-8a68-e8bfceb1f94b.png)

클러스터 특징:

- 클러스터 1: 대방동, 신대방1동. 거주인구가 많고 10대 비율이 타 지역보다 많다. 이 중 대방동은 이미 고등학교가 4개 설립되어 있다.
- 클러스터 2: 사당2동, 사당5동, 신대방2동. 2번째로 10대 비율이 많고, 이미 모두 고등학교가 설립된 지역.
- 클러스터 3: 노량진1동, 사당3동, 상도1동, 상도2동, 상도3동, 상도4동, 흑석동. 고등학교가 설립되어 있지 않으며, 2번째 클러스터와 10대 비율이 비슷하다.
- 클러스터 4: 노량진2동, 사당1동. 세대당 가구원수가 다른 지역에 비해 훨씬 적다(1인가구·고시원 영향으로 추정). 가장 중요한 10대 비율이 낮다.

클러스터 선정: 이미 고등학교가 설립된 지역과 10대 비율이 낮은 지역은 필터에서 제거하여 클러스터 1과 4에서 선정하되, 이미 고등학교가 많이 설립된 대방동은 제거한다. 신대방1동·상도3동·상도4동·상도2동·노량진1동·상도1동·흑석동·사당3동 8개를 필터링한다.

![그림3](https://user-images.githubusercontent.com/33515088/107917955-2756f380-6fac-11eb-959a-dafb06d9a47c.png)

### 2.2 행정동 2차 필터링

참고 데이터:

- 치안등급(2019-06). 출처: https://www.safemap.go.kr/main/smap.do?coreThemaMap=&flag=2&level=&searchOpt=&searchTxt=&sideMenu=3
- CCTV 개수(2020-03-27). 출처: https://data.seoul.go.kr/dataList/OA-13272/F/1/datasetView.do
- 버스 정류장 개수(2020-03-06). 출처: https://data.seoul.go.kr/dataList/OA-15067/S/1/datasetView.do

데이터 전처리:

1. 각 피처들의 특징을 동일한 scale로 스케일링 진행: 버스정류장 개수, CCTV 개수, 치안 등급을 정량화
2. 종합 인덱스 지표 생성: (버스정류장(교통) × 0.5) + (CCTV × 0.25) + (안전 × 0.25)
   흑석동 > 노량진1동 > 상도1동 > 상도4동 > 상도3동 > 사당3동 > 상도2동 > 신대방1동

![그림4](https://user-images.githubusercontent.com/33515088/107918168-83ba1300-6fac-11eb-8660-ed345012ea19.png)

이 중에서 접근성과 안정성 모두 상위에 스코어링된 노량진1동, 흑석동을 필터링한다.

### 2.3 상위 집단 대상 필터링, 최적의 입지

참고 데이터:

- 가용지 구분(2019-10-23). 출처: http://openapi.nsdi.go.kr/nsdi/eios/ServiceDetail.do
- 주변환경(2020-3-31). 출처: http://www.localdata.go.kr/devcenter/dataDown.do?menuNo=20001
- 행안부 업종데이터(2020-03-31). 출처: http://openapi.nsdi.go.kr/nsdi/eios/ServiceDetail.do?svcSe=F&svcId=F106

## 3. 입지 후보

최종적인 아웃풋은 대략적인 도로명 주소나 번지로 진행한다.
