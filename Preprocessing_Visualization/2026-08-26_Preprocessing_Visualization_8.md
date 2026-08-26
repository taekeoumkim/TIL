# [2026-08-26] 중복 데이터 처리 - duplicated()와 drop_duplicates()

## 학습 목표

- 중복 데이터의 개념과 발생 원인을 이해한다.
- 중복 데이터가 분석 결과에 어떤 영향을 주는지 설명할 수 있다.
- `duplicated()`를 이용해 중복 데이터를 확인할 수 있다.
- `subset`을 이용해 특정 컬럼을 기준으로 중복을 판단할 수 있다.
- `keep` 파라미터에 따른 중복 판별 결과의 차이를 이해한다.
- `drop_duplicates()`를 이용해 중복 데이터를 제거할 수 있다.
- 중복 처리 전후의 행 수를 비교하여 처리 결과를 검증할 수 있다.

---

# 1. 중복 데이터란?

중복 데이터는 **같은 데이터가 두 번 이상 기록된 상태**이다.

실무에서 데이터를 수집하다 보면 동일한 행이 두 번 또는 세 번 이상 저장되는 경우가 발생할 수 있다.

중복 데이터를 그대로 두면 `count()`나 `sum()` 같은 집계 결과가 실제보다 크게 계산되어 분석 결과가 왜곡될 수 있다.

예를 들어 다음과 같은 주문 데이터가 있다고 하자.

```python
import pandas as pd

orders = pd.DataFrame({
    "order_id": [101, 102, 102, 103],
    "customer": ["민수", "지수", "지수", "서연"],
    "amount": [15000, 30000, 30000, 20000]
})
```

단순히 매출을 합하면

```python
orders["amount"].sum()
```

결과는

```text
95000
```

이 된다.

하지만 `order_id=102`가 시스템 오류로 두 번 저장된 것이라면 실제 매출은 65,000원이다.

즉,

```text
중복 데이터
→ 집계 결과 왜곡
→ 잘못된 분석 결과
```

로 이어질 수 있다.

---

# 2. 중복 데이터가 발생하는 원인

학습자료에서는 대표적인 발생 원인을 다음과 같이 설명했다.

### ETL 파이프라인 오류

데이터를 추출(Extract), 변환(Transform), 적재(Load)하는 과정에서 동일한 데이터가 두 번 이상 처리될 수 있다.

### 여러 데이터 소스 병합

온라인 주문 데이터와 오프라인 주문 데이터처럼 서로 다른 데이터 소스를 결합했을 때 동일한 레코드가 양쪽 데이터에 존재할 수 있다.

### 수동 입력 오류

담당자가 동일한 데이터를 실수로 두 번 입력할 수 있다.

### concat() / merge() 사용 과정

데이터를 결합할 때 적절한 기준 없이 `concat()`이나 `merge()`를 사용하면 의도하지 않은 중복 데이터가 생길 수 있다.

---

# 3. 중복 판단 기준이 중요하다

중복 데이터를 처리할 때 가장 먼저 생각해야 할 것은

> **무엇을 기준으로 같은 데이터라고 판단할 것인가?**

이다.

중복은 크게 두 가지 기준으로 판단할 수 있다.

```text
1. 전체 행 기준
2. 특정 컬럼 기준
```

## 전체 행 기준

모든 컬럼의 값이 완전히 같은 경우 중복으로 판단한다.

```python
df.duplicated()
```

## 특정 컬럼 기준

특정 컬럼의 값이 같으면 중복으로 판단한다.

예를 들어 주문을 식별하는 `order_id`를 기준으로 확인하려면

```python
df.duplicated(
    subset=["order_id"]
)
```

를 사용한다.

---

# 4. 전체 행은 달라도 특정 컬럼은 중복일 수 있다

예를 들어 다음 데이터가 있다고 하자.

```text
user_id   order_time
1         10:00
1         15:00
```

두 행은 `order_time`이 다르기 때문에 전체 행 기준으로는 서로 다른 데이터이다.

하지만 `user_id`만 기준으로 보면 같은 사용자가 두 번 등장한다.

따라서

```python
df.duplicated()
```

와

```python
df.duplicated(
    subset=["user_id"]
)
```

의 결과는 다를 수 있다.

중복 처리에서는 분석 목적에 맞게 `subset`을 설정하는 것이 중요하다.

---

# 5. 여러 데이터를 합친 뒤 중복 가능성 확인하기

서로 다른 데이터 소스를 결합했다면 전체 행 수와 고유한 ID 개수를 비교해 중복 가능성을 확인할 수 있다.

```python
online = pd.read_csv(
    "ecommerce_orders_online.csv"
)

offline = pd.read_csv(
    "ecommerce_orders_offline.csv"
)

combined = pd.concat(
    [online, offline]
)
```

전체 행 수:

```python
len(combined)
```

고유한 주문 ID 개수:

```python
combined["order_id"].nunique()
```

만약

```text
전체 행 수 > 고유한 order_id 개수
```

라면 동일한 `order_id`가 여러 번 존재할 가능성이 있다.

단, 이것만으로 실제 오류라고 확정하는 것이 아니라 데이터의 의미를 추가로 확인해야 한다.

---

# 6. duplicated()

`duplicated()`는 각 행이 이전에 등장한 데이터와 중복되는지 확인하는 Pandas 메서드이다.

결과는 Boolean 값인 `True`와 `False`로 반환된다.

```python
df.duplicated()
```

기본 동작은

```text
첫 번째 등장 → False
이후 중복 → True
```

이다.

예:

```python
products = pd.DataFrame({
    "product_id": [
        "P01", "P02", "P02", "P03", "P03"
    ],
    "name": [
        "노트북", "마우스", "마우스",
        "키보드", "키보드"
    ]
})

products["is_duplicate"] = products.duplicated(
    subset=["product_id"]
)
```

결과:

```text
product_id   is_duplicate
P01          False
P02          False
P02          True
P03          False
P03          True
```

---

# 7. duplicated()와 Boolean Indexing

`duplicated()`의 결과는 Boolean Series이기 때문에 Boolean Indexing에 사용할 수 있다.

```python
df[
    df.duplicated()
]
```

이렇게 하면 중복으로 판별된 행만 조회할 수 있다.

특정 컬럼을 기준으로 조회하려면

```python
df[
    df.duplicated(
        subset=["order_id"]
    )
]
```

처럼 작성한다.

중복 여부를 새로운 컬럼으로 저장할 수도 있다.

```python
df["is_duplicate"] = df.duplicated(
    subset=["order_id"]
)
```

그리고

```python
duplicate_rows = df[
    df["is_duplicate"]
]
```

처럼 사용할 수 있다.

---

# 8. subset 파라미터

`subset`은 **어떤 컬럼을 기준으로 중복을 판단할지 지정하는 파라미터**이다.

```python
df.duplicated(
    subset=["user_id"]
)
```

여러 컬럼을 함께 기준으로 사용할 수도 있다.

```python
df.duplicated(
    subset=["user_id", "product_id"]
)
```

이 경우 `user_id`와 `product_id`의 조합이 같은 행을 중복으로 판단한다.

`subset`을 사용하지 않으면 전체 컬럼을 기준으로 중복을 확인한다.

---

# 9. keep 파라미터

`duplicated()`에서는 `keep`을 이용해 중복된 데이터 중 어떤 행을 원본처럼 취급할지 결정할 수 있다.

| keep 값 | 동작 |
|---|---|
| `'first'` | 첫 번째 행은 남기고 이후 중복을 `True`로 표시 |
| `'last'` | 마지막 행은 남기고 이전 중복을 `True`로 표시 |
| `False` | 중복된 모든 행을 `True`로 표시 |

---

# 10. keep='first'

기본값이다.

```python
df.duplicated(
    subset=["order_id"],
    keep="first"
)
```

예를 들어 같은 `order_id`가 세 번 등장하면

```text
첫 번째 → False
두 번째 → True
세 번째 → True
```

가 된다.

즉, 첫 번째 데이터를 원본으로 본다.

---

# 11. keep='last'

마지막으로 등장한 데이터를 원본처럼 취급한다.

```python
df.duplicated(
    subset=["order_id"],
    keep="last"
)
```

예:

```text
첫 번째 → True
두 번째 → True
세 번째 → False
```

최신 데이터나 마지막 기록을 유지해야 하는 상황에서 활용할 수 있다.

---

# 12. keep=False

중복된 데이터 전체를 확인하고 싶다면 `keep=False`를 사용한다.

```python
df.duplicated(
    subset=["order_id"],
    keep=False
)
```

예:

```text
첫 번째 → True
두 번째 → True
세 번째 → True
```

즉, 중복 그룹에 포함된 모든 행이 `True`가 된다.

---

# 13. drop_duplicates()

`drop_duplicates()`는 중복 행을 제거하고 고유한 행만 남기는 Pandas 메서드이다.

기본 사용법:

```python
df_clean = df.drop_duplicates()
```

일반적으로는

```text
duplicated()
→ 중복 확인

drop_duplicates()
→ 중복 제거
```

의 흐름으로 사용한다.

중복을 확인하지 않고 바로 제거하면 예상하지 못한 데이터까지 삭제할 수 있기 때문이다.

---

# 14. 특정 컬럼을 기준으로 중복 제거

예를 들어 고객에게 이벤트 쿠폰을 한 번씩만 지급해야 하고 동일한 `customer_id`가 여러 번 등록되어 있다면 고객 ID를 기준으로 중복을 제거할 수 있다.

```python
clean_customers = customers.drop_duplicates(
    subset=["customer_id"]
)
```

기본값은

```python
keep="first"
```

이므로 동일한 고객 ID가 여러 번 등장하면 첫 번째 행만 남는다.

---

# 15. 마지막 데이터 남기기

가장 마지막에 기록된 데이터를 유지하려면

```python
df_clean = df.drop_duplicates(
    subset=["order_id"],
    keep="last"
)
```

를 사용할 수 있다.

예를 들어 동일한 주문이 여러 번 저장되었고 마지막 기록을 최종 데이터로 판단해야 하는 상황에서 사용할 수 있다.

---

# 16. drop_duplicates() 주요 파라미터

| 파라미터 | 설명 | 기본값 |
|---|---|---|
| `subset` | 중복 판단에 사용할 컬럼 | 전체 컬럼 |
| `keep` | 어떤 행을 유지할지 결정 | `'first'` |
| `inplace` | 원본 DataFrame에 직접 적용할지 여부 | `False` |

예:

```python
df.drop_duplicates(
    subset=["user_id"],
    keep="last"
)
```

원본 DataFrame에 직접 적용하려면

```python
df.drop_duplicates(
    inplace=True
)
```

를 사용할 수 있다.

---

# 17. subset을 지정하지 않으면?

```python
df.drop_duplicates()
```

처럼 `subset`을 지정하지 않으면 **모든 컬럼의 값이 완전히 동일한 행**을 중복으로 판단하여 제거한다.

따라서 특정 컬럼만 중복 기준으로 사용하려면 반드시

```python
subset=["컬럼명"]
```

을 지정해야 한다.

예:

```python
df.drop_duplicates(
    subset=["order_id"]
)
```

---

# 18. duplicated()와 drop_duplicates() 차이

```text
duplicated()
→ 중복인지 확인

drop_duplicates()
→ 중복을 실제로 제거
```

예:

```python
# 1. 중복 확인
duplicate_mask = df.duplicated(
    subset=["order_id"]
)

# 2. 중복 행 확인
duplicate_rows = df[
    duplicate_mask
]

# 3. 중복 제거
df_clean = df.drop_duplicates(
    subset=["order_id"]
)
```

이 순서로 작업하면 중복 데이터를 확인한 뒤 안전하게 제거할 수 있다.

---

# 19. 중복 제거 후 검증이 중요하다

중복 데이터를 제거했다고 해서 작업이 끝난 것은 아니다.

처리 전후의 행 수를 비교해 실제로 몇 개의 데이터가 제거되었는지 확인해야 한다.

```python
before_count = len(orders)

clean_orders = orders.drop_duplicates(
    subset=["order_id"]
)

after_count = len(clean_orders)

removed_count = (
    before_count - after_count
)
```

결과를 출력한다.

```python
print(
    f"처리 전 행 수: {before_count}"
)

print(
    f"처리 후 행 수: {after_count}"
)

print(
    f"제거된 중복 행 수: {removed_count}"
)
```

학습자료의 예제에서는

```text
처리 전 행 수: 6
처리 후 행 수: 4
제거된 중복 행 수: 2
```

가 나왔다.

이렇게 처리 전후의 행 수를 비교하면 예상한 만큼 데이터가 제거되었는지 검증할 수 있다.

---

# 20. 중복 처리의 전체 흐름

오늘 배운 내용을 하나의 흐름으로 정리하면 다음과 같다.

```text
데이터 확인
    ↓
중복 판단 기준 결정
    ↓
duplicated()로 중복 확인
    ↓
중복 데이터 직접 확인
    ↓
제거해도 되는 데이터인지 판단
    ↓
drop_duplicates() 적용
    ↓
처리 전후 행 수 비교
    ↓
결과 검증
```

중요한 것은 단순히 중복을 제거하는 것이 아니라 **어떤 기준으로 중복이라고 판단했는지 설명할 수 있어야 한다는 것**이다.

---

# 21. 오늘의 핵심 코드

```python
import pandas as pd

df = pd.read_csv("data.csv")

# 전체 행 기준 중복 확인
df.duplicated()

# 중복 개수
df.duplicated().sum()

# 특정 컬럼 기준 중복 확인
df.duplicated(
    subset=["order_id"]
)

# 중복 행 조회
df[
    df.duplicated(
        subset=["order_id"]
    )
]

# 모든 중복 데이터 확인
df[
    df.duplicated(
        subset=["order_id"],
        keep=False
    )
]

# 첫 번째 데이터 유지
df_clean = df.drop_duplicates(
    subset=["order_id"],
    keep="first"
)

# 마지막 데이터 유지
df_clean = df.drop_duplicates(
    subset=["order_id"],
    keep="last"
)

# 전체 행 기준 중복 제거
df_clean = df.drop_duplicates()

# 처리 전후 행 수 검증
before_count = len(df)

df_clean = df.drop_duplicates(
    subset=["order_id"]
)

after_count = len(df_clean)

removed_count = (
    before_count - after_count
)

print(before_count)
print(after_count)
print(removed_count)
```

---

# 22. 오늘 배운 내용 정리

### 중복 데이터

```text
동일한 데이터가
두 번 이상 기록된 상태
```

발생 원인:

```text
ETL 파이프라인 오류
데이터 소스 병합
수동 입력 오류
concat() / merge() 과정의 오류
```

중복 데이터가 있으면

```text
count()
sum()
```

등의 집계 결과가 실제보다 부풀려질 수 있다.

---

### duplicated()

```python
df.duplicated()
```

중복 여부를 `True` / `False`로 반환한다.

```python
df.duplicated(
    subset=["order_id"]
)
```

특정 컬럼 기준으로 확인한다.

---

### keep

```text
keep='first'
→ 첫 번째 유지

keep='last'
→ 마지막 유지

keep=False
→ 중복 그룹 전체 표시
```

---

### drop_duplicates()

```python
df.drop_duplicates()
```

중복 데이터를 제거한다.

특정 컬럼 기준:

```python
df.drop_duplicates(
    subset=["order_id"]
)
```

---

### 가장 중요한 점

```text
중복 발견
→ 바로 삭제 X

중복 기준 결정
→ 실제 중복인지 확인
→ 제거
→ 처리 결과 검증
```

---

# 23. 오늘의 핵심 한 줄

> **중복 데이터 처리는 단순히 같은 값을 삭제하는 것이 아니라, 분석 목적에 맞는 중복 기준을 정하고 `duplicated()`로 확인한 뒤 `drop_duplicates()`로 처리하고 결과까지 검증하는 과정이다.**

---

# 24. 느낀 점

오늘은 중복 데이터가 단순히 같은 행이 여러 번 존재하는 문제를 넘어서 분석 결과 자체를 왜곡할 수 있다는 점을 배웠다.

특히 같은 주문이 두 번 저장되면 `sum()`으로 계산한 매출이 실제보다 크게 나오고, `count()` 역시 실제 건수보다 많게 계산될 수 있기 때문에 분석 전에 중복 여부를 확인하는 것이 중요하다는 것을 알게 되었다.

가장 기억에 남는 부분은 **중복의 기준이 하나로 정해져 있지 않다는 점**이다. 전체 행이 완전히 같아야 중복으로 볼 수도 있지만, 주문 데이터에서는 `order_id`, 고객 데이터에서는 `customer_id`처럼 특정 식별자를 기준으로 중복을 판단해야 할 수도 있다.

따라서 단순히

```python
df.duplicated()
```

만 사용하는 것이 아니라 분석 목적에 따라

```python
df.duplicated(
    subset=["order_id"]
)
```

처럼 기준을 명확하게 지정해야 한다.

또한 `keep='first'`, `keep='last'`, `keep=False`에 따라 어떤 데이터를 중복으로 표시할지 달라진다는 것도 배웠다. 특히 최신 데이터를 남겨야 하는 경우에는 무조건 첫 번째 행을 유지하는 것이 아니라 `keep='last'`가 더 적절할 수 있다는 점을 기억해야겠다.

앞으로 중복 데이터를 처리할 때는 다음 순서를 습관화해야겠다.

```text
중복 기준 결정
→ duplicated()로 확인
→ 중복 행 직접 확인
→ drop_duplicates()로 제거
→ 처리 전후 행 수 비교
```

그리고 중복 제거가 끝났다고 바로 분석으로 넘어가지 않고, 처리 전후 행 수를 비교하여 **의도한 만큼 데이터가 제거되었는지 검증하는 과정**까지 반드시 수행해야겠다.
