# 2026-09-11 누적합과 이전 행 비교

## 학습 목표

- `SUM() OVER`를 이용해 누적합을 계산할 수 있다.
- `PARTITION BY`와 `ORDER BY`가 누적 계산에서 수행하는 역할을 설명할 수 있다.
- 윈도우 프레임을 명시하여 누적합과 이동평균의 계산 범위를 제어할 수 있다.
- `LAG()`와 `LEAD()`로 이전·다음 행의 값을 가져올 수 있다.
- 현재 값과 이전 값을 이용해 증감량을 계산할 수 있다.
- 첫 행의 `NULL`과 정렬 기준이 시계열 비교에 미치는 영향을 이해할 수 있다.

---

## 1. 누적합(Running Total)이란?

누적합은 정해진 순서에 따라 첫 번째 행부터 현재 행까지의 값을 계속 더해 나가는 계산이다.

```text
첫 번째 행 → 값 1
두 번째 행 → 값 1 + 값 2
세 번째 행 → 값 1 + 값 2 + 값 3
네 번째 행 → 값 1 + 값 2 + 값 3 + 값 4
```

윈도우 함수에서는 `SUM()`과 윈도우 내부의 `ORDER BY`를 사용하여 누적합을 계산할 수 있다.

```sql
SUM(value_column) OVER (
    ORDER BY order_column
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

일반적인 `GROUP BY` 집계와 달리 기존 행은 그대로 유지되고, 각 행까지 누적된 값이 새로운 컬럼으로 추가된다.

---

## 2. 매장별 상품 재고 누적합

각 매장에서 상품 번호 순서대로 재고가 얼마나 누적되는지 계산해보자.

```sql
SELECT
    store_id,
    product_id,
    quantity,
    SUM(quantity) OVER (
        PARTITION BY store_id
        ORDER BY product_id
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_stock
FROM stocks
ORDER BY store_id, product_id;
```

1번 매장의 일부 결과는 다음과 같다.

| store_id | product_id | quantity | running_stock |
|---:|---:|---:|---:|
| 1 | 1 | 27 | 27 |
| 1 | 2 | 5 | 32 |
| 1 | 3 | 6 | 38 |

```text
상품 1 → 27
상품 2 → 27 + 5 = 32
상품 3 → 27 + 5 + 6 = 38
```

상품별 행은 유지되면서 현재 상품까지의 누적 재고가 `running_stock`에 추가된다.

---

## 3. 누적합 구성 요소 해석

```sql
SUM(quantity) OVER (
    PARTITION BY store_id
    ORDER BY product_id
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

각 요소의 역할은 다음과 같다.

| 구성 요소 | 의미 |
|---|---|
| `SUM(quantity)` | 지정된 범위의 재고 수량을 더한다. |
| `PARTITION BY store_id` | 매장별로 독립된 계산 그룹을 만든다. |
| `ORDER BY product_id` | 각 매장 안에서 상품 번호 순서로 누적한다. |
| `UNBOUNDED PRECEDING` | 현재 파티션의 첫 번째 행부터 시작한다. |
| `CURRENT ROW` | 현재 행까지 계산한다. |

```text
PARTITION BY store_id
→ 어느 그룹에서 누적할 것인가?
→ 같은 매장의 상품끼리

ORDER BY product_id
→ 어떤 순서로 누적할 것인가?
→ 상품 번호 순서로

UNBOUNDED PRECEDING ~ CURRENT ROW
→ 어디서부터 어디까지 계산할 것인가?
→ 파티션의 첫 행부터 현재 행까지
```

---

## 4. 파티션이 바뀌면 누적합도 다시 시작한다

`PARTITION BY store_id`를 사용했기 때문에 매장이 바뀌면 새로운 파티션이 시작되고 누적합도 다시 처음부터 계산된다.

```text
1번 매장
상품 1 → 27
상품 2 → 32
상품 3 → 38
...

2번 매장
상품 1 → 14
상품 2 → 30
상품 3 → 58
...
```

전체 데이터에서 하나의 누적합을 구하고 싶다면 `PARTITION BY`를 생략할 수 있고, 그룹별 누적합이 필요하면 적절한 그룹 컬럼을 지정한다.

---

## 5. 윈도우 프레임(Window Frame)

윈도우 프레임은 현재 행을 기준으로 실제 계산에 포함할 행의 범위를 정의한다.

| 프레임 키워드 | 의미 |
|---|---|
| `UNBOUNDED PRECEDING` | 파티션의 첫 번째 행부터 |
| `N PRECEDING` | 현재 행보다 N개 앞선 행부터 |
| `CURRENT ROW` | 현재 행 |
| `N FOLLOWING` | 현재 행보다 N개 뒤의 행까지 |
| `UNBOUNDED FOLLOWING` | 파티션의 마지막 행까지 |

예를 들어 다음 프레임은 파티션의 첫 행부터 현재 행까지를 의미한다.

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

다음 프레임은 현재 행과 앞의 두 행, 최대 세 행을 의미한다.

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
```

---

## 6. 이동평균(Moving Average)

이동평균은 전체 데이터를 계속 누적하는 것이 아니라 현재 행 주변의 일정한 범위만 이용해 평균을 계산하는 지표이다.

매장별로 상품 번호 순서에 따라 현재 행과 이전 두 행의 평균 재고를 계산해보자.

```sql
SELECT
    store_id,
    product_id,
    quantity,
    AVG(quantity) OVER (
        PARTITION BY store_id
        ORDER BY product_id
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3_rows
FROM stocks
ORDER BY store_id, product_id;
```

1번 매장의 일부 결과는 다음과 같다.

| store_id | product_id | quantity | 계산 대상 | moving_avg_3_rows |
|---:|---:|---:|---|---:|
| 1 | 1 | 27 | 27 | 27.00 |
| 1 | 2 | 5 | 27, 5 | 16.00 |
| 1 | 3 | 6 | 27, 5, 6 | 12.67 |
| 1 | 4 | 10 | 5, 6, 10 | 7.00 |

상품 4로 이동하면 상품 1은 계산 범위에서 빠지고 상품 2~4가 계산에 포함된다.

```text
상품 1 → [상품 1]
상품 2 → [상품 1, 상품 2]
상품 3 → [상품 1, 상품 2, 상품 3]
상품 4 → [상품 2, 상품 3, 상품 4]
```

현재 행이 이동하면서 계산 범위도 함께 이동하는 것이 이동평균의 핵심이다.

---

## 7. 누적합과 이동평균의 프레임 차이

| 계산 | 프레임 | 계산 범위 |
|---|---|---|
| 누적합 | `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` | 첫 행부터 현재 행까지 계속 확장 |
| 최근 3개 행 이동평균 | `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW` | 현재 행과 바로 앞의 두 행으로 이동 |

```text
[누적합]
행 1: [1]
행 2: [1, 2]
행 3: [1, 2, 3]
행 4: [1, 2, 3, 4]

[최근 3개 행 이동평균]
행 1: [1]
행 2: [1, 2]
행 3: [1, 2, 3]
행 4: [2, 3, 4]
```

누적 계산인지 최근 N개 행 계산인지에 따라 프레임을 명시적으로 선택해야 한다.

---

## 8. 프레임을 생략했을 때의 주의점

윈도우 함수에 `ORDER BY`만 지정하고 프레임을 생략하면 PostgreSQL은 기본적으로 다음과 같은 프레임을 적용할 수 있다.

```sql
RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

이 범위는 처음부터 현재 행까지를 의미하므로 `AVG()`를 사용하면 최근 N개 행의 이동평균이 아니라 누적평균이 계산된다.

```text
ORDER BY만 지정하고 프레임 생략
→ 기본적으로 첫 행부터 현재 행까지
→ 누적평균

최근 N개 행을 계산하고 싶음
→ ROWS BETWEEN N PRECEDING AND CURRENT ROW 명시
→ 이동평균
```

원하는 계산 범위를 명확히 표현하려면 프레임을 직접 작성하는 것이 좋다.

---

## 9. LAG(): 이전 행의 값 가져오기

`LAG()`는 윈도우 내부의 `ORDER BY` 순서를 기준으로 현재 행보다 앞에 있는 행의 값을 가져온다.

기본 구조는 다음과 같다.

```sql
LAG(column_name) OVER (
    ORDER BY order_column
)
```

```text
ORDER BY
→ 어떤 순서에서 이전 행을 정할 것인가?

LAG(column_name)
→ 그 순서에서 이전 행의 어떤 값을 가져올 것인가?
```

시간 순서에서 전일 값이나 전월 값을 가져와 현재 값과 비교할 때 활용할 수 있다.

---

## 10. 날짜별 주문 건수 계산

전일 대비 주문 건수의 변화를 계산하려면 먼저 날짜별 주문 수를 집계한다.

```sql
WITH daily_orders AS (
    SELECT
        order_date,
        COUNT(*) AS order_count
    FROM orders
    GROUP BY order_date
)
SELECT
    order_date,
    order_count
FROM daily_orders
ORDER BY order_date;
```

예시 데이터는 다음과 같다.

| order_date | order_count |
|---|---:|
| 2016-01-01 | 155 |
| 2016-01-02 | 138 |
| 2016-01-03 | 128 |

이 집계 결과를 시간 순서대로 배열한 뒤 이전 행의 값을 가져와야 한다.

---

## 11. 이전 날짜의 주문 건수 가져오기

```sql
WITH daily_orders AS (
    SELECT
        order_date,
        COUNT(*) AS order_count
    FROM orders
    GROUP BY order_date
)
SELECT
    order_date,
    order_count,
    LAG(order_count) OVER (
        ORDER BY order_date
    ) AS prev_order_count
FROM daily_orders
ORDER BY order_date;
```

`ORDER BY order_date`가 날짜 순서를 정하고, `LAG(order_count)`가 그 순서에서 바로 이전 행의 주문 건수를 가져온다.

```text
2016-01-01 | 155
    ↓
2016-01-02 | 138 ← 이전 값 155
    ↓
2016-01-03 | 128 ← 이전 값 138
```

---

## 12. 이전 행 대비 증감량 계산

현재 주문 건수에서 이전 행의 주문 건수를 빼면 증감량을 구할 수 있다.

```sql
order_count
- LAG(order_count) OVER (ORDER BY order_date)
```

전체 쿼리는 다음과 같다.

```sql
WITH daily_orders AS (
    SELECT
        order_date,
        COUNT(*) AS order_count
    FROM orders
    GROUP BY order_date
)
SELECT
    order_date,
    order_count,
    LAG(order_count) OVER (
        ORDER BY order_date
    ) AS prev_order_count,
    order_count
        - LAG(order_count) OVER (
            ORDER BY order_date
        ) AS order_diff
FROM daily_orders
ORDER BY order_date;
```

| order_date | order_count | prev_order_count | order_diff |
|---|---:|---:|---:|
| 2016-01-01 | 155 | `NULL` | `NULL` |
| 2016-01-02 | 138 | 155 | -17 |
| 2016-01-03 | 128 | 138 | -10 |

```text
2016-01-02: 138 - 155 = -17 → 이전 행보다 17건 감소
2016-01-03: 128 - 138 = -10 → 이전 행보다 10건 감소
```

양수이면 이전 행보다 증가했고, 음수이면 감소했으며, 0이면 변화가 없다고 해석할 수 있다.

---

## 13. 첫 번째 행이 NULL인 이유

정렬된 결과의 첫 번째 행에는 이전 행이 존재하지 않는다.

```text
2016-01-01 | 155 | 이전 행 없음
```

따라서 첫 번째 행에서 `LAG(order_count)`의 결과는 `NULL`이다. `NULL`과 산술 연산한 증감량도 `NULL`로 나타난다.

| order_date | order_count | prev_order_count | order_diff |
|---|---:|---:|---:|
| 2016-01-01 | 155 | `NULL` | `NULL` |

이는 오류가 아니라 비교할 이전 값이 없다는 의미이다.

---

## 14. LAG(column, n)

`LAG`의 두 번째 인자를 지정하면 현재 행을 기준으로 몇 행 이전의 값을 가져올지 정할 수 있다.

```sql
LAG(column_name, n) OVER (
    ORDER BY order_column
)
```

- `n`을 생략하면 기본값은 1이다.
- `LAG(value)`는 바로 이전 행의 값을 가져온다.
- `LAG(value, 2)`는 두 행 이전의 값을 가져온다.

```text
LAG(value)    → 1개 이전 행
LAG(value, 2) → 2개 이전 행
```

비교하려는 기간과 데이터의 한 행 단위를 고려하여 간격을 선택해야 한다.

---

## 15. LEAD(): 다음 행의 값 가져오기

`LEAD()`는 현재 행을 기준으로 다음 행의 값을 가져온다.

```sql
LEAD(column_name) OVER (
    ORDER BY order_column
)
```

두 번째 인자를 지정하면 N개 다음 행의 값을 가져올 수 있다.

```sql
LEAD(column_name, n) OVER (
    ORDER BY order_column
)
```

| 함수 | 가져오는 값 |
|---|---|
| `LAG(column)` | 바로 이전 행의 값 |
| `LAG(column, n)` | n개 이전 행의 값 |
| `LEAD(column)` | 바로 다음 행의 값 |
| `LEAD(column, n)` | n개 다음 행의 값 |

`LAG`와 `LEAD`는 전일·전월 대비나 다음 시점과의 비교처럼 정렬 순서상 인접한 값을 참조할 때 사용한다.

---

## 16. ORDER BY가 비교 대상을 결정한다

`LAG`와 `LEAD`에서 “이전”과 “다음”은 데이터의 물리적인 저장 위치가 아니라 `OVER()` 안의 `ORDER BY`가 만든 순서를 기준으로 한다.

```sql
LAG(order_count) OVER (
    ORDER BY order_date
)
```

```text
ORDER BY order_date ASC
→ 날짜가 빠른 행에서 늦은 행 순서
→ LAG는 이전 날짜 방향의 행 참조
```

정렬 기준이 잘못되거나 빠지면 원하는 시점 비교와 다른 결과가 나올 수 있다. 따라서 다음을 확인해야 한다.

- 한 행이 일, 월, 주문 중 어떤 단위를 의미하는가?
- 어떤 컬럼으로 시간 순서를 정할 것인가?
- 그룹별 비교가 필요하다면 `PARTITION BY`가 필요한가?
- 동일한 정렬값이 여러 행에 존재하지 않는가?

---

## 17. 계산 흐름 한 번에 정리하기

전일 대비 주문 건수 변화를 계산하는 흐름은 다음과 같다.

```text
원본 orders
    ↓ GROUP BY order_date
날짜별 주문 건수
    ↓ ORDER BY order_date
날짜 순서 결정
    ↓ LAG(order_count)
이전 행의 주문 건수 가져오기
    ↓ 현재 값 - 이전 값
전일 대비 증감량 계산
```

```text
ORDER BY order_date
→ 어떤 순서로 비교할 것인가?

LAG(order_count)
→ 어떤 이전 값을 가져올 것인가?

order_count - prev_order_count
→ 무엇을 계산할 것인가?
→ 이전 행 대비 증감량
```

---

## 18. 누적·이동·비교 계산 선택 기준

| 분석 목적 | 함수·프레임 |
|---|---|
| 처음부터 현재까지의 합계 | `SUM() OVER (... ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)` |
| 최근 N개 행의 평균 | `AVG() OVER (... ROWS BETWEEN N-1 PRECEDING AND CURRENT ROW)` |
| 바로 이전 행의 값 | `LAG(column)` |
| N개 이전 행의 값 | `LAG(column, n)` |
| 바로 다음 행의 값 | `LEAD(column)` |
| 현재 값과 이전 값의 차이 | `current_value - LAG(current_value)` |

분석 목적을 먼저 정한 뒤 계산 그룹, 정렬 순서, 프레임 또는 이동 간격을 작성해야 한다.

---

## 19. 핵심 용어 정리

| 용어 | 의미 |
|---|---|
| Running Total | 정해진 순서의 첫 행부터 현재 행까지 계속 더한 누적합 |
| Moving Average | 현재 행 주변의 일정한 범위만 이용해 계산한 평균 |
| Window Frame | 현재 행을 기준으로 실제 계산에 포함할 행의 범위 |
| `UNBOUNDED PRECEDING` | 현재 파티션의 첫 번째 행 |
| `CURRENT ROW` | 현재 계산 중인 행 |
| `N PRECEDING` | 현재 행보다 N개 앞에 있는 행 |
| `N FOLLOWING` | 현재 행보다 N개 뒤에 있는 행 |
| `LAG()` | 정렬 순서에서 이전 행의 값을 가져오는 윈도우 함수 |
| `LEAD()` | 정렬 순서에서 다음 행의 값을 가져오는 윈도우 함수 |
| Period-over-Period Change | 현재 시점과 이전 시점의 차이를 이용한 증감 비교 |

---

## 20. 오늘 배운 내용 정리

### 누적합

- `SUM() OVER`를 이용하면 기존 행을 유지하면서 누적합을 계산할 수 있다.
- `PARTITION BY`는 누적합을 계산할 그룹을 나눈다.
- `ORDER BY`는 그룹 안에서 값을 누적할 순서를 정한다.
- `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`는 첫 행부터 현재 행까지를 계산 범위로 지정한다.
- 파티션이 바뀌면 누적합도 다시 처음부터 계산한다.

### 이동평균과 프레임

- 이동평균은 전체 누적이 아니라 현재 행 주변의 일정 범위만 이용한다.
- `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`는 현재 행과 이전 두 행을 의미한다.
- `ROWS`는 정렬된 결과의 실제 행 개수를 기준으로 범위를 지정한다.
- 최근 N개 행만 계산하려면 프레임을 명시적으로 작성해야 한다.
- 프레임을 생략하면 의도한 이동평균 대신 누적평균이 계산될 수 있다.

### 이전·다음 행 비교

- `LAG()`는 정렬 순서에서 이전 행의 값을 가져온다.
- `LEAD()`는 정렬 순서에서 다음 행의 값을 가져온다.
- 두 번째 인자 `n`으로 몇 행 떨어진 값을 가져올지 지정할 수 있다.
- 현재 값에서 `LAG()`로 가져온 값을 빼면 이전 행 대비 증감량을 계산할 수 있다.
- 첫 번째 행은 이전 행이 없으므로 `LAG()`와 이를 이용한 증감량이 `NULL`이다.

### 분석 시 주의점

- 이전·다음 행은 `OVER()` 안의 `ORDER BY`가 정한 순서를 기준으로 결정된다.
- 계산 전에 일별·월별 등 비교하려는 데이터 단위로 먼저 집계해야 한다.
- 그룹별 비교가 필요하면 `PARTITION BY`를 함께 사용한다.
- 결과의 양수·음수·`NULL`이 각각 어떤 비즈니스 의미인지 명확히 해석해야 한다.

---

## 한 문장 정리

> 윈도우 함수에서는 파티션·정렬·프레임으로 누적 및 이동 범위를 정의하고, `LAG`와 `LEAD`로 인접한 행의 값을 참조하여 시점 간 증감을 계산할 수 있다.
