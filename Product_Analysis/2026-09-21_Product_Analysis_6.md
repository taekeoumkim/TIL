# 2026-09-21 이벤트 로그 기반 사용자 행동 분석

## 학습 목표

- 이벤트 로그를 이용한 사용자 행동 분석의 전체 흐름을 설명할 수 있다.
- 문제를 분석 질문으로 구체화하고 필요한 이벤트와 측정 기준을 정의할 수 있다.
- AARRR을 이용해 서비스 전체의 사용자 행동을 진단할 수 있다.
- 퍼널 분석으로 목표 행동까지의 전환율과 이탈률을 계산할 수 있다.
- 코호트 기반 리텐션 분석으로 시간에 따른 유지 패턴을 비교할 수 있다.
- 세그멘테이션으로 사용자 집단별 행동 차이를 확인할 수 있다.
- 분석에서 관찰한 현상과 원인에 대한 가설을 구분할 수 있다.

---

## 1. 이벤트 로그 기반 사용자 행동 분석의 흐름

이벤트 로그에는 사용자가 서비스에서 수행한 방문, 상품 조회, 장바구니 추가, 결제 시작, 구매, 공유 등의 행동이 기록된다. 이 데이터를 이용하면 서비스 전체의 행동부터 특정 구간과 사용자 집단의 문제까지 단계적으로 구체화할 수 있다.

이번 분석의 전체 흐름은 다음과 같다.

~~~text
문제 정의
    ↓
AARRR을 이용한 서비스 전체 행동 진단
    ↓
추가 분석 질문 설정
    ↓
퍼널을 이용한 세부 행동 분석
    ↓
리텐션·코호트를 이용한 시간 변화 분석
    ↓
세그멘테이션을 이용한 사용자 집단 비교
    ↓
시각화 및 결과 해석
    ↓
문제 구체화
    ↓
개선 가설 설정
~~~

중요한 것은 분석 기법 자체를 적용하는 데 그치지 않고, 하나의 분석 결과를 다음 질문으로 연결하는 것이다.

> 분석 질문 설정 → 데이터 집계 → 지표 계산 → 시각화 → 결과 해석 → 다음 분석 질문

---

## 2. 분석 질문과 기준 설정하기

온라인 쇼핑몰 운영팀에서 “사용자는 꾸준히 유입되지만 실제 구매와 재방문으로 충분히 이어지는지 확인하고 개선할 부분을 찾아달라”고 요청했다고 가정한다.

바로 그래프를 그리거나 지표를 계산하기보다 요청을 다음과 같은 분석 질문으로 구체화해야 한다.

~~~text
서비스 전체에서 어느 사용자 행동 영역을 살펴봐야 하는가?
    ↓
해당 영역의 구체적인 어느 행동에서 문제가 나타나는가?
    ↓
이러한 현상이 시간에 따라서도 나타나는가?
    ↓
특정 사용자 집단에서 더 크게 나타나는가?
~~~

각 질문은 서로 다른 분석 방법과 연결된다.

| 분석 질문 | 분석 방법 | 확인할 지표 |
|---|---|---|
| 서비스 전체에서 어떤 행동을 살펴봐야 하는가? | AARRR | 영역별 사용자 수·대표 비율 |
| 구매 과정의 어디에서 이탈하는가? | Funnel | 단계별 사용자 수·전환율·이탈률 |
| 시간이 지나도 다시 이용하는가? | Retention / Cohort | Day 1·7·14 리텐션율 |
| 어떤 사용자 집단에서 차이가 나타나는가? | Segmentation | 세그먼트별 구매율·리텐션율 |

분석에 들어가기 전에는 필요한 데이터가 준비되어 있는지도 점검해야 한다.

~~~python
analysis_events = [
    "visit",
    "view_product",
    "add_cart",
    "begin_checkout",
    "purchase",
    "share",
]

print(df["event_name"].value_counts().reindex(analysis_events))
print("시작:", df["event_time"].min())
print("종료:", df["event_time"].max())
print("유입 경로:", df["acquisition_channel"].unique())
print("이용 기기:", df["device"].unique())
~~~

이 코드는 결과를 계산하기 위한 것이 아니라 설정한 질문을 현재 데이터로 확인할 수 있는지 점검하는 과정이다.

~~~text
문제 정의
    ↓
분석 질문 설정
    ↓
필요한 사용자 행동 정의
    ↓
측정할 지표·시간·비교 기준 설정
    ↓
분석 방법 선택
    ↓
데이터 분석 및 해석
~~~

---

## 3. 분석 방법별 주요 관점

AARRR, Funnel, Retention, Cohort, Segmentation은 서로 경쟁하는 방법이 아니라 사용자 행동을 서로 다른 관점에서 살펴보기 위한 방법이다.

| 분석 방법 | 주요 관점 |
|---|---|
| AARRR | 서비스 전체에서 어떤 사용자 행동 영역을 살펴볼 것인가? |
| Funnel | 목표 행동까지 어느 구간에서 사용자가 이탈하는가? |
| Retention | 시간이 지나도 사용자가 다시 이용하는가? |
| Cohort | 비슷한 시기에 시작한 사용자들의 유지 패턴이 어떻게 다른가? |
| Segmentation | 어떤 사용자 집단에서 다른 행동이 나타나는가? |

모든 방법을 기계적으로 적용하기보다 현재 질문에 필요한 분석을 선택하고, 앞선 결과를 다음 질문으로 연결해야 한다.

---

## 4. AARRR로 서비스 전체 행동 분석하기

AARRR은 사용자 행동을 Acquisition, Activation, Retention, Revenue, Referral의 다섯 영역으로 나누어 서비스 전체를 살펴보는 프레임워크이다.

이번 쇼핑몰 분석에서는 각 영역을 다음과 같이 측정한다.

| AARRR | 대표 행동 | 확인할 지표 |
|---|---|---|
| Acquisition | 최초 서비스 방문 | 방문 사용자 수 |
| Activation | 상품 조회 | 상품 조회 사용자 수·비율 |
| Retention | Day 7 재방문 | Day 7 재방문 사용자 수·비율 |
| Revenue | 구매 완료 | 구매 사용자 수·비율 |
| Referral | 상품 또는 서비스 공유 | 공유 사용자 수·비율 |

### AARRR은 하나의 순차 퍼널이 아니다

Activation, Revenue, Referral은 전체 방문 사용자 중 해당 행동을 수행한 사용자 비율로 계산한다. Retention은 최초 방문 이후 7일이 지난 시점에 다시 방문한 사용자 비율로 계산한다.

따라서 AARRR의 각 영역을 반드시 이전 단계 대비 순차 전환율로 계산하는 것은 아니다.

~~~python
# 영역별 고유 사용자 수
acquisition_users = df.loc[df["event_name"] == "visit", "user_id"].nunique()
activation_users = df.loc[df["event_name"] == "view_product", "user_id"].nunique()
revenue_users = df.loc[df["event_name"] == "purchase", "user_id"].nunique()
referral_users = df.loc[df["event_name"] == "share", "user_id"].nunique()

# 사용자별 최초 방문일
first_visit = (
    df[df["event_name"] == "visit"]
    .groupby("user_id")["event_time"]
    .min()
    .dt.normalize()
)

visit_df = df[df["event_name"] == "visit"].copy()
visit_df["event_date"] = visit_df["event_time"].dt.normalize()
visit_df["first_visit_date"] = visit_df["user_id"].map(first_visit)
visit_df["days_after_first_visit"] = (
    visit_df["event_date"] - visit_df["first_visit_date"]
).dt.days

retention_users = visit_df.loc[
    visit_df["days_after_first_visit"] == 7, "user_id"
].nunique()
~~~

비율은 방문 사용자 수를 기준으로 계산한다.

~~~python
activation_rate = activation_users / acquisition_users * 100
retention_rate = retention_users / acquisition_users * 100
revenue_rate = revenue_users / acquisition_users * 100
referral_rate = referral_users / acquisition_users * 100
~~~

### AARRR 분석 결과

| AARRR | 사용자 수 | 비율 |
|---|---:|---:|
| Acquisition | 240명 | 100.0% |
| Activation | 194명 | 80.8% |
| Retention | 60명 | 25.0% |
| Revenue | 60명 | 25.0% |
| Referral | 17명 | 7.1% |

Revenue 비율은 다음과 같이 계산한다.

$$
\text{Revenue 비율}
= \frac{\text{구매 사용자 수}}{\text{방문 사용자 수}} \times 100
= \frac{60}{240} \times 100
= 25.0\%
$$

상품을 조회한 Activation 사용자는 비교적 많지만 구매 사용자는 전체 방문자의 25.0%이다. 그러나 “구매율이 25%이므로 Revenue가 문제다”라고 바로 결론 내릴 수는 없다.

AARRR의 지표는 측정하는 행동과 의미가 서로 다르다. Referral 7.1%가 Revenue 25.0%보다 낮더라도 Referral이 반드시 더 심각한 문제인 것은 아니다. 서비스 목표, 기존 성과와 목표값 등을 함께 고려해야 한다.

이번에는 구매 행동을 더 자세히 살펴보기 위해 다음 질문으로 연결한다.

> 구매하지 않은 사용자는 구매 과정의 어느 행동 구간에서 많이 이탈했을까?

---

## 5. 퍼널로 구매 과정의 이탈 구간 분석하기

전체 방문자 240명 중 60명이 구매해 구매율이 25.0%라는 사실만으로는 사용자가 구매 과정의 어디에서 이탈했는지 알 수 없다.

구매 완료까지의 행동을 다음과 같은 퍼널로 정의한다.

~~~text
서비스 방문 visit
    ↓
상품 조회 view_product
    ↓
장바구니 추가 add_cart
    ↓
결제 시작 begin_checkout
    ↓
구매 완료 purchase
~~~

각 단계에서는 이벤트 발생 횟수가 아니라 해당 행동을 한 번이라도 수행한 **고유 사용자 수**를 집계한다.

다만 실제 이벤트 로그에서는 행동 순서와 퍼널 진입 조건도 함께 고려해야 한다. 단순히 각 이벤트를 수행한 고유 사용자 수만 비교하면 실제 퍼널의 전환을 정확하게 나타내지 못할 수 있다.

### 전환율과 이탈률

$$
\text{전환율}
= \frac{\text{다음 단계 사용자 수}}{\text{이전 단계 사용자 수}} \times 100
$$

$$
\text{이탈률} = 100\% - \text{전환율}
$$

~~~python
funnel_events = [
    "visit",
    "view_product",
    "add_cart",
    "begin_checkout",
    "purchase",
]

funnel_users = (
    df[df["event_name"].isin(funnel_events)]
    .groupby("event_name")["user_id"]
    .nunique()
    .reindex(funnel_events)
)

funnel_summary = pd.DataFrame({
    "event_name": funnel_events,
    "users": funnel_users.values,
})

funnel_summary["conversion_rate"] = (
    funnel_summary["users"] / funnel_summary["users"].shift(1) * 100
)
funnel_summary.loc[0, "conversion_rate"] = 100
funnel_summary["dropoff_rate"] = 100 - funnel_summary["conversion_rate"]
~~~

### 퍼널 분석 결과

| 단계 | 사용자 수 | 이전 단계 대비 전환율 | 이탈률 |
|---|---:|---:|---:|
| visit | 240명 | 100.0% | 0.0% |
| view_product | 194명 | 80.8% | 19.2% |
| add_cart | 138명 | 71.1% | 28.9% |
| begin_checkout | 101명 | 73.2% | 26.8% |
| purchase | 60명 | 59.4% | 40.6% |

결제 시작 → 구매 완료 구간의 이탈률이 40.6%로 가장 높다. AARRR에서는 전체 방문자의 25.0%가 구매한다는 사실을 확인했고, 퍼널에서는 결제를 시작한 이후 구매 완료 전 구간에서 상대적으로 큰 이탈이 나타난다는 사실까지 문제를 구체화했다.

하지만 이 결과는 “결제 과정이 복잡해서 사용자가 이탈했다”는 원인을 의미하지 않는다. 이벤트 로그에서 확인한 것은 **어느 구간에서 이탈이 많이 발생했는지**이다.

~~~text
결제 시작 → 구매 완료 구간의 이탈률이 높다
    ↓
모든 사용자에게 비슷하게 나타나는가?
    ↓
특정 사용자 집단에서 더 크게 나타나는가?
~~~

---

## 6. 전체 리텐션 분석하기

퍼널은 구매 과정의 이탈 구간을 보여주지만, 사용자가 서비스를 이용한 뒤 시간이 지나도 다시 방문하는지는 알려주지 않는다.

이번 분석에서는 최초 방문일을 Day 0으로 설정하고, 최초 방문일로부터 정확히 1일, 7일, 14일 후 다시 방문했는지를 측정한다.

$$
\text{Day N 리텐션율}
= \frac{\text{Day N에 다시 방문한 사용자 수}}{\text{최초 사용자 수}} \times 100
$$

~~~python
total_users = df["user_id"].nunique()
retention_days = [1, 7, 14]
retention_users = []

for day in retention_days:
    users = visit_df.loc[
        visit_df["days_after_first_visit"] == day,
        "user_id",
    ].nunique()
    retention_users.append(users)

retention_summary = pd.DataFrame({
    "day": retention_days,
    "users": retention_users,
})

retention_summary["retention_rate"] = (
    retention_summary["users"] / total_users * 100
).round(1)
~~~

### 전체 리텐션 결과

| 시점 | 재방문 사용자 수 | 리텐션율 |
|---|---:|---:|
| Day 1 | 93명 | 38.8% |
| Day 7 | 60명 | 25.0% |
| Day 14 | 32명 | 13.3% |

시간이 지날수록 해당 시점에 다시 방문한 사용자의 비율이 감소한다. 리텐션 곡선으로 시각화하면 시간에 따른 유지 패턴을 쉽게 확인할 수 있다.

이번 실습의 Day 7 Retention은 Day 0 사용자 중 **정확히 7일 후** 다시 방문한 비율이다. 실제 분석에서는 서비스 특성과 목적에 따라 기준 행동, 확인 시점과 리텐션 계산 방식이 달라질 수 있다.

---

## 7. 코호트별 리텐션 비교하기

전체 리텐션만으로는 모든 시기에 가입한 사용자에게 같은 패턴이 나타나는지 알 수 없다. 사용자를 가입 시점이 비슷한 코호트로 나누면 집단별 유지 패턴을 비교할 수 있다.

이번에는 가입 주차별로 코호트를 구성하고 각 코호트의 Day 1, Day 7, Day 14 리텐션을 계산한다.

~~~python
user_cohort = (
    df[["user_id", "signup_date"]]
    .drop_duplicates("user_id")
    .copy()
)

user_cohort["cohort_week"] = (
    user_cohort["signup_date"]
    .dt.to_period("W-SUN")
    .apply(lambda x: x.start_time)
)

for day in [1, 7, 14]:
    retained_users = set(
        visit_df.loc[
            visit_df["days_after_first_visit"] == day,
            "user_id",
        ]
    )
    user_cohort[f"day_{day}"] = user_cohort["user_id"].isin(retained_users)

cohort_retention = (
    user_cohort
    .groupby("cohort_week")[["day_1", "day_7", "day_14"]]
    .mean()
    * 100
).round(1)
~~~

### 코호트별 리텐션 결과

| 가입 주차 코호트 | Day 1 | Day 7 | Day 14 |
|---|---:|---:|---:|
| 2026-08-03 | 35.0% | 30.0% | 13.3% |
| 2026-08-10 | 45.0% | 26.7% | 20.0% |
| 2026-08-17 | 35.0% | 21.7% | 10.0% |
| 2026-08-24 | 40.0% | 21.7% | 10.0% |

모든 코호트에서 시간이 지나면서 리텐션이 낮아지지만 코호트별 차이도 나타난다. 전체 Day 7 리텐션 25.0%만 볼 때보다 가입 시기에 따라 사용자 유지 패턴이 다르다는 사실을 확인할 수 있다.

그러나 “8월 17일 이후 서비스가 나빠져서 리텐션이 감소했다”고 바로 단정할 수 없다. 현재 확인한 것은 가입 시기에 따라 리텐션 차이가 관찰되었다는 사실이다.

원인을 확인하려면 유입 경로, 이용 기기, 서비스 변경 사항 등을 추가로 살펴봐야 한다.

~~~text
최초 이용 시점 정의
    ↓
재방문 기준 정의
    ↓
Day N 재방문 사용자 확인
    ↓
리텐션율 계산
    ↓
코호트 구성 및 비교
    ↓
시각화와 패턴 해석
    ↓
다음 분석 질문 설정
~~~

---

## 8. 세그멘테이션으로 사용자 집단 비교하기

**세그멘테이션(Segmentation)**은 전체 사용자를 하나의 집단으로 보지 않고 공통된 특성이나 행동을 가진 집단으로 나누어 비교하는 방법이다.

앞선 분석에서 다음 현상을 확인했다.

- 구매 퍼널에서 결제 시작 → 구매 완료 구간의 이탈률이 상대적으로 높았다.
- 가입 시기에 따라 Day 7 리텐션에 차이가 나타났다.

이제 질문을 다음과 같이 확장할 수 있다.

> 구매율과 Day 7 리텐션은 유입 경로나 이용 기기에 따라 다르게 나타나는가?

이번 데이터에서는 다음 두 기준을 사용한다.

| 세그멘테이션 기준 | 사용자 집단 |
|---|---|
| 유입 경로 | search / ad / referral |
| 이용 기기 | mobile / pc |

먼저 사용자별 구매 여부와 Day 7 재방문 여부를 만든다.

~~~python
user_segment = (
    df[["user_id", "acquisition_channel", "device"]]
    .drop_duplicates("user_id")
    .copy()
)

purchase_users = set(
    df.loc[df["event_name"] == "purchase", "user_id"]
)
day7_users = set(
    visit_df.loc[
        visit_df["days_after_first_visit"] == 7,
        "user_id",
    ]
)

user_segment["purchase"] = user_segment["user_id"].isin(purchase_users)
user_segment["day7_retention"] = user_segment["user_id"].isin(day7_users)
~~~

---

## 9. 유입 경로별 구매율과 리텐션

~~~python
channel_summary = (
    user_segment
    .groupby("acquisition_channel")
    .agg(
        users=("user_id", "nunique"),
        purchase_rate=("purchase", "mean"),
        day7_retention_rate=("day7_retention", "mean"),
    )
    .reset_index()
)

channel_summary["purchase_rate"] = (
    channel_summary["purchase_rate"] * 100
).round(1)
channel_summary["day7_retention_rate"] = (
    channel_summary["day7_retention_rate"] * 100
).round(1)
~~~

| 유입 경로 | 사용자 수 | 구매율 | Day 7 리텐션 |
|---|---:|---:|---:|
| ad | 84명 | 17.9% | 19.0% |
| referral | 52명 | 38.5% | 28.8% |
| search | 104명 | 24.0% | 27.9% |

전체 구매율은 25.0%였지만 유입 경로별로 나누면 ad는 17.9%, referral은 38.5%로 차이가 나타난다. Day 7 리텐션도 전체에서는 25.0%였지만 ad 유입 사용자는 19.0%로 다른 유입 경로보다 낮다.

하지만 “광고를 통해 들어왔기 때문에 구매율과 리텐션이 낮다”고 해석해서는 안 된다. 현재 확인한 것은 유입 경로와 행동 지표 사이의 차이이다. 광고 자체, 광고로 유입된 사용자의 특성 또는 다른 요인 중 무엇이 원인인지는 추가 검증이 필요하다.

---

## 10. 이용 기기별 구매율과 리텐션

| 이용 기기 | 사용자 수 | 구매율 | Day 7 리텐션 |
|---|---:|---:|---:|
| mobile | 140명 | 21.4% | 25.7% |
| pc | 100명 | 30.0% | 24.0% |

기기별로는 구매율에서 차이가 나타나지만 Day 7 리텐션은 상대적으로 비슷하다.

~~~text
전체 구매율 25.0%
    ↓ 유입 경로별 비교
ad 17.9% / referral 38.5% / search 24.0%
    ↓ 기기별 비교
mobile 21.4% / pc 30.0%
~~~

세그멘테이션 후에는 다음 분석 질문을 만들 수 있다.

- ad 유입 사용자의 구매율과 Day 7 리텐션이 낮은 이유는 무엇인가?
- mobile 사용자의 구매율이 pc 사용자보다 낮은 이유는 무엇인가?

이 질문은 검증된 원인이 아니라 다음 분석과 가설 설정의 출발점이다.

### 세그먼트별 지표의 기준

비교하려는 모든 집단에는 동일한 지표를 적용해야 한다.

$$
\text{세그먼트 구매율}
= \frac{\text{해당 세그먼트의 구매 사용자 수}}{\text{해당 세그먼트의 전체 사용자 수}} \times 100
$$

전체 평균과 세그먼트별 결과를 함께 보면 전체 지표에 가려진 사용자 행동 차이를 발견할 수 있다. 다만 세그먼트 간 차이는 원인의 증거가 아니라 추가로 확인할 문제를 찾는 단서이다.

---

## 11. 분석 결과를 해석할 때의 주의점

### 지표가 낮다는 이유만으로 문제라고 단정하지 않는다

AARRR의 영역마다 측정하는 행동과 의미가 다르다. 서비스 목표, 기존 성과와 목표 수준을 함께 고려해야 한다.

### 이벤트 수와 고유 사용자 수를 구분한다

한 사용자가 같은 이벤트를 여러 번 발생시킬 수 있다. 퍼널의 단계 도달자를 계산할 때는 분석 목적에 따라 고유 사용자 수를 사용해야 한다.

### 실제 퍼널에서는 행동 순서와 진입 조건을 확인한다

각 이벤트를 수행한 고유 사용자 수만 집계하면 앞 단계를 거치지 않았거나 순서가 다른 사용자까지 포함될 수 있다.

### 리텐션 계산 방식을 명확하게 정의한다

Day N을 정확히 N일 후 재방문으로 볼지, N일 이후의 재방문으로 볼지 등 기준에 따라 결과가 달라진다. 이번 실습은 정확히 Day 1, Day 7, Day 14의 방문을 측정했다.

### 관찰한 현상과 원인 가설을 구분한다

퍼널의 높은 이탈률, 코호트 간 리텐션 차이와 세그먼트 간 지표 차이는 데이터에서 관찰한 현상이다. 그 차이가 발생한 이유는 추가 데이터와 검증을 통해 확인해야 한다.

---

## 12. 사용자 행동 분석의 전체 연결 구조

~~~text
AARRR
서비스 전체에서 살펴볼 행동 영역 발견
    ↓
Funnel
목표 행동까지 문제가 발생하는 구간 확인
    ↓
Retention / Cohort
시간에 따른 유지 변화와 코호트 차이 확인
    ↓
Segmentation
차이가 집중되는 사용자 집단 확인
    ↓
추가 행동 분석과 가설 검증
~~~

각 분석은 최종 결론이 아니라 문제를 더 구체적으로 만드는 단계이다. 결과를 다음 질문으로 연결할수록 “전체 구매율이 낮다”와 같은 넓은 문제를 “특정 유입 경로 또는 기기의 사용자가 어느 행동 구간에서 이탈하는가?”처럼 실행 가능한 질문으로 발전시킬 수 있다.

---

## 13. 핵심 정리

- 이벤트 로그 분석은 문제를 분석 질문으로 구체화하는 것에서 시작한다.
- 분석 전에 필요한 이벤트, 기간, 사용자 구분 기준이 데이터에 존재하는지 확인한다.
- AARRR은 서비스 전체의 사용자 행동을 다섯 영역으로 나누어 진단한다.
- AARRR의 영역은 하나의 순차 퍼널이 아니며 각 영역의 대표 행동과 계산 기준을 명확히 해야 한다.
- 퍼널 분석은 목표 행동까지의 단계별 고유 사용자 수, 전환율과 이탈률을 확인한다.
- 실제 퍼널에서는 이벤트 수행 순서와 퍼널 진입 조건을 고려해야 한다.
- 리텐션은 사용자가 최초 이용 후 특정 시점에 다시 이용하는지를 측정한다.
- 코호트 분석은 시작 시기가 비슷한 사용자 집단의 유지 패턴을 비교한다.
- 전체 지표만 보면 숨겨질 수 있는 차이를 세그멘테이션으로 확인할 수 있다.
- 세그먼트별 지표는 각 집단에 동일한 기준으로 계산해야 한다.
- 지표 차이는 원인에 대한 증거가 아니라 추가 분석이 필요한 현상이다.
- 분석 질문, 집계, 계산, 시각화, 해석과 다음 질문 설정의 흐름을 반복하면서 문제를 구체화해야 한다.

---

## 오늘의 한 문장

> 이벤트 로그 기반 사용자 행동 분석은 여러 지표를 한 번 계산하고 끝내는 작업이 아니라, 전체 행동에서 문제 영역을 찾고 시간·구간·사용자 집단의 관점으로 질문을 좁혀가는 과정이다.