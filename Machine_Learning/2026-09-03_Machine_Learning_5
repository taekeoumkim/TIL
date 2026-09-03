# 2026-09-03 모델 학습 시 발생 문제와 처리 방법

## 학습 목표

- 편향-분산 상충 관계(Bias-Variance Trade-off)를 설명할 수 있다.
- 기대 오차(Expected Error)가 Bias², Variance, Irreducible Error로 나뉘는 원리를 이해할 수 있다.
- 과소적합(Underfitting)과 과대적합(Overfitting)을 학습 및 테스트 성능으로 구분할 수 있다.
- L1·L2 정규화의 차이를 이해하고 scikit-learn으로 적용할 수 있다.
- 모델의 학습 성능뿐 아니라 새로운 데이터에 대한 일반화 성능을 함께 평가할 수 있다.

---

## 1. 학습 데이터에서 잘 맞는 모델이 실전에서는 틀리는 이유

머신러닝 모델은 학습 데이터의 규칙을 익혀 처음 보는 데이터의 값을 예측한다. 그러나 학습 데이터에서 성능이 높다고 해서 새로운 데이터에서도 반드시 좋은 성능을 내는 것은 아니다.

- 모델이 너무 단순하면 데이터의 중요한 패턴도 배우지 못한다.
- 모델이 너무 복잡하면 실제 패턴뿐 아니라 학습 데이터의 노이즈까지 외울 수 있다.

따라서 좋은 모델은 학습 데이터를 무조건 완벽하게 맞히는 모델이 아니라, **처음 보는 데이터에서도 안정적으로 예측하는 모델**이다. 이를 일반화(Generalization) 성능이 좋은 모델이라고 한다.

```text
너무 단순한 모델                          너무 복잡한 모델
High Bias                               High Variance
Underfitting        ← 적절한 복잡도 →      Overfitting
```

---

## 2. 편향(Bias)과 분산(Variance)

### Bias

Bias는 여러 학습 데이터로 만든 모델들의 **평균 예측값과 실제 정답 사이의 차이**를 의미한다.

Bias가 크다는 것은 모델이 너무 단순하여 데이터의 패턴을 충분히 학습하지 못했다는 뜻이다. 이 경우 학습 데이터와 테스트 데이터 모두에서 오차가 크게 나타날 수 있다.

```text
High Bias
→ 모델이 지나치게 단순함
→ 실제 패턴을 충분히 표현하지 못함
→ Underfitting 발생 가능
```

### Variance

Variance는 **학습 데이터가 조금 달라졌을 때 모델의 예측이 얼마나 크게 변하는지**를 의미한다.

Variance가 크다는 것은 모델이 학습 데이터의 세부 특징이나 노이즈에 지나치게 민감하다는 뜻이다. 학습 데이터에서는 오차가 작지만 새로운 데이터에서는 오차가 커질 수 있다.

```text
High Variance
→ 모델이 지나치게 복잡함
→ 학습 데이터의 노이즈까지 학습함
→ Overfitting 발생 가능
```

### Bias와 Variance 비교

| 구분 | High Bias | High Variance |
|---|---|---|
| 모델 복잡도 | 너무 단순함 | 너무 복잡함 |
| 학습 상태 | 패턴을 충분히 학습하지 못함 | 노이즈와 세부 특징까지 학습함 |
| 학습 데이터 오차 | 큼 | 작음 |
| 테스트 데이터 오차 | 큼 | 큼 |
| 관련 문제 | Underfitting | Overfitting |

---

## 3. Bias-Variance Trade-off

Trade-off는 한쪽을 개선하면 다른 한쪽이 나빠질 수 있는 관계를 의미한다.

- 모델을 단순하게 만들면 Bias는 커지고 Variance는 작아지는 경향이 있다.
- 모델을 복잡하게 만들면 Bias는 작아지지만 Variance는 커질 수 있다.

따라서 머신러닝의 목표는 Bias와 Variance를 각각 무조건 0으로 만드는 것이 아니라, 두 값을 적절히 조절하여 **새로운 데이터의 기대 오차가 가장 작아지는 지점**을 찾는 것이다.

### 실제 데이터의 구성

실제 데이터의 정답은 다음과 같이 표현할 수 있다.

$$
y=f(x)+\epsilon
$$

- $f(x)$: 현실에 존재하는 입력과 정답 사이의 규칙
- $\epsilon$: 데이터에 포함된 노이즈
- $E[\epsilon]=0$
- $\mathrm{Var}(\epsilon)=\sigma^2$

학습 데이터 $D$로 학습한 모델의 예측을 다음과 같이 나타낸다.

$$
\hat{f}_D(x)
$$

학습 데이터가 달라지면 학습된 모델과 예측 결과도 달라질 수 있다.

### Expected Error

새로운 데이터에서의 평균 제곱 오차는 다음과 같다.

$$
E_{D,\epsilon}\left[(y-\hat{f}_D(x))^2\right]
$$

$y=f(x)+\epsilon$을 대입하면 다음과 같이 전개할 수 있다.

$$
E_{D,\epsilon}\left[(f(x)+\epsilon-\hat{f}_D(x))^2\right]
$$

노이즈의 평균이 0이고 분산이 $\sigma^2$이므로 다음과 같이 정리된다.

$$
E_{D,\epsilon}\left[(y-\hat{f}_D(x))^2\right]
=E_D\left[(f(x)-\hat{f}_D(x))^2\right]+\sigma^2
$$

### Bias와 Variance로 분해하기

여러 학습 데이터로 만든 모델들의 평균 예측값을 다음과 같이 정의한다.

$$
\bar{f}(x)=E_D[\hat{f}_D(x)]
$$

모델의 오차에 평균 예측값을 더하고 빼면 다음과 같다.

$$
f(x)-\hat{f}_D(x)
=f(x)-\bar{f}(x)+\bar{f}(x)-\hat{f}_D(x)
$$

이를 전개하여 정리하면 기대 예측 오차는 세 부분으로 분해된다.

$$
\text{Expected Error}
=\text{Bias}^2+\text{Variance}+\text{Irreducible Error}
$$

각 항은 다음과 같다.

$$
\text{Bias}^2
=\left(E_D[\hat{f}_D(x)]-f(x)\right)^2
$$

$$
\text{Variance}
=E_D\left[\left(\hat{f}_D(x)-E_D[\hat{f}_D(x)]\right)^2\right]
$$

$$
\text{Irreducible Error}=\sigma^2
$$

| 오차 요소 | 의미 | 모델로 줄일 수 있는가? |
|---|---|---|
| Bias² | 모델의 평균 예측과 실제 함수의 차이 | 모델 복잡도 등을 조절해 줄일 수 있음 |
| Variance | 학습 데이터 변화에 따른 예측의 변동 | 정규화, 데이터 추가 등으로 줄일 수 있음 |
| Irreducible Error | 데이터 자체의 노이즈로 발생하는 오차 | 완전히 제거할 수 없음 |

### 왜 Bias와 Variance를 모두 0으로 만들기 어려울까?

현실에서는 모델 복잡도를 높여 Bias를 줄이면 Variance가 커지고, 모델을 단순하게 만들어 Variance를 줄이면 Bias가 커지는 경우가 많다. 데이터 자체에 포함된 노이즈도 완전히 제거할 수 없다.

따라서 두 값을 균형 있게 조절하여 Expected Error가 가장 작은 모델을 찾는 것이 중요하다.

### 연습문제 1: 두 매출 예측 모델 판단

- 모델 A: 학습 데이터를 바꾸어도 비슷한 예측을 하지만 실제 매출과 계속 큰 차이가 난다.
- 모델 B: 학습 데이터에서는 실제 매출과 비슷하지만 학습 데이터를 조금만 바꿔도 예측이 크게 달라진다.

정답:

```python
model_a_state = "High Bias"
model_b_state = "High Variance"

print("모델 A:", model_a_state)
print("모델 B:", model_b_state)
```

- 모델 A는 예측이 일관되지만 계속 실제값에서 벗어나므로 High Bias에 가깝다.
- 모델 B는 학습 데이터 변화에 따라 예측이 크게 달라지므로 High Variance에 가깝다.

---

## 4. 과소적합(Underfitting)

Underfitting은 모델이 너무 단순하거나 충분히 학습되지 않아 데이터의 패턴을 제대로 학습하지 못한 상태이다.

### 특징

- High Bias와 관련이 있다.
- 학습 데이터의 오차가 크다.
- 테스트 데이터의 오차도 크다.
- 모델이 데이터의 중요한 관계를 충분히 표현하지 못한다.

### 개선 방법

- 모델의 복잡도를 높인다.
- 더 유용한 Feature를 추가한다.
- 학습이 부족한 경우 학습 횟수를 늘린다.
- 정규화가 지나치게 강하다면 강도를 낮춘다.

---

## 5. 과대적합(Overfitting)

Overfitting은 모델이 데이터에 비해 지나치게 복잡하여 학습 데이터의 노이즈나 불필요한 세부 특징까지 학습한 상태이다.

### 특징

- High Variance와 관련이 있다.
- 학습 데이터의 오차는 매우 작다.
- 테스트 데이터의 오차는 크다.
- 학습 데이터가 달라지면 예측도 크게 달라질 수 있다.

### 개선 방법

- 모델의 복잡도를 낮춘다.
- L1 또는 L2 정규화를 적용한다.
- 학습 데이터를 추가한다.
- 불필요한 Feature를 제거한다.

### Underfitting과 Overfitting 비교

| 구분 | Underfitting | 적절한 학습 | Overfitting |
|---|---|---|---|
| 모델 복잡도 | 너무 단순함 | 적절함 | 너무 복잡함 |
| Bias / Variance | High Bias | 균형 | High Variance |
| Train 성능 | 낮음 | 높음 | 매우 높음 |
| Test 성능 | 낮음 | 높음 | 낮음 |
| Train-Test 차이 | 둘 다 나쁨 | 작음 | 큼 |
| 대표 처방 | 복잡도·학습량 증가 | 현재 설정 검증 | 복잡도 감소·정규화·데이터 추가 |

---

## 6. Train MSE와 Test MSE로 모델 상태 진단하기

모델의 상태를 판단하려면 학습 데이터의 성능만 보는 것이 아니라 Train 성능과 Test 성능을 함께 확인해야 한다.

```python
models = {
    "모델 A": {"train_mse": 45, "test_mse": 50},
    "모델 B": {"train_mse": 10, "test_mse": 12},
    "모델 C": {"train_mse": 2, "test_mse": 35},
}

for name, score in models.items():
    print(
        name,
        "Train MSE:", score["train_mse"],
        "Test MSE:", score["test_mse"],
    )
```

결과 해석:

| 모델 | Train MSE | Test MSE | 판단 | 근거 |
|---|---:|---:|---|---|
| 모델 A | 45 | 50 | Underfitting 의심 | 두 오차가 모두 큼 |
| 모델 B | 10 | 12 | 비교적 적절한 학습 | 두 오차가 모두 작고 차이도 작음 |
| 모델 C | 2 | 35 | Overfitting 의심 | Train 오차는 작지만 Test 오차가 크게 증가함 |

학습 데이터의 성능이 매우 높더라도 Test 성능이 크게 떨어진다면 좋은 모델이라고 할 수 없다. 우리가 원하는 것은 학습 데이터를 외우는 모델이 아니라 새로운 데이터에도 적용할 수 있는 모델이다.

### 연습문제 2: 배송 시간 예측 모델 판단

| 모델 | Train MSE | Test MSE | 상태 |
|---|---:|---:|---|
| A | 40 | 43 | Underfitting |
| B | 8 | 10 | 적절한 학습 |
| C | 2 | 31 | Overfitting |

```python
models = {
    "A": {"train_mse": 40, "test_mse": 43},
    "B": {"train_mse": 8, "test_mse": 10},
    "C": {"train_mse": 2, "test_mse": 31},
}

states = {
    "A": "Underfitting",
    "B": "적절한 학습",
    "C": "Overfitting",
}

for name in models:
    print(f"모델 {name}: {states[name]}")
```

오차의 절대적인 기준은 데이터와 문제에 따라 달라질 수 있으므로, 실제 프로젝트에서는 단순한 고정 숫자만으로 판단하지 않고 기준 모델 및 검증 결과와 함께 비교해야 한다.

---

## 7. 정규화(Regularization)

정규화는 모델이 학습 데이터에 지나치게 맞춰지는 것을 막기 위해 **손실함수에 가중치 크기에 대한 벌점(Penalty)을 추가하는 방법**이다.

Feature가 많거나 모델이 복잡하면 특정 가중치가 지나치게 커지면서 Overfitting이 발생할 수 있다. 정규화는 큰 가중치에 비용을 부과해 모델을 단순하고 안정적으로 만든다.

```text
기존 손실함수
    +
가중치 크기에 대한 Penalty
    ↓
정규화가 적용된 손실함수
```

정규화의 강도는 $\lambda$로 표현하며, scikit-learn의 Lasso와 Ridge에서는 `alpha` 매개변수로 설정한다.

- `alpha`가 커짐 → 정규화가 강해지고 가중치가 더 많이 축소됨
- `alpha`가 작아짐 → 정규화 효과가 약해짐

---

## 8. L1 정규화와 Lasso

L1 정규화는 가중치 절댓값의 합을 Penalty로 사용한다.

$$
L_{L1}(W)=L(W)+\lambda\sum_{j=1}^{p}|w_j|
$$

### 특징

- 일부 가중치를 정확히 0으로 만들 수 있다.
- 중요하지 않은 Feature가 제거되는 효과가 있다.
- Feature가 많고 일부 Feature만 중요하다고 판단될 때 고려할 수 있다.
- scikit-learn에서는 `Lasso`로 적용한다.

---

## 9. L2 정규화와 Ridge

L2 정규화는 가중치 제곱의 합을 Penalty로 사용한다.

$$
L_{L2}(W)=L(W)+\lambda\sum_{j=1}^{p}w_j^2
$$

### 특징

- 전체 가중치의 크기를 부드럽게 줄인다.
- 일반적으로 가중치를 정확히 0으로 만들지는 않는다.
- 특정 Feature의 가중치만 지나치게 커지는 것을 방지한다.
- 대부분의 Feature를 유지하면서 모델을 안정화할 때 고려할 수 있다.
- scikit-learn에서는 `Ridge`로 적용한다.

### L1과 L2 비교

| 구분 | L1 정규화 | L2 정규화 |
|---|---|---|
| Penalty | 가중치 절댓값의 합 | 가중치 제곱의 합 |
| scikit-learn 모델 | `Lasso` | `Ridge` |
| 가중치 변화 | 일부를 0으로 만들 수 있음 | 전체 크기를 줄임 |
| Feature 처리 | 일부 Feature를 선택하는 효과 | 대부분의 Feature를 유지 |
| 적합한 상황 | 일부 Feature만 중요할 때 | 여러 Feature를 안정적으로 활용할 때 |

---

## 10. Lasso와 Ridge 적용하기

고객의 방문 횟수, 장바구니 추가 횟수, 쿠폰 사용 횟수로 구매 금액을 예측하는 예제이다.

```python
import numpy as np
from sklearn.linear_model import Lasso, Ridge

X = np.array([
    [2, 1, 0],
    [4, 2, 1],
    [5, 1, 2],
    [7, 3, 1],
    [8, 4, 3],
    [10, 5, 2],
])

y = np.array([20, 32, 35, 50, 60, 68])

lasso = Lasso(alpha=1.0)
ridge = Ridge(alpha=1.0)

lasso.fit(X, y)
ridge.fit(X, y)

print("Lasso 가중치:", np.round(lasso.coef_, 2))
print("Ridge 가중치:", np.round(ridge.coef_, 2))
```

실행 결과:

```text
Lasso 가중치: [5.62 0.94 0.  ]
Ridge 가중치: [4.34 2.8  1.35]
```

이 예제에서는 Lasso가 세 번째 Feature의 가중치를 0으로 만들었고, Ridge는 세 Feature를 모두 유지하면서 가중치의 크기를 조절했다.

- `Lasso(alpha=1.0)`은 L1 Penalty를 적용한다.
- `Ridge(alpha=1.0)`은 L2 Penalty를 적용한다.
- `coef_`에는 각 Feature에 대해 학습한 가중치가 저장된다.
- 같은 `alpha`를 사용해도 L1과 L2가 가중치를 줄이는 방식은 서로 다르다.

정규화는 훈련 데이터에 대한 오차를 약간 증가시킬 수 있지만, Variance와 Overfitting을 줄여 테스트 데이터에서 더 안정적인 성능을 만들기 위해 사용한다.

---

## 11. 오늘 배운 내용 정리

### Bias-Variance Trade-off

- High Bias는 모델이 너무 단순하여 패턴을 충분히 학습하지 못한 상태이다.
- High Variance는 모델이 너무 복잡하여 학습 데이터의 노이즈까지 학습한 상태이다.
- Expected Error는 Bias², Variance, Irreducible Error로 나눌 수 있다.
- 좋은 모델을 만들려면 Bias와 Variance의 균형을 맞춰야 한다.

### Underfitting과 Overfitting

- Underfitting은 Train과 Test 성능이 모두 낮은 상태이다.
- Overfitting은 Train 성능은 높지만 Test 성능이 낮은 상태이다.
- 모델을 평가할 때는 Train과 Test 성능을 반드시 함께 확인해야 한다.

### 정규화

- 정규화는 큰 가중치에 Penalty를 부과하여 Overfitting을 완화한다.
- L1은 가중치 절댓값의 합을 사용하며 일부 가중치를 0으로 만들 수 있다.
- L2는 가중치 제곱의 합을 사용하며 전체 가중치의 크기를 줄인다.
- `alpha`는 정규화의 강도를 조절한다.

### 전체 흐름

```text
Train·Test 성능 비교
    ↓
Underfitting 또는 Overfitting 진단
    ↓
Bias와 Variance 관점에서 원인 이해
    ↓
모델 복잡도·데이터·정규화 조절
    ↓
새로운 데이터에서의 일반화 성능 개선
```

---

## 12. 느낀 점

오늘은 학습 데이터의 성능이 높다고 해서 반드시 좋은 모델은 아니라는 점을 배웠다. 이전에는 Train Loss가 계속 낮아지면 학습이 잘되고 있다고 생각하기 쉬웠지만, 실제로는 Test Loss와의 차이를 함께 확인해야 Overfitting 여부를 판단할 수 있다는 점이 중요하게 느껴졌다.

또한 Bias와 Variance는 단순히 서로 반대되는 용어가 아니라, 모델이 너무 단순한지 또는 학습 데이터에 지나치게 민감한지를 설명하는 기준이라는 것을 이해했다. 특히 Expected Error가 Bias², Variance, 제거할 수 없는 Noise로 구성된다는 식을 통해 모델의 모든 오차를 학습만으로 없앨 수는 없다는 점도 알게 되었다.

L1과 L2 정규화는 모두 큰 가중치를 제한하지만, L1은 일부 가중치를 0으로 만들어 Feature를 선택하는 효과가 있고 L2는 전체 가중치를 작게 유지한다는 차이가 인상적이었다.

앞으로 모델을 학습할 때는 Train 성능만 확인하지 않고 검증 또는 Test 성능을 함께 기록해야겠다. 또한 `alpha` 값을 여러 수준으로 바꾸면서 Train MSE와 Test MSE, 가중치가 어떻게 달라지는지 직접 비교하여 정규화와 Bias-Variance Trade-off의 관계를 더 익숙하게 만들어야겠다.