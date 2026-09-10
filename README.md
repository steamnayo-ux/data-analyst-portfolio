\# Data Analyst Portfolio



데이터 라벨링, EDA, SQL, 시각화 실습을 정리한 프로젝트 모음입니다.



\## 프로젝트 구성



\### 1. CVAT 라벨링 + Python EDA (`portfolio1-cvat-eda/`)

\- CVAT을 활용한 이미지 라벨링: BBox(vehicle·person), Polygon(cat/dog), Polyline(차선), Attribute(occluded)

\- Pascal VOC / COCO / YOLO 3종 포맷 export 비교

\- Python EDA: XML 파싱, 클래스 분포 분석, bbox 면적 통계, Shoelace 공식을 이용한 polygon 면적 계산, 시각화



\*\*주요 결과\*\*

\- Vehicle bbox 면적 분포는 right-skewed 형태로, 대부분 작고 일부만 매우 큼

\- Cat/Dog 클래스별 개수·크기 비교



\### 2. SQL 심화 분석 (`portfolio2-sql/`)

\- CVAT 라벨링 데이터를 MySQL 스키마로 재구성 (images, annotations2 테이블)

\- 실습 내용:

&#x20; 1. GROUP BY + HAVING을 이용한 라벨별 평균 면적 필터링

&#x20; 2. 서브쿼리를 활용한 조건부 이미지 조회

&#x20; 3. 윈도우 함수(RANK + PARTITION BY)를 이용한 라벨별 면적 순위

&#x20; 4. CTE(WITH절)를 활용한 리뷰 상태별 비율 계산

&#x20; 5. JOIN + GROUP BY + 윈도우 함수를 종합한 작업자별 이미지 담당 현황



\## 사용 기술

\- \*\*Labeling\*\*: CVAT

\- \*\*분석\*\*: Python (pandas, matplotlib)

\- \*\*DB\*\*: MySQL Workbench

\- \*\*환경\*\*: Jupyter Lab

