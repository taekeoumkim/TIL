# 2026-09-03 선형회귀모델 학습 방법

## 학습 목표

- 회귀 문제에서 손실함수(Loss Function)가 필요한 이유를 설명할 수 있다.
- 평균제곱오차(MSE)를 계산하고 모델의 예측 성능을 비교할 수 있다.
- 최소제곱법(Least Squares Method)의 목적과 폐쇄형 해(Closed-form Solution)를 이해할 수 있다.
- 경사하강법(Gradient Descent)의 학습 과정을 설명하고 NumPy로 구현할 수 있다.
- 학습률(Learning Rate), 에포크(Epoch), 기울기(Gradient)가 모델 학습에 미치는 영향을 이해할 수 있다.

---

## 1. 선형회귀모델은 어떻게 학습할까?

선형회귀는 입력 데이터와 정답의 관계를 가장 잘 설명하는 직선 또는 평면을 찾는 모델이다.

입력 특성이 하나일 때 모델은 다음과 같이 표현할 수 있다.

```text
예측값 = Weight × 입력값 + Bias
ŷ = wx + b
```

좋은 직선을 찾으려면 먼저 예측값과 실제값이 얼마나 다른지 측정해야 한다. 이 차이를 하나의 숫자로 나타내는 것이 **손실함수**이며, 모델 학습은 손실이 작아지도록 `Weight`와 `Bias`를 조정하는 과정이다.

```text
현재 파라미터로 예측
    ↓
실제값과 비교하여 Loss 계산
    ↓
Loss가 작아지도록 파라미터 조정
```

---

## 2. 손실함수(Loss Function)

손실함수는 모델의 예측값과 실제값이 얼마나 차이 나는지를 수치로 나타내는 함수이다.

- Loss가 크다 → 예측값과 실제값의 차이가 크다.
- Loss가 작다 → 예측값이 실제값에 가깝다.
- 모델 학습의 목표 → Loss를 최소화하는 파라미터를 찾는 것이다.

### 평균제곱오차(MSE)

회귀에서는 대표적으로 평균제곱오차(Mean Squared Error, MSE)를 손실함수로 사용한다.

$$
\mathrm{MSE}=\frac{1}{n}\sum_{i=1}^{n}(y_i-\hat{y}_i)^2
$$

- $y_i$: 실제값
- $\hat{y}_i$: 예측값
- $n$: 전체 데이터 개수

### 왜 오차를 제곱할까?

오차를 그대로 더하면 양수 오차와 음수 오차가 서로 상쇄될 수 있다. 예를 들어 오차가 `+5`, `-5`라면 합은 0이지만 실제로는 두 예측 모두 틀렸다.

```text
오차의 합:       5 + (-5) = 0
오차 제곱의 합:  5² + (-5)² = 50
```

오차를 제곱하면 다음과 같은 특징이 생긴다.

- 오차의 부호가 사라져 서로 상쇄되지 않는다.
- 큰 오차일수록 손실에 더 큰 영향을 준다.
- 미분할 수 있어 파라미터 최적화에 활용하기 좋다.

### 두 배달 시간 예측 모델 비교

```python
import numpy as np

y_true = np.array([20, 30, 40, 50])
model_a = np.array([22, 28, 43, 47])
model_b = np.array([25, 25, 45, 55])

mse_a = np.mean((y_true - model_a) ** 2)
mse_b = np.mean((y_true - model_b) ** 2)

print(f"모델 A MSE: {mse_a:.2f}")
print(f"모델 B MSE: {mse_b:.2f}")
```

결과:

```text
모델 A MSE: 6.50
모델 B MSE: 25.00
```

모델 A의 MSE가 더 작으므로 모델 A의 예측값이 전체적으로 실제값에 더 가깝다고 판단할 수 있다.

### 연습: 월 매출 예측 모델 비교

```python
import numpy as np

y_true = np.array([100, 150, 200, 250])
model_a = np.array([110, 145, 195, 260])
model_b = np.array([120, 130, 220, 230])

mse_a = np.mean((y_true - model_a) ** 2)
mse_b = np.mean((y_true - model_b) ** 2)

print(mse_a)  # 62.5
print(mse_b)  # 400.0
```

모델 A의 MSE가 더 작으므로 모델 A가 실제 월 매출을 더 잘 예측한다.

---

## 3. 최소제곱법(Least Squares Method)

최소제곱법은 실제값과 예측값 사이의 **오차 제곱합(Sum of Squared Errors, SSE)**이 가장 작아지는 파라미터를 찾는 방법이다.

$$
\mathrm{SSE}=\sum_{i=1}^{m}(y_i-\hat{y}_i)^2
$$

MSE는 SSE를 데이터 개수로 나눈 값이므로 두 식을 최소화하는 파라미터는 같다.

선형회귀에서는 조건이 맞으면 반복 학습 없이 행렬 연산으로 최적의 파라미터를 한 번에 계산할 수 있다. 이와 같이 수식으로 해를 직접 구하는 방식을 **폐쇄형 해(Closed-form Solution)**라고 한다.

### Bias를 행렬에 포함하기

입력 데이터의 마지막 원소를 항상 1로 두면 Bias도 Weight와 함께 행렬 연산으로 계산할 수 있다.

$$
x=
\begin{bmatrix}
x_1 \\ x_2 \\ \vdots \\ x_n \\ 1
\end{bmatrix},\qquad
W=
\begin{bmatrix}
w_1 \\ w_2 \\ \vdots \\ w_n \\ b
\end{bmatrix}
$$

한 데이터의 예측값은 다음과 같다.

$$
\hat{y}=W^Tx=w_1x_1+w_2x_2+\cdots+w_nx_n+b
$$

전체 데이터의 입력 행렬을 $X$, 정답 벡터를 $Y$라고 하면 예측값은 다음과 같이 표현된다.

$$
\hat{Y}=XW
$$

### 최소제곱법의 목적식

최소제곱법은 아래 목적식을 최소화하는 $W$를 찾는다.

$$
\min_W \lVert Y-XW \rVert^2
=\min_W (Y-XW)^T(Y-XW)
$$

식을 전개하면 다음과 같다.

$$
(Y-XW)^T(Y-XW)
=Y^TY-2Y^TXW+W^TX^TXW
$$

$W$에 대해 미분한 값이 0이 되는 지점을 찾으면 다음과 같다.

$$
2X^TXW-2X^TY=0
$$

$$
X^TXW=X^TY
$$

$X^TX$의 역행렬이 존재한다면 양변에 역행렬을 곱해 최적의 파라미터를 구할 수 있다.

$$
W=(X^TX)^{-1}X^TY
$$

### 광고비와 매출의 관계 계산

```python
import numpy as np

# 광고비와 실제 매출
x = np.array([1, 2, 3, 4])
y = np.array([15, 20, 25, 30])

# Bias를 함께 계산하기 위해 1로 구성된 열 추가
X = np.column_stack([x, np.ones(len(x))])

# Closed-form Solution
W = np.linalg.inv(X.T @ X) @ X.T @ y

weight = W[0]
bias = W[1]

print(f"Weight: {weight:.1f}")
print(f"Bias: {bias:.1f}")
```

결과:

```text
Weight: 5.0
Bias: 10.0
```

따라서 학습된 선형회귀식은 다음과 같다.

```text
예측 매출 = 5 × 광고비 + 10
```

광고비가 3일 때의 예측 매출은 `5 × 3 + 10 = 25`이다.

### 연습: 공부 시간과 시험 점수

```python
import numpy as np

x = np.array([1, 2, 3, 4, 5])
y = np.array([50, 60, 70, 80, 90])

X = np.column_stack([x, np.ones(len(x))])
W = np.linalg.inv(X.T @ X) @ X.T @ y

print(f"Weight: {W[0]:.1f}")  # 10.0
print(f"Bias: {W[1]:.1f}")    # 40.0
```

학습된 식은 `예측 점수 = 10 × 공부 시간 + 40`이다.

### 최소제곱법의 한계

- $X^TX$의 역행렬이 존재해야 위 공식을 그대로 사용할 수 있다.
- Feature가 많아지면 역행렬 계산 비용이 커진다.
- 다양한 머신러닝·딥러닝 모델에는 폐쇄형 해가 존재하지 않을 수 있다.

이러한 한계 때문에 여러 모델에 공통적으로 적용할 수 있는 경사하강법을 사용한다.

---

## 4. 경사하강법(Gradient Descent)

경사하강법은 손실함수의 값을 최소화하는 방향으로 파라미터를 조금씩 업데이트하여 최적의 파라미터를 찾는 반복적인 최적화 알고리즘이다.

안개가 짙은 산에서 가장 낮은 곳을 찾기 위해 현재 위치의 경사를 확인하고, 가장 가파르게 내려가는 방향으로 한 걸음씩 이동하는 과정과 비슷하다.

### 경사하강법의 핵심 원리

- Gradient는 Loss가 가장 빠르게 **증가하는 방향과 크기**를 나타낸다.
- Loss를 줄이려면 Gradient의 **반대 방향**으로 이동해야 한다.
- Learning Rate는 한 번에 이동하는 크기를 결정한다.
- Epoch를 반복하면서 파라미터가 최적값에 가까워진다.

```text
파라미터 초기화
    ↓
예측값 계산
    ↓
Loss 계산
    ↓
Gradient 계산
    ↓
Gradient 반대 방향으로 파라미터 업데이트
    ↓
정해진 Epoch만큼 반복
```

### 1단계: 손실함수 정의

선형회귀의 MSE 손실함수는 다음과 같다.

$$
L(W)=\frac{1}{N}\sum_{i=1}^{N}(y_i-\hat{y}_i)^2
=\frac{1}{N}\lVert Y-XW\rVert^2
$$

### 2단계: Gradient 계산

손실함수를 파라미터 $W$에 대해 미분한다.

$$
\nabla_W L=\frac{2}{N}X^T(XW-Y)
$$

### 3단계: 파라미터 업데이트

손실을 줄이기 위해 Gradient의 반대 방향으로 파라미터를 업데이트한다.

$$
W^{(t+1)}=W^{(t)}-\eta\nabla_WL
$$

이를 선형회귀의 Gradient로 표현하면 다음과 같다.

$$
W^{(t+1)}=W^{(t)}-\eta\frac{2}{N}X^T(XW-Y)
$$

여기서 $\eta$는 Learning Rate이다.

### 4단계: 반복 수행

업데이트 과정을 미리 정한 횟수만큼 반복하거나 Loss가 충분히 작아질 때까지 반복한다. 학습이 잘 진행되면 Loss는 점차 감소하고 파라미터는 손실을 최소화하는 값으로 수렴한다.

### NumPy로 경사하강법 구현

```python
import numpy as np

# 방문 횟수와 실제 구매 금액
x = np.array([1.0, 2.0, 3.0, 4.0])
y = np.array([15.0, 20.0, 25.0, 30.0])

# 파라미터 초기화
weight = 0.0
bias = 0.0

learning_rate = 0.05
epochs = 1000
n = len(x)

for epoch in range(epochs):
    # 1. 예측
    y_pred = weight * x + bias

    # 2. Loss 계산
    error = y_pred - y
    loss = np.mean(error ** 2)

    # 3. Gradient 계산
    weight_gradient = (2 / n) * np.sum(error * x)
    bias_gradient = (2 / n) * np.sum(error)

    # 4. 파라미터 업데이트
    weight = weight - learning_rate * weight_gradient
    bias = bias - learning_rate * bias_gradient

print(f"Weight: {weight:.2f}")
print(f"Bias: {bias:.2f}")
print(f"Loss: {loss:.4f}")
```

실행 결과 예시:

```text
Weight: 5.00
Bias: 10.00
Loss: 0.0000
```

초기값은 `weight = 0`, `bias = 0`이지만, 예측과 업데이트를 반복하면서 데이터의 관계인 `y = 5x + 10`에 가까워진다.

---

## 5. 경사하강법의 핵심 용어

| 용어 | 의미 |
|---|---|
| Parameter | 모델이 학습을 통해 조정하는 값. 선형회귀에서는 Weight와 Bias가 해당한다. |
| Loss Function | 예측값과 실제값의 차이를 하나의 숫자로 나타내는 함수이다. |
| Gradient | Loss가 가장 빠르게 증가하는 방향과 그 크기이다. |
| Learning Rate | 한 번의 업데이트에서 파라미터를 얼마나 변경할지 결정하는 값이다. |
| Epoch | 전체 학습 데이터를 사용하여 학습을 한 번 수행한 횟수이다. |
| Convergence | 반복 학습을 통해 파라미터와 Loss가 최적값에 가까워지는 상태이다. |

Learning Rate가 너무 크면 최솟값을 지나쳐 Loss가 불안정해질 수 있고, 너무 작으면 수렴하는 데 많은 Epoch가 필요할 수 있다.

---

## 6. 최소제곱법과 경사하강법 비교

| 구분 | 최소제곱법 | 경사하강법 |
|---|---|---|
| 해를 찾는 방식 | 행렬 공식으로 한 번에 계산 | Gradient를 이용해 반복적으로 업데이트 |
| 대표식 | $W=(X^TX)^{-1}X^TY$ | $W^{(t+1)}=W^{(t)}-\eta\nabla_WL$ |
| 반복 학습 | 필요 없음 | 필요함 |
| 장점 | 조건을 만족하면 최적해를 직접 계산 | 큰 데이터와 다양한 모델에 적용 가능 |
| 한계 | 역행렬이 필요하고 Feature가 많으면 계산 비용 증가 | Learning Rate와 Epoch 설정이 필요하며 수렴에 시간이 걸림 |

두 방법 모두 **Loss를 최소화하는 파라미터를 찾는다**는 목적은 같지만, 최적값을 찾는 방식이 다르다.

---

## 7. 오늘 배운 내용 정리

### 손실함수

- 모델의 예측값과 실제값의 차이를 수치로 나타낸다.
- 회귀에서는 MSE를 대표적인 손실함수로 사용한다.
- 학습의 목적은 Loss가 작아지도록 파라미터를 조정하는 것이다.

### 최소제곱법

- 오차 제곱합(SSE)이 최소가 되는 파라미터를 찾는다.
- 조건을 만족하면 폐쇄형 해로 최적의 파라미터를 직접 계산할 수 있다.
- $X^TX$의 역행렬이 존재해야 하며, Feature가 많으면 계산 비용이 커진다.

### 경사하강법

- Gradient의 반대 방향으로 파라미터를 조금씩 업데이트한다.
- `예측 → Loss 계산 → Gradient 계산 → 파라미터 업데이트`를 반복한다.
- Learning Rate는 이동 크기, Epoch는 반복 횟수를 결정한다.

### 전체 학습 흐름

```text
선형회귀식 정의: ŷ = wx + b
    ↓
MSE로 예측 오차 측정
    ↓
최소제곱법 또는 경사하강법으로 파라미터 탐색
    ↓
Loss를 최소화하는 Weight와 Bias 학습
```

---

## 8. 느낀 점

오늘은 선형회귀가 단순히 데이터에 직선을 그리는 모델이 아니라, **손실함수를 정의하고 그 값을 최소화하는 Weight와 Bias를 찾는 과정**이라는 점을 배웠다.

특히 최소제곱법과 경사하강법은 같은 목표를 가지지만 접근 방식이 다르다는 점이 인상적이었다. 최소제곱법은 조건이 맞으면 공식으로 최적해를 한 번에 구할 수 있지만, 경사하강법은 현재 위치의 기울기를 이용해 최적값에 조금씩 접근한다.

또한 경사하강법 코드에서 `weight_gradient`와 `bias_gradient`를 직접 계산해 보니, 모델 학습이 막연히 자동으로 이루어지는 것이 아니라 **예측 → 오차 측정 → 미분 → 업데이트**의 반복이라는 것을 더 구체적으로 이해할 수 있었다.

앞으로는 Learning Rate를 너무 크거나 작게 설정했을 때 Loss가 어떻게 달라지는지 직접 실험하고, NumPy로 구현한 결과를 scikit-learn과 PyTorch의 선형회귀 결과와 비교해 보며 학습 과정을 더 깊게 이해해야겠다.
