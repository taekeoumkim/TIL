# 2026-10-06 배깅·부스팅 선택과 모델 해석(SHAP)

## 학습 목표

- 편향-분산 관점에서 Bagging과 Boosting의 차이를 설명할 수 있다.
- 동일한 평가 조건에서 Random Forest와 Boosting 계열 모델을 비교하고 선택 근거를 정리할 수 있다.
- 기본 Feature Importance의 의미와 한계를 이해한다.
- Global Interpretation과 Local Interpretation을 구분할 수 있다.
- SHAP의 기본 원리와 Base Value, SHAP Value의 관계를 설명할 수 있다.
- Bar Plot, Beeswarm Plot, Waterfall Plot을 이용하여 전체 모델과 개별 예측을 해석할 수 있다.

---

## 1. 성능이 높은 모델을 찾으면 분석이 끝날까?

고객 이탈 모델이 어떤 고객의 이탈 확률을 87%로 예측했다고 하자. 이 결과는 모델의 판단을 보여주지만, 판단의 근거까지 알려주지는 않는다.

실제 분석에서는 다음 두 가지를 함께 설명할 수 있어야 한다.

- 왜 이 모델을 선택했는가?
- 모델이 어떤 Feature를 이용하여 해당 예측을 만들었는가?

첫 번째는 모델 비교와 선택의 문제이고, 두 번째는 모델 해석의 문제이다. 좋은 예측 성능을 확인한 뒤에는 모델 전체의 특징과 개별 예측의 근거를 살펴봐야 한다.

---

## 2. 편향-분산 관점에서 Bagging과 Boosting 비교하기

**편향(Bias)**은 모델이 실제 패턴을 충분히 표현하지 못하는 데서 생기는 오차와 관련된다. **분산(Variance)**은 학습 데이터가 달라질 때 모델의 예측이 얼마나 민감하게 달라지는지와 관련된다.

### Bagging: 예측의 변동 줄이기

Bagging은 서로 다른 표본으로 여러 모델을 비교적 독립적으로 학습하고 예측을 결합한다.

Random Forest는 Bootstrap Sampling과 Feature Randomness를 통해 다양한 Tree를 만든다. 여러 Tree의 예측을 결합하면 특정 Tree 하나의 불안정한 판단에 의존하는 정도가 줄어든다.

따라서 Bagging은 깊은 Decision Tree처럼 분산이 큰 모델의 **분산을 줄이는 데 주요 효과가 있는 방법**으로 이해할 수 있다.

### Boosting: 설명하지 못한 패턴 보완하기

Boosting은 현재까지의 모델이 충분히 설명하지 못한 부분을 다음 모델이 순차적으로 학습한다.

단순한 모델을 계속 추가하면 전체 모델의 표현력이 높아지고 편향을 줄이는 방향으로 작용할 수 있다. 하지만 지나치게 학습하면 노이즈까지 설명하려 하면서 분산이 증가하고 과적합될 수 있다.

| 관점 | Bagging | Boosting |
|---|---|---|
| 학습 구조 | 여러 모델을 비교적 독립적으로 학습 | 이전까지의 예측을 이용하여 순차적으로 학습 |
| 주요 아이디어 | 다양한 예측을 결합하여 변동 완화 | 남아 있는 오류를 보완하여 표현력 향상 |
| 편향-분산 관점 | 주로 분산 감소 | 편향 감소에 도움이 될 수 있으나 과도한 학습 시 분산 증가 가능 |
| 대표 모델 | Random Forest | GBM, XGBoost |
| 조절할 요소 | Tree 깊이, Feature 후보 수 등 | 학습률, Tree 수, 깊이, 규제 등 |

이 비교는 일반적인 특성이다. Boosting이 항상 Bagging보다 우수하거나, Bagging에는 과적합이 발생하지 않는다는 뜻은 아니다.

---

## 3. 모델 선택에는 성능과 제약 조건이 함께 필요하다

모델을 비교할 때는 동일한 Train 데이터로 학습하고, 동일한 Validation 데이터와 평가 지표를 사용해야 한다.

다음은 선택 기준을 설명하기 위한 결과 예시이다.

| 모델 | Train F1 | Validation F1 | 학습 시간 |
|---|---:|---:|---:|
| Random Forest | 0.90 | 0.86 | 2초 |
| XGBoost | 0.91 | 0.87 | 8초 |

XGBoost의 Validation F1이 0.01 높지만, 이 예시에서는 학습 시간이 4배 길다.

빠른 반복 학습이 중요하다면 Random Forest를 선택할 근거가 있다. 작은 성능 개선의 가치가 큰 문제라면 XGBoost를 우선 고려할 수 있다.

선택 과정에서는 다음을 확인한다.

1. 문제의 목적과 중요한 평가 지표를 정한다.
2. 동일한 조건에서 후보 모델을 학습하고 평가한다.
3. Train과 Validation 성능의 차이를 확인한다.
4. 교차검증을 통해 분할에 따른 성능 변동을 살펴본다.
5. 학습 시간과 실제 활용 조건을 고려한다.
6. 최종 선택의 근거를 문장으로 정리한다.

Train 성능이 높다는 사실만으로 모델을 선택하면 안 된다. 학습 데이터에 얼마나 잘 맞았는지와 새로운 데이터에서 얼마나 잘 동작하는지는 구분해야 한다.

---

## 4. 기본 Feature Importance가 알려주는 것

Feature Importance는 모델이 어떤 Feature를 상대적으로 중요하게 활용했는지 나타낸다.

Random Forest에서는 Feature를 이용한 분할이 불순도를 얼마나 줄였는지 종합하여 중요도를 계산한다. 학습된 모델의 `feature_importances_`로 확인할 수 있다.

~~~python
import pandas as pd

# rf_model은 X_train으로 학습한 Random Forest라고 가정한다.
importance = pd.Series(
    rf_model.feature_importances_,
    index=X_train.columns,
).sort_values(ascending=False)

print(importance)
~~~

예를 들어 최근 미접속 기간의 중요도가 가장 높다면, 해당 모델이 전체적으로 이 Feature를 중요한 분할에 활용했다고 해석할 수 있다.

하지만 기본 중요도만으로 다음 내용까지 알 수는 없다.

- Feature 값이 높을 때 예측이 증가하는지 감소하는지
- 특정 고객에게 해당 Feature가 얼마나 기여했는지
- 모든 고객에게 같은 방향으로 작용하는지
- 해당 Feature가 실제 결과의 원인인지

중요도가 낮다고 곧바로 Feature를 삭제해서도 안 된다. 다른 Feature와 비슷한 정보를 담고 있으면 중요도가 나뉠 수 있고, 계산 방식에 따라 결과가 달라질 수도 있다.

Random Forest의 불순도 기반 중요도와 XGBoost의 gain 등 중요도는 계산 기준이 다를 수 있으므로 어떤 기준을 사용했는지 확인해야 한다.

---

## 5. Global Interpretation과 Local Interpretation

| 구분 | 해석 대상 | 예시 질문 |
|---|---|---|
| Global Interpretation | 설명 대상 데이터 전반에서의 모델 특성 | 전체 고객에서 어떤 Feature가 중요한가? |
| Local Interpretation | 특정 관측치 하나의 예측 | 고객 A는 왜 이탈할 것으로 예측되었는가? |

전체에서 중요한 Feature가 특정 고객에게도 가장 중요한 것은 아니다. 같은 Feature도 고객에 따라 예측을 높이거나 낮추는 방향으로 작용할 수 있다.

기본 Feature Importance는 주로 Global Interpretation에 해당한다. SHAP은 개별 예측의 기여도를 계산하고, 여러 관측치의 결과를 종합하여 Global Interpretation에도 사용할 수 있다.

---

## 6. SHAP의 기본 원리

**SHAP(SHapley Additive exPlanations)**은 모델의 예측을 각 Feature의 기여도로 나누어 설명하는 방법이다. 게임 이론의 **Shapley Value**를 기반으로 한다.

| 게임 이론 | 모델 해석 |
|---|---|
| 참가자 | Feature |
| 함께 만든 결과 | 모델의 예측 |
| 각 참가자의 기여도 | 각 Feature의 SHAP Value |

Feature의 기여도는 다른 Feature와 어떤 조합으로 사용되는지에 따라 달라질 수 있다. 따라서 한 Feature를 한 번 제거한 전후의 차이만 보는 것으로는 충분하지 않다.

예를 들어 A의 기여도를 계산할 때는 A가 혼자 추가된 경우, B와 함께 추가된 경우, C와 함께 추가된 경우 등 여러 조합에서의 변화가 고려된다. Shapley Value는 이러한 추가 기여를 적절한 가중치로 종합한다.

가능한 조합은 Feature 수가 늘수록 많아진다. 실제 계산에서는 모델에 맞는 알고리즘이나 근사 방법을 이용한다.

---

## 7. Base Value와 SHAP Value의 관계

**Base Value**는 설명의 출발점이 되는 모델의 기준 출력값이다. 설명에 사용한 기준 분포와 설정에 따라 정해지는 값이며, 단순히 실제 타깃의 평균과 항상 같다고 생각하면 안 된다.

SHAP은 이 기준값에서 현재 예측까지의 차이를 Feature별 기여도로 나눈다.

$$
f(x)=\phi_0+\sum_{j=1}^{p}\phi_j(x)
$$

- \(f(x)\): 설명 대상 모델 출력값
- \(\phi_0\): Base Value
- \(\phi_j(x)\): 해당 관측치에서 j번째 Feature의 SHAP Value
- \(p\): Feature 수

| SHAP Value | 해석 |
|---|---|
| 양수 | 설명하는 모델 출력값을 높이는 방향으로 기여 |
| 음수 | 설명하는 모델 출력값을 낮추는 방향으로 기여 |
| 절댓값이 큼 | 해당 예측에서 기여의 크기가 큼 |

설명을 단순화하여 **확률 스케일에서 계산된 SHAP**을 가정하면 다음처럼 해석할 수 있다.

| 항목 | 값 |
|---|---:|
| Base Value | 0.40 |
| 최근 미접속 기간의 기여 | +0.25 |
| 문의 횟수의 기여 | +0.15 |
| 가입 기간의 기여 | -0.10 |
| 월 이용 금액의 기여 | +0.05 |
| 최종 출력 | 0.75 |

$$
0.40+0.25+0.15-0.10+0.05=0.75
$$

이 예시에서 최근 미접속 기간은 출력을 가장 크게 높였고, 가입 기간은 낮추는 방향으로 작용했다.

---

## 8. 모델 출력의 스케일을 먼저 확인하기

SHAP Value가 항상 확률의 변화량인 것은 아니다. 회귀의 예측값, 분류의 확률, raw score 또는 log-odds 등 설명하는 출력에 따라 단위가 달라진다.

예를 들어 이진 XGBoost 모델의 raw margin을 설명한다면 Base Value와 SHAP Value의 합은 log-odds 스케일이다. 이때 `SHAP Value=+0.30`을 이탈 확률이 30%p 증가했다고 해석하면 안 된다.

raw margin \(z\)를 확률로 변환하는 관계는 다음과 같다.

$$
P(y=1\mid x)=\frac{1}{1+e^{-z}}
$$

이 변환은 비선형이므로 개별 SHAP Value를 각각 확률로 바꾼 뒤 더하는 방식도 적절하지 않다.

해석 전에는 다음을 확인한다.

- 어떤 클래스 또는 타깃의 출력을 설명하는가?
- 출력 스케일이 무엇인가?
- Base Value와 기여도의 합이 어떤 모델 출력과 대응하는가?

---

## 9. SHAP Global Importance: 평균 절대 기여도

개별 관측치의 SHAP Value를 종합하면 설명 대상 데이터에서 어떤 Feature가 전반적으로 큰 영향을 주었는지 확인할 수 있다.

대표적인 계산은 Feature별 **평균 절대 SHAP Value**이다.

$$
I_j=\frac{1}{N}\sum_{i=1}^{N}|\phi_j(x_i)|
$$

절댓값을 사용하는 이유는 양수와 음수의 기여가 서로 상쇄되는 것을 막기 위해서다. 어떤 Feature가 두 고객에게 +0.4와 -0.4로 기여했다면 단순 평균은 0이지만, 평균 절대값은 0.4이다.

최근 미접속 기간의 SHAP Value가 +0.30, -0.25, +0.20이라면 다음과 같다.

$$
\frac{0.30+0.25+0.20}{3}=0.25
$$

Global Importance는 **크기**를 종합한 값이다. 값이 높다고 모든 고객의 예측을 높이는 방향이라는 뜻은 아니다. 또한 어떤 고객 집합을 설명하는지에 따라 중요도 순위가 달라질 수 있다.

---

## 10. Bar Plot과 Beeswarm Plot 읽기

### Bar Plot

여러 관측치에 대한 SHAP Bar Plot은 Feature별 평균 절대 기여도를 비교한다.

- 막대가 길수록 설명 대상 데이터에서 평균적으로 영향의 크기가 크다.
- 전체적으로 중요한 Feature를 비교하기에 적합하다.
- Feature 값이 높을 때 기여가 양수인지 음수인지는 막대 길이만으로 알 수 없다.

### Beeswarm Plot

Beeswarm Plot은 중요도뿐 아니라 관측치별 기여 분포와 Feature 값의 관계를 보여준다.

| 요소 | 의미 |
|---|---|
| 세로축 | Feature |
| 점 하나 | 한 관측치의 해당 Feature에 대한 기여 |
| 가로축 | SHAP Value |
| 오른쪽 위치 | 모델 출력을 높이는 기여 |
| 왼쪽 위치 | 모델 출력을 낮추는 기여 |
| 점의 색 | 해당 Feature의 실제 값이 상대적으로 높은지 낮은지 |

일반적인 색상에서는 높은 Feature 값이 빨간색, 낮은 값이 파란색으로 표시된다.

최근 미접속 기간의 높은 값들이 주로 오른쪽에 있다면 다음과 같이 정리할 수 있다.

> 현재 모델과 설명 대상 데이터에서는 최근 미접속 기간이 긴 고객들의 해당 Feature가 모델 출력을 높이는 방향으로 기여하는 경향이 있다.

이것은 관찰된 경향이며, 모든 고객에게 반드시 같은 방향으로 작용하거나 실제 이탈의 원인이라는 의미는 아니다.

---

## 11. SHAP 계산과 전체 모델 해석 코드

다음은 수치형 Feature의 이진 분류 모델을 학습하고 SHAP을 계산하는 예시이다. 입력 데이터 `X`는 DataFrame, `y`는 0과 1로 구성된 타깃이라고 가정한다. 실행에는 `xgboost`와 `shap`이 필요하다.

~~~python
import numpy as np
import pandas as pd
import shap
from xgboost import XGBClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import f1_score

X_train, X_valid, y_train, y_valid = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)

model = XGBClassifier(
    n_estimators=100,
    learning_rate=0.1,
    max_depth=3,
    objective="binary:logistic",
    random_state=42,
)
model.fit(X_train, y_train)

print("Validation F1:", f1_score(
    y_valid, model.predict(X_valid)
))

# raw 출력을 명시하여 설명할 스케일을 고정한다.
explainer = shap.TreeExplainer(model, model_output="raw")
shap_values = explainer(X_valid)

importance = pd.DataFrame({
    "Feature": X_valid.columns,
    "mean_abs_shap": np.abs(shap_values.values).mean(axis=0),
}).sort_values("mean_abs_shap", ascending=False)

print(importance)
shap.plots.bar(shap_values)
shap.plots.beeswarm(shap_values)
~~~

이 예시는 이진 XGBoost의 raw margin을 설명하므로 SHAP 기여도는 확률 자체가 아닌 log-odds 스케일이다.

기본 Feature Importance와 SHAP Global Importance의 순위는 같지 않을 수 있다. 전자는 Tree의 학습 과정에서 분할에 기여한 정도를, 후자는 설명 대상 예측에 기여한 정도를 기준으로 하기 때문이다.

---

## 12. Waterfall Plot으로 개별 예측 해석하기

Local Interpretation에서는 한 고객의 Feature 값과 SHAP Value를 함께 확인한다. 앞에서 계산한 결과에서 첫 번째 고객을 선택하는 예시이다.

~~~python
customer = shap_values[0]

local_result = pd.DataFrame({
    "Feature": X_valid.columns,
    "Feature Value": X_valid.iloc[0].values,
    "SHAP Value": customer.values,
})

base_value = float(customer.base_values)
explained_output = base_value + customer.values.sum()
raw_output = model.predict(
    X_valid.iloc[[0]], output_margin=True
)[0]

print(local_result)
print("Base Value:", base_value)
print("Base + SHAP sum:", explained_output)
print("Model raw margin:", raw_output)
print("Class 1 probability:", model.predict_proba(
    X_valid.iloc[[0]]
)[0, 1])

shap.plots.waterfall(customer)
~~~

Waterfall Plot은 Base Value에서 출발하여 각 Feature의 양수·음수 기여를 더하고 최종 출력에 도달하는 과정을 보여준다.

Feature의 실제 값과 SHAP Value는 구분해야 한다. 문의 횟수가 10회라는 것은 입력값이고, SHAP Value가 +0.30이라는 것은 해당 고객의 예측에 대한 기여도이다.

같은 문의 횟수라도 다른 Feature와의 조합이 달라지면 기여도가 달라질 수 있다. 전체에서 가장 중요한 Feature가 특정 고객에게도 가장 큰 영향을 주는 것은 아니다.

---

## 13. 해석을 분석 문장으로 정리하기

어떤 고객에서 최근 미접속 기간과 문의 횟수는 양의 기여, 가입 기간은 음의 기여를 보였다면 다음과 같이 정리할 수 있다.

> 이 고객에서는 최근 미접속 기간과 문의 횟수가 기준 출력보다 예측을 높이는 방향으로 기여했다. 반면 가입 기간은 예측을 낮추는 방향으로 기여했다. 최종 예측은 이러한 기여를 종합한 결과이다.

SHAP은 모델이 어떻게 판단했는지 설명하는 방법이며, 실제 인과관계를 증명하는 방법이 아니다.

문의 횟수의 중요도가 높더라도 문의를 줄이면 실제 이탈이 감소한다고 결론 내릴 수 없다. 정책이나 서비스 개선의 효과를 판단하려면 별도의 검증이 필요하다.

---

## 14. 핵심 정리

- Bagging은 다양한 모델의 예측을 결합하여 주로 분산을 줄인다.
- Boosting은 남아 있는 오류를 순차적으로 보완하여 편향을 줄이는 데 도움이 될 수 있지만 과적합도 가능하다.
- 모델은 동일한 평가 기준에서 비교하고, 검증 성능과 학습 시간, 문제 목적을 함께 고려하여 선택한다.
- 기본 Feature Importance는 전체적인 중요도를 보여주지만 개별 예측의 방향과 크기를 직접 설명하지 못한다.
- Global Interpretation은 전체 모델을, Local Interpretation은 특정 관측치의 예측을 해석한다.
- SHAP은 여러 Feature 조합에서의 기여를 고려하는 Shapley Value를 기반으로 한다.
- Base Value와 각 Feature의 SHAP Value를 합하면 설명 대상 출력값에 연결된다.
- SHAP Value의 부호는 기여 방향, 절댓값은 기여 크기를 나타낸다.
- 평균 절대 SHAP Value로 Global Importance를 계산할 수 있다.
- Bar Plot은 전체 중요도, Beeswarm Plot은 기여 분포와 Feature 값의 관계, Waterfall Plot은 개별 예측의 구성 과정을 보여준다.
- SHAP을 해석하기 전에 출력의 클래스와 스케일을 확인해야 한다.
- SHAP은 모델 예측의 기여도를 설명하며 실제 인과관계를 증명하지 않는다.

---

## 오늘의 한 문장

> 모델 선택은 성능과 활용 조건을 비교하는 과정이고, SHAP 해석은 기준 출력에서 최종 예측까지 각 Feature가 어느 방향으로 얼마나 기여했는지 설명하는 과정이다.