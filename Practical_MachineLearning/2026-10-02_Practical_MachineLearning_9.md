# 2026-10-02 교차검증 전략과 데이터 누수 방지

## 학습 목표

- 한 번의 Train / Validation 분할만으로 모델을 평가할 때의 한계를 설명할 수 있다.
- K-Fold Cross-Validation의 동작 방식과 K 값에 따른 계산량을 이해한다.
- Fold별 점수의 평균과 표준편차를 함께 해석할 수 있다.
- 데이터 특성에 따라 KFold, StratifiedKFold, TimeSeriesSplit을 선택할 수 있다.
- 전처리와 Resampling 과정에서 발생하는 데이터 누수를 이해한다.
- Pipeline과 교차검증을 연결하여 올바른 검증 과정을 구성할 수 있다.

---

## 1. 한 번의 검증 점수만 믿어도 될까?

고객 이탈 여부를 예측하는 모델을 만들고 Validation 데이터에서 F1-score가 0.85를 기록했다고 가정해보자. 같은 모델을 사용하더라도 데이터를 다시 나누면 점수가 달라질 수 있다.

| 데이터 분할 | F1-score |
|---|---|
| 첫 번째 분할 | 0.85 |
| 두 번째 분할 | 0.78 |
| 세 번째 분할 | 0.72 |

어떤 데이터가 Train과 Validation에 포함되는지에 따라 모델이 학습하는 패턴과 검증의 난이도가 달라지기 때문이다. 특히 데이터가 적으면 일부 관측치의 위치가 결과에 큰 영향을 줄 수 있다.

한 번의 점수만으로는 검증 데이터가 우연히 예측하기 쉬웠는지 판단하기 어렵다. 따라서 여러 Train / Validation 조합에서 모델을 평가하고, 점수의 수준과 변동을 함께 확인해야 한다.

**일반화 성능**은 학습에 사용하지 않은 새로운 데이터에서 모델이 얼마나 잘 예측하는지를 의미한다. 검증의 목적은 학습 데이터에서 높은 점수를 얻는 것이 아니라, 새로운 데이터에서의 성능을 추정하는 것이다.

---

## 2. 교차검증과 K-Fold의 원리

**교차검증(Cross-Validation)**은 데이터를 여러 번 나누어 학습과 검증을 반복하는 평가 방법이다. 대표적인 방법이 **K-Fold Cross-Validation**이다.

K-Fold는 교차검증에 사용할 데이터를 K개의 조각인 Fold로 나눈다. 한 개를 Validation으로 사용하고 나머지 K-1개로 모델을 학습한다. Validation으로 사용할 Fold를 바꾸면서 총 K번 평가한다.

5-Fold의 구성은 다음과 같다.

| 평가 회차 | Train Fold | Validation Fold |
|---|---|---|
| 1회 | 2, 3, 4, 5 | 1 |
| 2회 | 1, 3, 4, 5 | 2 |
| 3회 | 1, 2, 4, 5 | 3 |
| 4회 | 1, 2, 3, 5 | 4 |
| 5회 | 1, 2, 3, 4 | 5 |

각 Fold는 한 번씩 Validation으로 사용된다. 매 회차마다 해당 Train Fold만으로 모델을 새로 학습하며, 앞 회차에서 학습한 모델을 그대로 이어서 사용하지 않는다.

최종 평가용 Test 데이터를 따로 확보했다면, Test를 제외한 개발용 데이터에서 교차검증을 수행한다.

### K 값과 계산량

- `n_splits=5`이면 학습과 검증을 5번 수행한다.
- K가 커지면 각 회차에서 학습에 사용하는 데이터의 비율이 커진다.
- 대신 모델을 더 많이 학습하므로 계산 시간과 비용이 증가한다.
- K를 크게 설정한다고 평가가 무조건 더 좋아지는 것은 아니다.

데이터 크기와 모델의 학습 비용을 고려하여 K를 정해야 한다.

---

## 3. 평균과 표준편차를 함께 확인하기

교차검증에서는 가장 높은 Fold 점수 하나만 선택하지 않고, 전체 Fold의 결과를 확인한다.

예를 들어 Fold별 F1-score가 다음과 같다고 하자.

~~~python
import numpy as np

scores = np.array([0.81, 0.84, 0.79, 0.83, 0.78])

print("평균 F1-score:", round(scores.mean(), 3))
print("표준편차:", round(scores.std(), 3))
~~~

실행 결과:

~~~text
평균 F1-score: 0.81
표준편차: 0.023
~~~

| 값 | 의미 |
|---|---|
| 평균 | 여러 Fold에서 얻은 전반적인 성능 수준 |
| 표준편차 | Fold에 따른 점수의 변동 정도 |
| 개별 Fold 점수 | 특정 분할에서 성능이 크게 떨어지는지 확인하는 근거 |

표준편차가 작으면 해당 분할들에서 점수 차이가 상대적으로 작다는 뜻이다. 표준편차가 크면 데이터 구성에 따라 모델 성능이 많이 달라질 수 있다.

따라서 평균이 높은지와 점수가 안정적인지를 함께 살펴봐야 한다. 다만 Fold별 표준편차가 작다고 실제 서비스에서도 반드시 안정적이라는 뜻은 아니며, 평균과 표준편차만으로 정확한 신뢰구간을 표현할 수도 없다.

---

## 4. 데이터 특성에 맞는 교차검증 전략

모든 데이터에 동일한 분할 방법을 사용하면 안 된다. 검증 데이터의 구조가 실제 예측 상황과 맞아야 한다.

| 방법 | 주로 사용하는 상황 | 핵심 특징 |
|---|---|---|
| KFold | 시간 순서나 그룹 제약이 없는 일반적인 회귀 데이터 | 데이터를 K개로 나누어 번갈아 검증 |
| StratifiedKFold | 일반적인 분류 데이터, 특히 클래스 불균형 데이터 | 각 Fold의 클래스 비율을 가능한 한 비슷하게 유지 |
| TimeSeriesSplit | 시간 순서가 중요한 데이터 | 과거 데이터로 학습하고 이후 데이터로 검증 |

### KFold

KFold는 타깃의 클래스 비율을 고려하지 않고 데이터를 나눈다. 데이터 순서에 의미가 없다면 `shuffle=True`를 사용해 섞은 뒤 분할할 수 있다.

~~~python
from sklearn.model_selection import KFold

kf = KFold(
    n_splits=5,
    shuffle=True,
    random_state=42,
)
~~~

`random_state`를 고정하면 같은 조건에서 분할을 재현할 수 있다. 시간 순서가 중요한 데이터에는 무작위 섞기를 그대로 적용하면 안 된다.

### StratifiedKFold

고객 이탈 데이터에서 이탈 고객이 10%, 유지 고객이 90%라면, 단순 분할에 따라 특정 Fold에 이탈 고객이 지나치게 적게 포함될 수 있다.

StratifiedKFold는 각 Fold에서 타깃 클래스의 비율을 가능한 한 유지하여 이런 차이를 줄인다.

~~~python
from sklearn.model_selection import StratifiedKFold

skf = StratifiedKFold(
    n_splits=5,
    shuffle=True,
    random_state=42,
)
~~~

클래스 비율을 유지하는 것은 소수 클래스를 늘리는 작업과 다르다. StratifiedKFold는 분할 방식이며, 데이터 자체의 불균형을 해소하는 방법은 아니다.

### TimeSeriesSplit

시계열 데이터는 과거 정보로 미래를 예측한다. 데이터를 무작위로 나누면 미래의 관측치로 학습하고 과거의 관측치를 검증하는 상황이 생길 수 있다.

TimeSeriesSplit은 시간 순서대로 정렬된 데이터를 대상으로, 앞쪽 구간을 학습에 사용하고 뒤쪽 구간을 검증에 사용한다. 기본 설정에서는 회차가 진행될수록 학습 구간이 확장된다.

~~~python
from sklearn.model_selection import TimeSeriesSplit

tscv = TimeSeriesSplit(n_splits=3)

# X는 시간 순서대로 정렬되어 있다고 가정한다.
for fold, (train_idx, valid_idx) in enumerate(tscv.split(X), start=1):
    print(f"Fold {fold}")
    print("Train 인덱스:", train_idx)
    print("Validation 인덱스:", valid_idx)
~~~

TimeSeriesSplit이 날짜를 자동으로 정렬하는 것은 아니다. 먼저 시간 순서를 확인하고, 실제 예측에 사용할 수 있는 정보만 Feature에 포함해야 한다.

---

## 5. 데이터 누수(Data Leakage)

데이터 누수는 학습 과정에서 사용해서는 안 되는 정보가 전처리, Feature 생성, 모델 학습 또는 선택 과정에 들어오는 문제이다.

검증 데이터의 정보가 미리 반영되면 모델이 새로운 데이터를 예측하는 상황을 공정하게 평가하기 어렵다. 평가 점수가 실제 성능보다 높게 나타날 수 있다.

대표적인 누수 상황은 다음과 같다.

| 상황 | 누수가 발생하는 이유 |
|---|---|
| 전체 데이터로 Scaling한 뒤 분할 | Validation의 평균과 표준편차가 변환 기준에 반영됨 |
| 전체 데이터로 결측치 대체 기준을 계산 | Validation의 분포가 전처리에 반영됨 |
| 전체 데이터에 Resampling한 뒤 분할 | 검증 대상의 정보가 학습 표본 생성에 사용될 수 있음 |
| Test 점수를 반복 확인하며 모델 수정 | Test가 모델 선택에 영향을 주어 최종 평가의 독립성이 약해짐 |

핵심 기준은 **검증 대상의 정보가 학습 과정에 들어갔는가**이다. 타깃을 직접 사용하지 않은 전처리에서도 누수는 발생할 수 있다.

---

## 6. Scaling은 학습 데이터에서만 fit하기

StandardScaler는 각 Feature의 평균과 표준편차를 계산하여 데이터를 변환한다.

$$
z = \frac{x-\mu_{train}}{\sigma_{train}}
$$

여기서 평균과 표준편차는 학습 데이터에서 계산해야 한다. 검증 데이터에는 학습 데이터에서 얻은 기준을 그대로 적용한다.

- `fit`: 변환에 필요한 기준을 데이터에서 학습한다.
- `transform`: 이미 학습한 기준으로 데이터를 변환한다.
- `fit_transform`: 기준 학습과 변환을 함께 수행한다.

다음 코드는 `X`, `y`가 준비되어 있고, 별도의 Test 데이터는 제외되어 있다고 가정한다.

~~~python
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

X_train, X_valid, y_train, y_valid = train_test_split(
    X,
    y,
    test_size=0.2,
    stratify=y,
    random_state=42,
)

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_valid_scaled = scaler.transform(X_valid)
~~~

검증 데이터에 `fit_transform()`을 다시 적용하면 검증 데이터 자체의 기준으로 변환하게 된다. 학습 데이터와 검증 데이터에 같은 변환 기준을 사용해야 한다.

교차검증에서도 원리는 동일하다. **각 Fold의 Train에서 전처리를 fit하고, 해당 Validation에는 transform만 적용한다.** 교차검증 전에 개발용 데이터 전체를 Scaling하는 것도 피해야 한다.

---

## 7. Resampling과 SMOTE에서의 누수

Resampling은 클래스 불균형을 완화하기 위해 학습 데이터의 클래스별 표본 수를 조정하는 방법이다. SMOTE는 소수 클래스의 관측치와 이웃 관측치를 이용해 새로운 표본을 생성한다.

전체 데이터에 SMOTE를 적용한 뒤 분할하면 Validation에 포함될 관측치의 정보가 합성 표본 생성에 사용될 수 있다. 그 결과 Train과 Validation이 독립적인 평가 관계를 유지하기 어렵다.

올바른 순서는 다음과 같다.

1. 원본 데이터를 Train과 Validation으로 나눈다.
2. Train 데이터에만 필요한 전처리를 학습한다.
3. Train 데이터에만 Resampling을 적용한다.
4. 조정된 Train 데이터로 모델을 학습한다.
5. Resampling하지 않은 Validation 데이터에서 평가한다.

교차검증에서는 매 Fold의 Train에만 Resampling을 적용해야 한다. Validation과 Test의 클래스 분포를 평가를 위해 인위적으로 바꾸지 않는다.

SMOTE는 이웃 사이의 거리를 사용하므로 Feature의 스케일이 크게 다르면 Scaling도 고려해야 한다. 이때 Scaling과 SMOTE 모두 해당 Fold의 Train 안에서 수행해야 한다.

---

## 8. Pipeline으로 전처리와 교차검증 연결하기

**Pipeline**은 전처리와 모델을 하나의 학습 과정으로 연결하는 도구이다. 교차검증에 Pipeline을 전달하면 각 Fold에서 전처리와 모델이 새로 학습된다.

다음 예시는 수치형 Feature로 구성된 이진 분류 데이터 `X`, `y`를 가정한다.

~~~python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import StratifiedKFold, cross_val_score

pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("model", LogisticRegression(max_iter=1000)),
])

cv = StratifiedKFold(
    n_splits=5,
    shuffle=True,
    random_state=42,
)

scores = cross_val_score(
    pipeline,
    X,
    y,
    cv=cv,
    scoring="f1",
)

print("Fold별 F1-score:", scores)
print("평균:", scores.mean())
print("표준편차:", scores.std())
~~~

각 Fold에서는 다음 과정이 자동으로 수행된다.

1. Train Fold에서 StandardScaler를 학습하고 데이터를 변환한다.
2. 변환된 Train Fold에서 LogisticRegression을 학습한다.
3. Validation Fold를 Train에서 학습한 Scaling 기준으로 변환한다.
4. Validation Fold에서 예측하고 F1-score를 계산한다.

`cross_val_score()`의 `scoring="f1"`은 기본적으로 양성 클래스가 1인 이진 분류의 F1-score를 계산한다. 다중 분류라면 평가 목적에 맞게 `f1_macro` 등의 지표를 선택해야 한다.

Pipeline은 내부에 포함된 전처리의 학습 순서를 관리한다. Pipeline 밖에서 이미 전체 데이터로 Scaling하거나 누수가 있는 Feature를 만들었다면 이를 자동으로 해결해주지는 않는다.

### SMOTE를 포함하는 Pipeline

SMOTE처럼 표본 수를 바꾸는 단계를 연결할 때는 `imbalanced-learn`의 Pipeline을 사용한다.

~~~python
from imblearn.pipeline import Pipeline
from imblearn.over_sampling import SMOTE
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import StratifiedKFold, cross_val_score

pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("smote", SMOTE(random_state=42)),
    ("model", LogisticRegression(max_iter=1000)),
])

cv = StratifiedKFold(
    n_splits=5,
    shuffle=True,
    random_state=42,
)

scores = cross_val_score(
    pipeline,
    X,
    y,
    cv=cv,
    scoring="f1",
)

print("평균 F1-score:", scores.mean())
print("표준편차:", scores.std())
~~~

이 구조에서는 각 Train Fold에만 SMOTE가 적용된다. Validation을 예측할 때는 Scaling과 모델 예측만 수행하며, 검증 표본을 합성하지 않는다.

SMOTE의 기본 설정에서는 각 Train Fold에 소수 클래스 표본이 충분히 있어야 한다. 표본 수가 매우 적으면 Fold 수와 `k_neighbors` 설정을 함께 확인해야 한다.

---

## 9. Train / Validation / Test의 역할

| 데이터 | 역할 | 사용 방식 |
|---|---|---|
| Train | 전처리 기준과 모델 파라미터 학습 | 전처리와 모델을 fit |
| Validation | 모델 및 하이퍼파라미터 선택 | 후보 모델을 비교하고 개선 방향 결정 |
| Test | 선택이 끝난 모델의 최종 성능 평가 | 개발 과정의 선택에 사용하지 않고 최종 평가 |

교차검증에서는 개발용 데이터 안에서 Train과 Validation의 역할을 바꾸어가며 모델을 비교한다. Test 데이터는 별도로 유지한다.

일반적인 검증 흐름은 다음과 같다.

1. 데이터의 시간 순서와 클래스 분포를 확인한다.
2. 실제 예측 상황에 맞게 최종 Test 데이터를 분리한다.
3. 나머지 개발용 데이터에 적절한 교차검증 전략을 적용한다.
4. Pipeline 안에서 전처리, 필요한 Resampling, 모델 학습을 수행한다.
5. Fold별 점수와 평균, 표준편차를 비교하여 모델을 선택한다.
6. 선택한 설정으로 개발용 데이터 전체에 Pipeline을 다시 학습한다.
7. 분리해 둔 Test 데이터에서 최종 성능을 평가한다.

Test 점수를 보고 반복적으로 모델을 수정하면 Test가 사실상 Validation 역할을 하게 된다. 검증의 신뢰성을 유지하려면 모델 선택과 최종 평가를 구분해야 한다.

---

## 10. 핵심 정리

- 한 번의 Train / Validation 분할 결과는 데이터 구성에 따라 달라질 수 있다.
- 교차검증은 여러 학습·검증 조합에서 모델을 평가하여 일반화 성능을 추정하는 방법이다.
- K-Fold는 K개의 Fold를 만들고 각 Fold를 한 번씩 Validation으로 사용한다.
- K가 커지면 학습 횟수가 늘어나 계산 비용이 증가한다.
- 평균 점수는 전반적인 성능을, 표준편차는 Fold별 점수의 변동을 나타낸다.
- 일반적인 회귀에는 KFold, 분류에는 StratifiedKFold, 시간 순서가 중요한 데이터에는 TimeSeriesSplit을 고려한다.
- 전처리 기준은 각 Fold의 Train에서만 학습해야 한다.
- Resampling은 Train에만 적용하고 Validation과 Test에는 적용하지 않는다.
- Pipeline을 교차검증에 전달하면 Fold별 전처리와 모델 학습을 올바른 순서로 연결할 수 있다.
- SMOTE를 포함할 때는 imbalanced-learn의 Pipeline을 사용한다.
- 최종 Test 데이터는 모델 선택에 사용하지 않고 최종 평가를 위해 유지한다.

---

## 오늘의 한 문장

> 신뢰할 수 있는 모델 평가는 여러 Fold의 성능을 비교하는 것에서 시작하고, 검증 데이터의 정보가 학습 과정에 들어가지 않도록 전처리와 분할 순서를 지키는 것으로 완성된다.