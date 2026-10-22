# 2026-10-22 TIL | MSE, BCE와 Cross Entropy의 확률적 이해

## 1. 예측 오차와 MSE

회귀 모델은 구매 금액, 배송 시간처럼 연속적인 수치를 예측한다. 실제값을 $y$, 예측값을 $\hat y$라고 하면 예측 오차는 다음과 같다.

$$
e=y-\hat y
$$

오차를 그대로 평균하면 양수와 음수가 상쇄될 수 있다. 예를 들어 오차가 $-5$와 $5$이면 평균은 0이지만, 두 예측 모두 실제값과 5만큼 차이가 난다.

이를 해결하는 방법 중 하나가 오차를 제곱하는 것이다.

$$
\mathrm{SquaredError}=(y-\hat y)^2
$$

### MSE 계산

MSE(Mean Squared Error, 평균제곱오차)는 데이터별 제곱오차의 평균이다.

$$
\mathrm{MSE}=\frac1n\sum_{i=1}^{n}(y_i-\hat y_i)^2
$$

구매 금액을 만 원 단위로 표현한 예시는 다음과 같다.

| 고객 | 실제값 $y$ | 예측값 $\hat y$ | 오차 $y-\hat y$ | 제곱오차 |
| --- | ---: | ---: | ---: | ---: |
| A | 10 | 12 | -2 | 4 |
| B | 20 | 18 | 2 | 4 |
| C | 30 | 27 | 3 | 9 |

$$
\mathrm{SSE}=4+4+9=17,\qquad
\mathrm{MSE}=\frac{17}{3}\approx5.667
$$

SSE(Sum of Squared Errors)는 제곱오차의 합이며, 이를 데이터 개수로 나누면 MSE가 된다.

```python
import numpy as np

y_true = np.array([10, 20, 30])
y_pred = np.array([12, 18, 27])

errors = y_true - y_pred
squared_errors = errors ** 2
mse = np.mean(squared_errors)

print(errors)              # [-2  2  3]
print(squared_errors)      # [4 4 9]
print(round(mse, 3))       # 5.667
```

### MSE를 해석할 때 주의할 점

- 오차가 두 배가 되면 제곱오차는 네 배가 되어, 큰 예측 오차에 더 큰 손실을 부여한다.
- 이상치나 일부 큰 오차가 전체 MSE에 큰 영향을 줄 수 있다.
- 실제값의 단위가 만 원이면 MSE의 단위는 $(\text{만 원})^2$이므로, MSE를 그대로 금액 오차로 해석하면 안 된다.
- 모델은 동일한 평가 데이터와 동일한 단위를 기준으로 비교해야 한다.

## 2. 정규분포와 회귀 모델

정규분포(Normal Distribution)는 평균을 중심으로 좌우 대칭인 종 모양의 확률밀도함수를 갖는 연속확률분포이다.

$$
Y\sim\mathcal N(\mu,\sigma^2)
$$

| 기호 | 의미 | 역할 |
| --- | --- | --- |
| $\mu$ | 평균 | 분포의 중심 위치 |
| $\sigma^2$ | 분산 | 평균 주변에 값들이 퍼져 있는 정도 |
| $\sigma$ | 표준편차 | 분산의 제곱근 |

$\mathcal N(20,4)$와 $\mathcal N(20,25)$는 평균이 같지만, 표준편차가 각각 2와 5이므로 두 번째 분포가 더 넓게 퍼져 있다.

NumPy의 `rng.normal(loc=20, scale=2, size=1000)`에서 `loc`은 평균, `scale`은 **분산이 아니라 표준편차**이다.

### 정규분포의 확률밀도함수

$$
f(y\mid\mu,\sigma^2)
=\frac{1}{\sqrt{2\pi\sigma^2}}
\exp\left(-\frac{(y-\mu)^2}{2\sigma^2}\right),
\qquad \sigma^2>0
$$

분산을 고정하면 관측값 $y$가 평균 $\mu$에 가까울수록 확률밀도가 높다. **확률밀도 값 자체는 확률이 아니며**, 구간에 속할 확률은 해당 구간의 밀도를 적분하여 구한다.

### 회귀 예측값을 조건부 평균으로 해석하기

모델의 예측값을 $f_\theta(\mathbf x)=\hat y$라고 하면 다음과 같이 가정할 수 있다.

$$
Y\mid X=\mathbf x;\theta
\sim\mathcal N\left(f_\theta(\mathbf x),\sigma^2\right)
$$

이는 주어진 입력에서 실제값이 모델의 예측값 주변에 정규분포 형태로 퍼져 있다고 보는 것이다. 오차 관점에서는 다음과 같다.

$$
Y=f_\theta(\mathbf x)+\varepsilon,\qquad
\varepsilon\mid X=\mathbf x\sim\mathcal N(0,\sigma^2)
$$

예측값이 20만 원이라는 것은 이 확률 모델에서 **조건부 평균을 20만 원으로 예측했다**는 의미이다. 실제 관측값이 항상 20만 원이라는 뜻은 아니다.

정규분포 오차는 회귀 모델을 설명하기 위한 가정이며, 모든 회귀 데이터에서 반드시 성립하는 법칙은 아니다.

## 3. Gaussian NLL과 MSE의 연결

정규분포를 가정한 회귀 모델에서는 관측값의 확률밀도로 우도를 구성한다. 한 데이터의 음의 로그우도(Negative Log-Likelihood, NLL)는 다음과 같다.

$$
-\ln f(y_i\mid\mathbf x_i;\theta)
=\frac12\ln(2\pi\sigma^2)
+\frac{(y_i-\hat y_i)^2}{2\sigma^2}
$$

입력과 모델 파라미터가 주어졌을 때 관측값들이 조건부 독립이고, 동일한 분산을 가진다고 가정하면 전체 NLL은 다음과 같다.

$$
\mathrm{NLL}(\theta)
=\frac n2\ln(2\pi\sigma^2)
+\frac{1}{2\sigma^2}\sum_{i=1}^{n}(y_i-\hat y_i)^2
$$

데이터 개수 $n$으로 나누면 평균 NLL을 얻는다.

$$
\mathrm{MeanNLL}(\theta)
=\frac12\ln(2\pi\sigma^2)
+\frac{1}{2\sigma^2}\mathrm{MSE}(\theta)
$$

**동일한 양의 분산을 고정하면 평균 NLL은 ‘상수 + 양의 상수 × MSE’ 형태이다.** 따라서 두 목적함수를 최소화하는 평균 모델의 파라미터는 같다.

$$
\arg\max_\theta L(\theta\mid D)
=\arg\min_\theta\mathrm{NLL}(\theta)
=\arg\min_\theta\mathrm{MSE}(\theta)
$$

MSE와 NLL의 **수치가 같다는 뜻은 아니며, 위 조건에서 최적 파라미터가 같다는 뜻**이다.

### 두 모델 비교

실제값이 `[10, 20, 30]`일 때 모델 A의 예측은 `[12, 18, 27]`, 모델 B의 예측은 `[9, 24, 33]`이라고 하자. 표준편차를 $\sigma=2$, 분산을 $\sigma^2=4$로 고정한다.

| 모델 | MSE | Gaussian 평균 NLL |
| --- | ---: | ---: |
| A | 5.667 | 2.320 |
| B | 8.667 | 2.695 |

```python
import numpy as np

y_true = np.array([10, 20, 30])
pred_a = np.array([12, 18, 27])
pred_b = np.array([9, 24, 33])
variance = 4.0

def gaussian_mean_nll(y_true, y_pred, variance):
    constant = 0.5 * np.log(2 * np.pi * variance)
    error_term = (y_true - y_pred) ** 2 / (2 * variance)
    return np.mean(constant + error_term)

for name, pred in [("A", pred_a), ("B", pred_b)]:
    mse = np.mean((y_true - pred) ** 2)
    mean_nll = gaussian_mean_nll(y_true, pred, variance)
    print(name, round(mse, 3), round(mean_nll, 3))

# A 5.667 2.32
# B 8.667 2.695
```

이 연결은 **조건부 정규분포, 관측값의 조건부 독립, 모든 데이터에 동일한 고정 분산**을 가정한다. 분산까지 학습하거나 입력마다 분산을 다르게 모델링하면 분산 관련 항을 포함한 전체 NLL을 살펴야 한다.

## 4. 베르누이 분포와 Binary Cross Entropy

이진 분류에서는 정답을 $y\in\{0,1\}$로 표현하고, 모델은 클래스 1에 속할 확률 $p$를 예측한다.

$$
z=\mathbf x\cdot\mathbf w+b,\qquad
p=\mathrm{Sigmoid}(z)=\frac{1}{1+e^{-z}}
$$

구매 여부를 예측한다면 $p$는 구매 확률이고, $1-p$는 미구매 확률이다.

### 베르누이 우도에서 BCE 도출하기

베르누이 분포의 확률질량함수는 다음과 같다.

$$
P(Y=y\mid p)=p^y(1-p)^{1-y}
$$

한 데이터의 우도에 자연로그와 음수를 차례로 적용하면 다음과 같다.

$$
\ln L=y\ln p+(1-y)\ln(1-p)
$$

$$
\mathrm{BCE}(y,p)
=-\left[y\ln p+(1-y)\ln(1-p)\right]
$$

BCE(Binary Cross Entropy, 이진 교차 엔트로피)는 **베르누이 분포의 한 데이터에 대한 NLL**이다.

| 실제 정답 | 정답에 부여한 확률 | BCE |
| --- | --- | --- |
| $y=1$ | $p$ | $-\ln p$ |
| $y=0$ | $1-p$ | $-\ln(1-p)$ |

예를 들어 $p=0.9$이면 실제 정답이 1일 때 손실은 약 0.105, 실제 정답이 0일 때 손실은 약 2.303이다.

### 여러 데이터의 평균 BCE

$$
\mathrm{MeanBCE}
=-\frac1n\sum_{i=1}^{n}
\left[y_i\ln p_i+(1-y_i)\ln(1-p_i)\right]
$$

| 고객 | 실제 정답 | 예측 구매 확률 | 정답에 부여한 확률 | BCE |
| --- | ---: | ---: | ---: | ---: |
| A | 1 | 0.9 | 0.9 | 0.105 |
| B | 0 | 0.2 | 0.8 | 0.223 |
| C | 1 | 0.6 | 0.6 | 0.511 |
| D | 0 | 0.7 | 0.3 | 1.204 |

평균 BCE는 약 0.511이다. 고객 D는 실제 정답인 미구매에 낮은 확률을 부여했으므로 손실이 가장 크다.

```python
import numpy as np

y_true = np.array([1, 0, 1, 0])
y_prob = np.array([0.9, 0.2, 0.6, 0.7])

# 로그 계산을 위해 확률을 0과 1에서 조금 떨어뜨린다.
eps = 1e-15
p = np.clip(y_prob, eps, 1 - eps)

bce_each = -(
    y_true * np.log(p)
    + (1 - y_true) * np.log(1 - p)
)

print(np.round(bce_each, 3))         # [0.105 0.223 0.511 1.204]
print(round(np.mean(bce_each), 3))  # 0.511
```

확률이 정확히 0 또는 1이면 직접 로그를 계산하는 과정에서 무한대나 `nan`이 발생할 수 있다. 위 코드는 학습용으로 클리핑을 사용하며, 실제 학습에서는 로짓으로부터 손실을 안정적으로 계산하는 구현을 활용할 수 있다.

## 5. 교차 엔트로피의 의미와 분류 결과의 차이

교차 엔트로피(Cross Entropy)는 정답 분포 $q$와 모델의 예측 분포 $p$를 이용해 다음과 같이 계산한다.

$$
H(q,p)=-\sum_c q(c)\ln p(c)
$$

정답이 구매라면 클래스 순서를 미구매·구매로 두었을 때 정답 분포는 $[0,1]$이다. 모델의 예측 분포가 $[0.2,0.8]$이면 다음과 같다.

$$
H(q,p)=-\left[0\ln0.2+1\ln0.8\right]
=-\ln0.8\approx0.223
$$

정답이 하나인 경우 정답 분포의 1이 해당 클래스의 확률을 선택한다. 교차 엔트로피는 두 분포의 차이를 반영하지만, 일반적인 거리 함수와 동일한 개념은 아니다.

### 높은 확신으로 틀리면 손실이 커진다

실제 정답이 1일 때 예측 확률에 따른 BCE는 다음과 같다.

| 정답에 부여한 확률 | BCE |
| ---: | ---: |
| 0.99 | 0.010 |
| 0.90 | 0.105 |
| 0.50 | 0.693 |
| 0.10 | 2.303 |
| 0.01 | 4.605 |

정답 확률이 0에 가까워질수록 손실은 매우 커진다. 이는 모델이 **틀린 예측을 높은 확신으로 할 때 큰 손실을 부여한다**는 의미이다.

### 손실 계산과 최종 클래스 결정 구분하기

실제 정답이 1이고 임계값을 0.5로 정하면 다음 두 예측은 모두 맞는다.

| 예측 구매 확률 | 최종 클래스 | BCE |
| ---: | ---: | ---: |
| 0.9 | 1 | 0.105 |
| 0.6 | 1 | 0.511 |

정확도는 두 예측을 모두 정답으로 평가하지만, BCE는 정답에 부여한 확률의 차이를 반영한다. 따라서 학습 손실은 보통 임계값을 적용한 최종 클래스가 아니라 **예측 확률 또는 로짓**으로 계산한다.

관측값의 조건부 독립을 가정한 베르누이 모델에서는 다음 관계가 성립한다.

$$
\mathrm{NLL}(\theta)=n\times\mathrm{MeanBCE}(\theta)
$$

동일한 데이터셋에서 BCE 최소화와 우도 최대화는 같은 최적화 목표를 갖는다. 정규화 항이나 추가 제약이 포함되면 전체 학습 목적함수도 함께 살펴야 한다.

## 6. 범주분포와 Categorical Cross Entropy

다중 클래스 분류는 여러 상호 배타적인 클래스 중 하나를 정답으로 선택하는 문제이다. 의류·식품·가전을 각각 클래스 0·1·2로 표현할 수 있다.

범주분포(Categorical Distribution)의 클래스별 확률은 다음 조건을 만족한다.

$$
p_k\geq0,\qquad \sum_{k=0}^{K-1}p_k=1
$$

### One-Hot Encoding과 우도

One-Hot Encoding은 정답 클래스의 위치를 1, 나머지를 0으로 표시한다.

| 정답 | 클래스 인덱스 | One-Hot 표현 |
| --- | ---: | --- |
| 의류 | 0 | $[1,0,0]$ |
| 식품 | 1 | $[0,1,0]$ |
| 가전 | 2 | $[0,0,1]$ |

정답의 One-Hot 벡터를 $\mathbf y$, 모델의 확률 벡터를 $\mathbf p$라고 하면 한 데이터의 우도는 다음과 같다.

$$
L(\mathbf p\mid\mathbf y)=\prod_{k=0}^{K-1}p_k^{y_k}
$$

실제 정답이 식품이고 예측 확률이 $[0.2,0.7,0.1]$이면 다음과 같다.

$$
L=0.2^0\times0.7^1\times0.1^0=0.7
$$

### Categorical Cross Entropy 도출하기

우도에 음의 로그를 적용하면 범주형 교차 엔트로피를 얻는다.

$$
\mathrm{CE}(\mathbf y,\mathbf p)
=-\ln\left(\prod_{k=0}^{K-1}p_k^{y_k}\right)
=-\sum_{k=0}^{K-1}y_k\ln p_k
$$

정답 클래스의 인덱스를 $c$라고 하면 One-Hot 표현에서 정답에 해당하는 항만 남는다.

$$
\mathrm{CE}=-\ln p_c
$$

위 식품 예시의 손실은 $-\ln0.7\approx0.357$이다. **가장 높은 예측 확률이 아니라, 실제 정답 클래스에 부여한 확률을 선택해야 한다.**

### 평균 CE와 MLE

데이터 $n$개의 평균 CE는 다음과 같다.

$$
\mathrm{MeanCE}
=-\frac1n\sum_{i=1}^{n}\sum_{k=0}^{K-1}y_{ik}\ln p_{ik}
=-\frac1n\sum_{i=1}^{n}\ln p_{i,c_i}
$$

| 상품 | 실제 정답 | 의류 확률 | 식품 확률 | 가전 확률 | CE |
| --- | --- | ---: | ---: | ---: | ---: |
| A | 의류 | 0.8 | 0.1 | 0.1 | 0.223 |
| B | 식품 | 0.2 | 0.6 | 0.2 | 0.511 |
| C | 가전 | 0.1 | 0.3 | 0.6 | 0.511 |

세 예측 모두 최종 클래스는 정답이지만 손실은 같지 않으며, 평균 CE는 약 0.415이다.

관측값의 조건부 독립을 가정하면 정답 확률의 곱으로 전체 우도를 계산할 수 있다.

$$
L=0.8\times0.6\times0.6=0.288
$$

$$
\mathrm{NLL}=-\ln0.288\approx1.245,\qquad
\mathrm{MeanCE}=\frac{\mathrm{NLL}}3\approx0.415
$$

범주분포의 우도 최대화는 NLL 최소화, 평균 CE 최소화와 연결된다. 데이터가 많을 때는 작은 확률들을 직접 곱하기보다 **로그를 먼저 계산한 뒤 합산**하는 것이 수치적으로 안정적이다.

## 7. NumPy 구현과 Softmax의 연결

### 클래스 인덱스로 정답 확률 선택하기

정답 인덱스 배열의 크기는 $(n,)$, 예측 확률 배열의 크기는 $(n,K)$이다. 각 행은 데이터 하나, 각 열은 클래스 하나에 대응한다.

```python
import numpy as np

y_true = np.array([0, 1, 2])
y_prob = np.array([
    [0.8, 0.1, 0.1],
    [0.2, 0.6, 0.2],
    [0.1, 0.3, 0.6],
])

true_class_prob = y_prob[np.arange(len(y_true)), y_true]
ce_each = -np.log(true_class_prob)

print(y_true.shape, y_prob.shape)  # (3,) (3, 3)
print(true_class_prob)             # [0.8 0.6 0.6]
print(np.round(ce_each, 3))         # [0.223 0.511 0.511]
print(round(np.mean(ce_each), 3))  # 0.415
```

`np.arange(len(y_true))`는 행 인덱스를, `y_true`는 각 행에서 선택할 정답 열 인덱스를 제공한다. One-Hot 배열을 직접 만들지 않아도 같은 CE를 계산할 수 있다.

One-Hot 표현으로 계산하면 다음과 같다.

```python
# 위 예시의 y_true와 y_prob를 사용한다.
y_onehot = np.eye(y_prob.shape[1])[y_true]
ce_onehot = -np.sum(y_onehot * np.log(y_prob), axis=1)

print(np.round(ce_onehot, 3))         # [0.223 0.511 0.511]
print(round(np.mean(ce_onehot), 3))  # 0.415
```

`axis=1`은 각 행의 클래스 방향으로 합산한다는 뜻이다. 두 예시의 확률은 모두 양수이며, 0이 포함된 확률을 직접 로그로 계산할 때는 별도의 처리가 필요하다.

### Softmax로 로짓을 확률로 변환하기

다중 클래스 모델은 클래스별 점수인 로짓을 계산하고, Softmax로 확률을 구한다.

$$
\mathbf z=\mathbf xW+\mathbf b,\qquad
p_k=\frac{\exp(z_k)}{\sum_{j=0}^{K-1}\exp(z_j)}
$$

```python
import numpy as np

def softmax(z):
    shifted = z - np.max(z, axis=1, keepdims=True)
    exp_z = np.exp(shifted)
    return exp_z / np.sum(exp_z, axis=1, keepdims=True)

logits = np.array([
    [2, 1, 0],
    [0, 2, 1],
    [1, 0, 3],
])
y_true = np.array([0, 1, 2])

probabilities = softmax(logits)
true_class_prob = probabilities[np.arange(len(y_true)), y_true]
ce_each = -np.log(true_class_prob)

print(np.round(probabilities, 3))
# [[0.665 0.245 0.09 ]
#  [0.09  0.665 0.245]
#  [0.114 0.042 0.844]]

print(np.round(ce_each, 3))         # [0.408 0.408 0.17 ]
print(round(np.mean(ce_each), 3))  # 0.328
```

각 행에서 최댓값을 빼도 Softmax 결과는 같으며, 지수 계산에서 오버플로가 발생할 위험을 줄인다. 다만 극단적인 로짓에서는 확률을 만든 뒤 로그를 취하는 계산에 여전히 수치 문제가 생길 수 있어, 실제 학습에서는 로짓에서 손실을 직접 계산하는 구현을 사용한다.

**Softmax는 확률을 계산하고, Argmax는 최종 클래스를 선택하며, CE는 정답에 부여한 확률로 손실을 계산한다.** 세 연산의 역할을 구분해야 한다.

## 8. 세 가지 손실 함수 비교

| 구분 | MSE | Binary Cross Entropy | Categorical Cross Entropy |
| --- | --- | --- | --- |
| 대표 문제 | 회귀 | 이진 분류 | 다중 클래스 분류 |
| 모델이 예측하는 값 | 연속적인 수치 | 클래스 1의 확률 | 클래스별 확률 |
| 확률적 해석 | 고정 분산 정규분포 | 베르누이 분포 | 범주분포 |
| 대표 출력 방식 | 수치 직접 출력 | Sigmoid | Softmax |
| 데이터 한 개의 손실 | $(y-\hat y)^2$ | $-[y\ln p+(1-y)\ln(1-p)]$ | $-\sum_k y_k\ln p_k$ |
| NLL과의 관계 | 고정 분산 조건에서 같은 최적 평균 파라미터 | 평균 BCE = 전체 NLL / $n$ | 평균 CE = 전체 NLL / $n$ |
| 손실이 작은 경우 | 예측 수치가 실제값에 가까움 | 실제 정답에 높은 확률 부여 | 실제 정답 클래스에 높은 확률 부여 |

MSE는 분류에서 절대로 사용할 수 없는 손실은 아니다. 다만 베르누이 확률 모델을 사용하는 이진 분류에서는 BCE가 NLL과 직접 연결된다.

또한 MSE·BCE·CE는 평가 대상과 계산 방식이 다르므로 **서로 다른 종류의 손실값 크기를 그대로 비교해 모델의 우열을 판단하면 안 된다.**

### 다중 클래스와 다중 레이블 구분하기

| 구분 | 다중 클래스 분류 | 다중 레이블 분류 |
| --- | --- | --- |
| 한 데이터의 정답 | 여러 클래스 중 하나 | 여러 레이블이 동시에 정답 가능 |
| 예시 | 상품을 의류·식품·가전 중 하나로 분류 | 이미지에 사람·자동차·나무가 각각 있는지 판단 |
| 대표 출력 | Softmax | 레이블별 Sigmoid |
| 대표 손실 | Categorical CE | 레이블별 BCE |
| 출력 확률의 합 | 1 | 1일 필요 없음 |

손실 함수는 클래스 개수만으로 고르는 것이 아니라, **정답의 구조와 확률 모델의 가정**에 맞춰 선택해야 한다. 학습 데이터의 손실이 작더라도 새 데이터의 성능이 보장되는 것은 아니므로 검증 데이터의 손실과 목적에 맞는 평가 지표도 함께 확인한다.

## 핵심 정리

- MSE는 오차의 부호 상쇄를 막고, 큰 예측 오차에 더 큰 손실을 부여한다.
- 회귀 오차에 독립적인 정규분포와 동일한 고정 분산을 가정하면, Gaussian NLL 최소화와 MSE 최소화는 같은 최적 평균 파라미터를 찾는다.
- BCE는 베르누이 분포의 NLL에서, Categorical CE는 범주분포의 NLL에서 도출된다.
- 정답이 하나인 분류에서 두 교차 엔트로피 손실은 정답에 부여한 확률의 음의 로그로 이해할 수 있다.
- 분류 손실은 최종 클래스의 정답 여부뿐 아니라 정답에 부여한 확률도 반영한다.
- 손실 계산에 사용하는 확률·로짓과 최종 예측에 사용하는 Threshold·Argmax를 구분해야 한다.
- 손실 함수를 선택할 때는 예측 대상, 정답 구조, 확률적 가정과 수치적 안정성을 함께 고려한다.

## 오늘의 한 문장

> MSE, BCE, Categorical Cross Entropy는 서로 다른 확률 모델에서 출발하지만, 관측 데이터를 더 잘 설명하도록 모델 파라미터를 학습한다는 최대우도추정의 관점으로 연결된다.