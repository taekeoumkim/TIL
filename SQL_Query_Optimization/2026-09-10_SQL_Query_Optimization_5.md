# 2026-09-10 CTE와 중첩 서브쿼리

## 학습 목표

- CTE와 중첩 서브쿼리의 구조적 차이를 설명할 수 있다.
- CTE를 사용해 복잡한 SQL을 의미 있는 단계로 분리할 수 있다.
- CTE의 재사용 구조를 이해할 수 있다.
- SQL 작성 구조와 실제 실행 방식이 다를 수 있음을 이해할 수 있다.
- 인라이닝과 구체화의 차이를 설명할 수 있다.
- 가독성·재사용성·실행계획을 고려하여 적합한 쿼리 구조를 선택할 수 있다.

---

## 1. CTE(Common Table Expression)

CTE는 `WITH`절을 사용하여 SQL 안에 임시 결과 집합을 정의하고 이름을 붙이는 문법이다. 정의한 CTE는 같은 SQL 문 안에서 테이블처럼 참조할 수 있다.

기본 문법은 다음과 같다.

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT ...
FROM cte_name;
```

복잡한 로직의 중간 결과를 쿼리 안쪽에 계속 중첩하는 대신, 바깥으로 꺼내 의미 있는 이름을 붙인다고 생각할 수 있다.

```text
원본 데이터
    ↓
첫 번째 CTE: 계산
    ↓
두 번째 CTE: 필터링
    ↓
최종 SELECT: 결과 조회
```

CTE의 주요 특징은 다음과 같다.

- `WITH 이름 AS (...)` 형태로 정의한다.
- 같은 SQL 문 안에서 테이블처럼 참조할 수 있다.
- 하나의 `WITH`절에서 여러 CTE를 쉼표로 연결해 정의할 수 있다.
- 뒤에 정의된 CTE가 앞에서 정의한 CTE를 참조할 수 있다.
- 복잡한 계산과 필터링을 단계별로 나누어 표현할 수 있다.

---

## 2. 여러 CTE를 연결하는 구조

고객별 총 구매금액을 계산한 뒤 10,000 이상 구매한 고객만 조회한다고 가정한다.

- 주문 정보: `orders`
- 주문 수량·가격·할인율: `order_items`

상품별 구매금액은 다음과 같이 계산한다.

```text
구매금액 = 수량 × 가격 × (1 - 할인율)
```

두 단계의 CTE로 작성하면 다음과 같다.

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
),
vip_customers AS (
    SELECT
        customer_id,
        total_amount
    FROM customer_totals
    WHERE total_amount >= 10000
)
SELECT
    customer_id,
    total_amount
FROM vip_customers
ORDER BY total_amount DESC;
```

처리 흐름은 다음과 같다.

```text
orders + order_items
    ↓
customer_totals
고객별 총 구매금액 계산
    ↓
vip_customers
10,000 이상 고객 필터링
    ↓
구매금액 내림차순으로 최종 조회
```

`vip_customers`가 앞에서 정의한 `customer_totals`를 참조하는 것처럼, CTE를 연결하면 한 단계의 결과를 다음 단계의 입력으로 사용할 수 있다.

---

## 3. CTE 결과를 다른 테이블과 JOIN하기

CTE는 본문 쿼리에서 일반 테이블처럼 다른 테이블과 조인할 수 있다.

고객별 총 구매금액을 계산한 뒤 고객 이름을 추가하고, 총 구매금액이 10,000을 초과한 고객을 조회해보자.

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
    c.first_name,
    c.last_name,
    ct.total_amount
FROM customer_totals AS ct
JOIN customers AS c
  ON c.customer_id = ct.customer_id
WHERE ct.total_amount > 10000;
```

```text
orders + order_items
    ↓
customer_totals
고객별 총 구매금액 계산
    ↓
customers와 JOIN
고객 이름 추가
    ↓
10,000 초과 고객 필터링
    ↓
이름과 총 구매금액 조회
```

중간 결과에 `customer_totals`라는 이름이 있으므로 각 단계의 목적을 쉽게 파악할 수 있다.

---

## 4. 중첩 서브쿼리

중첩 서브쿼리는 다른 SQL 문 안에 포함된 쿼리이다. 안쪽 서브쿼리에서 중간 결과를 만들고 바깥쪽 쿼리가 그 결과를 사용한다.

고객별 총 구매금액을 계산하고 10,000 이상인 고객을 조회하는 로직은 다음과 같이 작성할 수 있다.

```sql
SELECT *
FROM (
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
) AS customer_totals
WHERE total_amount >= 10000;
```

```text
안쪽 서브쿼리
고객별 총 구매금액 계산
    ↓
바깥쪽 쿼리
10,000 이상 고객 필터링
```

단순한 로직이라면 서브쿼리만으로도 충분히 명확하게 표현할 수 있다. 하지만 중첩 단계가 많아지면 안쪽부터 구조를 따라가야 하므로 쿼리를 읽고 수정하기 어려워질 수 있다.

---

## 5. 같은 로직을 CTE로 작성하기

앞의 중첩 서브쿼리와 같은 로직을 CTE로 작성하면 다음과 같다.

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
SELECT *
FROM customer_totals
WHERE total_amount >= 10000;
```

두 쿼리의 작성 형태는 다르지만 수행하려는 논리적인 작업은 같다.

| 구조 | 중간 결과 계산 | 후속 처리 |
|---|---|---|
| 중첩 서브쿼리 | 안쪽 쿼리에서 고객별 구매금액 계산 | 바깥쪽 쿼리에서 필터링 |
| CTE | `customer_totals`에서 고객별 구매금액 계산 | 본문 쿼리에서 필터링 |

CTE는 중간 결과의 역할을 이름으로 드러내므로 여러 단계의 데이터 처리 흐름을 표현하기 좋다.

---

## 6. 작성 구조와 실행 구조는 다를 수 있다

SQL을 CTE 또는 중첩 서브쿼리로 작성했다고 해서 PostgreSQL이 작성된 구조와 순서를 그대로 실행하는 것은 아니다.

옵티마이저는 더 효율적으로 처리할 수 있도록 실행 방법을 결정한다.

- 일반적인 중첩 서브쿼리는 바깥쪽 쿼리와 함께 최적화될 수 있다.
- PostgreSQL 12 이상에서는 일반 CTE도 조건을 만족하면 본문 쿼리와 합쳐 최적화할 수 있다.
- 따라서 중첩 서브쿼리와 일반 CTE가 비슷한 실행계획을 가질 수 있다.

```text
SQL 작성 형태
CTE 또는 중첩 서브쿼리
    ↓
옵티마이저가 전체 구조 검토
    ↓
실제 실행계획 결정
```

즉, CTE를 사용했다고 해서 항상 CTE를 먼저 실행하고 결과를 저장한 다음 본문 쿼리를 실행하는 것은 아니다.

---

## 7. EXPLAIN으로 실행계획 비교하기

중첩 서브쿼리와 CTE의 실제 처리 방식은 `EXPLAIN`으로 확인해야 한다.

### 중첩 서브쿼리 실행계획 확인

```sql
EXPLAIN
SELECT *
FROM (
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
) AS customer_totals
WHERE total_amount >= 10000;
```

### CTE 실행계획 확인

```sql
EXPLAIN
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
SELECT *
FROM customer_totals
WHERE total_amount >= 10000;
```

CTE가 실행계획에서 별도의 `CTE Scan`으로 나타나지 않는다면, PostgreSQL이 별도의 중간 결과를 만들지 않고 본문 쿼리와 함께 최적화한 것으로 볼 수 있다.

---

## 8. 인라이닝(Inlining)

인라이닝은 CTE를 별도의 중간 결과로 만들지 않고 본문 쿼리와 합쳐 전체 쿼리를 함께 최적화하는 방식이다.

PostgreSQL 12 이상에서는 일반 CTE가 한 번만 참조되고 부작용이 없는 등의 조건을 만족하면 인라이닝하여 처리할 수 있다.

```text
[작성한 SQL]
customer_totals CTE
    ↓
본문 SELECT

[실제 처리]
CTE + 본문 쿼리
    ↓
하나의 실행계획으로 최적화
```

인라이닝되면 CTE는 SQL을 읽기 좋게 구분하는 역할을 하지만, 실행할 때 반드시 독립적인 중간 결과로 저장되지는 않는다.

---

## 9. 구체화(Materialization)

구체화는 CTE를 먼저 계산해 중간 결과로 만든 뒤, 본문 쿼리가 그 결과를 읽어 사용하는 방식이다.

PostgreSQL에서는 `MATERIALIZED`를 사용하여 CTE를 먼저 계산하도록 명시할 수 있다.

```sql
WITH customer_totals AS MATERIALIZED (
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
SELECT *
FROM customer_totals
WHERE total_amount >= 10000;
```

처리 흐름은 다음과 같다.

```text
customer_totals 계산
    ↓
중간 결과 생성
    ↓
CTE Scan
    ↓
10,000 이상 고객 필터링
```

실행계획에는 다음과 같이 `CTE Scan`이 나타날 수 있다.

```text
CTE Scan on customer_totals
  Filter: (total_amount >= '10000'::double precision)
  CTE customer_totals
    -> ...
```

계산 비용이 큰 결과를 여러 번 참조한다면 구체화를 통해 중복 계산을 줄이는 데 도움이 될 수 있다. 하지만 중간 결과를 별도로 만들어야 하므로 항상 성능이 좋아지는 것은 아니다.

---

## 10. 인라이닝과 구체화 비교

| 항목 | 인라이닝(Inlining) | 구체화(Materialization) |
|---|---|---|
| 처리 방식 | CTE와 본문 쿼리를 합쳐 최적화 | CTE를 먼저 계산해 중간 결과 생성 |
| 중간 결과 | 별도로 만들지 않음 | 별도로 만듦 |
| 실행계획 | 별도의 `CTE Scan`이 나타나지 않을 수 있음 | `CTE Scan`이 나타날 수 있음 |
| 장점 | 옵티마이저가 전체 쿼리를 함께 최적화 가능 | 여러 번 참조하는 계산의 중복을 줄일 수 있음 |
| 고려할 점 | CTE가 독립적으로 먼저 실행된다고 볼 수 없음 | 중간 결과 생성 비용이 있어 항상 유리하지 않음 |

일반 CTE의 실제 처리 방식은 조건과 옵티마이저의 판단에 따라 달라질 수 있으므로 `EXPLAIN`으로 확인해야 한다.

---

## 11. CTE와 중첩 서브쿼리 비교

| 비교 항목 | 중첩 서브쿼리 | CTE(`WITH`절) |
|---|---|---|
| 작성 구조 | 쿼리 안에 다른 쿼리를 중첩 | 중간 결과를 앞부분에 분리해 이름 부여 |
| 여러 곳에서 참조 | 동일 로직을 반복 작성해야 할 수 있음 | 같은 SQL 안에서 이름으로 여러 번 참조 가능 |
| 가독성 | 중첩이 깊어질수록 낮아질 수 있음 | 처리 단계를 나누어 표현하기 좋음 |
| 실행계획 | 바깥쪽 쿼리에 인라인될 수 있음 | 인라인 또는 구체화될 수 있음 |
| 실행계획 표기 | 전체 쿼리에 합쳐진 형태 | `CTE Scan` 또는 인라인된 형태 |
| 적합한 상황 | 단순하고 짧은 로직 | 여러 단계의 계산·필터링 또는 재사용이 필요한 로직 |

작성 방식만 보고 성능을 단정할 수는 없다. 구조를 선택한 뒤 실제 실행계획을 비교해야 한다.

---

## 12. 가독성과 유지보수성

쇼핑몰에서 다음 분석을 수행한다고 가정한다.

1. 고객별 총 구매금액을 계산한다.
2. 총 구매금액이 10,000 이상인 고객만 남긴다.
3. 구매금액이 높은 순서로 조회한다.

중첩 서브쿼리로 작성하면 다음과 같다.

```sql
SELECT *
FROM (
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
) AS customer_totals
WHERE total_amount >= 10000
ORDER BY total_amount DESC;
```

같은 로직을 CTE로 나누면 다음과 같다.

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
),
high_value_customers AS (
    SELECT
        customer_id,
        total_amount
    FROM customer_totals
    WHERE total_amount >= 10000
)
SELECT *
FROM high_value_customers
ORDER BY total_amount DESC;
```

CTE에서는 `customer_totals`, `high_value_customers`처럼 각 처리 단계의 역할을 이름으로 드러낼 수 있다.

```text
orders + order_items
    ↓
customer_totals
고객별 총 구매금액 계산
    ↓
high_value_customers
10,000 이상 고객 필터링
    ↓
구매금액 내림차순 정렬
```

성능 차이가 크지 않다면 여러 단계의 로직을 명확히 표현하는 CTE가 유지보수와 협업에 유리할 수 있다.

---

## 13. 적합한 구조를 선택하는 기준

### 중첩 서브쿼리가 적합한 경우

- 로직이 단순하고 중첩 단계가 깊지 않다.
- 중간 결과를 다른 곳에서 다시 참조할 필요가 없다.
- 서브쿼리만으로도 쿼리의 목적이 명확하다.

### CTE가 적합한 경우

- 계산과 필터링이 여러 단계로 이어진다.
- 각 처리 단계에 의미 있는 이름을 붙이고 싶다.
- 같은 중간 결과를 SQL 안에서 여러 번 참조해야 한다.
- 협업자가 쿼리의 전체 흐름을 쉽게 이해해야 한다.

### 주의할 점

- 간단한 쿼리까지 무조건 CTE로 나누면 코드가 불필요하게 길어질 수 있다.
- CTE가 지나치게 많으면 오히려 전체 흐름을 파악하기 어려워질 수 있다.
- CTE를 적절한 처리 단위로 나누어야 한다.
- 성능은 작성 형태만으로 판단하지 않고 실행계획으로 확인해야 한다.

```text
로직의 복잡도와 단계 수
        +
중간 결과의 재사용 여부
        +
가독성과 유지보수성
        +
EXPLAIN 실행계획
        ↓
CTE 또는 중첩 서브쿼리 선택
```

---

## 14. 핵심 용어 정리

| 용어 | 의미 |
|---|---|
| CTE | `WITH`절로 정의하며 같은 SQL 안에서 테이블처럼 참조하는 임시 결과 집합 |
| Nested Subquery | 다른 SQL 문 안에 포함된 하위 쿼리 |
| Inlining | CTE를 별도의 중간 결과로 만들지 않고 본문 쿼리와 합쳐 최적화하는 방식 |
| Materialization | CTE를 먼저 계산해 중간 결과로 만든 뒤 본문에서 사용하는 방식 |
| `MATERIALIZED` | CTE를 구체화하도록 명시하는 PostgreSQL 문법 |
| `CTE Scan` | 구체화된 CTE의 중간 결과를 읽는 실행계획 노드 |
| Readability | 쿼리의 처리 흐름과 의도를 쉽게 파악할 수 있는 정도 |
| Reusability | 정의한 중간 결과를 같은 SQL 안의 여러 위치에서 다시 사용하는 특성 |
| Maintainability | 쿼리를 이해하고 수정하며 관리하기 쉬운 정도 |

---

## 15. 오늘 배운 내용 정리

### CTE의 구조

- CTE는 `WITH 이름 AS (...)` 형태로 정의한다.
- 같은 SQL 안에서 CTE를 테이블처럼 참조할 수 있다.
- 여러 CTE를 연결하면 뒤쪽 CTE가 앞쪽 CTE의 결과를 사용할 수 있다.
- 복잡한 계산과 필터링을 의미 있는 단계로 나누어 표현할 수 있다.

### CTE와 중첩 서브쿼리

- 같은 로직을 CTE와 중첩 서브쿼리로 모두 표현할 수 있다.
- 중첩 서브쿼리는 단순한 로직에 충분할 수 있다.
- CTE는 여러 단계의 로직과 중간 결과의 재사용에 유리하다.
- SQL 작성 구조가 실제 실행 순서를 그대로 의미하지는 않는다.

### 최적화 특성

- 일반적인 중첩 서브쿼리는 바깥쪽 쿼리와 함께 최적화될 수 있다.
- PostgreSQL 12 이상에서는 조건을 만족하는 일반 CTE도 본문에 인라인될 수 있다.
- `MATERIALIZED`를 사용하면 CTE를 먼저 계산해 중간 결과로 만들도록 지정할 수 있다.
- 구체화는 중복 계산을 줄일 수 있지만 항상 성능을 향상시키는 것은 아니다.
- 실제 처리 방식과 성능은 `EXPLAIN`으로 확인해야 한다.

### 선택 기준

- 실행계획상 성능 차이가 크지 않다면 가독성과 유지보수성이 중요한 기준이 된다.
- 로직의 단계가 많을수록 CTE의 가독성 이점이 커질 수 있다.
- 같은 중간 결과를 여러 번 참조하면 CTE로 중복 코드를 줄일 수 있다.
- CTE를 너무 많이 나누면 흐름이 복잡해질 수 있으므로 적절한 단위로 구성해야 한다.

---

## 한 문장 정리

> CTE는 복잡한 SQL을 이름 있는 단계로 나누어 가독성과 재사용성을 높이지만, 실제 성능은 인라이닝과 구체화 여부에 따라 달라지므로 실행계획으로 확인해야 한다.