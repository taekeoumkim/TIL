# 2026-08-24 Pandas 기초와 데이터 확인

## 학습 목표

-   Pandas의 역할과 데이터 전처리의 필요성을 이해한다.
-   DataFrame과 Series의 차이를 구분한다.
-   CSV/Excel 파일을 불러오고 저장하는 방법을 익힌다.
-   `head()`, `tail()`, `shape`, `dtypes`, `info()`로 데이터 구조를
    확인한다.
-   데이터 타입을 변환하고 데이터 분포를 파악한다.
-   `iloc`, `loc`, Boolean Indexing으로 데이터를 선택하고 필터링한다.

------------------------------------------------------------------------

## 1. 데이터 전처리(Data Preprocessing)

실제 데이터에는 결측값, 오류, 중복, 이상값, 오타, 단위 불일치 등이
존재할 수 있다. 이러한 원시 데이터를 분석이나 모델링에 적합한 형태로
정리하는 과정을 **데이터 전처리**라고 한다.

``` text
원본 데이터 → 문제 확인 → 정제/변환 → 분석 가능한 데이터 → 분석/모델링
```

전처리 전에 **무엇을 확인할 것인지**, **어떤 의사결정을 위한
분석인지**를 먼저 정의하는 것이 중요하다.

------------------------------------------------------------------------

## 2. Pandas란?

Pandas는 Python에서 데이터를 조작하고 분석하기 위한 라이브러리이다.
Excel처럼 행과 열로 구성된 표 형태의 데이터를 Python에서 다룰 수 있다.

``` python
import pandas as pd

orders = {
    "order_id": [101, 102, 103],
    "product": ["노트북", "마우스", "키보드"],
    "price": [1200000, 35000, 89000]
}
df = pd.DataFrame(orders)
```

Pandas를 이용하면 데이터를 불러오고, 구조를 확인하고, 필요한 데이터를
선택하고, 값을 계산하거나 정리할 수 있다.

------------------------------------------------------------------------

## 3. DataFrame과 Series

### DataFrame

행(row)과 열(column)로 이루어진 **2차원 표 형태의 자료구조**이다. 각
행은 하나의 데이터 레코드, 각 열은 하나의 특성(feature)을 나타낸다.

### Series

값과 index로 이루어진 **1차원 자료구조**이며 DataFrame의 한 열에
해당한다.

``` python
# Series 반환
product_series = df["product"]

# DataFrame 형태 유지
product_df = df[["product"]]
```

  구분             DataFrame       Series
  ---------------- --------------- -------------
  차원             2차원           1차원
  구조             행 + 열         값 + index
  단일 컬럼 선택   `df[["col"]]`   `df["col"]`

------------------------------------------------------------------------

## 4. 데이터 파일 불러오기와 저장하기

``` python
# CSV 불러오기
df = pd.read_csv("data.csv")

# Excel 불러오기
df = pd.read_excel("data.xlsx")

# CSV 저장
df.to_csv("output.csv", index=False)

# Excel 저장
df.to_excel("output.xlsx", index=False)
```

`index=False`를 사용하면 Pandas가 자동으로 생성한 `0, 1, 2, ...`
인덱스가 별도 데이터 열로 저장되지 않는다.

------------------------------------------------------------------------

## 5. Index 이해하기

Index는 DataFrame이나 Series에서 **각 행을 식별하는 값**이다. 별도로
지정하지 않으면 Pandas가 `0, 1, 2, ...` 형태의 기본 index를 생성한다.

------------------------------------------------------------------------

## 6. 데이터를 받으면 가장 먼저 확인할 것

데이터를 받자마자 분석하기보다 먼저 데이터의 모양과 상태를 확인해야
한다.

``` text
몇 행과 몇 열인가?
어떤 컬럼이 있는가?
각 컬럼의 데이터 타입은 무엇인가?
결측값은 존재하는가?
실제 값은 어떤 형태인가?
```

------------------------------------------------------------------------

## 7. 데이터 미리보기: head(), tail()

``` python
df.head()      # 앞 5행
df.head(10)    # 앞 10행
df.tail()      # 뒤 5행
df.tail(2)     # 뒤 2행
```

CSV를 불러온 직후 `head()`를 사용하면 파일이 정상적으로 불러와졌는지와
각 컬럼에 어떤 값이 저장되어 있는지 빠르게 확인할 수 있다.

------------------------------------------------------------------------

## 8. 데이터 구조 확인

  코드            역할
  --------------- -----------------------------------
  `df.shape`      행과 열 개수
  `df.columns`    컬럼명
  `df.dtypes`     각 컬럼의 데이터 타입
  `df.info()`     결측값, 타입, 메모리 등 종합 정보
  `df.rename()`   컬럼명 변경

``` python
print(df.shape)
print(df.columns)
print(df.dtypes)
df.info()

df = df.rename(columns={"price": "order_price"})
```

예를 들어 `shape`이 `(5000, 12)`라면 데이터가 5,000행과 12열로
구성되었다는 뜻이다.

------------------------------------------------------------------------

## 9. 데이터 타입 확인과 변환

숫자처럼 보이는 값도 실제로는 문자열(`object`)일 수 있다. 문자열이면
평균이나 합계 같은 수치 연산을 제대로 수행하기 어렵기 때문에 분석 전에
타입을 확인해야 한다.

  타입           의미
  -------------- ---------------------
  `int64`        정수형
  `float64`      실수형
  `object`       문자열(주로 텍스트)
  `bool`         True/False
  `datetime64`   날짜/시간
  `category`     범주형

``` python
# 문자열 → 정수
products["price"] = products["price"].astype(int)

# 문자열 → 날짜
products["sale_date"] = pd.to_datetime(products["sale_date"])
```

적절한 데이터 타입을 사용해야 연산을 정확하게 수행하고 이후 날짜 기반
분석 등에도 활용할 수 있다.

------------------------------------------------------------------------

## 10. 데이터 분포 파악

### describe()

수치형 데이터의 기본 통계량을 확인한다.

``` python
num_cols = ["quantity", "unit_price", "discount", "delivery_days", "customer_rating", "revenue"]
df[num_cols].describe()
```

`count`, `mean`, `std`, `min`, `25%`, `50%`, `75%`, `max` 등을 확인할 수
있다.

### value_counts()

각 값이 몇 번 등장했는지 확인한다.

``` python
df["product_category"].value_counts()
```

### nunique()

중복을 제외한 고유한 값의 개수를 확인한다.

``` python
df["customer_id"].nunique()
```

### count()와 nunique() 차이

``` text
count()   → 결측값이 아닌 데이터의 총 개수
nunique() → 중복을 제외한 고유한 값의 개수
```

------------------------------------------------------------------------

## 11. 데이터 선택: iloc과 loc

### iloc

정수 위치(position)를 기준으로 데이터를 선택한다.

``` python
customers.iloc[0]
```

### loc

index나 컬럼명 같은 레이블(label)을 기준으로 데이터를 선택한다.

``` python
customers.loc[:, ["name", "point"]]
```

  기능     선택 기준
  -------- --------------------------
  `iloc`   정수 위치
  `loc`    index / 컬럼명 등 레이블

------------------------------------------------------------------------

## 12. Boolean Indexing으로 조건 필터링

특정 조건을 만족하는 행만 선택할 때 Boolean Indexing을 사용할 수 있다.

``` python
vip_customers = customers[
    (customers["region"] == "서울")
    & (customers["point"] >= 3000)
]
```

여러 조건을 사용할 때 각 조건을 괄호로 감싸고 `&`를 사용하면 두 조건을
모두 만족하는 행을 선택할 수 있다.

------------------------------------------------------------------------

## 13. 오늘의 핵심 Pandas 코드

``` python
import pandas as pd

df = pd.read_csv("data.csv")

df.head()
df.tail()
df.shape
df.columns
df.dtypes
df.info()

df = df.rename(columns={"old_name": "new_name"})

series = df["column"]
df_one_col = df[["column"]]

df["price"] = df["price"].astype(int)
df["date"] = pd.to_datetime(df["date"])

df["price"].describe()
df["category"].value_counts()
df["customer_id"].nunique()

df.iloc[0]
df.loc[:, ["name", "point"]]

filtered = df[(df["region"] == "서울") & (df["point"] >= 3000)]

df.to_csv("output.csv", index=False)
```

------------------------------------------------------------------------

## 14. 오늘 배운 내용 정리

-   **데이터 전처리**: 원시 데이터를 분석에 적합한 상태로 정리하는 과정
-   **Pandas**: Python의 데이터 조작 및 분석 라이브러리
-   **DataFrame**: 행과 열로 이루어진 2차원 자료구조
-   **Series**: 값과 index로 이루어진 1차원 자료구조
-   `df["col"]`은 Series, `df[["col"]]`은 DataFrame을 반환한다.
-   `read_csv()`, `read_excel()`로 파일을 불러오고 `to_csv()`,
    `to_excel()`로 저장한다.
-   `head()`, `tail()`, `shape`, `columns`, `dtypes`, `info()`로
    데이터의 기본 구조를 확인한다.
-   `astype()`과 `pd.to_datetime()`으로 분석 목적에 맞게 타입을
    변환한다.
-   `describe()`는 기본 통계량, `value_counts()`는 값별 빈도,
    `nunique()`는 고유 값의 개수를 확인한다.
-   `iloc`은 정수 위치, `loc`은 레이블을 기준으로 데이터를 선택한다.
-   Boolean Indexing으로 원하는 조건을 만족하는 행만 필터링할 수 있다.

------------------------------------------------------------------------

## 15. 느낀 점

오늘은 데이터 분석에서 가장 기본적으로 사용하는 Pandas와 데이터를 처음
받았을 때 어떤 순서로 확인해야 하는지를 배웠다.

특히 데이터를 받자마자 분석을 시작하는 것이 아니라 `head()`, `shape`,
`dtypes`, `info()` 등을 이용해 **데이터의 구조와 상태부터 파악해야
한다**는 점이 중요하게 느껴졌다. 데이터가 어떤 형태인지 모르는 상태에서
분석하면 잘못된 데이터 타입이나 결측값 때문에 결과가 달라질 수 있기
때문이다.

또한 `df["컬럼명"]`은 Series가 되고 `df[["컬럼명"]]`은 DataFrame을
유지한다는 차이를 배웠다. 비슷해 보이는 코드지만 반환되는 자료구조가
다르기 때문에 앞으로 Pandas를 사용할 때 주의해야겠다.

`describe()`, `value_counts()`, `nunique()`를 이용하면 데이터를 일일이
확인하지 않고도 전체적인 분포와 특징을 빠르게 파악할 수 있다는 점도 알게
되었다. 특히 주문 건수와 실제 고객 수는 다를 수 있으므로 `count()`와
`nunique()`의 차이를 구분해서 사용해야겠다.

앞으로 데이터를 처음 받으면 다음 순서로 확인하는 습관을 만들어야겠다.

> **파일 불러오기 → 데이터 미리보기 → 구조 확인 → 데이터 타입 확인 및
> 변환 → 분포 확인 → 필요한 데이터 선택 및 필터링**

Pandas 함수 자체를 외우는 것보다 각 함수가 데이터 분석 과정에서 **왜
필요한지**를 생각하면서 익혀야겠다.