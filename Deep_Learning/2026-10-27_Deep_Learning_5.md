# 2026-10-27 TIL | CNN의 합성곱, Pooling과 이미지 분류 구조

## 1. CNN의 기본 특징

CNN(Convolutional Neural Network)은 이미지처럼 **위치와 주변 값의 관계가 중요한 데이터에서 공간적 특징을 학습하는 신경망**이다.

MLP에서도 28×28 흑백 이미지를 784개 값으로 펼쳐 분류할 수 있다. 하지만 일반적인 Dense 레이어는 이웃 픽셀의 공간적 관계를 직접 활용하도록 설계되어 있지는 않다.

CNN은 다음 두 가지 구조적 특징을 이용한다.

| 특징 | 의미 |
| --- | --- |
| 국소 영역 활용 | 작은 영역의 픽셀들을 함께 계산 |
| 가중치 공유(Weight Sharing) | 동일한 Filter를 입력의 여러 위치에 반복 적용 |

같은 패턴이 이미지의 여러 위치에 나타날 수 있다는 특성을 활용하며, 위치마다 서로 다른 가중치를 모두 학습할 필요가 없다.

### 주요 용어

| 용어 | 의미 |
| --- | --- |
| Filter·Kernel | 입력의 일부 영역에 적용하는 작은 가중치 배열 |
| Convolution | Filter를 이동시키며 각 위치에서 값을 계산하는 연산 |
| Feature Map | Convolution의 결과를 위치별로 배열한 특징 맵 |
| Stride | Filter가 한 번 이동하는 간격 |
| Padding | 입력 가장자리에 값을 추가하는 처리 |

**Filter는 계산에 사용하는 가중치이고, Feature Map은 그 계산으로 얻은 결과**이다.

## 2. Convolution과 Feature Map 계산

Convolution은 입력의 작은 영역과 Filter를 같은 위치끼리 곱한 뒤 합산하는 계산을 반복한다.

편향을 생략한 단일 채널 예제에서는 다음과 같이 표현할 수 있다.

$$
Y_{i,j}
=\sum_{u=0}^{K_h-1}\sum_{v=0}^{K_w-1}
X_{i+u,j+v}K_{u,v}
$$

이 식은 Stride 1, Padding 0일 때의 계산이다. 실제 CNN 레이어에서는 일반적으로 Filter별 편향도 더한다.

### 4×4 입력에 3×3 Filter 적용하기

입력 이미지와 Filter를 다음과 같이 설정한다.

$$
X=
\begin{bmatrix}
1&0&1&0\\
1&1&0&0\\
0&1&1&0\\
0&0&1&1
\end{bmatrix},
\qquad
K=
\begin{bmatrix}
1&0&0\\
0&1&0\\
0&0&1
\end{bmatrix}
$$

왼쪽 위 3×3 영역에서는 Filter의 대각선 위치와 대응하는 값이 각각 1이므로 출력은 3이다. 오른쪽으로 한 칸 이동하면 대응하는 세 값이 모두 0이므로 출력은 0이다.

전체 위치를 계산한 Feature Map은 다음과 같다.

$$
Y=
\begin{bmatrix}
3&0\\
3&3
\end{bmatrix}
$$

입력은 4×4이고 출력은 2×2이다. Padding 없이 Filter가 입력을 벗어나지 않는 위치에서만 계산했기 때문이다.

딥러닝에서 사용하는 Convolution은 일반적으로 Filter를 뒤집지 않는 Cross-Correlation 방식이다. 위 계산과 아래 NumPy 예제도 같은 방식을 사용한다.

### 입력 채널과 Filter 개수

일반적인 Convolution에서 하나의 Filter는 입력 채널 전체를 함께 고려해 출력 채널 하나를 계산한다.

예를 들어 RGB 이미지의 입력 채널은 3개이다. 여기에 적용하는 3×3 Filter 하나는 $3\times3\times3=27$개의 가중치를 가지며, 채널별 계산을 합산하고 편향을 더해 특징 맵 하나를 만든다.

| 설정 | 의미 |
| --- | --- |
| 입력 채널 수 | Filter가 함께 처리하는 채널 수 |
| Filter 개수 | 출력 Feature Map의 채널 수 |
| RGB 입력에 Filter 8개 | 입력 채널 3개를 이용해 출력 채널 8개 생성 |

실제 CNN에서는 Filter의 값도 학습 가능한 파라미터이다. 손실을 줄이도록 가중치를 업데이트하면서 데이터에 유용한 특징을 학습한다.

## 3. Stride, Padding과 출력 크기

### Stride

Stride는 Filter의 이동 간격이다. Stride가 커지면 계산하는 위치가 줄어들어 일반적으로 출력의 공간적 크기가 작아진다.

다만 일부 위치를 건너뛰므로 공간적 정보를 충분히 반영하지 못할 수 있다.

### Padding

Padding은 입력 가장자리에 값을 추가하는 방법이다. Zero Padding은 가장자리에 0을 추가한다.

입력의 한 변이 $N$이고 양쪽에 각각 $P$만큼 Padding을 적용하면 해당 변의 길이는 $N+2P$가 된다.

Padding은 가장자리에서의 Filter 계산 범위를 확장하고 출력 크기를 조절한다. 원래 없던 이미지 정보를 복원하는 연산은 아니다.

### 출력 크기 공식

Filter 간격을 별도로 늘리지 않는 일반적인 경우(dilation=1), 출력의 한 변은 다음과 같다.

$$
O=\left\lfloor\frac{N+2P-K}{S}\right\rfloor+1
$$

| 기호 | 의미 |
| --- | --- |
| $N$ | 입력 크기 |
| $K$ | Filter 크기 |
| $P$ | 한쪽 Padding 크기 |
| $S$ | Stride |
| $\lfloor\cdot\rfloor$ | 내림 |

직사각형 입력에서는 높이와 너비에 대해 각각 계산한다.

앞의 4×4 입력과 3×3 Filter를 기준으로 비교하면 다음과 같다.

| Stride | Padding | 출력 크기 |
| ---: | ---: | --- |
| 1 | 0 | 2×2 |
| 2 | 0 | 1×1 |
| 1 | 1 | 4×4 |

Stride 1, Padding 1에서는 다음과 같이 계산된다.

$$
O=\left\lfloor\frac{4+2\times1-3}{1}\right\rfloor+1=4
$$

## 4. NumPy로 Convolution 구현하기

다음 코드는 단일 채널 2차원 입력에 대해, 편향 없이 Filter를 적용하는 학습용 예제이다.

```python
import numpy as np

image = np.array([
    [1, 0, 1, 0],
    [1, 1, 0, 0],
    [0, 1, 1, 0],
    [0, 0, 1, 1],
])

kernel = np.array([
    [1, 0, 0],
    [0, 1, 0],
    [0, 0, 1],
])

def conv2d(image, kernel, stride=1, padding=0):
    padded = np.pad(
        image,
        ((padding, padding), (padding, padding)),
        mode="constant",
    )
    kernel_h, kernel_w = kernel.shape
    output_h = (padded.shape[0] - kernel_h) // stride + 1
    output_w = (padded.shape[1] - kernel_w) // stride + 1
    output = np.zeros((output_h, output_w))

    for i in range(output_h):
        for j in range(output_w):
            start_h = i * stride
            start_w = j * stride
            region = padded[
                start_h:start_h + kernel_h,
                start_w:start_w + kernel_w,
            ]
            output[i, j] = np.sum(region * kernel)

    return output

feature_map = conv2d(image, kernel)

print(feature_map)
# [[3. 0.]
#  [3. 3.]]

print(feature_map.shape)                         # (2, 2)
print(conv2d(image, kernel, stride=2).shape)      # (1, 1)
print(conv2d(image, kernel, padding=1).shape)     # (4, 4)
```

각 위치에서 사용하는 Filter의 가중치는 같고, 입력에서 선택하는 영역만 달라진다. 이러한 반복이 국소적인 계산과 가중치 공유를 구현한다.

## 5. Pooling으로 Feature Map 요약하기

Pooling은 Feature Map의 일부 영역을 대표값으로 요약하여 공간적 크기를 줄이는 연산이다.

| 연산 | 주요 역할 |
| --- | --- |
| Convolution | 학습 가능한 Filter로 특징 계산 |
| ReLU | 비선형성 추가 |
| Pooling | 영역의 값을 요약해 공간적 크기 감소 |

### Max Pooling과 Average Pooling

| 구분 | Max Pooling | Average Pooling |
| --- | --- | --- |
| 계산 방식 | 영역의 최댓값 선택 | 영역의 평균값 계산 |
| 남기는 정보 | 강하게 나타난 반응 | 영역 전체의 평균적인 수준 |
| 학습 가능한 가중치·편향 | 일반적으로 없음 | 일반적으로 없음 |

예를 들어 $[1,3,4,6]$에서 Max Pooling은 6, Average Pooling은 3.5이다.

다음 4×4 Feature Map에 2×2 영역, Stride 2를 적용한다.

$$
A=
\begin{bmatrix}
1&3&2&0\\
4&6&5&1\\
7&2&8&3\\
0&1&4&9
\end{bmatrix}
$$

계산 결과는 다음과 같다.

$$
\mathrm{MaxPool}(A)=
\begin{bmatrix}
6&5\\
7&9
\end{bmatrix},
\qquad
\mathrm{AvgPool}(A)=
\begin{bmatrix}
3.5&2.0\\
2.5&6.0
\end{bmatrix}
$$

```python
import numpy as np

pool_input = np.array([
    [1, 3, 2, 0],
    [4, 6, 5, 1],
    [7, 2, 8, 3],
    [0, 1, 4, 9],
], dtype=float)

def pooling2d(image, pool_size=2, stride=2, mode="max"):
    if mode not in ("max", "avg"):
        raise ValueError("mode는 max 또는 avg여야 합니다.")

    output_h = (image.shape[0] - pool_size) // stride + 1
    output_w = (image.shape[1] - pool_size) // stride + 1
    output = np.zeros((output_h, output_w))

    for i in range(output_h):
        for j in range(output_w):
            region = image[
                i * stride:i * stride + pool_size,
                j * stride:j * stride + pool_size,
            ]
            output[i, j] = (
                np.max(region) if mode == "max"
                else np.mean(region)
            )

    return output

print(pooling2d(pool_input, mode="max"))
# [[6. 5.]
#  [7. 9.]]

print(pooling2d(pool_input, mode="avg"))
# [[3.5 2. ]
#  [2.5 6. ]]
```

### Pooling과 Shape

일반적인 2차원 Pooling은 각 채널에 독립적으로 적용한다. Batch Size와 채널 수는 유지되고, 높이와 너비가 달라진다.

$$
(4,8,28,28)\longrightarrow(4,8,14,14)
$$

이는 2×2 영역, Stride 2, Padding 0을 적용한 예제이다. PyTorch에서는 `nn.MaxPool2d`와 `nn.AvgPool2d` 등을 사용할 수 있다.

### 정보 요약과 정보 손실

Pooling은 이후 레이어의 계산량을 줄이고, 작은 위치 변화에 대한 민감도를 줄이는 데 도움이 될 수 있다. 한편 여러 값을 하나로 요약하기 때문에 원래 값과 정확한 위치 정보는 일부 사라진다.

같은 Pooling 영역 안에서 강한 반응의 위치가 달라져도 최댓값이 같을 수 있지만, 영역의 경계를 넘어 이동할 때까지 같은 결과가 보장되는 것은 아니다.

**Pooling은 모든 CNN에 필수적인 구성 요소는 아니다.** Stride가 큰 Convolution으로 공간적 크기를 줄이는 구조도 사용할 수 있다.

## 6. CNN 이미지 분류 모델의 Shape 확인하기

28×28 흑백 이미지 4개를 0부터 9까지의 10개 클래스로 분류하는 모델을 생각해 보자.

PyTorch에서 일반적인 이미지 Batch의 차원 순서는 다음과 같다.

$$
(B,C,H,W)
=(\text{Batch Size},\text{Channels},\text{Height},\text{Width})
$$

입력 Shape는 $(4,1,28,28)$이다. Batch Size는 4이고, 흑백 이미지의 채널 수는 1이다.

### 레이어별 Shape 변화

| 단계 | 설정·계산 | 출력 Shape |
| --- | --- | --- |
| 입력 | 흑백 이미지 4개 | $(4,1,28,28)$ |
| Conv1 + ReLU | 출력 채널 4개, 3×3, Stride 1, Padding 1 | $(4,4,28,28)$ |
| Max Pooling 1 | 2×2, Stride 2 | $(4,4,14,14)$ |
| Conv2 + ReLU | 출력 채널 8개, 3×3, Stride 1, Padding 1 | $(4,8,14,14)$ |
| Max Pooling 2 | 2×2, Stride 2 | $(4,8,7,7)$ |
| Flatten | 이미지마다 $8\times7\times7=392$개로 펼침 | $(4,392)$ |
| Linear | 특징 392개로 클래스별 점수 10개 계산 | $(4,10)$ |

여러 Convolution 레이어를 연결하면 앞선 레이어에서 계산한 특징을 조합해 새로운 특징을 계산할 수 있다. 다만 각 레이어가 반드시 ‘선’, ‘곡선’처럼 고정된 의미의 특징을 학습하는 것은 아니다.

Flatten은 각 이미지의 특징을 벡터 형태로 재배열하는 연산이며, Batch 차원은 유지한다. 모든 CNN이 Flatten과 Dense를 사용하는 것은 아니며, Global Average Pooling 같은 다른 구조도 있다.

### 로짓·확률·예측 클래스

출력 Shape $(4,10)$은 이미지마다 클래스별 로짓 10개를 계산했다는 뜻이다.

| 연산 | 계산 결과 |
| --- | --- |
| 출력 Linear | 클래스별 로짓 |
| Softmax | 이미지마다 합이 1인 클래스별 확률 |
| Argmax | 가장 큰 로짓에 대응하는 클래스 |

Softmax는 값의 대소관계를 유지하므로 최대 로짓과 최대 확률이 가리키는 클래스는 같다. 최종 클래스만 필요하다면 로짓에 직접 Argmax를 적용할 수 있다.

`CrossEntropyLoss`로 학습할 때는 **Softmax를 미리 적용하지 않고 로짓을 그대로 전달한다.**

## 7. PyTorch로 CNN 전체 구조 구현하기

다음 코드는 학습보다 각 단계의 Shape를 확인하는 데 목적이 있다.

```python
import torch
import torch.nn as nn

torch.manual_seed(42)
X = torch.randn(4, 1, 28, 28)

model = nn.Sequential(
    nn.Conv2d(1, 4, kernel_size=3, stride=1, padding=1),
    nn.ReLU(),
    nn.MaxPool2d(kernel_size=2, stride=2),
    nn.Conv2d(4, 8, kernel_size=3, stride=1, padding=1),
    nn.ReLU(),
    nn.MaxPool2d(kernel_size=2, stride=2),
    nn.Flatten(),
    nn.Linear(8 * 7 * 7, 10),
)

model.eval()

with torch.no_grad():
    h1 = model[:3](X)
    h2 = model[3:6](h1)
    flat = model[6](h2)
    logits = model[7](flat)

    probabilities = torch.softmax(logits, dim=1)
    predictions = torch.argmax(logits, dim=1)

print(tuple(X.shape))              # (4, 1, 28, 28)
print(tuple(h1.shape))             # (4, 4, 14, 14)
print(tuple(h2.shape))             # (4, 8, 7, 7)
print(tuple(flat.shape))           # (4, 392)
print(tuple(logits.shape))         # (4, 10)
print(tuple(probabilities.shape))  # (4, 10)
print(tuple(predictions.shape))    # (4,)

print(round(probabilities[0].sum().item(), 4))  # 1.0
```

`model[:3](X)`는 첫 번째 Conv·ReLU·Pooling을, `model[3:6](h1)`은 두 번째 블록을 계산한다. `dim=1`은 클래스 방향이므로, 각 이미지의 클래스별 확률과 예측 클래스를 계산한다.

입력은 임의의 값이고 모델도 아직 학습하지 않았다. 따라서 출력된 클래스에 실제 숫자 분류 성능의 의미를 부여할 수는 없다. PyTorch 예제는 PyTorch가 설치된 환경에서 실행한다.

## 8. CNN의 학습과 MLP 비교

CNN도 기본적인 학습 흐름은 동일하다.

| 단계 | 역할 |
| --- | --- |
| 순전파 | Convolution·Pooling·분류층으로 로짓 계산 |
| 손실 계산 | 로짓과 실제 정답 비교 |
| 역전파 | 학습 파라미터의 Gradient 계산 |
| Optimizer 업데이트 | Filter와 Dense의 가중치·편향 변경 |

일반적인 Max Pooling과 Average Pooling에는 학습 가능한 가중치가 없지만, 역전파에서 필요한 기울기를 이전 연산으로 전달할 수 있다.

즉, CNN은 분류층뿐 아니라 **특징을 계산하는 Filter까지 데이터로부터 학습하는 모델**이다.

### MLP와 CNN 비교

| 구분 | MLP | CNN |
| --- | --- | --- |
| 주요 구조 | Dense 레이어 중심 | Convolution 레이어 중심 |
| 이미지 처리 | 일반적으로 벡터로 펼쳐 입력 | 공간적 배치를 유지하며 처리 |
| 특징 계산 | 입력을 가중합으로 조합 | 국소 영역에 Filter 적용 |
| 가중치 사용 | 연결마다 가중치 사용 | 같은 Filter를 여러 위치에서 공유 |
| 대표 활용 | 표 형태 데이터 등 | 이미지·공간적 데이터 등 |

MLP도 이미지 분류에 사용할 수 있고, CNN도 이미지 이외의 데이터에 사용할 수 있다. 핵심은 CNN이 국소적 계산과 가중치 공유로 공간 구조를 활용하도록 설계되었다는 점이다.

## 핵심 정리

- Filter는 학습 가능한 가중치, Convolution은 이를 적용하는 연산, Feature Map은 계산 결과이다.
- CNN은 국소 영역의 계산과 가중치 공유로 이미지의 공간적 특징을 활용한다.
- Stride와 Padding은 Filter를 적용하는 위치와 출력 크기를 조절한다.
- Max Pooling은 최댓값, Average Pooling은 평균값을 남기며, 일반적으로 채널 수를 유지하고 공간적 크기를 줄인다.
- Pooling은 정보를 요약하며 일부 정보를 잃을 수 있고, 모든 CNN에 필수인 것은 아니다.
- Flatten은 Batch 차원을 유지하면서 이미지별 특징을 분류층에 전달할 벡터로 바꾼다.
- CNN도 역전파와 Optimizer를 이용해 Filter와 분류층의 파라미터를 학습한다.

## 오늘의 한 문장

> CNN은 같은 Filter를 이미지의 여러 위치에 적용해 특징을 학습하고, 필요에 따라 Pooling으로 정보를 요약한 뒤 분류층에서 이미지의 클래스를 예측하는 신경망이다.
