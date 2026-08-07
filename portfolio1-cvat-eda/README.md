# 🚗🐱 CVAT 라벨링 + Python EDA (Portfolio 1)

CVAT을 이용해 CCTV 도로 데이터에 직접 라벨링을 수행하고, 
그 결과를 Python(pandas)으로 분석한 프로젝트입니다.
BBox, Polygon, Polyline, Attribute 4가지 라벨링 유형을 모두 실습했습니다.

## 📁 데이터 & 라벨링 결과

| 구분 | 유형 | 이미지 수 | 라벨 수 | 클래스 |
|---|---|---|---|---|
| 차량/사람 | BBox | 100장 | 1,519개 | vehicle(1,402) / person(117) |
| 동물 | Polygon | 27장 | 31개 | cat(17) / dog(14) |
| 차선 | Polyline | 5장 | 20개+ | lane |
| 차량 부가정보 | Attribute | - | - | occluded (true/false) |

## 🔍 분석 과정 (EDA)

1. CVAT XML → pandas DataFrame 파싱 함수 작성 (BBox / Polygon 겸용)
2. **BBox 면적**: `width × height`로 계산
3. **Polygon 면적**: Shoelace 공식으로 계산 (좌표만으로 다각형 넓이 산출)
4. 클래스별 개수·면적 통계 및 시각화 3종 (막대그래프 / boxplot / 히스토그램)

## 💡 주요 발견

- **클래스 불균형**: vehicle이 person보다 12배 이상 많음 (CCTV 특성상 자연스러운 결과)
- **BBox 면적 분포**: 오른쪽 꼬리가 긴(right-skewed) 형태 — 대부분 작고, 카메라에 가까운 소수 차량만 매우 큼
- **Cat vs Dog 면적**: 평균은 비슷하나(cat 60,209 / dog 59,025), 각각 이상치(outlier) 1건씩 존재
- **좌표/라벨 품질 자가점검**: 좌표 범위 이탈, 라벨 오타, 비정상적으로 작은 라벨 → 전량 이상 없음 확인

## 📦 Export 포맷 지원

동일 라벨 데이터를 3가지 표준 포맷으로 export하여 좌표 표현 방식 차이를 비교했습니다.

| 포맷 | 좌표 표현 | 비고 |
|---|---|---|
| Pascal VOC | `xmin, ymin, xmax, ymax` (픽셀) | 이미지당 XML 1개 |
| COCO | `[x, y, width, height]` (픽셀) | 전체 JSON 1개 |
| YOLO | `중심x, 중심y, width, height` (0~1 정규화) | 이미지당 txt 1개 |

## 📂 폴더 구조

```
portfolio1-cvat-eda/
├── raw_images/
│   ├── vehicle/
│   └── animal/
├── annotations/
│   ├── vehicle/annotations.xml
│   └── animal/annotations.xml
├── exports/
│   ├── voc/
│   ├── coco/
│   └── yolo/
└── eda_analysis.ipynb
```

## 🛠 사용 기술

`CVAT` `Python 3.14` `pandas` `matplotlib` `Jupyter Lab`

## ⚠️ 라벨링 판단 기준

- 겹친 객체는 보이는 만큼만 라벨링
- 오토바이 운전자는 person + vehicle 분리
- 화면 경계에 잘린 객체는 50% 이상 보일 때만 라벨링
- 가려진 차선(Polyline)은 이어서 그리지 않고 별도 선으로 분리
