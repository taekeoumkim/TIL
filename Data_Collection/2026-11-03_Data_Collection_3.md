# TIL | 2026-11-03 — API를 활용한 데이터 수집

## 오늘의 학습 주제

API의 요청·응답 흐름을 이해하고, Python으로 JSON 데이터를 받아 필요한 필드를 추출하는 방법을 배웠다. 이어서 페이지네이션으로 여러 응답을 모아 DataFrame과 CSV 파일로 구성하는 과정을 학습했다.

```text
GET 요청 → HTTP 응답 확인 → JSON 파싱 → 필드 추출
→ 다음 페이지 확인과 반복 → DataFrame 생성 → CSV 저장 및 확인
```

## 1. API의 기본 개념과 요청·응답 흐름

### API, Client, Server

API(Application Programming Interface)는 프로그램이 다른 프로그램의 데이터나 기능에 정해진 방식으로 접근할 수 있게 하는 인터페이스다.

이번 학습에서는 Python 프로그램이 요청을 보내는 **Client**, Quotes to Scrape의 웹 서버가 요청을 처리하고 응답하는 **Server**에 해당한다. HTTP는 이들이 요청과 응답을 주고받는 통신 규약이다.

```text
Client(Python) ── HTTP 요청 ──▶ Server
Client(Python) ◀── HTTP 응답 ── Server
```

GET은 서버의 데이터나 자원을 조회할 때 사용하는 HTTP 요청 방식이다. 이번에는 명언을 JSON으로 제공하는 경로에 GET 요청을 보내는 흐름을 학습했다.

### API URL의 구조

학습자료에서 사용한 요청 주소는 다음과 같다.

```text
https://quotes.toscrape.com/api/quotes?page=1
```

| 구성 요소 | 값 | 역할 |
| --- | --- | --- |
| 프로토콜 | `https` | 통신 방식 |
| 도메인 | `quotes.toscrape.com` | 요청할 서버 주소 |
| 경로(Path) | `/api/quotes` | 요청할 데이터의 경로 |
| 쿼리 파라미터 | `page=1` | 요청할 페이지 지정 |

쿼리 파라미터는 `?` 뒤에 작성하며, 여러 조건은 `&`로 구분한다. 사용할 수 있는 이름과 값은 API마다 다르다. Requests에서는 `params={"page": 1}`처럼 전달할 수 있다.

### HTTP 응답에서 확인할 것

| 항목 | 의미 | 확인 방법 |
| --- | --- | --- |
| Status Code | 요청 처리 결과 | `response.status_code` |
| Headers | 응답 형식 등의 부가 정보 | `response.headers` |
| Body | 실제 응답 데이터 | JSON 응답은 `response.json()`으로 파싱 |

`response.headers.get("Content-Type")`으로 응답 형식을 확인할 수 있다. 학습자료의 JSON 응답은 `application/json` 형식이다.

| 상태 코드 | 의미 | 확인할 사항 |
| --- | --- | --- |
| `200` | 정상 처리 | 실제 데이터와 필요한 필드도 확인 |
| `403` | 접근 거부 | 접근 권한과 사이트 정책 |
| `404` | 자원을 찾을 수 없음 | URL과 요청 경로 |
| `429` | 요청 횟수 제한 | 요청 빈도와 서비스 정책 |
| `500` | 서버 내부 오류 | 서버 측 문제 여부 |

**상태 코드 200은 HTTP 요청이 정상 처리되었다는 뜻이지, 필요한 데이터를 모두 확보했다는 뜻은 아니다.** 목록이 비어 있거나 필수 필드가 누락될 수 있다.

## 2. Python으로 API 호출 및 JSON 파싱

### 요청과 응답 확인

```python
import requests

url = "https://quotes.toscrape.com/api/quotes"

try:
    response = requests.get(url, params={"page": 1}, timeout=10)
    print("요청 URL:", response.url)
    print("상태 코드:", response.status_code)
    response.raise_for_status()
    print("응답 형식:", response.headers.get("Content-Type"))
    data = response.json()
except requests.exceptions.JSONDecodeError as error:
    print("JSON 파싱 실패:", error)
except requests.RequestException as error:
    print("API 요청 실패:", error)
```

- `timeout=10`: 연결이나 응답 읽기가 무기한 대기하지 않도록 제한한다. 전체 수집 작업을 10초로 제한한다는 뜻은 아니다.
- `raise_for_status()`: 400번대·500번대 HTTP 오류 상태에서 예외를 발생시킨다.
- `response.json()`: 응답에 담긴 JSON을 Python 자료형으로 변환한다.
- `response.url`: 쿼리 파라미터 등이 반영된 응답의 URL을 확인하는 데 사용한다.

JSON 파싱 실패와 요청 실패를 나누어 확인하면 어느 단계에 문제가 생겼는지 파악하기 쉽다.

### `json.loads()`와 `response.json()`

| 방법 | 사용 상황 |
| --- | --- |
| `json.loads(json_text)` | 가지고 있는 JSON 문자열을 직접 파싱 |
| `response.json()` | HTTP 응답에 포함된 JSON을 파싱 |

JSON Object는 Python의 Dictionary, Array는 List로 변환된다. JSON의 `true`와 `false`는 Python의 `True`와 `False`에 대응한다.

### 응답 구조와 필드 추출

학습자료의 응답 구조를 단순화하면 다음과 같다.

```json
{
  "page": 1,
  "has_next": true,
  "quotes": [
    {
      "text": "예시 명언입니다.",
      "author": {"name": "작성자 A"},
      "tags": ["life", "thinking"]
    }
  ]
}
```

| 정보 | 접근 방법 | 용도 |
| --- | --- | --- |
| 현재 페이지 | `data["page"]` | 수집 상태 확인 |
| 다음 페이지 여부 | `data["has_next"]` | 반복 제어 |
| 명언 목록 | `data["quotes"]` | 각 명언 순회 |
| 명언 본문 | `quote["text"]` | 텍스트 분석 |
| 작성자 이름 | `quote["author"]["name"]` | 작성자 구분 |
| 태그 | `quote["tags"]` | 주제 구분 |

작성자는 중첩된 Dictionary 안의 `name`까지 접근해야 한다. 태그는 CSV 저장을 고려해 `", ".join(quote["tags"])`로 연결한다.

```python
# 요청과 JSON 파싱이 성공한 뒤 실행하는 추출 부분
records = []

for quote in data["quotes"]:
    records.append({
        "text": quote["text"],
        "author": quote["author"]["name"],
        "tags": ", ".join(quote["tags"]),
        "source_url": response.url,
    })

print("추출한 명언 수:", len(records))
```

명언 하나를 Dictionary로, 여러 명언을 List로 정리한다. `source_url`에는 해당 데이터를 가져온 URL을 기록하여 출처를 추적할 수 있게 한다.

### 단계별 성공 여부 구분

| 확인 단계 | 발생할 수 있는 문제 |
| --- | --- |
| HTTP 요청 | 연결 오류, Timeout, HTTP 오류 |
| JSON 파싱 | JSON이 아닌 응답 또는 잘못된 JSON |
| 구조 확인 | `quotes`나 필수 Key 누락, 예상과 다른 자료형 |
| 데이터 추출 | 빈 목록, 빈 본문 등 |

`response.json()`이 성공해도 예상한 구조를 보장하지 않는다. 상태 코드, JSON 구조, 추출 건수와 주요 값을 각각 확인해야 한다.

## 3. 페이지네이션으로 여러 응답 수집

페이지네이션(Pagination)은 많은 데이터를 여러 페이지로 나누어 제공하는 방식이다. 전체 데이터를 모으려면 페이지별로 요청하고 결과를 누적해야 한다.

이번 학습에서는 `page`로 요청할 페이지를 지정하고, `has_next`로 종료 여부를 판단했다.

1. `page=1`부터 요청한다.
2. 현재 페이지의 명언을 추출해 `records`에 추가한다.
3. `has_next`가 `False`이면 종료한다.
4. `True`이면 페이지 번호를 증가시키고 다음 요청을 보낸다.

**마지막 페이지의 데이터도 먼저 추가한 뒤 종료 조건을 확인해야 한다.** 모든 페이지에서 같은 Dictionary 구조를 사용하면 전체 결과를 하나의 테이블로 만들 수 있다.

### 반복 수집 시 기억할 설정

- `max_pages`: 예상치 못한 무한 반복을 막기 위한 페이지 수 상한이다.
- `time.sleep(0.5)`: 다음 요청 전 짧은 간격을 둔다. 실제 요청 간격은 서비스 정책에 맞춰야 한다.
- 오류 발생 시 중단: 일부 페이지만 수집한 결과를 전체 수집 결과로 판단하지 않는다.
- 정상 종료 후 저장: 마지막 페이지까지 처리했을 때 CSV를 저장한다.

403이나 429 등이 발생하면 같은 요청을 무조건 반복하지 않고 접근 정책과 요청 빈도를 확인한다.

## 4. 여러 페이지 수집부터 CSV 저장까지

다음은 학습자료의 전체 흐름을 정리한 예시다. `requests`와 `pandas`가 필요하며, 실행 시 API에 실제 요청을 보낸다. 이 TIL을 작성하면서 해당 API 수집을 실행한 것은 아니다.

```python
import time

import pandas as pd
import requests

url = "https://quotes.toscrape.com/api/quotes"
records = []
page = 1
max_pages = 20

try:
    while True:
        if page > max_pages:
            raise RuntimeError("페이지 수 상한을 초과했습니다.")

        response = requests.get(
            url, params={"page": page}, timeout=10
        )
        response.raise_for_status()
        data = response.json()
        quotes = data["quotes"]

        if not isinstance(quotes, list) or not quotes:
            raise ValueError("명언 목록이 없거나 형식이 잘못되었습니다.")

        for quote in quotes:
            records.append({
                "text": quote["text"],
                "author": quote["author"]["name"],
                "tags": ", ".join(quote["tags"]),
                "source_url": response.url,
            })

        print(f"페이지 {page}: {len(quotes)}개 / 누적 {len(records)}개")

        if not data["has_next"]:
            break

        page += 1
        time.sleep(0.5)

except (requests.RequestException, ValueError, KeyError,
        TypeError, RuntimeError) as error:
    print("데이터 수집 중단:", error)
    print("이번 실행의 CSV를 저장하지 않았습니다.")

else:
    columns = ["text", "author", "tags", "source_url"]
    df = pd.DataFrame(records, columns=columns)
    file_path = "quotes_raw.csv"

    df.to_csv(file_path, index=False, encoding="utf-8-sig")
    loaded_df = pd.read_csv(file_path, encoding="utf-8-sig")

    print("전체 수집 건수:", len(df))
    print("저장 후 행 수:", len(loaded_df))
    print("컬럼:", loaded_df.columns.tolist())
    print(loaded_df.head())
```

이 예제는 학습자료처럼 비어 있는 `quotes`를 오류로 처리하고, `has_next`가 거짓일 때 반복을 종료한다. 모든 필드의 자료형과 데이터 품질을 완전하게 검증하는 코드는 아니다.

### 저장과 기본 확인

| 항목 | 의미 |
| --- | --- |
| `columns` 지정 | `text`, `author`, `tags`, `source_url` 순서 유지 |
| `index=False` | CSV에 불필요한 인덱스 컬럼 추가 방지 |
| `encoding="utf-8-sig"` | Excel 등에서 한글 인코딩 문제 완화 |
| `read_csv()` | 저장된 파일을 다시 읽어 결과 확인 |

`quotes_raw.csv`는 현재 작업 폴더에 저장된다. 수집 건수는 실행 시점의 응답에 따라 달라지므로 고정된 숫자로 단정하지 않는다.

수집이 중간에 실패해도 이전 실행에서 만든 같은 이름의 파일은 남아 있을 수 있다. 파일이 존재한다는 사실만으로 이번 실행이 성공했다고 판단하면 안 된다. 또한 위 예제에서 파일 쓰기·읽기 오류는 수집 오류 처리 범위와 별개다.

## 5. 수집 완료와 데이터 품질 검증 구분

이번 단계에서는 다음을 기본적으로 확인한다.

- `has_next`가 `False`인 마지막 페이지까지 처리했는가?
- 전체 수집 건수는 얼마인가?
- 네 개의 컬럼이 예상한 순서로 존재하는가?
- CSV를 저장하고 다시 읽었을 때 행 수와 구조가 유지되는가?

이 확인만으로 중복·결측·누락이 모두 해결되는 것은 아니다. 동일한 명언의 중복 여부, 빈 본문, 일부 데이터의 누락, 텍스트 분석 적합성 등은 이후 정제 단계에서 검증해야 한다.

`quotes_raw.csv`는 수집 결과를 테이블로 구성한 원시 데이터이고, 정제까지 완료한 최종 분석 데이터와는 구분한다.

## 오늘 배운 점

- API 수집은 요청을 보내는 것에서 끝나지 않고 응답과 추출 결과까지 확인해야 한다.
- `raise_for_status()`는 HTTP 오류 확인, `response.json()`은 JSON 파싱을 담당하므로 역할을 구분해서 이해해야 한다.
- 중첩 JSON에서는 필요한 Key의 경로를 정확히 따라가고, 추출 결과를 일관된 구조로 정리해야 한다.
- 여러 페이지를 수집할 때는 데이터 누적뿐 아니라 종료 조건과 실패 시 처리도 중요하다.
- 마지막 페이지까지 수집한 것과 데이터 품질이 보장되는 것은 별개이므로, 저장 후 검증과 정제 과정을 이어가야 한다.

> 오늘의 핵심: 요청 성공, JSON 파싱 성공, 데이터 추출 성공, 전체 페이지 수집 완료를 구분하여 확인하는 방법을 배웠다.