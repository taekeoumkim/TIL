# 2026-10-30 TIL | Scratch Training·전이학습 전략 비교, 평가 지표와 학습 효율

## 1. Scratch Training과 Transfer Learning의 출발점

동일한 CNN 구조로 강판의 7가지 결함 유형을 분류하더라도, 어떤 가중치에서 학습을 시작하는지에 따라 과정과 결과가 달라질 수 있다.

| 구분 | Training from Scratch | Transfer Learning |
| --- | --- | --- |
| Backbone 초기 가중치 | 새롭게 초기화 | 사전학습 가중치 활용 |
| 기존 학습 정보 | 활용하지 않음 | 기존 특징이나 표현 활용 |
| 특징 학습 | Target Data로 처음부터 학습 | 기존 특징을 재사용하거나 조정 |
| 주요 고려 사항 | 충분한 데이터와 학습 자원 | 기존 특징의 적합성과 학습 범위 |

Scratch Training과 Full Fine-tuning은 모두 전체 가중치를 학습할 수 있다. 두 방법의 핵심 차이는 **초기 가중치에 사전학습 정보를 활용하는지**다.

### 데이터 양과 기존 특징의 적합성 함께 고려

| 상황 | 검토할 수 있는 방향 |
| --- | --- |
| 데이터가 적고 기존 특징이 유용함 | 사전학습 모델 활용 |
| 데이터가 충분하고 학습 자원이 있음 | Scratch와 전이학습을 비교 |
| 기존 특징 일부의 조정이 필요함 | Fine-tuning 검토 |
| 기존 특징이 새 문제에 적합하지 않음 | 학습 범위 조정 또는 Scratch 검토 |

데이터가 적다고 전이학습이 항상 성공하거나, 데이터가 많다고 Scratch가 항상 좋은 것은 아니다.

Source Task와 Target Task가 달라도 일부 특징은 유용할 수 있다. 반대로 기존 정보 활용이 새 작업에 불리하게 작용하는 **Negative Transfer**도 가능하므로 실제 Validation 성능을 확인해야 한다.

## 2. Feature Extraction·Partial·Full Fine-tuning

세 방식은 모두 사전학습 가중치를 활용하지만, Backbone의 업데이트 범위가 다르다.

| 구분 | Feature Extraction | Partial Fine-tuning | Full Fine-tuning |
| --- | --- | --- | --- |
| Backbone | 전체 고정 | 일부 고정, 일부 학습 | 전체 학습 |
| 새로운 분류층 | 학습 | 학습 | 학습 |
| 기존 특징의 변경 | 고정된 특징 재사용 | 선택한 범위 조정 | 전체 범위 조정 |
| 주요 장점 | 학습 부담을 줄일 수 있음 | 유지와 조정의 균형 | 폭넓은 특징 조정 |
| 고려 사항 | 기존 특징의 적합성 | 학습할 레이어 선택 | 계산 비용과 과적합 가능성 |

Partial Fine-tuning에서는 뒷부분 Block부터 조정하는 방법을 검토할 수 있지만, 학습 범위는 모델과 데이터에 따라 선택한다.

### 학습 범위와 Learning Rate 함께 조정

사전학습 Backbone과 새 분류층은 출발점이 다르므로 서로 다른 학습률을 사용할 수 있다.

| 학습 대상 | 학습률 예시 |
| --- | ---: |
| 사전학습 Backbone | $10^{-4}$ |
| 새로운 분류층 | $10^{-3}$ |

Backbone의 큰 업데이트는 기존의 유용한 특징을 손상시킬 수 있고, 지나치게 작은 학습률은 적응을 느리게 할 수 있다. 위 값은 예시이며 모든 모델에 적합한 규칙은 아니다.

### Freeze와 평가 모드는 별도로 관리

- **Freeze:** 파라미터를 업데이트할지 제어한다.
- **학습·평가 모드:** Dropout과 Batch Normalization 등의 동작을 제어한다.

Gradient 계산을 중지한 파라미터가 있어도 학습 모드의 BN 통계는 갱신될 수 있다. 고정된 특징 추출기를 사용하려면 파라미터와 실행 상태를 함께 관리해야 한다.

Feature Extraction에서도 새 분류층은 과적합될 수 있다. Full Fine-tuning도 데이터와 설정이 적절하면 좋은 일반화 성능을 얻을 수 있으므로, 전략 이름만으로 과적합 여부를 결정할 수는 없다.

## 3. 학습 가능한 파라미터 수 비교

다음은 비교 원리를 설명하기 위한 **가상 CNN**의 구성이다. 실제 ResNet18의 파라미터 수와는 별도로 정의한 예시다.

| 구성 요소 | 파라미터 수 |
| --- | ---: |
| Backbone 전체 | 10,000,000 |
| Backbone에 포함된 마지막 Block | 2,000,000 |
| 새로운 분류층 | 3,500 |

Partial Fine-tuning에서 마지막 Block과 분류층을 학습한다고 가정한다.

| 학습 전략 | 계산 | 학습 가능한 파라미터 수 |
| --- | --- | ---: |
| Scratch | Backbone 전체 + 분류층 | 10,003,500 |
| Feature Extraction | 분류층 | 3,500 |
| Partial Fine-tuning | 마지막 Block + 분류층 | 2,003,500 |
| Full Fine-tuning | Backbone 전체 + 분류층 | 10,003,500 |

마지막 Block은 Backbone 전체에 이미 포함되어 있으므로 Full Fine-tuning에서는 중복해서 더하지 않는다.

### Python으로 계산 확인

```python
backbone = 10_000_000
last_block = 2_000_000
head = 3_500

trainable = {
    "Scratch": backbone + head,
    "Feature Extraction": head,
    "Partial Fine-tuning": last_block + head,
    "Full Fine-tuning": backbone + head,
}

for strategy, count in trainable.items():
    print(f"{strategy}: {count:,}")

# Scratch: 10,003,500
# Feature Extraction: 3,500
# Partial Fine-tuning: 2,003,500
# Full Fine-tuning: 10,003,500
```

**학습 가능한 파라미터 수와 모델 전체 파라미터 수를 구분해야 한다.** Feature Extraction에서도 Backbone의 가중치는 존재하며 순전파에 사용된다.

동일한 최종 구조를 유지한다면 위 네 모델의 전체 파라미터 수는 모두 10,003,500개다.

## 4. Accuracy와 Macro F1로 예측 성능 평가

### Accuracy: 전체 예측의 정답 비율

$$
\mathrm{Accuracy}
=
\frac{N_{\text{correct}}}{N}
$$

이미지 100개 중 85개를 올바르게 분류했다면 Accuracy는 0.85다.

클래스별 데이터 수가 크게 다르면 전체 정확도만으로 각 결함 유형의 성능을 충분히 파악하기 어렵다.

### Macro F1: 클래스별 F1을 같은 비중으로 평균

클래스 $k$의 Precision과 Recall을 각각 $P_k,R_k$라고 하면 F1은 다음과 같다.

$$
\mathrm{F1}_k
=
\frac{2P_kR_k}{P_k+R_k}
$$

클래스가 $K$개일 때 Macro F1은 클래스별 F1의 단순 평균이다.

$$
\mathrm{Macro\ F1}
=
\frac{1}{K}\sum_{k=1}^{K}\mathrm{F1}_k
$$

7개 결함 유형이라면 각 클래스의 F1을 더해 7로 나눈다. 클래스별 F1에 동일한 비중을 주므로 소수 클래스의 성능도 중요하게 반영된다.

### NumPy로 평균 계산 확인

다음은 평균 계산을 설명하기 위해 가정한 클래스별 F1 값이다.

```python
import numpy as np

class_f1 = np.array([0.80, 0.75, 0.70, 0.65, 0.60, 0.55, 0.50])
macro_f1 = class_f1.mean()

print("클래스 수:", len(class_f1))
print("Macro F1:", f"{macro_f1:.3f}")

# 클래스 수: 7
# Macro F1: 0.650
```

Macro F1도 평균 지표이므로 개별 결함 유형의 성능을 함께 확인한다. 필요에 따라 **클래스별 Precision·Recall·F1과 Confusion Matrix**를 비교한다.

## 5. 가상 실험으로 성능과 학습 시간 함께 비교

아래는 학습자료에서 제시한 **가상의 실험 결과**다. 실제 모델을 실행해 측정한 벤치마크가 아니며, 모든 데이터에서 성립하는 결과를 의미하지 않는다.

| 학습 전략 | Validation Accuracy | Validation Macro F1 | Target Task 학습 시간 |
| --- | ---: | ---: | ---: |
| Scratch | 0.82 | 0.68 | 120분 |
| Feature Extraction | 0.84 | 0.71 | 15분 |
| Partial Fine-tuning | 0.86 | 0.74 | 30분 |
| Full Fine-tuning | 0.87 | 0.76 | 45분 |

이 예시에서 Full Fine-tuning은 가장 높은 Validation 성능을 기록했고, Feature Extraction은 가장 짧은 학습 시간을 기록했다.

### 필요한 성능과 비용에 따라 선택이 달라진다

| 목표 | 위 가상 결과에서 검토할 선택 |
| --- | --- |
| 가장 높은 Macro F1을 우선 | Full Fine-tuning |
| Macro F1 0.70 이상을 만족하며 학습 시간 최소화 | Feature Extraction |
| Macro F1 0.73 이상을 만족하며 학습 시간 최소화 | Partial Fine-tuning |

Full과 Partial의 Macro F1 차이는 0.02이고 학습 시간 차이는 15분이다. 이 성능 차이가 실제로 일관된 개선인지, 추가 비용에 비해 유용한지는 요구사항과 반복 실험 결과를 바탕으로 판단한다.

한 번의 결과만으로 특정 전략이 항상 우수하다고 단정할 수는 없다. 초기화와 Mini-Batch 구성 등 무작위성의 영향도 고려해야 한다.

## 6. 학습 효율·추론 효율·비용 범위 구분

### 학습 시간이 결정되는 요소

학습 가능한 파라미터가 적으면 역전파와 업데이트 부담을 줄일 수 있다. 하지만 실제 학습 시간은 다음 조건에도 영향을 받는다.

- Epoch 수와 Early Stopping 적용 여부
- Batch Size와 학습 데이터의 양
- 모델 구조와 학습할 레이어 범위
- GPU 등 계산 환경

따라서 파라미터 수만으로 실제 학습 시간을 정확히 예측할 수는 없다. 사전학습 가중치로 더 빨리 목표 성능에 도달하면, 전체를 Fine-tuning하더라도 Scratch보다 짧게 학습할 수 있다.

### Target Task 학습 시간과 전체 비용

| 비용 범위 | 포함하는 내용 |
| --- | --- |
| Target Task 학습 시간 | 준비된 사전학습 모델을 새로운 문제에 적용해 학습하는 시간 |
| 전체 학습 비용 | 사전학습에 사용한 데이터·계산 비용까지 포함 |

앞선 가상 표의 시간은 **사전학습 모델을 이미 사용할 수 있다는 전제에서 Target Task를 학습한 시간**이다.

효율을 비교할 때는 사전학습 비용을 포함했는지, 어떤 환경에서 무엇을 측정했는지 명확하게 기록한다.

### 학습 효율과 추론 효율

| 구분 | 비교할 내용 |
| --- | --- |
| 학습 효율 | 학습 시간, 메모리, 역전파·업데이트 계산 부담 |
| 추론 효율 | 새 입력의 예측 시간과 계산 자원 |

Feature Extraction에서도 새로운 이미지의 특징을 계산하기 위해 Backbone의 순전파가 필요하다.

최종 모델 구조와 분류층이 같다면 가중치 값이 달라도 주요 추론 연산 구조는 일반적으로 같다. **학습할 가중치를 줄이는 것과 모델을 작거나 빠르게 만드는 것은 서로 다른 문제다.**

## 7. 공정한 비교와 최종 모델 선택

### 비교 기준과 각 전략에 맞는 설정

| 비교 항목 | 확인할 내용 |
| --- | --- |
| 데이터 | 동일한 Training·Validation 분할을 사용하는가? |
| 모델 | 구조의 차이와 초기 가중치의 차이를 구분하는가? |
| 전처리 | 모델별 입력 요구사항을 충족하며 비교 기준이 일관적인가? |
| 지표 | Accuracy·Macro F1 등을 같은 방식으로 계산하는가? |
| 학습 설정 | 학습률, Epoch, 학습 범위와 튜닝 조건을 기록했는가? |
| 계산 환경 | 시간과 비용을 비교할 수 있는 환경인가? |

공정한 비교가 모든 전략에 같은 Learning Rate를 적용해야 한다는 뜻은 아니다. Scratch와 Fine-tuning은 초기 가중치가 다르므로 적절한 학습 설정도 다를 수 있다.

| 실험 목적 | 조건 관리 방법 |
| --- | --- |
| 초기 가중치의 영향 확인 | 구조·학습 범위 등 다른 조건을 가능한 한 고정 |
| 실제 사용할 최적 전략 선택 | 각 전략에 맞는 설정을 검토하고 튜닝 조건·비용까지 기록 |

### 단계적으로 전략을 검토하는 방법

1. Feature Extraction으로 기존 특징만 활용했을 때의 성능과 비용을 확인한다.
2. 특징 조정이 필요하면 일부 레이어를 Unfreeze하여 Partial Fine-tuning을 비교한다.
3. 필요성과 자원이 충분하다면 Full Fine-tuning 또는 Scratch도 비교한다.
4. 목표 지표, 클래스별 성능, 학습 비용을 함께 고려해 전략을 선택한다.

이 순서는 가능한 접근 방법 중 하나다. Feature Extraction이 요구사항을 충족하면 모든 단계를 진행할 필요는 없다.

### Validation 선택 후 Test 평가

Validation Data로 학습 전략과 설정을 선택한 뒤, 별도로 분리한 Test Data에서 최종 성능을 평가한다.

Test 결과를 반복적으로 확인하며 전략을 바꾸면 해당 데이터에도 간접적으로 맞춰질 수 있다. 따라서 모델 선택과 최종 평가를 구분해야 한다.

## 핵심 정리

- Scratch와 전이학습의 핵심 차이는 사전학습 가중치를 출발점으로 활용하는지다.
- Feature Extraction·Partial·Full Fine-tuning은 Backbone의 업데이트 범위가 다르다.
- 학습 가능한 파라미터 수, 모델 전체 파라미터 수, 추론 연산량을 구분해야 한다.
- Macro F1은 클래스별 F1의 단순 평균으로, Accuracy와 함께 클래스별 성능을 해석하는 데 활용한다.
- 가상 비교 예시에서 가장 높은 성능의 전략과 가장 짧게 학습한 전략은 다르다.
- 학습 비용을 설명할 때 Target Task 비용과 사전학습까지 포함한 전체 비용을 구분한다.
- 실험 목적에 따라 조건을 고정하거나 전략별 설정을 조정하고, 사용한 조건과 비용을 기록한다.
- 최종 전략은 Validation 성능과 요구사항·비용으로 선택하고, Test Data로 최종 평가한다.

## 오늘의 한 문장

> 학습 전략은 같은 평가 기준에서 필요한 성능과 감당할 비용을 함께 확인하며 선택해야 한다.