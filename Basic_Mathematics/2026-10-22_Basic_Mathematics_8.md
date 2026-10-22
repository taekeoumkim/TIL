# 2026-10-22 TIL | Chain Rule, 순전파와 역전파

## 1. 합성함수와 Chain Rule

앞서 미분과 편미분으로 파라미터에 대한 손실의 변화율을 계산했다. 여러 연산이 연결된 신경망에서는 **각 연산의 미분값을 연결하여 전체 기울기를 구하는 방법**이 필요하다.

### 합성함수

합성함수(Composite Function)는 한 함수의 출력을 다른 함수의 입력으로 사용하는 구조이다.

$$
z=g(x)=3x+1,\qquad f=h(z)=z^2
$$

두 함수를 연결하면 다음과 같다.

$$
f(x)=h(g(x))=(3x+1)^2
$$

$x=2$이면 중간값 $z=7$, 최종 출력 $f=49$이다. 중간 변수 $z$는 입력 $x$와 최종 출력 $f$ 사이의 계산 결과를 전달한다.

### 연쇄법칙

Chain Rule(연쇄법칙)은 연결된 함수들의 미분값을 곱해 전체 변화율을 계산하는 규칙이다.

$$
\frac{df}{dx}
=\frac{df}{dz}\frac{dz}{dx}
$$

각 단계의 미분값을 국소 미분값(Local Derivative)이라고 한다.

$$
\frac{df}{dz}=2z,\qquad
\frac{dz}{dx}=3
$$

따라서 전체 도함수는 다음과 같다.

$$
\frac{df}{dx}
=2z\times3
=6(3x+1)
=18x+6
$$

| 계산 대상 | $x=2$에서의 결과 |
| --- | ---: |
| 중간값 $z$ | 7 |
| 최종 출력 $f$ | 49 |
| $\frac{df}{dz}$ | 14 |
| $\frac{dz}{dx}$ | 3 |
| $\frac{df}{dx}$ | 42 |

함수값 49와 미분값 42는 의미가 다르다. 미분값 42는 $x=2$ 부근에서 입력을 아주 조금 변경할 때 나타나는 **순간 변화율**이다.

```python
def f(x):
    z = 3 * x + 1
    return z ** 2

def derivative(x):
    z = 3 * x + 1
    df_dz = 2 * z
    dz_dx = 3
    return df_dz * dz_dx

x = 2.0

print(f(x))           # 49.0
print(derivative(x))  # 42.0
```

### 여러 연산이 연결되는 경우

다음처럼 세 단계의 연산이 연결되어 있어도 같은 원리를 적용한다.

$$
a=2x+1,\qquad z=a^2,\qquad L=(z-5)^2
$$

$$
\frac{dL}{dx}
=\frac{dL}{dz}\frac{dz}{da}\frac{da}{dx}
=2(z-5)\times2a\times2
$$

전체 식을 하나의 다항식으로 전개하지 않아도 각 단계의 미분값으로 기울기를 구할 수 있다. 하나의 변수가 여러 경로를 통해 손실에 영향을 주면 **경로별로 전달된 기울기를 합산**한다.

## 2. 순전파와 역전파의 역할

순전파(Forward Propagation)는 입력에서 시작해 모델의 예측값을 계산하는 과정이다. 학습할 때는 예측값과 정답을 비교해 손실도 계산한다.

역전파(Backpropagation)는 최종 손실에서 시작해 Chain Rule을 적용하고, 각 파라미터에 대한 Gradient를 계산하는 과정이다.

| 구분 | 계산하는 내용 | 목적 |
| --- | --- | --- |
| 순전파 | 입력과 현재 파라미터로 중간값·예측값 계산 | 현재 모델의 출력 확인 |
| 손실 계산 | 예측값과 정답의 차이 계산 | 현재 예측의 오차 수치화 |
| 역전파 | 중간 변수와 파라미터에 대한 손실의 기울기 계산 | 파라미터가 손실에 미치는 변화율 확인 |

역전파는 출력에서 입력을 복원하는 계산이 아니다. **최종 손실에 대한 기울기를 이전 연산으로 전달하는 과정**이다.

### 회귀 모델의 순전파

다음 모델과 제곱오차를 사용한다.

$$
\hat y=wx+b,\qquad L=(y-\hat y)^2
$$

$x=2$, $y=10$, $w=3$, $b=1$이면 다음과 같다.

$$
\hat y=3\times2+1=7,\qquad
L=(10-7)^2=9
$$

### 회귀 모델의 역전파

먼저 예측값에 대한 손실의 미분값을 계산한다.

$$
\frac{\partial L}{\partial\hat y}
=2(\hat y-y)
=2(7-10)
=-6
$$

예측값을 가중치와 편향에 대해 미분하면 다음과 같다.

$$
\frac{\partial\hat y}{\partial w}=x=2,\qquad
\frac{\partial\hat y}{\partial b}=1
$$

이 값들을 Chain Rule로 연결한다.

$$
\frac{\partial L}{\partial w}
=\frac{\partial L}{\partial\hat y}
\frac{\partial\hat y}{\partial w}
=-6\times2=-12
$$

$$
\frac{\partial L}{\partial b}
=\frac{\partial L}{\partial\hat y}
\frac{\partial\hat y}{\partial b}
=-6\times1=-6
$$

| 구분 | 계산 대상 | 결과 |
| --- | --- | ---: |
| 순전파 | 예측값 $\hat y$ | 7 |
| 손실 계산 | 손실 $L$ | 9 |
| 역전파 | $\frac{\partial L}{\partial\hat y}$ | -6 |
| 역전파 | $\frac{\partial L}{\partial w}$ | -12 |
| 역전파 | $\frac{\partial L}{\partial b}$ | -6 |

손실 함수를 직접 전개해 미분하는 방법과 같은 결과를 얻는다. 역전파에서는 중간 기울기 $-6$을 가중치와 편향의 Gradient 계산에 재사용한다.

## 3. 여러 연산을 거치는 기울기 전달

두 단계의 아핀 변환을 연결한 회귀 모델을 생각해 보자. 아핀 변환은 가중합에 편향을 더하는 연산이다.

$$
a=w_1x+b_1
$$

$$
\hat y=w_2a+b_2,\qquad L=(y-\hat y)^2
$$

입력과 파라미터는 다음과 같다.

| 항목 | 값 |
| --- | ---: |
| 입력 $x$ | 2 |
| 정답 $y$ | 12 |
| 첫 번째 가중치 $w_1$ | 2 |
| 첫 번째 편향 $b_1$ | 1 |
| 두 번째 가중치 $w_2$ | 2 |
| 두 번째 편향 $b_2$ | 0 |

### 순전파

$$
a=2\times2+1=5
$$

$$
\hat y=2\times5+0=10,\qquad
L=(12-10)^2=4
$$

### 출력에서 앞쪽 연산으로 역전파

먼저 손실을 예측값에 대해 미분한다.

$$
g_{\hat y}
=\frac{\partial L}{\partial\hat y}
=2(\hat y-y)=-4
$$

두 번째 연산의 파라미터 Gradient는 다음과 같다.

$$
\frac{\partial L}{\partial w_2}=g_{\hat y}a=-20,\qquad
\frac{\partial L}{\partial b_2}=g_{\hat y}=-4
$$

이어서 중간값 $a$에 대한 기울기를 계산한다.

$$
g_a=\frac{\partial L}{\partial a}
=g_{\hat y}\frac{\partial\hat y}{\partial a}
=g_{\hat y}w_2
=-4\times2=-8
$$

이 기울기를 첫 번째 연산의 파라미터까지 전달한다.

$$
\frac{\partial L}{\partial w_1}=g_ax=-16,\qquad
\frac{\partial L}{\partial b_1}=g_a=-8
$$

| 파라미터 | Gradient 계산 | 결과 |
| --- | --- | ---: |
| $w_1$ | $g_ax$ | -16 |
| $b_1$ | $g_a$ | -8 |
| $w_2$ | $g_{\hat y}a$ | -20 |
| $b_2$ | $g_{\hat y}$ | -4 |

각 연산에서는 **뒤에서 전달된 손실의 기울기와 해당 연산의 국소 미분값을 곱한다.** 순전파에서 계산한 중간값 $a$는 역전파에서 $w_2$의 Gradient를 구할 때 사용된다.

### 코드로 확인하기

```python
x, y = 2.0, 12.0
w1, b1 = 2.0, 1.0
w2, b2 = 2.0, 0.0

# 순전파와 손실 계산
a = w1 * x + b1
y_pred = w2 * a + b2
loss = (y - y_pred) ** 2

# 출력 쪽 연산의 역전파
dL_dy_pred = 2 * (y_pred - y)
dL_dw2 = dL_dy_pred * a
dL_db2 = dL_dy_pred

# 중간값을 거쳐 앞쪽 연산으로 기울기 전달
dL_da = dL_dy_pred * w2
dL_dw1 = dL_da * x
dL_db1 = dL_da

print(a, y_pred, loss)              # 5.0 10.0 4.0
print(dL_dw1, dL_db1)               # -16.0 -8.0
print(dL_dw2, dL_db2)               # -20.0 -4.0
```

이 예제는 역전파의 계산 순서를 보여주기 위해 아핀 변환만 연결했다. 비선형 활성화 함수가 없는 아핀 변환의 연속은 하나의 아핀 변환으로 합칠 수 있으며, 다음 은닉층 예제에서는 ReLU를 추가한다.

## 4. Sigmoid와 BCE의 결합 미분

이진 분류 모델은 로짓(Logit)을 계산하고, Sigmoid로 클래스 1의 확률을 구한다.

$$
z=wx+b,\qquad
p=\mathrm{Sigmoid}(z)=\frac{1}{1+e^{-z}}
$$

한 데이터에 대한 일반적인 BCE 손실은 다음과 같다.

$$
L=-\left[y\ln p+(1-y)\ln(1-p)\right]
$$

여기서 $y\in\{0,1\}$이고, $p$는 클래스 1에 부여한 예측 확률이다.

### 확률에 대한 미분과 로짓에 대한 미분 구분하기

BCE를 예측 확률 $p$에 대해 미분하면 다음과 같다.

$$
\frac{\partial L}{\partial p}
=-\frac yp+\frac{1-y}{1-p}
$$

Sigmoid의 도함수는 다음과 같다.

$$
\frac{dp}{dz}=p(1-p)
$$

두 식을 Chain Rule로 연결하면 다음과 같이 정리된다.

$$
\frac{\partial L}{\partial z}
=\frac{\partial L}{\partial p}\frac{dp}{dz}
=\left[-\frac yp+\frac{1-y}{1-p}\right]p(1-p)
$$

$$
=-y(1-p)+(1-y)p
=p-y
$$

따라서 핵심 결과는 다음과 같다.

$$
\boxed{\frac{\partial L}{\partial z}=p-y}
$$

**$p-y$는 확률 $p$가 아니라 로짓 $z$에 대한 손실의 미분값**이다. 이 값을 사용한 뒤 Sigmoid의 도함수 $p(1-p)$를 다시 곱하면 같은 연산을 중복 적용하게 된다.

이 공식은 한 데이터에 대한 Sigmoid와 가중치 없는 BCE의 결합에서 성립한다. 배치 평균 손실을 사용하면 평균에 따른 배율이 반영되며, 클래스 가중치 등을 추가하면 미분식도 그에 맞게 달라진다.

### 가중치와 편향의 Gradient

$z=wx+b$이므로 다음과 같다.

$$
\frac{\partial L}{\partial w}=(p-y)x,\qquad
\frac{\partial L}{\partial b}=p-y
$$

$x=2$, $y=1$, $w=0.5$, $b=0$을 대입하면 다음과 같다.

| 항목 | 계산 | 결과 |
| --- | --- | ---: |
| 로짓 $z$ | $0.5\times2+0$ | 1 |
| 예측 확률 $p$ | $\mathrm{Sigmoid}(1)$ | 약 0.731 |
| BCE $L$ | $-\ln p$ | 약 0.313 |
| $\frac{\partial L}{\partial z}$ | $p-1$ | 약 -0.269 |
| $\frac{\partial L}{\partial w}$ | $(p-1)\times2$ | 약 -0.538 |
| $\frac{\partial L}{\partial b}$ | $p-1$ | 약 -0.269 |

로짓에 대한 기울기가 음수이므로 현재 지점에서 $z$를 조금 증가시키면 손실이 감소한다. 정답이 1인 상황에서 예측 확률을 높이는 방향과 일치한다.

## 5. ReLU 은닉층이 있는 신경망의 역전파

은닉층에 비선형 활성화 함수 ReLU를 적용한 간단한 신경망은 다음과 같다.

$$
a=w_1x+b_1,\qquad h=\mathrm{ReLU}(a)
$$

$$
z=w_2h+b_2,\qquad p=\mathrm{Sigmoid}(z),\qquad
L=\mathrm{BCE}(y,p)
$$

여기서 $a$는 은닉층의 활성화 전 값이고, $h$는 ReLU를 통과한 값이다.

### 순전파 결과

$x=2$, $y=1$, $w_1=1$, $b_1=0$, $w_2=0.5$, $b_2=0$으로 계산한다.

| 계산 단계 | 계산 | 결과 |
| --- | --- | ---: |
| 은닉층 가중합 $a$ | $1\times2+0$ | 2 |
| ReLU 출력 $h$ | $\max(0,2)$ | 2 |
| 출력층 로짓 $z$ | $0.5\times2+0$ | 1 |
| 예측 확률 $p$ | $\mathrm{Sigmoid}(1)$ | 약 0.731 |
| BCE 손실 $L$ | $-\ln p$ | 약 0.313 |

### 출력층의 역전파

Sigmoid와 BCE의 결합 미분을 먼저 계산한다.

$$
g_z=\frac{\partial L}{\partial z}=p-y\approx-0.269
$$

출력층의 Gradient와 은닉층 출력에 대한 기울기는 다음과 같다.

$$
\frac{\partial L}{\partial w_2}=g_zh\approx-0.538,\qquad
\frac{\partial L}{\partial b_2}=g_z\approx-0.269
$$

$$
g_h=\frac{\partial L}{\partial h}=g_zw_2\approx-0.134
$$

### ReLU를 거쳐 은닉층으로 전달하기

ReLU와 그 도함수는 다음과 같다.

$$
\mathrm{ReLU}(a)=\max(0,a)
$$

$$
\mathrm{ReLU}'(a)=
\begin{cases}
1,&a>0\\
0,&a<0
\end{cases}
$$

$a=0$에서는 미분 가능하지 않으며, 아래 예제 구현에서는 역전파에 사용할 값을 0으로 정한다.

현재 $a=2>0$이므로 ReLU의 미분값은 1이다.

$$
g_a=\frac{\partial L}{\partial a}
=g_h\mathrm{ReLU}'(a)
\approx-0.134\times1=-0.134
$$

첫 번째 가중치와 편향까지 기울기를 전달한다.

$$
\frac{\partial L}{\partial w_1}=g_ax\approx-0.269,\qquad
\frac{\partial L}{\partial b_1}=g_a\approx-0.134
$$

| 파라미터 | Gradient 공식 | 결과 |
| --- | --- | ---: |
| $w_1$ | $g_ax$ | 약 -0.269 |
| $b_1$ | $g_a$ | 약 -0.134 |
| $w_2$ | $g_zh$ | 약 -0.538 |
| $b_2$ | $g_z$ | 약 -0.269 |

계산에는 반올림하지 않은 값을 사용하고, 표에서는 소수 셋째 자리까지 표시했다.

ReLU 입력이 음수이면 국소 미분값이 0이므로 **해당 경로를 통해 앞쪽으로 전달되는 기울기도 0**이 된다. 활성화 함수의 미분값이 기울기 전달에 직접 영향을 준다는 점을 알 수 있다.

## 6. NumPy로 은닉층 역전파 구현하기

아래 코드는 순전파에서 중간값을 계산한 뒤, 그 값과 국소 미분값을 이용해 모든 파라미터의 Gradient를 구한다.

```python
import numpy as np

def sigmoid(z):
    return 1 / (1 + np.exp(-z))

x, y = 2.0, 1.0
w1, b1 = 1.0, 0.0
w2, b2 = 0.5, 0.0

# 순전파
a = w1 * x + b1
h = np.maximum(0.0, a)
z = w2 * h + b2
p = sigmoid(z)

# 로짓으로 BCE 계산
loss = np.logaddexp(0, z) - y * z

# 출력층 역전파
dL_dz = p - y
dL_dw2 = dL_dz * h
dL_db2 = dL_dz

# 은닉층으로 기울기 전달
dL_dh = dL_dz * w2
relu_grad = 1.0 if a > 0 else 0.0
dL_da = dL_dh * relu_grad

# 첫 번째 층의 파라미터 Gradient
dL_dw1 = dL_da * x
dL_db1 = dL_da

print(round(p, 3))                     # 0.731
print(round(float(loss), 3))           # 0.313
print(round(dL_dw1, 3), round(dL_db1, 3))  # -0.269 -0.134
print(round(dL_dw2, 3), round(dL_db2, 3))  # -0.538 -0.269
```

### 로짓에서 BCE를 계산하는 이유

Sigmoid와 BCE를 결합하면 손실을 로짓으로 표현할 수 있다.

$$
L=\ln(1+e^z)-yz
$$

`np.logaddexp(0, z)`는 $\ln(e^0+e^z)$를 수치적으로 안정적으로 계산한다. 예측 확률이 0이나 1에 매우 가까울 때 확률의 로그를 직접 계산하며 생길 수 있는 문제를 줄이는 데 활용한다.

위 코드의 Sigmoid 구현은 예제의 작은 로짓을 계산하기 위한 형태이다. 실제 학습에서는 프레임워크가 제공하는 안정적인 로짓 기반 손실 구현을 활용할 수 있다.

## 7. 역전파, 자동 미분과 파라미터 업데이트

### 역전파와 업데이트 구분하기

역전파는 Gradient를 계산하는 과정이다. **계산된 Gradient를 이용해 파라미터를 변경하는 작업은 Optimizer가 수행한다.**

| 학습 단계 | 역할 | 결과 |
| --- | --- | --- |
| 순전파 | 현재 파라미터로 중간값과 예측값 계산 | 모델 출력 |
| 손실 계산 | 예측값과 실제 정답 비교 | 손실값 |
| 역전파 | Chain Rule로 기울기 계산 | 각 파라미터의 Gradient |
| 파라미터 업데이트 | 최적화 규칙에 따라 파라미터 변경 | 새로운 가중치와 편향 |

기본적인 경사하강법의 업데이트는 다음과 같다.

$$
\boldsymbol\theta_{\mathrm{new}}
=\boldsymbol\theta_{\mathrm{old}}
-\eta\nabla_{\boldsymbol\theta}L(\boldsymbol\theta_{\mathrm{old}})
$$

$\eta$는 학습률이다. 위 코드에서는 Gradient 계산까지 수행했으며, 파라미터 업데이트를 하려면 이와 같은 최적화 단계를 추가해야 한다.

앞쪽 레이어의 Gradient까지 계산을 마치기 전에 뒤쪽 가중치를 변경하면, 순전파 때와 다른 파라미터로 기울기를 계산하게 될 수 있다. 따라서 한 번의 학습 단계에서는 **같은 순전파에 대응하는 Gradient를 계산한 뒤 파라미터를 업데이트**한다.

### 자동 미분의 역할

자동 미분(Automatic Differentiation)은 연산별 미분 규칙을 연결해 Gradient를 계산하는 기능이다. 딥러닝 프레임워크는 순전파에서 수행한 연산 정보를 바탕으로 역전파를 실행한다.

자동 미분을 사용해도 수학적 원리는 같다.

- 각 연산의 국소 미분값을 사용한다.
- 뒤쪽에서 전달된 기울기와 국소 미분값을 연결한다.
- 여러 경로의 기울기는 필요한 지점에서 합산한다.
- 계산한 Gradient를 Optimizer가 파라미터 업데이트에 사용한다.

Chain Rule은 미분 규칙이고, 역전파는 그 규칙을 이용해 손실의 기울기를 효율적으로 계산하는 방법이다. 자동 미분은 이러한 계산을 자동화한다.

## 핵심 정리

- 합성함수는 한 함수의 출력을 다음 함수의 입력으로 사용하는 구조이다.
- Chain Rule은 각 단계의 국소 미분값을 연결해 전체 변화율을 구한다.
- 순전파는 중간값과 예측값을 계산하고, 역전파는 손실에 대한 기울기를 앞쪽 연산으로 전달한다.
- 역전파에는 순전파에서 계산한 중간값과 해당 시점의 파라미터가 사용된다.
- Sigmoid와 일반적인 BCE를 결합하면 한 데이터의 로짓에 대한 손실 미분은 $p-y$로 정리된다.
- ReLU의 미분값은 입력이 양수일 때 1, 음수일 때 0이며 해당 경로의 기울기 전달에 영향을 준다.
- 역전파는 Gradient를 계산하고, Optimizer는 그 Gradient로 파라미터를 변경한다.

## 오늘의 한 문장

> 역전파는 Chain Rule을 이용해 최종 손실의 기울기를 앞쪽 연산으로 전달하고, 각 가중치와 편향이 손실에 미치는 변화율을 효율적으로 계산하는 과정이다.