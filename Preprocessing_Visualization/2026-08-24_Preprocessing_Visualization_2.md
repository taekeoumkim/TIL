# \[2026-08-24\] Pandas 데이터 결합 - concat, merge, join

## 학습 목표

-   `concat()`, `merge()`, `join()`의 차이를 이해하고 상황에 맞는 결합
    방법을 선택할 수 있다.
-   `concat()`을 이용해 DataFrame을 행 또는 열 방향으로 연결할 수 있다.
-   `merge()`를 이용해 공통 키 컬럼을 기준으로 데이터를 병합할 수 있다.
-   `inner`, `left`, `right`, `outer` JOIN의 차이를 이해한다.
-   인덱스를 기준으로 결합할 때 `join()`을 사용할 수 있다.

------------------------------------------------------------------------

## 1. 왜 데이터를 결합해야 할까?

실무의 데이터는 하나의 파일에 모두 저장되어 있지 않은 경우가 많다.

예를 들어 이커머스 데이터를 분석한다고 하면 다음처럼 데이터가 나누어져
있을 수 있다.

-   온라인 주문 데이터
-   오프라인 주문 데이터
-   고객 기본 정보
-   고객별 주문 요약 데이터

고객 정보와 주문 내역처럼 서로 다른 테이블에 흩어진 정보를 함께
분석하려면 여러 데이터를 하나로 결합해야 한다.

Pandas에서는 대표적으로 다음 세 가지 방법을 사용할 수 있다.

``` text
concat() → DataFrame을 위아래 또는 좌우로 단순 연결
merge()  → 공통 키 컬럼을 기준으로 병합
join()   → 인덱스를 기준으로 결합
```

------------------------------------------------------------------------

## 2. concat(): DataFrame 단순 연결

`pd.concat()`은 두 개 이상의 DataFrame을 **물리적으로 이어붙일 때**
사용한다.

SQL의 `UNION`과 비슷한 개념으로 볼 수 있으며, 공통 키를 기준으로
데이터를 매칭하는 것이 아니라 DataFrame 자체를 행 또는 열 방향으로
연결한다.

### 기본 문법

``` python
pd.concat(
    [df1, df2],
    axis=0,
    ignore_index=True
)
```

### axis의 의미

``` text
axis=0 → 행 방향으로 연결 (위아래)
axis=1 → 열 방향으로 연결 (좌우)
```

------------------------------------------------------------------------

## 3. concat()으로 행 방향 결합하기

7월과 8월 주문 데이터가 동일한 컬럼 구조를 가지고 있다고 가정한다.

``` python
import pandas as pd

july_orders = pd.DataFrame({
    "order_id": [101, 102],
    "product": ["노트북", "마우스"]
})

august_orders = pd.DataFrame({
    "order_id": [103, 104],
    "product": ["키보드", "모니터"]
})
```

두 DataFrame을 위아래로 합치려면 `axis=0`을 사용한다.

``` python
all_orders = pd.concat(
    [july_orders, august_orders],
    axis=0,
    ignore_index=True
)

print(all_orders)
```

결과:

``` text
   order_id product
0       101     노트북
1       102     마우스
2       103     키보드
3       104     모니터
```

### ignore_index=True

각 DataFrame은 원래 별도의 index를 가지고 있기 때문에 그대로 합치면
index가 중복될 수 있다.

``` text
0
1
0
1
```

`ignore_index=True`를 사용하면 결합된 DataFrame의 index가 다시 정리된다.

``` text
0
1
2
3
```

이후 `loc`, `iloc` 등으로 데이터를 다룰 때 혼란을 줄일 수 있다.

``` python
pd.concat(
    [df1, df2],
    axis=0,
    ignore_index=True
)
```

행 방향으로 `concat()`을 사용할 때는 `ignore_index=True`를 함께 사용하는
습관을 들이는 것이 좋다.

------------------------------------------------------------------------

## 4. 컬럼이 다른 DataFrame을 concat()하면?

두 DataFrame의 컬럼 구조가 다를 수도 있다.

``` python
store_a = pd.DataFrame({
    "product": ["노트북", "마우스"],
    "price": [1200000, 35000]
})

store_b = pd.DataFrame({
    "product": ["키보드", "모니터"],
    "quantity": [5, 3]
})
```

이를 행 방향으로 결합한다.

``` python
result = pd.concat(
    [store_a, store_b],
    axis=0,
    ignore_index=True
)

print(result)
```

두 DataFrame에 공통으로 존재하지 않는 컬럼은 해당 행에서 **NaN**으로
채워진다.

``` text
product   price      quantity
노트북      값           NaN
마우스      값           NaN
키보드      NaN          값
모니터      NaN          값
```

따라서 `concat()`을 사용하기 전에 두 DataFrame의 컬럼 구조가 동일한지
확인하는 것이 중요하다.

------------------------------------------------------------------------

## 5. concat()으로 열 방향 결합하기

`axis=1`을 사용하면 DataFrame을 좌우로 연결할 수 있다.

``` python
product_info = pd.DataFrame({
    "product": ["노트북", "마우스", "키보드"],
    "price": [1200000, 35000, 89000]
})

sales_info = pd.DataFrame({
    "quantity": [2, 10, 5],
    "stock": [5, 30, 12]
})

result = pd.concat(
    [product_info, sales_info],
    axis=1
)

print(result)
```

``` text
axis=0 → ↓ 위아래 연결
axis=1 → → 좌우 연결
```

------------------------------------------------------------------------

## 6. merge(): 공통 키를 기준으로 병합

`merge()`는 단순히 데이터를 이어붙이는 `concat()`과 달리 **두
DataFrame의 공통 키 컬럼을 기준으로 관련된 행을 찾아 연결**한다.

SQL의 `JOIN`과 같은 방식이다.

예를 들어 주문 데이터에는 `user_id`만 있고 고객의 이름은 별도의 고객
데이터에 저장되어 있다고 가정한다.

``` python
orders = pd.DataFrame({
    "order_id": [101, 102, 103],
    "user_id": [1, 2, 3],
    "amount": [30000, 15000, 50000]
})

users = pd.DataFrame({
    "user_id": [1, 2, 3],
    "name": ["민수", "지영", "현우"]
})
```

두 DataFrame에는 `user_id`라는 공통 컬럼이 있다.

``` python
result = pd.merge(
    orders,
    users,
    on="user_id",
    how="left"
)

print(result)
```

`on="user_id"`를 지정하면 같은 `user_id`를 가진 행끼리 연결된다.

### 기본 구조

``` python
pd.merge(
    left_df,
    right_df,
    on="공통_키",
    how="JOIN_방식"
)
```

------------------------------------------------------------------------

## 7. JOIN 유형 이해하기

`merge()`에서는 `how`를 이용해 데이터를 어떤 방식으로 병합할지 지정할 수
있다.

  JOIN      동작
  --------- ---------------------------------------
  `inner`   양쪽 데이터에 모두 존재하는 키만 유지
  `left`    왼쪽 데이터는 모두 유지
  `right`   오른쪽 데이터는 모두 유지
  `outer`   양쪽 데이터를 모두 유지

### inner join

양쪽 DataFrame에 **모두 존재하는 키만** 남긴다.

``` python
result = pd.merge(
    df_left,
    df_right,
    on="customer_id",
    how="inner"
)
```

``` text
왼쪽 데이터     오른쪽 데이터
   A                A
   B                B
   C                D

        ↓ inner

        A
        B
```

양쪽 데이터의 일치 여부를 엄격하게 확인할 때 사용할 수 있다.

------------------------------------------------------------------------

### left join

**왼쪽 DataFrame을 모두 유지**하면서 오른쪽 데이터를 추가한다.

``` python
result = pd.merge(
    df_left,
    df_right,
    on="customer_id",
    how="left"
)
```

오른쪽 데이터에 일치하는 값이 없다면 해당 컬럼에는 `NaN`이 들어간다.

``` text
왼쪽 데이터     오른쪽 데이터
   A                A
   B                B
   C                D

        ↓ left

        A
        B
        C → 오른쪽 정보가 없으면 NaN
```

예를 들어 고객별 주문 데이터를 모두 유지하면서 고객의 회원 등급을
추가하고 싶다면 `left join`을 사용할 수 있다.

------------------------------------------------------------------------

### right join

`left join`과 반대로 **오른쪽 DataFrame을 모두 유지**한다.

``` python
result = pd.merge(
    df_left,
    df_right,
    on="customer_id",
    how="right"
)
```

왼쪽에 일치하는 데이터가 없으면 해당 부분은 `NaN`으로 표시된다.

------------------------------------------------------------------------

### outer join

양쪽 DataFrame의 데이터를 **모두 유지**한다.

``` python
result = pd.merge(
    df_left,
    df_right,
    on="customer_id",
    how="outer"
)
```

한쪽에만 존재하는 키도 결과에 포함되며, 상대 DataFrame에 값이 없는
부분은 `NaN`이 된다.

------------------------------------------------------------------------

## 8. inner join과 left join의 차이

두 방식의 가장 중요한 차이는 **기준 데이터의 행을 유지할 것인가**이다.

``` text
inner join
→ 양쪽에 모두 존재하는 데이터만 필요할 때

left join
→ 왼쪽 데이터를 모두 유지하면서
   오른쪽 정보를 추가하고 싶을 때
```

예를 들어 `customer_order_summary`에 존재하는 모든 고객의 주문 정보를
유지하면서 `customer_master`의 회원 등급을 추가하고 싶다면 다음과 같이
작성할 수 있다.

``` python
result = pd.merge(
    customer_order_summary,
    customer_master,
    on="customer_id",
    how="left"
)
```

반면 양쪽 데이터에 모두 존재하는 고객만 분석하고 싶다면 `inner join`을
사용한다.

``` python
result = pd.merge(
    customer_order_summary,
    customer_master,
    on="customer_id",
    how="inner"
)
```

`inner`와 `left`의 결과 행 개수가 다를 수 있으므로 병합 전후의 `shape`을
확인하는 것도 중요하다.

``` python
print(customer_order_summary.shape)
print(result.shape)
```

------------------------------------------------------------------------

## 9. join(): 인덱스 기반 결합

`join()`은 **결합 기준이 이미 DataFrame의 index로 설정되어 있을 때**
사용할 수 있다.

예를 들어 직원 번호가 두 DataFrame의 index로 설정되어 있다고 가정한다.

``` python
employee_info = pd.DataFrame({
    "name": ["민수", "지영", "현우"]
}, index=[101, 102, 103])

employee_score = pd.DataFrame({
    "score": [90, 85, 95]
}, index=[101, 102, 103])
```

두 DataFrame의 index를 기준으로 결합한다.

``` python
result = employee_info.join(employee_score)

print(result)
```

``` text
       name  score
101    민수     90
102    지영     85
103    현우     95
```

즉, 공통 컬럼을 기준으로 연결할 때는 `merge()`, 결합 기준이 index로
설정되어 있다면 `join()`을 사용할 수 있다.

------------------------------------------------------------------------

## 10. concat(), merge(), join() 비교

  메서드       결합 기준      주요 사용 상황
  ------------ -------------- -----------------------------------------
  `concat()`   행/열 방향     여러 DataFrame을 단순히 이어붙일 때
  `merge()`    공통 키 컬럼   관련된 데이터를 키를 기준으로 병합할 때
  `join()`     index          index를 기준으로 데이터를 결합할 때

### 어떤 방법을 선택할까?

``` text
데이터를 그냥 위아래/좌우로 붙이고 싶은가?
        ↓
     concat()

공통된 ID나 키를 기준으로 연결해야 하는가?
        ↓
      merge()

결합 기준이 이미 index인가?
        ↓
       join()
```

------------------------------------------------------------------------

## 11. 오늘의 핵심 Pandas 코드

``` python
import pandas as pd

# 1. 행 방향 concat
all_orders = pd.concat(
    [df_online, df_offline],
    axis=0,
    ignore_index=True
)

# 2. 열 방향 concat
result = pd.concat(
    [product_info, sales_info],
    axis=1
)

# 3. inner join
inner_result = pd.merge(
    customer_order_summary,
    customer_master,
    on="customer_id",
    how="inner"
)

# 4. left join
left_result = pd.merge(
    customer_order_summary,
    customer_master,
    on="customer_id",
    how="left"
)

# 5. right join
right_result = pd.merge(
    df_left,
    df_right,
    on="customer_id",
    how="right"
)

# 6. outer join
outer_result = pd.merge(
    df_left,
    df_right,
    on="customer_id",
    how="outer"
)

# 7. index 기반 join
result = df_left.join(df_right)
```

------------------------------------------------------------------------

## 12. 오늘 배운 내용 정리

### concat()

-   DataFrame을 단순하게 이어붙이는 방법
-   `axis=0` → 행 방향(위아래)
-   `axis=1` → 열 방향(좌우)
-   행 방향 결합 시 `ignore_index=True`를 사용하면 index를 다시 정리할
    수 있음
-   컬럼이 다른 데이터를 행 방향으로 결합하면 없는 값은 `NaN`으로 채워짐

### merge()

-   공통 키 컬럼을 기준으로 두 DataFrame을 병합
-   SQL의 JOIN과 같은 개념
-   `on` → 병합 기준 컬럼
-   `how` → JOIN 방식

``` text
inner → 양쪽에 모두 있는 데이터
left  → 왼쪽 데이터 모두 유지
right → 오른쪽 데이터 모두 유지
outer → 양쪽 데이터 모두 유지
```

### join()

-   index를 기준으로 DataFrame을 결합
-   결합 기준이 이미 index로 설정되어 있을 때 편리함

### 가장 중요한 차이

``` text
concat() = 단순 연결
merge()  = 공통 키 기반 병합
join()   = index 기반 결합
```

------------------------------------------------------------------------

## 13. 느낀 점

오늘은 Pandas에서 여러 DataFrame을 하나로 결합하는 방법을 배웠다.

처음에는 `concat()`, `merge()`, `join()`이 모두 데이터를 합치는 기능이라
비슷하게 느껴졌지만, 실제로는 **무엇을 기준으로 데이터를 합치는지**가
다르다는 점이 가장 중요하다고 느꼈다.

`concat()`은 같은 구조의 온라인/오프라인 주문 데이터처럼 데이터를 단순히
이어붙일 때 사용하고, `merge()`는 `customer_id`와 같은 공통 키를 이용해
서로 관련된 정보를 연결할 때 사용한다. 또한 결합 기준이 이미 index로
설정되어 있다면 `join()`을 사용할 수 있다는 것을 알게 되었다.

특히 `merge()`의 JOIN 방식은 SQL에서 배웠던 JOIN과 연결되는 개념이라
이해하기 쉬웠다. 하지만 `inner`와 `left`에 따라 결과의 행 개수가 달라질
수 있기 때문에 단순히 코드를 실행하는 것보다 **어떤 데이터를 반드시
유지해야 하는지 먼저 생각하고 JOIN 방식을 선택하는 것**이 중요할 것
같다.

앞으로 데이터를 결합할 때는 다음 순서로 생각해봐야겠다.

> **단순히 이어붙이는가? → concat()\
> 공통 키로 연결하는가? → merge()\
> index를 기준으로 연결하는가? → join()**

그리고 `merge()`를 사용한 뒤에는 `shape`이나 결측값을 확인하여 의도한
방식으로 데이터가 결합되었는지도 확인하는 습관을 들여야겠다.