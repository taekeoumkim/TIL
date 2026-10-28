# 2026-10-28 TIL | Multi-Head Attention, Positional Encoding, Transformer의 Encoder·Decoder 구조

## 1. Multi-Head Attention: 여러 관점의 관계 계산

Multi-Head Attention은 여러 Attention Head가 각각 Attention을 계산하고, 그 결과를 결합하여 새로운 표현을 만드는 구조다.

“철수는 도서관에서 책을 읽었습니다.”에서 ‘읽었습니다’는 행동의 주체, 대상, 장소 등 여러 관계를 가질 수 있다. 여러 Head는 서로 다른 학습 가능한 변환을 사용하여 이러한 관계를 다양한 방식으로 계산할 수 있다.

단, 특정 Head가 주어만, 다른 Head가 목적어만 담당하도록 정해져 있는 것은 아니다. 실제 학습에서는 여러 Head가 비슷한 정보를 학습할 수도 있다.

### Head마다 다른 Query, Key, Value 생성

Self-Attention에서 입력 표현을 $X$라고 하면 Head $i$의 Query, Key, Value는 다음과 같다. 편향 항은 생략했다.

$$
Q_i=XW_i^Q,\qquad K_i=XW_i^K,\qquad V_i=XW_i^V
$$

- 같은 입력에서 출발하지만 Head마다 서로 다른 가중치 행렬을 사용한다.
- 각 Head의 Query, Key, Value와 Attention Weight는 서로 달라질 수 있다.
- Head들은 앞선 Head의 결과를 차례로 받는 구조가 아니므로 병렬로 계산할 수 있다.

실제 구현에서는 큰 선형 변환으로 모든 Head의 Query, Key, Value를 한꺼번에 계산한 뒤 Head별로 나누기도 한다.

### 각 Head에서 Scaled Dot-Product Attention 수행

$$
\mathrm{head}_i
=
\mathrm{softmax}\left(
\frac{Q_iK_i^\top}{\sqrt{d_k}}
\right)V_i
$$

여기서 $d_k$는 **해당 Head의 Key 벡터 차원**이다.

각 Head가 수행하는 계산은 앞서 학습한 **내적 → Scaling → Softmax → Value 가중합**과 같다.

### Concatenation과 Output Projection

여러 Head의 결과는 더하거나 평균 내는 대신 벡터의 마지막 차원으로 연결한다.

| 연산 | 예시 |
| --- | --- |
| Head 1 출력 | $[0.8,0.2]$ |
| Head 2 출력 | $[0.3,0.7]$ |
| Concatenation | $[0.8,0.2,0.3,0.7]$ |

이후 학습 가능한 출력 가중치 행렬 $W^O$를 적용한다.

$$
\mathrm{MultiHead}(X)
=
\mathrm{Concat}(
\mathrm{head}_1,\ldots,\mathrm{head}_h
)W^O
$$

Concatenation은 Head별 정보를 이어 붙이고, **Output Projection은 연결된 정보를 다시 조합하는 선형 변환**이다.

## 2. Multi-Head Attention의 Shape

배치 크기 $B=2$, 토큰 수 $N=3$, 모델 차원 $d_{\text{model}}=8$, Head 수 $h=2$인 대표적인 구성을 생각해 본다.

모델 차원을 Head별로 동일하게 나누면 Head당 차원은 다음과 같다.

$$
d_{\text{head}}=\frac{d_{\text{model}}}{h}=\frac{8}{2}=4
$$

| 단계 | Shape | 의미 |
| --- | --- | --- |
| 입력 $X$ | $(2,3,8)$ | 배치, 토큰 수, 모델 차원 |
| 각 Head의 $Q_i,K_i,V_i$ | $(2,3,4)$ | Head별로 변환한 표현 |
| 각 Head의 점수·가중치 | $(2,3,3)$ | 각 Query가 세 Key를 참조 |
| 각 Head의 출력 | $(2,3,4)$ | 토큰별 4차원 출력 |
| 두 Head의 Concatenation | $(2,3,8)$ | 마지막 차원으로 연결 |
| Output Projection 이후 | $(2,3,8)$ | 모델 차원에 맞춘 최종 출력 |

Head 차원을 명시해 함께 저장하면 $Q,K,V$를 $(B,h,N,d_{\text{head}})$, 즉 $(2,2,3,4)$로 나타낼 수도 있다.

**Head 수가 늘어난다고 최종 출력 차원이 반드시 늘어나는 것은 아니다.**

### Head가 많을수록 좋은 것은 아니다

모델 차원을 8로 고정하면 다음과 같다.

| Head 수 | Head당 차원 |
| ---: | ---: |
| 1 | 8 |
| 2 | 4 |
| 4 | 2 |

Head 수를 늘리면 각 Head가 사용하는 차원이 작아질 수 있다. 또한 모든 Head가 서로 다른 유용한 관계를 학습한다는 보장도 없다. Head 수는 모델의 표현 능력과 구조, 계산 비용을 함께 고려하는 설정이다.

## 3. Positional Encoding: 토큰의 위치 정보 반영

### Token Embedding만으로는 위치를 알 수 없다

Token Embedding은 토큰을 숫자 벡터로 표현한 값이다. 이 벡터 자체에는 해당 토큰이 시퀀스의 몇 번째 위치에 있는지 나타내는 정보가 없다.

위치 정보와 Mask가 없는 기본 Self-Attention에서는 토큰의 순서를 바꾸면 출력의 순서도 함께 바뀐다. 같은 토큰이 다른 위치로 이동했다는 이유만으로 그 토큰의 출력 표현이 달라지는 구조는 아니다.

따라서 토큰의 내용뿐 아니라 순서와 위치에 따른 차이를 활용하려면 위치 관련 정보를 제공해야 한다.

### Token Embedding에 위치 벡터 더하기

대표적인 방식은 토큰의 Embedding과 해당 위치의 벡터를 더하는 것이다.

$$
x_p=e_{\text{token}_p}+p_p
$$

| 기호 | 의미 |
| --- | --- |
| $x_p$ | 위치 정보가 반영된 입력 표현 |
| $e_{\text{token}_p}$ | 위치 $p$에 있는 토큰의 Embedding |
| $p_p$ | 위치 $p$를 나타내는 벡터 |

같은 토큰의 Embedding이 $[0.2,0.5,0.1]$이라고 하자.

| 위치 | 위치 벡터 | 더한 결과 |
| --- | --- | --- |
| 위치 0 | $[0.1,0.0,0.2]$ | $[0.3,0.5,0.3]$ |
| 위치 1 | $[0.0,0.2,0.1]$ | $[0.2,0.7,0.2]$ |

같은 토큰이라도 위치가 달라지면 입력 표현이 달라질 수 있다. 위치 번호 0, 1, 2를 그대로 더하는 것이 아니라 **위치를 벡터로 표현하여 결합**한다.

### 위치 표현의 종류

| 구분 | Sinusoidal Positional Encoding | Learned Positional Embedding |
| --- | --- | --- |
| 생성 방법 | Sin·Cos 함수로 계산 | 위치별 벡터를 학습 |
| 위치 벡터 자체의 학습 | 필요 없음 | 필요 |
| 특징 | 정해진 규칙으로 위치 표현 생성 | 데이터에 맞게 위치 표현 조정 |

모든 Transformer가 위치 벡터를 Token Embedding에 더하는 것은 아니다. 예를 들어 RoPE는 위치 정보를 Query와 Key에 반영하는 방식이다.

위치 벡터를 더하는 구성에서는 Shape가 유지된다.

$$
(B,N,d_{\text{model}})+(N,d_{\text{model}})
\;\longrightarrow\;
(B,N,d_{\text{model}})
$$

위치 벡터는 브로드캐스팅을 통해 배치의 각 시퀀스에 적용할 수 있다.

### NumPy로 토큰 순서에 따른 입력 표현 확인

아래 위치 벡터는 개념을 설명하기 위해 직접 지정했다. Sinusoidal 계산식이나 학습 과정을 구현한 예시는 아니다.

```python
import numpy as np

token_embeddings = np.array([
    [0.2, 0.5, 0.1],  # A
    [0.7, 0.1, 0.3],  # B
    [0.4, 0.6, 0.2],  # C
])
position_vectors = np.array([
    [0.1, 0.0, 0.2],  # 위치 0
    [0.0, 0.2, 0.1],  # 위치 1
    [0.2, 0.1, 0.0],  # 위치 2
])

input_a = token_embeddings[[0, 1, 2]] + position_vectors
input_b = token_embeddings[[1, 0, 2]] + position_vectors

print("입력 A:")
print(np.round(input_a, 3))
print("입력 B:")
print(np.round(input_b, 3))
print("Shape:", input_a.shape)

# 입력 A:
# [[0.3 0.5 0.3]
#  [0.7 0.3 0.4]
#  [0.6 0.7 0.2]]
# 입력 B:
# [[0.8 0.1 0.5]
#  [0.2 0.7 0.2]
#  [0.6 0.7 0.2]]
# Shape: (3, 3)
```

토큰 A는 첫 번째 입력에서는 위치 0, 두 번째 입력에서는 위치 1의 벡터와 결합한다. 이 코드는 Attention에 전달하기 전의 입력 표현이 달라지는 것을 보여준다.

## 4. Encoder와 Decoder의 기본 구조

원래 Transformer는 Encoder가 입력 시퀀스를 처리하고, Decoder가 그 정보를 참조하여 출력 시퀀스를 생성하는 구조다.

### Encoder: 문맥이 반영된 입력 표현 생성

Encoder는 입력 토큰들의 관계를 계산하여 각 위치에 문맥이 반영된 벡터를 만든다.

| 구성 요소 | 역할 |
| --- | --- |
| Token Embedding·위치 정보 | 토큰의 내용과 위치를 입력 표현에 반영 |
| Multi-Head Self-Attention | 입력 시퀀스 내부의 관계를 계산하여 정보 결합 |
| Feed Forward Network(FFN) | 각 토큰의 표현을 추가로 변환 |
| 여러 Encoder 레이어 | 위 연산을 반복하여 표현 생성 |

- 원래 Transformer의 Encoder는 미래 위치를 가리는 Causal Mask를 사용하지 않아 입력의 앞뒤 토큰을 참조할 수 있다.
- FFN은 일반적으로 Linear 레이어와 활성화 함수 등으로 구성하며, 같은 레이어의 FFN 가중치를 여러 토큰 위치에 공통으로 적용한다.
- Encoder는 입력 문장을 반드시 하나의 벡터로 압축하지 않는다. 일반적으로 **입력 토큰마다 대응하는 출력 벡터**를 생성한다.

### Decoder: 다음 토큰을 예측할 표현 생성

Encoder-Decoder 구조의 Decoder는 다음 세 가지 연산을 중심으로 구성된다.

1. **Masked Multi-Head Self-Attention**으로 출력 시퀀스의 현재·이전 위치 정보를 활용한다.
2. **Cross-Attention**으로 Encoder의 출력 정보를 참조한다.
3. **FFN**으로 각 위치의 표현을 추가로 변환한다.

Encoder와 Decoder는 여러 레이어를 쌓아 구성할 수 있다. 실제 Transformer 블록에는 Residual Connection과 Layer Normalization 등도 함께 사용된다.

### 어휘별 Logit에서 다음 토큰 예측

Decoder의 표현에 Linear 레이어를 적용하면 어휘별 Logit을 얻는다.

$$
z_t=h_tW_{\text{vocab}}+b_{\text{vocab}}
$$

확률이 필요한 경우 다음과 같이 Softmax를 적용한다.

$$
P(y_t\mid y_{<t},x)=\mathrm{softmax}(z_t)
$$

여기서 $x$는 입력 시퀀스, $y_{<t}$는 이전 출력 토큰들이다.

**Attention 내부의 Softmax는 참조 위치의 가중치를 구하고, 출력 단계의 Softmax는 어휘별 확률을 구한다.** 두 연산의 대상과 역할을 구분해야 한다.

PyTorch의 `CrossEntropyLoss`로 학습할 때는 일반적으로 출력에 Softmax를 미리 적용하지 않고 Logit을 그대로 전달한다.

## 5. Masked Self-Attention과 Cross-Attention

### Causal Mask: 미래 출력 토큰 참조 제한

다음 토큰을 예측할 때 미래의 정답 토큰을 미리 참조하면 안 된다. 이를 막기 위해 Decoder의 Self-Attention에서는 Causal Mask를 사용한다.

| Query 위치 | Key 위치 1 | Key 위치 2 | Key 위치 3 |
| --- | --- | --- | --- |
| 위치 1 | 참조 가능 | 참조 불가 | 참조 불가 |
| 위치 2 | 참조 가능 | 참조 가능 | 참조 불가 |
| 위치 3 | 참조 가능 | 참조 가능 | 참조 가능 |

행을 Query, 열을 Key로 두면 대각선 위쪽이 미래 위치다. 해당 위치의 점수를 음의 무한대로 바꾼 뒤 Softmax를 적용한다.

$$
M_{ij}=
\begin{cases}
0 & j\le i \\
-\infty & j>i
\end{cases}
$$

$$
A=\mathrm{softmax}\left(
\frac{QK^\top}{\sqrt{d_k}}+M
\right)
$$

마스킹한 위치는 $\exp(-\infty)=0$이 되어 Attention Weight가 0이 된다.

### NumPy로 Causal Mask 확인하기

아래 코드는 Attention의 점수가 이미 계산되었다고 가정하고, 미래 위치를 가리는 부분만 확인한다.

```python
import numpy as np

scores = np.array([
    [1.0, 2.0, 3.0],
    [2.0, 1.0, 3.0],
    [3.0, 2.0, 1.0],
])

# True인 위치는 참조할 수 없는 미래 위치다.
mask = np.triu(np.ones_like(scores, dtype=bool), k=1)
masked_scores = np.where(mask, -np.inf, scores)

shifted = masked_scores - np.max(
    masked_scores, axis=-1, keepdims=True
)
exp_scores = np.exp(shifted)
weights = exp_scores / np.sum(
    exp_scores, axis=-1, keepdims=True
)

print("Causal Mask:")
print(mask)
print("Attention Weight:")
print(np.round(weights, 3))

# Causal Mask:
# [[False  True  True]
#  [False False  True]
#  [False False False]]
# Attention Weight:
# [[1.    0.    0.   ]
#  [0.731 0.269 0.   ]
#  [0.665 0.245 0.09 ]]
```

- 첫 번째 Query는 첫 번째 위치만 참조하므로 가중치가 $[1,0,0]$이다.
- 두 번째 Query는 첫 번째와 두 번째 위치만 참조한다.
- 세 번째 Query는 세 위치를 모두 참조할 수 있다.
- Softmax는 각 행에 적용되며, 허용된 위치들의 가중치 합은 1이다.

### Cross-Attention: Encoder 출력 참조

Encoder-Decoder 구조의 Decoder에서는 자신의 현재 표현을 Query로 사용하고, Encoder 출력에서 Key와 Value를 만든다.

| 구분 | Masked Self-Attention | Cross-Attention |
| --- | --- | --- |
| 목적 | 출력 시퀀스 내부의 정보 활용 | 입력 시퀀스의 정보 활용 |
| Query의 출처 | Decoder 표현 | Decoder 표현 |
| Key·Value의 출처 | Decoder 표현 | Encoder 출력 |
| Causal Mask | 미래 출력 위치를 차단 | 일반적으로 같은 목적의 Causal Mask는 사용하지 않음 |

Cross-Attention은 Decoder의 Query와 Encoder의 Key를 비교하여 가중치를 구하고, Encoder의 Value를 가중합한다. 이를 통해 입력 문장의 정보를 출력 생성에 반영한다.

## 6. 학습과 Autoregressive 생성의 차이

### 실제 생성: 이전에 생성한 토큰을 다시 활용

번역 결과를 ‘나는 머신러닝을 좋아한다’로 생성한다고 가정하면 다음과 같이 진행할 수 있다.

| 단계 | Decoder에 제공한 출력 토큰 | 다음에 생성할 토큰의 예 |
| --- | --- | --- |
| 1 | `<BOS>` | 나는 |
| 2 | `<BOS>`, 나는 | 머신러닝을 |
| 3 | `<BOS>`, 나는, 머신러닝을 | 좋아한다 |
| 4 | `<BOS>`, 나는, 머신러닝을, 좋아한다 | `<EOS>` |

- `<BOS>`는 출력 시퀀스의 시작을 나타내는 토큰이다.
- `<EOS>`는 종료를 나타내는 토큰이다.
- 생성한 토큰을 다음 예측의 입력에 포함하는 방식을 **Autoregressive Generation**이라고 한다.

실제 토큰 분할과 생성 결과는 모델에 따라 달라질 수 있다. 위 예시는 생성 흐름을 단순화한 것이다.

### 학습: 정답 토큰을 한 칸 이동하여 입력으로 사용

정답 문장이 주어진 학습에서는 다음과 같이 입력과 예측 대상을 맞출 수 있다.

| 구분 | 토큰 배열 |
| --- | --- |
| Decoder 입력 | `<BOS>`, 나는, 머신러닝을, 좋아한다 |
| 예측할 정답 | 나는, 머신러닝을, 좋아한다, `<EOS>` |

이 배열에서 입력의 각 위치는 자신의 다음 토큰을 예측한다. 예를 들어 입력이 ‘나는’인 위치는 ‘머신러닝을’을 예측한다.

**자신의 입력 위치를 참조해도 되는 이유는 정답을 한 칸 이동시켜 입력했기 때문이다.** Causal Mask는 뒤쪽 입력에 포함된 미래 정답을 참조하지 못하도록 한다.

| 구분 | 학습 | 실제 생성 |
| --- | --- | --- |
| 이전 토큰의 출처 | 정답 시퀀스의 이전 토큰 | 모델이 이전에 생성한 토큰 |
| 미래 토큰 | 입력 배열에 있어도 Mask로 참조 차단 | 아직 생성되지 않음 |
| 계산 방식 | 여러 출력 위치의 손실을 병렬로 계산 가능 | 일반적인 Autoregressive 생성은 다음 토큰을 순차적으로 생성 |

Attention 계산이 병렬 처리에 유리하다는 사실과, 실제 생성이 이전 결과에 의존한다는 사실을 함께 구분해서 이해해야 한다.

## 7. 목적에 따른 Transformer 구조

Transformer를 사용하는 모든 모델이 Encoder와 Decoder를 모두 갖추는 것은 아니다.

| 구분 | Encoder-only | Decoder-only | Encoder-Decoder |
| --- | --- | --- | --- |
| 주요 역할 | 입력의 문맥 표현 생성 | 이전 토큰으로 다음 토큰 예측 | 입력 처리 후 출력 시퀀스 생성 |
| 대표 Attention | 양방향 Self-Attention | Causal Self-Attention | Encoder Self-Attention·Decoder의 Masked Self-Attention·Cross-Attention |
| 대표 모델 | BERT | GPT 계열 | 원래 Transformer, T5 |
| 대표 활용 | 문장 분류, 토큰 분류 | 텍스트 생성 | 번역, 입력에 기반한 생성 |

일반적인 Decoder-only 언어 모델에서 Encoder 출력을 참조하는 Cross-Attention은 필수 구성 요소가 아니다.

원래 Encoder-Decoder 구조의 전체 흐름은 다음과 같이 정리할 수 있다.

| 단계 | 처리 내용 |
| --- | --- |
| 입력 표현 | 원문 토큰의 Embedding에 위치 정보 반영 |
| Encoder | Self-Attention과 FFN으로 토큰별 문맥 표현 생성 |
| Decoder 입력 | 이전 출력 토큰의 Embedding에 위치 정보 반영 |
| Decoder | Masked Self-Attention, Encoder를 참조하는 Cross-Attention, FFN 수행 |
| 출력 | Linear로 어휘별 Logit을 구하고 다음 토큰 예측 |
| 생성 반복 | 생성한 토큰을 다음 입력에 포함하여 종료까지 반복 |

## 핵심 정리

- Multi-Head Attention은 서로 다른 변환을 사용하는 여러 Head의 결과를 연결하고 Output Projection으로 조합한다.
- Head 수가 늘어도 최종 모델 차원은 유지될 수 있으며, Head가 많다고 성능이 항상 좋아지는 것은 아니다.
- Positional Encoding은 위치 정보를 반영한다. 위치 벡터를 더하는 방식에서는 입력 Shape가 유지된다.
- Encoder는 입력 토큰마다 문맥이 반영된 표현을 만들고, FFN은 각 위치의 표현을 추가로 변환한다.
- Decoder의 Masked Self-Attention은 미래 출력 위치를 차단하고, Cross-Attention은 Encoder의 정보를 참조한다.
- 학습에서는 이동시킨 정답 토큰과 Causal Mask로 여러 위치를 병렬 계산할 수 있다.
- 일반적인 Autoregressive 생성에서는 이전에 생성한 토큰을 이용해 다음 토큰을 순차적으로 예측한다.
- Transformer는 목적에 따라 Encoder-only, Decoder-only, Encoder-Decoder 구조로 사용할 수 있다.

## 오늘의 한 문장

> Transformer는 여러 Attention Head로 관계를 계산하고 위치 정보를 반영하며, Encoder와 Decoder의 역할을 목적에 맞게 구성하여 입력을 이해하고 출력을 생성한다.