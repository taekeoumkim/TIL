# 2026-09-11 윈도우 함수 구조와 집계 함수 비교

## 학습 목표

- 윈도우 함수가 필요한 상황을 설명할 수 있다.
- `OVER`, `PARTITION BY`, `ORDER BY`, 프레임의 역할을 구분할 수 있다.
- 윈도우 함수를 이용해 그룹 통계, 누적합, 이동평균, 순위를 계산할 수 있다.
- `GROUP BY` 집계와 윈도우 함수의 결과 구조 차이를 설명할 수 있다.
- 윈도우 함수 결과를 필터링하는 방법을 이해할 수 있다.
- 실행계획에서 윈도우 함수의 데이터 읽기·정렬·계산 흐름을 파악할 수 있다.

---

## 1. 윈도우 함수(Window Function)란?

윈도우 함수는 현재 결과의 각 행을 유지하면서 여러 행을 참고해 계산한 값을 새로운 컬럼으로 추가하는 기능이다.

일반 집계 함수로 전체 평균을 계산하면 결과가 하나의 행으로 줄어든다.

```sql
SELECT AVG(quantity) AS avg_quantity
FROM stocks;
```

```text
avg_quantity
------------
20
```

반면 각 상품의 재고 행을 유지하면서 전체 평균을 함께 표시하려면 `OVER()`를 사용한다.

```sql
SELECT
    product_id,
    quantity,
    AVG(quantity) OVER () AS avg_quantity
FROM stocks;
```

| product_id | quantity | avg_quantity |
|---:|---:|---:|
| 1 | 10 | 20 |
| 2 | 20 | 20 |
| 3 | 30 | 20 |

두 표현의 차이는 다음과 같다.

```text
AVG(quantity)
→ 여러 행의 평균을 하나의 결과로 계산

AVG(quantity) OVER ()
→ 기존 행을 유지하면서 여러 행의 평균을 각 행에 표시
```

윈도우 함수는 완전히 새로운 계산 방법이라기보다 `SUM`, `AVG`, `COUNT` 등의 함수를 원본 행을 유지한 상태로 사용할 수 있게 하는 방식으로 이해할 수 있다.

---

## 2. 윈도우 함수의 기본 구조

윈도우 함수의 기본적인 형태는 다음과 같다.

```sql
함수() OVER (
    PARTITION BY 그룹_기준
    ORDER BY 정렬_기준
    ROWS 또는 RANGE 프레임_범위
)
```

모든 요소를 항상 사용할 필요는 없다. 필요한 계산에 따라 하나씩 추가한다.

| 구성 요소 | 답하는 질문 | 역할 |
|---|---|---|
| `OVER()` | 여러 행을 어떻게 참고할 것인가? | 기존 행을 유지하면서 윈도우 계산을 수행한다. |
| `PARTITION BY` | 누구끼리 계산할 것인가? | 전체 데이터를 그룹별 파티션으로 나눈다. |
| `ORDER BY` | 어떤 순서로 계산할 것인가? | 파티션 안에서 계산 순서를 정한다. |
| `ROWS`·`RANGE` | 실제로 어디까지 계산할 것인가? | 현재 행을 기준으로 계산에 포함할 프레임을 정한다. |

```text
전체 데이터
    ↓ PARTITION BY
계산할 그룹
    ↓ ORDER BY
그룹 안의 행 순서
    ↓ ROWS / RANGE
현재 행에서 참고할 범위
    ↓
함수 계산
```

---

## 3. OVER(): 기존 행을 유지하면서 계산하기

`OVER()`는 일반 함수를 윈도우 함수로 사용하는 기준이 된다.

```sql
AVG(quantity) OVER ()
```

괄호 안에 아무 조건도 지정하지 않으면 전체 결과 행을 하나의 계산 집합으로 사용한다.

```text
전체 상품의 평균 계산
    ↓
각 상품 행에 같은 전체 평균 표시
```

원본의 `product_id`, `quantity`는 사라지지 않고 평균값이 추가 컬럼으로 붙는다.

---

## 4. PARTITION BY: 누구끼리 계산할 것인가?

전체 데이터를 특정 그룹끼리 나누어 계산하려면 `PARTITION BY`를 사용한다.

다음과 같이 두 매장의 상품 재고가 있다고 가정한다.

| store_id | product_id | quantity |
|---:|---:|---:|
| 1 | 1 | 10 |
| 1 | 2 | 20 |
| 1 | 3 | 30 |
| 2 | 1 | 40 |
| 2 | 2 | 50 |
| 2 | 3 | 60 |

매장별 평균 재고를 각 상품 행에 표시하는 쿼리는 다음과 같다.

```sql
SELECT
    store_id,
    product_id,
    quantity,
    AVG(quantity) OVER (
        PARTITION BY store_id
    ) AS avg_quantity
FROM stocks;
```

| store_id | product_id | quantity | avg_quantity |
|---:|---:|---:|---:|
| 1 | 1 | 10 | 20 |
| 1 | 2 | 20 | 20 |
| 1 | 3 | 30 | 20 |
| 2 | 1 | 40 | 50 |
| 2 | 2 | 50 | 50 |
| 2 | 3 | 60 | 50 |

```text
PARTITION BY store_id
→ 누구끼리 계산할 것인가?
→ 같은 매장에 속한 상품끼리
```

`GROUP BY`의 그룹과 비슷한 기준을 만들지만, 파티션을 나눈 뒤에도 각 상품 행은 그대로 유지된다.

---

## 5. ORDER BY: 어떤 순서로 계산할 것인가?

누적합, 순위, 이전·다음 행 비교처럼 계산 순서가 중요한 경우 `OVER()` 안에 `ORDER BY`를 사용한다.

매장별로 상품 번호 순서에 따라 재고를 누적하는 쿼리는 다음과 같다.

```sql
SELECT
    store_id,
    product_id,
    quantity,
    SUM(quantity) OVER (
        PARTITION BY store_id
        ORDER BY product_id
    ) AS cumulative_quantity
FROM stocks;
```

1번 매장의 계산 흐름은 다음과 같다.

| product_id | quantity | 계산 | cumulative_quantity |
|---:|---:|---|---:|
| 1 | 10 | 10 | 10 |
| 2 | 20 | 10 + 20 | 30 |
| 3 | 30 | 10 + 20 + 30 | 60 |
| 4 | 40 | 10 + 20 + 30 + 40 | 100 |

```text
PARTITION BY store_id
→ 같은 매장끼리 계산

ORDER BY product_id
→ 상품 번호 순서로 계산
```

---

## 6. 윈도우 내부 ORDER BY와 최종 ORDER BY

`OVER()` 안의 `ORDER BY`와 `SELECT`문의 마지막 `ORDER BY`는 역할이 다르다.

```sql
SELECT
    store_id,
    product_id,
    quantity,
    SUM(quantity) OVER (
        PARTITION BY store_id
        ORDER BY product_id
    ) AS cumulative_quantity
FROM stocks
ORDER BY store_id, product_id;
```

| 위치 | 역할 |
|---|---|
| `OVER(... ORDER BY product_id)` | 윈도우 함수가 값을 계산할 순서를 정한다. |
| 쿼리 마지막 `ORDER BY store_id, product_id` | 최종 조회 결과를 출력할 순서를 정한다. |

계산 순서와 화면에 나타나는 결과 순서는 서로 다른 목적이므로 구분해야 한다.

---

## 7. 프레임: 실제로 어디까지 계산할 것인가?

파티션과 순서를 정한 뒤, 현재 행을 기준으로 실제 계산에 포함할 범위를 프레임으로 지정할 수 있다.

현재 상품과 바로 앞의 두 상품까지, 최대 세 행의 평균을 계산하는 예시는 다음과 같다.

```sql
SELECT
    store_id,
    product_id,
    quantity,
    AVG(quantity) OVER (
        PARTITION BY store_id
        ORDER BY product_id
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_quantity
FROM stocks;
```

프레임 표현의 의미는 다음과 같다.

```text
2 PRECEDING
→ 현재 행보다 앞의 2개 행

CURRENT ROW
→ 현재 행

ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
→ 앞의 2개 행부터 현재 행까지 최대 3개 행
```

| product_id | quantity | 계산에 포함되는 값 | moving_avg_quantity |
|---:|---:|---|---:|
| 1 | 10 | 10 | 10 |
| 2 | 20 | 10, 20 | 15 |
| 3 | 30 | 10, 20, 30 | 20 |
| 4 | 40 | 20, 30, 40 | 30 |

첫 번째나 두 번째 행처럼 앞에 충분한 행이 없다면 현재 존재하는 행만 사용한다. 행이 이동하면 계산 범위도 함께 이동하므로 이동평균을 구할 수 있다.

---

## 8. ROWS와 RANGE

`ROWS`와 `RANGE`는 모두 윈도우 함수의 계산 범위를 지정한다.

| 프레임 방식 | 기준 |
|---|---|
| `ROWS` | 현재 행을 중심으로 실제 행의 개수를 기준으로 범위를 정한다. |
| `RANGE` | `ORDER BY`에 지정한 값의 범위를 기준으로 계산 대상을 정한다. |

처음에는 실제 앞·뒤 행을 기준으로 이해하기 쉬운 `ROWS`를 중심으로 학습하고, 값의 범위가 필요한 경우 `RANGE`를 사용할 수 있다.

---

## 9. 윈도우 함수 구조 한 번에 읽기

다음 표현을 구성 요소별로 해석해보자.

```sql
AVG(quantity) OVER (
    PARTITION BY store_id
    ORDER BY product_id
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
)
```

```text
OVER()
→ 기존 상품 행을 유지하면서 계산

PARTITION BY store_id
→ 같은 매장의 상품끼리 계산

ORDER BY product_id
→ 상품 번호 순서로 계산

ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
→ 현재 상품과 바로 앞의 두 상품을 계산에 포함

AVG(quantity)
→ 선택된 범위의 재고 평균 계산
```

문법을 한 번에 외우기보다 다음 세 질문을 순서대로 적용하면 이해하기 쉽다.

1. 누구끼리 계산할 것인가?
2. 어떤 순서로 계산할 것인가?
3. 그중 어디까지 계산할 것인가?

---

## 10. 주문별 상품 금액과 평균 비교

한 주문에는 여러 상품이 포함될 수 있다. 각 상품의 판매 금액은 다음과 같이 계산한다.

```text
판매 금액 = quantity × list_price × (1 - discount)
```

`GROUP BY`를 사용하면 주문별 평균 판매 금액을 구할 수 있다.

```sql
SELECT
    order_id,
    AVG(
        quantity * list_price * (1 - discount)
    ) AS avg_item_amount
FROM order_items
GROUP BY order_id
ORDER BY order_id;
```

하지만 같은 주문의 여러 상품이 하나의 결과 행으로 합쳐지므로 상품별 판매 금액은 결과에서 사라진다.

상품별 행을 유지하면서 주문별 평균을 함께 표시하려면 윈도우 함수를 사용한다.

```sql
SELECT
    order_id,
    item_id,
    quantity * list_price * (1 - discount) AS amount,
    AVG(
        quantity * list_price * (1 - discount)
    ) OVER (
        PARTITION BY order_id
    ) AS avg_item_amount
FROM order_items
ORDER BY order_id, item_id;
```

`PARTITION BY order_id`는 같은 주문에 포함된 상품끼리 평균을 계산하도록 한다. 상품별 행은 유지되므로 각 상품 금액과 주문 평균을 같은 결과에서 비교할 수 있다.

| item_id | amount | avg_item_amount |
|---:|---:|---:|
| 1 | 853.10 | 2650.14 |
| 2 | 697.49 | 2650.14 |
| 3 | 4049.99 | 2650.14 |
| 4 | 4999.99 | 2650.14 |

---

## 11. 주문별 누적 판매 금액

같은 주문 안에서 상품 번호 순서대로 판매 금액을 누적할 수 있다.

```sql
SELECT
    order_id,
    item_id,
    quantity * list_price * (1 - discount) AS amount,
    SUM(
        quantity * list_price * (1 - discount)
    ) OVER (
        PARTITION BY order_id
        ORDER BY item_id
    ) AS cumulative_amount
FROM order_items
ORDER BY order_id, item_id;
```

4번 주문의 계산 예시는 다음과 같다.

| item_id | amount | cumulative_amount |
|---:|---:|---:|
| 1 | 853.10 | 853.10 |
| 2 | 697.49 | 1550.59 |
| 3 | 4049.99 | 5600.58 |
| 4 | 4999.99 | 10600.57 |

`PARTITION BY order_id`로 주문별 계산 범위를 나누고, `ORDER BY item_id`로 누적 순서를 정한다.

---

## 12. 주문별 이동평균

같은 주문 안에서 현재 상품과 앞의 두 상품만 사용해 이동평균을 계산할 수도 있다.

```sql
SELECT
    order_id,
    item_id,
    quantity * list_price * (1 - discount) AS amount,
    AVG(
        quantity * list_price * (1 - discount)
    ) OVER (
        PARTITION BY order_id
        ORDER BY item_id
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_amount
FROM order_items
ORDER BY order_id, item_id;
```

```text
상품 1 → 상품 1만 사용
상품 2 → 상품 1, 상품 2 사용
상품 3 → 상품 1, 상품 2, 상품 3 사용
상품 4 → 상품 2, 상품 3, 상품 4 사용
```

파티션이 바뀌면 프레임도 새로운 주문 안에서 다시 계산된다.

---

## 13. GROUP BY와 윈도우 함수의 근본적 차이

`GROUP BY` 집계와 윈도우 함수는 모두 여러 행을 이용해 `SUM`, `AVG`, `COUNT` 등을 계산할 수 있지만 결과 행의 형태가 다르다.

### GROUP BY로 고객별 총 구매금액 계산

```sql
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
GROUP BY o.customer_id;
```

고객별 총 구매금액은 확인할 수 있지만 어떤 주문 상품이 금액을 구성했는지는 결과에서 사라진다.

### 윈도우 함수로 고객별 총 구매금액 계산

```sql
SELECT
    o.order_id,
    o.customer_id,
    oi.product_id,
    oi.quantity,
    oi.list_price,
    SUM(
        oi.quantity
        * oi.list_price
        * (1 - oi.discount)
    ) OVER (
        PARTITION BY o.customer_id
    ) AS customer_total_amount
FROM orders AS o
JOIN order_items AS oi
  ON o.order_id = oi.order_id;
```

주문 상품별 행을 유지하면서 각 행 옆에 해당 고객의 총 구매금액을 표시한다.

```text
[GROUP BY]
고객 A의 주문 상품 1 ┐
고객 A의 주문 상품 2 ├→ 고객 A | 총 구매금액
고객 A의 주문 상품 3 ┘

[윈도우 함수]
고객 A | 주문 상품 1 | 고객 총 구매금액
고객 A | 주문 상품 2 | 고객 총 구매금액
고객 A | 주문 상품 3 | 고객 총 구매금액
```

---

## 14. GROUP BY와 윈도우 함수 비교

| 비교 항목 | `GROUP BY` 집계 | 윈도우 함수 |
|---|---|---|
| 결과 행 | 그룹별 한 행으로 압축 | 원본 행 수 유지 |
| 개별 데이터 | 그룹에 포함된 개별 행이 결과에서 사라짐 | 개별 행을 그대로 확인 가능 |
| 그룹 통계 | 그룹별 요약값을 반환 | 각 행에 그룹 통계를 추가 |
| 주요 목적 | 그룹별 요약 데이터 생성 | 개별 데이터와 그룹 통계를 함께 비교 |
| 예시 | 고객별 총 구매금액만 조회 | 주문 상품별 정보와 고객 총 구매금액을 함께 조회 |

```text
그룹별 요약 결과만 필요
→ GROUP BY

개별 행과 그룹 통계를 함께 확인
→ 윈도우 함수
```

같은 `SUM`이나 `AVG`를 사용하더라도 `OVER()`의 유무에 따라 결과의 형태가 달라진다.

---

## 15. 윈도우 함수 결과를 필터링하는 방법

윈도우 함수는 SQL 실행 순서상 `WHERE`절보다 나중에 계산된다. 따라서 같은 쿼리의 `WHERE`절에서는 아직 윈도우 함수 결과가 존재하지 않는다.

```text
WHERE로 행 필터링
    ↓
윈도우 함수 계산
    ↓
SELECT 결과 생성
```

윈도우 함수 결과를 기준으로 필터링하려면 서브쿼리나 CTE에서 먼저 값을 계산한 뒤 바깥쪽 쿼리에서 `WHERE`를 적용한다.

```sql
WITH ranked_stocks AS (
    SELECT
        store_id,
        product_id,
        quantity,
        RANK() OVER (
            PARTITION BY store_id
            ORDER BY quantity DESC
        ) AS stock_rank
    FROM stocks
)
SELECT *
FROM ranked_stocks
WHERE stock_rank <= 3;
```

```text
안쪽 CTE에서 순위 계산
    ↓
바깥쪽 쿼리에서 순위 조건으로 필터링
```

---

## 16. 대표적인 윈도우 함수 유형

윈도우 함수는 목적에 따라 다음과 같이 분류할 수 있다.

| 유형 | 대표 함수 | 활용 예시 |
|---|---|---|
| 집계형 | `SUM`, `AVG`, `COUNT` | 그룹별 합계·평균·개수를 각 행에 표시 |
| 순위형 | `ROW_NUMBER`, `RANK` | 그룹 안에서 순번 또는 순위 부여 |
| 누적 계산 | `SUM` + 윈도우 `ORDER BY` | 시간이나 상품 번호 순서의 누적합 |
| 이동 계산 | `AVG` + 프레임 | 현재 행 주변의 이동평균 |

필요한 결과에 따라 함수와 `OVER()` 내부 구성을 함께 결정한다.

---

## 17. ROW_NUMBER와 RANK

`ROW_NUMBER()`는 윈도우의 정렬 순서에 따라 각 행에 순차적인 번호를 부여한다.

```sql
ROW_NUMBER() OVER (
    PARTITION BY store_id
    ORDER BY list_price
)
```

`RANK()`는 `ORDER BY` 값이 같으면 같은 순위를 부여하고, 동점 행의 수만큼 다음 순위를 건너뛴다.

```sql
SELECT
    store_id,
    product_id,
    quantity,
    RANK() OVER (
        PARTITION BY store_id
        ORDER BY quantity DESC
    ) AS stock_rank
FROM stocks;
```

```text
PARTITION BY store_id
→ 같은 매장의 상품끼리 비교

ORDER BY quantity DESC
→ 재고가 많은 상품부터 정렬

RANK()
→ 같은 재고에는 같은 순위를 부여
```

재고 30인 상품이 10개라면 모두 1위가 되고, 다음 재고 값의 순위는 11위가 된다.

```text
재고 30 → 1위
재고 30 → 1위
...
재고 30 → 1위  (총 10개)
재고 29 → 11위
```

---

## 18. 실행계획에서 윈도우 함수 읽기

윈도우 함수의 실제 처리 방식은 `EXPLAIN`으로 확인할 수 있다.

```sql
EXPLAIN
SELECT
    store_id,
    product_id,
    quantity,
    RANK() OVER (
        PARTITION BY store_id
        ORDER BY quantity DESC
    ) AS stock_rank
FROM stocks;
```

실행계획에서는 일반적으로 다음과 같은 처리 흐름을 확인할 수 있다.

```text
Seq Scan
필요한 데이터 읽기
    ↓
Sort
PARTITION BY와 ORDER BY 계산에 필요한 순서로 정렬
    ↓
WindowAgg
정렬된 데이터를 이용해 윈도우 함수 계산
```

| 실행계획 노드 | 역할 |
|---|---|
| `Seq Scan` | `stocks` 테이블에서 계산에 필요한 데이터를 읽는다. |
| `Sort` | 매장과 재고 순위 계산에 필요한 순서로 데이터를 정렬한다. |
| `WindowAgg` | 파티션과 정렬 순서를 이용해 `RANK()` 등의 윈도우 함수를 계산한다. |

윈도우 함수에서 `PARTITION BY`와 `ORDER BY`가 사용되면 계산 전에 정렬 작업이 필요할 수 있다. 데이터가 많다면 정렬 단계가 실행 비용에 영향을 줄 수 있으므로 실행계획에서 확인해야 한다.

---

## 19. 윈도우 함수 선택 체크리스트

| 확인 질문 | 선택 방향 |
|---|---|
| 그룹별 요약값만 필요한가? | `GROUP BY`를 우선 고려한다. |
| 개별 행을 유지해야 하는가? | 윈도우 함수를 고려한다. |
| 그룹별로 계산해야 하는가? | `PARTITION BY`를 사용한다. |
| 누적합이나 순위처럼 순서가 중요한가? | 윈도우 내부 `ORDER BY`를 사용한다. |
| 현재 행 주변의 일부 행만 계산해야 하는가? | `ROWS` 또는 `RANGE` 프레임을 지정한다. |
| 윈도우 결과로 행을 걸러야 하는가? | 서브쿼리나 CTE의 바깥쪽에서 필터링한다. |
| 데이터가 많고 정렬 비용이 걱정되는가? | `EXPLAIN`으로 `Sort`와 `WindowAgg`를 확인한다. |

---

## 20. 핵심 용어 정리

| 용어 | 의미 |
|---|---|
| Window Function | 기존 행을 유지하면서 여러 행을 참고한 계산 결과를 추가하는 함수 |
| `OVER()` | 일반 함수를 윈도우 함수 형태로 사용하도록 계산 범위를 정의하는 절 |
| `PARTITION BY` | 윈도우 계산을 수행할 그룹을 나누는 기준 |
| Window `ORDER BY` | 파티션 안에서 윈도우 함수의 계산 순서를 정하는 기준 |
| Frame | 현재 행을 기준으로 실제 계산에 포함할 행 또는 값의 범위 |
| `ROWS` | 실제 행의 위치와 개수를 기준으로 프레임을 정하는 방식 |
| `RANGE` | `ORDER BY` 값의 범위를 기준으로 프레임을 정하는 방식 |
| Cumulative Sum | 정해진 순서에 따라 처음부터 현재 행까지 값을 누적한 합계 |
| Moving Average | 현재 행 주변의 일정 범위만 사용해 계산한 평균 |
| `RANK()` | 동점에 같은 순위를 부여하고 이후 순위를 건너뛰는 순위 함수 |
| `WindowAgg` | 실행계획에서 윈도우 함수 계산을 수행하는 노드 |

---

## 21. 오늘 배운 내용 정리

### 윈도우 함수의 구조

- 윈도우 함수는 원본 행을 유지하면서 여러 행을 참고해 계산한다.
- `OVER()` 안에 아무 조건이 없으면 전체 행을 함께 사용한다.
- `PARTITION BY`는 누구끼리 계산할지 정한다.
- 윈도우 내부 `ORDER BY`는 어떤 순서로 계산할지 정한다.
- `ROWS`와 `RANGE`는 실제로 어디까지 계산할지 정한다.

### 집계 함수와의 차이

- `GROUP BY`는 같은 그룹의 여러 행을 하나의 결과 행으로 압축한다.
- 윈도우 함수는 기존 행을 유지한 채 그룹 계산 결과를 각 행에 추가한다.
- 그룹별 요약 결과만 필요하면 `GROUP BY`가 적합하다.
- 개별 데이터와 그룹 통계를 동시에 확인하려면 윈도우 함수가 적합하다.

### 주요 활용

- `SUM`, `AVG`, `COUNT`를 이용해 각 행에 그룹 통계를 표시할 수 있다.
- 윈도우 `ORDER BY`를 추가하면 누적 계산을 수행할 수 있다.
- 프레임을 지정하면 이동평균과 같이 제한된 범위의 계산을 수행할 수 있다.
- `ROW_NUMBER`, `RANK` 등으로 그룹 내 순번과 순위를 계산할 수 있다.
- 윈도우 함수 결과를 필터링하려면 CTE나 서브쿼리로 감싼다.

### 실행계획

- 윈도우 함수는 데이터를 읽고 필요한 순서로 정렬한 뒤 계산하는 흐름을 가질 수 있다.
- 실행계획에서 `Seq Scan`, `Sort`, `WindowAgg` 노드를 확인할 수 있다.
- 데이터가 많으면 정렬 작업이 비용에 영향을 줄 수 있으므로 실행계획을 점검해야 한다.

---

## 한 문장 정리

> 윈도우 함수는 `PARTITION BY`, `ORDER BY`, 프레임으로 계산 범위를 정의하여 원본 행을 유지한 채 그룹 통계·누적값·이동평균·순위를 함께 보여주는 기능이다.