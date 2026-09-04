# 2026-09-03 로지스틱 회귀와 분류 기초

## 학습 목표

- 분류 문제와 회귀 문제의 차이를 설명할 수 있다.
- 일반화 선형 모델(Generalized Linear Model, GLM)의 기본 개념을 이해할 수 있다.
- 로지스틱 회귀가 선형결합을 확률로 변환하여 분류하는 과정을 설명할 수 있다.
- Logit, Sigmoid, Softmax의 역할과 관계를 이해할 수 있다.
- 이항 로지스틱 회귀와 다항 로지스틱 회귀를 구분할 수 있다.
- 원-핫 인코딩을 이용해 순서가 없는 범주형 데이터를 숫자로 표현할 수 있다.

---

## 1. 회귀와 분류

선형회귀는 집값, 매출, 온도처럼 연속적인 숫자를 예측하는 대표적인 회귀 모델이다. 그러나 현실에는 고객의 이탈 여부나 이메일의 스팸 여부처럼 정해진 범주 중 하나를 선택해야 하는 문제도 많다.

이와 같이 데이터가 어떤 범주에 속하는지 예측하는 문제를 **분류(Classification)**라고 한다.

| 구분 | 회귀(Regression) | 분류(Classification) |
|---|---|---|
| 예측 대상 | 연속적인 숫자 | 정해진 범주 |
| 예시 | 집값, 매출, 온도 | 이탈/유지, 스팸/정상, 상품 종류 |
| 대표 모델 | Linear Regression | Logistic Regression |

선형회귀식은 모든 실수값을 출력할 수 있다.

$$
z=w^Tx+b
$$

예측 결과가 `-0.3`이나 `1.7`이라면 이를 그대로 확률로 해석할 수 없다. 확률은 반드시 0과 1 사이에 있어야 하기 때문이다.

로지스틱 회귀는 선형식의 결과를 확률로 변환한 뒤, 그 확률을 기준으로 클래스를 결정한다.

```text
입력 데이터 x
    ↓
선형결합 z = wᵀx + b
    ↓
Sigmoid 또는 Softmax
    ↓
각 클래스에 속할 확률
    ↓
최종 클래스 결정
```

---

## 2. 일반화 선형 모델(GLM)

일반화 선형 모델(Generalized Linear Model, GLM)은 선형 모델을 다양한 형태의 정답 데이터에 적용할 수 있도록 일반화한 모델의 모음이다.

선형회귀는 선형결합의 결과를 그대로 예측값으로 사용한다. 반면 GLM은 선형결합 결과를 **링크 함수(Link Function)**와 연결하여 정답의 특성에 맞는 형태로 표현한다.

```text
선형결합 wᵀx + b
    ↓ 링크 함수와 연결
정답 데이터에 적합한 값
```

로지스틱 회귀는 GLM의 대표적인 예이다. 이진 분류에서는 정답이 0 또는 1이므로 선형결합 결과를 0과 1 사이의 확률로 변환하여 사용한다.

### 로지스틱 회귀의 핵심

- 입력 Feature를 Weight와 Bias로 선형결합한다.
- 선형결합 결과를 확률로 변환한다.
- 계산된 확률을 기준으로 클래스를 선택한다.
- 이름에는 회귀가 있지만 실제 용도는 분류이다.

로지스틱 회귀라는 이름은 입력과 정답의 관계를 선형식으로 모델링하는 방식이 회귀와 비슷하기 때문에 붙었다. 최종 출력은 연속값이 아니라 확률을 이용한 클래스이다.

---

## 3. 고객 이탈 여부 예측하기

최근 접속 일수를 이용하여 고객의 서비스 이탈 여부를 예측한다.

- `0`: 서비스 유지
- `1`: 서비스 이탈

```python
import numpy as np
from sklearn.linear_model import LogisticRegression

X = np.array([
    [1],
    [3],
    [5],
    [10],
    [15],
    [20],
])

y = np.array([0, 0, 0, 1, 1, 1])

model = LogisticRegression()
model.fit(X, y)

new_customer = np.array([[12]])

probability = model.predict_proba(new_customer)
prediction = model.predict(new_customer)

print("유지 확률:", round(probability[0][0], 3))
print("이탈 확률:", round(probability[0][1], 3))
print("예측 클래스:", prediction[0])
```

실행 결과:

```text
유지 확률: 0.032
이탈 확률: 0.968
예측 클래스: 1
```

### 코드 해석

- `fit(X, y)`: 입력 데이터와 정답을 사용하여 모델을 학습한다.
- `predict_proba()`: 각 클래스에 속할 확률을 반환한다.
- `probability[0][0]`: 첫 번째 데이터가 클래스 0일 확률이다.
- `probability[0][1]`: 첫 번째 데이터가 클래스 1일 확률이다.
- `predict()`: 확률을 기준으로 최종 클래스를 반환한다.

이 고객의 이탈 확률이 유지 확률보다 높으므로 모델은 이탈 클래스인 `1`을 예측한다.

### 연습문제 1: 상품 구매 여부 예측

사이트 방문 횟수로 고객의 상품 구매 여부를 예측한다.

- `0`: 구매하지 않음
- `1`: 구매함

```python
import numpy as np
from sklearn.linear_model import LogisticRegression

X = np.array([[1], [2], [3], [5], [7], [9]])
y = np.array([0, 0, 0, 1, 1, 1])

model = LogisticRegression()
model.fit(X, y)

customer = np.array([[6]])
probability = model.predict_proba(customer)
prediction = model.predict(customer)

print("비구매 확률:", round(probability[0][0], 3))
print("구매 확률:", round(probability[0][1], 3))
print("예측 클래스:", prediction[0])
```

실행 결과:

```text
비구매 확률: 0.134
구매 확률: 0.866
예측 클래스: 1
```

사이트를 6회 방문한 고객은 구매 확률이 약 86.6%로 계산되어 구매 클래스인 `1`로 분류된다.

---

## 4. Logit

Logit은 0과 1 사이의 확률을 음의 무한대부터 양의 무한대까지의 실수로 변환하는 함수이다. 확률 $p$에 대한 Odds에 로그를 취한 값이므로 **Log-Odds**라고도 한다.

### Odds

어떤 사건이 발생할 확률을 $p$라고 하면 Odds는 다음과 같다.

$$
\mathrm{Odds}=\frac{p}{1-p}
$$

Odds는 사건이 발생할 가능성과 발생하지 않을 가능성의 비율이다.

예를 들어 사건 발생 확률이 0.8이면 다음과 같다.

$$
\mathrm{Odds}=\frac{0.8}{1-0.8}=4
$$

이는 사건이 발생할 가능성이 발생하지 않을 가능성보다 4배라는 뜻이다.

### Logit 함수

$$
\mathrm{logit}(p)=\log\left(\frac{p}{1-p}\right)
$$

로지스틱 회귀는 확률 자체가 아니라 확률의 Logit을 선형식으로 모델링한다.

$$
\log\left(\frac{p}{1-p}\right)=w^Tx+b
$$

즉, 입력의 선형결합 $w^Tx+b$를 Logit으로 보고, 여기에 Logit의 역함수인 Sigmoid를 적용하여 다시 확률을 얻는다.

---

## 5. Sigmoid 함수

Sigmoid는 모든 실수 입력을 0과 1 사이의 값으로 변환하는 함수이다.

$$
\sigma(z)=\frac{1}{1+e^{-z}}
$$

### 입력에 따른 특징

| 입력 $z$ | Sigmoid 출력 | 의미 |
|---:|---:|---|
| 매우 작은 음수 | 0에 가까움 | 양성 클래스일 확률이 낮음 |
| 0 | 0.5 | 두 클래스의 경계 |
| 매우 큰 양수 | 1에 가까움 | 양성 클래스일 확률이 높음 |

### 배달 주문 취소 확률 계산

```python
import numpy as np

def sigmoid(z):
    return 1 / (1 + np.exp(-z))

logit = 1.5
probability = sigmoid(logit)

print("Logit:", logit)
print("취소 확률:", round(probability, 3))
```

실행 결과:

```text
Logit: 1.5
취소 확률: 0.818
```

Logit이 양수이고 값이 커질수록 Sigmoid 결과는 1에 가까워진다. 반대로 Logit이 큰 음수이면 결과는 0에 가까워진다.

### 확률에서 클래스로 변환하기

이진 분류에서는 일반적으로 0.5를 기준값(Threshold)으로 사용한다.

$$
\hat{y}=
\begin{cases}
1 & \text{if } p \ge 0.5 \\
0 & \text{if } p < 0.5
\end{cases}
$$

기준값은 문제의 목적에 따라 변경할 수 있다. 따라서 확률과 최종 클래스는 같은 개념이 아니다.

### 연습문제 2: 여러 Logit을 Sigmoid 확률로 변환

```python
import numpy as np

def sigmoid(z):
    return 1 / (1 + np.exp(-z))

logits = np.array([-2, 0, 2])
probabilities = sigmoid(logits)

print(np.round(probabilities, 3))
```

실행 결과:

```text
[0.119 0.500 0.881]
```

- Logit `-2`는 낮은 양성 확률로 변환된다.
- Logit `0`은 정확히 0.5가 된다.
- Logit `2`는 높은 양성 확률로 변환된다.

---

## 6. Softmax 함수

Softmax는 여러 클래스에 대한 실수값을 각 클래스에 속할 확률로 변환하는 함수이다. 주로 클래스가 3개 이상인 다중 분류에서 사용한다.

클래스 $i$의 Logit이 $z_i$이고 전체 클래스 수가 $K$일 때 Softmax 확률은 다음과 같다.

$$
P(y=i)=\frac{e^{z_i}}{\sum_{j=1}^{K}e^{z_j}}
$$

모든 클래스 확률의 합은 1이다.

$$
\sum_{i=1}^{K}P(y=i)=1
$$

최종 예측은 확률이 가장 큰 클래스이다.

### 음식 추천 확률 계산

```python
import numpy as np

logits = np.array([2.0, 1.0, 0.5])

exp_logits = np.exp(logits)
probabilities = exp_logits / np.sum(exp_logits)

print(np.round(probabilities, 3))
print("예측 클래스:", np.argmax(probabilities))
```

실행 결과:

```text
[0.629 0.231 0.140]
예측 클래스: 0
```

첫 번째 클래스인 한식의 확률이 가장 높으므로 한식을 최종 예측으로 선택한다.

### 연습문제 3: 상품 카테고리 확률 계산

```python
import numpy as np

logits = np.array([1.0, 2.5, 0.5])

exp_logits = np.exp(logits)
probabilities = exp_logits / np.sum(exp_logits)
prediction = np.argmax(probabilities)

print("각 클래스 확률:", np.round(probabilities, 3))
print("예측 클래스:", prediction)
```

실행 결과:

```text
각 클래스 확률: [0.164 0.736 0.100]
예측 클래스: 1
```

클래스 1의 확률이 약 73.6%로 가장 높으므로 최종 예측은 클래스 `1`이다.

### Sigmoid와 Softmax 비교

| 구분 | Sigmoid | Softmax |
|---|---|---|
| 주 사용 문제 | 이진 분류 | 다중 분류 |
| 입력 | 하나의 Logit | 클래스별 여러 Logit |
| 출력 | 특정 클래스의 확률 | 모든 클래스의 확률 분포 |
| 출력 합 | 하나의 출력만으로 합을 논하지 않음 | 모든 클래스 확률의 합이 1 |
| 클래스 선택 | 기준값과 비교 | 가장 큰 확률의 인덱스 선택 |

---

## 7. 이항 로지스틱 회귀

이항 로지스틱 회귀(Binomial Logistic Regression)는 클래스가 2개인 이진 분류 문제에 사용한다.

예시는 다음과 같다.

- 고객 유지 / 이탈
- 시험 불합격 / 합격
- 이메일 정상 / 스팸

일반적으로 Sigmoid를 사용하여 양성 클래스에 속할 확률을 계산한다. 양성 클래스의 확률이 $p$라면 음성 클래스의 확률은 $1-p$이다.

```text
P(y=1) = p
P(y=0) = 1-p
```

기본 기준값이 0.5라면 $p \ge 0.5$일 때 클래스 1, 그렇지 않으면 클래스 0으로 예측한다.

---

## 8. 다항 로지스틱 회귀

다항 로지스틱 회귀(Multinomial Logistic Regression)는 클래스가 3개 이상인 다중 분류 문제에 사용한다.

예를 들어 다음 구매 상품을 세 종류 중 하나로 예측할 수 있다.

- `0`: 식품
- `1`: 의류
- `2`: 전자제품

각 클래스의 Logit에 Softmax를 적용하여 모든 클래스의 확률을 구하고, 그중 확률이 가장 큰 클래스를 선택한다.

### Iris 데이터 다중 분류

```python
from sklearn.datasets import load_iris
from sklearn.linear_model import LogisticRegression

iris = load_iris()
X = iris.data
y = iris.target

model = LogisticRegression(max_iter=200)
model.fit(X, y)

sample = X[[0]]
probabilities = model.predict_proba(sample)
prediction = model.predict(sample)

print("각 클래스 확률:", probabilities.round(3))
print("예측 클래스:", prediction[0])
```

실행 결과:

```text
각 클래스 확률: [[0.982 0.018 0.000]]
예측 클래스: 0
```

Iris 데이터에는 클래스가 세 개 존재한다. 첫 번째 클래스의 확률이 가장 높으므로 모델은 클래스 `0`을 예측한다.

### 두 가지 로지스틱 회귀 비교

| 구분 | Binomial Logistic Regression | Multinomial Logistic Regression |
|---|---|---|
| 클래스 개수 | 2개 | 3개 이상 |
| 문제 유형 | Binary Classification | Multi-class Classification |
| 사용하는 함수 | Sigmoid | Softmax |
| 출력 형태 | 특정 클래스의 확률 | 클래스별 확률이며 합은 1 |
| 예측 방법 | 확률을 기준값과 비교 | 확률이 가장 큰 클래스 선택 |

---

## 9. 범주형 데이터를 숫자로 바꿔야 하는 이유

머신러닝 모델은 일반적으로 숫자 형태의 입력을 사용한다. 따라서 결제 수단, 지역, 색상처럼 문자로 표현된 범주형 데이터는 숫자로 변환해야 한다.

하지만 순서가 없는 범주를 단순한 정수로 바꾸면 문제가 생길 수 있다.

```text
카드 = 1
현금 = 2
간편결제 = 3
```

이렇게 표현하면 모델은 실제로 존재하지 않는 다음 관계가 있다고 해석할 수 있다.

```text
간편결제 > 현금 > 카드
```

결제 수단 사이에는 크기나 순서가 없으므로 이러한 숫자 표현은 적절하지 않다.

---

## 10. 원-핫 인코딩(One-Hot Encoding)

원-핫 인코딩은 각 범주를 별도의 열로 만들고, 데이터가 해당 범주이면 1, 아니면 0으로 표현하는 방법이다.

```text
빨강 → (1, 0, 0)
노랑 → (0, 1, 0)
초록 → (0, 0, 1)
```

각 범주가 독립적인 Feature가 되므로 범주 사이에 존재하지 않는 순서나 크기 관계를 만들지 않는다.

### 결제 수단 원-핫 인코딩

```python
import pandas as pd

df = pd.DataFrame({
    "payment_method": ["카드", "현금", "간편결제", "카드"],
})

encoded = pd.get_dummies(
    df,
    columns=["payment_method"],
    dtype=int,
)

print(encoded)
```

실행 결과:

```text
   payment_method_간편결제  payment_method_카드  payment_method_현금
0                     0                  1                  0
1                     0                  0                  1
2                     1                  0                  0
3                     0                  1                  0
```

고객이 카드 결제를 사용했다면 카드 열만 1이고 나머지 열은 0이 된다.

### 연습문제 4: 배송 지역 원-핫 인코딩

```python
import pandas as pd

df = pd.DataFrame({
    "region": ["서울", "부산", "대전", "서울", "부산"],
})

encoded = pd.get_dummies(
    df,
    columns=["region"],
    dtype=int,
)

print(encoded)
```

실행 결과:

```text
   region_대전  region_부산  region_서울
0          0          0          1
1          0          1          0
2          1          0          0
3          0          0          1
4          0          1          0
```

### 원-핫 인코딩의 핵심

- 범주형 문자를 모델이 처리할 수 있는 숫자로 변환한다.
- 각 범주를 독립된 Feature로 만든다.
- 순서가 없는 범주 사이에 임의의 크기 관계가 생기는 것을 방지한다.
- 범주의 종류가 많으면 생성되는 Feature 수도 많아질 수 있다.

---

## 11. 핵심 용어 정리

| 용어 | 의미 |
|---|---|
| Classification | 입력 데이터를 정해진 클래스 중 하나로 분류하는 문제 |
| GLM | 선형결합을 정답의 특성에 맞는 형태와 연결하는 일반화된 선형 모델 |
| Logit | 확률의 Odds에 로그를 취해 실수 범위로 변환한 값 |
| Sigmoid | 하나의 실수값을 0과 1 사이의 값으로 변환하는 함수 |
| Softmax | 여러 Logit을 합이 1인 클래스별 확률로 변환하는 함수 |
| Threshold | 확률을 최종 클래스로 바꾸는 기준값 |
| Binary Classification | 두 클래스 중 하나를 예측하는 문제 |
| Multi-class Classification | 세 개 이상의 클래스 중 하나를 예측하는 문제 |
| One-Hot Encoding | 각 범주를 독립된 0과 1의 Feature로 변환하는 방법 |

---

## 12. 오늘 배운 내용 정리

### 로지스틱 회귀

- 선형결합 결과를 확률로 변환하여 분류 문제를 해결한다.
- 이름에는 회귀가 있지만 실제로는 분류에 사용한다.
- GLM의 대표적인 모델이다.

### Logit, Sigmoid, Softmax

- Logit은 확률을 전체 실수 범위로 변환한다.
- 로지스틱 회귀는 입력의 선형결합을 Logit으로 모델링한다.
- Sigmoid는 실수값을 0과 1 사이의 값으로 변환한다.
- Softmax는 여러 클래스의 값을 합이 1인 확률 분포로 변환한다.

### 이진 분류와 다중 분류

- 클래스가 2개이면 이진 분류이며 Sigmoid를 사용할 수 있다.
- 클래스가 3개 이상이면 다중 분류이며 Softmax를 사용할 수 있다.
- 확률을 기준으로 최종 클래스를 결정한다.

### 원-핫 인코딩

- 범주형 데이터를 모델이 사용할 수 있는 숫자로 바꾼다.
- 각 범주를 별도의 열로 표현한다.
- 순서가 없는 범주에 잘못된 크기 관계가 생기는 것을 막는다.

### 전체 흐름

```text
숫자형·원-핫 인코딩된 입력 데이터
    ↓
선형결합 z = wᵀx + b
    ↓
이진 분류: Sigmoid
다중 분류: Softmax
    ↓
클래스별 확률 계산
    ↓
기준값 또는 최대 확률로 최종 클래스 선택
```

---

## 13. 느낀 점

오늘은 로지스틱 회귀가 이름과 달리 분류 문제에 사용되는 이유를 배웠다. 선형회귀처럼 입력을 선형결합하지만, 그 결과를 그대로 사용하는 대신 Sigmoid나 Softmax를 통해 확률로 변환한다는 점이 핵심이라고 이해했다.

특히 Logit과 Sigmoid의 관계가 인상적이었다. 로지스틱 회귀는 확률 자체를 선형식으로 표현하는 것이 아니라 확률의 Logit을 선형식으로 모델링하고, Sigmoid를 통해 다시 확률로 변환한다. 이를 통해 선형 모델을 사용하면서도 결과를 0과 1 사이의 값으로 제한할 수 있다는 점을 알게 되었다.

또한 `predict_proba()`는 각 클래스의 확률을 반환하고 `predict()`는 그 확률로 결정한 최종 클래스를 반환한다는 차이를 확인했다. 실제 프로젝트에서는 클래스 결과만 보는 것이 아니라 확률과 기준값도 함께 확인해야겠다고 느꼈다.

원-핫 인코딩에서는 범주를 단순히 1, 2, 3으로 바꾸면 모델이 존재하지 않는 순서를 학습할 수 있다는 점이 중요했다. 앞으로 데이터를 전처리할 때 값이 숫자로 표현되어 있는지만 보는 것이 아니라, 그 숫자가 실제 크기나 순서를 의미하는지도 확인해야겠다.

다음 실습에서는 분류 모델의 확률 기준값을 바꾸었을 때 예측 결과가 어떻게 달라지는지 확인하고, 실제 정답과 예측값을 비교하는 분류 평가 지표도 함께 학습해 보고 싶다.