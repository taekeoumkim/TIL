# 2026-10-29 TIL | Training·Validation Loss, Learning Curve 분석, 학습 상태 진단과 개선 전략

## 1. Training Loss와 Validation Loss의 역할

Loss는 예측 결과와 정답의 차이를 손실 함수로 계산한 값이다. 일반적으로 작을수록 해당 손실 함수 기준의 예측 오차가 작다.

다만 Loss의 크기와 의미는 손실 함수, 데이터, 문제에 따라 달라진다. 특정 수치만으로 모델이 충분히 좋은지 판단하기보다 Baseline과 목표 평가 지표를 함께 확인해야 한다.

### 학습과 검증 과정의 차이

| 구분 | Training Loss | Validation Loss |
| --- | --- | --- |
| 사용 데이터 | 파라미터 학습에 사용하는 데이터 | 학습에 직접 사용하지 않은 검증 데이터 |
| 주요 목적 | 파라미터를 업데이트하고 학습 진행 확인 | 일반화 성능과 학습 상태 확인 |
| 역전파·업데이트 | 학습 단계에서 수행 | 일반적으로 수행하지 않음 |
| 활용 | 학습 데이터의 손실 감소 확인 | 모델·학습 설정 선택, Early Stopping |

학습에서는 **순전파 → 손실 계산 → 역전파 → Optimizer 업데이트**를 반복한다. 검증에서는 현재 모델로 예측하고 손실을 기록하며, 일반적으로 파라미터를 직접 업데이트하지 않는다.

Validation 결과는 설정 선택에 영향을 주므로 반복적인 조정 과정에서 Validation Data에도 간접적으로 맞춰질 수 있다. 최종 성능은 모델 선택을 마친 뒤 별도로 분리한 Test Data에서 평가한다.

## 2. Generalization Gap과 비교 조건

### 손실 기준의 일반화 격차

Training과 Validation 성능 사이의 차이를 Generalization Gap이라고 한다. 손실을 기준으로는 다음과 같이 계산할 수 있다.

$$
\mathrm{Gap}
=
L_{\text{val}}-L_{\text{train}}
$$

동일한 손실 함수와 비교 가능한 평가 조건을 사용했다고 가정하면 다음과 같다.

| 모델 | Training Loss | Validation Loss | Gap |
| --- | ---: | ---: | ---: |
| A | 0.20 | 0.60 | 0.40 |
| B | 0.30 | 0.35 | 0.05 |
| C | 2.00 | 2.00 | 0.00 |

모델 B는 A보다 Training Loss가 높지만 Validation Loss가 낮다. 현재 Validation Loss를 기준으로 보면 B의 결과가 더 좋다.

반면 C처럼 Gap이 0이어도 두 Loss와 예측 성능이 목표 수준에 미치지 못할 수 있다. **격차의 크기와 Validation 성능 자체를 함께 확인해야 한다.**

### Validation Loss가 더 낮아도 오류라고 단정할 수 없다

예를 들어 Training Loss가 0.45, Validation Loss가 0.38이면 Gap은 -0.07이다. 이런 결과가 나올 수 있는 이유는 다음과 같다.

| 원인 | 비교에 미치는 영향 |
| --- | --- |
| 학습·평가 모드의 차이 | Training에는 Dropout이 적용되고 Validation에는 적용되지 않을 수 있음 |
| BN 통계값의 차이 | 학습 모드와 평가 모드에서 정규화에 사용하는 통계가 다를 수 있음 |
| 데이터의 난이도·분포 차이 | Validation Data가 상대적으로 예측하기 쉬울 수 있음 |
| 측정 시점의 차이 | Training은 업데이트 도중의 평균, Validation은 Epoch 종료 후 모델로 계산할 수 있음 |
| 규제 항 포함 여부 | Training Loss에만 L2 규제 항 등을 포함했을 수 있음 |

따라서 두 Loss의 대소 관계만으로 계산 오류나 과적합을 확정할 수는 없다.

### 비교 조건을 통일하는 방법

더 공정한 비교가 필요하다면 Epoch 종료 후 같은 모델 상태로 다음과 같이 계산할 수 있다.

1. 모델을 평가 모드로 전환한다.
2. 같은 손실 함수를 사용한다.
3. 규제 항 포함 여부 등 계산 기준을 통일한다.
4. Training Data와 Validation Data에서 각각 손실을 계산한다.

PyTorch에서는 `model.eval()`로 평가 모드를 설정할 수 있다. 이 방식으로 계산 조건을 맞추더라도 데이터 자체의 분포나 난이도 차이는 남을 수 있다.

학습 중 기록한 Training Loss와, Epoch 종료 후 평가 모드로 다시 계산한 Training Loss는 측정 조건이 다르므로 구분해서 해석해야 한다.

## 3. Loss와 Accuracy가 함께 움직이지 않는 이유

Accuracy는 정답을 맞힌 비율이고, Loss는 손실 함수가 정의한 예측 오차다.

이진 분류에서 정답이 1이고 임계값이 0.5라고 하자. 이때 Binary Cross Entropy는 다음과 같다.

$$
L=-\ln p
$$

| 모델 | 정답 클래스의 예측 확률 $p$ | 예측 클래스 | 정답 여부 | BCE |
| --- | ---: | ---: | --- | ---: |
| A | 0.60 | 1 | 정답 | 약 0.511 |
| B | 0.95 | 1 | 정답 | 약 0.051 |

두 모델 모두 이 사례의 정답을 맞혔지만, 정답에 더 높은 확률을 부여한 B의 Loss가 작다.

이미 정답으로 분류한 사례들의 확률만 개선되면 Loss는 줄어도 Accuracy는 그대로일 수 있다. 따라서 **Loss 감소가 Accuracy나 F1 Score의 증가를 반드시 뜻하지는 않는다.**

모델의 목적에 맞게 Accuracy, Precision, Recall, F1 Score 등의 평가 지표도 함께 확인한다.

## 4. Learning Curve의 대표적인 패턴

Learning Curve는 학습 진행에 따른 Loss나 평가 지표의 변화를 나타낸 그래프다. 여기서는 X축을 Epoch, Y축을 Loss로 두고 Training·Validation 곡선을 함께 확인한다.

### 변화 방향·격차·발생 시점 관찰

| 관찰한 변화 | 가능한 해석 | 함께 확인할 점 |
| --- | --- | --- |
| 두 Loss가 함께 감소 | 두 데이터에서 손실이 개선되는 중 | 목표 성능과 평가 지표도 충분한가? |
| Training은 감소하고 Validation은 지속적으로 증가 | 과적합 가능성 | 분포·평가 조건의 차이, 최적 Validation 시점 |
| 성능이 부족한 상태에서 두 Loss가 거의 변하지 않음 | 학습 부족, 과소적합 또는 최적화 문제 | 학습률, Gradient, 모델·데이터 |
| Loss가 지속적으로 크게 증가 | 발산 또는 학습 불안정 가능성 | 학습률, Gradient 크기, 데이터 스케일 |

Learning Curve는 관찰한 현상을 보여주지만, 원인을 자동으로 확정해 주지는 않는다. 같은 패턴에도 서로 다른 원인이 있을 수 있다.

### 과소적합과 학습 정체 구분

| 구분 | 의미 |
| --- | --- |
| 과소적합(Underfitting) | 학습 데이터의 주요 관계를 충분히 학습하지 못한 상태 |
| 학습 정체(Stagnation) | 학습을 반복해도 Loss나 평가 지표가 거의 개선되지 않는 상태 |

Loss가 거의 변하지 않는 구간을 Plateau라고도 한다. 과소적합 상태에서 정체될 수도 있지만, 이미 좋은 성능에 도달한 뒤 정체될 수도 있다.

또한 Loss가 일시적으로 증가하거나 진동했다고 반드시 발산한 것은 아니다. Mini-Batch 구성 등에 따른 변동과 지속적인 증가 추세를 구분해야 한다.

## 5. 기록된 Loss로 Learning Curve 분석하기

다음은 학습 중 기록했다고 가정한 Loss 예시다.

| Epoch | Training Loss | Validation Loss | Gap |
| ---: | ---: | ---: | ---: |
| 1 | 0.90 | 0.95 | 0.05 |
| 2 | 0.70 | 0.75 | 0.05 |
| 3 | 0.50 | 0.55 | 0.05 |
| 4 | 0.35 | 0.60 | 0.25 |
| 5 | 0.20 | 0.75 | 0.55 |

- Training Loss는 계속 감소한다.
- Validation Loss는 Epoch 3까지 감소한 뒤 증가한다.
- Epoch 4부터 두 Loss의 격차가 커진다.
- 현재 기록 중 Validation Loss가 가장 낮은 시점은 **Epoch 3의 0.55**다.

이 패턴이 이어진다면 Epoch 3 이후 과적합 가능성을 검토할 수 있다. 다만 데이터와 계산 조건도 함께 점검해야 한다.

### NumPy와 Matplotlib으로 확인

아래 코드는 모델을 학습하는 코드가 아니라, 이미 기록된 Loss를 분석하고 시각화하는 예시다.

```python
import numpy as np
import matplotlib.pyplot as plt

epochs = np.arange(1, 6)
train_losses = np.array([0.90, 0.70, 0.50, 0.35, 0.20])
val_losses = np.array([0.95, 0.75, 0.55, 0.60, 0.75])

best_index = np.argmin(val_losses)
best_epoch = epochs[best_index]
best_val_loss = val_losses[best_index]

fig, ax = plt.subplots(figsize=(8, 4.5))
ax.plot(epochs, train_losses, marker="o", label="Training Loss")
ax.plot(epochs, val_losses, marker="o", label="Validation Loss")
ax.axvline(
    best_epoch,
    color="gray",
    linestyle="--",
    label=f"Best validation epoch: {best_epoch}",
)
ax.set_xlabel("Epoch")
ax.set_ylabel("Loss")
ax.set_title("Learning Curve")
ax.set_xticks(epochs)
ax.grid(alpha=0.3)
ax.legend()
fig.tight_layout()

print("Best Epoch:", best_epoch)
print("Minimum Validation Loss:", best_val_loss)
plt.show()

# Best Epoch: 3
# Minimum Validation Loss: 0.55
```

`np.argmin()`은 최솟값이 있는 배열 인덱스를 반환한다. 여기서는 인덱스 2에 해당하는 Epoch가 3이다.

**최적 Epoch를 찾아도 그 시점의 모델이 자동으로 복원되는 것은 아니다.** 실제 학습에서는 Validation 성능이 개선될 때 모델 상태를 Checkpoint로 저장하고, 사용할 시점에 복원해야 한다.

## 6. 학습 상태별 진단과 개선 전략

먼저 Training Loss의 변화, Validation Loss의 변화, 두 값의 수준과 차이, 문제가 시작된 시점을 확인한다. 그다음 데이터·모델·최적화 설정에서 가능한 원인을 검토한다.

### 과적합: 학습 데이터에는 잘 맞지만 검증 성능이 부족함

Training Loss는 낮아지는데 Validation Loss는 상대적으로 높거나 계속 증가하는 상황이다.

| 검토할 방법 | 확인 목적 |
| --- | --- |
| Dropout·Weight Decay | 특정 출력에 대한 의존이나 큰 가중치를 억제하는 규제 검토 |
| Early Stopping·Checkpoint | Validation 성능이 좋았던 시점의 모델 선택 |
| 모델 복잡도 조절 | 데이터에 비해 불필요하게 복잡한 구조인지 확인 |
| 데이터 확보·분할·전처리 점검 | 데이터의 다양성, 분포 차이, 평가 조건 확인 |

규제를 강하게 적용하거나 모델을 작게 만드는 것이 항상 정답은 아니다. 관찰한 현상과 가능한 원인을 기준으로 선택해야 한다.

### 과소적합: 학습·검증 성능이 모두 부족함

Training Data의 주요 관계도 충분히 학습하지 못한 상태다.

- 두 Loss가 여전히 감소 중이라면 학습 Epoch가 부족한지 확인한다.
- 모델이 필요한 관계를 표현하기 어렵다면 레이어·뉴런 수 등 표현 능력을 검토한다.
- Dropout이나 Weight Decay가 너무 강하다면 규제 강도를 줄여 비교한다.
- 학습률, 입력 특징, 전처리 등 학습에 필요한 조건도 확인한다.

성능이 부족하다는 이유만으로 모델 크기를 먼저 늘리기보다 **학습 횟수와 설정이 적절한지 확인**해야 한다.

### 학습 정체: 개선이 거의 나타나지 않음

먼저 현재 성능이 이미 충분한지 확인한다. 목표에 도달했다면 개선 폭이 작다는 사실 자체가 문제는 아닐 수 있다.

성능이 부족한 상태라면 다음 항목을 확인한다.

| 확인 항목 | 검토할 내용 |
| --- | --- |
| Learning Rate | 너무 작아 업데이트가 미미한가, 너무 커서 진동하는가? |
| Gradient | 역전파로 정상적인 Gradient가 계산되는가? |
| 모델·활성화 함수 | 표현 능력이나 Gradient 흐름에 문제가 있는가? |
| 입력·정답 | 데이터와 Label이 올바르게 구성되었는가? |

학습률이 너무 작아 정체되었다면 높이는 방향을 검토할 수 있지만, 큰 학습률로 진동하는 상황이라면 낮추는 것이 도움이 될 수 있다. **정체라는 현상만으로 조정 방향을 정할 수는 없다.**

### 발산·학습 불안정: Loss가 지속적으로 커짐

기본적인 업데이트는 다음과 같다.

$$
\theta_{t+1}
=
\theta_t-\eta g_t
$$

학습률 $\eta$ 또는 Gradient $g_t$가 지나치게 크면 업데이트가 과도해질 수 있다.

| 검토할 방법 | 확인 목적 |
| --- | --- |
| Learning Rate 감소 | 너무 큰 업데이트를 완화할 수 있는지 확인 |
| Gradient Clipping | Gradient가 지나치게 커지는 상황을 제한 |
| 입력 데이터 스케일 확인 | 큰 입력값이나 부적절한 정규화의 영향 확인 |
| 가중치 초기화 점검 | 초기 파라미터와 Gradient 크기가 적절한지 확인 |
| 수치 계산 확인 | `NaN`·`inf`, 0으로 나누기 등 비정상 연산 확인 |

Gradient Clipping이 모든 발산 문제를 해결하는 것은 아니다. 학습률과 데이터·수치 계산의 원인을 함께 확인한다.

## 7. 변경 효과를 공정하게 검증하기

새로운 기법을 적용했다는 사실보다 **Validation 성능과 목표 지표가 실제로 개선되었는지**가 중요하다.

다음은 같은 손실 함수와 비교 가능한 평가 조건에서 계산한 예시다.

| 구분 | Training Loss | Validation Loss | Gap |
| --- | ---: | ---: | ---: |
| Dropout 적용 전 | 0.15 | 0.60 | 0.45 |
| Dropout 적용 후 | 0.25 | 0.40 | 0.15 |

적용 후 Training Loss는 높아졌지만 Validation Loss는 낮아졌다. 이런 경우에는 Dropout이 일반화 성능 개선에 도움이 되었을 가능성이 있다.

Training Loss가 증가했다는 이유만으로 모델이 나빠졌다고 판단해서는 안 된다.

### 실험을 비교하는 기준

1. **평가 조건 통일:** 같은 모델 모드와 손실 함수, 규제 항 포함 기준으로 비교한다.
2. **비교할 변경 명확화:** 가능하면 한 번에 하나의 설정을 바꿔 어떤 변경이 영향을 주었는지 파악한다.
3. **목표 지표 확인:** Validation Loss와 함께 Accuracy·Precision·Recall·F1 등 문제에 맞는 지표를 확인한다.
4. **결과의 일관성 확인:** 초기화와 Mini-Batch 구성의 무작위성을 고려하고, 필요하면 같은 조건으로 반복 실험한다.
5. **최종 Test 평가:** Validation으로 모델과 설정을 선택한 뒤, 별도로 분리한 Test Data로 최종 성능을 확인한다.

학습 상태를 관찰하고 가능한 원인을 좁힌 뒤, 비교 조건을 유지하며 변경 효과를 검증하는 과정을 반복한다.

## 핵심 정리

- Training Loss는 학습 데이터에 대한 적합 정도를, Validation Loss는 검증 데이터에서의 손실을 보여준다.
- Gap이 작다는 사실만으로 좋은 모델이라고 판단할 수 없으며, Validation 성능 자체도 확인해야 한다.
- Validation Loss가 더 낮을 수 있으므로 모델 모드, 측정 시점, 규제 항, 데이터 특성을 함께 고려한다.
- Loss와 Accuracy는 서로 다른 정보를 제공하므로 목표 평가 지표도 확인한다.
- Learning Curve는 변화 방향·격차·발생 시점을 보여주며, 원인을 자동으로 확정하지는 않는다.
- 과소적합은 충분히 학습하지 못한 상태이고, 정체는 개선이 거의 없는 상태다.
- 정체나 발산에 대응할 때는 학습률과 Gradient, 데이터·모델 상태를 함께 점검한다.
- 최적 Validation Epoch의 모델은 실제로 저장·복원해야 사용할 수 있다.
- 변경 효과는 같은 조건에서 Validation으로 비교하고, Test Data는 최종 평가에 사용한다.

## 오늘의 한 문장

> 학습 곡선은 원인을 단정하는 답안이 아니라, 손실의 변화와 계산 조건을 바탕으로 가설을 세우고 개선 효과를 검증하는 근거다.