# 2026-09-30 TIL: Streamlit을 활용한 시계열 데이터 웹 구현

> 학습일: 2026-09-30  
> 주제: Streamlit으로 CSV 업로드부터 ARIMA 미래 예측까지 연결하기

## 학습 목표

- 웹 애플리케이션과 Streamlit의 역할을 설명할 수 있다.
- Streamlit의 기본 컴포넌트로 데이터와 그래프를 출력할 수 있다.
- 사용자의 선택값을 Python 데이터 처리 과정에 연결할 수 있다.
- 웹 화면에서 CSV 파일을 업로드하고 pandas DataFrame으로 불러올 수 있다.
- AirPassengers 데이터를 시계열 분석에 적합한 형태로 변환할 수 있다.
- 사용자가 선택한 기간만큼 ARIMA 미래 예측을 수행하는 웹앱을 구현할 수 있다.

---

## 0. 실습 데이터: AirPassengers

이번 실습에서는 1949년 1월부터 1960년 12월까지 월별 국제선 항공 승객 수를 기록한 `AirPassengers.csv`를 사용한다.

| 항목 | 내용 |
|---|---|
| 데이터 수 | 144개 |
| Frequency | 월별 |
| 기간 | 1949-01 ~ 1960-12 |
| 특징 | 장기적인 증가 추세와 반복적인 계절 패턴 |

원본 CSV는 다음과 같은 구조다.

| 컬럼 | 의미 | 처리 방법 |
|---|---|---|
| `Unnamed: 0` | 각 행의 번호 | 분석에 필요하지 않으므로 제거 |
| `time` | 연도와 소수로 표현된 월별 시점 | 일반적인 날짜 형식으로 변환 |
| `value` | 해당 월의 국제선 항공 승객 수 | 분석 대상 값으로 사용 |

예를 들어 `time` 값은 다음과 같이 날짜를 표현한다.

```text
1949.000000 → 1949년 1월
1949.083333 → 1949년 2월
1949.166667 → 1949년 3월
```

정수 부분은 연도이며, 소수 부분에 12를 곱한 뒤 1을 더하면 월을 계산할 수 있다.

```python
year = data["time"].astype(int)
month = ((data["time"] - year) * 12).round().astype(int) + 1

data["date"] = pd.to_datetime({
    "year": year,
    "month": month,
    "day": 1
})
```

최종적으로 `date`와 `value`만 남기고 날짜순으로 정렬한 뒤 `date`를 인덱스로 사용한다.

```python
data = data[["date", "value"]]
data = data.sort_values("date")
data = data.set_index("date")
```

---

## 1. 웹앱과 Streamlit의 이해

### 1.1 웹 애플리케이션이란?

웹 애플리케이션(Web Application)은 사용자가 웹 브라우저의 화면을 통해 기능을 사용할 수 있도록 만든 프로그램이다.

Python 분석 코드만 제공하면 사용자가 직접 실행 환경과 코드를 다뤄야 한다. 이를 웹앱으로 만들면 사용자는 파일을 올리거나 값을 선택하는 것만으로 분석 결과를 확인할 수 있다.

```text
기존 분석
사용자 → Python 코드 직접 실행 → 결과 확인

데이터 웹앱
사용자 → 웹 화면에서 입력 → 웹앱이 Python 코드 실행 → 화면에 결과 출력
```

### 1.2 Streamlit의 역할

Streamlit은 Python으로 데이터 분석 웹앱을 만들 수 있도록 도와주는 라이브러리다. HTML, CSS, JavaScript를 직접 작성하지 않아도 Python 함수로 화면을 구성하고 사용자 입력을 받을 수 있다.

```python
import streamlit as st

st.title("시계열 데이터 분석")
st.write("Streamlit으로 만든 웹 화면입니다.")
st.dataframe(data)
```

Streamlit이 데이터 분석이나 모델링을 대신하는 것은 아니다.

- pandas: 데이터 불러오기와 전처리
- statsmodels의 ARIMA: 모델 학습과 미래 예측
- Streamlit: 사용자 입력을 받고 데이터·분석 결과를 웹 화면에 표시

즉, 기존 Python 분석 로직에 사용자가 이용할 수 있는 화면을 연결하는 역할을 한다.

### 1.3 첫 Streamlit 앱 실행

uv 환경에서 Streamlit을 설치한다.

```bash
uv add streamlit
```

프로젝트 폴더에 `app.py`를 만든다.

```python
import streamlit as st

st.title("나의 첫 번째 데이터 웹앱")
st.write("안녕하세요.")
st.write("Streamlit으로 만든 웹 화면입니다.")
```

VS Code 터미널에서 다음 명령어로 실행한다.

```bash
uv run streamlit run app.py
```

실행 후 표시되는 Local URL로 접속하면 브라우저에서 앱을 확인할 수 있다. Streamlit 앱은 Jupyter Notebook 셀에서 실행하는 것이 아니라 `.py` 파일로 저장한 뒤 터미널에서 실행한다. Notebook에서 직접 실행하면 `missing ScriptRunContext`와 같은 경고가 나타날 수 있다.

```text
app.py 작성 및 저장
    ↓
터미널에서 Streamlit 실행
    ↓
Local URL 생성
    ↓
브라우저에서 웹앱 확인
```

---

## 2. Streamlit으로 웹 화면 구성하기

### 2.1 기본 컴포넌트

Streamlit에서 화면을 구성하는 요소를 컴포넌트(Component)라고 한다. 코드를 작성한 순서에 따라 컴포넌트가 웹 화면의 위에서 아래로 배치된다.

| 컴포넌트 | 역할 |
|---|---|
| `st.title()` | 웹앱 제목 표시 |
| `st.write()` | 문장, 값, 설명 표시 |
| `st.dataframe()` | pandas DataFrame을 표로 표시 |
| `st.line_chart()` | 데이터를 선 그래프로 표시 |
| `st.selectbox()` | 여러 항목 중 하나를 선택하도록 입력받기 |
| `st.file_uploader()` | 사용자로부터 파일 업로드 받기 |

### 2.2 데이터와 그래프 출력

```python
import pandas as pd
import streamlit as st

data = pd.DataFrame({
    "date": pd.date_range(
        start="2025-01-01",
        periods=6,
        freq="MS"
    ),
    "sales": [123, 128, 132, 129, 137, 139]
})
data = data.set_index("date")

st.title("월별 매출 분석")
st.write("월별 매출 데이터를 확인합니다.")

st.write("월별 매출 데이터")
st.dataframe(data)

st.write("월별 매출 변화")
st.line_chart(data["sales"])
```

시계열의 날짜를 인덱스로 설정하면 `st.line_chart()`가 시간의 흐름에 따라 값을 표시할 수 있다.

### 2.3 사용자 선택값을 데이터 처리에 연결하기

`st.selectbox()`는 사용자가 선택한 값을 Python 변수로 반환한다.

```python
period = st.selectbox(
    "확인할 기간을 선택하세요.",
    [3, 6]
)

recent_data = data.tail(period)

st.write(f"최근 {period}개월 매출")
st.dataframe(recent_data)
st.line_chart(recent_data["sales"])
```

사용자가 3을 선택하면 `period=3`이 되고, `data.tail(3)`이 최근 3개월을 반환한다.

```text
사용자 입력
    ↓
Python 변수에 저장
    ↓
Python 코드로 데이터 처리
    ↓
처리 결과를 웹 화면에 출력
```

이 구조는 이후 예측 기간을 입력받아 ARIMA의 `steps`에 전달할 때도 동일하게 사용된다.

---

## 3. 사용자 입력과 시계열 데이터 불러오기

### 3.1 CSV 파일 업로드

`st.file_uploader()`를 사용하면 사용자가 웹 화면에서 직접 파일을 선택할 수 있다.

```python
import pandas as pd
import streamlit as st

st.title("시계열 데이터 분석")

uploaded_file = st.file_uploader(
    "CSV 파일을 업로드하세요.",
    type="csv"
)

if uploaded_file is not None:
    data = pd.read_csv(uploaded_file)
    st.write("업로드한 원본 데이터")
    st.dataframe(data)
```

- 파일을 선택하지 않으면 `uploaded_file`은 `None`이다.
- 파일을 선택하면 업로드된 파일 객체가 저장된다.
- `type="csv"`는 선택할 수 있는 파일을 CSV로 제한한다.
- 업로드가 완료된 경우에만 `pd.read_csv()`를 실행해야 한다.

기존 Python 코드에서는 파일 경로를 전달하지만, Streamlit에서는 사용자가 업로드한 객체를 전달한다.

```python
# 로컬 파일 경로 사용
data = pd.read_csv("AirPassengers.csv")

# Streamlit 업로드 파일 사용
data = pd.read_csv(uploaded_file)
```

### 3.2 시계열 분석용 데이터로 전처리

```python
if uploaded_file is not None:
    data = pd.read_csv(uploaded_file)

    # 불필요한 행 번호 제거
    data = data.drop(columns=["Unnamed: 0"])

    # 소수 형태의 time을 연도와 월로 분리
    year = data["time"].astype(int)
    month = ((data["time"] - year) * 12).round().astype(int) + 1

    # 월의 첫날을 기준으로 날짜 생성
    data["date"] = pd.to_datetime({
        "year": year,
        "month": month,
        "day": 1
    })

    # 필요한 컬럼만 선택하고 시간순으로 정리
    data = data[["date", "value"]]
    data = data.sort_values("date")
    data = data.set_index("date")

    st.write("시계열 분석용 데이터")
    st.dataframe(data)

    st.write("시간에 따른 데이터 변화")
    st.line_chart(data["value"])
```

전체 데이터 흐름은 다음과 같다.

```text
CSV 업로드
    ↓
pd.read_csv()
    ↓
불필요한 컬럼 제거
    ↓
time을 연도·월로 변환
    ↓
date 컬럼 생성
    ↓
date, value만 선택
    ↓
날짜순 정렬 및 인덱스 설정
    ↓
표와 선 그래프로 출력
```

Streamlit이 별도의 전처리 방법을 제공하는 것은 아니다. pandas로 데이터를 정리하고 Streamlit으로 결과를 화면에 표시하는 구조다.

---

## 4. ARIMA 예측 기능 연결하기

### 4.1 이번 실습의 예측 방식

이전의 모델 평가에서는 데이터를 Train과 Test로 나누고, Test의 Actual과 Predicted를 비교해 MAE와 RMSE를 계산했다.

이번 웹앱에서는 업로드된 전체 데이터가 현재까지 관측된 데이터라고 가정한다. 전체 데이터로 ARIMA를 학습하고 마지막 관측 시점 이후의 실제 미래를 예측한다.

| 구분 | 모델 평가 | 이번 웹앱의 실제 미래 예측 |
|---|---|---|
| 학습 데이터 | Train 구간 | 업로드된 전체 데이터 |
| 예측 대상 | Actual을 숨겨둔 Test 구간 | 아직 관측되지 않은 미래 |
| Actual | 존재함 | 아직 없음 |
| 즉시 계산 가능한 결과 | 예측값, 오차, MAE, RMSE | 미래 예측값 |

미래의 실제값이 아직 없으므로 예측 직후 MAE나 RMSE를 계산할 수 없다. 이번 실습에서는 사용자가 선택한 기간의 미래 예측값을 확인하는 데 집중한다.

### 4.2 예측 기간 입력과 ARIMA 연결

```python
forecast_period = st.selectbox(
    "예측 기간을 선택하세요.",
    [1, 3, 6]
)

model = ARIMA(
    data["value"],
    order=(2, 1, 1)
)
model_fit = model.fit()

forecast = model_fit.forecast(
    steps=forecast_period
)

st.write(f"앞으로 {forecast_period}개월 예측 결과")
st.write(forecast)
```

사용자가 3개월을 선택하면 `forecast_period=3`이 되고, `forecast(steps=3)`을 통해 마지막 관측 시점 이후 3개 월을 예측한다.

이 실습에서는 Streamlit과 시계열 모델을 연결하는 과정에 집중하기 위해 ARIMA의 구조를 `order=(2,1,1)`로 고정한다.

```text
예측 기간 선택
    ↓
forecast_period 변수
    ↓
전체 시계열로 ARIMA(2,1,1) 학습
    ↓
forecast(steps=forecast_period)
    ↓
웹 화면에 미래 예측값 출력
```

---

## 5. 통합 웹앱 코드

다음 코드는 PDF에 포함된 흐름을 하나로 합친 `app.py` 예시다.

```python
import pandas as pd
import streamlit as st
from statsmodels.tsa.arima.model import ARIMA

st.title("시계열 데이터 예측")
st.write(
    "AirPassengers CSV를 업로드하고 "
    "미래 항공 승객 수를 예측합니다."
)

uploaded_file = st.file_uploader(
    "CSV 파일을 업로드하세요.",
    type="csv"
)

if uploaded_file is not None:
    # 1. CSV 불러오기
    data = pd.read_csv(uploaded_file)

    st.write("업로드한 원본 데이터")
    st.dataframe(data)

    # 2. 시계열 분석용 데이터로 변환
    data = data.drop(columns=["Unnamed: 0"])

    year = data["time"].astype(int)
    month = ((data["time"] - year) * 12).round().astype(int) + 1

    data["date"] = pd.to_datetime({
        "year": year,
        "month": month,
        "day": 1
    })

    data = data[["date", "value"]]
    data = data.sort_values("date")
    data = data.set_index("date")

    # 3. 정리된 데이터 출력
    st.write("시계열 분석용 데이터")
    st.dataframe(data)

    st.write("시간에 따른 승객 수 변화")
    st.line_chart(data["value"])

    # 4. 사용자로부터 예측 기간 입력
    forecast_period = st.selectbox(
        "예측 기간을 선택하세요.",
        [1, 3, 6]
    )

    # 5. 전체 데이터로 ARIMA 학습 및 미래 예측
    model = ARIMA(
        data["value"],
        order=(2, 1, 1)
    )
    model_fit = model.fit()

    forecast = model_fit.forecast(
        steps=forecast_period
    )

    # 6. 예측 결과 출력
    st.write(f"앞으로 {forecast_period}개월 예측 결과")
    st.write(forecast)
```

실행 명령어는 다음과 같다.

```bash
uv run streamlit run app.py
```

### 통합 실행 흐름

```text
AirPassengers.csv 업로드
    ↓
pandas로 데이터 불러오기
    ↓
time을 date로 변환하고 시간순 정렬
    ↓
데이터 표와 시계열 그래프 출력
    ↓
사용자가 예측 기간 선택
    ↓
전체 데이터로 ARIMA(2,1,1) 학습
    ↓
선택한 개월 수만큼 미래 예측
    ↓
웹 화면에 예측값 출력
```

---

## 주요 컴포넌트 정리

| 코드 | 입력 | 반환·결과 |
|---|---|---|
| `st.title(text)` | 제목 문자열 | 웹 화면에 큰 제목 표시 |
| `st.write(value)` | 문장, 변수, 객체 | 값을 웹 화면에 표시 |
| `st.dataframe(df)` | DataFrame | 스크롤 가능한 표 표시 |
| `st.line_chart(series)` | Series 또는 DataFrame | 선 그래프 표시 |
| `st.selectbox(label, options)` | 설명과 선택지 | 사용자가 선택한 값 반환 |
| `st.file_uploader(label, type)` | 설명과 파일 형식 | 업로드 파일 객체 또는 `None` 반환 |

## 구현할 때 확인할 점

1. Streamlit 코드는 Notebook이 아니라 `.py` 파일에 작성하고 터미널에서 실행한다.
2. 업로드된 파일이 없을 때는 `None`이므로 조건문 안에서만 읽는다.
3. 시계열은 날짜 자료형으로 변환하고 반드시 시간순으로 정렬한다.
4. 모델 입력값은 숫자형 시계열이어야 한다.
5. 이번 예측은 업로드된 전체 데이터를 이용한 실제 미래 예측이므로 즉시 MAE·RMSE를 계산하지 않는다.
6. 사용자의 입력값은 일반 Python 변수이므로 데이터 처리와 모델 매개변수에 연결할 수 있다.
7. `order=(2,1,1)`은 실습용 고정값이며 실제 분석에서는 별도의 모델 선택과 성능 검증이 필요하다.

## 핵심 정리

1. Streamlit은 기존 Python 데이터 처리와 모델링 코드를 웹 화면에 연결한다.
2. 화면 컴포넌트는 Python 코드에 작성한 순서대로 위에서 아래로 배치된다.
3. `st.selectbox()`의 선택 결과는 Python 변수로 받아 후속 처리에 사용할 수 있다.
4. `st.file_uploader()`와 `pd.read_csv()`를 연결하면 사용자가 올린 CSV를 분석할 수 있다.
5. AirPassengers의 소수형 `time`은 연도와 월을 계산해 `datetime`으로 변환한다.
6. 날짜순 정렬과 날짜 인덱스 설정은 시계열 처리와 시각화의 기본이다.
7. 전체 시계열로 ARIMA를 학습한 뒤 선택한 Forecast Horizon만큼 실제 미래를 예측한다.
8. 아직 미래의 Actual이 없으므로 예측 직후에는 오차, MAE, RMSE를 계산할 수 없다.

## 오늘의 한 문장

> Streamlit은 pandas와 ARIMA로 만든 분석 로직에 파일 업로드, 사용자 선택, 결과 출력을 연결하여 비개발자도 사용할 수 있는 데이터 웹앱으로 바꿔준다.