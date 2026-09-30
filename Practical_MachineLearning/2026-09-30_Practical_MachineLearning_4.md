# 2026-09-30 TIL: 시계열 데이터 모델링과 예측

> 학습일: 2026-09-30  
> 정리 범위: **1. 시계열 데이터의 Train/Test 분리** 이후  
> 주제: 시간 순서를 고려한 데이터 분리, AR·MA·ARIMA, 미래 예측과 불확실성

## 학습 목표

- 시계열 데이터에서 무작위 Train/Test 분리가 부적절한 이유를 설명할 수 있다.
- AR과 MA가 사용하는 정보 및 학습하는 관계를 구분할 수 있다.
- ARIMA의 `p`, `d`, `q`와 모델 학습 과정의 의미를 설명할 수 있다.
- ARIMA로 단일 시점 및 여러 미래 시점을 예측할 수 있다.
- 실제값, 예측값, 예측 오차, 예측 구간을 구분할 수 있다.
- 모델 개발·평가 단계와 실제 미래 예측 단계를 구분할 수 있다.

---

## 1. 시계열 데이터의 Train/Test 분리

### 1.1 Train Data와 Test Data의 역할

- **Train Data**: 모델이 시계열의 패턴과 파라미터를 학습하는 데 사용한다.
- **Test Data**: 학습에 사용하지 않은 미래 구간에서 예측 성능을 평가하는 데 사용한다.

일반적인 머신러닝에서는 데이터를 무작위로 섞어 Train과 Test로 나누기도 한다. 그러나 시계열 데이터는 관측 순서가 중요한 데이터이므로 이러한 방법을 그대로 적용하면 안 된다.

### 1.2 무작위 분리와 시간적 누수

과거 시점을 예측하면서 그보다 미래에 관측된 데이터를 Train에 포함하면, 실제 예측 시점에는 알 수 없는 정보를 모델이 학습하게 된다. 이를 **시간적 누수(Look-ahead)**라고 한다.

예를 들어 2024년 3월을 예측하는 모델이 2024년 8월이나 2025년 데이터를 이미 학습했다면 실제 상황보다 많은 정보를 사용한 것이다. 이 경우 Test 성능이 실제 운영 성능보다 좋게 평가될 수 있다.

따라서 시계열 예측에서는 다음 원칙을 지켜야 한다.

```text
과거 구간 → Train → 모델 학습
이후 구간 → Test  → 예측 성능 평가
```

| 일반적인 데이터 분리 | 시계열 데이터 분리 |
|---|---|
| 무작위 분리가 가능한 경우가 있음 | 시간 순서를 유지해야 함 |
| Train과 Test가 여러 위치에서 선택될 수 있음 | 과거를 Train, 이후 구간을 Test로 사용 |
| 각 관측값을 독립적으로 다룰 수 있음 | 과거와 미래의 관계가 중요 |
| 행의 순서가 중요하지 않을 수 있음 | 관측 순서가 핵심 정보 |

### 1.3 Test 기간을 정하는 기준

시계열 데이터를 반드시 80:20 또는 70:30과 같은 고정 비율로 나눌 필요는 없다. 실제로 모델이 예측해야 할 기간에 맞춰 Test 구간을 정하는 것이 중요하다.

- 다음 달 예측: 미래 1개월의 성능이 중요
- 분기 예측: 미래 3개월의 성능이 중요
- 반기 예측: 미래 6개월의 성능이 중요

즉, 단순한 분리 비율보다 **어느 시점까지 학습하고 얼마나 먼 미래를 평가할 것인지**가 핵심이다.

### 1.4 Python으로 시간 순서를 유지해 분리하기

```python
import pandas as pd
import matplotlib.pyplot as plt

# 날짜순으로 정렬하고 날짜를 인덱스로 설정
data = data.sort_values("date")
data = data.set_index("date")

# 마지막 6개월을 Test로 분리
train = data.iloc[:-6]
test = data.iloc[-6:]

print("Train:", train.index.min(), "~", train.index.max())
print("Test :", test.index.min(), "~", test.index.max())

plt.figure(figsize=(12, 5))
plt.plot(train.index, train["sales"], label="Train")
plt.plot(test.index, test["sales"], label="Test")
plt.xlabel("Date")
plt.ylabel("Sales")
plt.title("Train / Test Split")
plt.legend()
plt.show()
```

- `data.iloc[:-6]`: 마지막 6개 행을 제외한 과거 데이터
- `data.iloc[-6:]`: 마지막 6개 행인 미래 평가 구간

분리 후에는 Train과 Test의 시작일·종료일을 출력하고 그래프로 확인하여 시간 순서가 올바르게 유지됐는지 점검해야 한다.

---

## 2. AR, MA와 ARIMA 모델

### 2.1 AR(Autoregressive, 자기회귀)

AR 모델은 **과거 값과 현재 값의 관계**를 이용한다. 가장 단순한 AR(1)은 직전 시점의 값 하나로 현재 값을 설명한다.

$$
y_t = c + \phi_1 y_{t-1} + \epsilon_t
$$

예측 시점에는 현재 오차 $\epsilon_t$를 알 수 없으므로 예측값은 다음과 같다.

$$
\hat{y}_t = c + \phi_1 y_{t-1}
$$

| 기호 | 의미 |
|---|---|
| $y_t$ | 현재 시점의 값 |
| $y_{t-1}$ | 직전 시점의 값 |
| $c$ | 상수항 |
| $\phi_1$ | 직전 값에 적용되는 AR 계수 |
| $\epsilon_t$ | 모델이 설명하지 못한 현재 오차 |

AR 모델은 과거 값을 그대로 다음 값으로 사용하는 모델이 아니다. Train Data를 이용해 과거 값과 현재 값 사이의 관계를 나타내는 계수를 추정한다.

### AR의 차수 `p`

`p`는 현재 값을 설명할 때 사용할 과거 값의 개수다.

$$
AR(2):\quad y_t=c+\phi_1y_{t-1}+\phi_2y_{t-2}+\epsilon_t
$$

- AR(1): Lag 1 사용
- AR(2): Lag 1과 Lag 2 사용
- AR(3): Lag 1부터 Lag 3까지 사용

`p`는 분석자가 결정하는 모델 구조이고, 각 Lag에 적용되는 AR 계수는 모델이 Train Data에서 학습한다.

### 2.2 MA(Moving Average)

MA 모델은 **과거 예측 오차와 현재 값의 관계**를 이용한다. 이동평균 통계량과 이름은 같지만, ARIMA 문맥의 MA는 과거 오차를 사용하는 모델을 뜻한다.

가장 단순한 MA(1)은 다음과 같다.

$$
y_t=\mu+\theta_1\epsilon_{t-1}+\epsilon_t
$$

예측값은 다음과 같다.

$$
\hat{y}_t=\mu+\theta_1\epsilon_{t-1}
$$

| 기호 | 의미 |
|---|---|
| $\mu$ | 시계열의 평균 수준 |
| $\epsilon_{t-1}$ | 직전 시점의 예측 오차 |
| $\theta_1$ | 직전 오차에 적용되는 MA 계수 |
| $\epsilon_t$ | 현재 시점에서 새롭게 발생하는 오차 |

과거 값은 데이터에서 직접 관측할 수 있지만, 과거 오차는 모델의 파라미터와 예측 결과에 따라 달라진다. 모델은 Train Data를 바탕으로 오차 구조와 MA 계수를 함께 추정한다.

### MA의 차수 `q`

`q`는 사용할 과거 오차의 개수다.

$$
MA(2):\quad y_t=\mu+\theta_1\epsilon_{t-1}+\theta_2\epsilon_{t-2}+\epsilon_t
$$

- MA(1): 과거 오차 1개 사용
- MA(2): 과거 오차 2개 사용
- MA(3): 과거 오차 3개 사용

### 2.3 AR과 MA 비교

| 모델 | 사용하는 정보 | 모델이 학습하는 관계 |
|---|---|---|
| AR | 과거 관측값 | 과거 값과 현재 값의 관계 |
| MA | 과거 예측 오차 | 과거 오차와 현재 값의 관계 |

### 2.4 I(Integrated)와 차분

AR과 MA는 기본적으로 정상적인 시계열을 모델링한다. 추세 등으로 정상성을 만족하지 않는 시계열에는 차분(Differencing)을 적용할 수 있다.

$$
\Delta y_t=y_t-y_{t-1}
$$

ARIMA에서 차분에 해당하는 부분이 I(Integrated)이며, 차분 횟수를 `d`로 표현한다.

- `d=0`: 차분하지 않음
- `d=1`: 1차 차분
- `d=2`: 2차 차분

차분은 많이 적용할수록 좋은 것이 아니다. 정상성을 확보하는 데 필요한 만큼만 적용해야 한다.

### 2.5 ARIMA 모델

ARIMA(AutoRegressive Integrated Moving Average)는 필요한 경우 원본 시계열을 차분한 뒤, 차분된 시계열의 과거 값과 과거 오차 관계를 함께 모델링한다.

$$
ARIMA(p,d,q)
$$

| 파라미터 | 의미 |
|---|---|
| `p` | AR에서 사용할 과거 값의 개수 |
| `d` | 차분 횟수 |
| `q` | MA에서 사용할 과거 오차의 개수 |

### ARIMA(2, 1, 1)의 동작

1. `d=1`: 원본 시계열을 한 번 차분하여 변화량 시계열을 만든다.
2. `p=2`: 차분된 시계열에서 과거 두 시점의 값을 사용한다.
3. `q=1`: 직전 시점의 예측 오차 하나를 사용한다.
4. AR과 MA 관계로 다음 변화량을 예측한다.
5. 예측한 변화량을 마지막 원본 값에 반영하여 원래 단위의 예측값으로 되돌린다.

AR과 MA가 각각 별도의 예측을 만든 후 평균내는 방식이 아니다. 하나의 ARIMA 모델 안에서 과거 변화량과 과거 오차의 관계를 함께 사용한다.

```text
원본 시계열
    ↓ 1차 차분
변화량 시계열
    ↓ AR: 과거 변화량 + MA: 과거 오차
다음 변화량 예측
    ↓ 마지막 관측 수준에 반영
원래 단위의 미래 값 예측
```

### 2.6 ARIMA의 학습과 최대우도추정

분석자가 정하는 값과 모델이 학습하는 값을 구분해야 한다.

| 구분 | 내용 |
|---|---|
| 분석자가 지정 | `p`, `d`, `q` |
| 모델이 추정 | AR 계수, MA 계수, 기타 모델 파라미터 |

`statsmodels`의 ARIMA는 `fit()`을 실행할 때 Likelihood를 이용해 파라미터를 추정한다. Likelihood는 주어진 모델과 파라미터가 실제 관측 데이터를 얼마나 그럴듯하게 설명하는지를 나타낸다.

모델은 Likelihood가 커지는 방향으로 파라미터를 조정하며, 이를 **Maximum Likelihood Estimation(MLE, 최대우도추정)**이라고 한다. 실제 계산에서는 주로 Log-Likelihood를 사용한다.

따라서 `fit()`은 단순히 `p`, `d`, `q`를 저장하는 과정이 아니라 Train Data를 이용해 실제 AR·MA 계수 등을 추정하는 학습 과정이다.

### 2.7 Python으로 ARIMA 학습하기

```python
from statsmodels.tsa.arima.model import ARIMA

# 모델 구조 지정
model = ARIMA(train["sales"], order=(2, 1, 1))

# Train Data로 파라미터 추정
model_fit = model.fit()

# 학습된 계수와 Log-Likelihood 등 확인
print(model_fit.summary())
```

- `ARIMA(..., order=(2, 1, 1))`: 사용할 모델 구조를 지정한다.
- `fit()`: Likelihood를 기반으로 모델 파라미터를 추정한다.
- `summary()`: 학습된 계수와 Log-Likelihood 등의 결과를 확인한다.

`ARIMA(2,1,1)`은 원리를 설명하기 위한 하나의 예시다. 실제 분석에서는 데이터 특성과 Validation/Test 예측 성능을 비교해 적절한 조합을 선택해야 한다.

---

## 3. ARIMA를 이용한 시계열 예측

### 3.1 Test Data를 미래처럼 사용하기

모델 개발 단계에서는 이미 실제값을 가진 과거 데이터의 마지막 구간을 아직 발생하지 않은 미래라고 가정한다.

```text
Train으로 학습
    ↓
Test 기간을 미래라고 가정하고 예측
    ↓
예측이 끝난 후 숨겨둔 Actual과 비교
    ↓
미래 예측 성능 평가
```

Test의 실제값은 모델 학습이나 예측값 생성에 사용하지 않고, 예측이 끝난 뒤 성능 평가에만 사용한다.

### 3.2 1-step과 Multi-step Forecast

- **1-step ahead Forecast**: 현재 시점 바로 다음 한 시점을 예측한다.
- **Multi-step Forecast**: 현재 시점에서 여러 미래 시점을 한 번에 예측한다.
- **Forecast Horizon**: 현재 시점에서 앞으로 몇 시점까지 예측할 것인지 나타낸다.

예를 들어 6월까지 관측한 월별 데이터로 7월부터 12월까지 예측한다면 Forecast Horizon은 6이다. 7월은 1-step ahead, 12월은 6-step ahead Forecast다.

여러 시점을 예측할 때는 미래의 실제값을 차례로 확인하면서 다음 값을 예측하는 것이 아니다. 현재까지 관측한 정보와 학습된 모델 관계를 이용해 여러 미래 예측을 이어간다. 일반적으로 먼 미래일수록 알 수 없는 변화가 누적되어 불확실성이 커질 수 있다.

### 3.3 Actual, Predicted, Error

예측 오차는 다음과 같이 계산한다.

$$
Error=Actual-Predicted
$$

| 개념 | 의미 |
|---|---|
| Actual | 해당 시점에 실제로 관측된 값 |
| Predicted | 모델이 예측한 값 |
| Error | 실제값과 예측값의 차이 |

Error가 음수라면 모델이 실제값보다 크게 예측한 것이고, 양수라면 실제값보다 작게 예측한 것이다.

Test 기간에는 이미 숨겨둔 Actual이 있으므로 Error와 MAE·RMSE를 계산할 수 있다. 아직 발생하지 않은 실제 미래는 Actual이 없으므로 예측 순간에는 Error와 평가 지표를 계산할 수 없다.

### 3.4 모델 평가와 실제 미래 예측의 차이

| 구분 | 모델 개발·평가 | 실제 미래 예측 |
|---|---|---|
| 학습 데이터 | Train 구간 | 현재까지 수집된 전체 데이터 |
| 예측 대상 | 미래처럼 숨겨둔 Test | 아직 발생하지 않은 실제 미래 |
| Actual | 존재하지만 예측 때 사용하지 않음 | 아직 없음 |
| 확인 가능 정보 | Predicted, Prediction Interval, Error, MAE, RMSE | Predicted, Prediction Interval |
| 오차 계산 시점 | 예측 후 바로 가능 | 미래의 실제값이 관측된 이후 가능 |

운영 중 실제값이 누적되면 과거 예측과 비교하여 Error, MAE, RMSE 등을 계산하고 모델의 성능을 다시 점검할 수 있다.

### 3.5 Prediction Interval

점 예측값만으로는 미래 예측의 불확실성을 표현할 수 없다. Prediction Interval(예측 구간)은 미래 관측값에 대한 불확실성을 범위로 나타낸다.

95% 예측 구간의 기본 아이디어는 다음과 같다.

$$
Prediction\ Interval \approx Forecast \pm 1.96\times Forecast\ Standard\ Error
$$

예를 들어 Forecast가 140이고 Forecast Standard Error가 4라면,

$$
Lower=140-1.96\times4=132.16
$$

$$
Upper=140+1.96\times4=147.84
$$

따라서 약 132~148의 예측 구간을 얻는다.

Prediction Interval을 만들 때 미래의 실제값은 필요하지 않다. 모델이 Train Data에서 추정한 파라미터와 설명되지 않은 변동성을 이용해 Forecast Standard Error와 예측 구간을 계산한다.

- Forecast Standard Error가 작으면 예측 구간이 상대적으로 좁다.
- Forecast Standard Error가 크면 예측 구간이 상대적으로 넓다.
- 일반적으로 Forecast Horizon이 길수록 불확실성이 누적되어 구간이 넓어질 수 있다.
- 구간의 변화는 모델 구조와 데이터 특성에 따라 달라지므로 항상 계속 넓어지는 것은 아니다.

실제값이 예측 구간 안에 포함됐다는 사실 하나만으로 좋은 모델이라고 결론 내릴 수 없다. 구간이 지나치게 넓으면 실제값이 포함되기 쉬우므로 점 예측 오차, MAE·RMSE, 구간의 폭을 함께 봐야 한다.

### 3.6 Python으로 미래 값과 예측 구간 구하기

```python
import pandas as pd
import matplotlib.pyplot as plt
from statsmodels.tsa.arima.model import ARIMA

# ARIMA 모델 학습
model = ARIMA(train["sales"], order=(2, 1, 1))
model_fit = model.fit()

# Test 기간 길이만큼 미래 예측
forecast_result = model_fit.get_forecast(steps=len(test))

# 점 예측값과 95% 예측 구간
predicted = forecast_result.predicted_mean
prediction_interval = forecast_result.conf_int(alpha=0.05)

# 실제값, 예측값, 구간 정리
result = pd.DataFrame({
    "actual": test["sales"],
    "predicted": predicted,
    "lower": prediction_interval.iloc[:, 0],
    "upper": prediction_interval.iloc[:, 1]
})

result["error"] = result["actual"] - result["predicted"]
print(result)

# 결과 시각화
plt.figure(figsize=(12, 5))
plt.plot(train.index, train["sales"], label="Train")
plt.plot(test.index, test["sales"], label="Actual")
plt.plot(result.index, result["predicted"], label="Predicted")
plt.fill_between(
    result.index,
    result["lower"],
    result["upper"],
    alpha=0.2,
    label="95% Prediction Interval"
)
plt.xlabel("Date")
plt.ylabel("Sales")
plt.title("ARIMA Forecast")
plt.legend()
plt.tight_layout()
plt.show()
```

주요 메서드의 역할은 다음과 같다.

| 코드 | 역할 |
|---|---|
| `get_forecast(steps=n)` | Train 마지막 시점 이후 `n`개 시점 예측 |
| `predicted_mean` | 각 미래 시점의 점 예측값 |
| `conf_int(alpha=0.05)` | 95% 예측 구간 |

Forecast와 Prediction Interval은 Test의 Actual을 보지 않고 생성한다. Actual은 예측을 마친 뒤 Error와 평가 지표를 계산할 때 사용한다.

---

## 4. 시계열 예측 모델 평가 시 확인할 점

PDF의 모델 평가 단원은 도입부까지만 포함되어 있으므로, 앞에서 설명된 평가 개념을 기준으로 정리한다.

- 시간 순서를 유지한 Test 구간에서 평가한다.
- 예측값과 실제값을 같은 시점끼리 비교한다.
- Error의 부호뿐 아니라 오차의 전체 크기를 MAE·RMSE 등으로 확인한다.
- 점 예측 성능과 함께 Prediction Interval의 폭과 실제값 포함 여부를 살펴본다.
- 모델 개발 단계의 Test 성능과 실제 운영 중 누적되는 성능을 구분한다.
- 실제 사용 목적과 동일한 Forecast Horizon에서 모델을 비교한다.

---

## 전체 개념 비교

| 개념 | 사용하는 정보·역할 | 핵심 설정 또는 결과 |
|---|---|---|
| Train/Test 분리 | 과거로 학습하고 이후 구간으로 평가 | 시간 순서 유지 |
| AR | 과거 관측값 | `p`, AR 계수 |
| MA | 과거 예측 오차 | `q`, MA 계수 |
| I | 비정상 시계열 차분 | `d` |
| ARIMA | 차분 + AR + MA | `order=(p,d,q)` |
| Forecast Horizon | 얼마나 먼 미래까지 예측할지 결정 | `steps` |
| Error | Actual과 Predicted의 차이 | `Actual - Predicted` |
| Forecast Standard Error | 미래 예측의 불확실성 크기 | 예측 시점별 계산 |
| Prediction Interval | 불확실성을 범위로 표현 | 점 예측값 + 하한·상한 |

## 핵심 정리

1. 시계열 데이터는 무작위로 섞지 않고 과거를 Train, 이후 구간을 Test로 분리한다.
2. Test 기간은 고정 비율보다 실제 예측 목적과 Forecast Horizon에 맞춰 결정한다.
3. AR은 과거 값, MA는 과거 예측 오차를 사용한다.
4. ARIMA의 `p`, `d`, `q`는 각각 과거 값의 수, 차분 횟수, 과거 오차의 수를 의미한다.
5. `ARIMA()`는 모델 구조를 지정하고, `fit()`은 최대우도추정으로 실제 파라미터를 학습한다.
6. Multi-step Forecast에서는 먼 미래로 갈수록 일반적으로 불확실성이 커질 수 있다.
7. 실제 미래를 예측하는 순간에는 Predicted와 Prediction Interval만 확인할 수 있다.
8. Error와 MAE·RMSE는 Actual이 관측된 이후에 계산할 수 있다.
9. 예측 구간의 포함 여부만으로 모델을 판단하지 말고 점 예측 오차와 구간 폭을 함께 평가해야 한다.

## 오늘의 한 문장

> 시계열 예측의 핵심은 시간 순서를 지킨 평가 환경에서 과거 값과 오차의 구조를 학습하고, 미래의 점 예측뿐 아니라 예측 불확실성까지 함께 해석하는 것이다.
