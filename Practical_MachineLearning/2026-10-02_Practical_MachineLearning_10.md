# 2026-10-02 배깅과 랜덤포레스트

## 학습 목표

- 앙상블 학습이 여러 모델의 예측을 결합하는 방법임을 설명할 수 있다.
- Bootstrap Sampling과 Bagging의 원리를 이해한다.
- 분류와 회귀에서 여러 모델의 예측을 결합하는 방식을 구분할 수 있다.
- Random Forest가 데이터와 Feature에 무작위성을 추가하는 이유를 설명할 수 있다.
- 여러 Tree의 예측을 결합했을 때 분산이 감소하는 원리를 이해한다.
- OOB Score와 Feature Importance의 의미와 해석 시 주의점을 이해한다.

---

## 1. 하나의 Decision Tree가 불안정할 수 있는 이유

Decision Tree는 Feature를 기준으로 데이터를 반복해서 나누며 예측한다. 그런데 학습 데이터가 조금만 달라져도 첫 번째 분할 기준이나 이후 분할 구조가 달라질 수 있다.

고객 이탈 데이터를 예로 들면, 일부 고객이 학습 데이터에 추가되거나 빠지는 것만으로도 Tree가 선택하는 조건과 예측 결과가 달라질 수 있다.

이처럼 학습 데이터의 변화에 모델이 민감하게 반응하는 특성을 **분산(Variance)이 크다**고 표현한다. 여기서 분산은 입력 Feature 값의 분산이 아니라, 학습 데이터가 달라질 때 모델의 예측이 얼마나 달라지는지를 뜻한다.

특히 깊게 성장한 Tree는 학습 데이터의 세부적인 패턴과 노이즈까지 반영하여 Overfitting이 발생할 수 있다. 이를 완화하는 방법 중 하나가 서로 다른 여러 Tree를 학습하고 예측을 결합하는 것이다.

---

## 2. 앙상블 학습(Ensemble Learning)

앙상블 학습은 여러 모델의 예측 결과를 결합하여 하나의 최종 예측을 만드는 방법이다.

예를 들어 세 개의 Tree가 한 고객의 이탈 여부를 다음과 같이 예측했다고 하자.

| 모델 | 예측 |
|---|---|
| Tree 1 | 이탈 |
| Tree 2 | 이탈하지 않음 |
| Tree 3 | 이탈 |

다수결로 결합하면 최종 예측은 이탈이다. 여러 판단을 함께 이용하면 특정 Tree 하나의 예측에 의존하는 정도를 줄일 수 있다.

다만 모델의 개수가 많다는 것만으로 충분하지는 않다. 모든 모델이 항상 같은 예측을 한다면 결과를 합쳐도 하나의 모델과 큰 차이가 없다.

**앙상블의 효과를 높이려면 개별 모델이 유용한 예측을 하면서도 서로 다른 방식으로 실수할 수 있어야 한다.**

---

## 3. Bootstrap Sampling: 중복을 허용하는 표본 추출

Bootstrap Sampling은 원본 데이터에서 **중복을 허용하는 복원추출**로 표본을 만드는 방법이다.

복원추출은 한 번 선택한 데이터를 다음 추출에서도 다시 선택할 수 있다는 뜻이다. 일반적인 배깅에서는 원본 학습 데이터와 같은 크기의 Bootstrap Sample을 생성한다.

원본 데이터가 A, B, C, D, E, F의 6개라면 다음과 같은 표본을 만들 수 있다.

| 데이터 | 구성 |
|---|---|
| 원본 | A, B, C, D, E, F |
| Sample 1 | A, B, B, D, E, F |
| Sample 2 | A, A, C, D, F, F |
| Sample 3 | B, C, C, D, E, E |

각 Sample은 6개이지만 고유한 관측치의 수는 6개보다 적을 수 있다. 같은 관측치가 여러 번 포함되기도 하고, 어떤 관측치는 전혀 포함되지 않기도 한다.

다음 코드는 복원추출의 동작을 보여주는 예시이다.

~~~python
import numpy as np

original = np.array(["A", "B", "C", "D", "E", "F"])
rng = np.random.default_rng(42)

for i in range(3):
    sample = rng.choice(
        original,
        size=len(original),
        replace=True,
    )
    print(f"Sample {i + 1}:", sample)
~~~

`replace=True`가 중복 선택을 허용하는 설정이다. 이 과정을 반복하면 서로 다른 학습 표본을 만들 수 있다.

Bootstrap Sampling은 기존 관측치를 다시 뽑는 방법이며, 새로운 정보를 가진 관측치를 생성하는 방법은 아니다.

---

## 4. Bagging: 표본을 다르게 만들고 예측을 결합하기

**Bagging**은 **Bootstrap Aggregating**의 줄임말이다.

- Bootstrap: 복원추출로 여러 학습 표본을 만든다.
- Aggregating: 각 표본으로 학습한 모델의 예측을 결합한다.

배깅의 기본 과정은 다음과 같다.

1. 원본 학습 데이터에서 여러 Bootstrap Sample을 만든다.
2. 각 Sample로 개별 모델을 학습한다.
3. 새로운 데이터에 대해 각 모델이 예측한다.
4. 여러 예측을 결합하여 최종 결과를 만든다.

개별 모델은 앞 모델의 결과를 수정하는 방식이 아니라, 각각의 표본으로 독립적으로 학습된다. 따라서 모델 학습을 병렬로 수행하기에도 적합하다.

### 분류에서의 결합

분류에서는 클래스 예측의 다수결이나 클래스 확률의 평균을 사용할 수 있다.

예를 들어 세 모델의 예측이 이탈, 유지, 이탈이면 다수결 결과는 이탈이다. 확률을 결합하는 경우에는 각 모델의 클래스별 예측 확률을 평균한 뒤 최종 클래스를 선택한다.

### 회귀에서의 결합

회귀에서는 일반적으로 예측값을 평균한다.

$$
\hat{y} = \frac{1}{M}\sum_{m=1}^{M}\hat{y}_m
$$

- \(M\): 모델의 개수
- \(\hat{y}_m\): m번째 모델의 예측값

세 Tree의 예측값이 32, 38, 35라면 최종 예측은 다음과 같다.

$$
\hat{y} = \frac{32+38+35}{3}=35
$$

하나의 Tree가 상대적으로 높거나 낮게 예측하더라도 다른 Tree의 예측과 평균하면서 그 영향이 줄어들 수 있다.

---

## 5. Random Forest와 Feature Randomness

Bootstrap Sampling으로 학습 표본을 다르게 만들어도 Tree들이 비슷한 구조를 가질 수 있다. 예측력이 강한 Feature가 있으면 여러 Tree가 같은 Feature를 반복적으로 선택할 수 있기 때문이다.

**Random Forest**는 여러 Decision Tree를 결합하면서, Bootstrap Sampling에 **Feature Randomness**를 추가한다.

각 Tree의 노드에서 분할을 결정할 때 전체 Feature를 모두 후보로 사용하는 대신, 무작위로 선택한 일부 Feature를 후보로 고려한다. 그 후보 안에서 좋은 분할을 찾는다.

중요한 점은 **Tree 하나를 만들 때 Feature 집합을 한 번 고정하는 것이 아니라, 각 노드의 분할에서 Feature 후보를 무작위로 선택한다**는 것이다.

이 과정은 특정 Feature가 모든 Tree를 지배하는 정도를 줄이고, Tree 사이의 예측을 더 다양하게 만드는 데 도움이 된다.

### max_features

`max_features`는 각 노드의 분할에서 고려할 Feature 수를 정하는 주요 파라미터이다.

| 설정 | 의미 |
|---|---|
| 정수 | 지정한 개수의 Feature를 후보로 고려 |
| 0과 1 사이의 실수 | 전체 Feature의 해당 비율을 후보로 고려 |
| `"sqrt"` | 전체 Feature 수의 제곱근에 해당하는 개수를 후보로 고려 |
| `None` | 전체 Feature를 후보로 고려 |

Feature가 9개일 때 `max_features="sqrt"`이면 기본적으로 각 분할에서 3개의 Feature를 후보로 고려한다.

후보 Feature 수를 줄이면 Tree 사이의 다양성이 커질 수 있지만, 개별 Tree가 좋은 분할을 찾기 어려워질 수도 있다. 따라서 무작위성을 크게 만드는 것이 항상 성능 향상으로 이어지지는 않는다.

### Decision Tree 배깅과 Random Forest 비교

| 항목 | 기본적인 Tree 배깅 | Random Forest |
|---|---|---|
| 학습 표본 | Bootstrap Sampling | Bootstrap Sampling |
| 노드별 Feature 후보 | 일반적으로 전체 Feature | 무작위로 선택한 일부 Feature |
| 다양성의 주요 원천 | 학습 표본의 차이 | 학습 표본과 Feature 후보의 차이 |
| 목적 | 여러 예측을 결합하여 분산 완화 | Tree 간 상관성도 낮추어 분산 완화 |

배깅은 다양한 모델에 적용할 수 있는 일반적인 방법이고, Random Forest는 Decision Tree와 Feature 무작위성을 사용하는 대표적인 앙상블 모델이다.

---

## 6. 여러 Tree를 결합하면 분산이 줄어드는 이유

하나의 Tree는 특정 학습 표본에 민감하게 반응할 수 있다. 서로 다른 Tree의 예측을 평균하면 개별 Tree의 변동이 일부 상쇄될 수 있다.

하지만 모든 Tree가 같은 방향으로 실수하면 평균해도 그 실수가 남는다. 따라서 Tree 개수뿐 아니라 **Tree 사이의 상관성**이 중요하다.

- Tree의 예측이 덜 비슷할수록 결합에 따른 분산 감소 효과가 커질 수 있다.
- Tree들이 거의 같은 예측을 하면 개수를 늘려도 추가 효과가 제한적이다.
- Random Forest의 Feature Randomness는 Tree 사이의 상관성을 낮추는 데 도움을 준다.

편향-분산 관점에서 Bagging은 주로 **분산을 줄이는 방법**이다. 깊은 Tree처럼 학습 데이터의 변화에 민감한 모델에 특히 유용할 수 있다.

반대로 모든 개별 모델이 중요한 패턴을 놓치고 있다면, 예측을 평균하는 것만으로 그 문제를 해결하기 어렵다. 여러 모델을 결합한다고 편향과 분산이 모두 자동으로 감소하는 것은 아니다.

---

## 7. Random Forest 구현 예시

다음은 수치형 데이터의 이진 분류에 Random Forest를 적용하는 예시이다. 실행 흐름을 확인할 수 있도록 예제 데이터를 생성한다.

~~~python
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, f1_score

X, y = make_classification(
    n_samples=1000,
    n_features=9,
    n_informative=5,
    n_redundant=2,
    random_state=42,
)

X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    stratify=y,
    random_state=42,
)

model = RandomForestClassifier(
    n_estimators=200,
    max_features="sqrt",
    bootstrap=True,
    oob_score=True,
    random_state=42,
    n_jobs=-1,
)

model.fit(X_train, y_train)
y_pred = model.predict(X_test)

print("Test Accuracy:", accuracy_score(y_test, y_pred))
print("Test F1-score:", f1_score(y_test, y_pred))
print("OOB Accuracy:", model.oob_score_)
~~~

| 파라미터 | 역할 |
|---|---|
| `n_estimators` | 학습할 Tree의 개수 |
| `max_features` | 각 노드의 분할에서 고려할 Feature 수 |
| `bootstrap` | 각 Tree의 학습에 복원추출을 사용할지 지정 |
| `oob_score` | OOB 표본을 이용한 성능 추정 여부 |
| `random_state` | 무작위 과정을 재현하기 위한 설정 |
| `n_jobs` | 병렬 처리에 사용할 작업 수. -1은 사용 가능한 모든 CPU 사용 |

scikit-learn의 RandomForestClassifier는 각 Tree의 클래스 확률을 평균하고, 평균 확률이 가장 높은 클래스를 최종 예측으로 선택한다. 따라서 원리를 설명할 때 사용하는 단순 다수결과 구현의 세부 방식은 구분할 필요가 있다.

회귀에서는 `RandomForestRegressor`를 사용하며, 개별 Tree의 예측값을 평균한다.

---

## 8. OOB Sample과 OOB Score

Bootstrap Sampling에서는 같은 관측치가 여러 번 선택될 수 있다. 그 결과 각 Tree의 학습 표본에 한 번도 포함되지 않은 관측치가 생긴다.

이를 **OOB(Out-Of-Bag) Sample**이라고 한다.

원본이 A, B, C, D, E, F이고 특정 Tree의 학습 표본이 A, B, B, D, E, F라면 C는 그 Tree의 OOB 표본이다. 다른 Tree에서는 C가 학습에 사용될 수도 있다.

OOB 평가에서는 각 관측치에 대해 **그 관측치를 학습하지 않은 Tree들만** 사용하여 예측을 결합한다. 그런 다음 원래 타깃과 비교하여 성능을 추정한다.

| 구분 | 의미 |
|---|---|
| Bootstrap Sample | 해당 Tree의 학습에 선택된 표본 |
| OOB Sample | 해당 Tree의 학습에서 선택되지 않은 표본 |
| OOB Score | OOB 예측을 결합하여 계산한 성능 추정값 |

앞의 코드에서 `bootstrap=True`, `oob_score=True`로 설정하면 학습 후 `model.oob_score_`를 확인할 수 있다. 분류 모델의 기본 OOB 점수는 Accuracy이므로 Test F1-score와 같은 지표라고 생각하면 안 된다.

OOB Score는 학습 데이터 내부에서 성능을 추정하는 데 유용하지만, 별도 Test 평가를 언제나 대신하는 것은 아니다. 특히 시간 순서나 동일 고객의 반복 관측처럼 데이터 간 의존성이 있다면 실제 예측 상황에 맞는 검증을 따로 고려해야 한다.

---

## 9. Feature Importance의 의미와 주의점

Random Forest는 Feature가 Tree의 분할에 얼마나 기여했는지 나타내는 중요도를 제공한다.

scikit-learn의 `feature_importances_`는 각 Feature가 분할을 통해 불순도를 얼마나 감소시켰는지 집계한 중요도이다. 일반적인 경우 중요도 값의 합은 1이다.

앞에서 학습한 모델의 중요도를 표로 확인할 수 있다.

~~~python
import pandas as pd

feature_names = [f"feature_{i}" for i in range(X.shape[1])]

importance = pd.Series(
    model.feature_importances_,
    index=feature_names,
).sort_values(ascending=False)

print(importance)
~~~

중요도가 높다는 것은 해당 모델의 분할에서 상대적으로 크게 활용되었다는 뜻이다. 해석할 때는 다음을 구분해야 한다.

- **인과관계가 아니다.** 중요도가 높다고 그 Feature를 바꾸면 타깃이 반드시 달라지는 것은 아니다.
- **영향의 방향을 알려주지 않는다.** 값이 커질 때 예측이 증가하는지 감소하는지는 중요도만으로 알 수 없다.
- **상관된 Feature의 중요도가 나뉠 수 있다.** 비슷한 정보를 담은 Feature가 여러 개면 하나의 Feature 중요도가 낮게 나타날 수도 있다.
- **분할 후보가 많은 Feature에 유리할 수 있다.** 고유값이 많은 Feature는 불순도 기반 중요도에서 높게 평가될 수 있다.
- **모델과 데이터에 따라 달라진다.** 중요도는 Feature 자체의 절대적인 가치가 아니라, 학습한 모델 안에서의 상대적인 값이다.

따라서 중요도가 낮다는 이유만으로 Feature를 바로 삭제하기보다는, 해당 Feature의 의미와 제거 전후의 검증 성능을 함께 확인해야 한다.

---

## 10. 핵심 정리

- Decision Tree는 학습 데이터가 달라지면 구조와 예측이 크게 달라질 수 있는 모델이다.
- 앙상블 학습은 여러 모델의 예측을 결합하여 하나의 최종 예측을 만든다.
- Bootstrap Sampling은 중복을 허용하는 복원추출 방식이다.
- Bagging은 Bootstrap Sample로 여러 모델을 학습하고 예측을 결합한다.
- 분류에서는 투표나 확률 평균, 회귀에서는 예측값 평균을 사용할 수 있다.
- Random Forest는 Bootstrap Sampling에 노드별 Feature Randomness를 추가한다.
- `max_features`는 각 분할에서 고려할 Feature 후보 수를 조절한다.
- 여러 Tree의 예측을 결합하면 분산을 줄일 수 있으며, Tree 사이의 상관성이 낮을수록 효과가 커질 수 있다.
- OOB Sample은 특정 Tree의 학습 표본에 포함되지 않은 관측치이다.
- OOB Score는 각 관측치를 학습하지 않은 Tree들의 예측으로 성능을 추정한다.
- Feature Importance는 모델의 Feature 활용 정도를 보여주지만 인과관계나 영향의 방향을 의미하지 않는다.

---

## 오늘의 한 문장

> 랜덤포레스트는 서로 다른 데이터와 Feature 후보로 여러 Tree를 학습하고 예측을 결합하여, 하나의 Tree에 의존할 때 생기는 불안정성을 줄이는 모델이다.
