# 2026-10-07 CV 기반 하이퍼파라미터 튜닝과 모델 탐색

## 학습 목표

- Parameter와 Hyperparameter의 차이를 설명할 수 있다.
- 하이퍼파라미터 탐색에 교차검증을 결합해야 하는 이유를 이해한다.
- GridSearchCV와 RandomizedSearchCV의 차이와 탐색 비용을 비교할 수 있다.
- 탐색 결과의 평균 성능, 표준편차, 학습 시간을 해석할 수 있다.
- Hold-out Test로 최종 모델을 평가하고 학습된 모델을 저장하여 재사용할 수 있다.

---

## 1. 같은 모델도 설정에 따라 성능이 달라진다

Random Forest의 Tree 깊이를 바꾸면 학습할 수 있는 패턴의 복잡도가 달라진다. 깊이가 너무 작으면 필요한 패턴을 놓칠 수 있고, 지나치게 크면 학습 데이터의 세부적인 패턴에 과적합될 수 있다.

따라서 알고리즘을 선택하는 것에 더해, 해당 데이터에 적절한 설정을 찾는 과정이 필요하다. 이 과정을 **하이퍼파라미터 튜닝(Hyperparameter Tuning)**이라고 한다.

좋은 설정은 모델 이름만으로 정할 수 없다. 후보를 학습하고 검증하여 대상 데이터에서의 일반화 성능을 비교해야 한다.

---

## 2. Parameter와 Hyperparameter

| 구분 | Parameter | Hyperparameter |
|---|---|---|
| 의미 | 모델이 학습 과정에서 데이터로 결정하는 값 | 학습 전에 지정하는 모델 구조·학습 방식의 설정 |
| 결정 방식 | 모델이 데이터를 학습하면서 결정 | 사용자 또는 탐색 절차가 후보를 지정하고 선택 |
| 예시 | 선형 회귀의 계수, Tree의 실제 분할 기준 | max_depth, n_estimators, learning_rate |

Decision Tree에서 `max_depth=5`를 지정하는 것은 최대 깊이를 정하는 것이다. 실제로 어떤 Feature를 어떤 값에서 나눌지는 데이터로부터 학습한다.

Random Forest의 대표적인 하이퍼파라미터는 다음과 같다.

- `n_estimators`: Tree의 개수
- `max_depth`: 각 Tree의 최대 깊이
- `min_samples_split`: 노드를 분할하기 위한 최소 표본 수

XGBoost에서는 `learning_rate`처럼 각 Tree의 기여도를 조절하는 값도 탐색할 수 있다.

---

## 3. 하이퍼파라미터 탐색에 CV를 결합하는 이유

하나의 Validation 데이터에서 가장 높은 점수를 얻은 설정이 다른 분할에서도 좋은 것은 아니다. 특정 분할에 우연히 잘 맞는 후보를 선택할 수 있기 때문이다.

**교차검증(Cross-Validation, CV)**을 이용하면 같은 후보를 여러 Train / Validation 조합에서 반복 평가할 수 있다.

예를 들어 한 후보의 Fold별 F1-score가 0.85, 0.82, 0.84라면 평균은 약 0.837이다. 단일 점수보다 여러 분할의 성능을 종합하여 비교할 수 있다.

| 확인할 값 | 의미 |
|---|---|
| 평균 CV 점수 | 여러 Fold에서의 전반적인 검증 성능 |
| CV 표준편차 | Fold에 따라 점수가 얼마나 달라지는지 |
| 개별 Fold 점수 | 특정 분할에서 성능이 크게 떨어지는지 |

표준편차가 작으면 해당 분할들에서 성능 차이가 상대적으로 작다는 뜻이다. 실제 서비스에서의 안정성을 보장하거나 그 자체로 정확한 신뢰구간을 제공하는 것은 아니다.

교차검증 방법도 데이터에 맞게 선택한다. 일반적인 분류에는 StratifiedKFold, 시간 순서가 중요한 데이터에는 TimeSeriesSplit 등을 고려한다.

---

## 4. Hold-out Test는 먼저 분리해 두기

튜닝에 사용한 데이터에서 얻은 점수와 최종 평가 점수는 역할이 다르다.

| 데이터 | 역할 |
|---|---|
| 개발용 Train 데이터 | 내부 CV로 후보 모델과 하이퍼파라미터 비교 |
| 각 Fold의 Validation | 해당 후보의 검증 점수 계산 |
| Hold-out Test | 모든 선택이 끝난 뒤 최종 일반화 성능 평가 |

처음에 Test 데이터를 분리하고 나머지 데이터에서 탐색한다. CV의 각 회차에서는 일부가 Train, 일부가 Validation 역할을 한다.

Test 점수를 반복 확인하며 설정을 바꾸면 Test 정보가 모델 선택에 반영된다. 이 경우 Test는 독립적인 최종 평가 데이터로서의 역할을 잃는다.

**CV는 선택을 위한 평가이고, Hold-out Test는 선택이 끝난 모델의 최종 평가이다.**

---

## 5. GridSearchCV: 지정한 모든 조합 탐색하기

GridSearchCV는 지정한 후보 값들의 모든 조합을 교차검증으로 평가한다.

~~~python
param_grid = {
    "n_estimators": [100, 200, 300],
    "max_depth": [5, 10, 15],
}
~~~

Tree 개수 후보가 3개이고 깊이 후보가 3개이므로 조합은 9개이다.

$$
3\times3=9
$$

각 후보를 5-Fold CV로 평가하면 교차검증 과정의 학습 횟수는 45번이다.

$$
9\times5=45
$$

기본 설정인 `refit=True`에서는 탐색이 끝난 뒤 가장 좋은 후보로 **개발용 Train 데이터 전체에 한 번 더 학습**한다. 따라서 이 예시에서는 CV 학습 45번에 최종 재학습 1번이 추가된다.

### 장점과 한계

- 후보가 적을 때 지정한 조합을 빠짐없이 비교할 수 있다.
- 하이퍼파라미터와 후보 수가 늘어나면 조합 수가 빠르게 증가한다.
- 하이퍼파라미터 3개에 각각 후보 10개라면 조합은 1,000개이다.
- 여기에 5-Fold CV를 적용하면 CV 학습만 5,000번 필요하다.

모든 조합을 탐색하더라도 찾는 것은 **지정한 후보 안에서 가장 좋은 조합**이다. 전체 가능한 설정 중 절대적으로 최적이라는 뜻은 아니다.

---

## 6. GridSearchCV 구현 예시

다음은 이진 분류 데이터를 생성하고, Test를 분리한 뒤 Random Forest를 탐색하는 예시이다.

~~~python
from sklearn.datasets import make_classification
from sklearn.model_selection import (
    train_test_split, StratifiedKFold, GridSearchCV
)
from sklearn.ensemble import RandomForestClassifier

X, y = make_classification(
    n_samples=1000,
    n_features=10,
    n_informative=6,
    weights=[0.7, 0.3],
    random_state=42,
)

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,
    stratify=y,
    random_state=42,
)

cv = StratifiedKFold(
    n_splits=5,
    shuffle=True,
    random_state=42,
)

model = RandomForestClassifier(random_state=42, n_jobs=1)
param_grid = {
    "n_estimators": [100, 200, 300],
    "max_depth": [5, 10, 15],
}

grid_search = GridSearchCV(
    estimator=model,
    param_grid=param_grid,
    cv=cv,
    scoring="f1",
    n_jobs=-1,
    refit=True,
)

grid_search.fit(X_train, y_train)

print("선택한 설정:", grid_search.best_params_)
print("평균 CV F1:", grid_search.best_score_)
~~~

`scoring="f1"`은 여기서 타깃 1을 양성 클래스로 보는 이진 분류 F1-score를 사용한다. 다중 분류라면 목적에 맞게 `f1_macro` 등의 지표를 선택한다.

`n_jobs=-1`은 탐색을 병렬로 수행하도록 설정한다. 위 예시에서는 탐색 단계에서 병렬화하고, 개별 Random Forest의 `n_jobs`는 1로 두었다.

---

## 7. RandomizedSearchCV: 횟수를 제한하여 탐색하기

RandomizedSearchCV는 후보 목록이나 확률분포에서 하이퍼파라미터 조합을 무작위로 선택하여 평가한다. 모든 조합을 확인하는 대신 `n_iter`로 탐색할 후보 수를 정한다.

앞에서 만든 개발용 Train 데이터와 CV를 그대로 사용한다.

~~~python
from scipy.stats import randint
from sklearn.model_selection import RandomizedSearchCV
from sklearn.ensemble import RandomForestClassifier

param_distributions = {
    "n_estimators": randint(100, 500),
    "max_depth": randint(3, 20),
}

random_search = RandomizedSearchCV(
    estimator=RandomForestClassifier(
        random_state=42, n_jobs=1
    ),
    param_distributions=param_distributions,
    n_iter=20,
    cv=cv,
    scoring="f1",
    random_state=42,
    n_jobs=-1,
    refit=True,
)

random_search.fit(X_train, y_train)

print("선택한 설정:", random_search.best_params_)
print("평균 CV F1:", random_search.best_score_)
~~~

`randint(100, 500)`은 100 이상 500 미만의 정수에서 값을 뽑는다. 따라서 위 설정의 Tree 개수 범위는 100~499이고 깊이는 3~19이다.

`n_iter=20`, 5-Fold CV이면 CV 학습 횟수는 다음과 같다.

$$
20\times5=100
$$

기본 재학습까지 포함하면 여기에 1번이 추가된다. 분포에서 추출할 때는 같은 조합이 반복될 수도 있으므로 20회 탐색이 항상 서로 다른 20개 조합을 뜻하는 것은 아니다.

넓은 탐색 공간에서도 예산을 정해 탐색할 수 있지만, 중요한 조합을 놓칠 수 있다. `random_state`를 고정하면 후보 추출 과정을 재현하는 데 도움이 된다.

---

## 8. Grid Search와 Random Search 선택 기준

| 기준 | GridSearchCV | RandomizedSearchCV |
|---|---|---|
| 탐색 대상 | 지정한 모든 조합 | 지정한 횟수만큼 추출한 조합 |
| 탐색 비용 | 후보 조합 수에 따라 결정 | 주로 n_iter로 조절 |
| 적합한 상황 | 후보가 적고 전부 비교할 수 있음 | 탐색 공간이 넓거나 자원이 제한됨 |
| 주요 한계 | 후보 수가 많아지면 비용 급증 | 탐색하지 않은 좋은 조합을 놓칠 수 있음 |

먼저 Random Search로 넓은 범위를 살펴보고, 좋은 결과가 나온 주변을 좁혀 Grid Search로 비교하는 방식도 가능하다. 중요한 것은 탐색 범위와 비용을 의식하면서 검증하는 것이다.

---

## 9. 탐색 결과 읽기

| 속성 | 의미 |
|---|---|
| `best_params_` | 탐색 후보 중 가장 높은 평균 CV 점수를 얻은 설정 |
| `best_score_` | 해당 설정의 평균 CV 점수 |
| `best_estimator_` | 기본 refit 설정에서 개발용 Train 전체로 다시 학습된 모델 |
| `cv_results_` | 탐색한 모든 후보의 평가 결과 |

`cv_results_`를 DataFrame으로 만들면 점수와 변동, 학습 시간을 함께 비교할 수 있다.

~~~python
import pandas as pd

results = pd.DataFrame(random_search.cv_results_)

columns = [
    "params",
    "mean_test_score",
    "std_test_score",
    "mean_fit_time",
    "rank_test_score",
]

print(results[columns].sort_values("rank_test_score"))
~~~

| 결과 항목 | 해석 |
|---|---|
| `params` | 평가한 하이퍼파라미터 조합 |
| `mean_test_score` | CV의 Validation 점수 평균 |
| `std_test_score` | CV의 Validation 점수 표준편차 |
| `mean_fit_time` | 해당 후보의 Fold별 평균 학습 시간 |
| `rank_test_score` | 평균 CV 점수에 따른 순위 |

이름에 `test`가 있어도 `mean_test_score`는 별도로 남겨 둔 Hold-out Test 점수가 아니다. 여기서는 **CV 내부의 Validation 점수**를 의미한다.

`mean_fit_time`도 탐색 전체에 걸린 시간이 아니다. 후보 수와 Fold 수, 병렬화, 점수 계산과 재학습 등이 전체 탐색 시간에 영향을 준다.

---

## 10. 평균 성능, 안정성, 비용을 함께 비교하기

다음은 모델 선택을 설명하기 위한 예시이다.

| 모델 | 평균 CV F1 | CV 표준편차 | 평균 학습 시간 |
|---|---:|---:|---:|
| Logistic Regression | 0.81 | 0.01 | 0.2초 |
| Random Forest | 0.86 | 0.02 | 1.5초 |
| XGBoost | 0.87 | 0.03 | 3.8초 |

평균만 보면 XGBoost가 높다. 그러나 Random Forest와의 차이는 0.01이며, 이 예시에서는 변동과 학습 시간도 더 크다.

성능 개선이 중요한 문제라면 XGBoost를 고려할 수 있다. 빠른 반복 학습과 비용이 중요하다면 Random Forest를 선택할 근거가 있다.

다만 평균과 표준편차만으로 두 모델의 차이가 통계적으로 확실하다고 단정해서는 안 된다. 같은 Fold의 결과와 목적을 함께 살펴봐야 한다.

기본 탐색 객체는 평균 점수를 기준으로 후보를 선택한다. 비용이나 안정성을 이유로 다른 설정을 최종 선택한다면, 선택한 설정으로 개발용 Train 전체에 다시 학습해야 한다.

---

## 11. 최종 모델의 Hold-out Test 평가

Grid Search의 최고 평균 점수 후보를 최종 선택했다고 가정하자.

~~~python
from sklearn.metrics import f1_score

best_model = grid_search.best_estimator_

test_pred = best_model.predict(X_test)
test_f1 = f1_score(y_test, test_pred)

print("평균 CV F1:", grid_search.best_score_)
print("최종 Test F1:", test_f1)
~~~

`refit=True`로 탐색했다면 `best_estimator_`는 이미 개발용 Train 전체로 학습된 모델이다. Test 데이터로 `fit()`하지 않고 예측과 평가에만 사용한다.

CV 점수와 Test 점수가 비슷하면 성능 추정이 일관적인지 확인하는 근거가 된다. 다만 점수가 반드시 같아야 하는 것은 아니며, Test의 표본 수와 데이터 구성에 따라 차이가 날 수 있다.

또한 여러 후보 중 가장 높은 점수를 골랐기 때문에 `best_score_`만으로 최종 성능을 확정하면 안 된다. 독립적으로 남겨 둔 Test 평가가 필요한 이유이다.

---

## 12. 모델 저장과 재사용

선택한 모델을 저장하면 이후 새로운 데이터를 예측할 때 다시 학습하지 않고 사용할 수 있다.

~~~python
import joblib

joblib.dump(best_model, "best_model.pkl")

loaded_model = joblib.load("best_model.pkl")

# 재사용 예시: 실제 활용 시 같은 입력 구조의 new_data를 사용한다.
new_data = X_test[:5]
new_pred = loaded_model.predict(new_data)
print(new_pred)
~~~

새 데이터는 학습할 때와 Feature의 의미, 순서, 자료형이 일치해야 한다. 전처리가 필요한 모델이라면 모델과 함께 전처리 기준도 보존해야 한다.

전처리와 모델을 Pipeline으로 구성했다면 Pipeline 전체를 저장하는 것이 유용하다. 또한 신뢰할 수 있는 모델 파일만 불러와야 한다.

### 모델 정보 함께 기록하기

~~~python
model_info = {
    "model": "RandomForestClassifier",
    "best_params": grid_search.best_params_,
    "cv_f1": float(grid_search.best_score_),
    "test_f1": float(test_f1),
    "scoring": "f1",
    "cv_n_splits": 5,
    "random_state": 42,
}

joblib.dump(model_info, "model_info.pkl")
~~~

사용한 데이터 버전, Feature 목록, 전처리 과정과 라이브러리 버전도 기록하면 이후 결과를 재현하고 모델을 관리하기 쉽다.

---

## 13. 데이터 누수 없이 탐색하기

CV 탐색을 사용한다고 모든 데이터 누수가 자동으로 방지되는 것은 아니다. 전체 개발용 데이터로 전처리를 먼저 학습한 뒤 CV를 수행하면 Validation 정보가 반영될 수 있다.

Scaling, 결측치 대체, Feature 선택 등 학습이 필요한 전처리는 각 Fold의 Train에서만 학습해야 한다. Pipeline을 탐색 객체에 전달하면 전처리와 모델을 Fold별로 함께 학습할 수 있다.

Pipeline의 모델 단계 이름이 `model`이라면 탐색 키는 `model__max_depth`처럼 단계 이름과 파라미터를 이중 밑줄로 연결한다.

최종 흐름은 다음과 같다.

1. 문제와 평가 지표를 정의한다.
2. Hold-out Test를 분리한다.
3. 데이터 특성에 맞는 CV를 정한다.
4. 전처리와 모델, 후보 범위를 구성한다.
5. Grid Search 또는 Random Search로 후보를 평가한다.
6. 평균, 표준편차, 비용을 비교하여 설정을 선택한다.
7. 선택한 설정으로 개발용 Train 전체에 다시 학습한다.
8. Test에서 최종 성능을 평가한다.
9. 모델과 관련 정보를 저장한다.

---

## 14. 핵심 정리

- Parameter는 모델이 학습하는 값이고, Hyperparameter는 학습 전에 정하는 설정이다.
- 튜닝은 여러 설정을 비교하여 대상 데이터에 적절한 값을 선택하는 과정이다.
- CV를 결합하면 단일 Validation 분할에만 의존하지 않고 후보를 비교할 수 있다.
- GridSearchCV는 지정한 모든 조합을, RandomizedSearchCV는 지정한 횟수만큼 후보를 탐색한다.
- CV 학습 횟수는 후보 수와 Fold 수의 곱이며, 기본 refit에서는 최종 재학습 1회가 추가된다.
- best_score_는 선택한 후보의 평균 CV 점수이며 최종 Test 점수가 아니다.
- cv_results_로 평균 점수, 표준편차, 학습 시간과 순위를 함께 확인할 수 있다.
- 기본 refit 설정의 best_estimator_는 개발용 Train 전체로 다시 학습된 모델이다.
- Test는 모델 선택에 사용하지 않고 선택이 끝난 뒤 최종 평가에 사용한다.
- 학습한 모델과 전처리, 설정 정보를 함께 보존하면 재사용과 재현에 도움이 된다.

---

## 오늘의 한 문장

> 하이퍼파라미터 튜닝은 가장 높은 점수 하나를 찾는 작업이 아니라, 교차검증으로 성능과 안정성·비용을 비교하고 독립적인 Test 평가와 모델 저장까지 연결하는 과정이다.