# 2026-09-10 다단계 테이블 결합으로 복합 데이터셋 구성

## 학습 목표

- 세 개 이상의 테이블을 단계적으로 결합하여 필요한 데이터셋을 구성할 수 있다.
- 최종 결과에 필요한 컬럼과 테이블 간 관계를 바탕으로 JOIN 순서를 설계할 수 있다.
- 관계의 필수 여부에 따라 `INNER JOIN`과 `LEFT JOIN`을 선택할 수 있다.
- 1:N 관계로 인해 결과 행이 증가하는 현상을 설명할 수 있다.
- `LEFT JOIN`에서 필터 조건을 `ON`과 `WHERE` 중 어디에 두어야 하는지 판단할 수 있다.
- 조인 전후 행 수와 키별 행 수를 비교하여 결과를 검증할 수 있다.

---

## 1. 다단계 JOIN이 필요한 이유

실무에서 필요한 정보는 하나의 테이블에 모두 저장되어 있지 않은 경우가 많다. 고객, 주문, 주문 상품, 상품 정보처럼 서로 다른 목적의 데이터가 여러 테이블에 나뉘어 저장된다.

예를 들어 “어떤 고객이 어떤 상품을 주문했는가?”를 확인하려면 다음 테이블이 필요하다.

| 테이블 | 주요 정보 |
|---|---|
| `customers` | 고객 이름과 고객 정보 |
| `orders` | 주문 번호, 고객 번호, 주문 상태 등 주문 정보 |
| `order_items` | 주문에 포함된 상품, 수량 등 주문 상세 정보 |
| `products` | 상품 이름과 상품 정보 |

이처럼 원하는 분석 단위의 데이터셋을 만들려면 테이블 간 관계를 따라 여러 번 JOIN해야 한다.

```text
customers
    ↓ customer_id
orders
    ↓ order_id
order_items
    ↓ product_id
products
```

JOIN 조건이나 필터 위치를 잘못 선택하면 데이터가 누락되거나 예상하지 못한 행 증가가 발생할 수 있으므로, SQL을 작성하기 전에 결합 구조를 먼저 설계해야 한다.

---

## 2. 다단계 JOIN 설계 절차

다단계 JOIN은 무작정 `JOIN`을 이어 붙이는 것보다 다음 순서로 설계하는 것이 안전하다.

### 1단계: 필요한 컬럼과 테이블 확인

최종 결과에 어떤 컬럼이 필요하고 각 컬럼이 어느 테이블에 저장되어 있는지 정리한다.

```text
order_id       → orders
customer name  → customers
product_name   → products
quantity       → order_items
order_status   → orders
```

### 2단계: 테이블 간 관계 확인

각 테이블이 어떤 키로 연결되고 관계가 1:1, 1:N, N:M 중 무엇인지 확인한다.

| 기준 테이블 | 연결 테이블 | 연결 컬럼 |
|---|---|---|
| `customers` | `orders` | `customer_id` |
| `orders` | `order_items` | `order_id` |
| `orders` | `staffs` | `staff_id` |
| `order_items` | `products` | `product_id` |
| `products` | `stocks` | `product_id` |

### 3단계: 기준 테이블과 JOIN 순서 결정

결과의 기준이 되는 테이블을 정하고 관계를 하나씩 따라가며 결합 순서를 정한다.

```text
customers → orders → order_items → products
```

### 4단계: JOIN 조건과 종류 선택

각 테이블 쌍을 정확한 키 컬럼으로 연결하고, 관계가 반드시 존재해야 하는지에 따라 `INNER JOIN` 또는 `LEFT JOIN`을 선택한다.

### 5단계: 필터 위치 결정

필터를 `JOIN ... ON`에 둘지 `WHERE`에 둘지 결정한다. 특히 `LEFT JOIN`에서는 위치에 따라 왼쪽 테이블의 행이 유지되는지가 달라진다.

```text
필요한 컬럼·테이블 확인
    ↓
테이블 관계 파악
    ↓
기준 테이블과 JOIN 순서 결정
    ↓
JOIN 조건과 종류 선택
    ↓
필터링 조건 배치
    ↓
결과 검증
```

---

## 3. 네 개 테이블을 이용한 주문 상세 데이터셋

고객 이름, 주문, 상품, 수량을 한 번에 조회하는 쿼리는 다음과 같다.

```sql
SELECT
    o.order_id,
    c.first_name,
    c.last_name,
    p.product_name,
    oi.quantity,
    o.order_status
FROM orders AS o
JOIN customers AS c
  ON c.customer_id = o.customer_id
JOIN order_items AS oi
  ON oi.order_id = o.order_id
JOIN products AS p
  ON p.product_id = oi.product_id;
```

각 JOIN의 역할은 다음과 같다.

1. `orders`와 `customers`를 `customer_id`로 연결하여 주문 고객을 찾는다.
2. `orders`와 `order_items`를 `order_id`로 연결하여 주문에 포함된 상품과 수량을 찾는다.
3. `order_items`와 `products`를 `product_id`로 연결하여 상품 이름을 가져온다.

```text
orders
  ├─ customers 연결 → 고객 이름
  └─ order_items 연결 → 주문 상품·수량
          └─ products 연결 → 상품 이름
```

여러 테이블을 결합하더라도 한 번에 전체를 생각하기보다 두 테이블 사이의 관계를 하나씩 확인하면 구조를 이해하기 쉽다.

---

## 4. 1:N 관계와 행 증가

하나의 주문에는 여러 상품이 포함될 수 있다. 따라서 `orders`와 `order_items`는 1:N 관계이다.

```text
orders (1) ──────── (N) order_items
주문 1건              여러 주문 상품
```

한 주문에 상품이 두 종류 포함되면 JOIN 결과에서 같은 `order_id`가 두 행으로 나타난다.

| order_id | customer | product | quantity |
|---:|---|---|---:|
| 1 | 고객 A | 상품 A | 1 |
| 1 | 고객 A | 상품 B | 2 |
| 2 | 고객 B | 상품 C | 1 |

`order_id = 1`이 두 번 나타나는 것은 잘못된 중복이 아니다. 한 주문에 상품이 두 개 포함된 관계가 정상적으로 펼쳐진 결과이다.

```text
주문 1건
  ├─ 상품 A
  └─ 상품 B

JOIN 결과
  ├─ 주문 1 + 상품 A
  └─ 주문 1 + 상품 B
```

JOIN 후 행이 증가했다면 바로 중복 오류로 판단하지 말고 먼저 테이블 간 관계와 결과 데이터의 단위를 확인해야 한다.

---

## 5. 데이터의 기준 단위(Grain)

JOIN 결과를 해석하려면 한 행이 무엇을 의미하는지 명확히 해야 한다.

| 데이터셋 | 한 행의 의미 |
|---|---|
| `orders` | 주문 1건 |
| `order_items` | 한 주문에 포함된 상품 1종 |
| 주문·상품 JOIN 결과 | 주문에 포함된 상품 1종 |

`orders`를 기준으로 보면 동일 주문이 반복된 것처럼 보이지만, 주문 상품 단위로 보면 각 행은 서로 다른 상품을 의미한다.

```text
JOIN 전: 주문 단위
    ↓ 1:N JOIN
JOIN 후: 주문 상품 단위
```

따라서 다단계 JOIN을 설계할 때는 최종 데이터셋의 한 행이 고객, 주문, 주문 상품 중 무엇을 나타내야 하는지 먼저 정해야 한다.

---

## 6. INNER JOIN과 LEFT JOIN 선택

JOIN 종류는 어떤 데이터를 결과에 반드시 유지할 것인지에 따라 선택한다.

| JOIN 종류 | 결과에 남는 행 | 적합한 상황 |
|---|---|---|
| `INNER JOIN` | 양쪽 테이블의 조건이 일치하는 행 | 관계가 존재하는 데이터만 필요할 때 |
| `LEFT JOIN` | 왼쪽 테이블의 모든 행과 일치하는 오른쪽 행 | 오른쪽 데이터가 없어도 왼쪽 데이터를 유지해야 할 때 |

전체 고객과 주문 내역을 함께 조회하되 주문하지 않은 고객도 유지하려면 `LEFT JOIN`을 사용한다.

```sql
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    o.order_id,
    o.order_date
FROM customers AS c
LEFT JOIN orders AS o
  ON c.customer_id = o.customer_id;
```

왼쪽 테이블인 `customers`의 행은 모두 유지된다. 주문이 없는 고객은 `order_id`, `order_date`와 같은 오른쪽 테이블의 값이 `NULL`로 표시된다.

```text
고객에게 주문이 있음 → 고객 정보 + 주문 정보
고객에게 주문이 없음 → 고객 정보 + 주문 컬럼 NULL
```

---

## 7. 주문 경험이 없는 고객 찾기

`LEFT JOIN` 후 오른쪽 테이블의 키가 `NULL`인 행을 찾으면 주문 경험이 없는 고객을 조회할 수 있다.

```sql
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    o.order_id,
    o.order_date
FROM customers AS c
LEFT JOIN orders AS o
  ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

`orders`와 연결되지 않은 고객은 `o.order_id`가 `NULL`이므로 결과에 남는다.

한 고객이 여러 번 주문했다면 해당 고객은 주문 수만큼 여러 행으로 나타날 수 있다. 이것 역시 `customers`와 `orders`의 1:N 관계에서 자연스럽게 발생한다.

```text
customers (1) ──────── (N) orders
고객 1명                 여러 주문
```

---

## 8. ON과 WHERE의 역할

`ON`과 `WHERE`는 모두 조건을 작성하지만 적용되는 목적이 다르다.

| 위치 | 역할 |
|---|---|
| `ON` | 어떤 데이터를 서로 연결할지 결정한다. |
| `WHERE` | JOIN이 끝난 결과에서 어떤 행을 남길지 결정한다. |

`INNER JOIN`만 사용하는 경우에는 조건을 `ON`에 두거나 `WHERE`에 두어도 결과가 같은 경우가 많다. 그러나 `LEFT JOIN`에서는 오른쪽 테이블의 조건 위치에 따라 결과가 달라질 수 있다.

---

## 9. LEFT JOIN의 오른쪽 조건을 WHERE에 작성하는 경우

전체 고객과 1번 매장 주문을 조회하는 조건을 `WHERE`에 작성해보자.

```sql
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    o.order_id,
    o.store_id
FROM customers AS c
LEFT JOIN orders AS o
  ON c.customer_id = o.customer_id
WHERE o.store_id = 1;
```

처리 흐름은 다음과 같다.

```text
전체 고객과 주문을 LEFT JOIN
    ↓
주문 없는 고객의 오른쪽 값은 NULL
    ↓
WHERE o.store_id = 1 적용
    ↓
store_id가 1이 아니거나 NULL인 행 제거
    ↓
1번 매장 주문이 있는 고객만 남음
```

`LEFT JOIN`을 사용했더라도 조인 후 `WHERE`절에서 오른쪽 테이블의 값을 제한하면, 조건에 맞지 않는 행과 `NULL` 행이 제거된다.

---

## 10. LEFT JOIN의 오른쪽 조건을 ON에 작성하는 경우

같은 조건을 `ON`에 작성하면 결과의 의미가 달라진다.

```sql
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    o.order_id,
    o.store_id
FROM customers AS c
LEFT JOIN orders AS o
  ON c.customer_id = o.customer_id
 AND o.store_id = 1;
```

`ON`의 `o.store_id = 1`은 `orders`에서 어떤 행을 고객과 연결할지 제한한다. 왼쪽 테이블의 고객 자체를 제거하지는 않는다.

```text
customers 전체 유지
    ↓
1번 매장 주문이 있음 → 해당 주문 연결
1번 매장 주문이 없음 → 주문 정보 NULL
```

따라서 1번 매장에서 주문하지 않은 고객도 결과에 남고, 해당 고객의 `order_id`와 `store_id`는 `NULL`로 나타난다.

---

## 11. ON과 WHERE 조건 비교

| 요구사항 | 조건 위치 | 결과 |
|---|---|---|
| 1번 매장 주문이 있는 고객만 조회 | `WHERE o.store_id = 1` | 조건에 맞지 않는 고객과 주문 없는 고객 제외 |
| 전체 고객을 유지하면서 1번 매장 주문만 연결 | `ON ... AND o.store_id = 1` | 모든 고객 유지, 연결할 주문이 없으면 `NULL` |

```sql
-- 1번 매장 주문이 있는 고객만 결과에 남긴다.
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    o.order_id,
    o.store_id
FROM customers AS c
LEFT JOIN orders AS o
  ON c.customer_id = o.customer_id
WHERE o.store_id = 1;
```

```sql
-- 모든 고객을 유지하면서 1번 매장 주문만 연결한다.
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    o.order_id,
    o.store_id
FROM customers AS c
LEFT JOIN orders AS o
  ON c.customer_id = o.customer_id
 AND o.store_id = 1;
```

조건 위치는 문법적인 선택이 아니라 최종 데이터셋에 어떤 행을 남길 것인지 결정하는 설계 요소이다.

---

## 12. JOIN 결과 검증의 필요성

여러 테이블을 결합한 뒤에는 결과가 나왔다는 사실만 확인해서는 안 된다. 행 수가 예상과 맞는지, 증가한 이유가 테이블 관계로 설명되는지 검증해야 한다.

검증의 기본 순서는 다음과 같다.

```text
기준 테이블의 전체 행 수 확인
    ↓
JOIN 결과의 전체 행 수 확인
    ↓
기준 키별 결과 행 수 확인
    ↓
행이 많은 개별 사례 조회
    ↓
테이블 관계와 결과 단위로 설명 가능한지 판단
```

---

## 13. 조인 전후 행 수 비교

먼저 `orders` 테이블의 주문 수를 확인한다.

```sql
SELECT COUNT(*) AS order_count
FROM orders;
```

자료의 데이터에서는 주문이 500,000건이다.

```text
order_count
-----------
500000
```

주문 상품과 상품 정보를 결합한 결과의 행 수를 확인한다.

```sql
SELECT COUNT(*) AS joined_row_count
FROM orders AS o
JOIN order_items AS oi
  ON oi.order_id = o.order_id
JOIN products AS p
  ON p.product_id = oi.product_id;
```

```text
joined_row_count
----------------
1539911
```

주문은 500,000건이지만 JOIN 결과는 1,539,911행이다. 이는 `orders`와 `order_items`가 1:N 관계이기 때문에 발생할 수 있는 정상적인 증가이다.

---

## 14. 기준 키별 행 수 확인

어떤 주문이 여러 행으로 나타나는지 확인하려면 `order_id`별로 그룹화한다.

```sql
SELECT
    o.order_id,
    COUNT(*) AS row_count
FROM orders AS o
JOIN order_items AS oi
  ON oi.order_id = o.order_id
JOIN products AS p
  ON p.product_id = oi.product_id
GROUP BY o.order_id
HAVING COUNT(*) > 1
ORDER BY row_count DESC;
```

이 쿼리는 다음 작업을 수행한다.

- `order_id`별 JOIN 결과 행 수를 계산한다.
- 두 행 이상으로 나타난 주문만 남긴다.
- 결과 행이 많은 주문부터 정렬한다.

특정 주문이 여러 번 나타나는 원인이 여러 상품 때문인지 개별 데이터를 통해 확인할 수 있다.

```sql
SELECT
    oi.order_id,
    oi.product_id,
    p.product_name,
    oi.quantity
FROM order_items AS oi
JOIN products AS p
  ON p.product_id = oi.product_id
WHERE oi.order_id = 6;
```

자료에서는 `order_id = 6`에 상품 5개가 포함되어 있으므로 JOIN 결과에서도 주문 6번이 5행으로 나타난다. 이는 중복 오류가 아니라 주문 상품 단위로 펼쳐진 정상적인 결과이다.

---

## 15. 주문별 상품 수 분포 확인

주문별 JOIN 결과 행 수가 전체적으로 어떻게 분포하는지 확인할 수도 있다.

```sql
SELECT
    product_count,
    COUNT(*) AS order_count
FROM (
    SELECT
        order_id,
        COUNT(*) AS product_count
    FROM order_items
    GROUP BY order_id
) AS order_product_counts
GROUP BY product_count
ORDER BY product_count;
```

자료의 결과는 다음과 같다.

| 주문에 포함된 상품 수 | 주문 수 |
|---:|---:|
| 1개 | 39,868건 |
| 2개 | 99,965건 |
| 3개 | 190,434건 |
| 4개 | 119,854건 |
| 5개 | 49,879건 |

주문마다 1~5개의 상품이 있으므로 주문 상세 단위의 JOIN 결과가 원래 주문 수보다 많아지는 것이 자연스럽다는 사실을 확인할 수 있다.

---

## 16. JOIN 결과 검증 체크리스트

| 확인 항목 | 확인 질문 |
|---|---|
| 최종 결과의 단위 | 결과의 한 행은 고객, 주문, 주문 상품 중 무엇인가? |
| 테이블 관계 | 각 JOIN은 1:1, 1:N, N:M 중 어떤 관계인가? |
| 연결 키 | 각 테이블을 정확한 ID 컬럼으로 연결했는가? |
| JOIN 종류 | 관계가 없는 행도 유지해야 하는가? |
| NULL 처리 | 선택적 관계에서 오른쪽 데이터가 없을 때 `NULL`을 허용하는가? |
| 필터 위치 | `LEFT JOIN`의 오른쪽 조건을 `ON`과 `WHERE` 중 의도에 맞게 배치했는가? |
| 전체 행 수 | JOIN 전후 행 수가 어떻게 변했는가? |
| 키별 행 수 | 특정 기준 키가 예상보다 많이 반복되는가? |
| 개별 사례 | 반복 행이 실제 관계로 설명되는가? |

JOIN 결과의 행이 증가했다고 무조건 `DISTINCT`로 제거하면 정상적인 1:N 관계의 정보까지 잃을 수 있다. 먼저 관계와 결과 단위를 확인해야 한다.

---

## 17. 핵심 용어 정리

| 용어 | 의미 |
|---|---|
| Multistage JOIN | 세 개 이상의 테이블을 관계에 따라 단계적으로 결합하는 방식 |
| Join Key | 두 테이블의 행을 연결하는 기준 컬럼 |
| `INNER JOIN` | 양쪽 테이블에서 조건이 일치하는 행만 반환하는 JOIN |
| `LEFT JOIN` | 왼쪽 테이블의 모든 행을 유지하고 일치하는 오른쪽 데이터를 연결하는 JOIN |
| 1:N Relationship | 기준 테이블의 한 행이 상대 테이블의 여러 행과 연결되는 관계 |
| Grain | 데이터셋에서 한 행이 의미하는 기준 단위 |
| `ON` Condition | 어떤 행끼리 연결할지 지정하는 조건 |
| `WHERE` Condition | JOIN이 완료된 결과에서 남길 행을 지정하는 조건 |
| `NULL` | `LEFT JOIN`에서 연결되는 오른쪽 데이터가 없음을 나타내는 값 |
| Row Count Validation | JOIN 전후의 행 수와 키별 행 수를 비교하는 검증 방법 |

---

## 18. 오늘 배운 내용 정리

### 다단계 JOIN 설계

- 최종 결과에 필요한 컬럼이 어느 테이블에 있는지 먼저 확인한다.
- 테이블 간 연결 키와 1:1, 1:N, N:M 관계를 파악한다.
- 결과의 기준 테이블을 정한 뒤 관계를 하나씩 따라 JOIN 순서를 설계한다.
- 관계의 필수 여부에 맞게 `INNER JOIN`과 `LEFT JOIN`을 선택한다.

### 1:N 관계와 결과 행

- 1:N 관계를 JOIN하면 기준 테이블의 한 행이 여러 행으로 나타날 수 있다.
- 한 주문에 여러 상품이 있으면 주문 상품 수만큼 동일한 주문 번호가 반복된다.
- JOIN 후 행이 늘어났다는 이유만으로 중복 오류라고 판단해서는 안 된다.
- 결과 데이터의 한 행이 무엇을 의미하는지 기준 단위를 확인해야 한다.

### ON과 WHERE

- `ON`은 어떤 데이터를 연결할지 결정한다.
- `WHERE`는 JOIN이 끝난 결과에서 어떤 데이터를 남길지 결정한다.
- `LEFT JOIN`에서 오른쪽 테이블 조건을 `WHERE`에 두면 `NULL` 행이 제거될 수 있다.
- 왼쪽 데이터를 모두 유지하면서 특정 오른쪽 데이터만 연결하려면 조건을 `ON`에 둔다.

### 결과 검증

- 기준 테이블과 JOIN 결과의 전체 행 수를 비교한다.
- 기준 키별 행 수를 계산해 반복되는 키를 확인한다.
- 개별 사례를 조회해 행 증가가 실제 관계로 설명되는지 확인한다.
- 행 수의 변화가 예상과 다르면 연결 키, 관계, JOIN 종류, 필터 위치를 다시 점검한다.

---

## 한 문장 정리

> 다단계 JOIN은 테이블 관계와 결과의 기준 단위를 먼저 설계하고, JOIN 종류와 필터 위치를 의도에 맞게 선택한 뒤 행 수와 키별 반복을 검증해야 안전하게 구성할 수 있다.
