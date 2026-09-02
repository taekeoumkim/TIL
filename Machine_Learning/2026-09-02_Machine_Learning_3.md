# 2026-09-02 선형회귀와 회귀 평가지표

## 학습 목표

- 모델과 학습의 의미를 설명할 수 있다.
- 선형회귀의 가중치와 절편이 어떤 역할을 하는지 이해할 수 있다.
- 단순 선형회귀와 다중 선형회귀를 구분할 수 있다.
- scikit-learn으로 선형회귀 모델을 학습하고 새로운 값을 예측할 수 있다.
- 실제값과 예측값을 이용해 잔차를 계산할 수 있다.
- MAE, MSE, RMSE, R²의 의미와 차이를 설명하고 계산할 수 있다.
- 학습 데이터와 평가 데이터를 분리해 모델의 일반화 성능을 평가할 수 있다.

---

## 1. 회귀란?

회귀(Regression)는 입력 데이터를 이용해 연속적인 수치 값을 예측하는 지도학습 문제이다.

- 집의 넓이로 집값 예측
- 공부 시간으로 시험 점수 예측
- 배송 거리로 배달 시간 예측
- 고객 정보로 다음 달 구매 금액 예측

스팸 여부나 이탈 여부처럼 범주를 예측하면 분류 문제이고, 가격·시간·금액처럼 연속적인 수치를 예측하면 회귀 문제이다.

---

## 2. 모델과 학습

### 모델(Model)

모델은 하나 이상의 특성으로 구성된 입력 $X$를 받아 예측값 $\hat{y}$를 출력하는 함수 $f(X)$이다.

$$
\hat{y}=f(X)
$$

예를 들어 집의 넓이를 입력받아 집값을 출력하는 함수가 하나의 회귀 모델이다.

### 학습(Training)

학습은 모델의 예측값 $\hat{y}$가 실제 정답 $y$에 가까워지도록 모델 내부의 파라미터를 결정하는 과정이다.

선형회귀에서 학습되는 대표적인 파라미터는 다음과 같다.

- 가중치(Weight): 입력 특성이 예측값에 미치는 영향
- 절편(Bias 또는 Intercept): 모든 특성값이 0일 때의 기본 예측값

사람은 선형회귀와 같은 모델의 종류와 학습 방법을 선택하지만, 구체적인 가중치와 절편은 학습 데이터를 이용해 모델이 결정한다.

```text
학습 데이터 X + 실제 정답 y
              ↓
        모델 학습 fit()
              ↓
       가중치와 절편 결정
              ↓
새로운 데이터 + 학습된 모델 → 예측값
```

---

## 3. 첫 번째 선형회귀 모델

배송 거리와 실제 배달 시간의 관계를 학습해 보자.

```python
from sklearn.linear_model import LinearRegression

# 입력 데이터: 배달 거리(km)
X = [[1], [2], [3], [4], [5]]

# 실제 배달 시간(분)
y = [15, 20, 25, 30, 35]

model = LinearRegression()
model.fit(X, y)

predicted_time = model.predict([[6]])

print(f"가중치: {model.coef_[0]:.1f}")
print(f"절편: {model.intercept_:.1f}")
print(f"6km 예상 배달 시간: {predicted_time[0]:.1f}분")
```

실행 결과:

```text
가중치: 5.0
절편: 10.0
6km 예상 배달 시간: 40.0분
```

학습된 모델은 다음 관계를 찾았다.

$$
\widehat{배달시간}=5\times 배송거리+10
$$

- 가중치 5: 배송 거리가 1km 증가할 때 예상 배달 시간이 평균 5분 증가
- 절편 10: 배송 거리가 0km일 때 모델이 예측하는 배달 시간

`model.fit(X, y)`는 데이터로 가중치와 절편을 학습하고, `model.predict()`는 학습된 관계를 이용해 새로운 입력의 값을 예측한다.

---

## 4. 선형회귀의 기본 구조

선형회귀(Linear Regression)는 하나 이상의 입력 특성과 연속적인 정답 사이의 관계를 직선, 평면 또는 초평면으로 표현한다.

선형회귀는 각 특성이 변할 때 예측값이 일정한 비율로 변하는 선형 관계를 가정한다. 구조가 단순하고 각 특성의 영향을 계수로 해석할 수 있다는 장점이 있다.

---

## 5. 단순 선형회귀

단순 선형회귀(Simple Linear Regression)는 입력 특성이 하나인 선형회귀이다.

$$
\hat{y}=wx+b
$$

| 기호 | 의미 |
|---|---|
| $x$ | 입력 특성 |
| $w$ | 가중치 또는 기울기 |
| $b$ | 절편 |
| $\hat{y}$ | 모델의 예측값 |

그래프로 표현하면 하나의 직선이 된다.

### 공부 시간으로 점수 예측하기

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression

X = np.array([1, 2, 3, 4, 5]).reshape(-1, 1)
y = np.array([50, 60, 70, 80, 90])

model = LinearRegression()
model.fit(X, y)

score = model.predict([[6]])[0]

print(f"가중치: {model.coef_[0]:.2f}")
print(f"절편: {model.intercept_:.2f}")
print(f"6시간 공부 예상 점수: {score:.2f}점")

X_line = np.linspace(1, 6, 100).reshape(-1, 1)
y_line = model.predict(X_line)

plt.figure(figsize=(7, 4))
plt.scatter(X, y, color="steelblue", label="실제 데이터")
plt.plot(X_line, y_line, color="tomato", label="회귀선")
plt.xlabel("공부 시간")
plt.ylabel("시험 점수")
plt.title("단순 선형회귀")
plt.legend()
plt.tight_layout()
plt.show()
```

scikit-learn의 입력 `X`는 `(샘플 수, 특성 수)` 형태의 2차원 배열이어야 한다. 특성이 하나여도 `reshape(-1, 1)`을 이용해 열이 하나인 2차원 배열로 만든다.

---

## 6. 다중 선형회귀

다중 선형회귀(Multiple Linear Regression)는 입력 특성이 두 개 이상인 선형회귀이다.

$$
\hat{y}=w_1x_1+w_2x_2+\cdots+w_px_p+b
$$

각 특성에는 별도의 가중치가 존재한다. $w_j$는 **다른 특성들이 동일하다고 가정할 때**, $x_j$가 1단위 증가하면 예측값이 평균적으로 얼마나 변하는지를 나타낸다.

### 중고차 가격 예측하기

```python
import numpy as np
from sklearn.linear_model import LinearRegression

# 각 행: [주행 거리(만 km), 사용 연수]
X = np.array([
    [2, 1],
    [4, 2],
    [6, 3],
    [8, 4],
    [10, 5],
])

# 중고차 가격(백만 원)
y = np.array([36, 32, 28, 24, 20])

model = LinearRegression()
model.fit(X, y)

predicted_price = model.predict([[5, 2]])[0]

print("특성별 가중치:", model.coef_)
print("절편:", model.intercept_)
print(f"예상 중고차 가격: {predicted_price:.1f}백만 원")
```

`[5, 2]`는 주행 거리가 5만 km이고 사용 연수가 2년인 자동차 한 대를 의미한다.

### 단순·다중 선형회귀 비교

| 구분 | 단순 선형회귀 | 다중 선형회귀 |
|---|---|---|
| 입력 특성 수 | 1개 | 2개 이상 |
| 수식 | $\hat{y}=wx+b$ | $\hat{y}=w_1x_1+\cdots+w_px_p+b$ |
| 그래프 형태 | 직선 | 평면 또는 초평면 |
| 예시 | 운동 시간 → 칼로리 | 운동·수면·식단 → 칼로리 |

특성이 많다고 항상 예측 성능이 좋아지는 것은 아니다. 관련 없는 특성은 노이즈를 추가하고 과적합 가능성을 높일 수 있으므로 예측 시점에 사용할 수 있고 정답과 관련된 특성을 선택해야 한다.

---

## 7. 실제값, 예측값, 잔차

모델의 예측값과 실제값의 차이를 잔차(Residual)라고 한다.

$$
e_i=y_i-\hat{y}_i
$$

```python
import numpy as np

y_true = np.array([20, 30, 40, 50])
y_pred = np.array([18, 33, 37, 55])

residuals = y_true - y_pred

print("실제값:", y_true)
print("예측값:", y_pred)
print("잔차:", residuals)  # [2, -3, 3, -5]
```

- 잔차가 양수: 실제값이 예측값보다 큼 → 과소 예측
- 잔차가 음수: 실제값이 예측값보다 작음 → 과대 예측
- 잔차가 0: 실제값과 예측값이 같음

잔차를 그대로 평균 내면 양수와 음수가 상쇄될 수 있다. 따라서 절댓값을 사용하거나 제곱하는 방식으로 오차의 크기를 요약한다.

---

## 8. MAE: 평균절대오차

MAE(Mean Absolute Error)는 잔차의 절댓값을 평균 낸 값이다.

$$
MAE=\frac{1}{n}\sum_{i=1}^{n}|y_i-\hat{y}_i|
$$

### 특징

- 정답과 같은 단위를 사용한다.
- ‘평균적으로 실제값과 얼마나 차이 나는가?’로 직관적으로 해석할 수 있다.
- 오차 크기에 비례해 동일한 가중치를 부여한다.
- MSE와 RMSE보다 큰 오차와 이상치의 영향을 상대적으로 적게 받는다.

배달 시간 모델의 MAE가 3.25라면 예측값이 실제 배달 시간과 평균적으로 약 3.25분 차이 난다고 해석할 수 있다.

---

## 9. MSE: 평균제곱오차

MSE(Mean Squared Error)는 잔차를 제곱한 뒤 평균 낸 값이다.

$$
MSE=\frac{1}{n}\sum_{i=1}^{n}(y_i-\hat{y}_i)^2
$$

### 특징

- 큰 오차를 제곱하므로 크게 틀린 예측에 더 큰 패널티를 준다.
- 이상치의 영향을 크게 받을 수 있다.
- 단위가 원래 정답 단위의 제곱이 되어 직관적인 해석이 어렵다.
- 미분하기 쉬운 형태라 모델의 손실함수로 자주 사용된다.

배달 시간의 단위가 분이라면 MSE의 단위는 분²이 된다.

---

## 10. RMSE: 평균제곱근오차

RMSE(Root Mean Squared Error)는 MSE에 제곱근을 적용한 값이다.

$$
RMSE=\sqrt{\frac{1}{n}\sum_{i=1}^{n}(y_i-\hat{y}_i)^2}
$$

### 특징

- MSE처럼 큰 오차에 민감하다.
- 제곱근을 적용하므로 정답과 같은 단위로 해석할 수 있다.
- 큰 예측 실패를 중요하게 다루면서 원래 단위로 보고하고 싶을 때 유용하다.

---

## 11. MAE·MSE·RMSE 계산하기

```python
import numpy as np
from sklearn.metrics import mean_absolute_error, mean_squared_error

y_true = np.array([20, 30, 40, 50])
y_pred = np.array([18, 33, 37, 55])

mae = mean_absolute_error(y_true, y_pred)
mse = mean_squared_error(y_true, y_pred)
rmse = np.sqrt(mse)

print(f"MAE: {mae:.2f}")
print(f"MSE: {mse:.2f}")
print(f"RMSE: {rmse:.2f}")
```

실행 결과:

```text
MAE: 3.25
MSE: 11.75
RMSE: 3.43
```

세 지표는 동일한 평가 데이터와 동일한 정답 단위에서 비교할 때 일반적으로 작을수록 좋다. 다만 서로 공식과 단위가 다르므로 MAE 3.25와 RMSE 3.43의 숫자 크기만 보고 어느 지표가 더 좋은 결과라고 비교하는 것은 의미가 없다.

---

## 12. R²: 결정계수

R²(Coefficient of Determination, 결정계수)는 모델이 정답의 변동을 평균 예측 기준보다 얼마나 잘 설명하는지를 나타낸다.

$$
R^2=1-\frac{\sum_{i=1}^{n}(y_i-\hat{y}_i)^2}
{\sum_{i=1}^{n}(y_i-\bar{y})^2}
$$

- 분자: 모델 예측의 잔차 제곱합
- 분모: 모든 값을 정답 평균으로 예측했을 때의 오차 제곱합

### 해석

| R² | 해석 |
|---:|---|
| 1 | 모든 평가 데이터를 정확히 예측 |
| 0 | 정답 평균을 항상 예측하는 기준과 비슷함 |
| 음수 | 평균을 예측하는 기준보다도 성능이 나쁨 |

```python
from sklearn.metrics import r2_score

r2 = r2_score(y_true, y_pred)
print(f"R²: {r2:.4f}")
```

R²가 0.8이라고 해서 모든 예측이 실제값과 80% 정확하다는 뜻은 아니다. 또한 높은 R²가 인과관계, 좋은 잔차 형태, 새로운 데이터에서의 성능을 자동으로 보장하지 않는다.

---

## 13. 회귀 평가지표 비교

| 지표 | 계산 방식 | 단위 | 큰 오차 민감도 | 주요 해석 |
|---|---|---|---|---|
| MAE | 절대오차 평균 | 정답과 같음 | 상대적으로 낮음 | 평균적인 절대 차이 |
| MSE | 제곱오차 평균 | 정답 단위의 제곱 | 높음 | 큰 오차에 강한 패널티 |
| RMSE | MSE의 제곱근 | 정답과 같음 | 높음 | 큰 오차를 강조한 원래 단위 오차 |
| R² | 평균 기준 대비 설명력 | 단위 없음 | 제곱오차 기반 | 변동을 얼마나 설명하는지 |

### 지표 선택 예시

- 일반적인 예측 오차를 원래 단위로 쉽게 설명: MAE
- 큰 예측 실패가 특히 위험: RMSE 또는 MSE
- 서로 다른 모델의 평균 기준 대비 설명력 확인: R²

실무에서는 하나의 지표만 보기보다 MAE 또는 RMSE와 R²를 함께 확인하고, 비즈니스에서 허용 가능한 오차 범위도 함께 정의하는 것이 좋다.

---

## 14. 큰 오차가 지표에 미치는 영향

```python
import numpy as np
from sklearn.metrics import mean_absolute_error, mean_squared_error

y_true = np.array([10, 20, 30, 40])

predictions = {
    "고른 오차": np.array([7, 17, 27, 37]),
    "큰 오차 하나": np.array([10, 20, 30, 28]),
}

for name, y_pred_case in predictions.items():
    mae = mean_absolute_error(y_true, y_pred_case)
    rmse = np.sqrt(mean_squared_error(y_true, y_pred_case))
    print(f"{name}: MAE={mae:.2f}, RMSE={rmse:.2f}")
```

두 모델은 절대오차의 합이 같아 MAE가 같지만, 큰 오차 하나가 있는 모델의 RMSE는 더 커진다. 이를 통해 RMSE가 큰 예측 실패를 더 강하게 반영한다는 것을 확인할 수 있다.

---

## 15. 학습 데이터와 평가 데이터 분리

학습에 사용한 데이터로 성능을 평가하면 모델이 새로운 데이터에도 잘 작동하는지 알기 어렵다. 따라서 데이터를 학습 세트와 평가 세트로 나누어야 한다.

```python
import numpy as np
from sklearn.linear_model import LinearRegression
from sklearn.metrics import (
    mean_absolute_error,
    mean_squared_error,
    r2_score,
)
from sklearn.model_selection import train_test_split

X = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]).reshape(-1, 1)
y = np.array([48, 56, 61, 69, 74, 82, 87, 95, 101, 108])

X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42,
)

model = LinearRegression()
model.fit(X_train, y_train)

y_pred = model.predict(X_test)

mae = mean_absolute_error(y_test, y_pred)
mse = mean_squared_error(y_test, y_pred)
rmse = np.sqrt(mse)
r2 = r2_score(y_test, y_pred)

print(f"MAE: {mae:.3f}")
print(f"MSE: {mse:.3f}")
print(f"RMSE: {rmse:.3f}")
print(f"R²: {r2:.3f}")
```

`random_state=42`는 데이터 분할 결과를 재현하기 위한 난수 시드이다. 같은 데이터와 같은 값을 사용하면 학습·평가 세트가 동일하게 나뉜다.

표본이 매우 작으면 한 번의 분할 결과가 불안정할 수 있으므로 이후 교차검증을 통해 여러 분할에서 성능을 확인할 수 있다.

---

## 16. 잔차 시각화

평가지표는 오차를 하나의 숫자로 요약한다. 하지만 선형 관계가 적절한지 확인하려면 잔차의 패턴도 살펴봐야 한다.

```python
import matplotlib.pyplot as plt

residuals = y_test - y_pred

plt.figure(figsize=(7, 4))
plt.scatter(y_pred, residuals, color="steelblue")
plt.axhline(0, color="tomato", linestyle="--")
plt.xlabel("예측값")
plt.ylabel("잔차")
plt.title("잔차 그래프")
plt.tight_layout()
plt.show()
```

선형회귀가 데이터에 적절하다면 잔차가 0을 중심으로 특별한 패턴 없이 흩어지는 모습이 기대된다.

- 곡선 패턴: 선형 관계가 충분하지 않을 가능성
- 부채꼴 패턴: 예측값에 따라 잔차 분산이 달라지는 이분산 가능성
- 극단적으로 큰 잔차: 이상치 또는 설명되지 않은 사례 확인 필요

---

## 17. 선형회귀 해석 시 주의점

### 특성이 많다고 항상 좋은 것은 아니다

관련 없는 특성은 노이즈를 늘리고 새로운 데이터에서 성능을 낮출 수 있다. 예측 목적과 수집 시점을 고려해 의미 있는 특성을 선택해야 한다.

### 계수가 인과관계를 의미하지 않는다

가중치가 양수라고 해서 해당 특성이 결과를 직접 증가시킨다고 단정할 수 없다. 누락된 변수, 변수 사이의 상관, 데이터 수집 방식의 영향을 받을 수 있다.

### 관측 범위 밖 예측은 조심해야 한다

학습 데이터가 배송 거리 1~5km인데 100km의 시간을 예측하는 것은 학습 범위를 크게 벗어난 외삽이다. 선형 관계가 그 범위까지 유지된다는 보장이 없다.

### 지표는 같은 데이터에서 비교해야 한다

데이터의 규모나 정답 단위가 다르면 MAE와 RMSE도 달라진다. 서로 다른 데이터셋에서 계산한 값을 숫자만으로 직접 비교해서는 안 된다.

---

## 18. 오늘 배운 내용 정리

1. 모델은 입력 $X$를 받아 예측값 $\hat{y}$를 출력하는 함수이다.
2. 학습은 예측값과 실제값의 차이를 줄이도록 모델의 파라미터를 결정하는 과정이다.
3. 입력 특성이 하나이면 단순 선형회귀, 두 개 이상이면 다중 선형회귀이다.
4. 선형회귀의 가중치는 해당 특성이 1단위 변할 때 예측값이 얼마나 변하는지를 나타낸다.
5. 특성을 많이 추가한다고 항상 예측 성능이 좋아지는 것은 아니다.
6. 잔차는 실제값에서 예측값을 뺀 값이다.
7. MAE는 평균적인 절대 오차를 원래 단위로 나타내며 큰 오차의 영향이 상대적으로 작다.
8. MSE와 RMSE는 오차를 제곱하므로 큰 오차에 민감하며, RMSE는 정답과 같은 단위를 사용한다.
9. R²는 정답 평균을 예측하는 기준 대비 모델이 변동을 얼마나 설명하는지 나타낸다.
10. 모델은 학습에 사용하지 않은 평가 데이터에서 검증하고, 지표와 잔차 그래프를 함께 확인해야 한다.

---

## 19. 느낀 점

선형회귀는 단순히 직선을 그리는 방법이 아니라 데이터에서 가중치와 절편을 학습해 새로운 연속형 값을 예측하는 모델이라는 점을 이해했다. 다중 선형회귀에서는 각 계수를 해석할 때 다른 특성이 같다는 조건이 필요하고, 특성을 무작정 늘리는 것이 좋은 모델로 이어지지 않는다는 점도 중요하게 느껴졌다.

또한 모델의 성능을 ‘정확도’ 하나로 표현하는 것이 아니라 오차를 어떤 관점에서 볼 것인지에 따라 MAE, MSE, RMSE, R²를 선택해야 한다는 것을 배웠다. 앞으로 회귀 모델을 평가할 때 하나의 지표만 보고 결론 내리지 않고, 비즈니스에서 큰 오차가 얼마나 중요한지 고려하며 평가 데이터의 지표와 잔차 패턴을 함께 확인해야겠다.