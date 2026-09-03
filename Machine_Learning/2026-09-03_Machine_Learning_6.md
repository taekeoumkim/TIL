# 2026-09-03 회귀 모델 비교와 하이퍼파라미터 튜닝

## 학습 목표

- 단일 Train/Test 분할만으로 모델을 평가할 때의 한계를 설명할 수 있다.
- K-Fold 교차검증의 구조와 수행 과정을 이해할 수 있다.
- `cross_val_score()`를 사용하여 회귀 모델의 교차검증 MSE를 계산할 수 있다.
- 파라미터와 하이퍼파라미터의 차이를 설명할 수 있다.
- `GridSearchCV`로 하이퍼파라미터 후보를 탐색하고 최적 모델을 선택할 수 있다.
- 교차검증 결과와 별도로 보관한 Test Set을 이용해 최종 일반화 성능을 평가할 수 있다.

---

## 1. 한 번의 평가만으로 모델을 판단하기 어려운 이유

한 번의 시험 점수만으로 사람의 실력을 판단하면 그날의 컨디션이나 운에 따라 잘못 평가할 수 있다. 머신러닝 모델도 마찬가지이다.

전체 데이터를 Train Set과 Test Set으로 한 번만 나누면, 어떤 데이터가 각 집합에 포함되었는지에 따라 평가 결과가 달라질 수 있다.

- 우연히 예측하기 쉬운 데이터가 Test Set에 많으면 성능이 실제보다 높게 측정될 수 있다.
- 어려운 데이터가 Test Set에 많으면 성능이 실제보다 낮게 측정될 수 있다.
- 데이터의 수가 적을수록 한 번의 분할 결과에 더 크게 영향을 받을 수 있다.

따라서 단일 평가 결과만으로는 모델이 새로운 데이터에서도 잘 동작하는지, 즉 **일반화 성능**을 안정적으로 판단하기 어렵다.

---

## 2. 데이터를 Train, Validation, Test로 구분하는 이유

머신러닝 프로젝트에서는 각 데이터 집합의 역할을 구분해야 한다.

| 데이터 | 역할 | 사용 시점 |
|---|---|---|
| Train Set | 모델의 파라미터를 학습 | 모델 학습 과정 |
| Validation Set | 모델과 하이퍼파라미터를 비교·선택 | 모델 선택 과정 |
| Test Set | 선택이 끝난 모델의 최종 일반화 성능 확인 | 마지막에 한 번 |

K-Fold 교차검증을 사용할 때는 전체 데이터를 먼저 Train Set과 Test Set으로 분리한다. 이후 **Train Set 내부에서만** 여러 Validation Fold를 만들어 모델을 비교한다.

```text
전체 데이터
├── Train Set
│   └── K-Fold 교차검증: 학습 및 모델 선택
└── Test Set
    └── 선택된 최종 모델을 마지막에 한 번 평가
```

Test Set을 하이퍼파라미터 선택 과정에서 반복해서 사용하면 Test Set의 정보에 맞춰진 모델을 선택할 수 있다. 그러면 Test 성능이 더 이상 공정한 최종 평가 결과가 되기 어렵다.

---

## 3. K-Fold 교차검증(Cross-Validation)

K-Fold 교차검증은 Train Set을 비슷한 크기의 K개 Fold로 나눈 뒤, 한 Fold를 Validation 데이터로 사용하고 나머지 Fold를 Train 데이터로 사용하여 모델을 평가하는 방법이다.

이 과정을 K번 반복하면 모든 Fold가 한 번씩 Validation 역할을 맡는다. 마지막에는 K개의 검증 성능을 평균하여 모델의 성능을 평가한다.

### 5-Fold 교차검증의 흐름

| 반복 | 학습에 사용하는 Fold | Validation Fold |
|---:|---|---|
| 1 | 2, 3, 4, 5 | 1 |
| 2 | 1, 3, 4, 5 | 2 |
| 3 | 1, 2, 4, 5 | 3 |
| 4 | 1, 2, 3, 5 | 4 |
| 5 | 1, 2, 3, 4 | 5 |

```text
Train Set을 5개 Fold로 분할
    ↓
4개 Fold로 학습 + 1개 Fold로 검증
    ↓
Validation Fold를 바꾸며 총 5회 반복
    ↓
5개 검증 점수의 평균으로 모델 평가
```

### 장점

- 한 번의 데이터 분할에 따른 평가 편차를 줄일 수 있다.
- 모든 Train 데이터가 학습과 검증에 사용된다.
- 모델의 일반화 성능을 더 안정적으로 추정할 수 있다.
- 잘못된 모델이나 하이퍼파라미터를 선택할 가능성을 낮출 수 있다.

### 단점

- K번 모델을 학습하므로 단일 분할보다 계산 시간이 오래 걸린다.
- K가 커지면 더 많은 학습이 필요하다.

즉, K-Fold 교차검증은 추가 계산 비용을 사용하여 더 신뢰할 수 있는 평가 결과를 얻는 방법이다.

---

## 4. 당뇨병 데이터로 5-Fold 교차검증 수행하기

scikit-learn의 당뇨병 데이터를 사용하여 질병 진행 정도를 예측하는 선형회귀 모델을 평가한다.

```python
import numpy as np
from sklearn.datasets import load_diabetes
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split, cross_val_score

# 데이터 불러오기
data = load_diabetes()
X = data.data
y = data.target

# 전체 데이터의 20%를 최종 평가용 Test Set으로 분리
X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42,
)

model = LinearRegression()

# Train Set에서만 5-Fold 교차검증 수행
scores = cross_val_score(
    model,
    X_train,
    y_train,
    cv=5,
    scoring="neg_mean_squared_error",
)

# 음수 점수를 원래 MSE로 변환
mse_scores = -scores

print("각 Fold의 MSE:", np.round(mse_scores, 2))
print("평균 MSE:", round(mse_scores.mean(), 2))
```

실행 결과:

```text
각 Fold의 MSE: [2759.44 3473.36 2713.08 3192.30 3576.90]
평균 MSE: 3143.02
```

### 코드 해석

- `test_size=0.2`: 전체 데이터의 20%를 최종 Test Set으로 분리한다.
- `random_state=42`: 실행할 때마다 같은 방식으로 데이터가 분할되도록 한다.
- `cv=5`: Train Set을 5개의 Fold로 나누어 다섯 번 학습하고 검증한다.
- `scoring="neg_mean_squared_error"`: MSE를 기준으로 모델을 평가한다.

### 왜 MSE 점수에 음수를 붙일까?

scikit-learn의 모델 선택 도구는 기본적으로 **점수가 클수록 좋은 모델**이라는 규칙을 사용한다. 하지만 MSE는 값이 작을수록 좋은 지표이다.

그래서 scikit-learn은 MSE에 음수를 붙인 `neg_mean_squared_error`를 제공한다.

```text
실제 MSE가 3000
→ scikit-learn 점수는 -3000

-2800 > -3200
→ -2800에 해당하는 MSE 2800이 더 좋은 성능
```

사람이 이해하는 원래 MSE로 확인하려면 반환된 값에 `-`를 붙인다.

```python
mse_scores = -scores
```

---

## 5. 교차검증 결과를 해석하는 방법

평균 점수만 보는 것보다 각 Fold의 점수도 함께 확인하는 것이 좋다.

- 평균 MSE가 작을수록 전반적인 예측 오차가 작다.
- Fold별 MSE 차이가 작으면 데이터 분할이 달라져도 성능이 비교적 안정적이다.
- Fold별 MSE 차이가 크면 모델 성능이 데이터 구성에 민감할 수 있다.

이번 결과에서는 가장 작은 Fold MSE가 약 `2713.08`, 가장 큰 값이 약 `3576.90`이다. 평균 MSE는 약 `3143.02`이지만, 분할에 따라 어느 정도 성능 차이가 있다는 점도 함께 확인할 수 있다.

교차검증 점수는 최종 Test 점수가 아니라 **Train Set 안에서 모델을 선택하기 위한 검증 결과**라는 점에 주의해야 한다.

---

## 6. 파라미터와 하이퍼파라미터

### 파라미터(Parameter)

파라미터는 모델이 학습 데이터를 통해 자동으로 찾는 값이다.

예를 들어 선형회귀식에서 Weight와 Bias가 파라미터에 해당한다.

$$
\hat{y}=wx+b
$$

여기서 $w$와 $b$는 모델 학습 과정에서 결정된다.

### 하이퍼파라미터(Hyperparameter)

하이퍼파라미터는 모델을 학습하기 전에 사람이 설정하는 값이다. 모델의 학습 방법이나 구조, 복잡도 등에 영향을 준다.

예시는 다음과 같다.

- 경사하강법의 Learning Rate
- Ridge와 Lasso의 정규화 강도 `alpha`
- K-Fold 교차검증의 Fold 수 `cv`

| 구분 | 파라미터 | 하이퍼파라미터 |
|---|---|---|
| 결정 주체 | 모델이 데이터로부터 학습 | 사람이 학습 전에 설정 |
| 역할 | 입력을 예측값으로 변환 | 학습 방식이나 모델 특성을 조절 |
| 예시 | Weight, Bias | Learning Rate, `alpha`, Fold 수 |

어떤 하이퍼파라미터가 좋은지는 데이터를 보기 전에 정확히 알기 어렵기 때문에 여러 후보를 검증하여 선택해야 한다.

---

## 7. GridSearchCV

`GridSearchCV`는 scikit-learn에서 제공하는 하이퍼파라미터 탐색 도구이다.

사용자가 지정한 하이퍼파라미터 후보의 모든 조합을 하나씩 적용하고, 각 조합을 교차검증으로 평가한다. 이후 평균 검증 성능이 가장 좋은 조합을 자동으로 선택한다.

```text
하이퍼파라미터 후보 정의
    ↓
각 후보 조합마다 K-Fold 교차검증
    ↓
평균 검증 성능 비교
    ↓
최적 하이퍼파라미터와 모델 선택
    ↓
보관해 둔 Test Set으로 최종 평가
```

### Grid Search의 특징

- 지정한 모든 후보 조합을 체계적으로 평가한다.
- 사람이 후보를 하나씩 바꾸어 실행하는 작업을 자동화한다.
- 교차검증을 사용해 각 조합을 같은 기준으로 비교한다.
- 후보와 조합이 많아지면 학습 횟수와 계산 비용이 빠르게 증가한다.

예를 들어 `alpha` 후보가 5개이고 `cv=5`라면 후보마다 5-Fold 교차검증을 수행하므로 기본적으로 `5 × 5 = 25`회의 학습이 필요하다. 이후 최적 설정으로 전체 Train Set에 모델을 다시 학습한다.

---

## 8. Ridge의 최적 alpha 탐색하기

Ridge 회귀에서 `alpha`는 L2 정규화의 강도를 결정하는 하이퍼파라미터이다.

- `alpha`가 작으면 정규화가 약하다.
- `alpha`가 크면 가중치를 더 강하게 제한한다.
- 너무 작으면 Overfitting을 충분히 줄이지 못할 수 있다.
- 너무 크면 모델이 지나치게 단순해져 Underfitting이 발생할 수 있다.

```python
from sklearn.datasets import load_diabetes
from sklearn.linear_model import Ridge
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split, GridSearchCV

# 데이터 불러오기
data = load_diabetes()
X = data.data
y = data.target

# Test Set은 최종 평가를 위해 따로 보관
X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42,
)

model = Ridge()

# 탐색할 하이퍼파라미터 후보
param_grid = {
    "alpha": [0.01, 0.1, 1, 10, 100],
}

# 각 alpha를 5-Fold 교차검증으로 평가
grid_search = GridSearchCV(
    model,
    param_grid,
    cv=5,
    scoring="neg_mean_squared_error",
)

grid_search.fit(X_train, y_train)

print("최적의 alpha:", grid_search.best_params_)
print("최적의 교차검증 MSE:", -grid_search.best_score_)

# 가장 좋은 설정으로 학습된 모델 가져오기
best_model = grid_search.best_estimator_

# 별도로 보관한 Test Set으로 마지막 평가
test_pred = best_model.predict(X_test)
test_mse = mean_squared_error(y_test, test_pred)

print("Test MSE:", test_mse)
```

실행 결과:

```text
최적의 alpha: {'alpha': 0.1}
최적의 교차검증 MSE: 3125.19
Test MSE: 2856.49
```

### 주요 속성

| 속성 | 의미 |
|---|---|
| `best_params_` | 가장 좋은 평균 교차검증 성능을 기록한 하이퍼파라미터 |
| `best_score_` | 최적 조합의 평균 교차검증 점수 |
| `best_estimator_` | 최적 하이퍼파라미터로 학습된 모델 |
| `cv_results_` | 모든 후보의 평가 결과와 학습 시간 등의 정보 |

`scoring="neg_mean_squared_error"`를 사용했기 때문에 `best_score_`도 음수 MSE이다. 원래 MSE를 확인하려면 `-grid_search.best_score_`로 변환해야 한다.

---

## 9. alpha 후보별 결과 비교

모든 후보의 평균 교차검증 MSE는 `cv_results_`에서 확인할 수 있다.

```python
for alpha, score in zip(
    grid_search.cv_results_["param_alpha"],
    grid_search.cv_results_["mean_test_score"],
):
    print(f"alpha={alpha}: CV MSE={-score:.2f}")
```

실행 결과:

| alpha | 평균 CV MSE |
|---:|---:|
| 0.01 | 3131.98 |
| 0.1 | 3125.19 |
| 1 | 3639.40 |
| 10 | 5303.70 |
| 100 | 6025.02 |

후보 중 `alpha=0.1`의 평균 MSE가 가장 작으므로 최적값으로 선택된다. 이 데이터에서는 `alpha`가 지나치게 커질수록 정규화가 너무 강해져 검증 오차가 증가했다.

다만 최적값은 데이터와 전처리 방법, 후보 범위에 따라 달라지므로 `0.1`이 모든 Ridge 문제에서 좋은 값이라는 뜻은 아니다.

---

## 10. 올바른 모델 선택과 최종 평가 과정

모델 비교와 하이퍼파라미터 튜닝은 다음 순서로 진행한다.

```text
1. 전체 데이터를 Train Set과 Test Set으로 분리
2. Train Set에서 K-Fold 교차검증 수행
3. 여러 모델 또는 하이퍼파라미터의 평균 검증 성능 비교
4. 가장 좋은 설정 선택
5. 선택된 모델을 Train Set 전체로 학습
6. Test Set으로 최종 성능을 한 번 평가
```

### 주의할 점

- Test Set으로 하이퍼파라미터를 선택하지 않는다.
- 비교하는 모든 후보에 같은 교차검증 기준과 평가 지표를 사용한다.
- 회귀 문제에서 MSE는 작을수록 좋지만 `neg_mean_squared_error`는 클수록 좋다.
- 교차검증 평균뿐 아니라 Fold별 성능의 차이도 함께 살펴본다.
- 최적 결과가 탐색 범위의 끝값이라면 후보 범위를 다시 검토할 수 있다.
- 가장 낮은 오차뿐 아니라 계산 비용과 모델의 안정성도 고려한다.

---

## 11. 단일 분할, 교차검증, GridSearchCV 비교

| 방법 | 목적 | 장점 | 한계 |
|---|---|---|---|
| 단일 Train/Test 분할 | 모델을 한 번 학습하고 평가 | 빠르고 간단함 | 분할 결과에 따라 성능이 달라질 수 있음 |
| K-Fold 교차검증 | 모델의 일반화 성능을 안정적으로 추정 | 분할의 영향을 줄이고 모든 Train 데이터를 활용 | K번 학습하므로 계산 비용 증가 |
| GridSearchCV | 최적 하이퍼파라미터 조합 탐색 | 후보 조합을 교차검증으로 자동 비교 | 후보 조합이 많으면 계산량이 크게 증가 |

세 방법은 완전히 대체 관계가 아니다. 일반적으로 전체 데이터를 한 번 Train/Test로 분리하고, Train Set에서 K-Fold 기반 `GridSearchCV`를 수행한 뒤, 선택된 모델을 Test Set으로 최종 평가한다.

---

## 12. 오늘 배운 내용 정리

### K-Fold 교차검증

- 단일 데이터 분할은 어떤 데이터가 Test Set에 포함되는지에 따라 결과가 달라질 수 있다.
- K-Fold는 Train Set을 K개로 나누고 각 Fold를 한 번씩 Validation 데이터로 사용한다.
- K개의 검증 점수를 평균하면 모델의 일반화 성능을 더 안정적으로 추정할 수 있다.
- Test Set은 처음에 분리해 두고 최종 평가에서만 사용한다.

### 하이퍼파라미터

- 모델 학습 전에 사람이 설정하는 값이다.
- Ridge의 `alpha`와 경사하강법의 Learning Rate 등이 해당한다.
- 데이터에 적절한 값은 검증 결과를 통해 선택해야 한다.

### GridSearchCV

- 지정한 모든 하이퍼파라미터 조합을 교차검증으로 평가한다.
- `best_params_`로 최적 설정을 확인할 수 있다.
- `best_estimator_`로 선택된 모델을 가져올 수 있다.
- 선택이 끝난 모델은 별도로 보관한 Test Set으로 최종 평가한다.

### 전체 학습 흐름

```text
전체 데이터 분할
    ↓
Train Set에서 교차검증
    ↓
GridSearchCV로 후보 비교
    ↓
최적 하이퍼파라미터와 모델 선택
    ↓
Test Set으로 최종 일반화 성능 확인
```

---

## 13. 느낀 점

오늘은 모델의 성능을 한 번 측정한 결과만 믿으면 데이터가 우연히 어떻게 나뉘었는지에 따라 잘못된 결론을 내릴 수 있다는 점을 배웠다. K-Fold 교차검증은 모델을 여러 번 학습해야 해서 시간이 더 필요하지만, 모든 Fold를 검증 데이터로 사용해 성능을 더 안정적으로 판단할 수 있다는 점이 중요하게 느껴졌다.

또한 파라미터는 모델이 데이터에서 학습하는 값이고 하이퍼파라미터는 학습 전에 사람이 정하는 값이라는 차이를 명확히 이해했다. 하이퍼파라미터를 단순한 감으로 선택하는 것이 아니라, `GridSearchCV`를 이용해 각 후보를 동일한 교차검증 기준으로 비교할 수 있다는 점이 인상적이었다.

무엇보다 Test Set을 모델 선택에 사용하지 않고 마지막까지 따로 보관해야 한다는 점을 기억해야겠다. Test Set을 반복해서 확인하며 설정을 바꾸면 결국 Test 데이터에 맞는 모델을 선택하게 되어 실제 일반화 성능을 과대평가할 수 있기 때문이다.

앞으로 회귀 프로젝트에서는 먼저 평가 지표와 데이터 분할 방식을 정하고, Train Set 안에서 교차검증과 하이퍼파라미터 튜닝을 수행한 뒤, 최종 모델만 Test Set으로 평가하는 순서를 지켜야겠다. 또한 평균 교차검증 점수뿐 아니라 Fold별 점수의 차이도 확인하여 모델의 안정성까지 판단하는 습관을 들여야겠다.
