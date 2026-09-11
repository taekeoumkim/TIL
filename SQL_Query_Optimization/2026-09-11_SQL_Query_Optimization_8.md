# 2026-09-11 순위 산출과 그룹 내 비교 함수

## 학습 목표

- `ROW_NUMBER`, `RANK`, `DENSE_RANK`의 차이를 설명할 수 있다.
- 동점 처리 방식에 따라 적절한 순위 함수를 선택할 수 있다.
- `PARTITION BY`를 사용하여 그룹별 순위를 계산할 수 있다.
- 순위를 계산한 뒤 그룹별 상위 N개 데이터를 추출할 수 있다.
- `NTILE`로 정렬된 데이터를 여러 그룹으로 나눌 수 있다.
- 순위와 구간의 의미를 비즈니스 요구사항에 맞게 해석할 수 있다.

---

## 1. 순위 함수가 필요한 이유

실무에서는 다음과 같은 요구사항이 자주 발생한다.

- 부서별 매출 상위 3명의 직원을 찾는다.
- 카테고리별 판매량 상위 2개 상품을 찾는다.
- 매장별로 재고가 많은 상품의 순위를 계산한다.
- 구매금액을 기준으로 고객을 네 개 등급으로 나눈다.

윈도우 순위 함수를 사용하면 원본 행을 유지하면서 정렬 기준에 따라 순위 또는 그룹 번호를 추가할 수 있다.

```sql
순위함수() OVER (
    PARTITION BY 그룹_기준
    ORDER BY 순위_기준
)
```

`PARTITION BY`는 누구끼리 순위를 비교할지 정하고, `ORDER BY`는 어떤 값을 기준으로 높은 순위부터 배치할지 정한다.

---

## 2. 세 가지 순위 함수

대표적인 순위 함수는 다음 세 가지이다.

- `ROW_NUMBER()`
- `RANK()`
- `DENSE_RANK()`

세 함수는 모두 `ORDER BY`에 지정한 기준에 따라 순서를 계산한다. 가장 큰 차이는 같은 값이 존재할 때 순위를 부여하는 방법이다.

```sql
SELECT
    product_id,
    quantity,
    ROW_NUMBER() OVER (
        ORDER BY quantity DESC
    ) AS row_num,
    RANK() OVER (
        ORDER BY quantity DESC
    ) AS rank_num,
    DENSE_RANK() OVER (
        ORDER BY quantity DESC
    ) AS dense_rank_num
FROM stocks
WHERE store_id = 1;
```

재고 수량이 `30, 25, 25, 20`이라면 결과는 다음과 같다.

| product_id | quantity | row_num | rank_num | dense_rank_num |
|---:|---:|---:|---:|---:|
| 10 | 30 | 1 | 1 | 1 |
| 20 | 25 | 2 | 2 | 2 |
| 30 | 25 | 3 | 2 | 2 |
| 40 | 20 | 4 | 4 | 3 |

```text
ROW_NUMBER : 1 → 2 → 3 → 4
RANK       : 1 → 2 → 2 → 4
DENSE_RANK : 1 → 2 → 2 → 3
```

---

## 3. ROW_NUMBER()

`ROW_NUMBER()`는 정렬된 각 행에 서로 다른 연속 번호를 부여한다. 기준값이 같아도 번호는 중복되지 않는다.

```sql
ROW_NUMBER() OVER (
    ORDER BY quantity DESC
)
```

재고 수량이 같은 상품 두 개가 있어도 각각 2번과 3번을 받는다.

```text
재고 30 → 1
재고 25 → 2
재고 25 → 3
재고 20 → 4
```

다음과 같은 상황에 적합하다.

- 모든 행에 고유한 순번이 필요하다.
- 동점 여부와 관계없이 정확히 N개의 행을 선택해야 한다.
- 정렬된 데이터에 일련번호를 붙이고 싶다.

동점인 행의 순서까지 일관되게 정해야 한다면 `ORDER BY`에 추가 기준을 포함해야 한다.

---

## 4. RANK()

`RANK()`는 기준값이 같으면 같은 순위를 부여하고, 동점 행의 수만큼 다음 순위를 건너뛴다.

```sql
RANK() OVER (
    ORDER BY quantity DESC
)
```

```text
재고 30 → 1위
재고 25 → 2위
재고 25 → 2위
재고 20 → 4위
```

2위가 두 개이므로 3위는 없고 다음 순위가 4위가 된다. 일반적인 경기 순위처럼 공동 순위와 순위 간격을 함께 표현할 때 사용할 수 있다.

---

## 5. DENSE_RANK()

`DENSE_RANK()`도 기준값이 같으면 같은 순위를 부여한다. 그러나 `RANK()`와 달리 다음 순위를 건너뛰지 않는다.

```sql
DENSE_RANK() OVER (
    ORDER BY quantity DESC
)
```

```text
재고 30 → 1위
재고 25 → 2위
재고 25 → 2위
재고 20 → 3위
```

공동 순위를 인정하면서 순위 번호가 중간에 비지 않아야 하는 경우에 적합하다.

---

## 6. 동점 처리 방식 비교

| 함수 | 동점 처리 | 다음 순위 | `90, 90, 85` 예시 |
|---|---|---|---|
| `ROW_NUMBER()` | 동점이어도 서로 다른 번호 부여 | 항상 1씩 증가 | 1, 2, 3 |
| `RANK()` | 동점에 같은 순위 부여 | 동점 행 수만큼 건너뜀 | 1, 1, 3 |
| `DENSE_RANK()` | 동점에 같은 순위 부여 | 건너뛰지 않고 1 증가 | 1, 1, 2 |

동점이 없다면 세 함수는 동일한 순위 결과를 반환한다. 차이는 동점 데이터가 존재할 때 나타난다.

```text
각 행에 고유한 번호가 필요
→ ROW_NUMBER()

공동 순위와 순위 간격이 필요
→ RANK()

공동 순위는 인정하지만 순위 번호가 연속이어야 함
→ DENSE_RANK()
```

순위 함수를 선택하기 전에 데이터에 동점이 발생할 수 있는지와 비즈니스에서 동점을 어떻게 다룰지 확인해야 한다.

---

## 7. PARTITION BY로 그룹별 순위 계산하기

`PARTITION BY`를 순위 함수와 함께 사용하면 전체 데이터가 아니라 그룹별로 순위를 다시 계산할 수 있다.

```sql
RANK() OVER (
    PARTITION BY department
    ORDER BY sales_amount DESC
)
```

```text
PARTITION BY department
→ 부서마다 별도의 순위 계산 그룹 생성

ORDER BY sales_amount DESC
→ 각 부서 안에서 매출이 높은 순서로 순위 계산
```

파티션이 달라지면 순위는 다시 1부터 시작한다.

```text
부서 A: 직원 1 → 1위
        직원 2 → 2위
        직원 3 → 3위

부서 B: 직원 4 → 1위
        직원 5 → 2위
        직원 6 → 3위
```

`ORDER BY`의 `ASC`와 `DESC`에 따라 무엇이 1위가 되는지가 달라지므로 비즈니스 요구사항에 맞는 정렬 방향을 확인해야 한다.

---

## 8. 카테고리별 판매량 순위

상품의 카테고리는 `products.category_id`, 판매 수량은 `order_items.quantity`에 저장되어 있다.

먼저 `GROUP BY`로 상품별 총 판매 수량을 계산하고, 그 결과에 윈도우 함수를 적용해 카테고리별 순위를 계산할 수 있다.

```sql
SELECT
    p.category_id,
    p.product_id,
    p.product_name,
    SUM(oi.quantity) AS total_quantity,
    RANK() OVER (
        PARTITION BY p.category_id
        ORDER BY SUM(oi.quantity) DESC
    ) AS category_rank
FROM order_items AS oi
JOIN products AS p
  ON oi.product_id = p.product_id
GROUP BY
    p.category_id,
    p.product_id,
    p.product_name;
```

처리 흐름은 다음과 같다.

```text
order_items + products JOIN
    ↓
GROUP BY로 상품별 판매량 계산
    ↓
PARTITION BY category_id로 카테고리 구분
    ↓
판매량 내림차순으로 RANK 계산
```

```text
카테고리 1: 상품 A → 1위
             상품 B → 2위
             상품 C → 3위

카테고리 2: 상품 D → 1위
             상품 E → 2위
             상품 F → 3위
```

---

## 9. 그룹별 상위 N개 추출

윈도우 함수는 `WHERE`절보다 나중에 계산되므로 같은 SELECT문의 `WHERE`에서 계산한 순위 별칭을 바로 사용할 수 없다.

순위를 계산한 결과를 서브쿼리나 CTE로 감싼 뒤 바깥쪽 쿼리에서 필터링해야 한다.

```sql
SELECT *
FROM (
    SELECT
        p.category_id,
        p.product_id,
        p.product_name,
        SUM(oi.quantity) AS total_quantity,
        RANK() OVER (
            PARTITION BY p.category_id
            ORDER BY SUM(oi.quantity) DESC
        ) AS category_rank
    FROM order_items AS oi
    JOIN products AS p
      ON oi.product_id = p.product_id
    GROUP BY
        p.category_id,
        p.product_id,
        p.product_name
) AS ranked_products
WHERE category_rank <= 2;
```

```text
안쪽 쿼리
→ 카테고리별 판매량 순위 계산
    ↓
바깥쪽 쿼리
→ category_rank <= 2 필터링
    ↓
카테고리별 상위 2위 상품 조회
```

---

## 10. 상위 N개와 동점의 관계

“상위 2개”와 “2위 이내”는 동점이 있을 때 서로 다른 의미가 될 수 있다.

- `ROW_NUMBER() <= 2`: 각 그룹에서 정확히 두 행을 선택한다.
- `RANK() <= 2`: 공동 2위가 여러 개면 두 행보다 많이 선택될 수 있다.
- `DENSE_RANK() <= 2`: 서로 다른 상위 두 값에 해당하는 모든 행이 선택될 수 있다.

| 비즈니스 요구사항 | 적합한 함수 |
|---|---|
| 동점과 관계없이 정확히 N개 선택 | `ROW_NUMBER()` |
| 공동 순위를 모두 포함하고 일반적인 순위 간격 유지 | `RANK()` |
| 상위 N개의 서로 다른 등급·값을 모두 포함 | `DENSE_RANK()` |

따라서 그룹별 Top N을 구할 때는 단순히 `rank <= N`을 작성하기 전에 동점자를 포함할지 결정해야 한다.

---

## 11. NTILE(N)

`NTILE(N)`은 정렬된 행을 N개의 그룹으로 최대한 균등하게 나누는 윈도우 함수이다.

```sql
NTILE(4) OVER (
    ORDER BY total_amount DESC
)
```

구매금액이 높은 순서로 고객을 정렬한 뒤 네 그룹으로 나누면 다음처럼 해석할 수 있다.

```text
구매금액 높은 고객
    ↓
1번 그룹
    ↓
2번 그룹
    ↓
3번 그룹
    ↓
4번 그룹
    ↓
구매금액 낮은 고객
```

1번 그룹은 상대적으로 구매금액이 높은 고객 집단으로 활용할 수 있다.

---

## 12. 고객 구매금액 기준 4개 그룹 만들기

고객별 총 구매금액을 바로 확인할 수 없다면 CTE에서 먼저 집계한 뒤 `NTILE(4)`를 적용한다.

```sql
WITH customer_totals AS (
    SELECT
        o.customer_id,
        SUM(
            oi.quantity
            * oi.list_price
            * (1 - oi.discount)
        ) AS total_amount
    FROM orders AS o
    JOIN order_items AS oi
      ON o.order_id = oi.order_id
    GROUP BY o.customer_id
)
SELECT
    customer_id,
    total_amount,
    NTILE(4) OVER (
        ORDER BY total_amount DESC
    ) AS customer_group
FROM customer_totals;
```

### 처리 과정

1. `orders`와 `order_items`를 주문 번호로 연결한다.
2. 고객별 총 구매금액을 계산한다.
3. 구매금액을 내림차순으로 정렬한다.
4. 정렬된 고객을 최대한 비슷한 인원수의 네 그룹으로 나눈다.

---

## 13. NTILE 해석 시 주의할 점

`NTILE(4)`는 금액의 최솟값과 최댓값 사이를 동일한 금액 구간으로 나누는 함수가 아니다. 정렬된 **행의 개수**를 기준으로 최대한 균등하게 나눈다.

```text
NTILE(4)
→ 구매금액 범위를 동일한 간격으로 4등분: X
→ 정렬된 고객 수를 최대한 균등하게 4개 그룹으로 분할: O
```

| 구분 | 의미 |
|---|---|
| 금액 구간 분할 | 금액 범위 자체를 일정한 간격으로 나눔 |
| `NTILE(4)` | 금액순으로 정렬된 고객 행을 비슷한 수의 네 집단으로 나눔 |

고객 등급이나 상·중·하 구간으로 사용할 때는 그룹 번호가 고객 수 기준이라는 점을 비즈니스 해석에 반영해야 한다.

---

## 14. 매장별 매출 순위

매장별 총 매출을 비교하려면 먼저 매출을 집계한 뒤 순위 함수를 적용한다.

```sql
WITH store_sales AS (
    SELECT
        o.store_id,
        SUM(
            oi.quantity
            * oi.list_price
            * (1 - oi.discount)
        ) AS total_sales
    FROM orders AS o
    JOIN order_items AS oi
      ON o.order_id = oi.order_id
    GROUP BY o.store_id
)
SELECT
    store_id,
    total_sales,
    RANK() OVER (
        ORDER BY total_sales DESC
    ) AS sales_rank
FROM store_sales;
```

```text
매장별 판매 데이터
    ↓
매장별 총 매출 계산
    ↓
매출 내림차순 정렬
    ↓
RANK로 매출 순위 계산
```

두 매장의 매출이 같으면 같은 순위를 받는다. 2위가 두 매장이라면 다음 매장은 3위가 아니라 4위가 된다.

---

## 15. 순위 결과의 비즈니스 해석

순위 숫자만 계산하는 것보다 그 숫자가 어떤 의사결정에 사용되는지 정의하는 것이 중요하다.

### 상품 순위

- 카테고리별 인기 상품 진열
- 재고 확보 우선순위 설정
- 상위 판매 상품 프로모션

### 직원·부서 순위

- 부서별 성과 비교
- 인센티브 대상 선정
- 우수 사례 분석

### 고객 구간

- 구매금액 상위 고객군 정의
- 그룹별 마케팅 전략 수립
- 고객 등급별 혜택 설계

함수 선택에 따라 결과 대상 수와 동점 처리 방식이 달라지므로 다음 내용을 먼저 결정해야 한다.

- 공동 순위를 허용하는가?
- 정확히 N개의 행만 필요한가?
- 순위 번호가 연속이어야 하는가?
- 같은 기준값을 가진 모든 대상을 포함해야 하는가?
- 등급을 값의 범위로 나눌 것인가, 인원수로 나눌 것인가?

---

## 16. 순위 함수 선택 체크리스트

| 확인 질문 | 선택 기준 |
|---|---|
| 각 행에 고유한 번호가 필요한가? | `ROW_NUMBER()` |
| 공동 순위를 인정하고 다음 순위를 건너뛰어야 하는가? | `RANK()` |
| 공동 순위를 인정하면서 순위 번호를 연속으로 유지해야 하는가? | `DENSE_RANK()` |
| 그룹마다 순위를 다시 시작해야 하는가? | `PARTITION BY` 추가 |
| 그룹별 상위 N개를 추출해야 하는가? | 서브쿼리 또는 CTE에서 순위 계산 후 바깥에서 필터링 |
| 정렬된 행을 비슷한 크기의 N개 집단으로 나누어야 하는가? | `NTILE(N)` |
| 상위 N개에서 동점자를 모두 포함해야 하는가? | `RANK` 또는 `DENSE_RANK`의 의미 검토 |
| 정확히 N개만 반환해야 하는가? | `ROW_NUMBER`와 동점 정렬 기준 검토 |

---

## 17. 핵심 용어 정리

| 용어 | 의미 |
|---|---|
| `ROW_NUMBER()` | 동점 여부와 관계없이 각 행에 고유한 연속 번호를 부여하는 함수 |
| `RANK()` | 동점에 같은 순위를 부여하고 동점 수만큼 다음 순위를 건너뛰는 함수 |
| `DENSE_RANK()` | 동점에 같은 순위를 부여하되 다음 순위를 건너뛰지 않는 함수 |
| `PARTITION BY` | 전체 데이터를 그룹으로 나누어 각 그룹 안에서 윈도우 계산을 수행하는 절 |
| Window `ORDER BY` | 순위 또는 구간 계산의 기준과 방향을 정하는 절 |
| Top N | 정렬 또는 순위 기준으로 상위 N개 대상을 추출하는 방식 |
| `NTILE(N)` | 정렬된 행을 N개의 그룹으로 최대한 균등하게 나누는 함수 |
| Tie | 순위 기준값이 같은 동점 상태 |

---

## 18. 오늘 배운 내용 정리

### 순위 함수의 차이

- `ROW_NUMBER()`는 동점이어도 각 행에 서로 다른 번호를 부여한다.
- `RANK()`는 동점에 같은 순위를 부여하고 다음 순위를 건너뛴다.
- `DENSE_RANK()`는 동점에 같은 순위를 부여하지만 다음 순위를 건너뛰지 않는다.
- 동점이 없으면 세 함수의 결과는 동일하다.

### 그룹별 순위

- `PARTITION BY`를 사용하면 그룹마다 순위를 다시 1부터 계산한다.
- `ORDER BY`는 그룹 안에서 순위를 정할 기준과 방향을 결정한다.
- 상품별 판매량처럼 먼저 집계가 필요하면 `GROUP BY` 결과에 순위 함수를 적용할 수 있다.
- 그룹별 상위 N개는 순위 계산 결과를 서브쿼리나 CTE로 감싼 뒤 바깥에서 필터링한다.

### NTILE

- `NTILE(N)`은 정렬된 행을 N개 그룹으로 최대한 균등하게 나눈다.
- `ORDER BY`는 어느 행이 앞쪽 그룹에 들어갈지 결정한다.
- `NTILE(4)`는 값을 같은 금액 간격으로 나누는 것이 아니라 행 수를 기준으로 나눈다.
- 고객 등급이나 분위 집단을 만들 때 그룹 번호의 의미를 명확히 정의해야 한다.

### 비즈니스 해석

- 순위 함수를 선택하기 전에 동점 처리 규칙을 정해야 한다.
- `ROW_NUMBER`는 정확히 N개를 선택할 때, `RANK`는 공동 순위를 반영할 때 적합하다.
- 같은 “상위 N” 요청이라도 선택한 함수에 따라 결과 행 수가 달라질 수 있다.
- 순위와 그룹은 실제 의사결정 기준과 연결하여 해석해야 한다.

---

## 한 문장 정리

> 순위 함수는 동점 처리 규칙과 그룹 범위를 명확히 정한 뒤 사용해야 하며, `PARTITION BY`로 그룹별 순위를 계산하고 `NTILE`로 정렬된 대상을 균등한 집단으로 나눌 수 있다.
