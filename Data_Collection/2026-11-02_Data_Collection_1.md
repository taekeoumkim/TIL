# 2026-11-03 TIL | HTML의 구조와 태그, JSON의 Key·Value·Array, 분석용 데이터 구조 설계

## 1. 웹 데이터 수집에 HTML·JSON 구조가 필요한 이유

웹의 데이터는 처음부터 CSV나 DataFrame 형태로 정리되어 있지 않을 수 있다. 필요한 정보가 HTML 요소나 중첩된 JSON의 여러 위치에 나뉘어 있으므로, 먼저 **데이터 하나의 경계와 필요한 필드의 위치**를 파악해야 한다.

학습자료에서는 [Quotes to Scrape](https://quotes.toscrape.com/)의 명언 데이터를 예시로 사용한다.

| 수집할 정보 | 분석용 컬럼 |
| --- | --- |
| 명언 본문 | `text` |
| 작성자 이름 | `author` |
| 관련 태그 | `tags` |
| 수집 출처 | `source_url` |

이 문서의 명언과 작성자 이름은 구조를 설명하기 위한 가상 데이터다.

## 2. HTML의 태그·속성·계층 구조

HTML(HyperText Markup Language)은 웹페이지의 내용과 구조를 표현하는 마크업 언어다.

### 태그와 요소 구분

`<p>데이터 수집</p>`처럼 일반적인 요소는 시작 태그, 내용, 종료 태그로 구성된다.

| 구성 | 예시 |
| --- | --- |
| 시작 태그 | `<p>` |
| 내용 | 데이터 수집 |
| 종료 태그 | `</p>` |
| 요소(Element) | `<p>데이터 수집</p>` 전체 |

주요 태그의 역할은 다음과 같다.

| 태그 | 역할 |
| --- | --- |
| `<h1>`, `<h2>` | 제목 |
| `<p>` | 문단 |
| `<div>` | 여러 요소를 묶는 영역 |
| `<span>` | 텍스트 등 일부 내용을 묶는 영역 |
| `<a>` | 링크 |

### 속성은 요소에 추가 정보를 제공한다

`<span class="text">예시 명언</span>`에서 `span`은 태그 이름이고, `class="text"`는 속성이다.

| 속성 | 역할 | 예시 |
| --- | --- | --- |
| `class` | 요소를 그룹화하거나 구분 | `class="quote"` |
| `id` | 문서 안에서 특정 요소 식별 | `id="main"` |
| `href` | 링크의 이동 주소 | `href="/page/2/"` |

`class="text"`는 태그 이름이 아니라 요소를 찾을 때 활용할 수 있는 속성 값이다.

### 명언 하나의 HTML 예시

다음은 학습자료의 구조를 단순화한 HTML이다.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>명언 데이터 예시</title>
</head>
<body>
    <div class="quote">
        <span class="text">예시 명언입니다.</span>
        <small class="author">작성자 A</small>
        <div class="tags">
            <a class="tag" href="/tag/life/">life</a>
            <a class="tag" href="/tag/thinking/">thinking</a>
        </div>
    </div>
</body>
</html>
```

`div.quote` 안에 본문, 작성자, 태그 영역이 포함되어 있고, `div.tags` 안에는 여러 `a.tag`가 들어 있다.

### DOM과 부모·자식 관계

브라우저는 HTML을 해석하여 요소 간 관계를 가진 DOM(Document Object Model)을 구성한다. DOM은 문서를 트리 구조로 다룰 수 있게 표현한 것이다.

| 부모 요소 | 포함된 자식 요소 | 의미 |
| --- | --- | --- |
| `div.quote` | `span.text` | 해당 명언의 본문 |
| `div.quote` | `small.author` | 해당 명언의 작성자 |
| `div.quote` | `div.tags` | 해당 명언의 태그 영역 |
| `div.tags` | 여러 `a.tag` | 개별 태그 |

**명언 하나의 바깥 영역을 먼저 구분하고, 그 안에서 각 필드를 찾는 것이 중요하다.** 이렇게 하면 본문과 작성자 등이 같은 명언에 속하도록 묶을 수 있다.

## 3. 개발자 도구로 데이터 위치 확인하기

HTML의 모든 태그를 암기하기보다 원하는 데이터가 어느 요소에 포함되어 있는지 확인하는 것이 중요하다.

학습자료에서 제시한 확인 순서는 다음과 같다.

1. 브라우저에서 예시 페이지를 연다.
2. `F12`로 개발자 도구를 연다.
3. `Elements` 탭과 요소 선택 도구를 사용한다.
4. 본문·작성자 등을 선택하여 태그와 속성을 확인한다.
5. 여러 데이터를 구분하는 공통 바깥 영역을 찾는다.

### 태그와 class를 함께 나타내는 CSS 선택자

| 선택자 | 의미 | 찾으려는 정보 |
| --- | --- | --- |
| `div.quote` | `div` 태그이면서 `quote` 클래스를 가진 요소 | 명언 하나의 영역 |
| `span.text` | `span` 태그이면서 `text` 클래스를 가진 요소 | 본문 |
| `small.author` | `small` 태그이면서 `author` 클래스를 가진 요소 | 작성자 |
| `a.tag` | `a` 태그이면서 `tag` 클래스를 가진 요소 | 관련 태그 |

이 표기는 CSS 선택자다. 필요한 요소의 위치를 이해하는 데 사용할 수 있다.

Elements 탭에는 브라우저가 구성한 DOM이 표시된다. JavaScript로 내용이 추가되는 페이지에서는 처음 서버가 보낸 HTML과 현재 DOM이 다를 수 있다.

## 4. JSON의 Key·Value·Object·Array

JSON(JavaScript Object Notation)은 데이터를 구조화하여 표현하고 전달하는 텍스트 형식이다. 웹 API 응답 등에서 사용된다.

### Key와 Value

`"text": "예시 명언입니다."`에서 `text`는 항목 이름인 Key이고, `예시 명언입니다.`는 그에 대응하는 Value다.

JSON의 Key는 큰따옴표로 감싼 문자열이다. Value에는 문자열, 숫자, 참·거짓, `null`, Object, Array 등이 들어갈 수 있다.

### Object와 Array

| 구조 | 표기 | 역할 | Python 파싱 결과 |
| --- | --- | --- | --- |
| Object | `{}` | 여러 Key-Value 쌍을 묶음 | `dict` |
| Array | `[]` | 여러 값을 순서대로 저장 | `list` |

Object에서는 Key로 항목을 찾고, Array에서는 인덱스로 위치를 찾는다. Python의 List 인덱스는 0부터 시작한다.

다음은 명언 하나를 표현한 중첩 JSON 예시다.

```json
{
  "text": "예시 명언입니다.",
  "author": {
    "name": "작성자 A"
  },
  "tags": ["life", "thinking"]
}
```

- `author`의 값은 문자열이 아니라 Object다.
- 작성자 이름은 `author` 안의 `name`에 있다.
- `tags`의 값은 문자열 두 개를 담은 Array다.

필요한 값이 가장 바깥쪽에 있다고 가정하지 말고, Object와 Array의 중첩 관계를 따라가야 한다.

### JSON과 Python의 표현 차이

| JSON | JSON 텍스트 예시 | Python 변환 결과 |
| --- | --- | --- |
| String | `"hello"` | `str` |
| Number | `120`, `3.14` | `int`, `float` |
| Boolean | `true`, `false` | `True`, `False` |
| Null | `null` | `None` |
| Object | `{"name": "Kim"}` | `dict` |
| Array | `["life", "thinking"]` | `list` |

JSON 텍스트와 Python의 Dictionary는 구조가 비슷하지만 표현 규칙이 다르다. JSON 문자열은 `json.loads()`로 파싱하여 Python 자료구조로 변환한다.

## 5. Python에서 중첩된 JSON 확인하기

다음 코드는 학습자료의 응답 구조를 단순화한 가상 데이터다. 네트워크 요청 없이 문자열을 파싱하여 데이터 구조를 확인한다.

```python
import json

json_text = """
{
  "page": 1,
  "has_next": false,
  "quotes": [
    {
      "text": "첫 번째 예시 명언입니다.",
      "author": {"name": "작성자 A"},
      "tags": ["life", "thinking"]
    },
    {
      "text": "두 번째 예시 명언입니다.",
      "author": {"name": "작성자 B"},
      "tags": ["learning"]
    }
  ]
}
"""

data = json.loads(json_text)
quotes = data["quotes"]
first_quote = quotes[0]

print("전체 자료형:", type(data))
print("상위 Key:", list(data.keys()))
print("명언 개수:", len(quotes))
print("본문:", first_quote["text"])
print("작성자:", first_quote["author"]["name"])
print("태그 목록:", first_quote["tags"])
print("첫 번째 태그:", first_quote["tags"][0])
print("다음 페이지:", data["has_next"])

# 전체 자료형: <class 'dict'>
# 상위 Key: ['page', 'has_next', 'quotes']
# 명언 개수: 2
# 본문: 첫 번째 예시 명언입니다.
# 작성자: 작성자 A
# 태그 목록: ['life', 'thinking']
# 첫 번째 태그: life
# 다음 페이지: False
```

### 접근 경로 해석

| Python 표현 | 찾는 대상 | 결과의 자료형 |
| --- | --- | --- |
| `data["quotes"]` | 명언 목록 | `list` |
| `data["quotes"][0]` | 첫 번째 명언 | `dict` |
| `data["quotes"][0]["text"]` | 첫 번째 명언의 본문 | `str` |
| `data["quotes"][0]["author"]` | 작성자 정보 묶음 | `dict` |
| `data["quotes"][0]["author"]["name"]` | 작성자 이름 | `str` |
| `data["quotes"][0]["tags"]` | 태그 목록 | `list` |

이 예시에서는 **Key로 Object의 항목을 찾고, 인덱스로 Array의 항목을 찾는 작업**을 번갈아 수행한다.

## 6. HTML·JSON을 같은 분석용 컬럼으로 연결하기

### Record와 Field 구분

- **Record(레코드):** 하나의 데이터 대상을 나타내는 정보 묶음이다. 이 예시에서는 명언 하나가 레코드 하나다.
- **Field(필드):** 레코드의 속성 하나다. 본문, 작성자 이름, 태그 등이 해당한다.
- **Data Schema:** 최종 컬럼과 자료형 등 데이터 구조를 정의한 것이다.

여러 데이터를 구분하는 기준은 다음과 같다.

| 형식 | 명언 하나의 경계 |
| --- | --- |
| HTML | `div.quote` 영역 하나 |
| JSON | `quotes` Array 안의 Object 하나 |

HTML에서는 명언 영역별로 본문·작성자·태그를 찾고, JSON에서는 명언 Object별로 필요한 Key를 찾는다.

### 같은 정보에 대한 서로 다른 접근 경로

아래 JSON 표현은 파싱된 명언 하나를 `quote`라는 변수로 가져왔다고 가정한다.

| 최종 컬럼 | HTML에서의 위치 | JSON에서의 위치 | 정리할 값 |
| --- | --- | --- | --- |
| `text` | `span.text`의 텍스트 | `quote["text"]` | 본문 문자열 |
| `author` | `small.author`의 텍스트 | `quote["author"]["name"]` | 작성자 이름 문자열 |
| `tags` | 여러 `a.tag`의 텍스트 | `quote["tags"]` | 태그 목록 |

HTML과 JSON의 표현 방식은 달라도 최종적으로 필요한 정보와 컬럼 구조는 같을 수 있다.

### 원본 필드와 수집 메타데이터 구분

| 항목 | 의미 | 처리 방향 |
| --- | --- | --- |
| `text` | 명언 본문 | 레코드에서 추출 |
| `author` | 작성자 이름 | 중첩된 정보에서 이름 추출 |
| `tags` | 관련 태그 | 여러 값을 목록으로 관리 |
| `source_url` | 데이터를 가져온 출처 | 수집 프로그램이 URL을 기록하여 추가 |
| `page` | 현재 페이지 번호 | 페이지 처리에 활용 |
| `has_next` | 다음 페이지 존재 여부 | 다음 페이지 처리 판단에 활용 |

`page`와 `has_next`는 응답 전체의 페이지 정보이며 명언 본문 자체의 속성과 구분한다.

`source_url`은 원본 명언 Object에 항상 들어 있는 필드라고 가정하면 안 된다. 수집한 페이지나 API 주소를 기록하기 위해 추가할 수 있는 정보다.

최종 테이블에서는 **행 하나를 명언 하나, 열 하나를 정한 필드 하나**로 구성할 수 있다. `tags`처럼 여러 값을 담는 항목은 CSV 저장 시 어떤 문자열 형식으로 표현할지도 정해야 한다.

## 7. 데이터 제공 방식에 맞는 수집 방법 선택

| 제공 상황 | 검토할 방법 |
| --- | --- |
| 필요한 데이터를 제공하는 API가 있음 | JSON 응답의 Key·Object·Array 구조 확인 |
| API가 없고 HTML에 데이터가 포함됨 | 태그·속성·계층 구조를 이용한 HTML 기반 수집 |
| JavaScript 실행 후 데이터가 나타남 | 실제 데이터 요청 방식과 DOM 변화를 확인한 뒤 방법 선택 |

학습자료에는 다음 경로가 예시로 제시되어 있다.

- [HTML 페이지](https://quotes.toscrape.com/)
- [JSON 응답 예시 경로](https://quotes.toscrape.com/api/quotes?page=1)

구조화된 API가 필요한 정보를 제공한다면 우선 검토할 수 있다. 다만 HTML과 JSON 중 하나가 항상 더 좋은 것은 아니며, 필요한 데이터를 안정적으로 확보할 수 있는 제공 방식과 구조를 확인해야 한다.

실제 수집에서는 사이트의 이용약관과 접근 정책도 확인한다.

이번 학습의 흐름은 다음과 같이 정리할 수 있다.

1. 수집할 데이터와 필드를 정한다.
2. HTML 또는 JSON의 전체 구조를 확인한다.
3. 레코드 하나의 경계를 찾는다.
4. 레코드 안에서 필요한 값의 위치와 자료형을 확인한다.
5. 원본 필드와 수집 메타데이터를 구분한다.
6. 공통 컬럼 구조로 정리하여 분석에 연결한다.

## 핵심 정리

- HTML은 태그와 속성으로 웹페이지의 내용과 구조를 표현한다.
- DOM의 부모·자식 관계를 이해하면 같은 레코드에 속한 필드들을 묶어 찾을 수 있다.
- `div.quote`처럼 태그와 class를 조합한 표기는 CSS 선택자다.
- JSON은 Object와 Array가 중첩될 수 있으며, Key와 인덱스로 필요한 값을 찾는다.
- `json.loads()`로 JSON 문자열을 Python의 `dict`·`list` 등으로 변환한다.
- JSON의 `true`·`false`·`null`은 Python의 `True`·`False`·`None`으로 변환된다.
- 작성자 정보가 Object이면 그 안의 `name`까지 접근해야 이름을 얻을 수 있다.
- HTML·JSON에서 같은 정보를 추출해 공통 스키마로 정리할 수 있다.
- 원본 필드, 페이지 정보, 수집 과정에서 추가하는 `source_url`을 구분해야 한다.

## 오늘의 한 문장

> 웹 데이터 수집의 출발점은 HTML과 JSON의 구조 안에서 레코드 하나의 경계와 필요한 필드의 위치를 정확히 찾는 것이다.