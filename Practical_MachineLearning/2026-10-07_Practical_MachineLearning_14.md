# 2026-10-07 재현 가능한 End-to-End Pipeline 구축

## 학습 목표

- 전처리와 모델을 함께 관리해야 하는 이유를 설명할 수 있다.
- Pipeline의 학습 과정과 예측 과정의 차이를 이해한다.
- ColumnTransformer로 수치형·범주형 Feature에 서로 다른 전처리를 적용할 수 있다.
- Pipeline 전체를 교차검증과 하이퍼파라미터 탐색에 연결할 수 있다.
- 최종 Pipeline을 저장하고 새로운 Raw Data에 재사용할 수 있다.

---

## 1. 모델만 저장하면 충분할까?

머신러닝 예측은 모델 하나만으로 만들어지는 것이 아니다. 결측치 처리, Scaling, Encoding을 적용한 뒤 모델에 전달하는 경우가 많다.

학습에서는 나이와 월 이용 금액을 표준화했는데 예측에서는 원래 값을 입력하면, 모델이 학습한 입력 기준과 달라진다. 새 데이터에서 Scaler를 다시 학습하는 것도 문제이다. 새 데이터의 평균과 표준편차에 따라 같은 값의 변환 결과가 달라질 수 있기 때문이다.

따라서 재사용할 대상은 **학습된 전처리 기준과 모델을 포함한 전체 처리 과정**이다.

---

## 2. End-to-End Pipeline의 의미

End-to-End Pipeline은 데이터 입력부터 전처리와 예측까지의 흐름을 하나의 구조로 연결한 것이다.

| 단계 | 역할 |
|---|---|
| Raw Data 입력 | 학습 시 정의한 Feature 구조의 원본 데이터를 받음 |
| 결측치 처리 | 학습 데이터에서 정한 기준으로 누락된 값을 대체 |
| Scaling / Encoding | 수치형·범주형 값을 모델의 입력 형태로 변환 |
| 모델 | 변환된 Feature로 예측 수행 |
| Prediction | 최종 클래스나 예측값 반환 |

Pipeline은 코드를 짧게 만드는 도구를 넘어, 처리 순서와 학습된 객체를 함께 보존하는 구조이다. 학습과 예측에서 같은 기준을 적용하는 데 도움이 된다.

여기서 Raw Data는 아무 형식의 데이터가 아니라 학습 시 정한 컬럼과 자료형을 갖춘 입력을 뜻한다.

---

## 3. fit과 predict의 차이

`pipeline.fit(X_train, y_train)`을 호출하면 다음 과정이 수행된다.

1. Imputer가 결측치 대체 기준을 학습한다.
2. Scaler가 평균과 표준편차를, Encoder가 범주 정보를 학습한다.
3. 학습 데이터를 정해진 기준으로 변환한다.
4. 변환된 데이터로 마지막 단계의 모델을 학습한다.

`pipeline.predict(new_data)`에서는 전처리를 다시 학습하지 않는다. 저장된 결측치 대체 기준, Scaling 기준과 Encoding 정보를 적용한 뒤 모델로 예측한다.

| 호출 | 전처리 | 모델 |
|---|---|---|
| fit | 기준 학습과 변환 | 학습 |
| predict | 학습된 기준으로 변환 | 예측 |

새 데이터에서 fit을 다시 호출하면 기존 기준과 모델을 다시 학습하게 된다. 학습된 Pipeline을 예측에 사용할 때는 predict를 호출한다.

---

## 4. Feature별 전처리와 ColumnTransformer

수치형과 범주형 Feature에는 서로 다른 전처리가 필요할 수 있다.

| Feature | 종류 | 전처리 예시 |
|---|---|---|
| age | 수치형 | 중앙값으로 결측치 대체, StandardScaler |
| monthly_spend | 수치형 | 중앙값으로 결측치 대체, StandardScaler |
| membership | 범주형 | 최빈값으로 결측치 대체, OneHotEncoder |
| region | 범주형 | 최빈값으로 결측치 대체, OneHotEncoder |

이 조합은 예시이다. 모든 모델에 Scaling이 필요한 것은 아니며 대체 기준도 데이터의 의미에 맞게 선택해야 한다.

**ColumnTransformer**는 어떤 컬럼에 어떤 변환을 적용할지 지정하고, 각각의 결과를 하나의 Feature 데이터로 결합한다.

~~~python
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.linear_model import LogisticRegression

numeric_features = ["age", "monthly_spend"]
categorical_features = ["membership", "region"]

numeric_pipeline = Pipeline([
    ("imputer", SimpleImputer(strategy="median")),
    ("scaler", StandardScaler()),
])

categorical_pipeline = Pipeline([
    ("imputer", SimpleImputer(strategy="most_frequent")),
    ("encoder", OneHotEncoder(handle_unknown="ignore")),
])

preprocessor = ColumnTransformer([
    ("numeric", numeric_pipeline, numeric_features),
    ("categorical", categorical_pipeline, categorical_features),
])

pipeline = Pipeline([
    ("preprocessor", preprocessor),
    ("model", LogisticRegression(max_iter=1000)),
])
~~~

구조는 세 단계로 이해할 수 있다.

- Feature 종류별 Pipeline이 각각의 전처리 순서를 관리한다.
- ColumnTransformer가 대상 컬럼을 선택하고 변환 결과를 결합한다.
- 전체 Pipeline이 전처리와 모델을 연결한다.

위 설정에서 지정하지 않은 컬럼은 기본값인 `remainder="drop"`에 따라 제외된다. 필요한 컬럼이 빠지지 않도록 대상 컬럼을 명확히 정의해야 한다.

---

## 5. 학습에 없던 범주가 입력되면?

학습 데이터의 지역이 Seoul과 Busan뿐이어도 새로운 고객의 지역은 Jeju일 수 있다.

`OneHotEncoder(handle_unknown="ignore")`는 학습에 없던 범주로 인한 변환 오류를 피하도록 한다. 해당 Feature의 기존 범주에 대응하는 One-Hot 값은 모두 0이 된다.

예측할 때 Jeju라는 새 컬럼을 추가하는 것이 아니라, 학습 때 정한 출력 구조를 유지한다. 따라서 이 설정이 새로운 범주의 패턴을 학습한다는 뜻은 아니다.

---

## 6. Pipeline과 CV로 전처리 누수 방지하기

전체 데이터로 중앙값, 평균, 표준편차를 계산한 뒤 교차검증하면 Validation 정보가 전처리에 포함된다.

Pipeline 전체를 GridSearchCV에 전달하면 각 Fold의 Train만으로 전처리와 모델을 학습한다.

| 데이터 | 전처리 | 모델 |
|---|---|---|
| Fold의 Train | 기준 학습과 변환 | 학습 |
| Fold의 Validation | 해당 Train 기준으로 변환 | 예측과 평가 |
| Hold-out Test | 최종 Train 기준으로 변환 | 최종 평가 |

Pipeline 밖에서 전체 데이터에 전처리를 먼저 적용했다면 그 단계의 누수는 해결되지 않는다. **학습이 필요한 전처리를 Pipeline 내부에 포함하는 것**이 중요하다.

---

## 7. Pipeline 전체의 하이퍼파라미터 탐색

다음 코드는 앞에서 구성한 Pipeline을 사용한다. `data`는 4개 Feature와 이진 타깃 `churn`을 가진 DataFrame이며, 각 클래스에는 교차검증에 충분한 표본이 있다고 가정한다.

~~~python
from sklearn.model_selection import (
    train_test_split, StratifiedKFold, GridSearchCV
)

X = data[numeric_features + categorical_features]
y = data["churn"]

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,
    stratify=y,
    random_state=42,
)

cv = StratifiedKFold(
    n_splits=3,
    shuffle=True,
    random_state=42,
)

param_grid = {"model__C": [0.1, 1, 10]}

grid_search = GridSearchCV(
    estimator=pipeline,
    param_grid=param_grid,
    cv=cv,
    scoring="accuracy",
    refit=True,
)

grid_search.fit(X_train, y_train)
print("선택한 설정:", grid_search.best_params_)
print("평균 CV Accuracy:", grid_search.best_score_)
~~~

Pipeline 내부 설정은 **단계이름__파라미터이름**으로 지정한다. `model__C`는 model 단계의 LogisticRegression에 있는 C를 뜻한다. C는 규제 강도의 역수이므로 작은 값일수록 규제가 강해진다.

3개 후보를 3-Fold로 평가하므로 CV 학습은 9번이다. 기본 refit에서는 선택한 설정의 Pipeline을 Train 전체에 한 번 더 학습한다.

여기서는 Accuracy를 사용하지만, 불균형 분류에서는 문제에 맞게 F1이나 Recall 등의 지표도 고려해야 한다.

---

## 8. 최종 Pipeline 선택과 Test 평가

| 속성 | 의미 |
|---|---|
| best_params_ | 가장 높은 평균 CV 점수를 기록한 설정 |
| best_score_ | 선택한 설정의 평균 CV 점수 |
| best_estimator_ | 기본 refit 이후 전처리와 모델을 포함한 최종 Pipeline |

~~~python
from sklearn.metrics import accuracy_score

best_pipeline = grid_search.best_estimator_
test_pred = best_pipeline.predict(X_test)
test_accuracy = accuracy_score(y_test, test_pred)

print("Test Accuracy:", test_accuracy)
~~~

Test에는 변환 전의 Feature를 그대로 전달한다. Scaler나 Encoder를 따로 적용하면 전처리가 중복될 수 있다.

Test는 선택이 끝난 모델의 최종 평가에 사용한다. 여러 Pipeline의 Test 점수를 반복 확인하며 가장 좋은 모델을 고르면 Test도 모델 선택 과정에 사용된 것이다.

---

## 9. Pipeline 전체 저장과 새로운 데이터 예측

모델만 저장하면 결측치 대체 기준, Scaling 기준과 범주 정보가 함께 보존되지 않을 수 있다. 학습된 Pipeline 전체를 저장해야 같은 처리 흐름을 재사용하기 쉽다.

~~~python
import joblib

joblib.dump(best_pipeline, "churn_pipeline.pkl")
loaded_pipeline = joblib.load("churn_pipeline.pkl")
~~~

새 고객을 학습 때와 같은 Feature 구조로 입력한다.

~~~python
import pandas as pd
import numpy as np

new_customers = pd.DataFrame({
    "age": [34, np.nan],
    "monthly_spend": [55, 70],
    "membership": ["Basic", "Premium"],
    "region": ["Seoul", "Jeju"],
})

predictions = loaded_pipeline.predict(new_customers)
probabilities = loaded_pipeline.predict_proba(new_customers)

print("예측 클래스:", predictions)
print("클래스 순서:", loaded_pipeline.classes_)
print("클래스별 확률:", probabilities)
~~~

Imputer, Scaler, Encoder를 직접 호출할 필요가 없다. 저장된 학습 기준이 자동으로 적용된다. predict_proba의 각 열이 어떤 클래스에 해당하는지는 classes_로 확인한다.

새 데이터에서도 Feature의 이름과 의미, 단위, 자료형은 학습과 일치해야 한다. 신뢰할 수 있는 모델 파일만 불러오는 것도 필요하다.

---

## 10. 재현성을 위해 함께 기록할 정보

Pipeline은 처리 구조와 학습된 기준을 보존한다. 프로젝트를 다시 학습하고 비교하려면 데이터와 실행 환경도 기록해야 한다.

| 기록할 정보 | 목적 |
|---|---|
| 입력 Feature 이름과 자료형 | 새 데이터의 구조 확인 |
| 데이터 버전과 대상 기간 | 학습에 사용한 데이터 식별 |
| 분할 설정과 random_state | 학습·평가 조건 재현 |
| 전처리와 모델 설정 | 선택한 처리 방식 관리 |
| CV와 Test 지표·점수 | 검증 결과와 선택 근거 보존 |
| 라이브러리 버전 | 저장 객체의 호환성과 실행 환경 관리 |

random_state만 고정한다고 모든 환경에서 같은 결과가 보장되는 것은 아니다. 데이터, 설정과 실행 환경을 함께 관리해야 한다.

전체 작업은 다음 순서로 연결된다.

1. Feature와 타깃을 정의한다.
2. 최종 Test를 분리한다.
3. 종류별 전처리를 구성한다.
4. ColumnTransformer와 모델을 Pipeline으로 연결한다.
5. Pipeline 전체를 CV로 탐색한다.
6. 선택한 Pipeline의 Test 성능을 확인한다.
7. Pipeline과 관련 정보를 저장한다.
8. 새 Raw Data에 같은 기준으로 예측한다.

---

## 11. 핵심 정리

- 예측에는 모델뿐 아니라 학습된 전처리 기준이 필요하다.
- Pipeline은 전처리와 모델을 연결하여 학습과 예측의 순서를 일관되게 관리한다.
- fit에서는 전처리와 모델을 학습하고, predict에서는 저장된 기준으로 변환과 예측을 수행한다.
- ColumnTransformer는 Feature별로 다른 전처리를 적용하고 결과를 결합한다.
- handle_unknown="ignore"는 학습에 없던 범주를 기존 출력 구조에서 처리한다.
- Pipeline 전체를 CV 탐색에 전달하면 각 Fold의 Train에서만 전처리를 학습할 수 있다.
- Pipeline 내부 설정은 단계이름__파라미터이름으로 지정한다.
- 기본 refit 이후 best_estimator_는 Train 전체로 다시 학습된 최종 Pipeline이다.
- Test는 모델 선택이 끝난 뒤 최종 성능 평가에 사용한다.
- Pipeline 전체를 저장하면 새 Raw Data에 같은 전처리와 예측을 재사용할 수 있다.

---

## 오늘의 한 문장

> 재현 가능한 머신러닝은 모델만 보관하는 것이 아니라, Raw Data가 예측으로 바뀌는 전처리 기준과 처리 순서를 모델과 함께 저장하고 재사용하는 것이다.
