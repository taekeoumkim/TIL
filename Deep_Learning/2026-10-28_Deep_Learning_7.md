# 2026-10-28 TIL | Attention과 Self-Attention, Query·Key·Value, Scaled Dot-Product Attention

## 1. Attention: 관련성에 따라 정보를 결합하기

Attention은 현재 처리하는 정보와 다른 정보 사이의 관련성을 계산하고, 그 결과에 따라 여러 정보를 서로 다른 비중으로 결합하는 메커니즘이다.

예를 들어 “철수는 책을 샀습니다. 그는 그것을 읽었습니다.”에서 ‘그것을’의 표현을 계산할 때는 ‘책’의 정보가 도움이 될 수 있다. Attention은 이런 관계를 학습하여 각 위치의 정보를 반영한다.

전체 흐름은 다음과 같다.

1. 현재 위치와 참조할 위치 사이의 **관련성 점수(Attention Score)**를 계산한다.
2. 점수를 **가중치(Attention Weight)**로 변환한다.
3. 가중치를 이용해 정보 벡터들을 **가중합(Weighted Sum)**한다.

### Attention Score와 Attention Weight

Attention Score는 관련성을 나타내기 위해 계산한 점수다. 점수 자체가 확률이나 최종 가중치는 아니다.

일반적으로 Softmax를 적용하여 Attention Weight를 구한다.

$$
\alpha_j=\frac{\exp(s_j)}{\sum_{\ell=1}^{n}\exp(s_\ell)}
$$

- $s_j$: 참조할 위치 $j$의 점수
- $\alpha_j$: 해당 위치의 정보를 결합할 때 사용할 가중치
- $n$: 참조하는 위치의 개수

하나의 Query에 대해 계산한 가중치들의 합은 1이다.

| 참조할 정보 | Attention Score | Softmax로 구한 가중치 |
| --- | ---: | ---: |
| 철수 | 1.0 | 약 0.090 |
| 책 | 3.0 | 약 0.665 |
| 친구 | 2.0 | 약 0.245 |
| 합계 | — | 약 1.000 |

위 숫자는 계산 원리를 설명하기 위한 예시다. 실제 모델이 반드시 ‘책’에 위와 같은 가중치를 준다는 의미는 아니다.

### 가중합이 최종 출력을 만든다

정보의 값이 각각 10, 20, 30이고 가중치가 0.2, 0.5, 0.3이라면 다음과 같이 결합한다.

$$
10\times0.2+20\times0.5+30\times0.3=21
$$

실제 신경망에서는 정보가 벡터이므로 벡터에 가중치를 곱한 뒤 더한다.

$$
o=\sum_{j=1}^{n}\alpha_jv_j
$$

**Attention Weight는 정보를 결합하는 비중이며, Attention의 출력은 그 가중치로 Value 벡터들을 결합한 결과다.**

## 2. Self-Attention과 Cross-Attention

### Self-Attention: 같은 시퀀스 내부의 관계

Self-Attention은 동일한 입력 시퀀스에서 나온 요소들 사이에 Attention을 계산한다.

‘철수는 / 책을 / 읽었다’라는 입력이 있다면 각 위치는 자신을 포함한 시퀀스 내부의 여러 위치를 참조하여 새로운 표현을 만든다. 어떤 위치를 참조할 수 있는지는 Mask에 따라 제한될 수 있다.

Transformer는 일반적으로 입력을 **토큰(Token)** 단위로 처리한다. 토큰은 단어 전체일 수도 있고 단어의 일부일 수도 있다.

### 같은 단어도 문맥에 따라 표현이 달라진다

- “사과를 먹었다.”에서 ‘사과’는 과일을 뜻한다.
- “사과를 했다.”에서 ‘사과’는 잘못에 대해 사과하는 행동을 뜻한다.

Self-Attention은 다른 토큰의 정보를 반영하므로 같은 단어라도 문맥에 따라 서로 다른 표현을 계산할 수 있다.

### Cross-Attention: 다른 시퀀스의 정보 참조

Cross-Attention은 한 시퀀스의 표현으로 다른 시퀀스의 정보를 참조한다. Encoder-Decoder 번역 모델에서 Decoder가 Encoder의 출력을 참조하는 경우가 대표적이다.

| 구분 | Self-Attention | Cross-Attention |
| --- | --- | --- |
| 참조 관계 | 동일한 시퀀스 내부 | 서로 다른 시퀀스 또는 표현 사이 |
| Query의 출처 | 해당 시퀀스의 표현 | 정보를 참조하려는 쪽의 표현 |
| Key·Value의 출처 | Query와 동일한 시퀀스의 표현 | 참조할 다른 시퀀스의 표현 |
| 예시 | 원문 내부 토큰들의 관계 계산 | Decoder가 Encoder의 출력 참조 |

두 방식의 핵심 계산 원리는 같으며, **Query와 Key·Value를 어디에서 가져오는지**가 다르다.

## 3. Query, Key, Value의 역할과 생성

### 관련성 계산과 정보 전달의 역할 구분

| 구성 요소 | 역할 | 이해를 위한 질문 |
| --- | --- | --- |
| Query($Q$) | 관련성을 계산하는 기준 벡터 | 현재 위치에 어떤 정보가 필요한가? |
| Key($K$) | Query와 비교할 벡터 | 현재 Query와 얼마나 관련되어 있는가? |
| Value($V$) | 가중합하여 전달할 정보 벡터 | 어떤 정보를 출력에 반영할 것인가? |

Query와 Key를 비교해 점수를 구하고, 그 점수로 계산한 가중치를 Value에 적용한다.

예를 들어 ‘읽었다’ 위치의 Query를 $q_3$라고 하면 다음 순서로 처리한다.

1. $q_3$와 각 위치의 Key인 $k_1,k_2,k_3$를 비교한다.
2. 세 점수를 가중치 $\alpha_{31},\alpha_{32},\alpha_{33}$로 변환한다.
3. $v_1,v_2,v_3$를 가중합한다.

$$
o_3=\alpha_{31}v_1+\alpha_{32}v_2+\alpha_{33}v_3
$$

다른 위치에서는 그 위치의 Query를 사용하므로 가중치와 출력도 달라질 수 있다.

### 학습 가능한 선형 변환

Self-Attention에서는 같은 입력 표현 $X$에 서로 다른 가중치 행렬을 적용하여 $Q,K,V$를 만든다. 편향 항은 생략하면 다음과 같다.

$$
Q=XW_Q,\qquad K=XW_K,\qquad V=XW_V
$$

| 기호 | 의미 |
| --- | --- |
| $X$ | 토큰별 입력 표현을 모은 행렬 |
| $W_Q$ | Query를 생성하는 가중치 행렬 |
| $W_K$ | Key를 생성하는 가중치 행렬 |
| $W_V$ | Value를 생성하는 가중치 행렬 |

- $W_Q,W_K,W_V$는 역전파와 Optimizer를 통해 학습한다.
- 같은 입력에서 생성하더라도 서로 다른 변환을 거치므로 $Q,K,V$의 값이 같을 필요는 없다.
- Attention Weight는 입력과 학습된 변환을 바탕으로 계산되는 값이다. 변환 행렬의 가중치와 구분해서 이해해야 한다.

Self-Attention에서 ‘Self’는 **입력 시퀀스의 출처가 같다는 뜻**이다.

## 4. Scaled Dot-Product Attention의 계산 원리

Transformer에서 사용하는 대표적인 Attention 계산식은 다음과 같다.

$$
\mathrm{Attention}(Q,K,V)
=
\mathrm{softmax}\left(\frac{QK^\top}{\sqrt{d_k}}\right)V
$$

$d_k$는 **Key 벡터 하나의 차원**이다. Key의 개수나 시퀀스 길이와는 다르다.

### 1단계: Query와 Key의 내적

내적은 같은 위치의 원소끼리 곱한 뒤 더하는 연산이다.

$$
q\cdot k=\sum_{r=1}^{d_k}q_rk_r
$$

예를 들어 $q=[1,2]$, $k=[3,4]$이면 점수는 다음과 같다.

$$
q\cdot k=1\times3+2\times4=11
$$

여러 Query와 Key의 점수를 한꺼번에 계산하면 $QK^\top$가 된다. 내적은 벡터의 방향과 크기 모두에 영향을 받으므로, 높은 점수가 반드시 사람이 해석하는 의미적 유사성을 보장하지는 않는다.

### 2단계: 차원의 제곱근으로 Scaling

$$
S=\frac{QK^\top}{\sqrt{d_k}}
$$

Query와 Key의 차원이 커지면 일반적인 초기화 가정 아래에서 내적 점수의 크기가 커질 수 있다. 점수 간 차이가 지나치게 커지면 Softmax 결과가 특정 위치에 집중되고, Gradient가 작아져 학습이 어려워질 수 있다.

| Softmax에 넣은 점수 | 변환 결과 |
| --- | --- |
| $[1,2,3]$ | 약 $[0.090,0.245,0.665]$ |
| $[2,4,6]$ | 약 $[0.016,0.117,0.867]$ |

점수의 순서는 같지만, 점수 간 차이가 큰 두 번째 경우에는 가장 큰 점수의 가중치가 더 커진다.

이를 완화하기 위해 내적 점수를 $\sqrt{d_k}$로 나눈다.

- $d_k=4$라면 $\sqrt{d_k}=2$로 나눈다.
- $d_k=64$라면 $\sqrt{d_k}=8$로 나눈다.

Scaling은 차원 증가에 따른 점수 크기 문제를 완화하지만, 모든 상황에서 가중치 집중이나 학습 불안정을 완전히 없애는 것은 아니다.

### 3단계: Softmax로 Attention Weight 계산

각 Query에 대해 **Key 방향으로** Softmax를 적용한다.

$$
A_{ij}=\frac{\exp(S_{ij})}{\sum_{\ell=1}^{N_k}\exp(S_{i\ell})}
$$

$A_{ij}$는 Query $i$가 Key $j$에 대응하는 Value를 반영할 때 사용할 가중치다. 각 행의 가중치 합은 1이다.

Softmax는 모든 점수에 같은 상수를 더하거나 빼도 결과가 같다.

$$
\mathrm{softmax}(s-c)=\mathrm{softmax}(s)
$$

따라서 구현에서는 각 행의 최댓값을 뺀 뒤 지수함수를 계산하여 수치적 안정성을 높인다. 이는 점수의 차이를 줄이는 Scaling과 목적이 다르다.

### 4단계: Value의 가중합

$$
O=AV
$$

Softmax가 변환하는 대상은 **Query와 Key로 구한 점수**다. Value에는 계산된 가중치를 적용하여 최종 출력을 만든다.

전체 순서는 **내적 → Scaling → Softmax → Value의 가중합**이다.

## 5. 행렬의 Shape로 계산 흐름 확인하기

배치 차원과 Head 차원을 생략하고 하나의 Attention 계산을 살펴본다.

- $N_q$: Query의 개수
- $N_k$: Key와 Value의 개수
- $d_k$: Query와 Key 벡터의 차원
- $d_v$: Value 벡터의 차원

| 행렬 또는 연산 | Shape | 의미 |
| --- | --- | --- |
| $Q$ | $(N_q,d_k)$ | 출력할 위치마다 하나의 Query |
| $K$ | $(N_k,d_k)$ | 참조할 위치마다 하나의 Key |
| $V$ | $(N_k,d_v)$ | 참조할 위치마다 하나의 Value |
| $K^\top$ | $(d_k,N_k)$ | Key 행렬의 전치 |
| $QK^\top$ | $(N_q,N_k)$ | 각 Query와 각 Key의 점수 |
| $A$ | $(N_q,N_k)$ | 각 행에 Softmax를 적용한 가중치 |
| $AV$ | $(N_q,d_v)$ | Query마다 생성한 출력 벡터 |

계산이 성립하려면 Query와 Key의 벡터 차원이 같아야 한다. 또한 Key와 Value는 참조할 위치별로 대응하므로 개수가 같아야 한다. **Value의 차원 $d_v$는 $d_k$와 달라도 된다.**

길이 3인 시퀀스의 Self-Attention에서는 $N_q=N_k=3$이므로 점수 행렬이 $3\times3$이다. 각 행은 하나의 Query가 세 Key를 참조하는 점수이며, 행마다 다른 가중치와 출력이 계산된다.

Cross-Attention에서는 Query 쪽과 Key·Value 쪽의 시퀀스 길이가 다를 수 있다.

## 6. NumPy로 Attention 계산 확인하기

### 이미 주어진 가중치로 벡터 결합하기

먼저 Attention Weight가 주어졌다고 가정하고 가중합만 확인한다.

```python
import numpy as np

values = np.array([
    [1.0, 0.0],
    [0.0, 1.0],
    [1.0, 1.0],
])
weights = np.array([0.1, 0.7, 0.2])

output = weights @ values

print("가중치 합:", weights.sum())
print("출력:", output)

# 가중치 합: 1.0
# 출력: [0.3 0.9]
```

두 출력 원소는 다음과 같이 계산된다.

$$
o_1=0.1\times1+0.7\times0+0.2\times1=0.3
$$

$$
o_2=0.1\times0+0.7\times1+0.2\times1=0.9
$$

이 코드는 가중합만 보여준다. 실제 Attention에서는 Query와 Key를 이용해 가중치부터 계산한다.

### Scaled Dot-Product Attention 전체 계산

이번에는 하나의 Query와 세 개의 Key·Value를 사용한다. 아래 벡터는 계산 원리를 설명하기 위해 직접 지정한 값이다.

| 구분 | 벡터 |
| --- | --- |
| Query | $q=[2,0,0,0]$ |
| Key 1 | $k_1=[0,1,0,0]$ |
| Key 2 | $k_2=[1,0,0,0]$ |
| Key 3 | $k_3=[2,0,0,0]$ |
| Value 1 | $v_1=[1,0]$ |
| Value 2 | $v_2=[0,2]$ |
| Value 3 | $v_3=[1,1]$ |

Key의 개수는 3이지만, 각 Key의 차원인 $d_k$는 4이다. 따라서 내적 점수는 $\sqrt{4}=2$로 나눈다.

| 단계 | 계산 | 결과 |
| --- | --- | --- |
| 내적 | $q$와 각 Key 비교 | $[0,2,4]$ |
| Scaling | 점수를 2로 나누기 | $[0,1,2]$ |
| Softmax | 점수를 가중치로 변환 | 약 $[0.090,0.245,0.665]$ |
| 가중합 | 각 Value를 가중치로 결합 | 약 $[0.755,1.155]$ |

이를 NumPy로 구현하면 다음과 같다.

```python
import numpy as np


def softmax(x):
    shifted = x - np.max(x, axis=-1, keepdims=True)
    exp_x = np.exp(shifted)
    return exp_x / np.sum(exp_x, axis=-1, keepdims=True)


Q = np.array([[2.0, 0.0, 0.0, 0.0]])  # (1, 4)
K = np.array([
    [0.0, 1.0, 0.0, 0.0],
    [1.0, 0.0, 0.0, 0.0],
    [2.0, 0.0, 0.0, 0.0],
])  # (3, 4)
V = np.array([
    [1.0, 0.0],
    [0.0, 2.0],
    [1.0, 1.0],
])  # (3, 2)

scores = Q @ K.T                    # (1, 3)
d_k = K.shape[-1]
scaled_scores = scores / np.sqrt(d_k)
weights = softmax(scaled_scores)     # (1, 3)
output = weights @ V                # (1, 2)

print("내적 점수:", scores)
print("Scaling 결과:", scaled_scores)
print("Attention Weight:", np.round(weights, 3))
print("가중치 행별 합:", weights.sum(axis=-1))
print("Attention Output:", np.round(output, 3))
print("출력 Shape:", output.shape)

# 내적 점수: [[0. 2. 4.]]
# Scaling 결과: [[0. 1. 2.]]
# Attention Weight: [[0.09  0.245 0.665]]
# 가중치 행별 합: [1.]
# Attention Output: [[0.755 1.155]]
# 출력 Shape: (1, 2)
```

- `K.T`는 Key 행렬을 전치하여 모든 Query와 Key의 내적을 계산할 수 있게 한다.
- `axis=-1`은 각 Query에 대해 Key 방향으로 Softmax를 적용한다는 뜻이다.
- `keepdims=True`는 행별 최댓값과 합의 차원을 유지하여 브로드캐스팅이 가능하게 한다.
- `weights @ V`는 세 Value를 결합하여 하나의 출력 벡터를 만든다.
- 여러 Query를 행으로 쌓아 넣으면 같은 코드로 Query마다 하나의 출력을 계산할 수 있다.

## 7. RNN과의 차이, Self-Attention의 특징과 한계

### RNN과 정보 전달 방식 비교

| 구분 | RNN | Self-Attention |
| --- | --- | --- |
| 정보 활용 | 이전 Hidden State를 현재 시점으로 전달 | 참조 가능한 위치 사이의 관계를 직접 계산 |
| 주요 계산 | 현재 입력과 이전 상태의 결합 | 점수 계산과 Value의 가중합 |
| 시점 간 의존성 | 현재 상태 계산에 이전 상태가 필요 | 여러 위치의 관계를 행렬 연산으로 함께 계산 가능 |
| 멀리 있는 정보 | 중간 시점의 상태를 거쳐 전달 | 허용된 참조 범위 안에서 직접 결합 가능 |

Self-Attention은 시퀀스 내부의 여러 관계를 병렬로 계산하기에 유리하다. 다만 이것이 모든 작업에서 RNN보다 성능이 좋거나 항상 빠르다는 의미는 아니다.

### 긴 시퀀스의 계산 비용

기본적인 전체 Self-Attention에서 길이가 $N$이면 점수 행렬의 크기는 $N\times N$이다.

| 시퀀스 길이 $N$ | 점수의 개수 $N^2$ |
| ---: | ---: |
| 3 | 9 |
| 100 | 10,000 |
| 1,000 | 1,000,000 |

다른 조건이 같을 때 시퀀스 길이가 2배가 되면 점수의 개수는 4배가 된다. 긴 입력에서는 계산과 메모리 비용을 고려해야 한다.

### 순서 정보와 참조 범위

- **위치 정보**: Attention 연산 자체만으로 토큰의 순서가 자동으로 반영되는 것은 아니다. Transformer에서는 Positional Encoding 등의 방법으로 위치 정보를 제공한다.
- **Attention Mask**: 필요에 따라 참조할 수 있는 위치를 제한한다. 따라서 Self-Attention이라고 해서 항상 모든 토큰을 참조하는 것은 아니다.
- **점수 해석**: Attention Score와 Weight는 모델이 정보를 결합하기 위해 계산한 값이다. 이를 사람이 생각하는 의미적 관련성과 그대로 동일시해서는 안 된다.

## 핵심 정리

- Attention은 관련성에 따라 여러 정보에 서로 다른 가중치를 부여하여 결합한다.
- Self-Attention은 같은 시퀀스 내부를, Cross-Attention은 다른 시퀀스의 정보를 참조한다.
- Query와 Key는 점수를 계산하고, Value는 최종 출력에 반영할 정보를 제공한다.
- $Q,K,V$는 입력에 서로 다른 학습 가능한 선형 변환을 적용하여 생성한다.
- Scaled Dot-Product Attention은 **내적 → Scaling → Softmax → Value 가중합**의 순서로 계산한다.
- Softmax는 각 Query에 대해 Key 방향으로 적용하며, 각 행의 가중치 합은 1이다.
- Attention의 출력 개수는 Query 개수에 대응하고, 출력 벡터의 차원은 Value 차원에 대응한다.
- Self-Attention은 위치 간 관계 계산에 유리하지만, 위치 정보와 긴 시퀀스의 계산 비용도 함께 고려해야 한다.

## 오늘의 한 문장

> Attention은 Query와 Key로 참조할 정보의 비중을 계산하고, 그 비중에 따라 Value를 결합하여 문맥을 반영한 새로운 표현을 만드는 과정이다.