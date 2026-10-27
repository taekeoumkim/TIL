# 2026-10-27 TIL | SGD, Momentum, Adam과 학습률

## 1. Optimizer의 역할

Optimizer는 손실을 줄이기 위해 **역전파로 계산한 Gradient를 이용하여 가중치와 편향을 업데이트하는 알고리즘**이다.

| 단계 | 역할 | PyTorch 예시 |
| --- | --- | --- |
| 순전파 | 현재 파라미터로 예측값 계산 | `logits = model(X)` |
| 손실 계산 | 예측값과 실제 정답 비교 | `loss = loss_fn(logits, y)` |
| 역전파 | 파라미터별 Gradient 계산 | `loss.backward()` |
| Optimizer 업데이트 | 가중치와 편향 변경 | `optimizer.step()` |

Gradient를 계산하는 것과 파라미터를 변경하는 것은 서로 다른 단계이다. `loss.backward()`만 실행하면 기울기는 계산되지만, Optimizer의 업데이트는 수행되지 않는다.

### SGD와 Batch Size 구분하기

기초수학에서는 데이터 하나로 업데이트하는 방식을 SGD라고 구분했다. 하지만 PyTorch의 `torch.optim.SGD`는 데이터 하나뿐 아니라 Mini-Batch나 전체 데이터로 계산한 Gradient에도 사용할 수 있다.

**Optimizer는 업데이트 규칙이고, Batch Size는 한 번의 Gradient 계산에 사용하는 데이터 수이다.** 두 설정을 구분해야 한다.

## 2. 기본 SGD의 업데이트

기본 SGD(Stochastic Gradient Descent)는 현재 Gradient와 학습률로 업데이트한다.

$$
\theta_t=\theta_{t-1}-\eta g_t
$$

$$
\Delta\theta_t=-\eta g_t
$$

| 기호 | 의미 |
| --- | --- |
| $\theta_{t-1}$ | 업데이트 전 파라미터 |
| $\theta_t$ | 업데이트 후 파라미터 |
| $g_t$ | 현재 단계의 Gradient |
| $\eta$ | 학습률(Learning Rate) |
| $t$ | 업데이트 단계 |

추가 옵션이 없는 기본 SGD에서는 Gradient가 양수이면 파라미터를 줄이고, 음수이면 늘린다. 현재 Gradient가 0이면 변경량도 0이다.

### 수치 예제

초기 파라미터를 0, 학습률을 0.1로 두고, Gradient가 차례대로 $[-0.8,\;0.2,\;-0.6]$이라고 가정한다.

| 단계 | 현재 Gradient | 변화량 $-\eta g_t$ | 업데이트 후 파라미터 |
| ---: | ---: | ---: | ---: |
| 초기 | — | — | 0.00 |
| 1 | -0.8 | +0.08 | 0.08 |
| 2 | +0.2 | -0.02 | 0.06 |
| 3 | -0.6 | +0.06 | 0.12 |

기본 SGD는 이전 Gradient를 별도의 상태로 저장하지 않으므로, 현재 Gradient의 부호가 바뀌면 업데이트 방향도 즉시 바뀐다.

Mini-Batch에 포함된 데이터가 달라지면 Gradient의 방향과 크기도 달라질 수 있다. 이러한 변화 자체가 잘못된 학습을 의미하는 것은 아니다.

## 3. Momentum: 이전 Gradient의 누적 정보 반영

Momentum은 현재 Gradient에 **이전 Gradient를 통해 누적된 방향의 정보**를 함께 반영한다.

여기서는 Momentum 버퍼를 $u_t$로 표기한다.

$$
u_t=\beta u_{t-1}+g_t
$$

$$
\theta_t=\theta_{t-1}-\eta u_t
$$

$\beta$는 이전 Momentum을 얼마나 유지할지 정하는 계수이며, 초기값은 $u_0=0$으로 둔다. 이 식은 추가적인 Weight Decay, Dampening, Nesterov 등을 사용하지 않는 기본적인 SGD Momentum의 경우를 설명한다.

### SGD와 같은 Gradient로 비교하기

초기 파라미터 0, 학습률 0.1, Momentum 계수 $\beta=0.5$를 사용한다.

| 단계 | Gradient $g_t$ | Momentum $u_t$ | 변화량 $-\eta u_t$ | 업데이트 후 파라미터 |
| ---: | ---: | ---: | ---: | ---: |
| 1 | -0.8 | -0.8 | +0.08 | 0.08 |
| 2 | +0.2 | -0.2 | +0.02 | 0.10 |
| 3 | -0.6 | -0.7 | +0.07 | 0.17 |

두 번째 단계에서는 다음과 같이 계산한다.

$$
u_2=0.5\times(-0.8)+0.2=-0.2
$$

$$
\Delta\theta_2=-0.1\times(-0.2)=+0.02
$$

현재 Gradient는 양수이지만 이전의 음수 Gradient 정보가 남아 있어 파라미터가 계속 증가한다. 기본 SGD에서는 같은 단계의 변화량이 $-0.02$였다.

같은 방향의 Gradient가 반복되면 누적된 정보로 업데이트가 커질 수 있다. 이러한 성질은 일부 상황에서 진동을 줄이거나 이동을 빠르게 하는 데 도움이 된다.

다만 학습률과 Momentum 계수에 따라 최솟값을 지나치거나 불안정해질 수도 있으므로, 항상 더 빠르고 안정적인 것은 아니다.

## 4. Adam: 두 이동평균을 이용한 적응적 업데이트

Adam(Adaptive Moment Estimation)은 각 파라미터에 대해 다음 두 값을 관리한다.

| 상태값 | 사용하는 정보 | 주요 역할 |
| --- | --- | --- |
| $m_t$ | Gradient의 이동평균 | 기울기의 방향과 변화 이력 반영 |
| $v_t$ | Gradient 제곱의 이동평균 | 기울기 크기의 변화 이력 반영 |

### 두 이동평균 계산

$$
m_t=\beta_1m_{t-1}+(1-\beta_1)g_t
$$

$$
v_t=\beta_2v_{t-1}+(1-\beta_2)g_t^2
$$

초기값은 일반적으로 $m_0=v_0=0$이다. 제곱한 Gradient는 부호에 관계없이 0 이상이므로, $v_t$는 크기에 관한 정보를 반영한다.

### 초기 편향 보정

초기 이동평균이 0에서 시작한 영향을 보정한다.

$$
\hat m_t=\frac{m_t}{1-\beta_1^t},\qquad
\hat v_t=\frac{v_t}{1-\beta_2^t}
$$

여기서 편향 보정(Bias Correction)은 **이동평균의 0 초기화에 따른 영향을 보정하는 과정**이다.

### 파라미터 업데이트

$$
\theta_t
=\theta_{t-1}
-\eta\frac{\hat m_t}{\sqrt{\hat v_t}+\epsilon}
$$

$\epsilon$은 분모가 0이 되는 것을 막고 수치 계산을 안정화하기 위한 작은 값이다. 각 이동평균은 파라미터별로 관리하며, 여러 파라미터에서는 제곱·제곱근·나눗셈을 원소별로 적용한다.

Adam의 실제 변화량은 현재 Gradient에 학습률만 곱한 값이 아니라, **누적된 두 이동평균과 학습률을 함께 반영한 값**이다.

### 수치 예제

앞의 Gradient $[-0.8,\;0.2,\;-0.6]$에 다음 설정을 사용한다.

| 설정 | 값 |
| --- | --- |
| 초기 파라미터 | 0 |
| 학습률 $\eta$ | 0.1 |
| $\beta_1,\beta_2$ | 각각 0.5 |
| 초기 이동평균 $m_0,v_0$ | 각각 0 |
| $\epsilon$ | $10^{-8}$ |

계산을 쉽게 하려고 두 계수를 0.5로 설정했다. 일반적으로 사용하는 Adam 설정의 예로는 $\beta_1=0.9$, $\beta_2=0.999$가 있다.

첫 단계에서는 다음과 같다.

$$
m_1=0.5\times(-0.8)=-0.4,\qquad
v_1=0.5\times(-0.8)^2=0.32
$$

$$
\hat m_1=-0.8,\qquad \hat v_1=0.64
$$

$$
\Delta\theta_1
=-0.1\frac{-0.8}{\sqrt{0.64}+\epsilon}
\approx+0.1
$$

이후 계산을 정리하면 다음과 같다. 표의 값은 표시를 위해 반올림했다.

| 단계 | $m_t$ | $v_t$ | $\hat m_t$ | $\hat v_t$ | Adam 변화량 |
| ---: | ---: | ---: | ---: | ---: | ---: |
| 1 | -0.4000 | 0.3200 | -0.8000 | 0.6400 | +0.1000 |
| 2 | -0.1000 | 0.1800 | -0.1333 | 0.2400 | +0.0272 |
| 3 | -0.3500 | 0.2700 | -0.4000 | 0.3086 | +0.0720 |

두 번째 Gradient는 양수이지만 이동평균 $\hat m_2$는 여전히 음수이므로, Adam도 이전 이력의 영향을 받아 파라미터를 증가시킨다.

## 5. 세 Optimizer의 계산 결과 비교

### 동일한 Gradient를 주었을 때

앞에서 사용한 조건에서 업데이트 후 파라미터는 다음과 같다.

| 단계 | Gradient | SGD | SGD + Momentum | Adam |
| ---: | ---: | ---: | ---: | ---: |
| 초기 | — | 0.0000 | 0.0000 | 0.0000 |
| 1 | -0.8 | 0.0800 | 0.0800 | 0.1000 |
| 2 | +0.2 | 0.0600 | 0.1000 | 0.1272 |
| 3 | -0.6 | 0.1200 | 0.1700 | 0.1992 |

이 예제는 **계산 방식의 차이를 비교하려고 동일한 Gradient를 주었다고 가정한 것**이다. 실제 학습에서는 Optimizer마다 파라미터가 다르게 업데이트되므로, 다음 단계의 Gradient도 달라질 수 있다.

따라서 마지막 파라미터 값이나 이동량이 크다는 이유만으로 더 좋은 Optimizer라고 판단할 수 없다.

### NumPy로 계산 확인하기

```python
import numpy as np

gradients = np.array([-0.8, 0.2, -0.6])
lr = 0.1
momentum = 0.5
beta1 = beta2 = 0.5
eps = 1e-8

theta_sgd = theta_momentum = theta_adam = 0.0
buffer = m = v = 0.0

for t, grad in enumerate(gradients, start=1):
    # 기본 SGD
    theta_sgd -= lr * grad

    # Momentum
    buffer = momentum * buffer + grad
    theta_momentum -= lr * buffer

    # Adam: 이동평균, 편향 보정, 업데이트
    m = beta1 * m + (1 - beta1) * grad
    v = beta2 * v + (1 - beta2) * grad ** 2
    m_hat = m / (1 - beta1 ** t)
    v_hat = v / (1 - beta2 ** t)
    theta_adam -= lr * m_hat / (np.sqrt(v_hat) + eps)

    print(
        t,
        f"{theta_sgd:.4f}",
        f"{theta_momentum:.4f}",
        f"{theta_adam:.4f}",
    )

# 1 0.0800 0.0800 0.1000
# 2 0.0600 0.1000 0.1272
# 3 0.1200 0.1700 0.1992
```

### Adam의 파라미터별 적응성

학습률 0.1에서 두 파라미터의 첫 Gradient가 각각 $-2.0$, $+0.1$이라고 하자. 이동평균을 0으로 초기화하고 편향을 보정하면 첫 단계에서 $\hat m_1=g_1$, $\hat v_1=g_1^2$이다.

| 파라미터 | 첫 Gradient | SGD 변화량 | Adam 변화량 |
| --- | ---: | ---: | ---: |
| $w_1$ | -2.0 | +0.20 | 약 +0.10 |
| $w_2$ | +0.1 | -0.01 | 약 -0.10 |

Adam의 첫 변화량은 다음처럼 정리된다.

$$
\Delta\theta_1
=-\eta\frac{g_1}{|g_1|+\epsilon}
$$

이 예제처럼 Gradient의 절댓값이 $\epsilon$보다 충분히 크면 첫 업데이트 크기가 거의 같아질 수 있다. 하지만 이후에도 항상 같은 크기로 업데이트하는 것은 아니다.

두 번째 Gradient가 각각 $-1.5$, $-0.2$이면, $\beta_1=\beta_2=0.5$에서 다음과 같다.

| 파라미터 | 두 번째 Gradient | $\hat m_2$ | $\hat v_2$ | Adam 변화량 |
| --- | ---: | ---: | ---: | ---: |
| $w_1$ | -1.5 | -1.6667 | 2.8333 | 약 +0.0990 |
| $w_2$ | -0.2 | -0.1000 | 0.0300 | 약 +0.0577 |

**Adam은 각 파라미터의 Gradient 이력을 반영한다.** 작은 Gradient를 무조건 크게, 큰 Gradient를 무조건 작게 바꾸는 규칙으로 이해하면 안 된다.

## 6. 학습률과 수렴·진동·발산

학습률은 실제 파라미터 변경 크기를 조절한다. 기본 SGD에서는 다음과 같다.

$$
\Delta w=-\eta g
$$

Momentum과 Adam도 학습률을 사용하지만, 현재 Gradient를 그대로 곱하는 대신 누적 정보로 계산한 업데이트에 학습률을 반영한다.

따라서 **SGD와 Adam의 학습률을 같게 설정해도 실제 변경량이 같지는 않다.** Adam에서도 학습률 설정은 중요하다.

### 학습률별 한 번의 업데이트

다음 손실 함수를 사용한다.

$$
L(w)=(w-3)^2,\qquad \frac{dL}{dw}=2(w-3)
$$

초기값은 $w=1$이고, 초기 손실은 4, Gradient는 $-4$이다.

| 학습률 | 업데이트 후 $w$ | 업데이트 후 손실 | 해석 |
| ---: | ---: | ---: | --- |
| 0.05 | 1.2 | 3.24 | 작은 폭으로 손실 감소 |
| 0.20 | 1.8 | 1.44 | 손실 감소 |
| 0.80 | 4.2 | 1.44 | 최솟값을 지나쳤지만 손실 감소 |
| 1.10 | 5.4 | 5.76 | 손실 증가 |

최솟값을 지나치는 Overshooting이 생겼다고 반드시 손실이 증가하는 것은 아니다. 최솟값의 양쪽을 오가며 가까워질 수도 있고, 이동 폭이 커지며 발산할 수도 있다.

학습률 0.2와 0.8은 이 예제에서 손실 감소가 같아도 이동 경로가 다르다. 이러한 값이 모든 DNN에 적절한 학습률이라는 뜻은 아니다.

### PyTorch로 다섯 번 업데이트하기

```python
import torch

for lr in [0.05, 0.2, 0.8, 1.1]:
    w = torch.nn.Parameter(torch.tensor(1.0))
    optimizer = torch.optim.SGD([w], lr=lr)
    losses = []

    for _ in range(5):
        optimizer.zero_grad()
        loss = (w - 3) ** 2
        loss.backward()
        optimizer.step()

        with torch.no_grad():
            losses.append(((w - 3) ** 2).item())

    print(
        f"lr={lr}: w={w.item():.4f}, "
        f"loss={losses[-1]:.4f}"
    )
```

동일한 초기값에서 다섯 번 업데이트한 결과는 다음과 같다.

| 학습률 | 최종 $w$ | 최종 손실 |
| ---: | ---: | ---: |
| 0.05 | 1.8190 | 1.3947 |
| 0.20 | 2.8445 | 0.0242 |
| 0.80 | 3.1555 | 0.0242 |
| 1.10 | 7.9766 | 24.7669 |

코드에서 `no_grad()`는 업데이트 후 손실을 기록하는 구간에만 사용한다. Gradient 계산에 필요한 손실 계산과 역전파는 추적이 활성화된 상태에서 수행한다.

## 7. PyTorch에서 Optimizer 사용하기

### Optimizer 생성 예시

| 방법 | 생성 예시 |
| --- | --- |
| 기본 SGD | `torch.optim.SGD(model.parameters(), lr=0.01)` |
| SGD + Momentum | `torch.optim.SGD(model.parameters(), lr=0.01, momentum=0.9)` |
| Adam | `torch.optim.Adam(model.parameters(), lr=0.001)` |

위 학습률과 계수는 사용법을 보여주는 예시이다. 앞의 수치 계산에서 사용한 학습률 0.1과 계수 0.5는 계산 비교를 위한 별도의 조건이다.

Optimizer는 관리할 파라미터와 학습률 등을 받아 생성한다. Momentum과 Adam의 상태값은 업데이트를 거치며 유지되어야 하므로, 일반적인 학습에서는 Optimizer를 반복문 밖에서 만든다.

### 기본 학습 흐름

아래 코드는 로짓을 반환하는 `model`, 데이터 `X`·`y`, `BCEWithLogitsLoss` 같은 `loss_fn`을 이미 준비한 경우의 사용 예시이다.

```python
import torch

optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
model.train()

for _ in range(3):
    optimizer.zero_grad()
    logits = model(X)
    loss = loss_fn(logits, y)
    loss.backward()
    optimizer.step()
```

| 코드 | 역할 |
| --- | --- |
| `optimizer.zero_grad()` | 이전 Gradient 초기화 |
| `loss.backward()` | 현재 손실의 Gradient 계산 |
| `optimizer.step()` | Gradient와 Optimizer 상태를 이용한 업데이트 |

PyTorch 예제는 PyTorch가 설치된 환경에서 실행한다.

## 8. Optimizer와 학습률을 비교하는 기준

| 구분 | 기본 SGD | SGD + Momentum | Adam |
| --- | --- | --- | --- |
| 현재 Gradient 활용 | 사용 | 사용 | 사용 |
| 이전 Gradient 정보 | 별도 누적 없음 | Momentum 버퍼 | Gradient 이동평균 |
| 제곱 Gradient 이동평균 | 사용하지 않음 | 사용하지 않음 | 사용 |
| 업데이트의 특징 | 현재 기울기에 직접 비례 | 누적된 방향의 영향 반영 | 파라미터별 이력으로 조절 |
| 추가 상태 | 기본적으로 없음 | 버퍼 | 두 이동평균과 단계 정보 |

추가 상태를 저장하는 방법은 기본 SGD보다 메모리가 더 필요하다. 또한 Momentum과 Adam은 현재 Gradient가 방향을 바꾸더라도 과거 정보의 영향을 받아 이전 방향으로 이동할 수 있다.

특정 Optimizer가 항상 더 빠르게 수렴하거나 더 좋은 검증 성능을 보장하지는 않는다. 실제 비교에서는 초기 가중치, 데이터 구성, Batch Size와 학습 횟수 등을 맞추고, 각 Optimizer에 맞는 학습률도 함께 검토한다.

### 학습 중 관찰할 내용

| 관찰한 현상 | 검토할 사항 |
| --- | --- |
| 손실이 매우 천천히 감소 | 학습률, Gradient 크기, 모델 구조 |
| 손실이 안정적으로 감소 | 학습 추세와 검증 성능 |
| 손실이 크게 진동 | 학습률, Momentum 등 설정, Batch별 변동 |
| 손실이 급격히 증가 | 학습률, Gradient, 데이터 스케일과 수치 상태 |

손실 변화의 원인이 항상 학습률인 것은 아니다. 초기화, 활성화 함수, 데이터 특성과 모델 구조도 영향을 미친다.

**학습 손실이 감소하는 속도와 최종 검증 성능은 함께 평가해야 한다.** Adam의 적응적 업데이트가 학습률 조정을 대신해 주거나, 더 좋은 일반화 성능을 자동으로 보장하는 것은 아니다.

## 핵심 정리

- Optimizer는 역전파로 계산한 Gradient를 이용해 파라미터를 실제로 변경한다.
- 기본 SGD는 현재 Gradient를 사용하고, Momentum은 이전 Gradient의 누적 정보를 반영한다.
- Adam은 Gradient와 Gradient 제곱의 이동평균을 편향 보정해 파라미터별 업데이트를 조절한다.
- 같은 Gradient와 학습률을 사용해도 Optimizer의 계산 규칙에 따라 변경량이 달라진다.
- 동일한 Gradient를 주는 수치 예제는 계산 비교이며, 실제 학습에서는 파라미터가 달라지면 다음 Gradient도 달라질 수 있다.
- Adam도 학습률의 영향을 받으며, 큰 학습률은 진동이나 발산을 일으킬 수 있다.
- Optimizer를 선택할 때는 학습 손실, 검증 성능과 학습 설정을 함께 비교해야 한다.

## 오늘의 한 문장

> SGD, Momentum, Adam은 Gradient의 현재 값과 이력을 활용하는 방식이 다르며, 학습률과 함께 조정하면서 손실의 변화와 검증 성능을 확인해야 한다.
