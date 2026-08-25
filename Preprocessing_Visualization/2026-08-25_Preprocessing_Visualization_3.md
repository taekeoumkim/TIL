# \[2026-08-25\] Pandas 데이터 집계 - groupby, agg, pivot_table

## 학습 목표

-   `groupby()`의 **분할(Split) - 적용(Apply) - 결합(Combine)** 과정을
    이해한다.
-   그룹별 합계, 평균, 개수 등의 집계 결과를 만들 수 있다.
-   `agg()`를 사용하여 여러 집계 함수를 한 번에 적용할 수 있다.
-   `pivot_table()`을 사용하여 집계 결과를 행과 열 기준의 표 형태로
    재구성할 수 있다.
-   `reset_index()`가 필요한 이유를 이해하고 활용할 수 있다.

------------------------------------------------------------------------

## 1. 오늘의 코딩 테스트: 정렬(Sorting)

본격적인 데이터 집계에 들어가기 전에 정렬의 기본 개념을 학습했다.

정렬은 **데이터를 특정 기준에 따라 순서대로 배치하는 방법**이다.

Python에서는 `sorted()`를 이용해 리스트를 정렬할 수 있다.

``` python
numbers = [5, 2, 8, 1, 4]

ascending = sorted(numbers)
descending = sorted(numbers, reverse=True)

print(ascending)
print(descending)
```

결과:

``` text
[1, 2, 4, 5, 8]
[8, 5, 4, 2, 1]
```

### 핵심

``` python
sorted(numbers)
```

-   기본값은 **오름차순**

``` python
sorted(numbers, reverse=True)
```

-   `reverse=True`를 사용하면 **내림차순**

문자열도 같은 방법으로 정렬할 수 있다.

``` python
names = ["Charlie", "Alice", "Bob"]

print(sorted(names))
```

결과:

``` text
['Alice', 'Bob', 'Charlie']
```

### 코딩 테스트 예제: 배송 긴급도 정렬

배송 요청의 긴급도 점수가 높은 순서대로 처리해야 한다면 리스트를
내림차순으로 정렬하면 된다.

``` python
def solution(priorities):
    return sorted(priorities, reverse=True)
```

예:

``` python
priorities = [3, 1, 5, 2, 4]
```

결과:

``` text
[5, 4, 3, 2, 1]
```

------------------------------------------------------------------------

# 2. groupby(): 데이터를 그룹별로 집계하기

데이터 분석에서는 다음과 같은 질문을 자주 하게 된다.

``` text
지역별 매출은 얼마인가?
상품 카테고리별 평균 매출은 얼마인가?
결제수단별 판매 수량은 얼마인가?
회원 등급별 평균 구매금액은 얼마인가?
```

이처럼 **특정 기준별로 데이터를 묶은 뒤 계산**해야 할 때 Pandas의
`groupby()`를 사용할 수 있다.

## 기본 구조

``` python
df.groupby("그룹_기준")["집계할_컬럼"].집계함수()
```

예를 들어 지역별 주문 금액의 합계를 구한다.

``` python
import pandas as pd

df = pd.DataFrame({
    "region": ["서울", "서울", "부산", "부산", "대전"],
    "sales": [12000, 18000, 10000, 15000, 8000]
})

total_sales = (
    df.groupby("region")["sales"]
      .sum()
      .reset_index()
)

print(total_sales)
```

결과:

``` text
  region  sales
0   대전    8000
1   부산   25000
2   서울   30000
```

### 코드 해석

``` python
df.groupby("region")
```

`region`을 기준으로 같은 지역의 데이터를 하나의 그룹으로 묶는다.

``` text
서울 → 12000, 18000
부산 → 10000, 15000
대전 → 8000
```

그 다음:

``` python
["sales"].sum()
```

각 그룹의 `sales`를 합산한다.

------------------------------------------------------------------------

# 3. groupby()의 동작 원리: Split - Apply - Combine

`groupby()`는 크게 세 단계로 동작한다.

``` text
원본 데이터
    ↓
1. Split
그룹별로 데이터 분할
    ↓
2. Apply
각 그룹에 함수 적용
    ↓
3. Combine
계산 결과를 하나로 결합
```

## 1) Split

지정한 컬럼의 값을 기준으로 데이터를 그룹으로 나눈다.

``` python
df.groupby("region")
```

## 2) Apply

각 그룹에 원하는 집계 함수를 적용한다.

``` python
sum()
mean()
count()
max()
min()
```

## 3) Combine

각 그룹에서 계산한 결과를 하나의 결과로 합친다.

즉,

``` python
df.groupby("region")["sales"].sum()
```

은 다음 의미라고 이해할 수 있다.

> `region`별로 데이터를 나눈 뒤 → 각 지역의 `sales`를 더하고 → 결과를
> 하나로 합친다.

------------------------------------------------------------------------

# 4. 자주 사용하는 집계 함수

`groupby()` 뒤에는 다양한 집계 함수를 사용할 수 있다.

  함수        의미
  ----------- --------
  `sum()`     합계
  `mean()`    평균
  `count()`   개수
  `max()`     최댓값
  `min()`     최솟값

예:

``` python
df.groupby("product_category")["revenue"].sum()
```

상품 카테고리별 매출 합계

``` python
df.groupby("product_category")["revenue"].mean()
```

상품 카테고리별 평균 매출

``` python
df.groupby("product_category")["order_id"].count()
```

상품 카테고리별 주문 건수

------------------------------------------------------------------------

# 5. reset_index()는 왜 사용할까?

`groupby()`를 사용하면 그룹 기준으로 사용한 컬럼이 일반 컬럼이 아니라
**index**가 된다.

예:

``` python
df.groupby("region")["sales"].sum()
```

이후 결과를 일반적인 DataFrame처럼 다루기 쉽게 만들기 위해
`reset_index()`를 사용할 수 있다.

``` python
df.groupby("region")["sales"].sum().reset_index()
```

이렇게 하면 `region`이 다시 일반 컬럼으로 돌아온다.

따라서 학습자료에서는 이후 활용의 편의를 위해 `groupby()` 결과에
`reset_index()`를 붙이는 습관을 권장했다.

------------------------------------------------------------------------

# 6. groupby() 실습에서 배운 활용 패턴

## 상품 카테고리별 매출 분석

``` python
df.groupby("product_category")["revenue"].sum().reset_index()
```

``` python
df.groupby("product_category")["revenue"].mean().reset_index()
```

``` python
df.groupby("product_category")["order_id"].count().reset_index()
```

이를 통해 카테고리별로

-   총매출
-   평균 매출
-   주문 건수

를 확인할 수 있다.

------------------------------------------------------------------------

## 결제수단별 주문 수량 분석

``` python
df.groupby("payment_method")["quantity"].sum().reset_index()
```

결제수단별 전체 판매 수량을 확인할 수 있다.

``` python
df.groupby("payment_method")["quantity"].mean().reset_index()
```

결제수단별 평균 주문 수량을 확인할 수 있다.

------------------------------------------------------------------------

## 회원 등급별 구매금액 분석

고객 정보와 주문 요약 데이터가 서로 다른 파일에 있다면 먼저
`customer_id`를 기준으로 병합한다.

``` python
merged_df = pd.merge(
    customer_master,
    customer_order_summary,
    on="customer_id"
)
```

그 후 회원 등급을 기준으로 그룹화한다.

``` python
merged_df.groupby("membership_tier")["total_spent"].sum().reset_index()
```

``` python
merged_df.groupby("membership_tier")["total_spent"].mean().reset_index()
```

이 과정에서 이전에 배운 `merge()`와 오늘 배운 `groupby()`를 함께 사용할
수 있다는 점이 중요했다.

------------------------------------------------------------------------

# 7. agg(): 여러 집계 결과를 한 번에 계산하기

`groupby()`를 이용하면 원하는 집계를 할 수 있지만, 합계·평균·개수 등
여러 결과가 동시에 필요할 수 있다.

예를 들어 매장별로 다음 정보를 한 번에 확인한다고 생각해보자.

``` text
매장별 총매출
매장별 평균 매출
매장별 주문 건수
```

각각 별도로 `groupby()`를 실행하는 대신 `agg()`를 사용하면 한 번에
계산할 수 있다.

## 기본 구조

``` python
df.groupby("그룹_기준").agg(
    결과컬럼명=("대상컬럼", "집계함수"),
    결과컬럼명=("대상컬럼", "집계함수")
).reset_index()
```

------------------------------------------------------------------------

## 매장별 매출 현황 집계

``` python
result = (
    df.groupby("store")
      .agg(
          total_sales=("sales", "sum"),
          avg_sales=("sales", "mean"),
          order_count=("order_id", "count")
      )
      .reset_index()
)
```

### 각 코드의 의미

``` python
total_sales=("sales", "sum")
```

→ `sales`의 합계를 계산하고 결과 컬럼 이름을 `total_sales`로 지정

``` python
avg_sales=("sales", "mean")
```

→ `sales`의 평균을 계산하고 `avg_sales`로 저장

``` python
order_count=("order_id", "count")
```

→ `order_id`의 개수를 계산하고 `order_count`로 저장

------------------------------------------------------------------------

# 8. agg()의 장점

`agg()`의 가장 큰 장점은 **여러 집계 결과를 하나의 표에서 한 번에 계산할
수 있다는 것**이다.

예:

``` python
result = (
    df.groupby("region")
      .agg(
          total_sales=("revenue", "sum"),
          avg_sales=("revenue", "mean"),
          order_count=("order_id", "count")
      )
      .reset_index()
)
```

결과를 다음과 같은 형태로 만들 수 있다.

``` text
region | total_sales | avg_sales | order_count
```

또한 결과 컬럼에

``` text
total_sales
avg_sales
order_count
```

처럼 분석 목적을 명확하게 보여주는 이름을 직접 지정할 수 있다.

------------------------------------------------------------------------

# 9. agg() 실습 활용 패턴

## 상품 카테고리별 주문 현황

상품 카테고리별로 매출, 판매 수량, 주문 건수를 한 번에 확인할 수 있다.

``` python
result = (
    df.groupby("product_category")
      .agg(
          total_revenue=("revenue", "sum"),
          total_quantity=("quantity", "sum"),
          order_count=("order_id", "count")
      )
      .reset_index()
)
```

이를 통해 하나의 표에서

-   총매출
-   총 판매 수량
-   주문 건수

를 비교할 수 있다.

------------------------------------------------------------------------

## 회원 등급별 구매 행동

먼저 고객 정보와 주문 요약 데이터를 결합한다.

``` python
merged_df = pd.merge(
    customer_master,
    customer_order_summary,
    on="customer_id"
)
```

그 후 `membership_tier`를 기준으로 집계한다.

``` python
result = (
    merged_df.groupby("membership_tier")
             .agg(
                 total_sales=("total_spent", "sum"),
                 avg_spent=("total_spent", "mean"),
                 customer_count=("customer_id", "count")
             )
             .reset_index()
)
```

회원 등급별

-   전체 구매금액
-   평균 구매금액
-   고객 수

를 하나의 결과에서 확인할 수 있다.

------------------------------------------------------------------------

# 10. pivot_table(): 데이터를 표 형태로 재구성하기

`groupby()`를 사용하면 그룹별 집계 결과를 만들 수 있다.

하지만 **두 가지 기준을 행과 열에 배치하여 비교**하고 싶다면
`pivot_table()`이 더 편리하다.

학습자료에서는 다음 상황을 예로 들었다.

> 요일과 결제수단에 따라 주문 금액이 어떻게 달라지는지 비교한다.

요일을 행에 두고 결제수단을 열에 배치하면 결과를 훨씬 쉽게 비교할 수
있다.

## 기본 구조

``` python
pd.pivot_table(
    df,
    index="행에_배치할_컬럼",
    columns="열에_배치할_컬럼",
    values="집계할_값",
    aggfunc="집계함수",
    fill_value=0
)
```

------------------------------------------------------------------------

## 요일별·결제수단별 주문 금액 비교

``` python
pivot = pd.pivot_table(
    df,
    index="day",
    columns="payment",
    values="sales",
    aggfunc="sum",
    fill_value=0
)

print(pivot)
```

각 옵션의 의미는 다음과 같이 정리할 수 있다.

``` text
index      → 행 기준
columns    → 열 기준
values     → 실제 집계할 값
aggfunc    → 적용할 집계 함수
fill_value → 값이 없는 위치에 넣을 값
```

예를 들어

``` python
index="day"
```

→ 요일을 행에 배치

``` python
columns="payment"
```

→ 결제수단을 열에 배치

``` python
values="sales"
```

→ 주문 금액을 집계

``` python
aggfunc="sum"
```

→ 주문 금액의 합계를 계산

``` python
fill_value=0
```

→ 해당 조합의 데이터가 없으면 `0`으로 표시

------------------------------------------------------------------------

# 11. groupby()와 pivot_table()의 차이

둘 다 데이터를 집계하는 데 사용할 수 있지만 목적이 조금 다르다.

  기능              주요 목적
  ----------------- ----------------------------------------
  `groupby()`       특정 기준으로 데이터를 묶어 집계
  `agg()`           그룹별 여러 집계 결과를 한 번에 계산
  `pivot_table()`   집계 결과를 행과 열 기준의 표로 재구성

예를 들어:

``` text
지역별 총매출은?
→ groupby()

지역별 총매출 + 평균 매출 + 주문 건수는?
→ groupby() + agg()

요일 × 결제수단별 매출을 표로 비교하고 싶다면?
→ pivot_table()
```

------------------------------------------------------------------------

# 12. 오늘의 핵심 코드

``` python
import pandas as pd

# 정렬
sorted_data = sorted(data, reverse=True)

# groupby - 합계
result = (
    df.groupby("region")["sales"]
      .sum()
      .reset_index()
)

# groupby - 평균
result = (
    df.groupby("region")["sales"]
      .mean()
      .reset_index()
)

# groupby + agg
result = (
    df.groupby("region")
      .agg(
          total_sales=("revenue", "sum"),
          avg_sales=("revenue", "mean"),
          order_count=("order_id", "count")
      )
      .reset_index()
)

# pivot_table
pivot = pd.pivot_table(
    df,
    index="day",
    columns="payment",
    values="sales",
    aggfunc="sum",
    fill_value=0
)
```

------------------------------------------------------------------------

# 13. 오늘 배운 내용 정리

## groupby()

특정 컬럼을 기준으로 데이터를 그룹으로 나누고 각 그룹에 집계 함수를
적용한다.

``` text
Split → Apply → Combine
```

자주 사용하는 집계 함수:

``` python
sum()
mean()
count()
max()
min()
```

------------------------------------------------------------------------

## reset_index()

`groupby()` 후 그룹 기준이 index가 되었을 때 다시 일반 컬럼으로
변경한다.

``` python
df.groupby("region")["sales"].sum().reset_index()
```

------------------------------------------------------------------------

## agg()

하나의 그룹에 여러 집계 함수를 동시에 적용한다.

``` python
df.groupby("region").agg(
    total_sales=("revenue", "sum"),
    avg_sales=("revenue", "mean"),
    order_count=("order_id", "count")
).reset_index()
```

------------------------------------------------------------------------

## pivot_table()

두 개 이상의 기준을 행과 열로 배치하여 집계 결과를 비교하기 좋은 표
형태로 만든다.

``` python
pd.pivot_table(
    df,
    index="day",
    columns="payment",
    values="sales",
    aggfunc="sum",
    fill_value=0
)
```

------------------------------------------------------------------------

# 14. 오늘의 핵심 한 줄

> **`groupby()`로 데이터를 그룹화하고, `agg()`로 여러 지표를 집계하며,
> `pivot_table()`로 비교하기 좋은 표 형태로 재구성할 수 있다.**

------------------------------------------------------------------------

# 15. 느낀 점

오늘은 Pandas에서 단순히 데이터를 조회하는 것을 넘어 **데이터를 분석
목적에 맞게 요약하는 방법**을 배웠다.

특히 `groupby()`의 핵심이 단순히 문법을 외우는 것이 아니라 **분할(Split)
→ 적용(Apply) → 결합(Combine)** 과정이라는 점이 중요했다. 앞으로
`groupby()` 코드를 볼 때 어떤 컬럼을 기준으로 데이터를 나누고, 무엇을
계산하는지 생각하면 코드를 더 쉽게 이해할 수 있을 것 같다.

또한 처음에는 합계와 평균을 구하기 위해 각각 `groupby()`를 작성할 수
있지만, `agg()`를 사용하면 총매출, 평균 매출, 주문 건수처럼 여러 지표를
하나의 결과로 정리할 수 있다는 점이 유용했다. 실제 데이터 분석에서는
하나의 지표만 보는 경우보다 여러 지표를 함께 비교하는 경우가 많기 때문에
자주 사용하게 될 것 같다.

`pivot_table()`은 `groupby()`와 비슷하게 집계를 수행하지만, 요일과
결제수단처럼 **두 기준을 행과 열로 펼쳐 비교하기 쉽게 만든다는 차이**를
이해했다.

그리고 이전에 배운 `merge()`와 오늘의 `groupby()`를 함께 사용하여 고객
데이터와 주문 데이터를 결합한 뒤 회원 등급별 구매 행동을 집계하는 흐름도
연습했다. 각각의 Pandas 기능을 따로 배우는 것에서 끝나는 것이 아니라
실제 분석에서는 여러 기능을 연결해서 사용한다는 점을 기억해야겠다.

앞으로 집계 문제를 보면 먼저 다음 세 가지를 생각해보려고 한다.

> **무엇을 기준으로 묶을 것인가?\
> 어떤 값을 어떻게 집계할 것인가?\
> 결과를 어떤 형태로 비교할 것인가?**

이 세 가지를 먼저 정하면 `groupby()`, `agg()`, `pivot_table()` 중 어떤
방법을 사용해야 할지 판단하기 쉬울 것 같다.
