# 2026-09-02 데이터 분석 환경 구축과 EDA·데이터 전처리

## 학습 목표

- 데이터 분석과 머신러닝 개발 환경을 구축할 수 있다.
- NumPy, pandas, scikit-learn, XGBoost, PyTorch의 주요 역할을 설명할 수 있다.
- EDA와 데이터 전처리의 의미와 차이를 이해할 수 있다.
- pandas와 NumPy로 데이터의 구조와 품질을 점검할 수 있다.
- 결측치, 중복 데이터, 이상치를 상황에 맞게 처리할 수 있다.
- 표준화, Min-Max 스케일링, 원-핫 인코딩의 목적과 주의점을 설명할 수 있다.

---

## 1. 좋은 모델은 환경과 데이터에서 시작한다

머신러닝 프로젝트에서는 모델 알고리즘만큼 데이터와 실행 환경이 중요하다. 라이브러리 버전이나 GPU 환경이 올바르지 않으면 코드가 실행되지 않을 수 있고, 데이터에 결측치·이상치·중복·형식 불일치가 있으면 모델이 잘못된 패턴을 학습할 수 있다.

전체 작업 흐름은 다음과 같이 이해할 수 있다.

```text
분석 환경 구축
      ↓
데이터 불러오기
      ↓
EDA로 구조와 문제 파악
      ↓
목적에 맞게 전처리
      ↓
EDA로 처리 결과 재확인
      ↓
모델 학습용 데이터 준비
```

EDA와 전처리는 서로 완전히 분리된 단계가 아니다. 데이터를 탐색한 결과를 바탕으로 전처리 방법을 결정하고, 전처리 후 데이터가 의도대로 바뀌었는지 다시 탐색하는 반복 과정이다.

---

## 2. 데이터 분석·머신러닝 개발 환경

### 주요 라이브러리

| 라이브러리 | 주요 역할 |
|---|---|
| NumPy | 다차원 배열과 빠른 수치 연산 |
| pandas | 표 형태의 데이터를 불러오고 정리·분석 |
| scikit-learn | 전처리, 머신러닝 모델 학습·예측·평가 |
| XGBoost | 정형 데이터에서 널리 사용하는 Gradient Boosting 모델 |
| PyTorch | 신경망 구성과 딥러닝 모델 학습 |

`uv`를 사용하는 프로젝트에서는 다음과 같이 머신러닝 라이브러리를 추가할 수 있다.

```bash
uv add numpy pandas scikit-learn xgboost
```

설치한 라이브러리의 버전은 다음 코드로 확인할 수 있다.

```python
import numpy as np
import pandas as pd
import sklearn

print("NumPy:", np.__version__)
print("pandas:", pd.__version__)
print("scikit-learn:", sklearn.__version__)
```

처음부터 모든 라이브러리를 깊이 이해할 필요는 없다. 현재는 각 도구가 어떤 역할을 하는지 알고, 필요한 학습 단계에서 하나씩 사용하는 것이 중요하다.

---

## 3. PyTorch 딥러닝 환경과 GPU 확인

PyTorch는 CPU에서도 사용할 수 있지만, NVIDIA GPU가 있다면 큰 신경망의 연산과 학습을 빠르게 수행할 수 있다.

### 1) NVIDIA GPU 확인

터미널에서 다음 명령어를 실행한다.

```bash
nvidia-smi
```

GPU가 정상적으로 인식되면 GPU 이름, 드라이버 버전, 지원하는 CUDA 버전 등을 확인할 수 있다.

### 2) GPU 버전 PyTorch 설치

학습 자료에서는 CUDA 13.0용 인덱스를 사용해 다음과 같이 설치했다.

```bash
uv add torch torchvision --index pytorch-cu130=https://download.pytorch.org/whl/cu130
```

설치 명령의 CUDA 빌드는 운영체제, GPU, NVIDIA 드라이버의 지원 범위에 맞게 선택해야 한다. `nvidia-smi`에 표시되는 CUDA 버전과 PyTorch 빌드의 CUDA 버전이 반드시 완전히 같아야 하는 것은 아니지만, 드라이버가 해당 빌드를 지원해야 한다.

### 3) PyTorch에서 GPU 사용 가능 여부 확인

```python
import torch

print("PyTorch 버전:", torch.__version__)
print("PyTorch CUDA 버전:", torch.version.cuda)
print("GPU 사용 가능:", torch.cuda.is_available())

if torch.cuda.is_available():
    print("GPU 이름:", torch.cuda.get_device_name(0))
```

`torch.cuda.is_available()`이 `True`라면 PyTorch가 GPU를 사용할 수 있는 상태이다.

---

## 4. EDA란?

EDA(Exploratory Data Analysis, 탐색적 데이터 분석)는 통계량과 시각화를 이용해 데이터의 전체적인 특성을 파악하는 과정이다.

EDA에서는 다음을 확인한다.

- 데이터의 행과 열 개수
- 컬럼의 의미와 자료형
- 수치형 변수의 분포와 범위
- 범주형 변수의 종류와 빈도
- 결측치와 중복 데이터
- 이상치와 입력 오류
- 변수 사이의 관계
- 레이블의 분포와 불균형

EDA의 목적은 그래프를 많이 그리는 것이 아니라, 데이터에 어떤 문제가 있고 어떤 전처리와 분석 방법이 필요한지 판단하는 데 있다.

---

## 5. pandas로 데이터 불러오기와 기본 점검

### CSV 불러오기

```python
import pandas as pd

df = pd.read_csv("data.csv")
```

### 구조 확인

```python
print(df.shape)       # 행과 열의 개수
print(df.columns)     # 컬럼명
print(df.head())      # 앞부분 데이터
print(df.tail())      # 뒷부분 데이터
print(df.info())      # 자료형과 결측치 개요
```

`df.info()`는 결과를 화면에 출력하는 메서드이므로 일반적으로 `print(df.info())`보다 `df.info()`로 호출하면 불필요한 `None` 출력을 피할 수 있다.

```python
df.info()
```

### 기초 통계량 확인

```python
# 수치형 변수
print(df.describe())

# 범주형을 포함한 전체 변수
print(df.describe(include="all"))
```

수치형 변수에서는 평균, 표준편차, 최솟값, 사분위수, 최댓값을 확인할 수 있다. 최솟값과 최댓값이 현실적으로 가능한 범위인지 살펴보면 입력 오류나 이상치 후보를 찾는 데 도움이 된다.

### 범주형 변수 확인

```python
print(df["category"].value_counts(dropna=False))
print(df["category"].unique())
print(df["category"].nunique(dropna=False))
```

`dropna=False`를 사용하면 결측값도 하나의 항목으로 포함하여 빈도를 확인할 수 있다.

---

## 6. 데이터 전처리란?

데이터 전처리(Data Preprocessing)는 원시 데이터를 분석과 머신러닝 모델이 사용할 수 있도록 정제하고 변환하는 과정이다.

| 전처리 영역 | 내용 | 예시 |
|---|---|---|
| Data Cleaning | 데이터 품질 문제 정리 | 결측치, 이상치, 중복 처리 |
| Data Transformation | 값의 표현 방식 변환 | 스케일링, 인코딩 |
| Data Integration | 여러 데이터 소스 결합 | 조인, 병합 |
| Data Reduction | 데이터 크기나 특성 수 축소 | 샘플링, 특성 선택, 차원 축소 |

전처리 방법은 데이터의 특성, 모델, 분석 목적에 따라 선택해야 한다. 모든 데이터에 동일한 규칙을 기계적으로 적용해서는 안 된다.

---

## 7. 결측치 확인과 처리

결측치(Missing Value)는 특정 관측값이 존재하지 않는 상태이다. 분석 도구와 모델에 따라 결측치를 직접 처리하지 못할 수 있으며, 처리 방식에 따라 분석 결과도 달라질 수 있다.

### 결측치 현황 확인

```python
missing_count = df.isna().sum()
missing_ratio = (df.isna().mean() * 100).round(2)

missing_summary = pd.DataFrame({
    "결측치_개수": missing_count,
    "결측치_비율(%)": missing_ratio,
}).sort_values("결측치_비율(%)", ascending=False)

print(missing_summary)
```

### 결측치 제거

```python
# 결측치가 있는 행 제거
df_dropped_rows = df.dropna()

# 특정 컬럼이 결측인 행만 제거
df_dropped_target = df.dropna(subset=["target"])

# 결측 비율이 매우 높은 특정 열 제거
df_dropped_column = df.drop(columns=["unnecessary_column"])
```

제거는 간단하지만 데이터 손실이 발생한다. 특히 레이블의 특정 집단에서 결측치가 많이 발생했다면 행 제거가 편향을 키울 수 있다.

### 결측치 대체

```python
# 수치형: 이상치가 있을 수 있으므로 중앙값 사용
df["age"] = df["age"].fillna(df["age"].median())

# 범주형: 최빈값 사용
mode_value = df["city"].mode()[0]
df["city"] = df["city"].fillna(mode_value)

# 결측 자체에 의미가 있다면 별도 범주 사용
df["job"] = df["job"].fillna("Unknown")
```

결측치는 무조건 제거하거나 평균으로 대체해서는 안 된다. 미입력, 측정 실패, 해당 없음 등 발생 원인과 결측 비율을 파악한 뒤 방법을 선택해야 한다.

---

## 8. 중복 데이터 확인과 처리

중복 데이터는 동일한 관측이 여러 번 포함된 상태이다. 단순 입력 중복일 수도 있지만, 실제로 같은 값이 여러 번 발생한 정상 데이터일 수도 있다.

```python
# 모든 컬럼이 같은 완전 중복 행의 개수
print("중복 행:", df.duplicated().sum())

# 중복된 모든 행 확인
duplicates = df[df.duplicated(keep=False)]
print(duplicates)

# 고객 ID와 날짜 기준 중복 확인
key_duplicates = df[df.duplicated(
    subset=["customer_id", "date"],
    keep=False,
)]
print(key_duplicates)
```

중복이 오류임을 확인한 경우에만 제거한다.

```python
df = df.drop_duplicates()
```

주문 데이터에서는 한 고객이 같은 날짜에 여러 번 주문할 수 있으므로 `customer_id`와 `date`가 같다는 이유만으로 바로 삭제해서는 안 된다. 데이터에서 한 행이 무엇을 의미하는지 먼저 확인해야 한다.

---

## 9. 이상치 확인과 처리

이상치(Outlier)는 다른 관측값과 비교해 지나치게 크거나 작거나, 일반적인 패턴에서 크게 벗어난 값이다.

- 나이가 300세: 입력 오류일 가능성이 큼
- 수억 원의 거래 금액: 극단적이지만 실제 거래일 수 있음

따라서 이상치는 곧 오류나 삭제 대상이라는 뜻이 아니다.

### IQR을 이용한 이상치 후보 탐지

IQR(Interquartile Range)은 데이터의 중앙 50%가 위치하는 범위이다.

$$
IQR = Q_3-Q_1
$$

$$
Lower = Q_1-1.5\times IQR
$$

$$
Upper = Q_3+1.5\times IQR
$$

하한보다 작거나 상한보다 큰 값을 이상치 후보로 판단한다.

```python
column = "amount"

q1 = df[column].quantile(0.25)
q3 = df[column].quantile(0.75)
iqr = q3 - q1

lower_bound = q1 - 1.5 * iqr
upper_bound = q3 + 1.5 * iqr

outlier_mask = (
    (df[column] < lower_bound)
    | (df[column] > upper_bound)
)

outliers = df.loc[outlier_mask]

print("하한:", lower_bound)
print("상한:", upper_bound)
print("이상치 후보 수:", outlier_mask.sum())
print(outliers)
```

### 처리 방법

- 입력 오류라면 원본을 확인해 수정하거나 제거
- 분석 목적에 불필요한 값이라면 근거를 기록하고 제거
- 극단값의 영향을 줄이기 위해 로그 변환 또는 범위 제한
- 실제로 발생 가능한 중요한 관측이라면 유지
- 이상치에 덜 민감한 통계량이나 모델 사용

IQR은 이상치 후보를 찾는 통계적 기준일 뿐이다. 금융 거래, 매출, 사용자 활동량처럼 본래 오른쪽 꼬리가 긴 데이터에서는 정상적인 큰 값도 IQR 범위를 벗어날 수 있으므로 도메인 지식과 함께 판단해야 한다.

---

## 10. 스케일링(Scaling)

스케일링은 수치형 특성들의 값의 크기와 범위를 일정한 기준으로 조정하는 과정이다.

나이가 20~70이고 연봉이 1,000,000~2,000,000이라면 연봉의 숫자 범위가 훨씬 크다. 거리 기반 모델에서는 연봉이 거리 계산을 지배할 수 있고, 경사하강법을 사용하는 모델에서는 학습이 비효율적일 수 있다.

### 표준화(Standardization)

평균이 0, 표준편차가 1이 되도록 변환한다.

$$
z=\frac{x-\mu}{\sigma}
$$

```python
from sklearn.preprocessing import StandardScaler

numeric_columns = ["age", "salary"]

scaler = StandardScaler()
scaled_values = scaler.fit_transform(df[numeric_columns])

df_standardized = df.copy()
df_standardized[numeric_columns] = scaled_values
```

### Min-Max Scaling

일반적으로 값을 0과 1 사이로 변환한다.

$$
x'=\frac{x-x_{min}}{x_{max}-x_{min}}
$$

```python
from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler()

df_minmax = df.copy()
df_minmax[numeric_columns] = scaler.fit_transform(df[numeric_columns])
```

### 모델에 따른 필요성

| 스케일링의 영향이 큰 모델 | 영향이 상대적으로 작은 모델 |
|---|---|
| KNN, K-Means, SVM | Decision Tree |
| 선형·로지스틱 회귀 | Random Forest |
| 신경망 | XGBoost 등 트리 기반 부스팅 |

모든 모델에 스케일링이 반드시 필요한 것은 아니다. 트리 기반 모델은 값의 순서와 분할 기준을 사용하므로 일반적으로 스케일 변화의 영향을 거의 받지 않는다.

---

## 11. 인코딩(Encoding)

머신러닝 모델은 범주형 문자열을 직접 처리하지 못하는 경우가 많다. 인코딩은 범주형 값을 모델이 처리할 수 있는 수치형 표현으로 변환하는 과정이다.

고양이, 강아지, 호랑이를 각각 1, 2, 3으로 바꾸면 모델이 ‘호랑이 > 강아지 > 고양이’ 같은 존재하지 않는 크기 관계를 학습할 수 있다. 순서가 없는 명목형 범주에는 원-핫 인코딩을 사용할 수 있다.

### 원-핫 인코딩(One-Hot Encoding)

```text
고양이 → [1, 0, 0]
강아지 → [0, 1, 0]
호랑이 → [0, 0, 1]
```

pandas에서는 다음과 같이 적용할 수 있다.

```python
encoded_df = pd.get_dummies(
    df,
    columns=["animal"],
    dtype=int,
)

print(encoded_df.head())
```

scikit-learn에서는 다음과 같이 사용할 수 있다.

```python
from sklearn.preprocessing import OneHotEncoder

encoder = OneHotEncoder(
    handle_unknown="ignore",
    sparse_output=True,
)

encoded = encoder.fit_transform(df[["animal"]])
print(encoder.get_feature_names_out(["animal"]))
```

`handle_unknown="ignore"`를 사용하면 학습 데이터에 없던 새로운 범주가 예측 단계에 나타났을 때 오류를 피할 수 있다.

### 원-핫 인코딩의 한계

범주의 종류가 매우 많은 High Cardinality 특성은 원-핫 인코딩 후 생성되는 열의 수가 크게 증가한다. 이로 인해 메모리 사용량과 학습 비용이 늘 수 있다.

원-핫 결과는 대부분 0인 희소 데이터이므로 `OneHotEncoder`의 희소 행렬을 활용할 수 있다. 또한 고객 ID처럼 고유값이 지나치게 많은 식별자는 무조건 원-핫 인코딩하기보다 특성에서 제외하거나 목적에 맞는 다른 표현 방법을 검토해야 한다.

---

## 12. 학습·평가 데이터와 데이터 누수 주의

스케일러와 대체값은 전체 데이터가 아니라 **학습 데이터만 이용해 학습**해야 한다. 전체 데이터의 평균, 중앙값, 최솟값, 최댓값을 먼저 사용하면 평가 데이터의 정보가 학습 과정에 들어가는 데이터 누수가 발생한다.

```python
from sklearn.model_selection import train_test_split
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler

X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42,
)

imputer = SimpleImputer(strategy="median")
scaler = StandardScaler()

# 학습 데이터로 기준을 계산
X_train_imputed = imputer.fit_transform(X_train)
X_train_scaled = scaler.fit_transform(X_train_imputed)

# 평가 데이터에는 학습 데이터에서 계산한 기준만 적용
X_test_imputed = imputer.transform(X_test)
X_test_scaled = scaler.transform(X_test_imputed)
```

- `fit()`: 평균, 중앙값, 범주, 스케일 등의 기준을 학습
- `transform()`: 학습된 기준으로 데이터를 변환
- `fit_transform()`: 기준 학습과 변환을 한 번에 수행

평가 데이터에는 `fit_transform()`이 아니라 `transform()`만 사용해야 한다.

---

## 13. 실전 EDA 체크리스트

```python
import pandas as pd

df = pd.read_csv("data.csv")

print("데이터 크기:", df.shape)
print("\n컬럼명:")
print(df.columns.tolist())

print("\n샘플 데이터:")
print(df.head())

print("\n자료형과 결측치:")
df.info()

print("\n수치형 기초 통계:")
print(df.describe())

print("\n결측치 개수:")
print(df.isna().sum().sort_values(ascending=False))

print("\n중복 행 개수:", df.duplicated().sum())

print("\n범주형 고유값 개수:")
print(df.select_dtypes(include="object").nunique())
```

코드를 실행한 뒤 숫자만 출력하고 끝내지 않고 다음 질문에 답해야 한다.

1. 한 행은 무엇을 의미하는가?
2. 예측 대상인 레이블은 무엇인가?
3. 각 컬럼은 예측 시점에 사용할 수 있는 정보인가?
4. 결측치와 이상치는 왜 발생했는가?
5. 자료형과 값의 단위가 올바른가?
6. 특정 범주나 레이블이 지나치게 적지 않은가?
7. 어떤 전처리가 필요하며 그 근거는 무엇인가?

---

## 14. 오늘 배운 내용 정리

1. NumPy와 pandas는 수치 연산과 표 형태의 데이터 분석에 사용하는 기본 라이브러리이다.
2. scikit-learn은 전처리와 전통적인 머신러닝, XGBoost는 정형 데이터 부스팅, PyTorch는 딥러닝에 주로 사용한다.
3. PyTorch의 GPU 사용 가능 여부는 `torch.cuda.is_available()`로 확인할 수 있다.
4. EDA는 데이터의 분포, 관계, 품질 문제를 파악해 분석과 전처리 방향을 결정하는 과정이다.
5. 전처리는 정제, 변환, 통합, 축소로 구분할 수 있으며 EDA와 반복해서 수행한다.
6. 결측치는 발생 원인과 비율을 확인한 뒤 제거 또는 적절한 값으로 대체해야 한다.
7. IQR 범위를 벗어난 값은 이상치 후보일 뿐이며 데이터 의미를 확인하지 않고 자동 삭제하면 안 된다.
8. 표준화는 평균 0·표준편차 1로, Min-Max Scaling은 일반적으로 0~1 범위로 변환한다.
9. 원-핫 인코딩은 명목형 범주에 불필요한 순서를 부여하지 않지만, 범주 수가 많으면 차원이 크게 증가한다.
10. 전처리 기준은 학습 데이터에서만 계산하고 평가 데이터에는 동일한 기준을 적용해야 데이터 누수를 막을 수 있다.

---

## 15. 느낀 점

머신러닝 성능을 높이기 위해서는 복잡한 모델을 선택하기 전에 데이터의 구조와 품질을 제대로 이해하는 것이 먼저라는 점을 배웠다. 특히 결측치나 IQR 범위를 벗어난 값을 무조건 삭제하는 것이 아니라, 왜 그런 값이 발생했는지 확인하고 분석 목적에 따라 처리해야 한다는 점이 중요하게 느껴졌다.

또한 스케일링과 인코딩은 단순히 함수를 실행하는 작업이 아니라 사용할 모델과 데이터의 의미를 고려해 선택해야 하는 과정이었다. 앞으로 EDA를 진행할 때 코드 실행 결과만 나열하지 않고, 각 문제의 원인과 처리 근거, 처리 전후의 데이터 변화를 기록하며 재현 가능한 분석을 해야겠다.
