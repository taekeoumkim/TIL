# 2026-08-20 SQL 데이터 집계와 분석

## 학습 목표

- SQL 집계함수 `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`를 활용한다.
- `GROUP BY`를 이용해 특정 기준으로 데이터를 그룹화한다.
- `HAVING`과 `WHERE`의 차이를 이해하고 적절히 사용한다.
- 전체 데이터를 요약하고 기준별로 비교·분석하는 SQL을 작성한다.

---

## 1. 데이터 집계와 분석

단순히 데이터를 한 행씩 조회하는 것에서 더 나아가, 데이터의 전체적인 통계적 의미를 파악하려면 **집계와 분석**이 필요하다.

집계함수를 사용하면 많은 양의 데이터를 한 번에 요약하여 평균, 합계, 최댓값, 최솟값 등을 확인할 수 있다.

또한 `GROUP BY`를 사용하면 성별, 지역, 카테고리, 직원, 매장 등의 특정 기준으로 데이터를 묶어 그룹별로 비교할 수 있다.

이번 학습에서는 PostgreSQL을 사용하여 다음 내용을 학습했다.

```text
집계함수
  ↓
전체 데이터 요약

GROUP BY
  ↓
기준별 데이터 그룹화 및 집계

WHERE
  ↓
그룹화하기 전 개별 행 필터링

HAVING
  ↓
그룹화한 후 집계 결과 필터링
```

---

## 2. SQL 집계함수

집계함수(Aggregate Function)는 여러 행을 대상으로 계산을 수행하고 **하나의 결과값을 반환하는 함수**이다.

주요 집계함수는 다음과 같다.

| 함수 | 기능 |
|---|---|
| `COUNT` | 데이터 또는 행의 개수 |
| `SUM` | 값의 합계 |
| `AVG` | 값의 평균 |
| `MIN` | 최솟값 |
| `MAX` | 최댓값 |

---

## 3. COUNT

`COUNT`는 데이터의 개수를 계산한다.

전체 행의 개수를 확인할 때는 `COUNT(*)`를 사용한다.

```sql
SELECT COUNT(*) AS cnt
FROM orders;
```

### COUNT(*)와 COUNT(열명)의 차이

두 방식은 NULL 처리에서 차이가 있다.

```sql
COUNT(*)
```

는 NULL 여부와 관계없이 **전체 행의 수**를 센다.

반면,

```sql
COUNT(열명)
```

은 해당 열에서 **NULL이 아닌 값의 개수**만 센다.

예를 들어 7개의 행 중 날짜 값이 NULL인 행이 2개 있다면:

```text
COUNT(*)     → 7
COUNT(날짜)  → 5
```

따라서 전체 데이터 건수를 셀 때는 `COUNT(*)`, 특정 열의 유효한 데이터 개수를 셀 때는 `COUNT(열명)`을 사용하는 것이 중요하다.

---

## 4. SUM

`SUM`은 숫자 데이터의 합계를 계산한다.

예를 들어 전체 판매 수량을 구할 수 있다.

```sql
SELECT SUM(quantity) AS total_quantity
FROM order_items;
```

전체 재고 수량을 구하는 경우에도 사용할 수 있다.

```sql
SELECT SUM(quantity) AS total_stock
FROM stocks;
```

---

## 5. AVG

`AVG`는 숫자 데이터의 평균을 계산한다.

상품 가격의 평균을 확인하려면 다음과 같이 작성할 수 있다.

```sql
SELECT AVG(list_price) AS avg_price
FROM products;
```

---

## 6. MIN과 MAX

`MIN`은 최솟값, `MAX`는 최댓값을 반환한다.

```sql
SELECT MIN(list_price) AS min_price,
       MAX(list_price) AS max_price
FROM products;
```

이를 통해 상품 가격의 최소 수준과 최대 수준을 한 번에 확인할 수 있다.

---

## 7. 여러 집계함수 함께 사용하기

하나의 `SELECT`문에서 여러 집계함수를 동시에 사용할 수 있다.

예를 들어 주문 상품의 전체 건수와 가격의 평균·최대·최소·합계를 한 번에 확인할 수 있다.

```sql
SELECT COUNT(*) AS cnt,
       AVG(list_price) AS avg_price,
       MAX(list_price) AS max_price,
       MIN(list_price) AS min_price,
       SUM(list_price) AS sum_price
FROM order_items;
```

또한 주문 상품의 판매 수량을 종합적으로 확인할 수도 있다.

```sql
SELECT COUNT(*) AS item_count,
       SUM(quantity) AS total_quantity,
       AVG(quantity) AS avg_quantity,
       MAX(quantity) AS max_quantity,
       MIN(quantity) AS min_quantity
FROM order_items;
```

자료의 실행 결과 예시에서는 `order_items`에 대해 전체 행 4,722건, 총 수량 7,078개, 평균 수량 약 1.499개, 최대 2개, 최소 1개가 확인되었다.

---

## 8. GROUP BY

전체 데이터를 하나로 집계하는 것만으로는 그룹별 차이를 확인하기 어렵다.

이때 `GROUP BY`를 사용하면 특정 열을 기준으로 데이터를 묶고 **그룹별 통계**를 계산할 수 있다.

예를 들어 직원별 담당 주문 건수를 확인하려면:

```sql
SELECT staff_id,
       COUNT(*) AS order_count
FROM orders
GROUP BY staff_id;
```

실행 결과는 직원별로 하나의 행을 가지며 각 직원이 담당한 주문 건수를 확인할 수 있다.

예시:

```text
staff_id | order_count
---------+------------
2        | 164
3        | 184
6        | 553
7        | 540
...
```

즉,

```text
GROUP BY staff_id
        ↓
같은 직원의 주문끼리 그룹화
        ↓
COUNT(*)
        ↓
각 그룹의 주문 건수 계산
```

---

## 9. GROUP BY의 핵심 규칙

`GROUP BY`를 사용할 때 중요한 규칙이 있다.

**SELECT절에 집계함수가 아닌 일반 열을 표시한다면 해당 열은 GROUP BY에도 포함해야 한다.**

예를 들어 다음과 같이 기준 열과 집계함수를 함께 사용할 수 있다.

```sql
SELECT staff_id,
       COUNT(*) AS order_count
FROM orders
GROUP BY staff_id;
```

`staff_id`는 집계함수가 아닌 일반 열이므로 `GROUP BY staff_id`가 필요하다.

그룹화 없이 다음처럼 작성하면 문제가 발생한다.

```sql
SELECT staff_id,
       COUNT(*)
FROM orders;
```

이유는 `COUNT(*)`는 여러 행을 하나의 결과로 줄이지만 `staff_id`는 여러 값을 그대로 반환하려 하기 때문에 결과의 행 수가 맞지 않기 때문이다.

따라서 `GROUP BY`를 이용해 `staff_id`별로 하나의 그룹을 만든 후 각 그룹의 `COUNT(*)`를 계산해야 한다.

---

## 10. 다양한 GROUP BY 활용

### 매장별 주문 건수

```sql
SELECT store_id,
       COUNT(*) AS order_count
FROM orders
GROUP BY store_id;
```

### 브랜드별 평균 상품 가격

```sql
SELECT brand_id,
       AVG(list_price) AS avg_price
FROM products
GROUP BY brand_id;
```

### 카테고리별 상품 개수

```sql
SELECT category_id,
       COUNT(*) AS product_count
FROM products
GROUP BY category_id;
```

### 직원별 담당 주문 건수

```sql
SELECT staff_id,
       COUNT(*) AS order_count
FROM orders
GROUP BY staff_id;
```

### 매장별 전체 재고 수량

```sql
SELECT store_id,
       SUM(quantity) AS total_stock
FROM stocks
GROUP BY store_id;
```

이처럼 `GROUP BY`와 집계함수를 함께 사용하면 단순한 전체 통계가 아니라 **기준별 비교 분석**이 가능해진다.

---

## 11. GROUP BY의 의미

`GROUP BY`는 단순히 데이터를 정렬하는 기능이 아니다.

```text
전체 데이터
    ↓
특정 기준으로 그룹 생성
    ↓
각 그룹에 집계함수 적용
    ↓
그룹별 통계 결과 반환
```

예를 들어 `store_id`를 기준으로 그룹화한다면:

```text
store_id = 1인 데이터 → 하나의 그룹
store_id = 2인 데이터 → 하나의 그룹
store_id = 3인 데이터 → 하나의 그룹
...
```

그리고 각 그룹에 `SUM(quantity)`를 적용하면 매장별 전체 재고 수량을 확인할 수 있다.

---

## 12. SQL의 논리적 처리 순서

이번 학습에서 특히 중요한 부분은 SQL이 논리적으로 처리되는 순서이다.

자료에서는 다음 순서를 제시한다.

```text
FROM
  ↓
WHERE
  ↓
GROUP BY
  ↓
HAVING
  ↓
SELECT
  ↓
ORDER BY
```

이 순서를 이해하면 `WHERE`와 `HAVING`의 차이를 이해하기 쉬워진다.

---

## 13. WHERE

`WHERE`는 **GROUP BY 전에 개별 행을 필터링**한다.

예를 들어 재고가 0보다 큰 데이터만 대상으로 집계하려면:

```sql
SELECT store_id,
       SUM(quantity) AS total_stock
FROM stocks
WHERE quantity > 0
GROUP BY store_id;
```

여기서는 먼저 `quantity > 0`인 개별 행만 남긴 후 `store_id`별로 그룹화한다.

```text
stocks의 개별 행
      ↓
WHERE quantity > 0
      ↓
재고가 있는 행만 남음
      ↓
GROUP BY store_id
      ↓
매장별 그룹 생성
```

---

## 14. HAVING

`HAVING`은 `GROUP BY` 이후 만들어진 **집계 결과를 필터링**한다.

예를 들어 재고가 있는 데이터만 대상으로 매장별 재고를 합산한 후, 총 재고가 100개 이상인 매장만 찾을 수 있다.

```sql
SELECT store_id,
       SUM(quantity) AS total_stock
FROM stocks
WHERE quantity > 0
GROUP BY store_id
HAVING SUM(quantity) >= 100;
```

처리 과정은 다음과 같다.

```text
1. WHERE quantity > 0
   ↓
   개별 행에서 재고가 있는 데이터만 선택

2. GROUP BY store_id
   ↓
   매장별로 그룹화

3. SUM(quantity)
   ↓
   매장별 전체 재고 계산

4. HAVING SUM(quantity) >= 100
   ↓
   집계 결과가 100 이상인 매장만 선택
```

---

## 15. WHERE와 HAVING의 차이

두 문법의 가장 중요한 차이는 **어떤 단계의 데이터를 필터링하느냐**이다.

| 구분 | WHERE | HAVING |
|---|---|---|
| 적용 시점 | GROUP BY 전 | GROUP BY 후 |
| 대상 | 개별 행 | 그룹/집계 결과 |
| 용도 | 원본 데이터 필터링 | 집계 결과 필터링 |
| 예시 | `quantity > 0` | `SUM(quantity) >= 100` |

쉽게 정리하면:

```text
WHERE  → 개별 데이터에 조건
HAVING → 그룹화한 결과에 조건
```

따라서 집계함수의 결과에 조건을 걸어야 한다면 `HAVING`을 사용한다.

---

## 16. WHERE와 HAVING을 함께 사용하기

실무에서는 `WHERE`와 `HAVING`을 함께 사용하는 경우가 많다.

```sql
SELECT store_id,
       SUM(quantity) AS total_stock
FROM stocks
WHERE quantity > 0
GROUP BY store_id
HAVING SUM(quantity) >= 100;
```

이 쿼리는 두 단계의 조건을 적용한다.

```text
WHERE
→ 재고가 있는 개별 데이터만 대상으로 함

GROUP BY
→ 매장별로 그룹화

SUM
→ 매장별 재고 합계 계산

HAVING
→ 합계가 100 이상인 매장만 선택
```

즉, **행을 먼저 좁힌 다음 그룹별 집계를 수행하고, 마지막으로 집계 결과를 다시 필터링**하는 방식이다.

---

## 17. 오늘의 핵심 SQL

### 전체 주문 건수

```sql
SELECT COUNT(*) AS order_count
FROM orders;
```

### 상품 가격의 평균·최대·최소

```sql
SELECT AVG(list_price) AS avg_price,
       MAX(list_price) AS max_price,
       MIN(list_price) AS min_price
FROM products;
```

### 전체 판매 수량

```sql
SELECT SUM(quantity) AS total_quantity
FROM order_items;
```

### 매장별 주문 건수

```sql
SELECT store_id,
       COUNT(*) AS order_count
FROM orders
GROUP BY store_id;
```

### 브랜드별 평균 상품 가격

```sql
SELECT brand_id,
       AVG(list_price) AS avg_price
FROM products
GROUP BY brand_id;
```

### 카테고리별 상품 개수

```sql
SELECT category_id,
       COUNT(*) AS product_count
FROM products
GROUP BY category_id;
```

### 매장별 재고 합계

```sql
SELECT store_id,
       SUM(quantity) AS total_stock
FROM stocks
GROUP BY store_id;
```

### WHERE + GROUP BY + HAVING

```sql
SELECT store_id,
       SUM(quantity) AS total_stock
FROM stocks
WHERE quantity > 0
GROUP BY store_id
HAVING SUM(quantity) >= 100;
```

---

## 18. 오늘 배운 핵심 정리

오늘은 단순한 데이터 조회를 넘어 **데이터를 요약하고 기준별로 비교하는 방법**을 학습했다.

먼저 `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`와 같은 집계함수를 사용하면 여러 행의 데이터를 하나의 통계값으로 요약할 수 있다는 것을 배웠다.

`COUNT(*)`는 전체 행의 개수를 계산하고, `COUNT(열명)`은 해당 열에서 NULL이 아닌 데이터의 개수를 계산한다는 차이도 중요했다.

그다음 `GROUP BY`를 이용하면 전체 데이터를 하나로 집계하는 것이 아니라 직원, 매장, 브랜드, 카테고리 등 특정 기준으로 데이터를 묶어서 그룹별 통계를 계산할 수 있었다.

특히 `GROUP BY`를 사용할 때는 **SELECT절에 있는 일반 열을 GROUP BY에도 포함해야 한다**는 규칙을 기억해야 한다.

또한 `WHERE`와 `HAVING`의 차이도 중요한 내용이었다.

> **WHERE는 그룹화하기 전 개별 행을 필터링하고, HAVING은 그룹화한 후 집계 결과를 필터링한다.**

따라서 단순한 행 조건은 `WHERE`, `SUM`, `COUNT`, `AVG` 등의 집계 결과에 조건을 적용해야 하는 경우에는 `HAVING`을 사용한다.

결국 오늘 배운 핵심 흐름은 다음과 같이 정리할 수 있다.

```text
전체 데이터
   ↓
WHERE로 필요한 행 필터링
   ↓
GROUP BY로 기준별 그룹화
   ↓
COUNT / SUM / AVG / MIN / MAX로 집계
   ↓
HAVING으로 집계 결과 필터링
   ↓
그룹별 데이터 분석
```

이번 학습을 통해 SQL은 단순히 원하는 행을 조회하는 도구가 아니라, **많은 데이터를 요약하고 그룹별 차이를 비교하여 데이터의 의미를 파악하는 분석 도구**로도 활용할 수 있다는 것을 이해했다.