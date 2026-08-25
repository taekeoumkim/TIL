# \[2026-08-25\] Pandas 데이터 정렬과 변환 - sort_values, sort_index, lambda, split

## 학습 목표

-   `sort_values()`를 사용하여 특정 컬럼의 값을 기준으로 데이터를 정렬할
    수 있다.
-   `sort_index()`를 사용하여 인덱스를 기준으로 데이터를 정렬할 수 있다.
-   `reset_index(drop=True)`를 이용해 정렬 후 인덱스를 정리할 수 있다.
-   `apply()`와 `lambda`를 이용해 컬럼의 값을 간단하게 변환할 수 있다.
-   `split()`과 `.str.split()`의 차이를 이해하고 문자열 데이터를 분리할
    수 있다.

------------------------------------------------------------------------

## 1. 데이터 정렬과 변환은 왜 필요할까?

데이터 분석을 하다 보면 단순히 데이터를 조회하는 것뿐만 아니라 원하는
순서로 정리하거나 기존 값을 새로운 형태로 변환해야 하는 경우가 많다.

예를 들면 다음과 같다.

``` text
매출이 높은 주문부터 확인하기
배송 기간이 긴 주문부터 확인하기
고객 평점이 낮은 주문부터 확인하기
상품 가격에 할인율 적용하기
문자열로 된 상품 코드를 여러 정보로 분리하기
```

Pandas에서는 `sort_values()`와 `sort_index()`로 데이터를 정렬할 수 있고,
`apply()`와 `lambda`, `.str.split()` 등을 이용하여 데이터를 변환할 수
있다.

------------------------------------------------------------------------

# 2. sort_values(): 컬럼 기준 정렬

`sort_values()`는 **특정 컬럼의 값을 기준으로 DataFrame을 정렬**할 때
사용한다.

## 기본 구조

``` python
df.sort_values(
    "정렬할_컬럼",
    ascending=True
)
```

`ascending`의 의미는 다음과 같다.

``` text
ascending=True  → 오름차순 (기본값)
ascending=False → 내림차순
```

------------------------------------------------------------------------

## 3. 매출이 높은 상품부터 정렬하기

온라인 쇼핑몰의 상품별 매출 데이터가 있다고 가정한다.

``` python
import pandas as pd

df = pd.DataFrame({
    "product": ["노트북", "마우스", "키보드", "모니터"],
    "sales": [3200000, 850000, 1200000, 2400000]
})
```

매출이 높은 상품부터 확인하려면 `sales`를 기준으로 내림차순 정렬한다.

``` python
result = (
    df.sort_values(
        "sales",
        ascending=False
    )
    .reset_index(drop=True)
)

print(result)
```

결과:

``` text
  product    sales
0   노트북  3200000
1   모니터  2400000
2   키보드  1200000
3   마우스   850000
```

핵심은 다음 코드이다.

``` python
df.sort_values("sales", ascending=False)
```

`ascending=False`이므로 `sales`가 큰 값부터 작은 값 순으로 정렬된다.

------------------------------------------------------------------------

# 4. 오름차순과 내림차순

## 오름차순

작은 값 → 큰 값 순서로 정렬한다.

``` python
df.sort_values(
    "customer_rating",
    ascending=True
)
```

예를 들어 고객 만족도가 낮은 주문을 먼저 확인하고 싶다면 고객 평점을
오름차순으로 정렬할 수 있다.

``` text
1.0
2.0
3.0
4.0
5.0
```

## 내림차순

큰 값 → 작은 값 순서로 정렬한다.

``` python
df.sort_values(
    "revenue",
    ascending=False
)
```

예를 들어 매출이 높은 주문부터 확인할 때 사용할 수 있다.

``` text
500000
400000
300000
200000
100000
```

------------------------------------------------------------------------

# 5. 정렬 후 상위 데이터 확인하기

매출이 높은 상위 10개의 주문만 확인하고 싶다면 `sort_values()`와
`head()`를 함께 사용할 수 있다.

``` python
result = (
    df.sort_values(
        "revenue",
        ascending=False
    )
    .head(10)
)

print(result)
```

분석에서는 다음과 같은 패턴을 자주 사용할 수 있다.

``` python
df.sort_values("컬럼", ascending=False).head(10)
```

즉,

> **정렬 → 상위 데이터 추출**

의 흐름이다.

------------------------------------------------------------------------

# 6. 정렬 후 reset_index(drop=True)를 사용하는 이유

`sort_values()`를 사용하면 데이터의 순서는 변경되지만 기존 index는
그대로 유지될 수 있다.

예를 들어 원래 index가 다음과 같다고 가정한다.

``` text
0
1
2
3
```

정렬 결과가 다음처럼 될 수 있다.

``` text
2
0
3
1
```

데이터 자체에는 문제가 없지만 이후 `iloc` 등으로 데이터를 다룰 때 혼란이
생길 수 있다.

따라서 정렬 후 다음과 같이 index를 다시 정리할 수 있다.

``` python
df.sort_values(
    "sales",
    ascending=False
).reset_index(drop=True)
```

### drop=True의 의미

``` python
reset_index()
```

기존 index를 새로운 컬럼으로 남긴다.

반면,

``` python
reset_index(drop=True)
```

기존 index를 버리고 새로운 index를 `0`부터 부여한다.

정렬 후 기존 index가 필요하지 않다면 `drop=True`를 사용하는 것이
편리하다.

------------------------------------------------------------------------

# 7. 배송 기간이 긴 주문부터 확인하기

배송 지연 가능성이 높은 주문을 확인하고 싶다면 `delivery_days`를
내림차순으로 정렬할 수 있다.

``` python
result = (
    df.sort_values(
        "delivery_days",
        ascending=False
    )
    .reset_index(drop=True)
)

result[
    ["order_id", "delivery_days", "region"]
].head(10)
```

이렇게 하면 배송 기간이 가장 긴 주문부터 확인할 수 있다.

------------------------------------------------------------------------

# 8. 고객 평점이 낮은 주문부터 확인하기

고객 만족도가 낮은 주문을 먼저 확인하려면 `customer_rating`을
오름차순으로 정렬한다.

``` python
result = (
    df.sort_values(
        "customer_rating",
        ascending=True
    )
    .reset_index(drop=True)
)

result[
    ["order_id", "product_category", "customer_rating"]
].head(10)
```

여기서 중요한 것은 **분석 목적에 따라 오름차순과 내림차순을 선택해야
한다는 점**이다.

``` text
매출 높은 순 → descending → ascending=False
배송 오래 걸린 순 → descending → ascending=False
평점 낮은 순 → ascending → ascending=True
```

------------------------------------------------------------------------

# 9. sort_index(): 인덱스 기준 정렬

`sort_values()`가 컬럼의 **값(value)** 을 기준으로 정렬한다면,
`sort_index()`는 **index label**을 기준으로 정렬한다.

## 예제

``` python
sales = pd.Series(
    [1800000, 2500000, 1300000],
    index=["Seoul", "Busan", "Daegu"]
)

result = sales.sort_index()

print(result)
```

결과:

``` text
Busan    2500000
Daegu    1300000
Seoul    1800000
dtype: int64
```

여기서 `2500000`, `1300000`, `1800000`이라는 실제 데이터 값을 정렬한
것이 아니다.

``` text
Busan
Daegu
Seoul
```

이라는 **index label을 기준으로 정렬**한 것이다.

------------------------------------------------------------------------

# 10. sort_values()와 sort_index() 비교

  메서드            정렬 기준     사용 예
  ----------------- ------------- ----------------------------
  `sort_values()`   컬럼의 값     매출 높은 순, 평점 낮은 순
  `sort_index()`    index label   지역 이름순, 날짜 index 순

쉽게 기억하면 다음과 같다.

``` text
값(value)을 정렬하고 싶은가?
→ sort_values()

index를 정렬하고 싶은가?
→ sort_index()
```

------------------------------------------------------------------------

# 11. groupby()와 sort_index() 함께 사용하기

이전에 배운 `groupby()`와 `sort_index()`를 함께 사용할 수도 있다.

예를 들어 지역별 매출 합계를 계산한다.

``` python
region_sales = (
    df.groupby("region")["revenue"]
      .sum()
)
```

이때 `region`이 index가 된다.

지역 이름을 기준으로 정렬하려면 다음과 같이 작성한다.

``` python
result = region_sales.sort_index()
```

상품 카테고리별 매출도 같은 방식으로 정리할 수 있다.

``` python
category_sales = (
    df.groupby(
        "product_category",
        sort=False
    )["revenue"]
    .sum()
)

sorted_result = category_sales.sort_index()
```

`groupby(sort=False)`를 사용하면 원본 데이터에 등장한 순서를 유지하고,
이후 `sort_index()`를 이용하여 index를 원하는 순서로 정렬할 수 있다.

------------------------------------------------------------------------

# 12. lambda: 이름 없는 익명 함수

`lambda`는 **이름을 만들지 않고 한 줄로 작성하는 간단한 함수**이다.

일반 함수는 다음처럼 작성한다.

``` python
def double(x):
    return x * 2
```

같은 기능을 `lambda`로 표현하면 다음과 같다.

``` python
lambda x: x * 2
```

구조를 보면 다음과 같다.

``` text
lambda 입력값: 반환할 표현식
```

즉,

``` python
lambda x: x * 2
```

는

> x를 입력받아 x에 2를 곱한 값을 반환한다.

라는 의미이다.

------------------------------------------------------------------------

# 13. apply() + lambda

Pandas에서는 `apply()`와 `lambda`를 함께 사용하여 Series의 각 값에
간단한 변환을 적용할 수 있다.

예를 들어 상품 가격에 20% 할인을 적용한다고 가정한다.

``` python
df = pd.DataFrame({
    "product": ["노트북", "마우스", "키보드"],
    "price": [1000000, 50000, 100000]
})
```

새로운 할인 가격 컬럼을 만든다.

``` python
df["discount_price"] = (
    df["price"]
    .apply(lambda x: x * 0.8)
)

print(df)
```

결과:

``` text
product    price    discount_price
노트북    1000000      800000.0
마우스      50000       40000.0
키보드     100000       80000.0
```

`apply()`가 `price` 컬럼의 각 값에 `lambda` 함수를 적용한다.

------------------------------------------------------------------------

# 14. 새로운 컬럼 만들기

`apply()`와 `lambda`는 기존 데이터를 변환하여 새로운 컬럼을 만들 때
활용할 수 있다.

## 매출에 10% 반영하기

``` python
df["revenue_adjusted"] = (
    df["revenue"]
    .apply(lambda x: x * 1.1)
)
```

기존 `revenue` 값에 `1.1`을 곱한 값을 `revenue_adjusted`라는 새로운
컬럼에 저장한다.

------------------------------------------------------------------------

## 할인율을 백분율로 변환하기

할인율이 다음과 같이 소수 형태로 저장되어 있다고 가정한다.

``` text
0.1
0.2
0.3
```

백분율 값으로 바꾸려면 다음과 같이 작성할 수 있다.

``` python
df["discount_percent"] = (
    df["discount"]
    .apply(lambda x: x * 100)
)
```

결과:

``` text
0.1 → 10
0.2 → 20
0.3 → 30
```

------------------------------------------------------------------------

# 15. lambda에 조건식 사용하기

`lambda` 안에서도 간단한 조건식을 사용할 수 있다.

배송 기간이 3일 이하이면 `"빠른 배송"`, 그 외에는 `"일반 배송"`으로
구분한다고 가정한다.

``` python
df["delivery_type"] = (
    df["delivery_days"]
    .apply(
        lambda x: "빠른 배송"
        if x <= 3
        else "일반 배송"
    )
)
```

조건식 구조는 다음과 같다.

``` python
lambda x: 참일_때_값 if 조건 else 거짓일_때_값
```

이를 활용하면 특정 기준에 따라 데이터를 간단하게 분류할 수 있다.

------------------------------------------------------------------------

# 16. lambda와 def는 언제 사용할까?

둘 다 함수를 만들 수 있지만 사용 목적이 다르다.

## lambda

간단하고 한 줄로 표현할 수 있는 변환에 적합하다.

``` python
lambda x: x * 100
```

``` python
lambda x: "빠른 배송" if x <= 3 else "일반 배송"
```

## def

로직이 복잡하거나 여러 번 재사용해야 할 때 적합하다.

``` python
def classify_delivery(days):
    if days <= 3:
        return "빠른 배송"
    else:
        return "일반 배송"
```

정리하면:

``` text
간단한 한 줄 변환
→ lambda

복잡한 로직 또는 재사용할 함수
→ def
```

------------------------------------------------------------------------

# 17. split(): 문자열 분리

`split()`은 문자열을 특정 **구분자(delimiter)** 를 기준으로 나누는
메서드이다.

예를 들어 다음과 같은 상품 코드가 있다고 가정한다.

``` text
FOOD-001
BOOK-002
TOY-003
```

상품 코드에는

``` text
카테고리 - 번호
```

라는 두 정보가 들어 있다.

이 값을 `-`를 기준으로 나누면 카테고리와 번호를 각각 분석할 수 있다.

------------------------------------------------------------------------

# 18. 문자열 하나에서 split() 사용하기

일반 Python 문자열 하나를 분리할 때는 `.split()`을 사용한다.

``` python
product_code = "FOOD-001"

result = product_code.split("-")

print(result)
```

결과:

``` text
['FOOD', '001']
```

즉,

``` python
문자열.split("구분자")
```

형태로 사용한다.

------------------------------------------------------------------------

# 19. Pandas에서 .str.split() 사용하기

DataFrame이나 Series의 문자열 컬럼 전체에 문자열 메서드를 적용할 때는
`.str` 접근자를 사용한다.

``` python
df["product_code"].str.split("-")
```

여기에 `expand=True`를 사용하면 분리된 결과를 여러 컬럼으로 펼칠 수
있다.

``` python
df[["category", "number"]] = (
    df["product_code"]
    .str.split(
        "-",
        expand=True
    )
)
```

결과:

``` text
product_code | category | number
FOOD-001     | FOOD     | 001
BOOK-002     | BOOK     | 002
TOY-003      | TOY      | 003
```

------------------------------------------------------------------------

# 20. split()과 .str.split() 차이

이 부분은 헷갈리기 쉬우므로 구분해서 기억해야 한다.

## 문자열 하나

``` python
"2026-08-25".split("-")
```

`.str`이 필요 없다.

## Pandas Series 전체

``` python
df["date"].str.split("-")
```

`.str`을 붙여야 한다.

정리하면:

``` text
문자열 값 하나
→ split()

Series의 문자열 전체
→ .str.split()
```

`.str`은 **이 컬럼의 모든 값에 문자열 메서드를 적용한다**고 이해하면
된다.

------------------------------------------------------------------------

# 21. expand=True

`.str.split()`을 사용할 때 `expand=True`를 지정하면 나누어진 값을 여러
컬럼으로 만들 수 있다.

예를 들어 주문 날짜가 `/`로 구분되어 있다고 가정한다.

``` text
08/25/2026
```

다음과 같이 분리할 수 있다.

``` python
df[["month", "day", "year"]] = (
    df["order_date"]
    .str.split(
        "/",
        expand=True
    )
)
```

결과:

``` text
order_date  | month | day | year
08/25/2026  | 08    | 25  | 2026
```

------------------------------------------------------------------------

# 22. 고객 ID 분리하기

고객 ID가 다음과 같이 저장되어 있다고 가정한다.

``` text
CUST-001
CUST-002
```

`-`를 기준으로 문자 부분과 번호 부분을 나눌 수 있다.

``` python
df[["prefix", "number"]] = (
    df["customer_id"]
    .str.split(
        "-",
        expand=True
    )
)
```

결과:

``` text
customer_id | prefix | number
CUST-001    | CUST   | 001
CUST-002    | CUST   | 002
```

이처럼 하나의 문자열 안에 여러 정보가 포함되어 있다면 `str.split()`을
이용해 분석에 필요한 컬럼을 새롭게 만들 수 있다.

------------------------------------------------------------------------

# 23. 오늘의 핵심 코드

``` python
import pandas as pd

# 1. 컬럼 값 기준 오름차순
result = df.sort_values(
    "customer_rating",
    ascending=True
)

# 2. 컬럼 값 기준 내림차순
result = df.sort_values(
    "revenue",
    ascending=False
)

# 3. 정렬 후 index 재설정
result = (
    df.sort_values(
        "revenue",
        ascending=False
    )
    .reset_index(drop=True)
)

# 4. index 기준 정렬
result = df.sort_index()

# 5. apply + lambda
df["revenue_adjusted"] = (
    df["revenue"]
    .apply(lambda x: x * 1.1)
)

# 6. lambda 조건식
df["delivery_type"] = (
    df["delivery_days"]
    .apply(
        lambda x: "빠른 배송"
        if x <= 3
        else "일반 배송"
    )
)

# 7. 문자열 하나 분리
"FOOD-001".split("-")

# 8. Series 문자열 분리
df[["category", "number"]] = (
    df["product_code"]
    .str.split(
        "-",
        expand=True
    )
)
```

------------------------------------------------------------------------

# 24. 오늘 배운 내용 정리

## sort_values()

특정 컬럼의 **값을 기준으로 정렬**한다.

``` python
df.sort_values("sales")
```

``` text
ascending=True  → 오름차순
ascending=False → 내림차순
```

------------------------------------------------------------------------

## sort_index()

DataFrame 또는 Series의 **index label을 기준으로 정렬**한다.

``` python
df.sort_index()
```

`groupby()` 결과처럼 index가 의미 있는 값을 가지고 있을 때 유용하다.

------------------------------------------------------------------------

## reset_index(drop=True)

정렬 후 기존 index를 버리고 `0`부터 새로운 index를 만든다.

``` python
df.reset_index(drop=True)
```

------------------------------------------------------------------------

## lambda

이름 없는 간단한 함수를 한 줄로 작성한다.

``` python
lambda x: x * 2
```

Pandas에서는 `apply()`와 함께 컬럼의 각 값을 변환할 때 활용할 수 있다.

``` python
df["price"].apply(
    lambda x: x * 0.8
)
```

------------------------------------------------------------------------

## split()

문자열 하나를 특정 구분자로 나눈다.

``` python
"FOOD-001".split("-")
```

------------------------------------------------------------------------

## .str.split()

Pandas Series의 문자열 전체를 분리한다.

``` python
df["product_code"].str.split(
    "-",
    expand=True
)
```

`expand=True`를 사용하면 분리된 값을 여러 컬럼으로 만들 수 있다.

------------------------------------------------------------------------

# 25. 오늘의 핵심 한 줄

> **`sort_values()`와 `sort_index()`로 데이터를 원하는 순서로 정리하고,
> `lambda`와 `split()`을 이용해 분석하기 좋은 형태로 데이터를 변환할 수
> 있다.**

------------------------------------------------------------------------

# 26. 느낀 점

오늘은 Pandas에서 데이터를 단순히 조회하거나 집계하는 것에서 더 나아가
**분석하기 좋은 형태로 데이터를 정렬하고 변환하는 방법**을 배웠다.

`sort_values()`와 `sort_index()`는 둘 다 정렬 기능이지만 무엇을 기준으로
정렬하는지가 다르다는 점이 중요했다. `sort_values()`는 실제 컬럼 값을
기준으로 정렬하고, `sort_index()`는 index를 기준으로 정렬한다. 앞으로
정렬 문제를 보면 먼저 **값을 정렬하려는 것인지 index를 정렬하려는
것인지** 판단해야겠다.

또한 정렬 후에도 기존 index가 그대로 남아 있을 수 있기 때문에
`reset_index(drop=True)`를 사용하여 index를 다시 정리하는 이유도
이해했다. 특히 이후 `iloc` 등을 사용할 때 혼란을 줄이기 위해 정렬 후
index 상태를 확인하는 습관을 들이는 것이 좋을 것 같다.

`lambda`는 처음 보면 일반 함수와 문법이 달라 조금 낯설지만, `apply()`와
함께 사용하면 컬럼의 모든 값에 간단한 계산이나 조건을 적용할 수 있다는
점이 유용했다. 다만 모든 함수를 `lambda`로 작성하는 것이 아니라 **간단한
한 줄 변환에는 lambda, 복잡하거나 재사용해야 하는 로직에는 def**를
사용하는 것이 좋다는 점도 기억해야겠다.

`split()`에서는 일반 Python 문자열과 Pandas Series에서 사용하는 방식이
다르다는 점이 중요했다.

``` text
문자열 하나 → split()
Series 전체 → .str.split()
```

특히 `expand=True`를 이용하면 상품 코드나 날짜처럼 하나의 문자열에 여러
정보가 포함된 데이터를 각각의 컬럼으로 분리할 수 있기 때문에 실제 데이터
전처리에서 자주 활용할 수 있을 것 같다.

오늘 배운 내용을 정리하면 데이터 분석에서는 단순히 값을 계산하는 것뿐만
아니라 **어떤 순서로 데이터를 볼 것인지, 기존 데이터를 어떤 형태로
변환할 것인지**도 중요하다는 것을 알게 되었다.

앞으로 데이터를 다룰 때는 다음과 같이 생각해보려고 한다.

> **값을 기준으로 정렬할까? → `sort_values()`\
> index를 기준으로 정렬할까? → `sort_index()`\
> 간단한 값을 변환할까? → `apply()` + `lambda`\
> 문자열을 여러 정보로 나눌까? → `split()` / `.str.split()`**