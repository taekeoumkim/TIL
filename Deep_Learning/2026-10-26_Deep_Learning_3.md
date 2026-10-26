# 2026-10-26 TIL | PyTorch 자동 미분, 기울기 소실·폭주와 Clipping

## 1. 역전파와 자동 미분의 역할

역전파(Backpropagation)는 최종 손실에서 시작해 Chain Rule을 적용하고, 각 가중치와 편향에 대한 Gradient를 계산하는 과정이다.

$$
\frac{\partial L}{\partial w}
=\frac{\partial L}{\partial z}\frac{\partial z}{\partial w}
$$

간단한 회귀 모델에서는 미분식을 직접 작성할 수 있지만, 실제 DNN에는 많은 파라미터와 연산이 존재한다. PyTorch의 자동 미분(Autograd)은 **추적한 연산의 관계와 각 연산의 미분 규칙을 이용해 Gradient를 계산**한다.

### 계산 그래프와 역전파

Gradient 추적이 활성화된 상태에서 학습 파라미터와 연결된 연산을 수행하면, PyTorch는 연산 관계를 계산 그래프(Computation Graph)로 기록한다.

손실을 구한 후 `loss.backward()`를 호출하면 그래프를 따라 역전파가 수행되고, 학습 파라미터의 기울기가 `.grad`에 저장된다.

| 단계 | 주요 역할 | 대표 코드 |
| --- | --- | --- |
| 순전파 | 현재 파라미터로 예측값 계산 | `logits = model(X)` |
| 손실 계산 | 예측값과 실제 정답 비교 | `loss = loss_fn(logits, y)` |
| 역전파 | 파라미터별 Gradient 계산 | `loss.backward()` |
| 파라미터 업데이트 | 가중치와 편향 변경 | `optimizer.step()` |

**역전파를 실행하는 것만으로 가중치와 편향이 업데이트되지는 않는다.** 실제 변경은 Optimizer의 업데이트 단계에서 이루어진다.

## 2. PyTorch에서 Gradient 확인하기

입력 3개, 은닉층 4개·2개, 출력 1개인 MLP를 생각해 보자. 출력층에서는 로짓을 반환하고, 이진 분류 손실인 `BCEWithLogitsLoss`를 사용한다.

이 손실 함수는 Sigmoid와 BCE 계산을 결합하므로, 모델 출력에 Sigmoid를 미리 적용하지 않는다.

### DNN의 역전파 예제

```python
import torch
import torch.nn as nn

torch.manual_seed(2)

X = torch.tensor([
    [0.5, 0.3, 0.8],
    [0.2, 0.0, 0.3],
    [0.8, 0.4, 1.0],
    [0.3, 0.1, 0.5],
], dtype=torch.float32)

y = torch.tensor([
    [1.0],
    [0.0],
    [1.0],
    [0.0],
], dtype=torch.float32)

model = nn.Sequential(
    nn.Linear(3, 4),
    nn.ReLU(),
    nn.Linear(4, 2),
    nn.ReLU(),
    nn.Linear(2, 1),
)
loss_fn = nn.BCEWithLogitsLoss()

model.train()
model.zero_grad()
weight_before = model[0].weight.detach().clone()

logits = model(X)
loss = loss_fn(logits, y)
loss.backward()

print(tuple(X.shape))       # (4, 3)
print(tuple(logits.shape))  # (4, 1)
print("손실:", loss.item())

for name, parameter in model.named_parameters():
    if parameter.grad is not None:
        print(
            name,
            tuple(parameter.shape),
            tuple(parameter.grad.shape),
        )

weight_changed = not torch.equal(
    weight_before, model[0].weight.detach()
)
print("역전파 후 가중치 변경:", weight_changed)  # False
```

가중치를 비교하기 위한 `detach().clone()`은 기울기 추적에서 분리한 값의 복사본을 만든다. 역전파 후에도 가중치가 그대로인 것은 `optimizer.step()`을 실행하지 않았기 때문이다.

### 파라미터와 Gradient의 Shape

| 파라미터 | 저장 위치 | 파라미터 Shape | Gradient Shape |
| --- | --- | --- | --- |
| 첫 은닉층 가중치 | `model[0].weight` | $(4,3)$ | $(4,3)$ |
| 첫 은닉층 편향 | `model[0].bias` | $(4,)$ | $(4,)$ |
| 두 번째 은닉층 가중치 | `model[2].weight` | $(2,4)$ | $(2,4)$ |
| 두 번째 은닉층 편향 | `model[2].bias` | $(2,)$ | $(2,)$ |
| 출력층 가중치 | `model[4].weight` | $(1,2)$ | $(1,2)$ |
| 출력층 편향 | `model[4].bias` | $(1,)$ | $(1,)$ |

파라미터의 각 원소에 대응하는 기울기가 필요하므로 **Gradient는 해당 파라미터와 같은 Shape**를 가진다. 이 모델은 가중치 22개와 편향 7개, 총 29개의 학습 가능한 스칼라 값을 사용한다.

## 3. Gradient 누적과 추적 상태

### 이전 Gradient 초기화하기

PyTorch는 기본적으로 역전파에서 계산한 Gradient를 기존 `.grad`에 누적한다. 일반적인 학습 반복에서는 이전 단계의 기울기를 초기화한 뒤 새로운 기울기를 계산한다.

| 코드 | 역할 |
| --- | --- |
| `model.zero_grad()` | 모델 파라미터의 기존 Gradient 초기화 |
| `optimizer.zero_grad()` | Optimizer가 관리하는 파라미터의 기존 Gradient 초기화 |
| `loss.backward()` | 이번 손실의 Gradient 계산·누적 |

초기화한 Gradient를 항상 0으로 채운 텐서로 보관하는 것은 아니며, 설정에 따라 `None`이 될 수 있다.

`parameter.grad is None`은 기울기가 0인 텐서와 다르다. 초기화 직후이거나 해당 파라미터가 이번 손실의 계산에 사용되지 않은 경우 등에도 `None`일 수 있으므로, Norm을 확인하기 전에 구분해야 한다.

### 학습 모드와 Gradient 추적 구분하기

`model.train()`은 모델을 학습 모드로 전환한다. 이것만으로 모든 구간의 Gradient 추적이 강제로 켜지는 것은 아니다.

`torch.no_grad()`는 Gradient 계산을 위한 연산 기록을 비활성화한다. 따라서 일반적인 역전파 학습의 순전파·손실 계산 구간 전체를 `no_grad()`로 감싸면 안 된다.

중간 연산 결과인 텐서의 `.grad`는 기본적으로 보관되지 않을 수 있다. 중간값의 기울기를 직접 살펴보려면 아래 예제처럼 `retain_grad()`를 사용할 수 있다.

## 4. 기울기 소실과 기울기 폭주

| 구분 | 기울기 소실(Vanishing Gradient) | 기울기 폭주(Exploding Gradient) |
| --- | --- | --- |
| 현상 | 역전파 중 기울기가 지나치게 작아짐 | 역전파 중 기울기가 지나치게 커짐 |
| 학습에 미치는 영향 | 앞쪽 레이어의 업데이트가 거의 없거나 학습 정체 | 과도한 업데이트와 학습 불안정 |
| 관련 요인 | 작은 국소 미분값의 반복적인 곱 등 | 큰 국소 미분값의 반복적인 곱 등 |
| 관찰 가능한 결과 | 학습이 느려지거나 손실 개선이 어려움 | 손실 급증·진동·발산, 심하면 `inf`·`nan` |

### 반복되는 미분값의 영향

역전파에서는 전달받은 기울기에 현재 연산의 미분값이 곱해진다. 단순하게 0.5 또는 2가 반복해서 곱해지는 경우를 비교하면 다음과 같다.

| 곱셈 횟수 | $0.5$를 반복해서 곱한 값 | $2$를 반복해서 곱한 값 |
| ---: | ---: | ---: |
| 1 | 0.5 | 2 |
| 2 | 0.25 | 4 |
| 4 | 0.0625 | 16 |
| 8 | 0.00390625 | 256 |

실제 DNN에서는 가중치와 활성화 함수 등 여러 요소가 기울기 전달에 함께 영향을 준다. 위 표는 반복되는 미분값이 기울기의 크기를 바꾸는 원리를 단순화한 것이다.

### 파라미터 업데이트와 연결하기

기본 경사하강법의 변화량은 다음과 같다.

$$
\Delta w=-\alpha g
$$

학습률 $\alpha=0.01$일 때 Gradient가 $10^{-6}$이면 변화량은 $-10^{-8}$이다. 반대로 Gradient가 $100000$이면 변화량은 $-1000$으로 커진다.

따라서 학습률이 같아도 Gradient의 크기에 따라 실제 업데이트 크기가 달라진다.

### PyTorch로 반복 곱셈의 기울기 확인하기

입력 $x=1$에 상수 $c$를 8번 곱하고, 목표값 1에 대한 제곱오차를 계산한다.

$$
h_8=c^8x,\qquad L=(h_8-1)^2
$$

$$
\frac{\partial L}{\partial h_8}=2(h_8-1),\qquad
\frac{\partial L}{\partial x}=2(h_8-1)c^8
$$

```python
import torch

def inspect_gradient(scale, depth=8):
    x = torch.tensor(1.0, requires_grad=True)
    values = [x]
    h = x

    for _ in range(depth):
        h = h * scale
        h.retain_grad()
        values.append(h)

    loss = (h - 1.0) ** 2
    loss.backward()

    print("배율:", scale)
    print("최종 출력:", f"{h.item():.6f}")
    print("손실:", f"{loss.item():.6f}")
    print("마지막 출력 Gradient:", f"{h.grad.item():.6f}")
    print("첫 단계 Gradient:", f"{values[1].grad.item():.6f}")
    print("입력 Gradient:", f"{x.grad.item():.6f}")

inspect_gradient(0.5)
inspect_gradient(2.0)
```

| 항목 | $c=0.5$ | $c=2.0$ |
| --- | ---: | ---: |
| 최종 출력 | 0.003906 | 256.000000 |
| 손실 | 0.992203 | 65025.000000 |
| 마지막 출력의 Gradient | -1.992188 | 510.000000 |
| 첫 단계 출력의 Gradient | -0.015564 | 65280.000000 |
| 입력의 Gradient | -0.007782 | 130560.000000 |

이 예제는 Gradient 전달을 보여주는 단순한 연산이며 실제 DNN 학습 예제는 아니다. 두 경우의 최종 출력과 손실 규모도 다르므로, 입력 Gradient의 차이를 반복 곱셈의 효과만으로 해석하지 않아야 한다.

## 5. 활성화 함수와 기울기 전달

| 활성화 함수 | 미분값의 특징 | 학습에서 고려할 점 |
| --- | --- | --- |
| Sigmoid | 최대 0.25, 포화 영역에서 0에 가까움 | 깊은 은닉층에서 기울기 소실 가능 |
| Tanh | 최대 1, 포화 영역에서 0에 가까움 | 포화 영역에서 기울기 소실 가능 |
| ReLU | 양수에서 1, 음수에서 0 | 양수 영역의 포화 문제 완화, Dead ReLU 가능 |

Sigmoid와 Tanh는 입력값의 절댓값이 큰 포화 영역에서 미분값이 작아진다. ReLU는 양수 영역에서 미분값이 1이지만, 음수 영역에서는 기울기가 0이다.

특정 뉴런이 계속 음수 가중합을 계산하면 입력 쪽 파라미터에 기울기가 전달되지 않아 학습이 어려워질 수 있다. 이를 Dead ReLU 문제와 연결할 수 있다.

**ReLU를 사용한다고 모든 기울기 문제가 해결되는 것은 아니다.** 활성화 함수뿐 아니라 가중치, 초기화, 네트워크 구조도 함께 고려해야 한다.

## 6. Gradient Norm으로 학습 상태 확인하기

Gradient Norm은 여러 기울기 값을 하나의 크기로 요약하는 방법이다.

기울기를 하나의 벡터 $\mathbf g$로 펼쳤을 때 L2 Norm은 다음과 같다.

$$
\|\mathbf g\|_2=\sqrt{\sum_i g_i^2}
$$

예를 들어 $[3,4]$의 L2 Norm은 $\sqrt{3^2+4^2}=5$이다.

### 파라미터별 확인

`parameter.grad.norm()`으로 해당 파라미터의 기울기 크기를 확인할 수 있다.

```python
# 앞의 DNN 역전파 예제에 이어서 실행한다.
for name, parameter in model.named_parameters():
    if parameter.grad is None:
        print(name, "Gradient 없음")
    else:
        print(name, f"{parameter.grad.norm().item():.6f}")
```

여러 파라미터 텐서의 Gradient를 합쳐 본 전체 L2 Norm은 다음과 같다.

$$
\|\mathbf g_{\mathrm{total}}\|_2
=\sqrt{\sum_j\|\mathbf g_j\|_2^2}
$$

파라미터별 Norm과 전체 Norm은 구분해야 한다.

### 숫자 하나보다 추세 살펴보기

| 관찰한 현상 | 함께 확인할 내용 |
| --- | --- |
| 앞쪽 레이어의 Gradient가 지속적으로 매우 작음 | 기울기 소실 가능성, 손실 개선 여부 |
| 특정 레이어의 Gradient가 갑자기 커짐 | 기울기 폭주 가능성, 입력·손실 규모 |
| 손실이 급격하게 증가 | 학습률, Gradient, 수치 계산 상태 |
| 손실 개선이 거의 없음 | Gradient, 학습률, 모델 구조 등 |

Gradient가 작다고 항상 기울기 소실은 아니다. 최솟값 부근에서는 작은 Gradient가 자연스러울 수 있다.

Norm은 파라미터 개수와 손실 설정 등에 따라서도 달라진다. **여러 학습 단계에서의 레이어별 변화와 손실 추세를 함께 확인해야 한다.**

## 7. Gradient Clipping의 원리와 적용 위치

Gradient Clipping은 계산된 기울기의 크기가 기준을 초과할 때 제한하는 방법이다. 큰 Gradient로 인한 과도한 업데이트를 완화하는 데 사용한다.

### L2 Norm 기준 Clipping

전체 Gradient의 Norm이 기준 $M$보다 클 때, 모든 Gradient를 같은 비율로 줄인다. 0이 아닌 기울기에 대한 이상적인 계산은 다음과 같다.

$$
\mathbf g_{\mathrm{clip}}
=\mathbf g\cdot
\min\left(1,\frac{M}{\|\mathbf g\|_2}\right)
$$

예를 들어 Gradient가 $[3,4]$이고 $M=1$이면 다음과 같다.

$$
\mathbf g_{\mathrm{clip}}
=[3,4]\times\frac15=[0.6,0.8]
$$

기준보다 작은 Gradient는 그대로 둔다. **이 방식은 각 원소를 개별적으로 같은 최댓값에 맞추는 것이 아니라, 전체 Norm을 기준으로 공통 비율을 적용한다.**

### PyTorch에서 적용하기

다음 코드는 앞의 DNN 역전파를 마친 후 실행한다.

```python
import torch
import torch.nn as nn

parameters = list(model.parameters())

def total_gradient_norm(parameters):
    norms = [
        parameter.grad.norm(2)
        for parameter in parameters
        if parameter.grad is not None
    ]
    return torch.linalg.vector_norm(torch.stack(norms))

weight_before_clip = model[0].weight.detach().clone()
norm_before = total_gradient_norm(parameters).item()

# Clipping 동작을 확인하기 위한 예제 기준값
max_norm = 0.03
returned_norm = nn.utils.clip_grad_norm_(
    parameters, max_norm=max_norm
)
norm_after = total_gradient_norm(parameters).item()

print("Clipping 전:", norm_before)
print("반환된 Norm:", returned_norm.item())
print("Clipping 후:", norm_after)

weight_changed = not torch.equal(
    weight_before_clip, model[0].weight.detach()
)
print("Clipping 후 가중치 변경:", weight_changed)  # False
```

`clip_grad_norm_()`은 `.grad`에 저장된 기울기를 조정하며, 반환값은 **Clipping 적용 전의 전체 Norm**이다. 적용 후의 Norm은 다시 계산해 확인한다.

`max_norm=0.03`은 동작을 설명하기 위한 값이며, 모든 모델에 적절한 기준이라는 뜻은 아니다. 작은 기준을 의도적으로 정해 Clipping이 발생했다는 사실만으로 기울기 폭주가 있었다고 판단할 수는 없다.

### Clipping과 업데이트 구분하기

Clipping은 기울기를 조정하고 가중치 자체는 변경하지 않는다. 따라서 다음 순서로 적용한다.

| 순서 | 단계 | 변경되는 대상 |
| ---: | --- | --- |
| 1 | Gradient 초기화 | 이전 기울기 |
| 2 | 순전파·손실 계산 | 중간 계산값과 손실 |
| 3 | 역전파 | 파라미터의 `.grad` |
| 4 | 필요 시 Gradient Clipping | 계산된 Gradient |
| 5 | Optimizer 업데이트 | 가중치와 편향 |

**Clipping은 역전파 후, Optimizer 업데이트 전에 수행한다.** 큰 기울기를 제한하는 방법이므로 기울기 소실을 직접 해결하지는 않는다.

## 8. 기울기 문제에 대응하는 기본적인 방법

| 방법 | 주요 목적 | 함께 기억할 점 |
| --- | --- | --- |
| 활성화 함수 선택 | 기울기 전달이 지나치게 약해지는 문제 완화 | ReLU도 음수 영역에서 미분값이 0 |
| 가중치 초기화 조정 | 초기 신호와 기울기 전달 안정화 | 모델 구조와 활성화 함수에 맞춰 고려 |
| 학습률 조정 | 실제 업데이트 크기 조절 | 역전파에서 계산되는 Gradient 자체의 문제와 구분 |
| Gradient Clipping | 큰 Gradient의 영향 제한 | 기울기 폭주 완화에 활용 |

대표적인 초기화 방법으로는 Tanh 등과 함께 고려하는 Xavier 초기화, ReLU 계열과 함께 고려하는 He 초기화가 있다. 초기화의 세부 수식보다 **초기 파라미터의 크기가 학습 안정성에 영향을 준다**는 점을 먼저 이해한다.

학습률을 줄이면 큰 Gradient로 인한 업데이트 크기를 줄일 수 있다. 하지만 이미 역전파에서 지나치게 작게 계산된 Gradient의 원인을 직접 해결하는 것은 아니다.

### 기본 학습 단계와 연결하기

아래 코드는 앞에서 정의한 모델과 데이터로 한 번의 업데이트를 수행하는 흐름이다.

```python
# 앞의 DNN 예제에서 model, X, y, loss_fn을 정의한 뒤 실행한다.
optimizer = torch.optim.SGD(model.parameters(), lr=0.01)

optimizer.zero_grad()
logits = model(X)
loss = loss_fn(logits, y)
loss.backward()

# 필요한 경우에만 사용한다. 기준값은 예제용이다.
nn.utils.clip_grad_norm_(model.parameters(), max_norm=0.03)

optimizer.step()
```

PyTorch 예제는 PyTorch가 설치된 환경에서 실행한다. 손실과 레이어별 Gradient Norm을 여러 단계에 걸쳐 확인하면서 학습률, 활성화 함수, 초기화와 모델 구조를 함께 점검해야 한다.

## 핵심 정리

- Autograd는 계산 그래프와 미분 규칙을 이용해 Gradient를 계산한다.
- `loss.backward()`는 기울기를 `.grad`에 계산·누적하며, 파라미터 업데이트는 `optimizer.step()`이 수행한다.
- 일반적인 반복 학습에서는 이전 Gradient를 초기화하고, 역전파에 필요한 연산 추적을 유지해야 한다.
- 작은 미분값이나 큰 미분값이 반복되면 기울기 소실 또는 폭주가 발생할 수 있다.
- Gradient Norm의 숫자 하나만으로 문제를 단정하지 않고 손실과 레이어별 변화 추세를 함께 살펴야 한다.
- Gradient Clipping은 전체 기울기의 크기를 제한하며, 역전파와 파라미터 업데이트 사이에 적용한다.
- 활성화 함수, 초기화, 학습률과 Clipping은 각각 역할이 다르므로 원인에 맞게 고려해야 한다.

## 오늘의 한 문장

> 역전파로 기울기를 계산하는 것에서 더 나아가, 기울기가 각 레이어에 적절하게 전달되는지 확인하고 업데이트가 안정적으로 이루어지도록 관리해야 한다.