# 2026-08-20 SQL 테이블 결합

## 학습 목표

- SQL에서 `UNION`을 사용하여 테이블을 행 방향으로 결합할 수 있다.
- SQL에서 `JOIN`을 사용하여 테이블을 열 방향으로 결합할 수 있다.
- `UNION`과 `JOIN`의 차이를 이해하고 상황에 맞게 사용할 수 있다.
- `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL OUTER JOIN`의 결과 차이를 설명할 수 있다.
- `JOIN`으로 여러 테이블의 데이터를 조건에 맞게 조회할 수 있다.

---

## 1. 테이블 결합이 필요한 이유

데이터베이스에서는 정보를 효율적으로 관리하기 위해 여러 테이블로 나누어 저장한다.

예를 들어 주문 정보는 `orders` 테이블에, 고객 정보는 `customers` 테이블에 따로 저장할 수 있다. 하지만 실제 분석에서는 **"어느 고객이 얼마를 주문했는가?"**처럼 여러 테이블의 데이터를 함께 확인해야 하는 경우가 많다.

이때 사용하는 대표적인 방법이 `UNION`과 `JOIN`이다.

- `UNION` → 여러 조회 결과를 **행 방향으로 쌓는다.**
- `JOIN` → 관련된 테이블을 **열 방향으로 연결한다.**

---

## 2. UNION: 행 방향으로 테이블 합치기

### UNION이란?

`UNION`은 두 개 이상의 `SELECT` 결과를 **세로(행) 방향으로 이어 붙이는** 방법이다.

마치 명단표 두 개를 위아래로 붙이는 것처럼 결과를 합친다.

```sql
SELECT order_id, customer_id, order_date
FROM orders
WHERE order_date < '2017-01-01'

UNION

SELECT order_id, customer_id, order_date
FROM orders
WHERE order_date >= '2017-01-01';
```

서로 다른 테이블뿐만 아니라 **같은 테이블에서 서로 다른 조건으로 조회한 결과**도 `UNION`으로 합칠 수 있다.

### UNION 사용 조건

`UNION`을 사용하려면 다음 조건이 필요하다.

- 각 `SELECT` 문의 **열 개수가 같아야 한다.**
- 대응되는 열의 **데이터 타입이 서로 호환되어야 한다.**
- 열 이름이 같을 필요는 없다.
- 결합 기준은 열 이름이 아니라 **열의 위치와 데이터 타입**이다.
- 최종 결과의 열 이름은 **첫 번째 `SELECT`의 열 이름**을 따른다.

---

## 3. UNION vs UNION ALL

두 연산자의 가장 큰 차이는 **중복 행을 처리하는 방식**이다.

| 구분 | 특징 |
|---|---|
| `UNION` | 중복 행을 제거 |
| `UNION ALL` | 중복을 제거하지 않고 모든 행을 유지 |

### UNION

```sql
SELECT city
FROM customers
WHERE state = 'NY'

UNION

SELECT city
FROM customers
WHERE state = 'CA';
```

`UNION`은 중복 행을 하나만 남긴다.

중복을 제거하기 위해 정렬 또는 해시 기반 연산이 필요할 수 있으므로 추가 비용이 발생할 수 있다.

### UNION ALL

```sql
SELECT city
FROM customers
WHERE state = 'NY'

UNION ALL

SELECT city
FROM customers
WHERE state = 'CA';
```

`UNION ALL`은 중복 여부와 관계없이 모든 행을 포함한다.

따라서 중복을 유지해야 하는 경우 사용할 수 있고, 중복 제거 작업이 없기 때문에 일반적으로 `UNION`보다 빠르다.

### 기억할 점

> **UNION = 중복 제거**  
> **UNION ALL = 모든 행 유지**

---

## 4. JOIN: 열 방향으로 테이블 합치기

### JOIN이란?

`JOIN`은 두 테이블을 **가로(열) 방향으로 연결**하는 방법이다.

공통 키를 기준으로 두 테이블의 행을 연결하여 하나의 넓은 테이블처럼 조회할 수 있다.

예를 들어 `orders`에는 담당 직원의 `staff_id`가 있고 직원의 실제 이름은 `staffs` 테이블에 저장되어 있다면, `staff_id`를 기준으로 두 테이블을 JOIN하여 주문 정보와 직원 이름을 함께 조회할 수 있다.

```sql
SELECT
    o.order_id,
    o.order_date,
    s.first_name,
    s.last_name
FROM orders o
JOIN staffs s
    ON o.staff_id = s.staff_id;
```

### 기본 문법

```sql
SELECT 테이블1.열, 테이블2.열
FROM 테이블1
JOIN 테이블2
    ON 테이블1.공통키 = 테이블2.공통키;
```

`JOIN`에서는 **ON 절**에 두 테이블을 연결할 조건을 작성한다.

---

## 5. JOIN할 열을 찾는 방법: PK와 FK

JOIN을 작성할 때 어떤 열끼리 연결해야 할지 모르겠다면 **PK(Primary Key)와 FK(Foreign Key)의 관계**를 먼저 확인하는 것이 좋다.

예를 들어:

```text
customers.customer_id ← PK
        ↑
orders.customer_id ← FK
```

이 관계라면 다음과 같이 연결할 수 있다.

```sql
SELECT
    o.order_id,
    o.order_date,
    c.first_name,
    c.last_name
FROM orders o
JOIN customers c
    ON o.customer_id = c.customer_id;
```

일반적으로 한 테이블의 **PK와 이를 참조하는 다른 테이블의 FK**를 연결한다.

### 별칭(alias)

여러 테이블을 사용할 때 별칭을 사용하면 SQL을 간결하게 작성할 수 있다.

```sql
FROM orders o
JOIN customers c
    ON o.customer_id = c.customer_id
```

여기서 `o`는 `orders`, `c`는 `customers`를 의미한다.

### ON 절에 여러 조건 사용하기

`ON` 절에서는 `AND`를 사용하여 여러 조건을 연결할 수도 있다.

```sql
FROM orders o
JOIN deliveries d
    ON o.order_id = d.order_id
    AND o.store_id = d.store_id
```

---

## 6. JOIN의 종류

JOIN은 **어느 테이블의 행을 결과에 남길 것인지**에 따라 종류가 달라진다.

| JOIN 종류 | 결과 |
|---|---|
| `INNER JOIN` | 양쪽 테이블에서 조건이 일치하는 행만 |
| `LEFT JOIN` | 왼쪽 테이블 전체 + 오른쪽에서 일치하는 행 |
| `RIGHT JOIN` | 오른쪽 테이블 전체 + 왼쪽에서 일치하는 행 |
| `FULL OUTER JOIN` | 양쪽 테이블의 모든 행 |

JOIN 종류를 선택할 때는 단순히 문법을 외우기보다,

> **"어느 테이블의 행을 반드시 남겨야 하는가?"**

를 먼저 생각하는 것이 중요하다.

---

## 7. INNER JOIN

### 개념

`INNER JOIN`은 두 테이블에서 `ON` 조건이 **일치하는 행만 결과에 포함**한다.

어느 한쪽 테이블에만 존재하는 행은 결과에서 제외된다.

```sql
SELECT
    o.order_id,
    c.customer_name,
    o.amount
FROM orders o
INNER JOIN customers c
    ON o.customer_id = c.customer_id;
```

`JOIN`이라고만 작성해도 기본적으로 `INNER JOIN`과 같은 의미로 사용할 수 있다.

```sql
FROM orders o
JOIN customers c
    ON o.customer_id = c.customer_id
```

### 사용 상황

예를 들어 **주문한 고객의 정보만 조회하고 주문하지 않은 고객은 제외**하고 싶을 때 사용할 수 있다.

> **INNER JOIN = 교집합**

---

## 8. LEFT JOIN

### 개념

`LEFT JOIN`은 `FROM` 뒤에 있는 **왼쪽 테이블의 모든 행을 결과에 포함**한다.

오른쪽 테이블에 일치하는 데이터가 없다면 오른쪽 테이블의 열은 `NULL`로 표시된다.

```sql
SELECT
    c.first_name,
    c.last_name,
    o.order_id
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id;
```

이 쿼리는 **주문을 하지 않은 고객까지 포함한 전체 고객 목록**을 확인할 때 유용하다.

### 예시

상품 전체를 기준으로 재고 등록 여부를 확인한다면:

```sql
SELECT
    p.product_id,
    p.product_name,
    s.store_id,
    s.quantity
FROM products p
LEFT JOIN stocks s
    ON p.product_id = s.product_id;
```

재고 정보가 없는 상품은 `store_id`, `quantity`가 `NULL`로 표시된다.

> **LEFT JOIN = 왼쪽 테이블을 반드시 유지**

---

## 9. RIGHT JOIN

### 개념

`RIGHT JOIN`은 `JOIN` 뒤에 있는 **오른쪽 테이블의 모든 행을 결과에 포함**한다.

```sql
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    o.order_id,
    o.order_date
FROM customers c
RIGHT JOIN orders o
    ON c.customer_id = o.customer_id;
```

왼쪽 테이블에 연결되는 데이터가 없다면 왼쪽 테이블의 열이 `NULL`로 표시된다.

`LEFT JOIN`과 방향이 반대라고 이해하면 된다.

다만 학습 자료에서는 실무에서 **테이블 순서를 바꾸어 LEFT JOIN을 사용하는 경우가 더 많다**고 설명한다.

> **RIGHT JOIN = 오른쪽 테이블을 반드시 유지**

---

## 10. FULL OUTER JOIN

### 개념

`FULL OUTER JOIN`은 두 테이블의 **모든 행을 포함**한다.

한쪽에만 존재하는 데이터도 결과에 포함되며, 반대쪽에 연결되는 데이터가 없다면 해당 열은 `NULL`로 표시된다.

PostgreSQL에서는 다음 두 표현을 사용할 수 있다.

```sql
FULL OUTER JOIN
```

또는

```sql
FULL JOIN
```

예시:

```sql
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    o.order_id,
    o.order_date
FROM customers c
FULL OUTER JOIN orders o
    ON c.customer_id = o.customer_id;
```

이를 이용하면 고객 정보는 있지만 주문 이력이 없는 고객과, 고객 정보와 연결되지 않는 주문을 모두 확인할 수 있다.

> **FULL OUTER JOIN = 양쪽 모두 유지**

---

## 11. JOIN 종류 한눈에 정리

```text
INNER JOIN
→ 양쪽에서 일치하는 데이터만

LEFT JOIN
→ 왼쪽 전체 + 오른쪽 일치 데이터

RIGHT JOIN
→ 오른쪽 전체 + 왼쪽 일치 데이터

FULL OUTER JOIN
→ 양쪽 전체
```

JOIN을 선택할 때는 결과에서 **반드시 남겨야 하는 기준 테이블**을 먼저 정하면 이해하기 쉽다.

---

## 12. UNION과 JOIN 비교

두 방법 모두 여러 데이터를 하나의 결과로 만들지만, 방향과 목적이 다르다.

| 구분 | UNION | JOIN |
|---|---|---|
| 결합 방향 | 행 방향 | 열 방향 |
| 목적 | 데이터를 아래로 쌓기 | 관련 정보를 옆으로 붙이기 |
| 기준 | 열 개수와 데이터 타입 | 연결 조건 |
| 핵심 문법 | `UNION`, `UNION ALL` | `JOIN ... ON` |
| 대표 사용 상황 | 여러 조회 결과를 하나로 합치기 | 서로 다른 테이블의 관련 정보 조회 |

가장 쉽게 구분하는 방법은 다음 질문이다.

> **"데이터를 쌓을까, 정보를 붙일까?"**

- 데이터를 **쌓는다** → `UNION`
- 정보를 **붙인다** → `JOIN`

---

## 13. 오늘 배운 SQL 활용 예시

### 주문 상품과 상품명 연결

`order_items`에 있는 `product_id`를 `products`의 `product_id`와 연결하면 주문 상품의 실제 상품명을 함께 조회할 수 있다.

```sql
SELECT
    oi.order_id,
    oi.product_id,
    p.product_name,
    oi.quantity
FROM order_items oi
INNER JOIN products p
    ON oi.product_id = p.product_id;
```

### 주문과 고객 정보 연결

```sql
SELECT
    o.order_id,
    o.order_date,
    c.first_name,
    c.last_name
FROM orders o
INNER JOIN customers c
    ON o.customer_id = c.customer_id;
```

### 상품과 재고 정보 연결

```sql
SELECT
    p.product_id,
    p.product_name,
    s.product_id,
    s.quantity
FROM products p
JOIN stocks s
    ON p.product_id = s.product_id;
```

### 전체 상품의 재고 현황 확인

재고가 없는 상품도 포함하려면 `LEFT JOIN`을 사용한다.

```sql
SELECT
    p.product_id,
    p.product_name,
    s.store_id,
    s.quantity
FROM products p
LEFT JOIN stocks s
    ON p.product_id = s.product_id;
```

---

## 14. 오늘의 핵심 코드

### UNION ALL

```sql
SELECT store_id, product_id, quantity
FROM stocks
WHERE store_id = 1

UNION ALL

SELECT store_id, product_id, quantity
FROM stocks
WHERE store_id = 2;
```

### JOIN

```sql
SELECT
    o.order_id,
    o.order_date,
    s.first_name,
    s.last_name
FROM orders o
JOIN staffs s
    ON o.staff_id = s.staff_id;
```

### INNER JOIN

```sql
SELECT
    o.order_id,
    c.first_name,
    c.last_name,
    o.order_date
FROM orders o
INNER JOIN customers c
    ON o.customer_id = c.customer_id;
```

### LEFT JOIN

```sql
SELECT
    c.first_name,
    c.last_name,
    o.order_id
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id;
```

### RIGHT JOIN

```sql
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    o.order_id,
    o.order_date
FROM customers c
RIGHT JOIN orders o
    ON c.customer_id = o.customer_id;
```

### FULL OUTER JOIN

```sql
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    o.order_id,
    o.order_date
FROM customers c
FULL OUTER JOIN orders o
    ON c.customer_id = o.customer_id;
```

---

## 15. 오늘 배운 내용 정리

### UNION

- `UNION`과 `UNION ALL`은 여러 `SELECT` 결과를 **행 방향으로 결합**한다.
- 서로 다른 테이블뿐만 아니라 같은 테이블의 서로 다른 조회 결과도 결합할 수 있다.
- 결합하는 `SELECT`의 **열 개수가 같아야 한다.**
- 대응되는 열의 데이터 타입도 서로 호환되어야 한다.
- 열 이름이 달라도 되며 **열의 위치와 데이터 타입**을 기준으로 결합한다.
- 결과 열 이름은 첫 번째 `SELECT`의 열 이름을 따른다.
- `UNION`은 중복 행을 제거한다.
- `UNION ALL`은 중복을 제거하지 않고 모든 행을 유지한다.

### JOIN

- `JOIN`은 관련된 테이블을 **열 방향으로 연결**한다.
- `ON` 절에 연결 조건을 작성한다.
- 일반적으로 PK와 이를 참조하는 FK를 연결한다.
- 별칭(alias)을 사용하면 여러 테이블을 사용하는 SQL을 간결하게 작성할 수 있다.

### JOIN 종류

- `INNER JOIN` → 일치하는 행만
- `LEFT JOIN` → 왼쪽 테이블 전체
- `RIGHT JOIN` → 오른쪽 테이블 전체
- `FULL OUTER JOIN` → 양쪽 테이블 전체
- 연결되는 데이터가 없는 경우 유지되는 쪽과 반대쪽의 값은 `NULL`로 표시될 수 있다.

---

## 16. 느낀 점

오늘은 SQL에서 여러 테이블의 데이터를 하나의 결과로 만드는 **테이블 결합**을 배웠다.

특히 `UNION`과 `JOIN`이 모두 테이블을 결합하는 기능이지만, 목적이 완전히 다르다는 점을 이해하는 것이 중요했다. `UNION`은 여러 조회 결과를 행 방향으로 이어 붙이는 방식이고, `JOIN`은 공통 키를 기준으로 서로 다른 테이블의 정보를 열 방향으로 붙이는 방식이었다.

처음에는 `UNION`과 `JOIN`을 단순히 문법으로 구분하려고 했지만, **"데이터를 쌓을까, 정보를 붙일까?"**라는 기준으로 생각하면 훨씬 쉽게 구분할 수 있을 것 같다.

또한 `JOIN`에서는 단순히 `ON` 조건을 작성하는 것보다 먼저 PK와 FK의 관계를 확인하는 것이 중요하다는 것을 배웠다. 실제 데이터베이스에서는 하나의 테이블에 모든 정보를 저장하는 것이 아니라 고객, 주문, 상품, 직원, 재고 등의 정보가 여러 테이블로 나뉘어 있기 때문에 JOIN은 데이터를 분석할 때 매우 자주 사용될 것 같다.

특히 `LEFT JOIN`을 통해 **기준이 되는 테이블의 모든 데이터를 유지하면서 연결되지 않은 데이터는 `NULL`로 확인할 수 있다는 점**이 인상적이었다. 따라서 JOIN 종류를 외우기보다 "어느 테이블의 행을 반드시 남겨야 하는가?"를 먼저 생각하는 습관을 들여야겠다고 느꼈다.

앞으로 SQL을 연습할 때 `UNION`, `UNION ALL`, `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL OUTER JOIN`을 다양한 테이블 관계에 적용해 보면서 각각의 결과가 어떻게 달라지는지 직접 확인해 봐야겠다.

결국 오늘 배운 내용은 단순히 테이블을 합치는 문법을 익힌 것이 아니라,

> **필요한 데이터를 어떤 방식으로 결합해야 하는지 판단하는 방법**

을 배웠다는 점에서 의미가 있었다.