# 2026-09-07 분류 평가지표

> 이 TIL은 학습자료 중 **Confusion Matrix부터 이후 내용만** 정리했다.

## 학습 목표

- Confusion Matrix의 TP, FP, FN, TN을 구분하고 문제 상황에 맞게 해석할 수 있다.
- Accuracy, Precision, Recall, F1-score의 계산식과 의미 차이를 설명할 수 있다.
- False Positive와 False Negative 중 어떤 오류가 더 중요한지에 따라 평가 지표를 선택할 수 있다.
- Threshold 변화와 Precision-Recall의 관계를 이해할 수 있다.
- ROC Curve와 AUC를 이용해 모델의 전반적인 클래스 구분 능력을 평가할 수 있다.
- 불균형 데이터에서 Accuracy만 사용하는 것이 위험한 이유를 설명할 수 있다.

---

## 1. Confusion Matrix란?

Confusion Matrix(혼동 행렬)는 모델의 예측 결과와 실제 정답을 비교하여 표로 정리한 것이다.

이진 분류에서는 실제 클래스와 예측 클래스를 조합하여 결과를 다음 네 가지로 나눈다.

| 실제값 \ 예측값 | Positive로 예측 | Negative로 예측 |
|---|---|---|
| 실제 Positive | TP(True Positive) | FN(False Negative) |
| 실제 Negative | FP(False Positive) | TN(True Negative) |

### TP, FP, FN, TN

| 구분 | 의미 | 예측 결과 |
|---|---|---|
| TP | 실제 Positive를 Positive로 예측 | 정답 |
| FP | 실제 Negative를 Positive로 예측 | 오답 |
| FN | 실제 Positive를 Negative로 예측 | 오답 |
| TN | 실제 Negative를 Negative로 예측 | 정답 |

`True`와 `False`는 예측이 맞았는지를 나타내고, `Positive`와 `Negative`는 모델이 어느 클래스로 예측했는지를 나타낸다.

```text
True  → 예측이 맞음
False → 예측이 틀림

Positive → 양성으로 예측
Negative → 음성으로 예측
```

### Positive 클래스 정의가 먼저다

TP, FP, FN, TN을 해석하려면 어떤 클래스를 Positive로 정했는지 먼저 확인해야 한다.

예를 들어 스팸 메일을 Positive로 정의하면 다음과 같다.

- TP: 스팸 메일을 스팸으로 올바르게 분류
- FP: 정상 메일을 스팸으로 잘못 분류
- FN: 스팸 메일을 정상 메일로 잘못 분류
- TN: 정상 메일을 정상 메일로 올바르게 분류

같은 예측 결과라도 어떤 클래스를 Positive로 정의하는지에 따라 TP와 TN, FP와 FN의 해석이 바뀐다.

---

## 2. 대출 연체 예측의 Confusion Matrix

대출 연체 고객을 Positive, 정상 상환 고객을 Negative로 정의한다.

- `1`: 연체 고객(Positive)
- `0`: 정상 상환 고객(Negative)

```python
from sklearn.metrics import confusion_matrix

y_true = [1, 0, 1, 1, 0, 0, 1, 0]
y_pred = [1, 0, 0, 1, 1, 0, 1, 0]

tn, fp, fn, tp = confusion_matrix(y_true, y_pred).ravel()

print("TP:", tp)
print("FP:", fp)
print("FN:", fn)
print("TN:", tn)
```

실행 결과:

```text
TP: 3
FP: 1
FN: 1
TN: 3
```

결과는 다음과 같이 해석할 수 있다.

- `TP = 3`: 실제 연체 고객 3명을 연체 고객으로 올바르게 예측했다.
- `FP = 1`: 정상 상환 고객 1명을 연체 고객으로 잘못 예측했다.
- `FN = 1`: 실제 연체 고객 1명을 정상 고객으로 잘못 예측했다.
- `TN = 3`: 정상 상환 고객 3명을 정상 고객으로 올바르게 예측했다.

Confusion Matrix를 보면 단순히 몇 개를 맞혔는지를 넘어, 모델이 **어떤 종류의 실수**를 했는지 알 수 있다.

### `confusion_matrix().ravel()`의 순서

scikit-learn의 이진 Confusion Matrix를 `ravel()`로 펼치면 다음 순서로 반환된다.

```python
tn, fp, fn, tp = confusion_matrix(y_true, y_pred).ravel()
```

순서를 `tp, fp, fn, tn`으로 착각하지 않도록 주의해야 한다.

---

## 3. 연습문제 1: 불량 제품 검사

- `1`: 불량 제품(Positive)
- `0`: 정상 제품(Negative)

```python
from sklearn.metrics import confusion_matrix

y_true = [1, 1, 0, 0, 1, 0, 1, 0, 0, 1]
y_pred = [1, 0, 0, 0, 1, 1, 1, 0, 0, 0]

tn, fp, fn, tp = confusion_matrix(y_true, y_pred).ravel()

print("TP:", tp)
print("FP:", fp)
print("FN:", fn)
print("TN:", tn)
```

실행 결과:

```text
TP: 3
FP: 1
FN: 2
TN: 4
```

모델은 실제 불량 제품 5개 중 3개를 찾아냈고, 불량 제품 2개를 정상으로 놓쳤다. 또한 정상 제품 1개를 불량으로 잘못 판정했다.

---

## 4. 연습문제 2: 놓친 이탈 고객 수

- `1`: 이탈 고객(Positive)
- `0`: 유지 고객(Negative)

실제로 이탈했지만 모델이 유지할 것이라고 예측한 고객은 FN이다.

```python
from sklearn.metrics import confusion_matrix

y_true = [0, 1, 1, 0, 1, 0, 0, 1]
y_pred = [0, 1, 0, 0, 0, 0, 1, 1]

tn, fp, fn, tp = confusion_matrix(y_true, y_pred).ravel()

print("FN:", fn)
```

실행 결과:

```text
FN: 2
```

따라서 모델이 놓친 실제 이탈 고객은 2명이다.

---

## 5. Accuracy, Precision, Recall, F1-score

Confusion Matrix의 TP, FP, FN, TN을 이용해 대표적인 분류 평가 지표를 계산할 수 있다.

### 정확도(Accuracy)

Accuracy는 전체 예측 중 올바르게 예측한 비율이다.

$$
\mathrm{Accuracy}
=\frac{TP+TN}{TP+TN+FP+FN}
$$

Accuracy가 답하는 질문은 다음과 같다.

> 전체 데이터 중 얼마나 많이 맞혔는가?

클래스 비율이 비교적 균형 잡혀 있고 각 오류의 중요도가 비슷할 때 직관적으로 사용할 수 있다.

### 정밀도(Precision)

Precision은 모델이 Positive라고 예측한 데이터 중 실제로 Positive인 비율이다.

$$
\mathrm{Precision}
=\frac{TP}{TP+FP}
$$

Precision이 답하는 질문은 다음과 같다.

> 모델이 양성이라고 판단한 것 중 진짜 양성은 얼마나 되는가?

FP, 즉 실제 음성을 양성으로 잘못 판단하는 비용이 클 때 중요하다.

예를 들어 정상 메일을 스팸으로 분류하면 중요한 메일을 놓칠 수 있으므로 스팸 분류에서는 Precision도 중요하게 볼 수 있다.

### 재현율(Recall)

Recall은 실제 Positive 데이터 중 모델이 Positive로 올바르게 찾아낸 비율이다. 민감도(Sensitivity) 또는 TPR(True Positive Rate)이라고도 한다.

$$
\mathrm{Recall}
=\frac{TP}{TP+FN}
$$

Recall이 답하는 질문은 다음과 같다.

> 실제 양성 중 모델이 얼마나 많이 찾아냈는가?

FN, 즉 실제 양성을 놓치는 비용이 클 때 중요하다.

예를 들어 질병 환자를 정상이라고 놓치는 것이 위험한 질병 진단 문제에서는 Recall을 중요하게 볼 수 있다.

### F1-score

F1-score는 Precision과 Recall의 조화평균이다.

$$
F1
=\frac{2\times\mathrm{Precision}\times\mathrm{Recall}}
{\mathrm{Precision}+\mathrm{Recall}}
$$

Precision과 Recall이 모두 높아야 F1-score도 높아진다. 두 지표의 균형이 중요하거나 클래스가 불균형할 때 사용할 수 있다.

### 네 가지 지표 비교

| 지표 | 답하는 질문 | 중요하게 보는 오류 | 적합한 상황 |
|---|---|---|---|
| Accuracy | 전체 예측 중 얼마나 맞혔는가? | 전체 오류 | 클래스 비율이 균형 잡혀 있을 때 |
| Precision | 양성 예측 중 실제 양성은 얼마나 되는가? | FP | 거짓 양성의 비용이 클 때 |
| Recall | 실제 양성 중 얼마나 찾아냈는가? | FN | 거짓 음성의 비용이 클 때 |
| F1-score | Precision과 Recall의 균형은 어떠한가? | FP와 FN | 두 지표가 모두 중요하거나 데이터가 불균형할 때 |

---

## 6. 이상 거래 탐지 모델 평가

카드 거래에서 `1`은 이상 거래, `0`은 정상 거래를 의미한다.

```python
from sklearn.metrics import accuracy_score
from sklearn.metrics import precision_score
from sklearn.metrics import recall_score
from sklearn.metrics import f1_score

y_true = [0, 0, 0, 1, 0, 1, 0, 0, 1, 0]
y_pred = [0, 0, 0, 1, 0, 0, 0, 1, 1, 0]

print("Accuracy:", accuracy_score(y_true, y_pred))
print("Precision:", precision_score(y_true, y_pred))
print("Recall:", recall_score(y_true, y_pred))
print("F1-score:", f1_score(y_true, y_pred))
```

실행 결과:

```text
Accuracy: 0.8
Precision: 0.6666666666666666
Recall: 0.6666666666666666
F1-score: 0.6666666666666666
```

### 결과 해석

- Accuracy 0.8: 전체 거래의 80%를 올바르게 분류했다.
- Precision 약 0.667: 이상 거래라고 예측한 건 중 약 66.7%가 실제 이상 거래였다.
- Recall 약 0.667: 실제 이상 거래 중 약 66.7%를 찾아냈다.
- F1-score 약 0.667: Precision과 Recall의 균형 수준이 약 66.7%이다.

같은 숫자라도 각 지표가 의미하는 기준 집합이 다르다. Precision의 분모는 **Positive로 예측한 데이터**, Recall의 분모는 **실제 Positive 데이터**이다.

---

## 7. 연습문제 3: 광고 클릭 예측 지표

- `1`: 클릭(Positive)
- `0`: 클릭하지 않음(Negative)

```python
from sklearn.metrics import (
    accuracy_score,
    precision_score,
    recall_score,
    f1_score,
)

y_true = [1, 0, 1, 0, 1, 0, 0, 1, 0, 1]
y_pred = [1, 0, 1, 0, 0, 0, 1, 1, 0, 0]

print("Accuracy:", accuracy_score(y_true, y_pred))
print("Precision:", precision_score(y_true, y_pred))
print("Recall:", recall_score(y_true, y_pred))
print("F1-score:", f1_score(y_true, y_pred))
```

실행 결과:

```text
Accuracy: 0.7
Precision: 0.75
Recall: 0.6
F1-score: 0.667
```

Confusion Matrix는 `TN=4`, `FP=1`, `FN=2`, `TP=3`이다.

직접 계산하면 다음과 같다.

$$
\mathrm{Accuracy}=\frac{3+4}{3+4+1+2}=0.7
$$

$$
\mathrm{Precision}=\frac{3}{3+1}=0.75
$$

$$
\mathrm{Recall}=\frac{3}{3+2}=0.6
$$

$$
F1=\frac{2\times0.75\times0.6}{0.75+0.6}\approx0.667
$$

---

## 8. 연습문제 4: 질병 진단 모델 비교

질병이 있는 환자를 놓치지 않는 것이 중요하므로 FN과 직접 관련된 Recall을 비교해야 한다.

```python
from sklearn.metrics import recall_score

y_true = [1, 1, 1, 1, 0, 0, 0, 0]

pred_a = [1, 1, 1, 0, 0, 0, 0, 0]
pred_b = [1, 1, 0, 0, 0, 0, 0, 0]

recall_a = recall_score(y_true, pred_a)
recall_b = recall_score(y_true, pred_b)

print("모델 A Recall:", recall_a)
print("모델 B Recall:", recall_b)
```

실행 결과:

```text
모델 A Recall: 0.75
모델 B Recall: 0.5
```

- 모델 A는 실제 환자 4명 중 3명을 찾아냈다.
- 모델 B는 실제 환자 4명 중 2명을 찾아냈다.

따라서 다른 조건이 같고 환자를 놓치지 않는 것이 가장 중요하다면 Recall이 더 높은 모델 A가 적합하다.

---

## 9. Precision과 Recall의 Trade-off

분류 모델은 예측 확률이 Threshold 이상이면 Positive, 미만이면 Negative로 분류한다.

Threshold를 변경하면 Positive로 예측하는 데이터의 수가 달라지므로 Precision과 Recall도 달라진다.

### Threshold를 낮추는 경우

조금이라도 의심되는 데이터를 Positive로 분류한다.

```text
Positive 예측 증가
→ 실제 Positive를 더 많이 찾음
→ FN 감소, Recall 상승 가능
→ 실제 Negative도 Positive로 판단할 수 있음
→ FP 증가, Precision 하락 가능
```

### Threshold를 높이는 경우

확실한 데이터만 Positive로 분류한다.

```text
Positive 예측 감소
→ FP 감소, Precision 상승 가능
→ 실제 Positive를 놓칠 수 있음
→ FN 증가, Recall 하락 가능
```

따라서 Precision 또는 Recall 하나만 무조건 높이는 것이 항상 좋은 것은 아니다. 실제 문제에서 FP와 FN 중 어느 실수가 더 큰 비용을 발생시키는지 판단해야 한다.

두 지표가 모두 중요하다면 F1-score를 함께 확인할 수 있다.

---

## 10. ROC Curve

ROC(Receiver Operating Characteristic) Curve는 Threshold를 변화시키면서 TPR과 FPR이 어떻게 변하는지를 나타낸 그래프이다.

### TPR(True Positive Rate)

TPR은 실제 Positive 중 모델이 Positive로 올바르게 찾아낸 비율이며 Recall과 같다.

$$
\mathrm{TPR}=\frac{TP}{TP+FN}
$$

### FPR(False Positive Rate)

FPR은 실제 Negative 중 모델이 Positive로 잘못 예측한 비율이다.

$$
\mathrm{FPR}=\frac{FP}{FP+TN}
$$

ROC Curve에서는 일반적으로 다음과 같이 표현한다.

- 가로축: FPR
- 세로축: TPR

Threshold를 낮추면 Positive로 예측하는 데이터가 많아져 TPR과 FPR이 함께 증가할 수 있다. ROC Curve는 하나의 Threshold 결과가 아니라 여러 Threshold에서의 구분 성능을 보여준다.

---

## 11. AUC

AUC(Area Under the Curve)는 ROC Curve 아래의 면적이다.

- AUC가 1에 가까울수록 Positive와 Negative를 잘 구분한다.
- AUC가 0.5에 가까우면 무작위로 예측하는 것과 비슷한 수준이다.
- 하나의 고정 Threshold에 의존하지 않고 모델의 전반적인 순위 구분 능력을 평가한다.

AUC는 모델이 Positive 데이터에 Negative 데이터보다 더 높은 점수를 부여하는 능력으로 이해할 수 있다.

### 고객 이탈 예측 모델 비교

```python
from sklearn.metrics import roc_auc_score

# 1: 이탈, 0: 유지
y_true = [0, 0, 1, 1, 0, 1, 0, 1]

model_a_prob = [0.1, 0.2, 0.8, 0.9, 0.3, 0.7, 0.4, 0.85]
model_b_prob = [0.3, 0.4, 0.6, 0.7, 0.5, 0.55, 0.45, 0.65]

auc_a = roc_auc_score(y_true, model_a_prob)
auc_b = roc_auc_score(y_true, model_b_prob)

print("모델 A AUC:", auc_a)
print("모델 B AUC:", auc_b)
```

실행 결과:

```text
모델 A AUC: 1.0
모델 B AUC: 1.0
```

두 모델이 출력한 확률값은 서로 다르지만, 두 모델 모두 모든 이탈 고객에게 유지 고객보다 높은 점수를 부여했다. 따라서 클래스의 순서를 완벽하게 구분하여 AUC가 모두 1.0이다.

즉, AUC는 확률이 단순히 0.5 이상인지 평가하는 것이 아니라 Positive와 Negative의 **상대적인 순위**를 얼마나 잘 구분하는지 평가한다.

---

## 12. 연습문제 5: 상품 구매 예측 AUC

```python
from sklearn.metrics import roc_auc_score

y_true = [0, 1, 0, 1, 0, 1, 1, 0]
y_prob = [0.15, 0.75, 0.30, 0.80, 0.40, 0.65, 0.90, 0.20]

auc = roc_auc_score(y_true, y_prob)

print("AUC:", auc)
```

실행 결과:

```text
AUC: 1.0
```

모든 실제 구매 고객이 모든 비구매 고객보다 높은 구매 확률을 받았으므로 두 클래스를 완벽하게 구분했다.

---

## 13. 연습문제 6: 대출 심사 모델 AUC 비교

```python
from sklearn.metrics import roc_auc_score

y_true = [0, 1, 0, 1, 0, 1, 0, 1]

model_a_prob = [0.1, 0.9, 0.2, 0.8, 0.3, 0.7, 0.4, 0.6]
model_b_prob = [0.2, 0.6, 0.5, 0.7, 0.3, 0.4, 0.6, 0.8]

auc_a = roc_auc_score(y_true, model_a_prob)
auc_b = roc_auc_score(y_true, model_b_prob)

print("모델 A AUC:", auc_a)
print("모델 B AUC:", auc_b)
```

실행 결과:

```text
모델 A AUC: 1.0
모델 B AUC: 0.84375
```

모델 A의 AUC가 더 높으므로 여러 Threshold를 고려했을 때 연체 고객과 정상 고객을 전반적으로 더 잘 구분한다.

---

## 14. 데이터 불균형과 지표 선택

데이터 불균형은 특정 클래스의 데이터가 다른 클래스보다 매우 적은 상태이다.

예를 들어 전체 거래 중 사기 거래가 1%이고 정상 거래가 99%라고 가정한다. 모든 거래를 정상이라고만 예측하는 모델도 Accuracy는 99%가 된다.

```text
전체 거래: 10,000건
정상 거래: 9,900건
사기 거래:   100건

모든 거래를 정상으로 예측
→ 맞힌 데이터 9,900건
→ Accuracy = 99%
→ 실제 사기 거래는 한 건도 찾지 못함
→ Recall = 0%
```

따라서 불균형 데이터에서는 Accuracy가 높다는 이유만으로 좋은 모델이라고 판단할 수 없다.

### 상황에 따른 지표 선택

| 상황 | 우선 확인할 지표 | 이유 |
|---|---|---|
| 클래스 비율이 균형 잡힘 | Accuracy | 전체 정답 비율을 직관적으로 확인할 수 있음 |
| Positive 오탐의 비용이 큼 | Precision | FP를 줄이는 것이 중요함 |
| 실제 Positive를 놓치면 위험함 | Recall | FN을 줄이는 것이 중요함 |
| Precision과 Recall이 모두 중요함 | F1-score | 두 지표의 균형을 확인함 |
| Threshold 전반의 구분 능력 비교 | ROC-AUC | 하나의 기준값에 의존하지 않음 |

평가 지표는 가장 높은 숫자가 나오는 것을 선택하는 것이 아니라, **비즈니스 문제에서 어떤 오류가 더 위험한지**를 기준으로 선택해야 한다.

---

## 15. 오늘 배운 내용 정리

### Confusion Matrix

- 실제값과 예측값을 TP, FP, FN, TN으로 나눈다.
- TP와 TN은 올바른 예측이고 FP와 FN은 잘못된 예측이다.
- Positive 클래스를 무엇으로 정의했는지 먼저 확인해야 한다.
- scikit-learn에서 `ravel()`의 반환 순서는 `tn, fp, fn, tp`이다.

### 주요 분류 평가지표

- Accuracy는 전체 예측 중 맞힌 비율이다.
- Precision은 Positive 예측 중 실제 Positive의 비율이다.
- Recall은 실제 Positive 중 찾아낸 비율이다.
- F1-score는 Precision과 Recall의 조화평균이다.

### 오류 비용과 지표 선택

- FP의 비용이 크면 Precision이 중요하다.
- FN의 비용이 크면 Recall이 중요하다.
- 두 오류가 모두 중요하면 F1-score를 함께 확인한다.
- 불균형 데이터에서는 Accuracy만으로 모델을 평가하면 안 된다.

### ROC-AUC

- ROC Curve는 Threshold 변화에 따른 TPR과 FPR을 보여준다.
- AUC는 ROC Curve 아래의 면적이다.
- AUC가 1에 가까울수록 두 클래스를 잘 구분한다.
- AUC는 특정 Threshold가 아니라 전반적인 순위 구분 능력을 평가한다.

### 전체 평가 흐름

```text
Positive 클래스 정의
    ↓
실제값과 예측값으로 Confusion Matrix 확인
    ↓
TP, FP, FN, TN의 오류 유형 해석
    ↓
문제의 오류 비용에 맞는 지표 선택
    ↓
Accuracy·Precision·Recall·F1-score 계산
    ↓
예측 확률이 있다면 ROC-AUC도 확인
```

---

## 16. 느낀 점

오늘은 분류 모델의 성능을 단순히 맞힌 비율 하나로 판단하면 안 된다는 점을 배웠다. Confusion Matrix를 사용하면 모델이 틀린 횟수뿐 아니라 실제 양성을 놓쳤는지, 실제 음성을 양성으로 잘못 판단했는지까지 구분할 수 있었다.

특히 Precision과 Recall의 분모가 다르다는 점이 중요하게 느껴졌다. Precision은 모델이 양성이라고 예측한 대상을 기준으로 보고, Recall은 실제 양성 대상을 기준으로 본다. 따라서 같은 예측 결과라도 문제의 목적에 따라 더 중요한 지표가 달라질 수 있다.

질병 진단이나 고객 이탈처럼 실제 Positive를 놓치는 피해가 크다면 FN을 줄이기 위해 Recall을 중요하게 봐야 한다. 반면 정상 사용자를 잘못 차단하는 상황처럼 FP의 피해가 크다면 Precision을 더 중요하게 봐야 한다. 평가 지표 선택은 단순한 수학 문제가 아니라 실제 서비스의 비용과 연결된 판단이라는 것을 알게 되었다.

또한 AUC가 확률값의 크기 자체보다 Positive와 Negative의 순서를 얼마나 잘 구분하는지 평가한다는 점이 인상적이었다. 두 모델의 확률값이 달라도 클래스의 순위를 똑같이 완벽하게 구분하면 AUC가 모두 1이 될 수 있었다.

앞으로 분류 프로젝트에서는 먼저 Positive 클래스를 명확히 정의하고 Confusion Matrix를 확인한 뒤, FP와 FN 중 어떤 오류가 더 중요한지 판단하여 평가 지표를 선택해야겠다. 클래스가 불균형한 데이터에서는 Accuracy에만 의존하지 않고 Precision, Recall, F1-score와 AUC를 함께 살펴보는 습관을 들여야겠다.
