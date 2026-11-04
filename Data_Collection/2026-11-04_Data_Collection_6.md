# TIL | 2026-11-04 — 데이터 정제와 최종 데이터셋 검증

## 오늘의 학습 주제

수집 데이터에서 발견한 중복·결측·불필요한 공백을 분석 목적에 맞게 처리하는 방법을 배웠다. 정제한 데이터를 텍스트 분석용 테이블로 구성하고, CSV 저장 후 다시 읽어 건수·컬럼·내용을 검증하는 과정까지 학습했다.

```text
원시 데이터 보존 → 필요한 컬럼 선택 → 문자열 양쪽 공백 정리
→ 빈 문자열을 결측값으로 통일 → 본문 누락 제외 → 중복 제거
→ 분석용 DataFrame 구성 → CSV 저장 → 다시 읽어 검증
```

## 1. 분석 목적에 따른 정제 기준

데이터 정제는 품질 문제를 발견하는 데서 나아가, 유지·보완·제외 기준을 적용하여 분석에 사용할 데이터를 만드는 과정이다.

이번 데이터셋의 목적은 명언 본문을 활용한 텍스트 분석이다. 따라서 모든 결측 행을 일괄 삭제하지 않고, 어떤 필드가 비어 있는지에 따라 처리한다.

| 품질 문제 | 이번 학습의 처리 기준 |
| --- | --- |
| 명언 본문 누락 | 원본 확인 후 복구할 수 없다면 제외 |
| 동일한 본문과 작성자 | 첫 번째 행만 유지 |
| 작성자 누락 | 본문이 있으면 유지 |
| 태그 누락 | 본문이 있으면 유지 |
| 문자열 양쪽 공백 | 제거 |
| 출처 누락 | 수집 기록을 확인하고 추가 검토 |

이 기준은 이번 분석을 위한 규칙이다. 다른 분석에서는 필수 필드와 중복 기준을 다시 정해야 한다.

### 공백 정리를 먼저 하는 이유

`"명언 A"`와 `" 명언 A "`는 그대로 비교하면 다른 문자열이다. 양쪽 공백을 제거하면 같은 기준으로 비교할 수 있고, 공백만 있는 본문도 누락으로 판단할 수 있다.

```python
df["text"] = df["text"].astype("string").str.strip()
df["text"] = df["text"].replace("", pd.NA)
```

`str.strip()`은 양쪽 공백을 정리한다. 문장 내부의 단어 사이 공백이나 문장부호까지 일괄 제거하는 처리는 아니다.

### 본문 누락만 지정하여 제외하기

```python
df = df.dropna(subset=["text"]).copy()
```

`subset=["text"]`를 지정하면 작성자·태그가 누락되었다는 이유로 행이 제외되지 않는다. 원본에 본문이 있는지, 추출이 실패했는지 먼저 확인하는 것이 원칙이며, 학습 예제에서는 본문이 없는 행을 제외한다.

### 중복 기준을 명시하기

```python
df = df.drop_duplicates(
    subset=["text", "author"],
    keep="first",
).reset_index(drop=True)
```

| 옵션·기능 | 역할 |
| --- | --- |
| `subset` | 중복 판단에 사용할 컬럼 지정 |
| `keep="first"` | 같은 값 중 첫 번째 행 유지 |
| `keep="last"` | 같은 값 중 마지막 행 유지 |
| `reset_index(drop=True)` | 기존 인덱스를 컬럼으로 추가하지 않고 재정렬 |

태그와 출처가 달라도 본문과 작성자가 같으면 이번 기준에서는 하나만 남긴다. 따라서 제거되는 행에만 있는 태그나 출처 정보가 사라질 수 있음을 검토해야 한다.

작성자가 누락된 행도 같은 본문과 동일한 결측 작성자 값을 가지면 중복으로 처리될 수 있다. 이는 이번에 정한 규칙이며, 실제 출처와 분석 목적에 따라 별도 판단이 필요하다.

## 2. 텍스트 분석용 데이터셋 구성

분석 단위는 **명언 하나를 한 행으로 저장하는 것**이다. 태그가 여러 개여도 명언을 여러 행으로 나누지 않고 문자열로 저장한다.

| 컬럼 | 형태 | 역할과 조건 |
| --- | --- | --- |
| `text` | 문자열 | 실제 분석 대상, 내용이 반드시 존재해야 함 |
| `author` | 문자열 | 작성자별 비교, 누락 허용 |
| `tags` | 문자열 | 태그별 비교, 누락 허용 |
| `source_url` | 문자열 | 출처 추적, 누락 여부 별도 확인 |

JSON에서 태그가 `["life", "thinking"]`처럼 제공되면 수집 단계에서 `", ".join(tags)`로 `"life, thinking"` 형태로 정리할 수 있다.

선택적 정보의 결측은 최종 저장용 테이블에서 빈 문자열로 표현할 수 있다. 다만 빈 문자열로 채우는 것은 없던 정보를 복구하는 작업이 아니다. 특히 출처는 실제 수집 기록을 확인해야 하며 임의의 URL을 만들지 않는다.

### 원문의 의미를 유지하기

이번 단계는 데이터 수집과 기본 품질 정제가 목적이다. 문장부호 제거, 불용어 제거, 토큰화, 형태소 분석, 어간 추출 등은 이후 분석 방법에 따라 결정한다.

예를 들어 `"  Learning is important.  "`는 `"Learning is important."`로 정리하되, 문장 내용과 의미는 유지한다.

### 같은 데이터셋도 분석마다 사용할 행이 다르다

| 분석 목적 | 필요한 값 |
| --- | --- |
| 단어 빈도 분석 | `text` |
| 작성자별 명언 비교 | `text`, `author` |
| 태그별 명언 비교 | `text`, `tags` |

작성자가 없는 명언도 단어 빈도 분석에는 사용할 수 있다. 작성자별 분석에서는 작성자가 있는 행을 별도로 선택한다.

## 3. 원시 데이터와 정제 데이터 구분

| 파일 | 역할 |
| --- | --- |
| `quotes_raw.csv` | 수집한 원시 데이터 보존 |
| `quotes_clean.csv` | 정제한 최종 분석용 데이터 저장 |

원시 파일을 보존하면 정제 기준을 변경하거나 처리 과정의 문제를 확인할 때 처음부터 다시 생성할 수 있다. DataFrame도 `raw_df`와 정제용 복사본을 구분한다.

### 단계별 제외 건수 기록

```text
최종 건수 = 원시 건수 - 본문 누락 제외 건수 - 이후 중복 제거 건수
```

예를 들어 원시 100행에서 본문 누락 3행을 제외한 뒤, 남은 데이터의 중복 5행을 제거했다면 최종 92행이다. 이 숫자는 설명을 위한 가상 예시다.

중복 제거 건수는 반드시 **본문 누락을 제외한 다음 단계에서** 계산한다. 단계별 제외 건수를 기록해야 같은 행을 두 번 계산하지 않고 최종 행 수를 설명할 수 있다.

앞서 품질 탐색에서 `duplicated(keep=False)`로 확인한 중복 후보 전체 행 수와 실제 제거 건수는 다를 수 있다. 제거 건수는 중복 제거 전후 행 수의 차이로 계산한다.

## 4. 정제부터 CSV 저장·재검증까지

다음은 학습자료의 전체 흐름을 정리한 예시다. Pandas와 앞서 수집한 `quotes_raw.csv`가 필요하다. 이 TIL 작성 과정에서 실제 원시 CSV를 정제하거나 수집 결과를 실행 검증한 것은 아니다.

```python
from pathlib import Path
import pandas as pd

raw_path = Path("quotes_raw.csv")
clean_path = Path("quotes_clean.csv")
columns = ["text", "author", "tags", "source_url"]

if not raw_path.exists():
    raise FileNotFoundError("quotes_raw.csv 파일이 없습니다.")

# 1. 문자열과 결측값 읽기 조건을 명시
raw_df = pd.read_csv(
    raw_path,
    encoding="utf-8-sig",
    dtype="string",
    keep_default_na=False,
)

missing_columns = [col for col in columns if col not in raw_df.columns]
if missing_columns:
    raise ValueError(f"누락된 컬럼: {missing_columns}")

# 2. 필요한 컬럼만 정해진 순서로 복사
df = raw_df[columns].copy()
raw_count = len(df)

for col in columns:
    df[col] = df[col].str.strip().replace("", pd.NA)

# 3. 본문 누락 제외 후 중복 제거
missing_text_count = int(df["text"].isna().sum())
df = df.dropna(subset=["text"]).copy()

before_dedup = len(df)
df = df.drop_duplicates(subset=["text", "author"], keep="first")
duplicate_removed = before_dedup - len(df)
df = df.reset_index(drop=True)

# 4. 정제 결과 확인
expected_count = raw_count - missing_text_count - duplicate_removed
assert len(df) == expected_count
assert df["text"].notna().all()
assert df["text"].str.strip().ne("").all()
assert not df.duplicated(subset=["text", "author"]).any()

missing_source_count = int(df["source_url"].isna().sum())

# 5. 남은 선택적 결측을 빈 문자열로 표현하여 저장
clean_df = df.fillna("")
clean_df.to_csv(clean_path, index=False, encoding="utf-8-sig")

# 6. 저장 전과 같은 읽기 조건으로 재확인
saved_df = pd.read_csv(
    clean_path,
    encoding="utf-8-sig",
    dtype="string",
    keep_default_na=False,
)

assert saved_df.columns.tolist() == columns
assert len(saved_df) == len(clean_df)
assert clean_df.equals(saved_df)

print("원시 데이터:", raw_count)
print("본문 누락 제외:", missing_text_count)
print("중복 제거:", duplicate_removed)
print("최종 데이터:", len(saved_df))
print("출처 누락:", missing_source_count)
print(saved_df.head())

if saved_df.empty:
    print("확인 필요: 최종 분석 대상 데이터가 없습니다.")
elif missing_source_count:
    print("기본 정제 완료. 출처 누락에 대한 추가 확인이 필요합니다.")
else:
    print("기본 정제와 저장 결과 확인 완료")
```

### 저장 옵션과 비교 조건

| 설정·기능 | 역할 |
| --- | --- |
| `dtype="string"` | 문자열 자료형으로 읽어 비교 조건 통일 |
| `keep_default_na=False` | 빈 필드와 기본 결측 표기 문자열의 자동 변환 방지 |
| `fillna("")` | 남은 선택적 결측을 저장용 빈 문자열로 표현 |
| `index=False` | 인덱스가 불필요한 CSV 컬럼으로 저장되는 것 방지 |
| `encoding="utf-8-sig"` | Excel 등에서 한글 인코딩 문제 완화 |
| `equals()` | 저장 전후 값과 자료형을 포함한 동일성 비교 |

CSV는 자료형과 결측 표현을 완전하게 보존하는 형식이 아니므로, 다시 읽을 때 조건을 일관되게 적용한다. `keep_default_na=False`에서는 `"NULL"` 같은 문자열이 그대로 남을 수 있어 실제 결측 표기인지 별도 기준으로 확인해야 한다.

예제의 `assert`는 정해진 조건을 만족하지 않을 때 오류를 발생시키는 학습용 검증이다. 조건을 통과해도 텍스트의 의미나 출처 URL의 실제 유효성까지 확인한 것은 아니다.

## 5. 최종 검증에서 구분할 것

| 검증 대상 | 확인 내용 |
| --- | --- |
| 스키마 | 네 개의 컬럼과 순서, 문자열 처리 조건 |
| 본문 | 결측·빈 문자열이 없는지 |
| 중복 | 정한 본문·작성자 기준의 중복이 없는지 |
| 건수 | 단계별 제외 내역과 최종 행 수가 일치하는지 |
| 저장 무결성 | CSV를 다시 읽어도 행 수·컬럼·실제 값이 같은지 |
| 출처 | 누락 여부와 원본 추적 가능성 |
| 수집 완전성 | 마지막 페이지까지 수집했는지, 실패 요청이 없는지 |

특히 **정제 결과의 품질과 전체 수집 완료 여부는 별도로 확인**해야 한다. 일부 페이지가 누락되어도 저장된 행들의 본문과 형식은 정상일 수 있기 때문이다. 전체 수집 여부는 페이지별 기록과 종료 조건도 함께 검토한다.

또한 정제 결과가 0행이어도 본문·중복 조건 검사는 통과할 수 있다. 실제 분석 대상이 남아 있는지와 충분한 범위가 확보되었는지도 확인해야 한다.

## 오늘 배운 점

- 결측값 처리는 분석 목적에 맞춰야 하며, 본문이 있으면 작성자나 태그가 없어도 유지할 수 있다.
- 공백과 빈 문자열을 먼저 통일하면 결측과 중복을 일관된 기준으로 처리할 수 있다.
- 중복 제거는 남길 행과 비교 컬럼을 명시하고, 태그·출처 정보의 손실 가능성도 확인해야 한다.
- 원시 데이터를 보존하고 단계별 제외 건수를 기록해야 정제 결과를 설명하고 다시 만들 수 있다.
- 최종 데이터셋은 저장 후 다시 읽어 컬럼·건수·값을 비교해야 한다.
- 기본 정제, 분석별 활용 가능성, 전체 수집 완료 여부를 각각 검증해야 한다.

> 오늘의 핵심: 분석 목적에 맞는 정제 규칙을 적용하고, 원시 데이터에서 최종 CSV까지의 변경 내역과 저장 결과를 검증하는 방법을 배웠다.