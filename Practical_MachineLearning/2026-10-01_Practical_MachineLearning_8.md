# 2026-10-02 TIL: 데이터 불균형과 평가지표 왜곡

> 학습일: 2026-10-02  
> 주제: 불균형 데이터에서 Accuracy의 한계와 분류 평가지표 해석

## 학습 목표

- 데이터 불균형의 의미와 Accuracy만으로 평가할 때의 문제를 설명할 수 있다.
- Confusion Matrix의 TP, FP, FN, TN을 구분할 수 있다.
- Precision, Recall, F1-score를 계산하고 해석할 수 있다.
- FP와 FN의 비용에 따라 적절한 평가지표를 선택할 수 있다.
- Threshold 변화가 Precision과 Recall에 미치는 영향을 설명할 수 있다.

---

## 1. 데이터 불균형과 Accuracy의 한계

**불균형 데이터(Imbalanced Data)**는 클래스별 데이터 수가 크게 차이 나는 데이터다. 예를 들어 신용카드 거래 10,000건 가운데 정상 거래가 9,900건이고 사기 거래가 100건이라면 정상은 99%, 사기는 1%를 차지한다.

모델이 모든 거래를 정상이라고 예측해도 Accuracy는 다음과 같이 99%가 된다.

$$
Accuracy=\frac{올바르게\ 예측한\ 데이터\ 수}{전체\ 데이터\ 수}
=\frac{9,900}{10,000}=0.99
$$

수치만 보면 성능이 좋아 보이지만, 이 모델은 실제로 찾아야 하는 사기 거래 100건을 단 한 건도 발견하지 못했다.

```text
클래스 불균형
    ↓
다수 클래스만 예측해도 높은 Accuracy
    ↓
소수 클래스에 대한 실제 성능이 가려질 수 있음
```

따라서 불균형 데이터에서는 Accuracy만 보지 않고, 모델이 **어떤 클래스를 맞혔고 어떤 클래스를 놓쳤는지** 함께 확인해야 한다.

---

## 2. Confusion Matrix

Confusion Matrix는 실제 클래스와 예측 클래스를 비교하여 결과를 네 가지로 나눈다. 사기 거래를 Positive, 정상 거래를 Negative로 정의하면 다음과 같다.

| 구분 | 의미 | 사기 탐지 예시 |
|---|---|---|
| TP (True Positive) | 실제 Positive를 Positive로 예측 | 실제 사기를 사기로 예측 |
| FP (False Positive) | 실제 Negative를 Positive로 잘못 예측 | 실제 정상을 사기로 예측 |
| FN (False Negative) | 실제 Positive를 Negative로 잘못 예측 | 실제 사기를 정상으로 예측 |
| TN (True Negative) | 실제 Negative를 Negative로 예측 | 실제 정상을 정상으로 예측 |

Confusion Matrix의 구조는 다음과 같다.

|  | 실제 Positive | 실제 Negative |
|---|---:|---:|
| 예측 Positive | TP | FP |
| 예측 Negative | FN | TN |

불균형 데이터에서는 전체 정답 수뿐 아니라 TP, FP, FN, TN이 각각 얼마나 발생했는지 확인하는 것이 중요하다.

---

## 3. Precision, Recall, F1-score

PDF의 예시 모델이 다음 결과를 냈다고 가정한다.

```text
TP = 70, FP = 30, FN = 30, TN = 9,870
```

### 3.1 Precision

**Precision(정밀도)**은 모델이 Positive라고 예측한 데이터 중 실제로 Positive였던 비율이다.

$$
Precision=\frac{TP}{TP+FP}
=\frac{70}{70+30}=0.70
$$

Precision이 중요하다는 것은 **FP를 줄이는 것이 중요하다**는 뜻이다. 정상 거래를 사기로 잘못 판단할 때 거래 차단, 고객 확인 절차, 고객 불편 등의 비용이 크다면 Precision을 중요하게 볼 수 있다.

### 3.2 Recall

**Recall(재현율)**은 실제 Positive 데이터 중 모델이 찾아낸 비율이다.

$$
Recall=\frac{TP}{TP+FN}
=\frac{70}{70+30}=0.70
$$

Recall이 중요하다는 것은 **FN을 줄이는 것이 중요하다**는 뜻이다. 사기 거래를 정상으로 잘못 판단했을 때 큰 금전적 손실이 발생한다면 실제 사기를 최대한 놓치지 않도록 Recall을 중요하게 봐야 한다.

### 3.3 F1-score

**F1-score**는 Precision과 Recall의 조화평균으로, 두 지표를 함께 고려하고 싶을 때 사용한다.

$$
F1=2\times\frac{Precision\times Recall}{Precision+Recall}
$$

조화평균은 두 값 중 하나가 매우 낮을 때 함께 낮아지는 특성이 있어, Precision과 Recall 간 균형을 평가하는 데 적합하다.

### 지표 선택 기준

| 지표 | 평가 관점 | 중요하게 볼 수 있는 상황 |
|---|---|---|
| Accuracy | 전체 데이터 중 맞힌 비율 | 클래스 비율이 비교적 균형적인 경우 |
| Precision | Positive 예측의 정확성 | FP의 비용이 큰 경우 |
| Recall | 실제 Positive 탐지율 | FN의 비용이 큰 경우 |
| F1-score | Precision과 Recall의 균형 | 두 오류를 함께 고려해야 하는 경우 |

어떤 지표가 항상 가장 좋은 것은 아니다. 서비스에서 FP와 FN 중 어떤 오류가 더 큰 손실을 만드는지에 따라 평가 기준을 선택해야 한다.

---

## 4. Python으로 분류 지표 확인하기

`scikit-learn`의 metric 함수를 사용하면 분류 지표를 계산할 수 있다.

```python
from sklearn.metrics import (
    accuracy_score,
    precision_score,
    recall_score,
    f1_score,
)

print("Accuracy:", accuracy_score(y_valid, y_pred))
print("Precision:", precision_score(y_valid, y_pred))
print("Recall:", recall_score(y_valid, y_pred))
print("F1-score:", f1_score(y_valid, y_pred))
```

지표를 해석할 때는 어떤 클래스를 Positive로 지정했는지 먼저 확인해야 한다. 동일한 예측 결과라도 Positive 클래스의 정의에 따라 Precision과 Recall의 의미가 달라지기 때문이다.

---

## 5. Threshold와 Precision-Recall Trade-off

분류 모델은 각 데이터가 Positive일 확률을 출력하고, 이를 **Threshold(임계값)**와 비교하여 최종 클래스를 결정할 수 있다.

예측 확률이 다음과 같다고 가정한다.

```text
거래 A: 0.82
거래 B: 0.64
거래 C: 0.47
거래 D: 0.31
```

Threshold가 0.5라면 A와 B를 사기로 분류한다. Threshold를 0.3으로 낮추면 A, B, C, D를 모두 사기로 분류한다.

| Threshold 변화 | Positive 예측 수 | 일반적인 영향 |
|---|---:|---|
| 낮춤 | 증가 | Recall은 높아지는 방향, Precision은 낮아질 수 있음 |
| 높임 | 감소 | Precision은 높아지는 방향, Recall은 낮아질 수 있음 |

Threshold를 낮추면 더 많은 데이터를 Positive로 판단하므로 실제 Positive를 더 많이 찾을 가능성이 커진다. 하지만 실제 Negative까지 Positive로 예측하는 FP도 늘 수 있다.

반대로 Threshold를 높이면 Positive 판정이 줄어 FP를 줄일 가능성이 있지만, 실제 Positive를 놓치는 FN이 늘 수 있다.

```text
Threshold ↓ → Positive 예측 증가 → Recall ↑ 방향 / Precision ↓ 가능
Threshold ↑ → Positive 예측 감소 → Precision ↑ 방향 / Recall ↓ 가능
```

따라서 기본값 0.5를 항상 정답처럼 사용하면 안 된다. 비즈니스 목적과 FP·FN의 비용을 기준으로 적절한 Threshold를 판단해야 한다.

---

## 6. 불균형 데이터 평가 흐름

불균형 분류 문제는 다음 흐름으로 접근할 수 있다.

1. 클래스별 데이터 수와 비율을 확인한다.
2. 어떤 클래스를 Positive로 볼지 정의한다.
3. Accuracy와 함께 Confusion Matrix를 확인한다.
4. FP와 FN이 각각 서비스에 미치는 비용을 비교한다.
5. 문제 상황에 맞게 Precision, Recall, F1-score를 선택한다.
6. 필요하다면 Threshold를 조정하고 지표 변화를 비교한다.
7. 같은 기준으로 불균형 처리 전후 성능을 비교한다.

PDF는 이후 적용할 불균형 처리 방법으로 다음 세 가지를 예고한다.

| 기법 | PDF에서 제시한 개요 |
|---|---|
| `class_weight` | 소수 클래스의 오류에 더 큰 가중치 부여 |
| Undersampling | 다수 클래스 데이터 감소 |
| SMOTE | 소수 클래스의 합성 데이터 생성 |

특정 방법이 항상 가장 좋은 것은 아니다. 데이터 특성과 문제 상황을 고려하고, 동일한 평가 기준으로 처리 전후의 Precision, Recall, F1-score 변화를 비교해야 한다.

---

## 핵심 정리

- 불균형 데이터에서는 다수 클래스만 예측해도 Accuracy가 높게 나올 수 있다.
- Confusion Matrix를 통해 TP, FP, FN, TN을 구분해야 실제 오류의 종류를 알 수 있다.
- FP의 비용이 크다면 Precision, FN의 비용이 크다면 Recall을 중요하게 본다.
- F1-score는 Precision과 Recall을 함께 고려하는 조화평균이다.
- Threshold를 낮추면 일반적으로 Recall이 높아지는 방향으로, 높이면 Precision이 높아지는 방향으로 움직인다.
- 평가지표와 Threshold는 고정된 정답이 아니라 서비스 목적과 오류 비용에 따라 결정해야 한다.
- 불균형 처리 기법은 적용 전후의 성능을 같은 Validation 기준에서 비교하여 선택해야 한다.

## 오늘의 한 줄

> 불균형 데이터에서는 높은 Accuracy보다 **어떤 오류를 얼마나 만들었는지**가 더 중요하며, 서비스가 감당하기 어려운 오류를 기준으로 지표와 Threshold를 선택해야 한다.