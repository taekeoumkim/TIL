# 2026-10-06 부스팅 계열 모델의 원리와 발전

## 학습 목표

- Boosting이 이전까지의 예측 오류를 다음 모델이 순차적으로 보완하는 앙상블 방법임을 설명할 수 있다.
- Bagging과 Boosting의 학습 방식과 모델 간 관계를 구분할 수 있다.
- Gradient Boosting Machine이 잔차와 Negative Gradient를 이용하는 원리를 이해한다.
- `learning_rate`와 `n_estimators`가 예측과 과적합에 미치는 영향을 설명할 수 있다.
- XGBoost가 예측 오류와 모델 복잡도를 함께 고려하는 방법을 이해한다.
- GBM과 XGBoost의 특징을 비교하고 검증 성능을 기준으로 모델을 선택할 수 있다.

---

## 1. 앞의 모델이 놓친 부분을 다음 모델이 보완하기

하나의 모델이 실제값 100을 70으로 예측했다면 30만큼의 오차가 남는다. 여기서 학습을 끝내는 대신, 다음 모델이 부족한 부분을 보완하도록 만들 수 있다.

예를 들어 두 번째 모델이 20을 보완하고 세 번째 모델이 8을 추가로 보완하면 최종 예측은 다음과 같다.

$$
70 + 20 + 8 = 98
$$

처음의 예측 70보다 실제값 100에 가까워진다. 이는 오차 보완의 원리를 설명하기 위한 단순화된 예시이다.

**Boosting**은 여러 개의 비교적 단순한 모델을 순차적으로 학습하고 결합하여 하나의 강한 모델을 만드는 앙상블 방법이다. 뒤에 학습하는 모델은 앞 모델과 무관하게 만들어지는 것이 아니라, **현재까지 결합된 모델의 예측 결과**를 이용하여 학습한다.

중요한 점은 바로 직전 Tree 하나의 오류만 보는 것이 아니라, 지금까지의 전체 예측이 남긴 오류를 보완한다는 것이다.

---

## 2. Bagging과 Boosting의 차이

두 방법 모두 여러 모델의 예측을 결합하지만 학습 구조가 다르다.

| 구분 | Bagging | Boosting |
|---|---|---|
| 학습 방식 | 여러 모델을 비교적 독립적으로 학습 | 앞선 예측 결과를 이용해 순차적으로 학습 |
| 모델 간 관계 | 다른 모델의 예측 결과가 학습에 직접 필요하지 않음 | 현재까지의 예측이 다음 모델의 학습 방향에 영향을 줌 |
| 결합 방식 | 분류의 투표·확률 평균, 회귀의 평균 등 | Gradient Boosting에서는 보정값을 차례대로 더함 |
| 주요 아이디어 | 다양한 모델의 예측을 결합하여 변동 완화 | 이전까지 설명하지 못한 부분을 추가 학습 |
| 대표 모델 | Random Forest | GBM, XGBoost |

Random Forest는 서로 다른 Tree의 예측을 결합하여 주로 분산을 줄이는 방식이다. Boosting은 남아 있는 예측 오류를 계속 보완하면서 전체 모델의 표현력을 높인다.

다만 계속 오류를 보완한다고 일반화 성능이 반드시 좋아지는 것은 아니다. 학습 데이터의 노이즈까지 따라가면 Overfitting이 발생할 수 있다.

---

## 3. GBM과 Residual의 의미

**Gradient Boosting Machine(GBM)**은 새로운 모델을 순차적으로 추가하면서 Loss가 줄어드는 방향으로 예측을 수정하는 방법이다. 여기서는 Decision Tree를 사용하는 GBM을 생각한다.

제곱 오차를 사용하는 회귀에서는 **Residual(잔차)**을 통해 원리를 이해할 수 있다.

$$
r_i = y_i - F(x_i)
$$

- \(y_i\): 실제값
- \(F(x_i)\): 현재까지의 모델이 예측한 값
- \(r_i\): 실제값과 현재 예측값의 차이

| 잔차 | 의미 | 필요한 수정 방향 |
|---|---|---|
| 양수 | 실제값보다 작게 예측함 | 예측값을 높이는 방향 |
| 음수 | 실제값보다 크게 예측함 | 예측값을 낮추는 방향 |
| 0 | 실제값과 예측값이 같음 | 해당 관측치의 보정이 필요 없음 |

새로운 Tree는 원래 타깃을 처음부터 다시 예측하는 대신, 기존 Feature를 이용하여 이 보정값을 학습한다.

---

## 4. 숫자로 이해하는 GBM의 학습 과정

방문 횟수로 고객의 구매 금액을 예측한다고 가정하자.

| 고객 | 방문 횟수 | 실제 구매 금액 |
|---|---:|---:|
| A | 1 | 100 |
| B | 2 | 120 |
| C | 8 | 280 |
| D | 10 | 300 |

### 초기 예측 만들기

제곱 오차를 사용하는 회귀에서는 실제값의 평균을 초기 예측으로 사용할 수 있다.

$$
F_0(x) = \frac{100+120+280+300}{4}=200
$$

처음에는 모든 고객을 200으로 예측한다.

### 잔차 계산하기

| 고객 | 실제값 | 초기 예측 | 잔차 |
|---|---:|---:|---:|
| A | 100 | 200 | -100 |
| B | 120 | 200 | -80 |
| C | 280 | 200 | +80 |
| D | 300 | 200 | +100 |

다음 Tree의 입력은 기존 Feature인 방문 횟수이고, 학습할 타깃은 잔차이다.

### Tree로 잔차 학습하기

Tree가 `방문 횟수 < 5`를 기준으로 고객을 나누었다고 가정하면 각 Leaf의 평균 잔차는 다음과 같다.

- 왼쪽 Leaf: A와 B의 평균 잔차 = (-100 - 80) / 2 = -90
- 오른쪽 Leaf: C와 D의 평균 잔차 = (80 + 100) / 2 = +90

이 Tree는 구매 금액 자체가 아니라 **현재 예측을 얼마나 수정할지**를 예측한다.

### 보정값을 기존 예측에 추가하기

원리를 단순하게 보기 위해 여기서는 `learning_rate=1.0`으로 보정값을 전부 반영한다.

| 고객 | 기존 예측 | Tree의 보정값 | 새로운 예측 | 새로운 잔차 |
|---|---:|---:|---:|---:|
| A | 200 | -90 | 110 | -10 |
| B | 200 | -90 | 110 | +10 |
| C | 200 | +90 | 290 | -10 |
| D | 200 | +90 | 290 | +10 |

처음보다 실제값에 가까워졌다. 다음 Tree는 새로운 잔차를 타깃으로 학습한다. 새로운 Tree를 학습할 때마다 잔차를 다시 계산하므로 학습 대상도 계속 달라진다.

---

## 5. 왜 Gradient Boosting이라고 할까?

잔차를 학습한다는 설명은 제곱 오차 회귀에서 이해하기 쉬운 형태이다. 일반적인 Gradient Boosting은 **Loss의 Negative Gradient를 학습하는 과정**으로 이해해야 한다.

Gradient는 현재 예측에서 Loss가 증가하는 방향을 나타낸다. Loss를 줄이기 위해서는 그 반대 방향인 Negative Gradient를 이용한다.

예를 들어 Loss를 다음과 같이 정의하면,

$$
L(y,F)=\frac{1}{2}(y-F)^2
$$

예측값 \(F\)에 대한 Negative Gradient는 다음과 같다.

$$
-\frac{\partial L}{\partial F}=y-F
$$

즉, 이 경우 Negative Gradient는 잔차와 같다. 새로운 Tree가 잔차를 학습하는 이유를 Loss를 줄이는 방향이라는 관점에서 설명할 수 있다.

다른 Loss를 사용하는 문제에서는 새 Tree가 학습할 값도 달라진다. 분류에서 단순히 `실제 클래스 - 예측 클래스`를 계산하는 것으로 생각하면 안 된다.

**GBM의 일반적인 핵심은 현재 예측의 Loss를 줄이는 방향을 새 Tree가 학습하고, 그 결과를 기존 모델에 추가하는 것이다.**

---

## 6. learning_rate: 새로운 Tree를 얼마나 반영할까?

GBM에서는 새 Tree의 예측을 그대로 모두 반영하는 대신 `learning_rate`를 곱하여 기여도를 조절할 수 있다.

$$
F_m(x)=F_{m-1}(x)+\eta h_m(x)
$$

- \(F_{m-1}(x)\): 기존 모델의 예측
- \(h_m(x)\): 새 Tree의 보정값
- \(\eta\): learning_rate
- \(F_m(x)\): 업데이트된 예측

기존 예측이 100이고 새로운 Tree가 40을 예측했다면 다음과 같이 달라진다.

| learning_rate | 반영하는 보정값 | 새로운 예측 |
|---|---:|---:|
| 1.0 | 40 | 140 |
| 0.1 | 4 | 104 |
| 0.01 | 0.4 | 100.4 |

작은 learning_rate는 각 Tree의 영향을 줄여 조금씩 예측을 수정하게 한다. 이처럼 기여도를 줄이는 것을 **Shrinkage**라고 한다.

- 큰 값: 각 단계의 수정 폭이 커지고, 학습 데이터의 패턴을 빠르게 따라갈 수 있다.
- 작은 값: 수정 폭이 작아지고, 충분히 학습하려면 일반적으로 더 많은 Tree가 필요하다.

작은 값이 무조건 좋은 것은 아니다. Tree 수가 부족하면 필요한 패턴을 충분히 학습하지 못할 수 있다.

---

## 7. n_estimators와 learning_rate를 함께 고려하기

`n_estimators`는 순차적으로 추가하는 Tree의 수를 조절한다. 회귀용 `GradientBoostingRegressor`에서는 이를 추가할 Tree 개수로 이해할 수 있다.

| 설정 예시 | 학습 방식 |
|---|---|
| learning_rate=0.5, n_estimators=20 | 큰 보정값을 상대적으로 적은 횟수로 반영 |
| learning_rate=0.05, n_estimators=200 | 작은 보정값을 여러 단계에 걸쳐 반영 |

두 설정은 learning_rate와 Tree 수의 곱이 같더라도 같은 모델이 아니다. 매 단계의 예측이 달라지면 다음 Tree의 학습 대상과 구조도 달라지기 때문이다.

Tree가 많아지면 학습 데이터의 오류를 더 줄일 수 있지만, 검증 성능은 어느 시점부터 악화될 수 있다.

다음은 과적합을 설명하기 위한 가상의 오차이다.

| Tree 개수 | Train Error | Validation Error |
|---|---:|---:|
| 10 | 25 | 30 |
| 50 | 12 | 18 |
| 100 | 5 | 16 |
| 200 | 1 | 24 |

이 예시에서는 Train Error가 가장 작은 200개 설정보다 Validation Error가 가장 작은 100개 설정을 선택하는 것이 적절하다.

**두 파라미터는 학습 시간과 검증 성능을 함께 비교하며 조절해야 한다.**

---

## 8. GBM 구현과 설정 비교

다음은 회귀 데이터를 생성하고 세 가지 설정의 Train MSE와 Validation MSE를 비교하는 코드이다. MSE는 작은 값이 더 좋다.

~~~python
import pandas as pd
from sklearn.datasets import make_regression
from sklearn.model_selection import train_test_split
from sklearn.ensemble import GradientBoostingRegressor
from sklearn.metrics import mean_squared_error

X, y = make_regression(
    n_samples=1000,
    n_features=8,
    n_informative=5,
    noise=15,
    random_state=42,
)

X_train, X_valid, y_train, y_valid = train_test_split(
    X, y, test_size=0.2, random_state=42
)

settings = [(0.5, 20), (0.1, 100), (0.05, 200)]
results = []

for learning_rate, n_estimators in settings:
    model = GradientBoostingRegressor(
        learning_rate=learning_rate,
        n_estimators=n_estimators,
        max_depth=3,
        random_state=42,
    )
    model.fit(X_train, y_train)

    results.append({
        "learning_rate": learning_rate,
        "n_estimators": n_estimators,
        "train_mse": mean_squared_error(
            y_train, model.predict(X_train)
        ),
        "valid_mse": mean_squared_error(
            y_valid, model.predict(X_valid)
        ),
    })

result_table = pd.DataFrame(results).sort_values("valid_mse")
print(result_table)
~~~

Train MSE는 학습 데이터에 얼마나 잘 적합했는지 나타낸다. 모델을 선택할 때는 Validation MSE도 확인하여 학습 데이터에만 잘 맞는 설정을 선택하지 않도록 해야 한다.

이 예시는 설정을 비교하는 용도이다. 최종 성능 평가에는 모델 선택에 사용하지 않은 Test 데이터를 별도로 확보한다. 분할에 따른 변동을 고려하려면 Cross-Validation을 이용할 수 있다.

---

## 9. XGBoost: 예측 오류와 모델 복잡도를 함께 고려하기

**XGBoost(eXtreme Gradient Boosting)**는 Gradient Boosting을 기반으로 규제와 효율적인 Tree 학습 등을 추가한 모델이다.

기본적인 학습 구조는 GBM과 같다. 현재까지의 예측을 바탕으로 새로운 Tree를 순차적으로 학습한다. Random Forest처럼 Tree들을 독립적으로 학습하는 구조는 아니다.

목적함수는 개념적으로 다음과 같이 표현할 수 있다.

$$
\mathrm{Objective}=\mathrm{Loss}+\mathrm{Regularization}
$$

| 항목 | 역할 |
|---|---|
| Loss | 실제값과 예측값의 차이를 평가한다. |
| Regularization | Tree의 구조와 Leaf의 값이 지나치게 복잡해지는 것을 억제한다. |

Ridge와 Lasso가 계수에 규제를 적용하는 것처럼, XGBoost는 Tree의 복잡도와 Leaf의 값에 규제를 적용한다. 예측 오류를 줄이면서 모델이 지나치게 복잡해지지 않도록 고려하는 것이다.

규제가 있다고 과적합이 자동으로 해결되는 것은 아니다. 모델 설정과 검증 결과를 함께 확인해야 한다.

---

## 10. XGBoost의 주요 과적합 제어 파라미터

| 파라미터 | 역할 | 조절할 때의 고려사항 |
|---|---|---|
| `learning_rate` | 각 Tree의 기여도를 조절 | 작은 값에서는 일반적으로 더 많은 Tree가 필요하다. |
| `n_estimators` | Boosting 반복 횟수를 조절 | 지나치게 늘리면 노이즈까지 학습할 수 있다. |
| `max_depth` | Tree의 최대 깊이를 제한 | 작은 값은 개별 Tree의 표현력을 제한한다. |
| `min_child_weight` | 자식 노드에 필요한 Hessian 합의 최솟값 | 큰 값은 분할을 더 보수적으로 만든다. |
| `subsample` | 각 단계에서 사용할 학습 행의 비율 | 1보다 작은 값으로 일부 행만 사용한다. |
| `colsample_bytree` | 각 Tree에서 사용할 Feature의 비율 | 일부 Feature만 사용하여 Tree를 학습한다. |
| `reg_alpha` | Leaf 가중치에 대한 L1 규제 강도 | 가중치가 지나치게 커지는 것을 억제한다. |
| `reg_lambda` | Leaf 가중치에 대한 L2 규제 강도 | 가중치의 크기에 페널티를 적용한다. |

### min_child_weight는 단순한 데이터 개수가 아니다

Hessian은 Loss의 2차 미분과 관련된 값이다. `min_child_weight`는 자식 노드 내부의 Hessian 합에 대한 최소 조건을 설정한다.

값을 크게 하면 조건을 충족하지 못하는 분할이 억제된다. 일반적으로 세밀한 분할을 제한하는 방향으로 작용하지만, 단순히 Leaf에 필요한 최소 데이터 개수를 지정하는 파라미터와는 구분해야 한다.

### 데이터와 Feature의 일부만 사용하기

`subsample=0.8`은 각 Boosting 단계에서 학습 행의 일부를 사용하고, `colsample_bytree=0.8`은 각 Tree에서 Feature의 일부를 사용하는 설정이다.

전체 데이터와 전체 Feature에 대한 의존을 줄여 과적합을 제어하는 데 활용할 수 있다. 다만 사용하는 정보를 너무 줄이면 학습 부족이 발생할 수 있다.

Random Forest와 비슷한 무작위성을 활용하더라도, XGBoost의 Tree는 계속 순차적으로 학습된다.

---

## 11. XGBoost 구현 예시

앞에서 분리한 회귀용 Train / Validation 데이터로 XGBoost를 학습하는 예시이다. 실행에는 `xgboost`가 필요하다.

~~~python
from xgboost import XGBRegressor
from sklearn.metrics import mean_squared_error

xgb_model = XGBRegressor(
    objective="reg:squarederror",
    n_estimators=200,
    learning_rate=0.05,
    max_depth=3,
    min_child_weight=1,
    subsample=0.8,
    colsample_bytree=0.8,
    reg_alpha=0.1,
    reg_lambda=1.0,
    random_state=42,
)

xgb_model.fit(X_train, y_train)

print("Train MSE:", mean_squared_error(
    y_train, xgb_model.predict(X_train)
))
print("Validation MSE:", mean_squared_error(
    y_valid, xgb_model.predict(X_valid)
))
~~~

위 설정은 사용 예시이며 모든 데이터에서 최적의 값은 아니다. 같은 분할과 평가 지표로 후보 모델을 비교해야 한다.

회귀에는 `XGBRegressor`, 분류에는 `XGBClassifier`를 사용한다. 구매 여부와 같은 이진 타깃은 분석 목적에 맞게 분류 문제로 정의하고 분류 지표로 평가한다.

---

## 12. GBM과 XGBoost 비교 및 선택

| 관점 | GBM | XGBoost |
|---|---|---|
| 기본 원리 | Loss를 줄이는 방향으로 모델을 순차적으로 추가 | Gradient Boosting의 원리를 확장 |
| 학습 관계 | 이전 단계의 예측을 다음 학습에 활용 | 이전 단계의 예측을 다음 학습에 활용 |
| 복잡도 제어 | Tree 깊이, 학습률, 반복 횟수 등 | 이러한 설정에 더해 명시적인 규제 항 등을 활용 |
| 특징 | 잔차와 예측 업데이트의 원리를 이해하기 좋음 | 규제와 효율적인 학습을 위한 기능 제공 |
| 선택 기준 | 검증 성능과 필요한 기능 확인 | 검증 성능과 조절·운영 부담 확인 |

GBM은 Gradient Boosting이라는 방법을 가리키는 표현이기도 하며, XGBoost도 이 계열에 포함된다. 구현을 비교할 때는 scikit-learn의 `GradientBoostingRegressor`와 `XGBRegressor`를 비교하는 것으로 이해하면 명확하다.

모델을 선택할 때는 다음을 확인한다.

1. 회귀인지 분류인지 명확히 정의한다.
2. 문제에 적합한 평가 지표와 데이터 분할 방법을 정한다.
3. 동일한 조건에서 GBM과 XGBoost를 학습하고 평가한다.
4. Train과 Validation 성능의 차이로 과적합이나 학습 부족을 확인한다.
5. 학습률, 반복 횟수, Tree 복잡도와 규제를 조절한다.
6. 검증 성능에 더해 학습 시간과 운영의 편의성을 고려한다.

XGBoost가 반드시 더 높은 성능을 보이는 것은 아니다. 모델 이름보다 대상 데이터에서의 일반화 성능을 기준으로 선택해야 한다.

---

## 13. 핵심 정리

- Boosting은 모델을 순차적으로 학습하여 현재까지의 예측이 설명하지 못한 부분을 보완한다.
- Bagging은 모델을 비교적 독립적으로 학습하고, Boosting은 앞선 예측을 다음 학습에 활용한다.
- 제곱 오차 회귀 GBM에서는 평균으로 초기 예측을 만들고, 잔차를 새로운 Tree의 타깃으로 사용할 수 있다.
- 새 Tree는 원래 타깃을 다시 예측하는 대신 기존 예측에 필요한 보정값을 학습한다.
- Gradient Boosting의 일반적인 원리는 Loss의 Negative Gradient를 이용하는 것이다.
- learning_rate는 각 Tree의 기여도를, n_estimators는 반복 횟수를 조절하므로 함께 고려해야 한다.
- Tree를 늘리면 Train 성능은 좋아져도 Validation 성능은 나빠질 수 있다.
- XGBoost는 Loss와 규제를 함께 고려하여 모델의 복잡도를 제어한다.
- 깊이, 분할 조건, 행과 Feature의 샘플링, L1·L2 규제 등으로 과적합을 조절할 수 있다.
- 가장 좋은 설정은 데이터에 따라 달라지므로 Validation이나 Cross-Validation으로 비교해야 한다.

---

## 오늘의 한 문장

> 부스팅은 앞선 예측이 남긴 오류를 순차적으로 보완하는 방법이며, 학습률과 Tree 개수, 모델 복잡도를 함께 조절해야 새로운 데이터에서도 좋은 예측을 만들 수 있다.