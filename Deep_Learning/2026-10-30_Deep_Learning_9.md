# 2026-10-30 TIL | 과적합과 일반화, Dropout·Weight Decay, Batch Normalization·Early Stopping

## 1. 과적합과 일반화

### 학습 데이터의 성능만으로 모델을 판단할 수 없다

**과적합(Overfitting)**은 모델이 학습 데이터의 세부 패턴이나 노이즈에 지나치게 맞춰져 새로운 데이터에서 충분한 성능을 얻지 못하는 현상이다.

**일반화(Generalization)**는 학습 과정에서 보지 못한 데이터에도 학습한 관계를 잘 적용하는 능력이다.

모델을 만드는 목적은 새로운 데이터에 대한 예측이므로, 학습 성능과 함께 별도 데이터에서의 성능을 확인해야 한다.

### Training·Validation·Test Data의 역할

| 구분 | 주요 역할 | 활용 예 |
| --- | --- | --- |
| Training Data | 가중치와 편향 등 파라미터 학습 | 순전파, 손실 계산, 역전파, 업데이트 |
| Validation Data | 모델과 학습 설정 선택 | 학습률·규제 강도 비교, Early Stopping |
| Test Data | 선택이 끝난 모델의 최종 성능 평가 | 최종 일반화 성능 확인 |

Validation Data로 설정을 선택하더라도 일반적으로 그 데이터의 손실을 역전파하여 가중치를 직접 업데이트하지는 않는다.

Test Data를 반복해서 확인하며 모델을 수정하면 Test Data에도 간접적으로 맞춰질 수 있다. 최종 평가 데이터는 모델 선택 과정과 분리해야 한다.

### Training과 Validation 성능을 함께 해석하기

| 모델 | Training 정확도 | Validation 정확도 | 해석 |
| --- | ---: | ---: | --- |
| A | 98% | 70% | 과적합 가능성 또는 데이터·평가 조건의 차이 확인 |
| B | 90% | 88% | 세 모델 중 Validation 정확도가 가장 높음 |
| C | 65% | 63% | 학습 부족이나 과소적합 가능성 확인 |

- Training 성능은 높고 Validation 성능이 낮다면 과적합을 의심할 수 있다.
- 두 성능이 모두 낮다면 학습 데이터의 관계조차 충분히 학습하지 못한 **과소적합(Underfitting)**일 수 있다.
- 두 성능의 차이가 작다는 이유만으로 좋은 모델이라고 판단할 수는 없다.
- 정확도 수준은 클래스 분포와 Baseline 성능 등을 함께 고려해야 한다.

### 원인과 대응 방향

| 확인할 문제 | 검토할 대응 |
| --- | --- |
| 데이터에 비해 모델이 지나치게 복잡함 | 레이어·뉴런 수 등 모델 복잡도 조절 |
| 학습 데이터의 양이나 다양성이 부족함 | 문제를 대표하는 데이터와 품질 확보 |
| 노이즈나 우연한 패턴까지 학습함 | 데이터 점검, 적절한 규제 적용 |
| 학습을 오래 진행하며 Validation 성능이 악화됨 | Early Stopping 검토 |
| Training·Validation 분포나 평가 조건이 다름 | 데이터 분할과 전처리·평가 방식 확인 |

성능 차이만으로 원인을 확정하지 말고 시간·그룹을 고려한 데이터 분할, 전처리 기준, 데이터 누수 여부도 확인한다.

## 2. Dropout: 일부 출력을 확률적으로 비활성화

Dropout은 학습 중 일부 뉴런의 출력을 무작위로 0으로 만들어 특정 출력에 대한 과도한 의존을 완화하는 기법이다.

뉴런이나 가중치를 영구적으로 삭제하는 것이 아니라, 순전파할 때 일부 출력을 임시로 비활성화한다. 다음 순전파에서는 다른 출력이 비활성화될 수 있다.

### Dropout Rate는 비활성화 확률이다

Dropout Rate를 $p$라고 하면 각 대상 출력은 확률 $p$로 비활성화되고, 확률 $1-p$로 유지된다.

| $p$ | 비활성화 확률 | 유지 확률 |
| ---: | ---: | ---: |
| 0.1 | 10% | 90% |
| 0.2 | 20% | 80% |
| 0.5 | 50% | 50% |

확률적으로 결정하므로 매번 정확히 전체 출력의 $p$ 비율이 0이 되는 것은 아니다.

### Inverted Dropout의 크기 보정

일반적인 Inverted Dropout은 학습 중 유지된 출력에 $1/(1-p)$를 곱한다.

$$
\tilde h_i=\frac{m_i h_i}{1-p},
\qquad
m_i\sim\mathrm{Bernoulli}(1-p),
\qquad 0\le p<1
$$

| 기호 | 의미 |
| --- | --- |
| $h_i$ | Dropout 적용 전 출력 |
| $m_i$ | 유지하면 1, 비활성화하면 0인 마스크 |
| $\tilde h_i$ | 보정까지 적용한 학습 중 출력 |

유지 확률이 $1-p$이므로 보정을 거친 출력의 기대값은 원래 출력과 같다.

$$
\mathrm{E}[\tilde h_i]
=\frac{(1-p)h_i}{1-p}
=h_i
$$

이는 한 번의 실행에서 출력값이나 출력 합계가 반드시 유지된다는 뜻은 아니다.

### NumPy로 보정 원리 확인하기

아래 마스크는 설명을 위해 고정한 한 번의 비활성화 결과다. 실제 Dropout에서는 마스크를 확률적으로 생성한다.

```python
import numpy as np

h = np.array([2.0, 4.0, 6.0, 8.0])
p = 0.5
mask = np.array([1.0, 0.0, 1.0, 0.0])

train_output = h * mask / (1 - p)
eval_output = h.copy()

print("학습 모드:", train_output)
print("평가 모드:", eval_output)

# 학습 모드: [ 4.  0. 12.  0.]
# 평가 모드: [2. 4. 6. 8.]
```

Inverted Dropout을 사용하는 평가·추론 모드에서는 출력을 비활성화하지 않고, 추가 크기 보정 없이 그대로 전달한다.

Dropout을 너무 강하게 적용하면 필요한 정보까지 충분히 활용하지 못해 학습이 어려워질 수 있다.

## 3. Weight Decay: 가중치 크기를 억제하는 업데이트

Weight Decay는 가중치를 0에 가까워지게 하는 방향의 항을 파라미터 업데이트에 추가하는 방법이다.

### 기본 SGD에서의 계산

학습률을 $\eta$, Weight Decay 계수를 $\lambda$, 데이터 손실의 Gradient를 $g_t$라고 하면 다음과 같다.

$$
w_{t+1}
=w_t-\eta g_t-\eta\lambda w_t
=(1-\eta\lambda)w_t-\eta g_t
$$

$\lambda$가 규제 강도를 조절한다. 위 식은 Momentum 등이 없는 기본 SGD를 기준으로 한 설명이며, Optimizer에 따라 구체적인 적용 방식은 달라질 수 있다.

현재 가중치가 2, Gradient가 0.4, 학습률이 0.1, $\lambda=0.05$라면 다음과 같다.

| 구분 | 계산 | 새 가중치 |
| --- | --- | ---: |
| 기본 SGD | $2-0.1\times0.4$ | 1.96 |
| Weight Decay 적용 | $2-0.1\times0.4-0.1\times0.05\times2$ | 1.95 |

Weight Decay 항은 가중치의 크기를 줄이는 방향으로 작용한다. 다만 데이터 손실의 Gradient도 함께 작용하므로 **매 업데이트에서 최종 가중치의 절댓값이 반드시 줄어드는 것은 아니다.**

가중치가 크다는 사실만으로 과적합이라고 판단할 수도 없다.

### L2 Regularization과의 관계

L2 Regularization은 데이터 손실에 가중치 제곱합의 규제 항을 더한다.

$$
L_{\text{total}}
=
L_{\text{data}}
+
\frac{\lambda}{2}\sum_j w_j^2
$$

규제 항을 $w_j$로 미분하면 $\lambda w_j$가 추가된다. 따라서 기본 SGD에서는 위 L2 규제와 Weight Decay가 같은 업데이트를 만들 수 있다.

하지만 Adam과 같은 적응적 Optimizer에서는 두 방식이 항상 같지는 않다. **AdamW는 Weight Decay를 Gradient 기반 업데이트와 분리하여 적용**한다.

### Dropout과 Weight Decay 비교

| 구분 | Dropout | Weight Decay |
| --- | --- | --- |
| 조절 대상 | 신경망의 중간 출력 | 파라미터 업데이트 |
| 동작 | 일부 출력을 확률적으로 0으로 만듦 | 가중치 크기를 줄이는 방향의 항 적용 |
| 적용 시점 | 학습 중 순전파 | 학습 중 업데이트 |
| 규제 자체의 무작위성 | 있음 | 없음 |
| 목적 | 특정 출력에 대한 과도한 의존 완화 | 불필요하게 큰 가중치 억제 |

두 기법은 함께 사용할 수도 있지만, 적용 여부와 강도는 Validation 성능으로 판단해야 한다.

## 4. Batch Normalization: 중간 출력의 정규화

Batch Normalization(BN)은 Mini-Batch의 통계값으로 신경망의 중간 출력을 정규화하고, 학습 가능한 Scale과 Shift를 적용하는 방법이다.

학습 중 값의 크기를 조절하여 학습 안정성과 속도 개선에 도움을 줄 수 있다. 모든 모델에서 반드시 성능이 좋아지는 것은 아니다.

### 평균·분산 계산과 정규화

하나의 특성에 대해 Mini-Batch에 $m$개의 값이 있다고 하면 다음과 같이 계산한다.

$$
\mu_B=\frac{1}{m}\sum_{i=1}^{m}x_i
$$

$$
\sigma_B^2
=\frac{1}{m}\sum_{i=1}^{m}(x_i-\mu_B)^2
$$

$$
\hat x_i
=\frac{x_i-\mu_B}{\sqrt{\sigma_B^2+\epsilon}}
$$

| 기호 | 의미 |
| --- | --- |
| $\mu_B$ | 현재 Mini-Batch의 평균 |
| $\sigma_B^2$ | 현재 Mini-Batch의 분산 |
| $\epsilon$ | 0으로 나누는 문제 등을 방지하는 작은 양수 |
| $\hat x_i$ | 정규화된 값 |

예를 들어 $[2,4,6,8]$의 평균과 분산은 모두 5이다.

$$
\mu_B=\frac{2+4+6+8}{4}=5
$$

$$
\sigma_B^2=\frac{9+1+1+9}{4}=5
$$

$\epsilon$의 영향을 무시하면 정규화 결과는 약 $[-1.342,-0.447,0.447,1.342]$이다.

### 학습 가능한 Scale과 Shift

정규화한 값에는 학습 가능한 파라미터 $\gamma,\beta$를 적용한다.

$$
y_i=\gamma\hat x_i+\beta
$$

- $\gamma$: 정규화된 값의 크기를 조절하는 Scale
- $\beta$: 값을 이동시키는 Shift

정규화 직후에는 평균이 약 0, 분산이 약 1이지만, **Scale과 Shift까지 적용한 최종 출력이 반드시 평균 0·분산 1인 것은 아니다.**

실제 신경망에서는 여러 특성 또는 채널에 대해 각각 정규화를 적용한다.

### NumPy로 정규화와 Scale·Shift 확인하기

아래에서는 계산을 확인하기 위해 $\gamma=2$, $\beta=1$로 직접 지정했다.

```python
import numpy as np

x = np.array([2.0, 4.0, 6.0, 8.0])
epsilon = 1e-5
gamma, beta = 2.0, 1.0

mean = x.mean()
variance = x.var(ddof=0)
normalized = (x - mean) / np.sqrt(variance + epsilon)
output = gamma * normalized + beta

print("평균:", mean)
print("분산:", variance)
print("정규화:", np.round(normalized, 3))
print("최종 출력:", np.round(output, 3))

# 평균: 5.0
# 분산: 5.0
# 정규화: [-1.342 -0.447  0.447  1.342]
# 최종 출력: [-1.683  0.106  1.894  3.683]
```

## 5. 학습 모드와 평가·추론 모드

Dropout과 Batch Normalization은 학습 중인지 평가 중인지에 따라 동작이 달라진다.

| 레이어 | 학습 모드 | 평가·추론 모드 |
| --- | --- | --- |
| Dropout | 출력을 무작위로 비활성화하고 유지된 출력에 보정 적용 | 출력을 그대로 전달 |
| Batch Normalization | 현재 Mini-Batch의 통계로 정규화하고 이동 통계 갱신 | 학습 중 누적한 통계로 정규화하며 통계 갱신 안 함 |

BN의 비교는 학습 중 이동 통계를 누적하여 사용하는 일반적인 설정을 기준으로 한다. Scale과 Shift에는 두 모드 모두 학습된 $\gamma,\beta$가 적용된다.

PyTorch에서는 `model.train()`과 `model.eval()`로 실행 모드를 전환한다. Validation과 Test 평가 시에는 평가 모드로 설정해야 한다.

## 6. Early Stopping: Validation 성능으로 중단 시점 판단

Early Stopping은 Validation 성능이 더 이상 개선되지 않을 때 학습을 중단하는 방법이다.

Training Loss가 계속 줄어도 Validation 성능이 좋아진다는 보장은 없다. 따라서 중단 기준에는 일반적으로 Validation Loss 또는 문제에 맞는 Validation 평가 지표를 사용한다.

| 지표 | 개선 방향 |
| --- | --- |
| Validation Loss | 작아질수록 개선 |
| Validation Accuracy | 커질수록 개선 |
| Validation F1 Score | 커질수록 개선 |

### Patience로 일시적인 성능 변동 허용

Patience는 성능이 개선되지 않아도 기다릴 횟수다. 매번 **지금까지 기록한 최고 성능**과 비교하며, 개선되면 최고 기록을 갱신하고 미개선 횟수를 초기화한다.

Validation Loss를 기준으로 `patience=2`를 적용하는 예시는 다음과 같다.

| Epoch | Training Loss | Validation Loss | 연속 미개선 횟수 | 판단 |
| ---: | ---: | ---: | ---: | --- |
| 1 | 0.80 | 0.85 | 0 | 현재 최고 기록 저장 |
| 2 | 0.60 | 0.65 | 0 | 개선, 최고 기록 갱신 |
| 3 | 0.40 | 0.50 | 0 | 개선, 최고 기록 갱신 |
| 4 | 0.30 | 0.55 | 1 | 기다리며 학습 계속 |
| 5 | 0.20 | 0.65 | 2 | 학습 중단 |

이 예시에서는 두 번 연속 개선되지 않은 Epoch 5에서 중단한다. 한 번의 Validation Loss 증가만으로 즉시 중단하지 않아 일시적인 변동을 허용할 수 있다.

### 중단 시점과 최적 모델 시점은 다르다

위 예시의 중단 시점은 Epoch 5이지만, 가장 낮은 Validation Loss는 **Epoch 3의 0.50**이다.

따라서 다음과 같이 학습을 관리한다.

1. Epoch마다 Validation 성능을 확인한다.
2. 최고 성능이 개선될 때 해당 모델 상태를 저장한다.
3. 정해진 횟수 동안 개선되지 않으면 중단한다.
4. 저장한 최고 성능 모델을 복원하여 사용한다.
5. 모델 선택을 마친 뒤 별도의 Test Data로 최종 평가한다.

Early Stopping을 사용한다고 모든 구현이 자동으로 최적 상태를 복원하는 것은 아니다. 최고 성능 시점의 모델을 저장하고 복원하도록 설정했는지 확인해야 한다.

## 7. 네 가지 기법을 목적에 맞게 활용하기

| 기법 | 주요 목적 | 조절하는 부분 | 주요 설정·파라미터 |
| --- | --- | --- | --- |
| Dropout | 특정 출력에 대한 과도한 의존 완화 | 학습 중 순전파의 출력 | 비활성화 확률 $p$ |
| Weight Decay | 지나치게 큰 가중치 억제 | 파라미터 업데이트 | 규제 계수 $\lambda$ |
| Batch Normalization | 중간 출력 정규화와 학습 안정화 | 신경망 내부 레이어의 계산 | 학습 가능한 $\gamma,\beta$ 등 |
| Early Stopping | 적절한 학습 중단 시점 선택 | 전체 학습 과정 | Validation 지표, Patience |

기법의 이름보다 현재 학습 상태와 적용 목적을 먼저 확인해야 한다.

- Training 성능은 좋고 Validation 성능이 낮다면 과적합과 데이터·평가 조건을 점검하고 규제를 검토한다.
- 두 성능이 모두 낮다면 모델의 표현 능력, 학습 부족, 지나친 규제 등 과소적합 원인을 확인한다.
- 학습이 불안정하다면 학습률과 모델 구조 등을 확인하고 BN과 같은 학습 안정화 방법을 검토한다.
- 학습 후반에 Validation 성능이 악화된다면 Early Stopping과 최고 성능 모델 저장을 검토한다.

규제를 강하게 적용하거나 여러 기법을 동시에 사용한다고 일반화 성능이 반드시 좋아지는 것은 아니다. 적용 여부와 강도는 Validation 성능을 기준으로 판단한다.

## 핵심 정리

- 일반화 성능을 판단하려면 Training과 Validation 성능을 함께 확인해야 한다.
- Training·Validation의 성능 차이만으로 과적합을 확정할 수 없으며, 두 성능이 모두 낮다면 과소적합도 확인한다.
- Dropout은 출력을 확률적으로 비활성화한다. Inverted Dropout은 학습 중 유지된 출력을 $1/(1-p)$배 보정한다.
- Weight Decay는 가중치를 줄이는 방향의 업데이트를 추가하지만, 최종 절댓값이 매번 줄어드는 것은 아니다.
- 기본 SGD에서는 L2 규제와 Weight Decay가 같은 업데이트를 만들 수 있지만, 적응적 Optimizer에서는 구분이 필요하다.
- BN은 Mini-Batch 통계로 정규화하고 학습 가능한 Scale·Shift를 적용한다.
- Dropout과 BN은 학습·평가 모드에서 동작이 달라 모드를 올바르게 설정해야 한다.
- Early Stopping의 중단 시점과 최고 성능 모델의 시점은 다를 수 있으므로 최적 모델을 별도로 저장·복원한다.
- Validation으로 설정과 모델을 선택하고, Test Data는 최종 평가에 사용한다.

## 오늘의 한 문장

> 좋은 학습은 Training Loss를 끝까지 낮추는 것이 아니라, 적절한 규제와 학습 관리로 새로운 데이터에서도 유용한 관계를 유지하는 과정이다.