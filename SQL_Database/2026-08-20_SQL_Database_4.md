# 2026-08-20 SQL 데이터 조회하기

## 학습 목표

- `SELECT`, `FROM`을 이용해 기본적인 SQL 조회문을 작성한다.
- `DISTINCT`로 중복된 값을 제거한다.
- `WHERE`로 원하는 조건의 데이터만 조회한다.
- `CASE WHEN`으로 조건에 따라 새로운 값을 만든다.
- 비교·논리·범위 연산자를 조합해 복합 조건을 작성한다.
- `ORDER BY`로 조회 결과를 원하는 순서로 정렬한다.

---

## 1. 실습 데이터와 데이터 구조

이번 강의에서는 **Kaggle의 Bike Store Sample Database**를 사용했다. 고객, 주문, 주문 상품, 상품, 직원, 재고 정보를 담고 있는 관계형 데이터베이스이며 다음 6개 테이블을 사용한다.

| 테이블 | 설명 |
|---|---|
| `customers` | 고객의 이름, 연락처, 주소 등의 정보 |
| `orders` | 고객의 주문일, 배송일, 담당 직원 등의 주문 정보 |
| `order_items` | 주문에 포함된 상품, 수량, 가격, 할인율 정보 |
| `products` | 상품명, 가격, 모델 연도 등의 상품 정보 |
| `staffs` | 주문을 담당하는 직원 정보 |
| `stocks` | 매장별 상품 재고 정보 |

주요 테이블 연결은 다음과 같다.

```text
customers → orders → order_items → products
```

추가적으로 주문 담당 직원과 재고 정보도 연결할 수 있다.

```text
staffs ── staff_id ── orders
products ── product_id ── stocks
```

여러 테이블의 데이터가 필요한 경우 모든 구조를 외우기보다는 **필요한 데이터가 어느 테이블에 있는지 확인하고 PK/FK를 따라 JOIN할 테이블을 찾는 것**이 중요하다.

---

## 2. SQL 기본 구조: SELECT, FROM

SQL에서 데이터를 조회할 때 가장 기본이 되는 문법은 `SELECT`와 `FROM`이다.

- `SELECT` : 조회할 열(Column)을 지정한다.
- `FROM` : 데이터를 가져올 테이블을 지정한다.

예를 들어 고객의 이름과 이메일만 조회하려면 다음과 같이 작성한다.

```sql
SELECT first_name,
       last_name,
       email
FROM customers;
```

모든 열을 조회하려면 `*`를 사용할 수 있다.

```sql
SELECT *
FROM customers;
```

하지만 실무에서는 필요한 열만 명시하는 것이 권장된다. 불필요한 데이터를 가져오면 성능이 저하될 수 있고, 테이블 구조가 변경되었을 때 예상하지 못한 열이 포함될 수 있으며 가독성과 유지보수성도 떨어질 수 있기 때문이다.

---

## 3. DISTINCT로 중복 제거

조회 결과에서 동일한 값이 여러 번 반복되는 경우 `DISTINCT`를 사용하면 중복을 제거하고 고유한 값만 확인할 수 있다.

```sql
SELECT DISTINCT store_id
FROM orders;
```

`DISTINCT`는 **데이터에 어떤 종류의 고유한 값들이 존재하는지 확인할 때** 유용하다.

예를 들어 고객의 지역 종류를 확인할 수도 있다.

```sql
SELECT DISTINCT state
FROM customers;
```

---

## 4. WHERE로 조건에 맞는 데이터 조회

특정 조건을 만족하는 행만 조회하려면 `WHERE`를 사용한다.

```sql
SELECT product_name,
       list_price
FROM products
WHERE list_price >= 2000;
```

`WHERE`는 `FROM` 뒤에 작성하며 조건식이 참인 행만 결과에 포함된다.

### 주요 비교 연산자

| 연산자 | 의미 |
|---|---|
| `>` | 초과 |
| `<` | 미만 |
| `>=` | 이상 |
| `<=` | 이하 |
| `=` | 같음 |
| `!=` | 같지 않음 |

---

## 5. 여러 조건 조합하기

`WHERE`에서는 `AND`, `OR`, `NOT` 등의 논리 연산자를 이용해 여러 조건을 조합할 수 있다.

### AND

두 조건을 모두 만족해야 한다.

```sql
SELECT *
FROM products
WHERE list_price >= 500
  AND list_price <= 1500;
```

### OR

두 조건 중 하나라도 만족하면 된다.

```sql
SELECT *
FROM products
WHERE model_year = 2017
   OR model_year = 2018;
```

조건의 우선순위를 명확하게 하고 싶다면 괄호를 사용하는 것이 좋다.

---

## 6. 주요 조건 연산자

### BETWEEN

특정 범위에 포함되는 데이터를 조회할 때 사용한다.

```sql
SELECT *
FROM products
WHERE list_price BETWEEN 500 AND 1500;
```

`BETWEEN 500 AND 1500`은 **500 이상 1500 이하**를 의미한다.

### IN

여러 후보 중 하나에 해당하는지 확인한다.

```sql
SELECT *
FROM products
WHERE model_year IN (2017, 2018);
```

### LIKE

문자열의 특정 패턴을 찾을 때 사용한다.

```sql
SELECT *
FROM customers
WHERE first_name LIKE '%Ann%';
```

`%`를 사용하면 해당 문자열 앞뒤에 다른 값이 있어도 검색할 수 있다.

### IS NULL / IS NOT NULL

NULL 여부를 확인한다.

```sql
SELECT *
FROM orders
WHERE shipped_date IS NULL;
```

반대로 값이 존재하는 행은 다음과 같이 조회한다.

```sql
SELECT *
FROM orders
WHERE shipped_date IS NOT NULL;
```

### NOT IN

특정 목록에 포함되지 않는 데이터를 찾을 수 있다.

```sql
SELECT *
FROM products
WHERE model_year NOT IN (2017, 2018);
```

---

## 7. 조건을 조합한 조회

다음 조건의 상품을 프로모션 대상으로 찾는다고 가정한다.

- 가격이 500 이상 1500 이하
- 모델 연도가 2017 또는 2018

```sql
SELECT product_name,
       model_year,
       list_price
FROM products
WHERE list_price BETWEEN 500 AND 1500
  AND model_year IN (2017, 2018);
```

여기서 `BETWEEN`은 가격 범위를, `IN`은 여러 후보 중 하나인지 확인하며 `AND`는 두 조건을 모두 만족하도록 만든다.

---

## 8. CASE WHEN

`CASE WHEN`은 조건에 따라 서로 다른 값을 반환하는 조건부 표현식이다. 프로그래밍의 `if-else`와 유사하며 `SELECT`절에서 새로운 열을 만들 때 주로 활용한다.

기본 구조:

```sql
SELECT
    CASE
        WHEN 조건식1 THEN 결과1
        WHEN 조건식2 THEN 결과2
        ELSE 결과3
    END AS 새로운열이름
FROM 테이블명;
```

### 상품 가격에 따른 등급 만들기

```sql
SELECT product_name,
       list_price,
       CASE
           WHEN list_price < 500 THEN '일반'
           WHEN list_price < 1500 THEN '프리미엄'
           ELSE '고급'
       END AS product_grade
FROM products;
```

`CASE WHEN`은 **위에서부터 조건을 확인하며 먼저 만족한 조건의 결과를 반환한다.** 모든 `WHEN` 조건이 만족되지 않으면 `ELSE`가 실행된다.

---

## 9. CASE WHEN 활용

숫자나 코드로 저장된 데이터를 사람이 이해하기 쉬운 형태로 변환할 때 `CASE WHEN`을 사용할 수 있다.

예를 들어 주문 상태 코드가 다음과 같다고 가정한다.

```text
1 → 처리중
2 → 거부
3 → 배송완료
4 → 취소
```

이를 상태명으로 변환할 수 있다.

```sql
SELECT order_id,
       order_status,
       CASE
           WHEN order_status = 1 THEN '처리중'
           WHEN order_status = 2 THEN '거부'
           WHEN order_status = 3 THEN '배송완료'
           WHEN order_status = 4 THEN '취소'
       END AS order_status_name
FROM orders;
```

직원의 `active` 값에 따라 근무 상태를 표시할 수도 있다.

```sql
SELECT *,
       CASE
           WHEN active = 1 THEN '재직'
           ELSE '비활성'
       END AS work_status
FROM staffs;
```

---

## 10. ORDER BY로 결과 정렬

`ORDER BY`는 조회 결과를 원하는 순서로 정렬할 때 사용한다.

```sql
SELECT *
FROM orders
ORDER BY order_date;
```

내림차순은 `DESC`, 오름차순은 `ASC`를 사용한다.

```sql
SELECT *
FROM orders
ORDER BY order_date DESC;
```

```sql
SELECT *
FROM orders
ORDER BY order_date ASC;
```

따라서 최근 주문부터 확인하려면 주문일을 기준으로 내림차순 정렬할 수 있다.

---

## 11. 데이터 타입과 NULL 확인

CSV 데이터를 가져오는 과정에서는 일부 `NULL` 값이 문자열 `'NULL'`로 저장되거나 날짜 및 숫자 데이터가 문자열 타입으로 저장될 수 있다.

따라서 데이터를 사용하기 전에 컬럼의 데이터 타입과 NULL 값을 확인하는 과정이 중요하다.

PostgreSQL의 `information_schema.columns`를 이용하면 테이블의 컬럼 정보를 확인할 수 있다.

```sql
SELECT table_name,
       column_name,
       data_type
FROM information_schema.columns
WHERE table_schema = 'public'
  AND table_name IN
      ('customers', 'orders', 'order_items',
       'products', 'staffs', 'stocks')
ORDER BY table_name,
         ordinal_position;
```

테이블의 데이터 건수는 `COUNT(*)`로 확인할 수 있다.

```sql
SELECT COUNT(*)
FROM customers;
```

---

## 12. 오늘 배운 SQL 조회 흐름

오늘 배운 내용을 하나의 흐름으로 정리하면 다음과 같다.

```text
SELECT
  ↓
어떤 열을 볼 것인가?

FROM
  ↓
어느 테이블에서 가져올 것인가?

DISTINCT
  ↓
중복을 제거할 것인가?

WHERE
  ↓
어떤 조건의 행만 가져올 것인가?

CASE WHEN
  ↓
조건에 따라 새로운 값을 만들 것인가?

ORDER BY
  ↓
결과를 어떤 순서로 보여줄 것인가?
```

---

## 13. 오늘의 핵심 코드

### 필요한 열만 조회

```sql
SELECT first_name,
       last_name,
       email
FROM customers;
```

### 중복 제거

```sql
SELECT DISTINCT store_id
FROM orders;
```

### 특정 조건 조회

```sql
SELECT product_name,
       list_price
FROM products
WHERE list_price >= 2000;
```

### 여러 조건 조합

```sql
SELECT product_name,
       model_year,
       list_price
FROM products
WHERE list_price BETWEEN 500 AND 1500
  AND model_year IN (2017, 2018);
```

### CASE WHEN으로 등급 생성

```sql
SELECT product_name,
       list_price,
       CASE
           WHEN list_price < 500 THEN '일반'
           WHEN list_price < 1500 THEN '프리미엄'
           ELSE '고급'
       END AS product_grade
FROM products;
```

### NULL 확인

```sql
SELECT *
FROM orders
WHERE shipped_date IS NULL;
```

### 결과 정렬

```sql
SELECT *
FROM orders
ORDER BY order_date DESC;
```

---

## 14. 오늘 배운 핵심 정리

오늘은 SQL을 이용해 **원하는 데이터를 골라서 조회하는 기본적인 방법**을 학습했다.

가장 먼저 `SELECT`와 `FROM`을 통해 필요한 열과 테이블을 지정하는 방법을 익혔다. `SELECT *`로 모든 열을 조회할 수도 있지만, 실무에서는 필요한 열만 명시하는 것이 성능과 가독성, 유지보수 측면에서 좋다는 점을 배웠다.

`DISTINCT`는 데이터의 중복을 제거하여 고유한 값만 확인할 때 사용한다. 데이터에 어떤 종류의 값이 존재하는지 파악할 때 유용했다.

`WHERE`는 조건을 만족하는 행만 조회하기 위한 핵심 문법이었다. 비교 연산자뿐만 아니라 `AND`, `OR`, `NOT`, `BETWEEN`, `IN`, `LIKE`, `IS NULL`, `IS NOT NULL` 등을 조합하여 다양한 조건을 만들 수 있었다.

특히 `BETWEEN`과 `IN`을 `AND`와 함께 사용하면 실제 업무에서 특정 가격대와 특정 연도에 해당하는 상품처럼 **여러 조건을 만족하는 데이터만 추려낼 수 있다.**

`CASE WHEN`은 조건에 따라 새로운 값을 만들어내는 기능이었다. 숫자로 된 주문 상태 코드나 직원의 상태처럼 사람이 이해하기 어려운 데이터를 의미 있는 이름이나 등급으로 변환할 때 유용했다. 또한 여러 `WHEN` 조건은 위에서부터 순서대로 확인되므로 조건의 작성 순서가 중요하다는 점을 기억해야 한다.

마지막으로 `ORDER BY`를 사용하면 조회 결과를 원하는 순서로 정렬할 수 있었다. 이를 활용하면 최근 주문부터 확인하는 등 데이터를 원하는 기준으로 정리해서 볼 수 있다.

결국 오늘 배운 핵심은

> **SELECT로 필요한 데이터를 선택하고 → FROM으로 테이블을 지정하고 → WHERE로 조건을 걸고 → CASE WHEN으로 필요한 값을 가공하고 → ORDER BY로 결과를 정렬하는 SQL 조회의 기본 흐름**

이라고 정리할 수 있다.