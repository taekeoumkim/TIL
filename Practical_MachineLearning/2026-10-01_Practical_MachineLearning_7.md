# 2026-10-01 TIL: 규제 - L1(Lasso), L2(Ridge), ElasticNet

> 학습일: 2026-10-01  
> 주제: 선형 모델의 규제 원리와 규제 강도

> 참고: 업로드된 PDF의 실제 본문은 **규제의 기본 원리와 `alpha`의 역할**까지 수록되어 있으며, Ridge·Lasso·ElasticNet의 세부 단원은 목차와 개요만 포함되어 있다. 아래 내용은 PDF에 실제로 제시된 범위를 기준으로 정리했다.

## 학습 목표

- 규제가 필요한 이유를 과적합과 연결하여 설명할 수 있다.
- 선형 모델에서 규제가 계수를 제한하는 원리를 이해할 수 있다.
- 예측 오차와 Penalty를 함께 고려하는 손실함수의 구조를 설명할 수 있다.
- `alpha`에 따른 규제 강도와 과적합·과소적합의 관계를 이해할 수 있다.
- Ridge, Lasso, ElasticNet의 핵심적인 차이를 구분할 수 있다.

---

## 1. 규제란 무엇인가?

### 1.1 과적합과 일반화 성능

모델이 Train 데이터에 지나치게 맞춰지면 학습 데이터에서는 높은 성능을 보이지만 새로운 데이터에서는 성능이 낮아질 수 있다.

```text
Train R² = 0.96
Validation R² = 0.72
```

Train과 Validation 성능 차이가 크다면 모델이 Train의 세부적인 특징까지 과도하게 학습한 과적합을 의심할 수 있다.

**규제(Regularization)**는 모델의 복잡도를 제한하여 과적합을 완화하고 새로운 데이터에 대한 일반화 성능을 개선하기 위한 방법이다.

### 1.2 모델별 규제 방법

모델의 구조에 따라 복잡도를 제한하는 방법이 다르다.

| 모델 | 복잡도를 제한하는 대표 방법 |
|---|---|
| Decision Tree | `max_depth` 등으로 트리 구조 제한 |
| Neural Network | Dropout, Early Stopping 등 적용 |
| Linear Model | 학습되는 계수의 크기 제한 |

이번 학습에서는 선형 모델의 계수 크기에 Penalty를 부여하는 규제를 다룬다.

---

## 2. 선형 모델에서 규제가 동작하는 방식

### 2.1 Linear Regression의 예측식

Linear Regression은 각 Feature에 적용할 계수와 절편을 학습한다.

$$
\hat{y}=\beta_0+\beta_1x_1+\beta_2x_2+\cdots+\beta_px_p
$$

- $\beta_0$: 절편
- $\beta_1,\beta_2,\ldots,\beta_p$: 각 Feature의 계수
- $x_1,x_2,\ldots,x_p$: 입력 Feature
- $\hat{y}$: 예측값

일반적인 Linear Regression은 실제값과 예측값의 오차를 작게 만드는 방향으로 계수를 학습한다.

```text
일반 Linear Regression
    ↓
예측 오차 최소화
```

계수에 별도의 제한이 없다면 데이터에 따라 일부 계수가 지나치게 크게 학습될 수 있다.

### 2.2 계수 크기에 Penalty 추가하기

규제가 적용된 선형 모델은 예측 오차뿐 아니라 계수의 크기도 함께 고려한다.

$$
Regularized\ Loss
=Prediction\ Error+\alpha\times Penalty
$$

- **Prediction Error**: 실제값과 예측값의 차이
- **Penalty**: 계수가 지나치게 커지는 것을 제한하기 위해 추가하는 벌점
- **`alpha`**: Penalty의 영향을 결정하는 규제 강도

```text
예측 오차를 줄이는 목표
        +
계수를 작게 유지하는 목표
        ↓
과도하게 복잡한 모델 제한
```

### 2.3 계수 축소

규제를 적용하여 계수의 크기를 줄이는 것을 계수 축소라고 한다.

PDF의 예시에서는 일반 Linear Regression과 Ridge의 계수가 다음과 같이 달라졌다.

```text
Linear Regression: [2.1, 3.4, 48.7]
Ridge            : [1.9, 3.0, 31.2]
```

Ridge를 적용하자 세 번째 계수가 48.7에서 31.2로 줄어드는 등 계수의 크기가 전반적으로 작아졌다. 모델은 예측 오차를 줄이는 동시에 계수가 지나치게 커지지 않도록 학습한 것이다.

```python
from sklearn.linear_model import LinearRegression, Ridge

linear_model = LinearRegression()
linear_model.fit(X_train, y_train)

ridge_model = Ridge(alpha=10)
ridge_model.fit(X_train, y_train)

print("Linear Regression:", linear_model.coef_)
print("Ridge:", ridge_model.coef_)
```

---

## 3. `alpha`와 규제 강도

`alpha`는 규제 강도를 결정하는 하이퍼파라미터다.

| `alpha` | Penalty의 영향 | 규제 강도 | 일반적인 결과 |
|---|---|---|---|
| 작음 | 작음 | 약함 | 계수를 비교적 자유롭게 학습 |
| 큼 | 큼 | 강함 | 계수의 크기를 더 강하게 제한 |

### 3.1 규제가 너무 약한 경우

- 계수에 대한 제한이 충분하지 않다.
- Train 데이터에 지나치게 맞춰질 수 있다.
- 과적합이 충분히 완화되지 않을 수 있다.

### 3.2 규제가 적절한 경우

- 불필요하게 큰 계수를 제한한다.
- Train 성능과 Validation 성능의 균형을 맞출 수 있다.
- 일반화 성능이 개선될 가능성이 있다.

### 3.3 규제가 너무 강한 경우

- 필요한 계수까지 과도하게 제한될 수 있다.
- 데이터의 중요한 패턴을 충분히 학습하지 못할 수 있다.
- 과소적합이 발생할 수 있다.

```text
규제 너무 약함
    ↓
과적합을 충분히 완화하지 못할 수 있음

적절한 규제
    ↓
계수 크기와 예측 성능의 균형

규제 너무 강함
    ↓
필요한 패턴까지 제한하여 과소적합 가능
```

따라서 `alpha`는 무조건 크게 설정하는 값이 아니다. 여러 값을 비교하고 Train과 Validation 성능을 확인하여 적절한 규제 강도를 선택해야 한다.

---

## 4. Ridge, Lasso, ElasticNet 개요

계수에 어떤 방식으로 Penalty를 부여하는지에 따라 대표적으로 세 가지 규제 모델을 사용할 수 있다.

| 모델 | 규제 종류 | PDF 개요에서 제시된 핵심 특징 |
|---|---|---|
| Ridge | L2 | 계수를 전반적으로 작게 만듦 |
| Lasso | L1 | 일부 계수를 0으로 만들 수 있음 |
| ElasticNet | L1 + L2 | 두 규제를 함께 사용 |

### 4.1 Ridge

Ridge는 L2 규제를 사용하는 선형 모델이다. PDF에 포함된 예시처럼 일반 Linear Regression보다 계수의 크기를 전반적으로 줄이는 방향으로 학습한다.

```python
from sklearn.linear_model import Ridge

model = Ridge(alpha=10)
model.fit(X_train, y_train)
```

### 4.2 Lasso

Lasso는 L1 규제를 사용한다. PDF 개요에서는 일부 계수를 0으로 만들 수 있어 변수 선택과 연결된다는 점을 제시한다.

### 4.3 ElasticNet

ElasticNet은 L1과 L2 규제를 함께 사용한다. PDF 개요에서는 다음 두 하이퍼파라미터를 다룬다고 안내한다.

- `alpha`: 전체 규제 강도
- `l1_ratio`: L1과 L2를 어느 비율로 사용할지 결정

업로드된 PDF에는 Lasso와 ElasticNet의 상세 수식, 코드 및 계수 변화 과정은 포함되어 있지 않으므로 여기서는 개요 수준으로 구분한다.

---

## 5. 규제 모델을 선택하는 관점

PDF의 학습 흐름은 다음 과정을 제시한다.

```text
규제의 원리 이해
    ↓
Ridge의 계수 축소 확인
    ↓
Lasso의 계수 축소와 변수 선택 확인
    ↓
ElasticNet의 L1 + L2 규제 확인
    ↓
규제 강도에 따른 계수 변화 비교
    ↓
Validation 성능으로 규제 모델 비교
```

규제 모델은 계수가 얼마나 작아졌는지만으로 선택하면 안 된다. 핵심 목표는 계수를 가장 작게 만드는 것이 아니라 **새로운 데이터에서 좋은 성능을 내는 것**이다.

따라서 다음 항목을 함께 확인해야 한다.

1. 규제 적용 전후 Train 성능
2. 규제 적용 전후 Validation 성능
3. Train과 Validation 성능 차이
4. `alpha` 변화에 따른 계수 크기
5. 규제가 너무 강해 과소적합이 발생하지 않았는지
6. 모델이 필요한 Feature의 정보를 충분히 유지하는지

### 모델 선택 흐름

```text
규제가 없는 모델로 Baseline 확인
    ↓
여러 규제 모델과 alpha 후보 설정
    ↓
각 후보를 Train 데이터로 학습
    ↓
계수와 Train·Validation 성능 비교
    ↓
일반화 성능이 좋은 모델 선택
    ↓
선택에 사용하지 않은 Test 데이터로 최종 평가
```

---

## 핵심 용어 정리

| 용어 | 의미 |
|---|---|
| Regularization | 모델의 복잡도를 제한하여 과적합을 완화하는 방법 |
| Penalty | 계수가 지나치게 커지는 것을 제한하기 위해 손실에 추가하는 벌점 |
| Coefficient Shrinkage | 규제를 적용하여 계수의 크기를 줄이는 것 |
| `alpha` | Penalty의 영향력을 결정하는 규제 강도 |
| Ridge | 계수를 전반적으로 축소하는 L2 규제 모델 |
| Lasso | 일부 계수를 0으로 만들 수 있는 L1 규제 모델 |
| ElasticNet | L1과 L2 규제를 함께 사용하는 모델 |
| `l1_ratio` | ElasticNet에서 L1과 L2 규제의 비율을 조절하는 값 |

## 핵심 정리

1. 규제는 모델의 복잡도를 제한하여 과적합을 완화하고 일반화 성능을 개선하기 위한 방법이다.
2. 선형 모델의 규제는 예측 오차와 함께 계수 크기에 대한 Penalty를 고려한다.
3. 규제로 계수 크기가 작아지는 것을 계수 축소라고 한다.
4. `alpha`가 커질수록 Penalty의 영향이 커지고 규제가 강해진다.
5. 규제가 너무 약하면 과적합이 남을 수 있고, 너무 강하면 과소적합이 발생할 수 있다.
6. Ridge는 계수를 전반적으로 축소하고, Lasso는 일부 계수를 0으로 만들 수 있으며, ElasticNet은 L1과 L2를 함께 사용한다.
7. 규제 강도와 모델 종류는 Train 성능이 아니라 Validation 일반화 성능을 중심으로 선택해야 한다.

## 오늘의 한 문장

> 규제의 목적은 계수를 무조건 작게 만드는 것이 아니라, Train 데이터에 과도하게 맞춰지는 것을 막아 새로운 데이터에서도 안정적인 성능을 얻는 것이다.