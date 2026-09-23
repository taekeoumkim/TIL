# 2026-09-23 TIL: 핵심 모델과 선택 기준 압축 정리

> 학습일: 2026-09-23  
> 주제: 지도학습·비지도학습의 핵심 모델과 모델 선택 기준

## 학습 목표

- Linear Regression과 Logistic Regression의 목적과 차이를 설명할 수 있다.
- Decision Tree와 Random Forest의 동작 방식과 장단점을 비교할 수 있다.
- KNN에서 거리, Feature Scaling, K의 의미를 설명할 수 있다.
- K-Means의 군집화 과정과 KNN과의 차이를 구분할 수 있다.
- PCA의 주성분과 설명분산비율을 이해하고 차원 축소에 활용할 수 있다.
- 문제 유형과 데이터 특성에 맞는 모델 후보를 선택할 수 있다.

---

## 1. Linear Regression과 Logistic Regression

### 1.1 Linear Regression

Linear Regression(선형 회귀)은 여러 Feature와 연속형 Target 사이의 관계를 직선 또는 선형식으로 표현하는 회귀 모델이다.

$$
\hat{y}=w_1x_1+w_2x_2+\cdots+w_nx_n+b
$$

- $x_1, x_2, \ldots, x_n$: 입력 Feature
- $w_1, w_2, \ldots, w_n$: 각 Feature의 회귀계수
- $b$: 절편
- $\hat{y}$: 예측값

다른 Feature가 같다는 조건에서 특정 Feature가 1만큼 변할 때 예측값이 얼마나 달라지는지를 해당 회귀계수로 해석할 수 있다.

예를 들어 광고 횟수와 구매 횟수의 관계가 다음과 같다면,

$$
\text{구매 횟수}=2\times\text{광고 횟수}+1
$$

광고 횟수가 1회 늘어날 때 구매 횟수의 예측값은 2회 증가한다.

### 1.2 Logistic Regression

Logistic Regression(로지스틱 회귀)은 이름에 Regression이 들어가지만 **분류** 문제에 사용하는 모델이다. Feature의 선형 결합으로 만든 값 $z$를 Sigmoid 함수에 통과시켜 0과 1 사이의 확률로 변환한다.

$$
z=w_1x_1+w_2x_2+\cdots+w_nx_n+b
$$

$$
\sigma(z)=\frac{1}{1+e^{-z}}
$$

예측 확률을 기준값과 비교하여 클래스를 결정한다. 임계값이 0.5라면 일반적으로 다음과 같이 해석한다.

- 예측 확률 $\geq 0.5$: 클래스 1
- 예측 확률 $< 0.5$: 클래스 0

### 1.3 두 모델 비교

| 구분 | Linear Regression | Logistic Regression |
|---|---|---|
| 문제 유형 | 회귀 | 분류 |
| Target | 연속값 | 클래스 |
| 출력 | 연속적인 예측값 | 클래스 1일 확률 |
| 핵심 | Feature의 선형 결합 | 선형 결합 + Sigmoid |
| 장점 | 단순하고 빠르며 해석이 쉬움 | 확률을 제공하고 해석이 비교적 쉬움 |
| 한계 | 복잡한 비선형 관계 표현이 어려움 | 복잡한 비선형 경계를 표현하기 어려움 |

두 모델 모두 단순하고 계산이 빠르며 좋은 Baseline이 된다. 다만 Feature의 Scale 차이가 크면 Scaling을 고려하고, 계수를 해석할 때는 다른 조건이 같다는 전제를 기억해야 한다.

---

## 2. Decision Tree

Decision Tree(의사결정나무)는 Feature에 대한 조건을 반복적으로 적용하여 데이터를 나누고 예측하는 모델이다.

```text
Root Node
  ├─ 조건 만족 → 하위 Node
  └─ 조건 불만족 → 하위 Node
                     └─ Leaf Node → 최종 예측
```

- **Root Node**: 첫 번째 분할이 시작되는 노드
- **Internal Node**: 추가 조건으로 데이터를 나누는 노드
- **Leaf Node**: 최종 예측값을 제공하는 노드

### 2.1 분류와 회귀

- **분류**: 클래스가 잘 구분되도록 불순도를 낮추는 방향으로 분할한다.
- **회귀**: Target 값이 비슷한 데이터끼리 묶이도록 분할하고 Leaf의 대표값으로 연속값을 예측한다.

분류 트리에서는 Gini Impurity와 같은 기준을 사용할 수 있다. 한 클래스만 있는 순수한 노드의 Gini 값은 0이고, 이진 분류에서 두 클래스가 50:50으로 섞이면 0.5이다.

### 2.2 장단점과 과적합 제어

장점:

- 선형이 아닌 복잡한 관계를 표현할 수 있다.
- 분할 규칙을 확인할 수 있어 비교적 해석하기 쉽다.
- 거리 기반 모델과 달리 일반적으로 Feature Scaling이 필수는 아니다.

한계:

- 트리가 너무 깊어지면 Train 데이터에 과도하게 맞는 과적합이 발생할 수 있다.
- 데이터가 조금만 바뀌어도 분할 구조가 크게 달라질 수 있다.

주요 하이퍼파라미터:

- `max_depth`: 트리의 최대 깊이
- `min_samples_split`: 노드를 분할하는 데 필요한 최소 샘플 수
- `min_samples_leaf`: Leaf Node에 필요한 최소 샘플 수

---

## 3. Random Forest

Random Forest는 여러 Decision Tree를 만들고 그 결과를 결합하는 Ensemble 모델이다. 하나의 트리에 의존하지 않으므로 개별 트리보다 안정적인 예측을 기대할 수 있다.

### 3.1 동작 원리

1. 원본 데이터에서 중복을 허용해 여러 Bootstrap Sample을 만든다.
2. 각 트리를 학습할 때 분할 후보 Feature의 일부를 무작위로 선택한다.
3. 여러 트리의 예측 결과를 결합한다.

- 분류: 여러 트리의 **다수결**
- 회귀: 여러 트리의 예측값 **평균**

데이터와 Feature에 무작위성을 주어 트리 간 차이를 만들고, 여러 결과를 합쳐 단일 트리의 높은 분산과 과적합 위험을 줄인다.

### 3.2 주요 설정

```python
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(
    n_estimators=100,
    max_depth=5,
    random_state=42
)
```

- `n_estimators`: 생성할 트리의 수
- `max_depth`: 각 트리의 최대 깊이
- `min_samples_split`: 노드 분할에 필요한 최소 샘플 수
- `min_samples_leaf`: Leaf에 필요한 최소 샘플 수
- `max_features`: 분할할 때 고려할 Feature의 수

### 3.3 Decision Tree와 비교

| 구분 | Decision Tree | Random Forest |
|---|---|---|
| 구성 | 단일 트리 | 여러 트리의 Ensemble |
| 결과 결합 | 없음 | 다수결 또는 평균 |
| 안정성 | 데이터 변화에 민감 | 상대적으로 안정적 |
| 해석 가능성 | 높음 | 개별 트리보다 낮음 |
| Scaling | 일반적으로 불필요 | 일반적으로 불필요 |

Random Forest는 Feature Importance를 제공하지만, 중요도가 높다는 사실이 곧 인과관계를 의미하지는 않는다.

---

## 4. KNN (K-Nearest Neighbors)

KNN은 새로운 데이터와 가장 가까운 K개의 학습 데이터를 찾아 그 이웃들의 정보를 이용해 예측한다.

- **분류**: 가까운 이웃의 클래스에 대한 다수결
- **회귀**: 가까운 이웃의 Target 값에 대한 평균 등

### 4.1 거리와 Feature Scaling

두 점 $(x_1, y_1)$과 $(x_2, y_2)$ 사이의 Euclidean Distance는 다음과 같다.

$$
d=\sqrt{(x_1-x_2)^2+(y_1-y_2)^2}
$$

KNN은 Feature로 직접 거리를 계산한다. 따라서 방문 횟수는 1~20이고 구매 금액은 10,000~5,000,000이라면 구매 금액이 거리 계산을 지배할 수 있다. 이런 경우 Feature Scaling이 중요하다.

```python
from sklearn.preprocessing import StandardScaler
from sklearn.neighbors import KNeighborsClassifier

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

model = KNeighborsClassifier(n_neighbors=5)
model.fit(X_train_scaled, y_train)
```

Train 데이터로 `fit`한 Scaler를 Test 데이터에는 `transform`만 해야 한다.

### 4.2 K의 크기

- **K가 작을 때**: 가까운 데이터의 지역적 특성을 세밀하게 반영하지만 노이즈에 민감할 수 있다.
- **K가 클 때**: 예측이 부드럽고 안정적일 수 있지만 지역적인 패턴을 놓칠 수 있다.

K는 고정된 정답이 아니라 Validation 또는 교차검증으로 선택해야 하는 하이퍼파라미터다.

KNN은 복잡한 모델 파라미터를 학습하기보다 Train 데이터를 저장해 두었다가 예측 시점에 가까운 이웃을 탐색한다. 학습은 단순하지만 데이터가 많아질수록 예측 비용이 커질 수 있다.

| 구분 | KNN Classification | KNN Regression |
|---|---|---|
| 문제 유형 | 분류 | 회귀 |
| 주변 정보 | 클래스 | Target 값 |
| 대표 예측 방식 | 다수결 | 평균 |
| 주요 하이퍼파라미터 | K | K |
| Scaling | 중요 | 중요 |

---

## 5. K-Means

K-Means는 Target 없이 Feature가 비슷한 데이터를 K개의 Cluster로 묶는 비지도학습 알고리즘이다. 각 Cluster를 대표하는 중심점인 Centroid를 사용한다.

### 5.1 군집화 과정

1. K개의 Centroid를 초기화한다.
2. 각 데이터를 가장 가까운 Centroid의 Cluster에 배정한다.
3. 각 Cluster에 속한 데이터의 평균 위치로 Centroid를 갱신한다.
4. Cluster 배정이 변하지 않거나 Centroid 변화가 충분히 작아질 때까지 2~3을 반복한다.

K-Means의 `Means`는 Cluster의 평균 위치로 Centroid를 갱신한다는 특징과 연결된다.

### 5.2 KNN과의 차이

| 구분 | KNN | K-Means |
|---|---|---|
| 학습 유형 | 지도학습 | 비지도학습 |
| Target | 있음 | 없음 |
| K의 의미 | 참고할 이웃의 수 | 만들 Cluster의 수 |
| 주요 목적 | 새 데이터 예측 | 비슷한 데이터 그룹화 |
| 거리 활용 | 가까운 이웃 탐색 | 가까운 Centroid 탐색 |

두 알고리즘 모두 거리를 사용하므로 Feature Scale의 영향을 크게 받는다.

```python
from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

model = KMeans(n_clusters=3, random_state=42)
clusters = model.fit_predict(X_scaled)
```

### 5.3 K 선택과 군집 해석

K가 커질수록 Cluster 내부의 거리 제곱합인 Inertia는 일반적으로 감소한다. **Elbow Method**는 K를 늘릴 때 Inertia의 감소 폭이 눈에 띄게 작아지는 지점을 K의 후보로 삼는다.

다만 Elbow가 항상 명확한 것은 아니다. 적절한 K를 정할 때는 다음을 함께 고려해야 한다.

- 수치적인 군집 품질
- 비즈니스 목적
- Cluster의 해석 가능성 및 실제 활용 가능성

군집화는 Cluster를 만드는 것으로 끝나지 않는다. 각 Cluster의 Feature 분포를 살펴보고 ‘핵심 고객군’, ‘구매 전환이 필요한 고객군’, ‘저활성 고객군’처럼 분석 목적에 맞는 의미를 부여해야 한다.

---

## 6. PCA (Principal Component Analysis)

PCA는 여러 Feature의 정보를 새로운 축인 Principal Component(주성분)로 변환하여 차원을 줄이는 비지도학습 방법이다.

### 6.1 주성분의 의미

- **PC1**: 데이터의 분산을 가장 많이 담는 방향
- **PC2**: PC1과 직교하면서 남은 분산을 가장 많이 담는 방향
- 이후 주성분: 앞선 주성분과 직교하면서 남은 분산을 최대한 담는 방향

PCA는 기존 Feature 중 일부를 고르는 Feature Selection이 아니다. 기존 Feature들을 조합해 새로운 Feature를 만든다.

| 구분 | Feature Selection | PCA |
|---|---|---|
| 방식 | 기존 Feature 일부 선택 | 기존 Feature를 조합해 주성분 생성 |
| 결과 Feature | 원래 의미 유지 | 새로운 축으로 변환 |
| 해석 | 비교적 쉬움 | 상대적으로 어려움 |

### 6.2 설명분산비율

Explained Variance Ratio(설명분산비율)는 각 주성분이 원본 데이터의 전체 분산을 얼마나 담고 있는지를 나타낸다.

예를 들어 PC1이 50%, PC2가 30%라면 두 주성분의 누적 설명분산비율은 80%다. 목적에 맞는 정보 보존 수준을 정한 뒤 필요한 수의 주성분을 선택한다.

### 6.3 Scaling과 구현

PCA는 분산을 기준으로 새로운 축을 찾기 때문에 Feature의 단위나 Scale 차이가 크면 범위가 큰 Feature가 결과를 지배할 수 있다. 이런 경우 Scaling 후 PCA를 적용한다.

```python
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

pca = PCA(n_components=2)
X_pca = pca.fit_transform(X_scaled)

print(pca.explained_variance_ratio_)
```

`n_components=2`는 원본 데이터를 두 개의 주성분으로 변환한다는 의미다.

### 6.4 장점과 한계

장점:

- 많은 Feature를 더 적은 수의 주성분으로 압축할 수 있다.
- 모델이 처리해야 하는 Feature 수를 줄일 수 있다.
- 고차원 데이터를 2차원 또는 3차원으로 줄여 시각화할 수 있다.
- 서로 관련된 여러 Feature의 정보를 함께 압축할 수 있다.

한계:

- 차원을 줄이는 과정에서 일부 정보가 손실된다.
- 주성분은 여러 원본 Feature의 조합이므로 비즈니스 의미를 직관적으로 해석하기 어렵다.

따라서 PCA는 Feature 수를 무조건 줄이는 도구가 아니라, 얼마나 많은 정보를 유지할지와 변환 결과를 어디에 사용할지를 함께 판단해야 하는 방법이다.

---

## 7. 모델 선택 기준

특정 모델이 모든 데이터에서 항상 가장 좋은 것은 아니다. 문제 유형과 데이터 특성으로 후보를 정하고, 실제 요구사항을 고려한 뒤 Validation 성능을 비교해야 한다.

### 7.1 먼저 문제 유형을 정의한다

| 해결하려는 문제 | 문제 유형 | 모델·방법 후보 |
|---|---|---|
| 연속적인 값 예측 | 회귀 | Linear Regression, Decision Tree Regressor, Random Forest Regressor, KNN Regressor |
| 클래스 예측 | 분류 | Logistic Regression, Decision Tree Classifier, Random Forest Classifier, KNN Classifier |
| Target 없이 비슷한 데이터 그룹화 | 군집화 | K-Means |
| 많은 Feature의 정보 압축 | 차원 축소 | PCA |

### 7.2 데이터 관계와 모델 특성을 확인한다

| 모델 | 잘 맞는 관점 | Scaling | 해석 가능성·주의점 |
|---|---|---|---|
| Linear Regression | 연속형 Target과 선형 관계 | 상황에 따라 고려 | 계수 해석이 쉬우나 비선형 관계에 한계 |
| Logistic Regression | 클래스 확률과 선형 결정 경계 | 상황에 따라 고려 | 확률·계수 해석이 비교적 쉬움 |
| Decision Tree | 비선형 관계와 조건 분할 | 대체로 불필요 | 직관적이지만 과적합·불안정성 주의 |
| Random Forest | 비선형 관계와 안정적 예측 | 대체로 불필요 | 성능이 안정적이나 단일 트리보다 해석이 어려움 |
| KNN | 주변 데이터가 비슷한 답을 갖는 구조 | 중요 | K 선택과 예측 비용에 주의 |
| K-Means | 거리 기반의 비슷한 데이터 그룹화 | 중요 | K 선택 후 군집 의미를 별도로 해석 |
| PCA | 관련 Feature의 정보 압축 | Scale 차이가 크면 중요 | 정보 손실과 해석 난이도 고려 |

### 7.3 실제 요구사항과 성능을 함께 비교한다

모델 후보를 정한 후에는 다음 항목을 함께 확인한다.

- Validation 또는 교차검증 성능
- 결과의 해석 가능성
- 학습 및 예측에 필요한 계산 비용
- 데이터 크기와 Feature 수
- Scaling 등 필요한 전처리
- 과적합 가능성과 하이퍼파라미터 민감도
- 실제 서비스에서의 운영 및 활용 가능성

좋은 모델 선택은 가장 복잡한 모델을 고르는 일이 아니라, **문제에 맞는 후보를 공정하게 비교하고 목적에 맞는 균형점을 찾는 과정**이다.

---

## 전체 비교

| 방법 | 학습 유형 | 주요 목적 | Target | 핵심 설정·개념 |
|---|---|---|---|---|
| Linear Regression | 지도학습 | 연속값 예측 | 있음 | 회귀계수, 절편 |
| Logistic Regression | 지도학습 | 클래스 예측 | 있음 | Sigmoid, 임계값 |
| Decision Tree | 지도학습 | 회귀·분류 | 있음 | 분할 기준, 트리 깊이 |
| Random Forest | 지도학습 | 회귀·분류 | 있음 | 트리 수, Bootstrap, 무작위 Feature |
| KNN | 지도학습 | 회귀·분류 | 있음 | 거리, 이웃 수 K, Scaling |
| K-Means | 비지도학습 | 군집화 | 없음 | Cluster 수 K, Centroid, Inertia |
| PCA | 비지도학습 | 차원 축소 | 없음 | 주성분 수, 설명분산비율, Scaling |

## 핵심 정리

1. 문제 유형을 먼저 회귀, 분류, 군집화, 차원 축소 중 하나로 정의한다.
2. 선형 모델은 빠르고 해석하기 쉬워 Baseline으로 유용하다.
3. Tree 계열은 비선형 관계를 표현하며 보통 Scaling이 필수는 아니다.
4. Random Forest는 여러 트리의 결과를 결합해 단일 Tree보다 안정적이다.
5. KNN과 K-Means는 모두 거리 기반이므로 Scaling이 중요하지만, 학습 유형과 K의 의미가 다르다.
6. PCA는 기존 Feature를 선택하는 것이 아니라 조합해 주성분을 만들며, 설명분산비율로 정보 보존 정도를 확인한다.
7. 최종 모델은 Validation 성능뿐 아니라 해석 가능성, 계산 비용, 운영 목적을 함께 고려해 선택한다.

## 오늘의 한 문장

> 모델 선택은 이름이나 복잡도로 결정하는 것이 아니라, 문제 유형을 정확히 정의하고 데이터 특성·성능·해석 가능성·활용 목적을 함께 비교하는 과정이다.