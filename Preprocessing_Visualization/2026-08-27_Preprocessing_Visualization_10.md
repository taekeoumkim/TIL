# 2026-08-27 관계·비교·추세 데이터 시각화

## 학습 목표

- 산점도에서 두 수치형 변수의 관계 방향과 강도를 해석할 수 있다.
- 피어슨 상관계수와 상관관계 히트맵을 사용해 여러 수치형 변수의 관계를 확인할 수 있다.
- 상관관계와 인과관계의 차이를 설명할 수 있다.
- 막대그래프와 히스토그램의 차이를 구분할 수 있다.
- 비교 목적에 따라 수직·수평 막대그래프와 그룹 막대그래프를 선택할 수 있다.
- 선 그래프로 시간에 따른 변화를 표현하고 이동평균으로 단기 변동을 완화할 수 있다.

---

## 1. 시각화 목적에 따른 차트 선택

데이터 시각화에서는 익숙한 그래프를 바로 선택하기보다 **무엇을 확인하려는지** 먼저 생각해야 한다.

| 확인하려는 내용 | 적합한 차트 |
|---|---|
| 두 수치형 변수 사이의 관계 | 산점도 |
| 여러 수치형 변수 사이의 상관관계 | 상관관계 히트맵 |
| 서로 다른 범주의 값 비교 | 막대그래프 |
| 두 범주형 변수를 함께 비교 | 그룹 막대그래프 |
| 시간에 따른 값의 변화 | 선 그래프 |
| 단기적인 변동을 줄인 전체 흐름 | 이동평균 선 그래프 |

이번 학습에서는 관계, 비교, 추세라는 세 가지 목적에 맞는 차트를 배웠다.

---

## 2. 산점도

산점도(Scatter Plot)는 두 수치형 변수의 값을 각각 X축과 Y축에 배치하고, 하나의 관측값을 하나의 점으로 표시하는 차트이다.

점들이 퍼진 모양을 통해 두 변수의 관계 방향과 강도를 직관적으로 확인할 수 있다.

```python
import matplotlib.pyplot as plt

ad_cost = [10, 20, 30, 40, 50, 60]
sales = [120, 150, 180, 210, 250, 270]

plt.scatter(ad_cost, sales)
plt.xlabel("광고비(만원)")
plt.ylabel("매출(만원)")
plt.title("광고비와 매출의 관계")
plt.show()
```

예를 들어 `(30, 180)`이라는 점은 광고비가 30만 원일 때 매출이 180만 원이었다는 뜻이다. 점들이 전체적으로 왼쪽 아래에서 오른쪽 위로 분포하므로 광고비와 매출 사이에 양의 관계가 있을 가능성을 확인할 수 있다.

### 산점도 패턴 해석

| 점의 분포 | 해석 |
|---|---|
| 왼쪽 아래 → 오른쪽 위 | X가 증가할수록 Y도 증가하는 양의 관계 |
| 왼쪽 위 → 오른쪽 아래 | X가 증가할수록 Y는 감소하는 음의 관계 |
| 일정한 방향 없이 흩어짐 | 뚜렷한 선형 관계가 거의 없음 |
| 점들이 직선 주변에 좁게 모임 | 강한 선형 관계 |
| 점들이 넓게 퍼짐 | 약한 선형 관계 |

산점도에서는 관계의 방향뿐 아니라 비선형 패턴, 집단의 구분, 다른 점들과 멀리 떨어진 이상치도 함께 살펴봐야 한다.

### 결측치를 제거한 산점도

두 열 중 하나라도 결측치가 있으면 해당 행을 제외한 후 산점도를 그릴 수 있다.

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv("ecommerce_orders_online.csv")
scatter_df = df[["unit_price", "revenue"]].dropna()

plt.scatter(
    scatter_df["unit_price"],
    scatter_df["revenue"],
    alpha=0.6
)
plt.xlabel("Unit Price")
plt.ylabel("Revenue")
plt.title("Unit Price and Revenue")
plt.show()
```

`alpha`는 점의 투명도를 조절한다. 데이터가 많아 점이 겹칠 때 1보다 작은 값을 사용하면 밀집된 영역을 확인하기 쉬워진다.

---

## 3. 피어슨 상관계수

산점도로 두 변수의 관계를 눈으로 확인할 수 있지만, 관계의 방향과 강도를 숫자로 표현하려면 피어슨 상관계수(Pearson correlation coefficient)를 사용할 수 있다.

피어슨 상관계수 `r`은 두 변수의 **선형 관계**를 -1부터 +1 사이의 값으로 나타낸다.

```python
import pandas as pd

df = pd.DataFrame({
    "distance": [1, 2, 3, 4, 5, 6],
    "delivery_time": [12, 17, 21, 28, 32, 39]
})

correlation = df["distance"].corr(df["delivery_time"])
print(correlation)
```

결과가 `+1`에 가깝다면 배달 거리가 증가할수록 배달 시간도 증가하는 강한 양의 선형 관계가 있다는 의미이다.

### 상관계수 해석

| 값의 범위 | 일반적인 해석 |
|---|---|
| `r = +1` | 완전한 양의 선형 관계 |
| `0.7 ≤ r < 1` | 강한 양의 상관관계 |
| `0.3 ≤ r < 0.7` | 중간 정도의 양의 상관관계 |
| `-0.3 < r < 0.3` | 약한 선형 관계 또는 거의 없음 |
| `-0.7 < r ≤ -0.3` | 중간 정도의 음의 상관관계 |
| `-1 < r ≤ -0.7` | 강한 음의 상관관계 |
| `r = -1` | 완전한 음의 선형 관계 |

상관계수의 부호는 관계의 **방향**, 절댓값은 선형 관계의 **강도**를 나타낸다. 다만 강도 구간은 절대적인 규칙이 아니므로 분석 분야와 데이터 특성을 함께 고려해야 한다.

### 표준편차, 공분산과 상관계수

- 표준편차: 값들이 평균으로부터 얼마나 퍼져 있는지를 나타낸다.
- 공분산: 두 변수가 함께 변하는 방향과 정도를 나타내지만 단위의 영향을 받는다.
- 피어슨 상관계수: 공분산을 두 변수의 표준편차 곱으로 나누어 -1부터 +1 사이로 표준화한 값이다.

### 피어슨 상관계수의 한계

피어슨 상관계수는 선형 관계를 측정한다. 두 변수 사이에 뚜렷한 곡선 형태의 관계가 있어도 상관계수가 0에 가깝게 나올 수 있으므로, 숫자만 확인하지 말고 산점도도 함께 살펴보는 것이 좋다.

---

## 4. 상관관계 히트맵

변수가 많으면 모든 변수 조합의 산점도를 하나씩 확인하기 어렵다. 이때 `DataFrame.corr()`로 상관계수 행렬을 계산하고 `sns.heatmap()`으로 시각화하면 여러 수치형 변수의 관계를 한눈에 비교할 수 있다.

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

df = pd.DataFrame({
    "visit_time": [10, 20, 30, 40, 50],
    "purchase_amount": [20, 35, 55, 70, 90],
    "purchase_count": [1, 2, 3, 4, 5]
})

corr = df.corr()

sns.heatmap(
    corr,
    annot=True,
    fmt=".2f",
    cmap="coolwarm",
    vmin=-1,
    vmax=1,
    center=0
)
plt.title("고객 행동 상관관계 히트맵")
plt.show()
```

### 주요 옵션

| 옵션 | 의미 |
|---|---|
| `annot=True` | 각 칸에 상관계수 표시 |
| `fmt=".2f"` | 숫자를 소수점 둘째 자리까지 표시 |
| `cmap="coolwarm"` | 음수와 양수를 서로 다른 색상으로 표현 |
| `vmin=-1`, `vmax=1` | 색상 범위를 -1부터 +1로 고정 |
| `center=0` | 0을 색상표의 중심으로 설정 |

히트맵의 대각선은 각 변수를 자기 자신과 비교한 값이므로 항상 1이다. 또한 행과 열이 바뀌어도 같은 변수 쌍을 비교하므로 대각선을 기준으로 대칭 구조가 나타난다.

히트맵은 강한 관계가 의심되는 변수 조합을 빠르게 찾는 도구이다. 구체적인 분포와 이상치를 확인하려면 해당 변수 조합의 산점도를 추가로 그려야 한다.

---

## 5. 상관관계와 인과관계

상관관계가 높다는 것은 두 변수가 함께 변하는 경향이 있다는 의미이지, 한 변수가 다른 변수의 원인이라는 의미는 아니다.

예를 들어 배달 거리와 배달 시간의 상관계수가 높더라도 배달 시간에는 교통량, 날씨, 주문량과 같은 다른 변수가 함께 영향을 줄 수 있다.

```text
상관관계: 두 변수가 함께 변하는 경향이 있음
인과관계: 한 변수의 변화가 다른 변수의 변화를 발생시킴
```

인과관계를 주장하려면 시간적 선후관계, 다른 요인의 통제, 실험 또는 적절한 인과추론 방법 등 추가적인 근거가 필요하다.

---

## 6. 막대그래프

막대그래프(Bar Plot)는 서로 다른 범주의 값 크기를 막대의 길이로 비교하는 차트이다.

X축에는 메뉴, 지역, 상품 카테고리와 같은 범주를 배치하고 Y축에는 주문량, 매출과 같은 비교할 값을 배치한다.

```python
import matplotlib.pyplot as plt

menus = ["아메리카노", "카페라떼", "바닐라라떼", "에이드"]
orders = [80, 55, 40, 25]

plt.bar(menus, orders)
plt.xlabel("메뉴")
plt.ylabel("주문량")
plt.title("메뉴별 주문량")
plt.show()
```

각 막대의 높이를 비교하면 아메리카노의 주문량이 가장 많다는 사실을 빠르게 확인할 수 있다.

### 막대그래프와 히스토그램의 차이

| 구분 | 막대그래프 | 히스토그램 |
|---|---|---|
| 목적 | 범주 간 값 비교 | 연속형 수치 데이터의 분포 확인 |
| X축 | 서로 구분되는 범주 | 연속된 수치 구간 |
| 막대 순서 | 분석 목적에 따라 변경 가능 | 수치 구간 순서를 유지해야 함 |
| 막대 사이 | 일반적으로 간격이 있음 | 연속 구간이므로 일반적으로 붙어 있음 |

두 그래프가 모두 막대를 사용하지만 데이터의 성격과 사용 목적이 다르다.

### 범주별 값 집계

원본 데이터에서 바로 막대그래프를 그리기보다 분석 목적에 맞게 먼저 집계하는 경우가 많다.

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv("ecommerce_sales_analytics_5000.csv")

category_sales = (
    df.groupby("category")["revenue"]
      .sum()
      .sort_values(ascending=False)
)

plt.bar(category_sales.index, category_sales.values)
plt.xlabel("Category")
plt.ylabel("Revenue")
plt.title("Revenue by Category")
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

`groupby()`로 범주별 매출 합계를 계산하고 `sort_values()`로 큰 값부터 정렬하면 비교가 쉬워진다.

---

## 7. 수직·수평 막대 선택과 정렬

### 수직 막대그래프

범주의 이름이 짧고 범주 수가 많지 않을 때 사용하기 좋다.

```python
plt.bar(categories, values)
```

### 수평 막대그래프

범주의 이름이 길거나 범주 수가 많아 X축 라벨이 겹칠 때 `plt.barh()`를 사용하면 읽기 편하다.

```python
inquiry_types = [
    "상품 배송 일정 문의",
    "결제 취소 및 환불 문의",
    "회원 정보 변경 문의"
]
counts = [45, 32, 18]

sorted_data = sorted(zip(counts, inquiry_types))
sorted_counts = [item[0] for item in sorted_data]
sorted_types = [item[1] for item in sorted_data]

plt.barh(sorted_types, sorted_counts)
plt.xlabel("문의 수")
plt.title("문의 유형별 건수")
plt.tight_layout()
plt.show()
```

수평 막대그래프는 값을 오름차순으로 정렬하면 가장 큰 막대가 위쪽에 나타나 비교하기 쉽다. 반대로 순위 자체를 강조하는 표나 수직 막대그래프에서는 큰 값부터 내림차순으로 배치하는 경우가 많다.

시간이나 등급처럼 자연스러운 순서가 있는 범주는 단순히 값의 크기로 재정렬하지 않고 원래 순서를 유지해야 한다.

---

## 8. 그룹 막대그래프

단순 막대그래프는 하나의 범주별 값을 비교하는 데 적합하다. 하나의 범주 안에서 또 다른 그룹을 함께 비교하려면 그룹 막대그래프를 사용한다.

예를 들어 회원 등급별 주문량을 모바일과 PC로 나누어 비교할 수 있다.

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

orders = pd.DataFrame({
    "grade": ["Silver", "Silver", "Gold", "Gold", "VIP", "VIP"],
    "device": ["Mobile", "PC", "Mobile", "PC", "Mobile", "PC"],
    "order_count": [80, 50, 110, 70, 140, 95]
})

sns.barplot(
    data=orders,
    x="grade",
    y="order_count",
    hue="device",
    errorbar=None
)
plt.xlabel("회원 등급")
plt.ylabel("주문량")
plt.title("회원 등급 및 기기별 주문량")
plt.show()
```

`hue="device"`는 각 회원 등급 안의 데이터를 기기별 색상으로 나누어 표시한다. 이를 통해 회원 등급 간 차이와 같은 등급 안의 기기별 차이를 동시에 비교할 수 있다.

### `sns.barplot()`의 에러바

원본 관측값을 `sns.barplot()`에 전달하면 기본적으로 각 그룹의 평균을 막대로 나타내며, 에러바는 평균 추정의 불확실성을 표현한다. 이미 그룹별 합계나 건수로 집계한 데이터를 전달한다면 `errorbar=None`으로 에러바를 생략할 수 있다.

### 그룹이 너무 많을 때

그룹이 많아지면 색상과 막대 수가 증가하여 차트를 읽기 어려워진다.

- 핵심 그룹만 선택한다.
- 차트를 여러 개의 작은 그래프로 나눈다.
- 전체 크기와 구성비를 함께 보여주고 싶다면 누적 막대그래프를 고려한다.

그룹 막대그래프는 그룹 간 정확한 크기 비교에 유리하고, 누적 막대그래프는 전체와 구성비를 함께 보는 데 유리하다.

---

## 9. 선 그래프

선 그래프(Line Plot)는 순서가 있는 데이터 포인트를 선으로 연결하여 연속적인 변화를 표현하는 차트이다. 특히 날짜, 시간, 월처럼 시간 순서에 따른 추세를 확인할 때 적합하다.

```python
import matplotlib.pyplot as plt

months = [1, 2, 3, 4, 5, 6]
sales = [100, 120, 115, 145, 160, 180]

plt.plot(months, sales, marker="o")
plt.xlabel("월")
plt.ylabel("매출")
plt.title("월별 매출 추세")
plt.show()
```

점이 선으로 연결되기 때문에 값이 증가하는지, 감소하는지 또는 반복되는 패턴이 있는지 확인하기 쉽다.

### 날짜형 변환과 정렬

날짜가 문자열이면 시간 순서가 잘못될 수 있으므로 `pd.to_datetime()`으로 날짜형으로 변환하고 정렬해야 한다.

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv("ecommerce_orders_online.csv")
df["order_date"] = pd.to_datetime(df["order_date"])

daily_revenue = (
    df.groupby("order_date")["revenue"]
      .sum()
      .sort_index()
)

plt.plot(daily_revenue.index, daily_revenue.values)
plt.xlabel("Date")
plt.ylabel("Daily Revenue")
plt.title("Daily Online Revenue")
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

분석 흐름은 다음과 같다.

```text
날짜형 변환
    ↓
날짜별 집계
    ↓
시간 순서로 정렬
    ↓
선 그래프로 시각화
```

---

## 10. 여러 선 비교와 차트 구성 요소

여러 집단의 추세를 비교하려면 하나의 차트에 여러 선을 그릴 수 있다. 이때 각 선의 의미를 구분할 수 있도록 축 이름, 제목과 범례를 반드시 추가해야 한다.

```python
import matplotlib.pyplot as plt

months = [1, 2, 3, 4, 5, 6]
seoul_orders = [120, 135, 150, 165, 180, 200]
busan_orders = [100, 115, 125, 140, 155, 170]

fig, ax = plt.subplots(figsize=(9, 5))

ax.plot(
    months,
    seoul_orders,
    marker="o",
    label="서울점"
)
ax.plot(
    months,
    busan_orders,
    marker="s",
    linestyle="--",
    label="부산점"
)

ax.set_xlabel("월")
ax.set_ylabel("주문량")
ax.set_title("매장별 월별 주문량 변화")
ax.legend(title="매장")
ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

### 선 그래프의 주요 옵션

| 옵션 | 예시 | 의미 |
|---|---|---|
| `color` | `"steelblue"`, `"red"` | 선 색상 |
| `linestyle` | `"-"`, `"--"`, `":"`, `"-."` | 선 모양 |
| `marker` | `"o"`, `"s"`, `"^"`, `"D"` | 데이터 포인트 모양 |
| `linewidth` | `1`, `2.5` | 선 두께 |
| `alpha` | `0.0`~`1.0` | 투명도 |
| `label` | `"서울점"` | 범례에 표시할 이름 |

`plt.plot()` 방식도 사용할 수 있지만 `fig, ax = plt.subplots()` 방식은 여러 그래프를 배치하거나 각 축을 세밀하게 조절할 때 유용하다.

---

## 11. 이동평균

시간 데이터는 하루마다 값이 크게 달라질 수 있어 원본 선만으로 전체 추세를 판단하기 어려울 수 있다. 이동평균(Rolling Average)은 현재 시점을 포함한 최근 N개의 평균을 연속적으로 계산하여 단기적인 변동을 줄인다.

```python
import pandas as pd
import matplotlib.pyplot as plt

days = pd.date_range("2026-01-01", periods=14)
orders = [
    100, 130, 90, 140, 110, 150, 120,
    160, 130, 170, 140, 180, 150, 190
]

df = pd.DataFrame({
    "date": days,
    "orders": orders
})

df["ma3"] = df["orders"].rolling(window=3).mean()

fig, ax = plt.subplots(figsize=(10, 5))
ax.plot(
    df["date"],
    df["orders"],
    alpha=0.4,
    label="일별 주문량"
)
ax.plot(
    df["date"],
    df["ma3"],
    linewidth=2,
    label="3일 이동평균"
)
ax.set_xlabel("날짜")
ax.set_ylabel("주문량")
ax.set_title("일별 주문량과 3일 이동평균")
ax.legend()
plt.tight_layout()
plt.show()
```

`rolling(window=3)`은 현재 값을 포함한 최근 3개의 값을 하나의 구간으로 묶는다. 처음 세 값이 100, 130, 90이라면 세 번째 날짜의 이동평균은 다음과 같다.

```text
(100 + 130 + 90) / 3 = 106.67
```

다음 날짜에서는 계산 구간이 한 칸 이동하여 130, 90, 140의 평균을 계산한다.

### 이동평균의 첫 값이 `NaN`인 이유

3일 이동평균을 계산하려면 세 개의 값이 필요하다. 첫 번째와 두 번째 날짜에는 데이터가 충분하지 않으므로 기본 설정에서는 `NaN`이 되고, 이동평균 선은 세 번째 날짜부터 표시된다.

### `window` 크기의 영향

- `window`가 작음: 실제 데이터 변화에 빠르게 반응하지만 단기 변동이 많이 남는다.
- `window`가 큼: 선이 더 부드러워져 장기 추세를 보기 쉽지만 변화에 늦게 반응한다.
- 지나치게 큰 `window`: 중요한 실제 변화까지 가릴 수 있다.

따라서 일별 데이터의 7일 이동평균처럼 데이터의 주기와 분석 목적을 고려해 크기를 정해야 한다.

---

## 12. 세 가지 시각화 목적 비교

| 목적 | 대표 질문 | 데이터 유형 | 대표 차트 | 핵심 해석 |
|---|---|---|---|---|
| 관계 | X와 Y가 함께 변하는가? | 수치형 + 수치형 | 산점도, 상관관계 히트맵 | 방향, 강도, 이상치 확인 |
| 비교 | 어떤 범주의 값이 더 큰가? | 범주형 + 수치형 | 막대, 그룹 막대 | 막대 길이와 범주별 차이 비교 |
| 추세 | 시간이 지나며 어떻게 변하는가? | 시간 + 수치형 | 선 그래프, 이동평균 | 증가·감소·반복 패턴 확인 |

차트는 서로 대체하는 것이 아니라 질문에 따라 선택한다. 예를 들어 날짜별 매출은 선 그래프로 추세를 확인하고, 지역별 총매출은 막대그래프로 비교하며, 광고비와 매출의 관계는 산점도로 확인하는 것이 적절하다.

---

## 13. 오늘 배운 내용 정리

### 관계 파악

- `plt.scatter()` → 두 수치형 변수의 관계를 점으로 표현
- `Series.corr()` → 두 변수의 피어슨 상관계수 계산
- `DataFrame.corr()` → 여러 수치형 변수의 상관계수 행렬 계산
- `sns.heatmap()` → 상관계수 행렬 시각화
- 상관계수는 선형 관계를 나타내며 인과관계를 증명하지 않음

### 범주 비교

- `plt.bar()` → 수직 막대그래프
- `plt.barh()` → 수평 막대그래프
- `sns.barplot(hue=...)` → 그룹 막대그래프
- 범주명이 길면 수평 막대가 읽기 편함
- 비교 목적에서는 값 기준 정렬이 유용하지만 자연스러운 순서가 있는 범주는 원래 순서를 유지
- 막대그래프는 범주 비교, 히스토그램은 연속형 데이터의 분포 확인에 사용

### 시간 추세

- `pd.to_datetime()` → 문자열을 날짜형으로 변환
- `groupby()` → 날짜별·월별 값 집계
- `sort_index()` → 시간 순서로 정렬
- `plt.plot()` 또는 `ax.plot()` → 시간에 따른 변화를 선으로 표현
- `rolling(window=N).mean()` → N개 데이터의 이동평균 계산
- 제목, 축 이름과 범례를 추가하여 그래프의 의미를 명확하게 전달

---

## 14. 느낀 점

오늘은 데이터에서 무엇을 알고 싶은지에 따라 차트를 선택하는 방법을 배웠다. 산점도는 두 수치형 변수의 관계, 막대그래프는 범주별 차이, 선 그래프는 시간에 따른 변화를 보여준다는 기준이 정리되었다.

특히 상관계수가 높더라도 한 변수가 다른 변수의 원인이라고 바로 결론 내릴 수 없다는 점이 중요하다고 느꼈다. 앞으로 상관계수를 확인할 때는 산점도를 함께 살펴보고, 숨겨진 다른 요인이 있을 가능성도 고려해야겠다.

막대그래프에서는 단순히 그래프를 그리는 것보다 범주 이름의 길이와 정렬 순서를 고려해야 비교가 쉬워진다는 점을 알게 되었다. 그룹 막대그래프도 많은 정보를 보여줄 수 있지만 그룹이 너무 많으면 오히려 복잡해질 수 있으므로 핵심 비교 대상만 선택해야겠다.

선 그래프를 그릴 때는 날짜형 변환과 시간 순서 정렬이 먼저 이루어져야 한다. 또한 원본 데이터의 변동이 심할 때 이동평균을 함께 표시하면 전체 흐름을 더 쉽게 파악할 수 있었다. 다만 `window`가 커질수록 그래프는 부드러워지지만 실제 변화에 늦게 반응할 수 있으므로 분석 목적에 맞는 값을 선택하는 연습이 필요하다고 느꼈다.