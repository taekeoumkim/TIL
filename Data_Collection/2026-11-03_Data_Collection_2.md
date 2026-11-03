# TIL | 2026-11-03

## 오늘의 학습 주제

**분석 목적에 맞는 필드를 설계하고, HTML·JSON에서 추출한 데이터를 Pandas DataFrame으로 변환하여 CSV로 저장하는 과정**을 배웠다.

전체 흐름은 다음과 같다.

```text
분석 목적 정의 → 필드와 행 단위 설계 → HTML·JSON 파싱
→ 필요한 값 추출 → Dictionary의 List로 정리
→ DataFrame 생성 → CSV 저장 → 다시 읽어 검증
```

## 1. 분석 목적에 필요한 필드 선택

### 데이터를 추출하기 전에 구조부터 정하기

원본에 있는 모든 정보를 저장할 필요는 없다. 어떤 분석을 할지 먼저 정하고, 그 분석에 필요한 필드를 선택해야 한다.

| 분석 목적 | 필요한 필드 |
| --- | --- |
| 명언에 자주 등장하는 단어 확인 | `text` |
| 작성자별 명언 비교 | `text`, `author` |
| 태그별 명언 분류 | `text`, `tags` |
| 수집 출처 추적 | `source_url` |

페이지 번호나 다음 페이지 존재 여부는 수집 과정에서 필요할 수 있지만, 최종 텍스트 분석용 테이블에 반드시 포함해야 하는 것은 아니다.

### 핵심 용어

| 용어 | 의미 |
| --- | --- |
| 필드(Field) | 본문, 작성자처럼 하나의 속성을 나타내는 항목 |
| 레코드(Record) | 하나의 대상을 설명하는 정보의 묶음 |
| 스키마(Schema) | 컬럼명과 자료형 등 데이터 구조의 정의 |
| 필수 필드(Required Field) | 해당 분석을 수행하는 데 반드시 필요한 항목 |
| 메타데이터(Metadata) | 출처나 수집 과정 등을 설명하는 부가 정보 |

### 한 행은 명언 하나

이번 데이터셋에서는 **하나의 명언을 하나의 행으로 유지**한다. 태그가 여러 개여도 명언을 여러 행으로 나누지 않는다.

| 최종 컬럼 | 의미 | 값의 형태 |
| --- | --- | --- |
| `text` | 명언 본문 | 문자열 |
| `author` | 작성자 이름 | 문자열 |
| `tags` | 관련 태그 | 추출 시 List, 저장 시 문자열 |
| `source_url` | 수집 출처 | 문자열 |

원본 JSON의 `author`가 중첩된 객체여도 최종 테이블에는 이름만 꺼내어 `author` 컬럼에 저장할 수 있다. `source_url`처럼 수집 프로그램에서 추가하는 컬럼도 있다.

```python
quote["text"]            # 본문
quote["author"]["name"]  # 중첩된 객체 안의 작성자 이름
quote["tags"]            # 태그 목록
```

필수 여부는 분석 목적에 따라 달라진다. 본문이 없으면 텍스트 분석이 어렵고, 작성자가 없으면 작성자별 비교에 제한이 생긴다. 구체적인 결측값 처리는 이후 정제 단계에서 다룬다.

## 2. HTML과 JSON에서 값 추출

파싱은 원시 데이터의 구조를 해석하여 필요한 부분에 접근할 수 있게 만드는 과정이다. HTML과 JSON은 구조가 다르므로 접근 방법도 다르다.

| 구분 | HTML | JSON |
| --- | --- | --- |
| 구조 해석 | `BeautifulSoup()` | `json.loads()` |
| 데이터 접근 | 태그와 CSS 선택자 | Key와 List 인덱스 |
| 명언 목록 | `select("div.quote")` | `data["quotes"]` |
| 작성자 | `select_one("small.author")` | `quote["author"]["name"]` |
| 태그 목록 | `select("a.tag")` | `quote["tags"]` |

### HTML: 명언 영역 안에서 필요한 요소 찾기

- `select_one()`은 선택자와 일치하는 첫 번째 요소를 찾는다.
- `select()`는 선택자와 일치하는 모든 요소를 찾는다.
- `get_text(strip=True)`는 요소의 텍스트를 가져오고 양쪽 공백을 제거한다.

먼저 `div.quote`로 명언 하나의 범위를 정한 뒤, **해당 영역 안에서** 본문·작성자·태그를 추출한다. 이렇게 해야 서로 다른 명언의 정보가 잘못 연결되는 일을 줄일 수 있다.

```python
from bs4 import BeautifulSoup

# html_text는 확보한 HTML 문자열이다.
soup = BeautifulSoup(html_text, "html.parser")
html_records = []

for quote in soup.select("div.quote"):
    tags = [
        tag.get_text(strip=True)
        for tag in quote.select("a.tag")
    ]
    html_records.append({
        "text": quote.select_one("span.text").get_text(strip=True),
        "author": quote.select_one("small.author").get_text(strip=True),
        "tags": ", ".join(tags),
        "source_url": "https://quotes.toscrape.com/",
    })
```

위 코드는 필요한 요소가 존재한다는 가정의 기본 예시다. 실제 수집에서는 요소 누락이나 추출 실패 여부도 확인해야 한다.

### JSON: Key와 인덱스로 값에 접근하기

`json.loads()`는 JSON 문자열을 Python 객체로 변환한다. JSON의 Object는 Dictionary로, Array는 List로 변환되므로 중첩된 Key와 인덱스로 값에 접근할 수 있다.

```python
import json

# json_text는 확보한 JSON 문자열이다.
data = json.loads(json_text)
json_records = []

for quote in data["quotes"]:
    json_records.append({
        "text": quote["text"],
        "author": quote["author"]["name"],
        "tags": ", ".join(quote["tags"]),
        "source_url": "https://quotes.toscrape.com/api/quotes?page=1",
    })
```

`source_url`에는 실제 요청에 사용한 URL을 기록한다. 위 URL은 학습자료의 예시이며, 이 코드는 네트워크 요청이 아니라 이미 확보한 문자열의 추출 과정을 보여 준다.

### 원본 형식이 달라도 결과 구조는 통일하기

명언 하나는 Dictionary, 여러 명언은 Dictionary를 담은 List로 구성한다. HTML과 JSON 모두 컬럼명과 값의 형태를 맞추면 같은 구조의 DataFrame으로 변환할 수 있다.

```python
tags = ["life", "thinking"]
tags_text = ", ".join(tags)
# "life, thinking"
```

HTML과 JSON이 동일한 명언을 표현하는 예시라면 `text`, `author`, `tags`를 비교할 수 있다. 수집 경로가 다르면 `source_url`은 달라질 수 있으므로 주요 내용과 구분하여 확인한다.

## 3. DataFrame 변환과 CSV 저장

### Dictionary의 List를 테이블로 변환하기

`pd.DataFrame()`으로 변환하면 각 Dictionary는 한 행이 되고, Key는 컬럼명, Value는 해당 행의 값이 된다. `columns`를 지정하면 원하는 컬럼 순서도 명시할 수 있다.

다음은 가상 데이터 두 건으로 저장과 재확인 흐름을 정리한 예시다.

```python
from pathlib import Path
import pandas as pd

records = [
    {
        "text": "첫 번째 예시 명언입니다.",
        "author": "작성자 A",
        "tags": "life, thinking",
        "source_url": "https://quotes.toscrape.com/",
    },
    {
        "text": "두 번째 예시 명언입니다.",
        "author": "작성자 B",
        "tags": "learning",
        "source_url": "https://quotes.toscrape.com/",
    },
]

columns = ["text", "author", "tags", "source_url"]
df = pd.DataFrame(records, columns=columns)

print(df.head())
print(df.shape)  # (2, 4)
print(df.columns.tolist())
df.info()

file_path = Path("quotes_example.csv")
df.to_csv(file_path, index=False, encoding="utf-8-sig")

loaded_df = pd.read_csv(file_path, encoding="utf-8-sig")

print("행 수 일치:", len(df) == len(loaded_df))
print("컬럼 일치:", df.columns.tolist() == loaded_df.columns.tolist())
print("데이터 일치:", df.equals(loaded_df))
```

### 저장 옵션과 확인 방법

| 기능·설정 | 역할 |
| --- | --- |
| `head()` | 앞부분의 데이터 확인 |
| `shape` | 행과 열 개수 확인 |
| `columns` | 컬럼명과 순서 확인 |
| `info()` | 자료형과 기본 정보 확인 |
| `index=False` | 인덱스가 별도 CSV 컬럼으로 저장되는 것을 방지 |
| `encoding="utf-8-sig"` | Excel 등에서 한글 인코딩 문제를 줄이기 위한 설정 |
| `read_csv()` | 저장한 CSV를 다시 DataFrame으로 읽기 |
| `equals()` | 자료형을 포함해 두 DataFrame의 동일성 확인 |

상대 경로로 지정한 CSV는 코드를 실행하는 현재 작업 폴더에 저장된다. 파일이 만들어졌는지만 보지 말고, 다시 읽어 행 수·컬럼·내용이 유지되는지 확인해야 한다.

CSV는 자료형을 완전하게 보존하지 않는다. 숫자·날짜·결측값 등이 있는 실제 데이터에서는 다시 읽은 뒤 자료형과 주요 값도 점검해야 한다.

## 4. 수집 결과와 정제 결과 구분

HTML과 JSON으로 같은 명언을 각각 수집한 뒤 단순히 합치면 중복 데이터가 생길 수 있다. 예를 들어 같은 명언 10개를 두 방식으로 수집해 합치면 20행이 되지만, 서로 다른 명언이 20개라는 뜻은 아니다.

이번 학습에서는 두 형식을 추출 방식의 비교 대상으로 이해하고, 실제 수집에서는 선택한 방식으로 데이터를 확보한다.

| 파일 | 역할 |
| --- | --- |
| `quotes_example.csv` | 가상 데이터로 저장 흐름을 확인하는 예제 파일 |
| `quotes_raw.csv` | 실제 수집한 값을 기본 테이블로 정리한 파일 |
| `quotes_clean.csv` | 결측·중복 등 데이터 품질을 확인하고 정제한 파일 |

테이블로 만들었다고 정제가 끝난 것은 아니다. 본문 누락, 중복 명언, 수집 실패나 누락, 필수 컬럼 구성 등을 별도로 확인해야 한다. 실제 수집과 구체적인 정제 방법은 이후 학습에서 다룰 내용이다.

## 오늘 배운 점

- 데이터 수집의 시작은 분석 목적과 최종 데이터 구조를 정하는 일이다.
- 원본 구조를 그대로 옮기기보다, 분석에 필요한 값을 일관된 컬럼으로 정리해야 한다.
- HTML은 요소의 범위를 기준으로, JSON은 Key와 인덱스를 기준으로 접근한다.
- 추출 방식이 달라도 Dictionary의 List라는 공통 구조로 맞추면 DataFrame으로 연결하기 쉽다.
- CSV 저장 후에는 다시 읽어 구조와 값을 검증해야 하며, 수집과 정제는 구분해서 생각해야 한다.

> 오늘의 핵심: 분석 목적에 맞게 데이터를 설계하고, 필요한 값을 추출한 뒤, 저장 결과까지 확인하는 흐름을 익혔다.