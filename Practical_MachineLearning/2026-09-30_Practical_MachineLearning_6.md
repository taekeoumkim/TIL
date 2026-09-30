# 2026-09-30 TIL: 편향-분산 트레이드오프와 과적합·과소적합 진단

> 학습일: 2026-09-30  
> 주제: 모델 복잡도, 일반화 성능, 학습곡선과 검증곡선을 이용한 진단

## 학습 목표

- 과소적합과 과적합을 Train·Validation 성능으로 구분할 수 있다.
- 편향과 분산이 모델 복잡도에 따라 어떻게 달라지는지 설명할 수 있다.
- 모델별 주요 하이퍼파라미터와 복잡도의 관계를 이해할 수 있다.
- Generalization Gap을 올바르게 해석할 수 있다.
- Learning Curve로 데이터 추가의 효과를 판단할 수 있다.
- Validation Curve로 적절한 하이퍼파라미터 영역을 찾을 수 있다.

---

## 1. 편향-분산 트레이드오프와 모델 복잡도

### 1.1 과소적합, 적절한 적합, 과적합

Decision Tree의 `max_depth`를 바꾸면 모델이 데이터를 표현할 수 있는 복잡도가 달라진다.

| `max_depth` | Train Accuracy | Validation Accuracy | 해석 |
|---:|---:|---:|---|
| 1 | 0.72 | 0.70 | 너무 단순하여 과소적합 가능성 |
| 5 | 0.90 | 0.87 | 두 성능이 높고 차이가 작아 일반화가 양호 |
| 제한 없음 | 1.00 | 0.78 | Train에 지나치게 맞아 과적합 가능성 |

#### 과소적합(Underfitting)

모델이 너무 단순하여 데이터의 중요한 패턴을 충분히 학습하지 못한 상태다.

- Train 성능이 낮다.
- Validation 성능도 낮다.
- 모델 복잡도가 부족하거나 Feature가 패턴을 충분히 담지 못했을 수 있다.

#### 과적합(Overfitting)

모델이 Train 데이터의 세부적인 특징이나 잡음까지 지나치게 학습하여 새로운 데이터에서 성능이 떨어지는 상태다.

- Train 성능은 매우 높다.
- Validation 성능은 상대적으로 낮다.
- Train과 Validation 성능 차이가 크다.

#### 적절한 적합

Train 데이터의 주요 패턴을 충분히 학습하면서 Validation에서도 높은 성능을 유지하는 상태다. 머신러닝의 목적은 Train을 완벽히 맞히는 것이 아니라 학습하지 않은 데이터에서도 좋은 성능을 내는 **일반화(Generalization)**에 있다.

### 1.2 편향(Bias)과 분산(Variance)

**편향**은 모델이 실제 데이터의 패턴을 충분히 표현하지 못해 발생하는 오차와 관련된다.

```text
모델이 너무 단순함
    ↓
중요한 패턴을 충분히 표현하지 못함
    ↓
Bias 증가
    ↓
과소적합 가능성
```

**분산**은 학습 데이터가 달라졌을 때 모델의 예측이 얼마나 크게 달라지는지와 관련된다.

```text
모델이 지나치게 복잡함
    ↓
Train의 세부 특징과 잡음까지 학습
    ↓
학습 데이터가 바뀌면 예측도 크게 변함
    ↓
Variance 증가
    ↓
과적합 가능성
```

### 1.3 편향-분산 트레이드오프

일반적으로 모델 복잡도가 증가하면 Bias는 감소하지만 Variance는 증가할 수 있다. 반대로 복잡도를 낮추면 Variance는 감소하지만 Bias는 증가할 수 있다. 이 관계를 **편향-분산 트레이드오프(Bias-Variance Tradeoff)**라고 한다.

| 구분 | 과소적합 | 적절한 적합 | 과적합 |
|---|---|---|---|
| 모델 복잡도 | 낮음 | 적절함 | 높음 |
| Bias | 높음 | 균형 | 낮음 |
| Variance | 낮음 | 균형 | 높음 |
| Train 성능 | 낮음 | 높음 | 매우 높음 |
| Validation 성능 | 낮음 | 높음 | Train보다 크게 낮음 |

모델의 복잡도를 무조건 높이는 것이 목표가 아니다. 데이터의 주요 패턴을 학습하면서 새로운 데이터에서도 성능을 유지하는 균형점을 찾아야 한다.

---

## 2. 모델별 주요 하이퍼파라미터와 튜닝 방향

하이퍼파라미터마다 값이 커질 때 모델이 복잡해지는 방향이 다르다. 먼저 모델 상태를 진단하고, 각 값과 복잡도의 관계를 이해한 뒤 조절해야 한다.

### 2.1 Decision Tree

#### `max_depth`

- 작게 설정: Tree가 얕아져 단순해지고 과소적합 가능성이 증가한다.
- 크게 설정: Tree가 깊어져 복잡해지고 과적합 가능성이 증가한다.

#### `min_samples_split`

하나의 Node를 다시 나누기 위해 필요한 최소 데이터 수다.

- 작게 설정: 적은 데이터로도 분할되어 복잡도가 증가한다.
- 크게 설정: 분할이 제한되어 복잡도가 감소한다.

#### `min_samples_leaf`

최종 Leaf에 남아야 하는 최소 데이터 수다.

- 작게 설정: 작은 Leaf와 세밀한 규칙을 허용하여 복잡도가 증가한다.
- 크게 설정: 작은 Leaf 생성을 막아 복잡도가 감소한다.

| 하이퍼파라미터 | 과적합을 줄이는 방향 | 과소적합을 줄이는 방향 |
|---|---:|---:|
| `max_depth` | 감소 | 증가 |
| `min_samples_split` | 증가 | 감소 |
| `min_samples_leaf` | 증가 | 감소 |

### 2.2 Random Forest

Random Forest는 여러 Decision Tree를 학습해 결과를 결합한다. Forest의 규모와 개별 Tree의 복잡도를 함께 조절할 수 있다.

| 하이퍼파라미터 | 역할 | 해석 |
|---|---|---|
| `n_estimators` | Tree 개수 | 증가하면 예측이 안정될 수 있지만 시간·메모리 사용 증가 |
| `max_depth` | 개별 Tree의 최대 깊이 | 증가하면 개별 Tree의 복잡도 증가 |
| `min_samples_split` | Node 분할 최소 샘플 수 | 증가하면 개별 Tree 단순화 |
| `min_samples_leaf` | Leaf의 최소 샘플 수 | 증가하면 작은 Leaf 생성을 제한 |

`n_estimators`를 늘리는 것은 Tree의 수를 늘리는 것이며 개별 Tree 자체를 더 복잡하게 만드는 것은 아니다. 과적합이 의심되면 `max_depth`를 낮추거나 `min_samples_split`, `min_samples_leaf`를 높이는 방향을 우선 고려할 수 있다.

### 2.3 KNN

#### `n_neighbors`

- 작게 설정: 가까운 소수의 데이터에 크게 영향받아 결정 경계가 복잡해지고 과적합 가능성이 증가한다.
- 크게 설정: 많은 이웃을 함께 고려하여 결정 경계가 부드러워지고 과소적합 가능성이 증가한다.

따라서 과적합이면 `n_neighbors`를 늘리고, 과소적합이면 줄이는 방향을 고려한다.

#### `weights`

- `weights="uniform"`: 주변 이웃을 동일하게 반영한다.
- `weights="distance"`: 가까운 이웃에 더 큰 가중치를 부여한다.

기본적인 튜닝에서는 먼저 `n_neighbors`를 확인하고, 거리 관계의 중요성에 따라 `weights`도 비교한다.

### 2.4 Logistic Regression

#### `C`

`C`는 규제 강도와 반대 방향으로 움직인다.

- `C` 감소: 규제가 강해져 계수 크기를 더 제한하고 모델이 단순해진다.
- `C` 증가: 규제가 약해져 Train 데이터에 더 자유롭게 적합한다.

따라서 과적합이면 `C`를 낮추고, 과소적합이면 높이는 방향을 고려한다.

#### `penalty`

`L1`, `L2`, `ElasticNet` 등 어떤 규제 방식을 사용할지 지정한다.

### 2.5 Linear Regression

기본 Linear Regression에는 Tree의 깊이나 KNN의 이웃 수처럼 복잡도를 직접 조절하는 대표 하이퍼파라미터가 많지 않다. Feature를 선택하거나 Ridge, Lasso, ElasticNet과 같은 규제 회귀를 사용하고 규제 강도 `alpha`를 조절할 수 있다.

### 2.6 ARIMA

| 파라미터 | 의미 | 주의점 |
|---|---|---|
| `p` | AR에서 사용할 과거 값의 개수 | 증가하면 AR 파라미터가 늘어남 |
| `d` | 차분 횟수 | 정상성 확보에 필요한 만큼 설정 |
| `q` | MA에서 사용할 과거 오차의 개수 | 증가하면 MA 파라미터가 늘어남 |

`p`와 `q`를 지나치게 크게 설정하면 Train 데이터에 과도하게 맞을 수 있다. 반면 `d`는 과적합·과소적합을 단순 조절하는 값이 아니라 정상성을 확보하기 위한 차분 횟수다.

### 2.7 모델별 튜닝 요약

| 모델 | 주요 하이퍼파라미터 | 과적합 시 고려 | 과소적합 시 고려 |
|---|---|---|---|
| Decision Tree | `max_depth` | 감소 | 증가 |
| Decision Tree | `min_samples_split` | 증가 | 감소 |
| Decision Tree | `min_samples_leaf` | 증가 | 감소 |
| Random Forest | `n_estimators` | 안정성과 계산량 비교 | 안정성과 계산량 비교 |
| Random Forest | Tree 관련 값 | 개별 Tree 단순화 | 개별 Tree 복잡도 증가 |
| KNN | `n_neighbors` | 증가 | 감소 |
| Logistic Regression | `C` | 감소 | 증가 |
| Linear Regression | 규제·Feature | 규제 강화·Feature 정리 | 규제 완화·Feature 개선 |
| ARIMA | `p`, `q` | 과도한 차수 축소 검토 | 필요한 시차 관계 추가 검토 |
| ARIMA | `d` | 정상성을 기준으로 결정 | 정상성을 기준으로 결정 |

---

## 3. Train과 Validation 성능으로 모델 상태 진단

### 3.1 성능 수준과 차이를 함께 본다

모델 진단은 다음 두 질문에서 시작한다.

1. Train과 Validation 성능의 절대적인 수준은 어떠한가?
2. 두 성능의 차이는 얼마나 큰가?

| Train 성능 | Validation 성능 | 차이 | 진단 |
|---|---|---|---|
| 낮음 | 낮음 | 작을 수 있음 | 과소적합 가능성 |
| 높음 | 상대적으로 낮음 | 큼 | 과적합 가능성 |
| 높음 | 높음 | 작음 | 비교적 좋은 일반화 |

Train 성능이 가장 높은 모델이 최선은 아니다. 학습하지 않은 데이터에서의 Validation 성능을 함께 확인해야 한다.

### 3.2 Generalization Gap

정확도처럼 값이 높을수록 좋은 지표에서는 다음과 같이 일반화 차이를 계산할 수 있다.

$$
Generalization\ Gap=Train\ Score-Validation\ Score
$$

예를 들어 Train Accuracy가 0.95, Validation Accuracy가 0.80이라면 Gap은 0.15다. 큰 Gap은 Train에 비해 Validation 성능이 크게 낮다는 뜻이므로 과적합을 의심할 수 있다.

그러나 Gap만 비교하면 안 된다.

| 모델 | Train | Validation | Gap | 해석 |
|---|---:|---:|---:|---|
| A | 0.65 | 0.63 | 0.02 | Gap은 작지만 두 성능이 낮아 과소적합 가능성 |
| B | 0.91 | 0.88 | 0.03 | 두 성능이 높고 Gap도 작아 일반화가 양호 |

Gap의 크기보다 먼저 성능 수준을 확인하고, 그다음 차이를 봐야 한다.

```text
Train 성능 확인
    ↓
Validation 성능 확인
    ↓
두 성능의 차이 확인
    ↓
과소적합·과적합·적절한 적합 판단
```

성능이 높거나 낮다는 기준은 문제에 따라 달라진다. Baseline, 기존 모델, 비즈니스 목표, 오류 비용 등을 함께 고려해야 한다.

---

## 4. 학습곡선으로 데이터 양의 영향 확인

### 4.1 Learning Curve란?

학습곡선(Learning Curve)은 Train 데이터의 양을 증가시키면서 Train과 Validation 성능이 어떻게 달라지는지 보여준다.

```text
적은 데이터로 학습·평가
    ↓
더 많은 데이터로 다시 학습·평가
    ↓
Train/Validation 성능 변화와 수렴 수준 확인
```

데이터가 적을 때는 모델이 소수 Train 데이터의 세부 특징을 쉽게 학습하므로 Train 성능은 높고 Validation 성능은 낮을 수 있다. 데이터가 증가하면 두 성능이 점차 가까워질 수 있다.

### 4.2 과소적합의 학습곡선

- Train과 Validation 성능이 모두 낮다.
- 데이터를 추가해도 낮은 수준에서 빠르게 수렴한다.
- 데이터만 더 모으는 것으로 큰 개선을 기대하기 어렵다.

이때는 모델 복잡도를 높이고, Feature를 개선하거나 더 적합한 모델을 고려한다.

### 4.3 과적합의 학습곡선

- Train 성능은 높다.
- Validation 성능은 상대적으로 낮다.
- 두 성능 사이의 Gap이 크다.

데이터가 증가하면서 Validation 성능이 계속 좋아지고 Gap이 줄어든다면 추가 데이터가 일반화 성능 개선에 도움이 될 가능성이 있다. 다만 데이터 추가만이 유일한 해결책은 아니며 모델 복잡도를 낮추는 방법도 함께 고려한다.

### 4.4 Python으로 Learning Curve 확인하기

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import make_classification
from sklearn.model_selection import learning_curve
from sklearn.tree import DecisionTreeClassifier

X, y = make_classification(
    n_samples=1000,
    n_features=10,
    n_informative=6,
    n_redundant=2,
    random_state=42
)

model = DecisionTreeClassifier(
    max_depth=8,
    random_state=42
)

train_sizes, train_scores, validation_scores = learning_curve(
    model,
    X,
    y,
    train_sizes=np.linspace(0.1, 1.0, 10),
    cv=5,
    scoring="accuracy"
)

train_mean = train_scores.mean(axis=1)
validation_mean = validation_scores.mean(axis=1)

plt.figure(figsize=(8, 5))
plt.plot(train_sizes, train_mean, marker="o", label="Train")
plt.plot(train_sizes, validation_mean, marker="o", label="Validation")
plt.xlabel("Training Data Size")
plt.ylabel("Accuracy")
plt.title("Learning Curve")
plt.legend()
plt.grid()
plt.show()
```

그래프에서는 다음을 함께 확인한다.

1. Train과 Validation 성능이 어느 수준으로 수렴하는가?
2. 데이터가 증가할수록 두 성능의 차이가 어떻게 변하는가?
3. Validation 성능이 아직 상승하고 있는가, 이미 정체됐는가?

| 학습곡선 패턴 | 해석 | 개선 방향 |
|---|---|---|
| 두 성능이 낮은 수준에서 수렴 | 과소적합 가능성 | 모델 복잡도 증가, Feature 개선 |
| Train은 높고 Validation과 Gap이 큼 | 과적합 가능성 | 데이터 추가, 모델 단순화 |
| 데이터 증가에 따라 Validation 상승 | 추가 데이터가 도움 될 가능성 | 데이터 추가 검토 |
| 두 성능이 높은 수준에서 가까워짐 | 비교적 좋은 일반화 | 현재 모델 유지 또는 추가 개선 검토 |

---

## 5. 검증곡선으로 모델 복잡도 확인

### 5.1 Validation Curve란?

검증곡선(Validation Curve)은 하나의 하이퍼파라미터 값을 변화시키면서 Train과 Validation 성능의 변화를 보여준다.

예를 들어 Decision Tree에서 `max_depth`를 증가시키면 다음 흐름이 나타날 수 있다.

| 영역 | Train 성능 | Validation 성능 | 해석 |
|---|---|---|---|
| 깊이가 너무 작음 | 낮음 | 낮음 | 과소적합 가능성 |
| 적절한 깊이 | 높음 | 높음 | 일반화 성능이 좋은 영역 |
| 깊이가 너무 큼 | 계속 증가 | 감소 | 과적합 가능성 |

하이퍼파라미터는 무조건 크게 또는 작게 설정하는 것이 아니라 Validation 성능이 좋은 영역을 찾아야 한다.

### 5.2 Python으로 Validation Curve 확인하기

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import make_classification
from sklearn.model_selection import validation_curve
from sklearn.tree import DecisionTreeClassifier

X, y = make_classification(
    n_samples=1000,
    n_features=10,
    n_informative=6,
    n_redundant=2,
    random_state=42
)

model = DecisionTreeClassifier(random_state=42)
depth_range = np.arange(1, 16)

train_scores, validation_scores = validation_curve(
    model,
    X,
    y,
    param_name="max_depth",
    param_range=depth_range,
    cv=5,
    scoring="accuracy"
)

train_mean = train_scores.mean(axis=1)
validation_mean = validation_scores.mean(axis=1)

plt.figure(figsize=(8, 5))
plt.plot(depth_range, train_mean, marker="o", label="Train")
plt.plot(depth_range, validation_mean, marker="o", label="Validation")
plt.xlabel("max_depth")
plt.ylabel("Accuracy")
plt.title("Validation Curve")
plt.legend()
plt.grid()
plt.show()
```

- `param_name`: 변화시킬 하이퍼파라미터 이름
- `param_range`: 비교할 하이퍼파라미터 값
- `cv=5`: 각 값에서 5-Fold 교차검증
- `train_scores`, `validation_scores`: 각 값과 Fold에서 얻은 성능

Validation 평균 성능이 가장 높은 값은 다음처럼 구할 수 있다.

```python
best_index = np.argmax(validation_mean)
best_depth = depth_range[best_index]
best_score = validation_mean[best_index]

print("Best max_depth:", best_depth)
print("Best Validation Accuracy:", best_score)
```

Train 성능이 가장 높은 값을 고르는 것이 아니라 Validation 성능을 기준으로 일반화 성능을 비교한다. 모델과 하이퍼파라미터 선택에 Validation을 사용한 후, 최종 성능은 선택 과정에 사용하지 않은 Test 데이터에서 별도로 평가해야 한다.

### 5.3 하이퍼파라미터 방향에 주의하기

값의 증가가 항상 모델 복잡도의 증가를 뜻하지 않는다.

| 하이퍼파라미터 | 값 증가 시 변화 |
|---|---|
| Decision Tree `max_depth` | 복잡도 증가 |
| KNN `n_neighbors` | 결정 경계가 부드러워져 복잡도 감소 |
| Logistic Regression `C` | 규제가 약해져 더 자유롭게 적합 |

Validation Curve를 해석하기 전에 해당 하이퍼파라미터가 모델 복잡도와 어떤 관계인지 먼저 이해해야 한다.

---

## 6. Learning Curve와 Validation Curve 비교

| 구분 | Learning Curve | Validation Curve |
|---|---|---|
| 변화시키는 것 | Train 데이터 양 | 하나의 하이퍼파라미터 값 |
| 확인하는 것 | 데이터 양에 따른 성능 변화 | 하이퍼파라미터에 따른 성능 변화 |
| 주요 질문 | 데이터를 더 추가하면 도움이 될까? | 어떤 설정에서 일반화 성능이 좋을까? |
| 예시 | 데이터 100개 → 1,000개 | `max_depth` 1 → 15 |

- 데이터가 부족한지 판단하려면 Learning Curve를 본다.
- 모델 복잡도의 적절한 범위를 찾으려면 Validation Curve를 본다.
- 두 곡선 모두 Train과 Validation 성능의 수준 및 차이를 함께 해석한다.

---

## 7. 진단 결과에 따른 개선 방향

### 과소적합으로 판단될 때

- 모델 복잡도를 높인다.
- 지나치게 강한 규제를 완화한다.
- 더 유용한 Feature를 만들거나 선택한다.
- 데이터의 관계를 더 잘 표현하는 모델을 사용한다.
- 데이터만 추가하기 전에 모델 자체가 패턴을 학습할 능력이 있는지 확인한다.

### 과적합으로 판단될 때

- 모델 복잡도를 낮춘다.
- 규제를 강화한다.
- 불필요한 Feature를 정리한다.
- 학습 데이터를 추가할 수 있는지 검토한다.
- Learning Curve에서 데이터 증가에 따라 Validation 성능이 개선되는지 확인한다.

### 반복적인 개선 절차

```text
Train·Validation 성능 측정
    ↓
성능 수준과 Generalization Gap 확인
    ↓
과소적합 또는 과적합 진단
    ↓
Learning Curve / Validation Curve 확인
    ↓
데이터·Feature·모델·하이퍼파라미터 개선
    ↓
다시 학습하고 Validation 성능 확인
    ↓
선택 완료 후 Test에서 최종 평가
```

## 핵심 정리

1. 과소적합은 모델이 너무 단순해 Train과 Validation 성능이 모두 낮은 상태다.
2. 과적합은 Train 성능은 높지만 Validation 성능이 상대적으로 낮고 차이가 큰 상태다.
3. 복잡도가 증가하면 일반적으로 Bias는 감소하고 Variance는 증가할 수 있다.
4. 가장 높은 Train 성능이 아니라 새로운 데이터에서의 일반화 성능을 목표로 해야 한다.
5. Generalization Gap은 성능의 절대 수준과 함께 해석해야 한다.
6. 모델별로 하이퍼파라미터 값과 복잡도의 관계가 다르다.
7. Learning Curve는 데이터 양의 효과를, Validation Curve는 하이퍼파라미터의 효과를 확인한다.
8. Validation으로 모델을 선택하고, Test는 선택이 끝난 최종 모델의 성능 평가에 사용한다.
9. 모델 개선은 진단, 조정, 재평가를 반복하는 과정이다.

## 오늘의 한 문장

> 좋은 모델은 Train 데이터를 가장 완벽하게 외운 모델이 아니라, 편향과 분산의 균형을 찾아 새로운 데이터에서도 안정적인 성능을 내는 모델이다.