# Portfolio 2 — SQL 심화 분석 (CVAT 라벨링 데이터 기반)

Portfolio 1(CVAT 라벨링 + Python EDA)에서 사용한 데이터를 재활용하여,
MySQL에서 JOIN, 서브쿼리, 윈도우 함수, CTE 등 SQL 심화 문법을 실습한 프로젝트입니다.

## 사용 도구
- MySQL Workbench

## 데이터셋 구조

**images 테이블** (13행)
| 컬럼 | 설명 |
|---|---|
| image_id | 이미지 고유 ID (PK) |
| file_name | 파일명 |
| width, height | 이미지 크기 |
| upload_date | 업로드 날짜 |
| resolution_width, resolution_height | 해상도 |
| annotator | 라벨링 작업자 |

**annotations2 테이블** (34행)
| 컬럼 | 설명 |
|---|---|
| ann_id | 어노테이션 고유 ID (PK) |
| image_id | images 테이블 참조 (FK) |
| label | 객체 클래스 (vehicle / person) |
| area, bbox_area | 바운딩박스 면적 |
| occluded | 객체 가림 여부 |
| review_status | 검수 상태 (approved / pending / rejected) |

---

## 분석 1: 라벨별 평균 면적 필터링 (GROUP BY + HAVING)

**질문**: 평균 바운딩박스 면적이 15000 이상인 라벨은 무엇인가?

```sql
SELECT
    label,
    COUNT(*) AS count,
    AVG(bbox_area) AS avg_area
FROM annotations2
GROUP BY label
HAVING AVG(bbox_area) >= 15000;
```

**결과**: `vehicle` 라벨만 조건 충족 (18개, 평균 면적 20,900)

**포인트**: `WHERE`는 그룹핑 이전 개별 행을 걸러내고, `HAVING`은 그룹핑 이후 집계 결과를 걸러낸다. `AVG()` 같은 집계 함수는 `WHERE`가 아니라 `HAVING`에서만 사용 가능하다.

---

## 분석 2: 평균 이상 객체를 포함한 이미지 조회 (서브쿼리)

**질문**: 전체 평균 면적보다 큰 객체를 하나라도 포함한 이미지는?

```sql
SELECT DISTINCT image_id
FROM annotations2
WHERE bbox_area > (SELECT AVG(bbox_area) FROM annotations2);
```

**결과**: 총 12개 이미지 조회됨 (`DISTINCT`로 중복 제거)

**포인트**: 서브쿼리로 동적 기준값(전체 평균)을 계산해 조건에 바로 활용할 수 있다. 하드코딩 없이 데이터가 바뀌어도 기준이 자동으로 갱신된다.

---

## 분석 3: 라벨별 면적 순위 매기기 (윈도우 함수)

**질문**: 같은 라벨 안에서 면적이 큰 순서로 순위를 매기면?

```sql
SELECT
    ann_id,
    label,
    bbox_area,
    RANK() OVER (PARTITION BY label ORDER BY bbox_area DESC) AS rank_in_label
FROM annotations2
ORDER BY label, rank_in_label;
```

**결과**: `person`, `vehicle` 각 그룹 안에서 1위부터 순위가 매겨짐

**포인트**: `PARTITION BY`는 `GROUP BY`와 달리 그룹핑 후 행을 압축하지 않고 원본 행을 그대로 유지하면서 그룹별 계산(순위, 누적값 등)을 수행한다.

---

## 분석 4: 리뷰 상태별 비율 계산 (CTE)

**질문**: 검수 상태(approved/pending/rejected)의 개수와 비율은?

```sql
WITH status_count AS (
    SELECT review_status, COUNT(*) AS cnt
    FROM annotations2
    GROUP BY review_status
)
SELECT
    review_status,
    cnt,
    ROUND(cnt * 100.0 / (SELECT SUM(cnt) FROM status_count), 1) AS percentage
FROM status_count;
```

**결과**:
| review_status | count | percentage |
|---|---|---|
| approved | 17 | 50.0% |
| pending | 11 | 32.4% |
| rejected | 6 | 17.6% |

**포인트**: CTE(`WITH`절)로 복잡한 쿼리를 단계별로 나누면 가독성이 높아지고, 같은 집계 결과를 여러 번 재사용할 수 있다.

---

## 분석 5: 작업자별 이미지 담당 현황 (JOIN + GROUP BY + 윈도우 함수)

**질문**: 작업자(annotator)별로 담당한 이미지마다 객체 수를 비교하면?

```sql
SELECT
    i.annotator,
    i.image_id,
    COUNT(a.ann_id) AS object_count,
    RANK() OVER (PARTITION BY i.annotator ORDER BY COUNT(a.ann_id) DESC) AS rank_by_annotator
FROM images i
LEFT JOIN annotations2 a ON i.image_id = a.image_id
GROUP BY i.annotator, i.image_id;
```

**결과 예시**:
| annotator | image_id | object_count | rank_by_annotator |
|---|---|---|---|
| reviewer1 | 1 | 4 | 1 |
| reviewer1 | 5 | 2 | 2 |
| sansan | 13 | 3 | 1 |

**포인트**: `GROUP BY`와 윈도우 함수를 함께 쓸 때는 `GROUP BY`가 먼저 적용되어 집계된 후, 그 결과에 윈도우 함수가 적용된다. `RANK()`는 동점일 경우 같은 순위를 부여하고 다음 순위를 건너뛴다 (예: 공동 2위가 있으면 다음은 4위).

---

## 배운 점 정리

- **WHERE vs HAVING**: 집계 전/후 필터링의 차이
- **서브쿼리**: 동적 기준값을 조건절에 활용하는 방법
- **PARTITION BY vs GROUP BY**: 원본 행 유지 여부의 차이
- **CTE**: 복잡한 쿼리를 단계별로 나눠 가독성과 재사용성 확보
- **RANK() 순위 규칙**: 동점 처리 방식 (공동 순위 후 스킵)
- **RIGHT JOIN**: 실무에서는 LEFT JOIN으로 대체 가능해 거의 사용하지 않음

## 다음 목표
- Portfolio 3: Tableau 대시보드 제작
