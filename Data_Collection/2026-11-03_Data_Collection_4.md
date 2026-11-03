# TIL | 2026-11-03 — 웹 크롤링과 수집 방법 선택

## 오늘의 학습 주제

정적·동적 크롤링의 차이를 이해하고, HTML 구조와 CSS 선택자로 필요한 데이터를 추출하는 방법을 배웠다. 또한 Next 링크를 따라 여러 페이지의 데이터를 모으고, API와 HTML 크롤링 중 분석 목적에 적합한 방법을 선택하는 기준을 학습했다.

```text
데이터 제공 방식 확인 → 수집 방법 선택 → HTML 요청과 파싱
→ 명언별 필드 추출 → Next 링크 확인과 반복
→ DataFrame 변환 → CSV 저장 및 확인
```

## 1. 정적 크롤링과 동적 크롤링

웹 크롤링은 웹페이지에 접근하여 필요한 데이터를 자동으로 수집하는 과정이다. 브라우저에 보이는 내용이 서버에서 받은 초기 HTML에도 들어 있는지에 따라 수집 방법이 달라진다.

| 구분 | 정적 크롤링 | 동적 크롤링 |
| --- | --- | --- |
| 데이터 확보 | 서버가 반환한 HTML에서 추출 | JavaScript 실행이나 사용자 동작 후 나타난 데이터 수집 |
| 대표 도구 | Requests, BeautifulSoup | Playwright, Selenium 등 |
| 브라우저 실행 | 일반적으로 불필요 | 필요할 수 있음 |
| 구현 복잡도 | 상대적으로 단순 | 상대적으로 복잡할 수 있음 |
| 핵심 확인 사항 | 초기 HTML에 필요한 데이터가 있는가? | 데이터가 언제, 어떻게 추가되는가? |

학습자료에서는 Quotes to Scrape의 일반 페이지와 JavaScript 기반 페이지를 비교했다.

- 일반 페이지: `https://quotes.toscrape.com/`
- JavaScript 기반 페이지: `https://quotes.toscrape.com/js/`

일반 페이지는 초기 HTML에 명언 요소가 포함되어 있어 바로 추출할 수 있다. JavaScript 기반 페이지는 브라우저가 JavaScript를 실행하면서 내용을 구성하므로 초기 HTML에서 같은 요소를 찾지 못할 수 있다.

Requests는 HTML을 가져오지만 브라우저처럼 JavaScript를 실행하지 않는다. BeautifulSoup도 받은 HTML을 분석하는 도구이므로 JavaScript 렌더링을 대신하지 않는다.

### JavaScript 페이지라고 브라우저 자동화가 항상 필요한 것은 아니다

필요한 데이터가 초기 HTML이나 `script` 태그에 있거나, 별도의 JSON 요청으로 제공될 수 있다. 따라서 다음 순서로 확인한다.

1. 필요한 필드를 제공하며 이용 가능한 API가 있는지 확인한다.
2. 초기 HTML에 필요한 데이터가 있으면 정적 크롤링을 검토한다.
3. 요소가 없다면 `script` 태그와 개발자 도구의 Network 탭에서 데이터 제공 방식을 확인한다.
4. 브라우저 실행이나 사용자 동작이 필요할 때 동적 크롤링을 검토한다.

이번 학습은 동적 크롤링의 원리와 선택 기준까지 다루며, Selenium·Playwright를 이용한 자동화 구현은 포함하지 않는다.

### 초기 HTML에 데이터가 있는지 확인하는 예시

```python
import requests
from bs4 import BeautifulSoup

pages = {
    "일반 페이지": "https://quotes.toscrape.com/",
    "JavaScript 페이지": "https://quotes.toscrape.com/js/",
}

for name, url in pages.items():
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        soup = BeautifulSoup(response.text, "html.parser")
        print(name, response.status_code, len(soup.select("div.quote")))
    except requests.RequestException as error:
        print(name, "요청 실패:", error)
```

위 코드는 초기 HTML의 명언 영역 수를 비교하는 예시다. 상태 코드가 200인데 요소를 찾지 못했다고 요청 실패로 단정하면 안 된다. 렌더링 방식, HTML 구조 변경, 선택자 등을 따로 확인해야 한다.

## 2. HTML 구조와 CSS 선택자로 데이터 추출

### 태그·class·계층 관계로 요소 찾기

CSS 선택자는 HTML의 태그, class, id, 계층 관계 등을 이용해 원하는 요소를 선택하는 표현이다.

| 선택자 | 의미 |
| --- | --- |
| `div` | 모든 div 요소 |
| `.quote` | class에 `quote`가 포함된 요소 |
| `div.quote` | class에 `quote`가 포함된 div 요소 |
| `span.text` | 명언 본문 요소 |
| `small.author` | 작성자 요소 |
| `div.tags a.tag` | 태그 영역 안에 있는 태그 링크 |
| `li.next a` | 다음 페이지로 이동하는 링크 |

`div.quote span.text`에서 공백은 앞 요소의 **하위에 있는 요소**를 선택한다는 뜻이다. 반드시 바로 아래의 자식만 의미하지는 않는다.

학습자료의 명언 구조는 다음과 같다.

```text
div.quote — 명언 하나
├─ span.text — 본문
├─ small.author — 작성자
└─ div.tags — 태그 영역
   └─ a.tag — 각 태그
```

### BeautifulSoup의 주요 메서드

| 메서드 | 역할 | 일치하는 요소가 없을 때 |
| --- | --- | --- |
| `select()` | 일치하는 모든 요소 탐색 | 빈 목록 |
| `select_one()` | 첫 번째 일치 요소 탐색 | `None` |
| `get_text(strip=True)` | 텍스트 추출 및 양쪽 공백 제거 | 호출할 요소가 먼저 존재해야 함 |

`select_one()`의 결과가 `None`인데 `get_text()`를 호출하면 오류가 발생한다. 따라서 본문과 작성자 요소가 존재하는지 먼저 확인한다.

### 명언 영역을 기준으로 추출하는 이유

본문 전체와 작성자 전체를 따로 모아 순서대로 연결하면 일부 요소가 누락되었을 때 잘못된 짝이 만들어질 수 있다. 먼저 `div.quote`를 순회하고, **같은 명언 영역 안에서** 본문·작성자·태그를 가져오는 방식이 적절하다.

명언 하나는 Dictionary 하나로 정리하고, 여러 명언은 List에 누적한다.

| 최종 컬럼 | 저장할 내용 |
| --- | --- |
| `text` | 명언 본문 |
| `author` | 작성자 이름 |
| `tags` | 여러 태그를 쉼표로 연결한 문자열 |
| `source_url` | 데이터를 수집한 페이지 URL |

태그가 여러 개여도 명언 하나를 한 행으로 유지한다. 예를 들어 `["life", "thinking"]`은 `", ".join(tags)`를 통해 `"life, thinking"`으로 저장한다.

### CSS 선택자와 XPath 비교

두 방법 모두 문서에서 원하는 요소를 찾는 데 사용한다.

| 구분 | CSS 선택자 | XPath |
| --- | --- | --- |
| 표현 기준 | 태그, class, id, 계층 관계 등 | 문서의 경로와 조건 |
| 예시 | `div.quote span.text` | `//div[@class='quote']//span[@class='text']` |
| 이번 학습에서의 사용 | BeautifulSoup의 `select()` 등 | 개념 비교만 진행 |

위 XPath 예시는 class 속성값이 정확히 `quote`, `text`인 요소를 찾는다. CSS의 class 선택자는 여러 class 중 해당 이름이 포함되어도 일치하므로, 여러 class가 붙은 HTML에서는 두 표현의 결과가 달라질 수 있다.

BeautifulSoup의 `select()`는 CSS 선택자를 사용한다. XPath는 이를 지원하는 다른 도구나 라이브러리에서 사용하며, 이번 추출에는 별도 구현이 필요하지 않다.

## 3. Next 링크를 따라 여러 페이지 수집

API에서는 `page`와 `has_next`로 페이지 이동을 제어했다. HTML에서는 다음 페이지 링크를 찾아 이동한다.

```html
<li class="next">
    <a href="/page/2/">Next</a>
</li>
```

`soup.select_one("li.next a")`로 링크를 찾고 `get("href")`로 주소를 가져온다. Next 링크가 없으면 마지막 페이지로 판단해 반복을 종료한다.

### 상대경로를 전체 URL로 변환하기

`/page/2/`는 도메인의 루트를 기준으로 하는 경로다. `urljoin()`으로 현재 응답 URL과 결합하면 요청에 사용할 전체 URL을 만들 수 있다.

```python
from urllib.parse import urljoin

next_url = urljoin("https://quotes.toscrape.com/", "/page/2/")
# https://quotes.toscrape.com/page/2/
```

문자열을 단순히 붙이기보다 URL의 경로 규칙에 맞게 결합하는 것이 중요하다.

### 전체 수집과 CSV 저장 예시

다음 코드는 학습자료의 수집 흐름을 정리한 것이다. `requests`, `beautifulsoup4`, `pandas`가 필요하며 실행하면 실제 웹 요청을 보낸다. 이 TIL 작성 과정에서 크롤링을 실행한 것은 아니다.

```python
import time
from urllib.parse import urljoin

import pandas as pd
import requests
from bs4 import BeautifulSoup

url = "https://quotes.toscrape.com/"
records = []
page_count = 0
max_pages = 20

while url is not None:
    if page_count >= max_pages:
        raise RuntimeError("페이지 수 상한을 초과했습니다.")

    response = requests.get(url, timeout=10)
    response.raise_for_status()
    soup = BeautifulSoup(response.text, "html.parser")

    quotes = soup.select("div.quote")
    if not quotes:
        raise ValueError("명언 영역을 찾지 못했습니다.")

    for quote in quotes:
        text_element = quote.select_one("span.text")
        author_element = quote.select_one("small.author")

        if text_element is None or author_element is None:
            raise ValueError("필수 HTML 요소가 누락되었습니다.")

        tags = [
            tag.get_text(strip=True)
            for tag in quote.select("div.tags a.tag")
        ]
        records.append({
            "text": text_element.get_text(strip=True),
            "author": author_element.get_text(strip=True),
            "tags": ", ".join(tags),
            "source_url": response.url,
        })

    page_count += 1
    print(f"{page_count}페이지: {len(quotes)}개 / 누적 {len(records)}개")

    next_link = soup.select_one("li.next a")
    if next_link is None:
        url = None
    else:
        href = next_link.get("href")
        if not href:
            raise ValueError("다음 페이지 주소가 없습니다.")
        url = urljoin(response.url, href)
        time.sleep(0.5)

columns = ["text", "author", "tags", "source_url"]
df = pd.DataFrame(records, columns=columns)

file_path = "quotes_html_raw.csv"
df.to_csv(file_path, index=False, encoding="utf-8-sig")

loaded_df = pd.read_csv(
    file_path, encoding="utf-8-sig", keep_default_na=False
)

print("수집 페이지 수:", page_count)
print("전체 명언 수:", len(df))
print("저장 후 명언 수:", len(loaded_df))
print("컬럼:", loaded_df.columns.tolist())
print(loaded_df.head())
```

### 코드에서 기억할 점

- `raise_for_status()`로 HTTP 오류를 확인하고, 명언 영역과 필수 요소도 별도로 확인한다.
- `source_url`에 각 응답의 URL을 기록하면 어느 페이지에서 가져왔는지 추적할 수 있다.
- 현재 페이지의 명언을 모두 추가한 뒤 Next 링크를 확인하므로 마지막 페이지의 데이터도 포함한다.
- `max_pages`는 예상치 못한 반복을 제한하고, 요청 사이에는 간격을 둔다.
- 요청·추출 중 예외가 발생하면 실행이 중단되어 뒤의 CSV 저장 단계로 진행하지 않는다.
- 같은 이름의 이전 파일이 있다면 오류가 나도 남아 있을 수 있으므로 이번 실행의 결과와 구분한다.

페이지 구조가 바뀌어 Next 선택자가 더 이상 맞지 않으면 조기에 종료될 수도 있다. 종료 조건을 충족했다는 사실만으로 전체 데이터의 완전성이 보장되는 것은 아니다.

### CSV 저장 옵션

| 설정 | 역할 |
| --- | --- |
| `columns=columns` | 네 개의 컬럼 순서 명시 |
| `index=False` | 인덱스를 별도 컬럼으로 저장하지 않음 |
| `encoding="utf-8-sig"` | Excel 등에서 한글 인코딩 문제를 줄임 |
| `keep_default_na=False` | 다시 읽을 때 빈 문자열 등을 기본 결측값으로 자동 해석하지 않도록 설정 |

CSV는 현재 작업 폴더에 저장된다. 저장 후에는 행 수, 컬럼명과 순서, 주요 값을 확인한다. `keep_default_na=False`는 빈 값을 검증하거나 정제하는 기능과는 구분해야 한다.

## 4. API와 HTML 크롤링 비교 및 방법 선택

| 구분 | API 수집 | HTML 크롤링 |
| --- | --- | --- |
| 응답 형식 | JSON | HTML |
| 데이터 접근 | Key와 List | CSS 선택자 |
| 페이지 이동 | `page`, `has_next` | Next 링크와 `href` |
| 주요 도구 | Requests | Requests, BeautifulSoup |
| 학습자료의 저장 파일 | `quotes_raw.csv` | `quotes_html_raw.csv` |
| 출처 컬럼 | API 응답 URL | HTML 페이지 URL |

두 방식 모두 `text`, `author`, `tags`, `source_url`이라는 같은 구조로 결과를 만들 수 있다. 하지만 같은 명언을 수집한 결과를 단순히 합치면 중복이 생길 수 있으므로, 이번 학습에서는 방법을 비교하는 용도로 구분한다.

수집 방법은 도구의 종류보다 **분석에 필요한 필드를 안정적으로 확보할 수 있는지**를 기준으로 선택한다. API가 있어도 필요한 필드가 없거나 접근이 제한된다면 다른 방법을 검토해야 한다.

수집 전에는 사이트의 이용약관, robots.txt 등 접근 정책과 요청 제한을 확인하고 필요한 범위에서 요청한다.

## 5. 수집 결과에서 확인할 사항

HTTP 요청 성공, HTML 요소 탐색 성공, 필드 추출 성공, 전체 페이지 수집 완료는 각각 나누어 확인해야 한다.

- 필요한 명언 영역과 필수 요소를 찾았는가?
- 본문과 작성자를 같은 명언 영역에서 가져왔는가?
- 태그와 수집 출처를 올바르게 저장했는가?
- 수집 페이지 수와 레코드 수가 예상과 크게 다르지 않은가?
- CSV를 다시 읽었을 때 행 수와 컬럼 구조가 유지되는가?

CSV를 만들었다고 분석용 데이터셋이 완성되는 것은 아니다. 중복 명언, 빈 본문, 일부 페이지의 누락, 텍스트 분석 적합성은 이후 데이터 품질 검증과 정제 단계에서 확인해야 한다.

## 오늘 배운 점

- 브라우저 화면과 초기 HTML은 다를 수 있으므로 데이터가 실제로 어디에서 제공되는지 먼저 확인해야 한다.
- JavaScript 사용 여부만으로 수집 도구를 결정하지 않고, 초기 HTML과 별도의 데이터 요청을 함께 살펴봐야 한다.
- 반복되는 HTML에서는 레코드의 영역을 먼저 잡고 그 안에서 속성을 추출해야 정보의 연결을 유지하기 쉽다.
- HTML 페이지네이션은 Next 링크와 상대경로를 해석하는 과정이 핵심이다.
- 수집 방식이 달라도 결과 스키마를 통일하면 비교와 후속 처리가 쉬워진다.
- 수집과 저장의 성공은 데이터 품질 검증의 출발점이며, 중복·결측·누락 확인이 뒤따라야 한다.

> 오늘의 핵심: 데이터가 제공되는 구조를 확인한 뒤 적절한 수집 방법을 선택하고, 명언 단위의 연결을 유지하며 여러 페이지의 데이터를 모으는 방법을 익혔다.

## 학습자료

- 2026년 11월 3일 제공된 자료: 정적·동적 크롤링의 차이와 선택 기준 / HTML 구조와 CSS 선택자를 이용한 데이터 추출 / 웹페이지 데이터 수집과 분석 목적에 따른 방법 선택
- 학습 대상: [Quotes to Scrape 일반 페이지](https://quotes.toscrape.com/), [JavaScript 기반 페이지](https://quotes.toscrape.com/js/)
