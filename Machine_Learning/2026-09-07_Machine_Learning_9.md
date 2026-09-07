# 2026-09-07 결정트리와 앙상블

## 학습 목표

- 결정트리(Decision Tree)가 질문을 반복하여 예측하는 원리를 설명할 수 있다.
- 분류 트리와 회귀 트리의 차이를 이해할 수 있다.
- 불순도(Impurity), Gini Index, Entropy의 의미를 설명하고 계산할 수 있다.
- CART 알고리즘이 분할 기준을 선택하고 트리를 생성하는 과정을 이해할 수 있다.
- 트리의 깊이와 과대적합의 관계를 설명할 수 있다.
- 앙상블(Ensemble)과 랜덤 포레스트(Random Forest)의 학습 원리를 이해할 수 있다.

---

## 1. 결정트리란?

결정트리는 데이터를 여러 기준으로 분할하여 최종 결과를 예측하는 머신러닝 모델이다.

각 단계에서 다음과 같은 예·아니오 질문을 던지며 데이터를 작은 그룹으로 나눈다.

- 나이가 30세 이상인가?
- 소득이 5,000만 원 이상인가?
- 신용등급이 A등급 이상인가?

사람이 스무고개를 하거나 의사결정을 내리는 과정과 비슷하므로, 모델이 어떤 기준으로 예측했는지 비교적 쉽게 이해할 수 있다.

```text
전체 데이터
    ↓ 첫 번째 질문
조건을 만족함 / 만족하지 않음
    ↓ 다음 질문
더 작은 그룹으로 반복 분할
    ↓
Leaf Node에서 최종 예측
```

### 결정트리의 구성 요소

| 용어 | 의미 |
|---|---|
| Root Node | 모든 데이터가 처음 들어 있는 최상위 노드 |
| Decision Node | 특정 Feature와 기준값으로 질문하는 중간 노드 |
| Branch | 질문의 결과에 따라 다음 노드로 연결되는 경로 |
| Leaf Node | 더 이상 분할하지 않고 최종 예측을 내리는 노드 |
| Depth | Root Node에서 Leaf Node까지 내려가는 분할 단계 수 |

새로운 데이터가 들어오면 Root Node부터 시작해 조건을 차례대로 확인한다. 조건에 맞는 Branch를 따라가다가 Leaf Node에 도달하면 예측이 결정된다.

---

## 2. 분류 트리와 회귀 트리

결정트리는 예측하려는 Label의 종류에 따라 분류 트리와 회귀 트리로 나뉜다.

### 분류 트리(Classification Tree)

분류 트리는 합격/불합격, 이탈/유지처럼 범주형 Label을 예측한다. Leaf Node에 도달하면 그 노드에 속한 학습 데이터 중 가장 많은 클래스를 최종 예측으로 선택한다.

### 회귀 트리(Regression Tree)

회귀 트리는 가격, 매출, 월세처럼 연속형 숫자를 예측한다. Leaf Node에 도달하면 일반적으로 그 노드에 속한 학습 데이터의 평균값을 예측값으로 사용한다.

| 구분 | 분류 트리 | 회귀 트리 |
|---|---|---|
| 예측 대상 | 범주형 값 | 연속형 숫자 |
| 예시 | 가입/미가입, 합격/불합격 | 가격, 매출, 월세 |
| Leaf의 예측 | 가장 많은 클래스 | 데이터 값의 평균 |
| 대표 분할 기준 | Gini Index, Entropy | MSE |
| scikit-learn | `DecisionTreeClassifier` | `DecisionTreeRegressor` |

---

## 3. 연습문제 1: 멤버십 가입 여부 예측

고객의 방문 횟수와 누적 구매 금액을 이용하여 멤버십 가입 여부를 예측한다.

- `0`: 미가입
- `1`: 가입

```python
import pandas as pd
from sklearn.tree import DecisionTreeClassifier

data = pd.DataFrame({
    "visits": [1, 2, 3, 5, 7, 10],
    "purchase": [3, 5, 8, 20, 30, 50],
    "membership": [0, 0, 0, 1, 1, 1],
})

X = data[["visits", "purchase"]]
y = data["membership"]

model = DecisionTreeClassifier(
    max_depth=2,
    random_state=42,
)
model.fit(X, y)

new_customer = pd.DataFrame({
    "visits": [8],
    "purchase": [35],
})

prediction = model.predict(new_customer)

print("멤버십 가입 예측:", prediction[0])
```

실행 결과:

```text
멤버십 가입 예측: 1
```

학습된 트리의 구조는 다음과 같다.

```text
visits <= 4.00
├── True  → class: 0
└── False → class: 1
```

신규 고객은 방문 횟수가 8회이므로 `visits > 4` 경로로 이동하여 가입 클래스 `1`로 예측된다.

---

## 4. 연습문제 2: Regression Tree로 월세 예측

집의 면적과 지하철역까지의 거리를 이용하여 월세를 예측한다.

```python
import pandas as pd
from sklearn.tree import DecisionTreeRegressor

data = pd.DataFrame({
    "area": [25, 30, 40, 50, 60, 70],
    "station_minutes": [15, 12, 10, 6, 5, 3],
    "rent": [45, 50, 65, 80, 95, 110],
})

X = data[["area", "station_minutes"]]
y = data["rent"]

model = DecisionTreeRegressor(random_state=42)
model.fit(X, y)

new_house = pd.DataFrame({
    "area": [50],
    "station_minutes": [5],
})

prediction = model.predict(new_house)

print("예측 월세:", prediction[0])
```

실행 결과:

```text
예측 월세: 80.0
```

회귀 트리는 질문을 따라 Leaf Node에 도달한 뒤 그 Leaf에 해당하는 값을 이용해 연속적인 숫자를 예측한다.

### 결정트리를 끝없이 깊게 만들면 안 되는 이유

트리가 깊어지면 학습 데이터를 매우 세밀하게 분할할 수 있다. 학습 데이터는 잘 맞힐 수 있지만, 노이즈까지 학습하여 새로운 데이터에서는 성능이 떨어지는 과대적합(Overfitting)이 발생하기 쉽다.

이를 막기 위해 다음과 같은 하이퍼파라미터로 트리의 복잡도를 제한할 수 있다.

- `max_depth`: 트리의 최대 깊이
- `min_samples_split`: 노드를 분할하기 위해 필요한 최소 샘플 수
- `min_samples_leaf`: Leaf Node에 남아 있어야 하는 최소 샘플 수

---

## 5. 불순도(Impurity)

불순도는 한 노드 안에 서로 다른 클래스가 얼마나 섞여 있는지를 나타내는 값이다.

```text
한 클래스만 존재
→ 완전히 순수한 노드
→ 불순도 0

여러 클래스가 비슷한 비율로 섞임
→ 분류하기 어려운 노드
→ 높은 불순도
```

예를 들어 사과 10개만 있는 상자는 순수하지만, 사과 5개와 배 5개가 들어 있는 상자는 두 클래스가 같은 비율로 섞여 있어 불순도가 높다.

결정트리는 여러 분할 후보를 비교하여 자식 노드의 불순도를 가장 많이 감소시키는 질문을 선택한다.

---

## 6. Gini Index

Gini Index는 분류 트리에서 노드의 불순도를 측정하는 대표적인 지표이다.

클래스 $i$의 비율을 $p_i$라고 하면 Gini Index는 다음과 같다.

$$
\mathrm{Gini}=1-\sum_{i=1}^{K}p_i^2
$$

### Gini Index 직접 구현

```python
def gini(class_counts):
    total = sum(class_counts)
    gini_value = 1

    for count in class_counts:
        probability = count / total
        gini_value -= probability ** 2

    return gini_value


mixed_group = gini([5, 5])
pure_group = gini([10, 0])

print("5명 / 5명 그룹:", mixed_group)
print("10명 / 0명 그룹:", pure_group)
```

실행 결과:

```text
5명 / 5명 그룹: 0.5
10명 / 0명 그룹: 0.0
```

직접 계산하면 다음과 같다.

$$
\mathrm{Gini}([5,5])
=1-(0.5^2+0.5^2)=0.5
$$

$$
\mathrm{Gini}([10,0])
=1-(1^2+0^2)=0
$$

두 클래스가 같은 비율로 섞인 노드보다 하나의 클래스만 있는 노드가 더 순수하다.

---

## 7. Entropy

Entropy도 노드의 불순도를 측정하는 지표이다.

$$
\mathrm{Entropy}
=-\sum_{i=1}^{K}p_i\log_2(p_i)
$$

한 클래스의 비율이 0이면 해당 항은 계산에서 0으로 처리한다.

### Gini Index와 Entropy 비교

| 구분 | Gini Index | Entropy |
|---|---|---|
| 수식 | $1-\sum p_i^2$ | $-\sum p_i\log_2p_i$ |
| 순수한 노드 | 0 | 0 |
| 역할 | 클래스 혼합 정도 측정 | 클래스 혼합 정도 측정 |
| scikit-learn 지정 | `criterion="gini"` | `criterion="entropy"` |

두 지표 모두 값이 작을수록 노드가 더 순수하다. scikit-learn의 `DecisionTreeClassifier`는 기본적으로 Gini Index를 사용한다.

---

## 8. 연습문제 3: 두 고객 그룹의 Gini Index

- 그룹 A: 반응 고객 8명, 미반응 고객 2명
- 그룹 B: 반응 고객 5명, 미반응 고객 5명

```python
def gini(class_counts):
    total = sum(class_counts)
    return 1 - sum((count / total) ** 2 for count in class_counts)


gini_a = gini([8, 2])
gini_b = gini([5, 5])

print("그룹 A Gini:", gini_a)
print("그룹 B Gini:", gini_b)
```

실행 결과:

```text
그룹 A Gini: 0.32
그룹 B Gini: 0.5
```

직접 계산하면 다음과 같다.

$$
\mathrm{Gini}(A)
=1-(0.8^2+0.2^2)=0.32
$$

$$
\mathrm{Gini}(B)
=1-(0.5^2+0.5^2)=0.5
$$

Gini Index가 더 작은 그룹 A가 그룹 B보다 순수하다.

---

## 9. CART 알고리즘

CART(Classification and Regression Tree)는 scikit-learn의 결정트리가 사용하는 알고리즘이다.

CART는 각 단계에서 데이터를 두 그룹으로 나누는 여러 질문을 평가하고, 현재 노드에서 가장 좋은 질문을 선택한다.

- Classification Tree: Gini Index 등의 불순도를 이용한다.
- Regression Tree: MSE를 이용한다.

### 1단계: 가능한 분할 기준 평가

현재 노드에서 모든 Feature와 가능한 기준값을 하나씩 적용한다. 각 후보로 데이터를 두 그룹으로 나누고 분할 후의 불순도 또는 MSE를 계산한다.

예를 들면 다음과 같은 후보를 비교할 수 있다.

```text
visits <= 3.5인가?
visits <= 4.5인가?
purchase <= 15인가?
purchase <= 25인가?
```

### 2단계: 가장 좋은 분할 선택

분할 후 자식 노드의 불순도 또는 MSE가 가장 많이 감소하는 기준을 선택한다.

분류 트리에서는 부모 노드와 자식 노드의 불순도 차이를 이용해 정보 이득을 생각할 수 있다. 자식 노드의 크기가 다르므로 각 자식의 샘플 비율을 반영한 가중 불순도를 사용한다.

$$
\text{Impurity Decrease}
=I(\text{parent})
-\left(
\frac{n_L}{n}I(\text{left})
+\frac{n_R}{n}I(\text{right})
\right)
$$

### 3단계: 반복 및 종료

새로 생성된 각 자식 노드에서 가능한 분할을 다시 평가한다. 다음과 같은 종료 조건을 만족하면 분할을 멈추고 Leaf Node를 만든다.

- 더 이상 불순도나 MSE를 줄일 수 없음
- `max_depth`에 도달함
- `min_samples_split` 조건을 만족하지 못함
- 분할 후 `min_samples_leaf` 조건을 만족하지 못함

```text
모든 분할 후보 평가
    ↓
현재 가장 좋은 질문 선택
    ↓
두 개의 자식 노드 생성
    ↓
각 자식 노드에서 같은 과정 반복
    ↓
종료 조건을 만족하면 Leaf Node 생성
```

---

## 10. CART는 Greedy 알고리즘

CART는 각 단계에서 **현재 가장 좋은 분할**을 선택하는 Greedy 알고리즘이다.

현재 노드에서 최선인 질문이 전체 트리에서도 항상 최선이라는 보장은 없다. 즉, CART가 항상 Global Optimum 트리를 만드는 것은 아니다.

그러나 가능한 모든 트리 구조를 생성하여 비교하면 경우의 수가 매우 많아 계산하기 어렵다. CART는 각 단계에서 좋은 선택을 반복하여 계산량을 크게 줄이면서도 실용적으로 충분히 좋은 트리를 빠르게 만든다.

---

## 11. 연습문제 4: 첫 번째 분할 질문 확인

사용자의 이용 시간과 방문 횟수로 유료 구독 여부를 예측한다.

```python
import pandas as pd
from sklearn.tree import DecisionTreeClassifier, export_text

data = pd.DataFrame({
    "usage_minutes": [10, 15, 20, 30, 45, 60, 70, 90],
    "visits": [1, 2, 2, 3, 5, 7, 8, 10],
    "subscription": [0, 0, 0, 0, 1, 1, 1, 1],
})

X = data[["usage_minutes", "visits"]]
y = data["subscription"]

model = DecisionTreeClassifier(random_state=42)
model.fit(X, y)

tree_rules = export_text(
    model,
    feature_names=["usage_minutes", "visits"],
)

print(tree_rules)
```

실행 결과:

```text
|--- usage_minutes <= 37.50
|   |--- class: 0
|--- usage_minutes >  37.50
|   |--- class: 1
```

모델이 선택한 첫 번째 질문은 `usage_minutes <= 37.5인가?`이다.

- 이용 시간이 37.5분 이하이면 미구독 클래스 `0`
- 이용 시간이 37.5분보다 크면 구독 클래스 `1`

이 기준만으로 두 클래스가 완전히 나뉘므로 추가 질문이 필요하지 않다.

---

## 12. 연습문제 5: max_depth 비교

두 결정트리의 `max_depth`를 각각 1과 3으로 설정하여 트리 구조를 비교한다.

```python
import pandas as pd
from sklearn.tree import DecisionTreeClassifier, export_text

data = pd.DataFrame({
    "study_hours": [1, 2, 3, 4, 5, 6, 7, 8],
    "attendance": [50, 60, 65, 70, 80, 85, 90, 95],
    "pass": [0, 0, 0, 1, 0, 1, 1, 1],
})

X = data[["study_hours", "attendance"]]
y = data["pass"]

tree_depth_1 = DecisionTreeClassifier(
    max_depth=1,
    random_state=42,
)

tree_depth_3 = DecisionTreeClassifier(
    max_depth=3,
    random_state=42,
)

tree_depth_1.fit(X, y)
tree_depth_3.fit(X, y)

feature_names = ["study_hours", "attendance"]

print("max_depth=1")
print(export_text(tree_depth_1, feature_names=feature_names))

print("max_depth=3")
print(export_text(tree_depth_3, feature_names=feature_names))
```

`max_depth=1`의 결과:

```text
|--- study_hours <= 3.50
|   |--- class: 0
|--- study_hours >  3.50
|   |--- class: 1
```

`max_depth=3`의 결과:

```text
|--- study_hours <= 3.50
|   |--- class: 0
|--- study_hours >  3.50
|   |--- attendance <= 82.50
|   |   |--- attendance <= 75.00
|   |   |   |--- class: 1
|   |   |--- attendance >  75.00
|   |   |   |--- class: 0
|   |--- attendance >  82.50
|   |   |--- class: 1
```

### 결과 비교

- 깊이 1 모델은 질문 하나만 사용하므로 구조가 단순하다.
- 깊이 3 모델은 추가 질문을 사용하여 학습 데이터를 더 세밀하게 구분한다.
- 깊이가 커지면 모델의 표현력은 높아지지만 과대적합 위험도 커진다.
- 깊이가 너무 작으면 중요한 패턴을 배우지 못해 과소적합이 발생할 수 있다.

따라서 `max_depth`는 Train 성능만 보고 결정하지 않고 Validation 또는 교차검증 성능을 이용해 선택해야 한다.

---

## 13. 앙상블(Ensemble)

앙상블은 여러 모델을 학습한 뒤 각 모델의 예측을 결합하여 최종 결과를 만드는 방법이다.

한 명의 심사위원에게만 판단을 맡기면 그 사람의 실수가 그대로 최종 결과가 된다. 여러 심사위원이 서로 다른 관점에서 평가하고 결과를 종합하면 한 사람의 실수를 다른 사람이 보완할 수 있다.

앙상블도 서로 다른 데이터나 방식으로 여러 모델을 학습하여 다음 효과를 얻는다.

- 개별 모델의 실수를 다른 모델이 보완할 수 있다.
- 예측의 변동성을 줄이고 안정성을 높일 수 있다.
- 새로운 데이터에 대한 일반화 성능을 높일 수 있다.

앙상블에서는 개별 모델들이 모두 똑같은 실수를 하지 않도록 서로 다른 모델을 만드는 것이 중요하다.

---

## 14. 랜덤 포레스트(Random Forest)

랜덤 포레스트는 여러 결정트리를 학습하고 예측을 결합하는 대표적인 앙상블 모델이다.

### 서로 다른 트리를 만드는 두 가지 무작위성

#### 1. Bootstrap Sampling

각 결정트리는 원본 데이터에서 복원추출한 서로 다른 학습 데이터로 학습한다.

복원추출은 한 번 선택한 데이터를 다시 선택할 수 있는 추출 방법이다. 따라서 어떤 데이터는 여러 번 포함되고 어떤 데이터는 포함되지 않을 수 있다.

#### 2. Feature Sampling

각 분할에서 전체 Feature를 모두 비교하지 않고 일부 Feature를 무작위로 선택하여 후보로 사용한다.

이 두 방법을 통해 트리들이 서로 비슷해지는 것을 줄이고 다양한 판단 기준을 갖도록 만든다.

```text
원본 데이터
    ↓ Bootstrap Sampling
서로 다른 학습 데이터 여러 개
    ↓ 각 분할에서 일부 Feature Sampling
서로 다른 Decision Tree 여러 개
    ↓ 예측 결합
최종 예측
```

### 최종 예측 결합

| 문제 유형 | 결합 방식 |
|---|---|
| 분류 | 각 트리의 예측을 다수결(Voting)로 결정 |
| 회귀 | 각 트리의 예측값을 평균(Averaging) |

예를 들어 결정트리 100개 중 70개가 고객 이탈을, 30개가 유지를 예측했다면 다수결에 따라 최종 결과는 이탈이다.

### Random Forest의 특징

- 단일 결정트리보다 예측이 안정적이다.
- 하나의 트리가 만드는 오류의 영향을 줄일 수 있다.
- 단일 결정트리보다 과대적합에 강한 편이다.
- 여러 트리를 학습하므로 계산량과 메모리 사용량은 증가한다.
- 단일 트리보다 전체 모델의 판단 과정을 해석하기 어렵다.

---

## 15. Decision Tree와 Random Forest 비교

```python
import pandas as pd
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier

data = pd.DataFrame({
    "visits": [1, 2, 2, 3, 4, 5, 6, 7, 8, 9],
    "purchase": [2, 5, 4, 10, 8, 20, 25, 30, 35, 50],
    "vip": [0, 0, 0, 0, 0, 1, 1, 1, 1, 1],
})

X = data[["visits", "purchase"]]
y = data["vip"]

tree = DecisionTreeClassifier(random_state=42)

forest = RandomForestClassifier(
    n_estimators=100,
    random_state=42,
)

tree.fit(X, y)
forest.fit(X, y)

new_customer = pd.DataFrame({
    "visits": [6],
    "purchase": [27],
})

print("Decision Tree:", tree.predict(new_customer)[0])
print("Random Forest:", forest.predict(new_customer)[0])
```

실행 결과:

```text
Decision Tree: 1
Random Forest: 1
```

두 모델 모두 신규 고객을 VIP 클래스 `1`로 예측했다.

- `DecisionTreeClassifier`는 하나의 트리로 예측한다.
- `RandomForestClassifier`는 여러 트리의 예측을 종합한다.
- `n_estimators=100`은 랜덤 포레스트를 구성하는 결정트리의 수를 100개로 설정한다.

이 예제에서 두 모델의 예측이 같더라도 학습 방식은 다르다. 모델을 비교할 때는 한 데이터의 예측만 확인하지 않고 별도의 Test Set이나 교차검증을 이용해 전체 성능을 평가해야 한다.

### 두 모델의 차이

| 구분 | Decision Tree | Random Forest |
|---|---|---|
| 트리 수 | 1개 | 여러 개 |
| 학습 데이터 | 전체 학습 데이터 | 트리마다 Bootstrap Sample |
| 분할 Feature | 일반적으로 모든 Feature 후보 | 일부 Feature를 무작위 선택 |
| 분류 결과 | 한 트리의 Leaf 클래스 | 여러 트리의 다수결 |
| 회귀 결과 | 한 트리의 Leaf 평균 | 여러 트리 예측의 평균 |
| 해석력 | 비교적 높음 | 상대적으로 낮음 |
| 안정성 | 데이터 변화에 민감할 수 있음 | 여러 트리를 결합해 비교적 안정적 |
| 과대적합 | 발생하기 쉬움 | 단일 트리보다 강한 편 |

---

## 16. 핵심 용어 정리

| 용어 | 의미 |
|---|---|
| Decision Tree | 질문을 반복해 데이터를 나누고 예측하는 모델 |
| Classification Tree | 범주형 Label을 예측하는 결정트리 |
| Regression Tree | 연속형 Label을 예측하는 결정트리 |
| Impurity | 한 노드에 서로 다른 클래스가 섞여 있는 정도 |
| Gini Index | $1-\sum p_i^2$로 계산하는 불순도 지표 |
| Entropy | $-\sum p_i\log_2p_i$로 계산하는 불순도 지표 |
| CART | 현재 단계에서 가장 좋은 이진 분할을 반복하는 트리 알고리즘 |
| Greedy Algorithm | 각 단계에서 현재 가장 좋은 선택을 하는 알고리즘 |
| Ensemble | 여러 모델의 예측을 결합하는 방법 |
| Bootstrap Sampling | 원본 데이터에서 복원추출하여 학습 데이터를 만드는 방법 |
| Feature Sampling | 분할 후보 Feature 일부를 무작위로 선택하는 방법 |
| Random Forest | 서로 다른 여러 결정트리의 예측을 결합하는 앙상블 모델 |

---

## 17. 오늘 배운 내용 정리

### 결정트리

- Feature와 기준값을 이용한 질문으로 데이터를 반복해서 분할한다.
- 분류 트리는 Leaf Node의 다수 클래스로 예측한다.
- 회귀 트리는 Leaf Node에 속한 값의 평균으로 예측한다.
- 트리가 너무 깊으면 학습 데이터에 과대적합되기 쉽다.

### 불순도와 CART

- 불순도는 한 노드에 서로 다른 클래스가 섞여 있는 정도이다.
- Gini Index와 Entropy는 분류 노드의 대표적인 불순도 지표이다.
- CART는 분류에서는 불순도, 회귀에서는 MSE가 많이 감소하는 분할을 선택한다.
- 각 단계에서 현재 가장 좋은 분할을 선택하는 Greedy 알고리즘이다.

### 랜덤 포레스트

- 여러 결정트리의 예측을 결합하는 앙상블 모델이다.
- Bootstrap Sampling과 Feature Sampling으로 서로 다른 트리를 만든다.
- 분류는 다수결, 회귀는 평균으로 최종 예측을 결정한다.
- 단일 결정트리보다 안정적이고 과대적합에 강한 편이다.

### 전체 학습 흐름

```text
현재 노드의 모든 분할 후보 평가
    ↓
불순도 또는 MSE가 가장 많이 감소하는 질문 선택
    ↓
두 자식 노드로 분할
    ↓
종료 조건까지 반복하여 Decision Tree 생성
    ↓
여러 트리를 서로 다르게 학습
    ↓
Voting 또는 Averaging으로 Random Forest 예측
```

---

## 18. 느낀 점

오늘은 결정트리가 복잡한 수식을 직접 출력하는 대신 예·아니오 질문을 반복하여 데이터를 분리한다는 점을 배웠다. 트리 구조를 텍스트로 출력했을 때 모델이 첫 번째로 어떤 Feature와 기준값을 선택했는지 확인할 수 있어 로지스틱 회귀보다 예측 과정을 직관적으로 이해하기 쉬웠다.

하지만 트리를 계속 깊게 만들면 학습 데이터를 세밀하게 구분할 수 있는 대신 노이즈까지 학습하여 과대적합이 발생할 수 있다는 점도 중요했다. `max_depth=1`과 `max_depth=3`의 결과를 비교하면서 깊이가 커질수록 질문이 많아지고 구조가 복잡해지는 모습을 직접 확인했다.

Gini Index는 한 노드 안에 클래스가 얼마나 섞여 있는지를 숫자로 표현한다. `[5, 5]` 그룹은 0.5이고 `[10, 0]` 그룹은 0이 되는 계산을 통해, 결정트리가 불순도가 낮아지는 질문을 선택한다는 의미를 이해할 수 있었다.

또한 CART는 각 단계에서 현재 가장 좋은 질문을 선택하지만 전체적으로 완벽한 트리를 보장하지는 않는 Greedy 알고리즘이라는 점이 인상적이었다. 가능한 모든 트리를 비교하는 대신 계산 가능한 범위에서 좋은 트리를 빠르게 만드는 현실적인 방법이라고 이해했다.

랜덤 포레스트는 단순히 같은 트리를 여러 번 만드는 것이 아니라 Bootstrap Sampling과 Feature Sampling으로 서로 다른 트리를 만들고 결과를 결합한다. 개별 트리의 실수를 다른 트리가 보완하여 단일 결정트리보다 안정적인 예측을 만든다는 점이 핵심이라고 생각한다.

앞으로 결정트리를 사용할 때는 Train 성능만 높이기 위해 트리를 깊게 만들지 않고, `max_depth`, `min_samples_split`, `min_samples_leaf`를 교차검증으로 조정해야겠다. 그리고 단일 트리와 랜덤 포레스트의 Test 성능 및 예측 안정성을 비교해 앙상블의 효과를 직접 확인해 보고 싶다.
