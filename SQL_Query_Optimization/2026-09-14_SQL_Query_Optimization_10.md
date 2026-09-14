# 2026-09-14 EXPLAIN 결과 해석과 실행계획 요소 식별

## 학습 목표

- 복합 쿼리의 실행계획을 트리 구조로 읽을 수 있다.
- `Scan`, `Join`, `Sort`, `WindowAgg` 등 주요 실행계획 노드의 역할을 구분할 수 있다.
- `cost`, `rows`, `actual time`, `loops`의 의미를 이해할 수 있다.
- `EXPLAIN ANALYZE`를 사용해 실제 실행 시간과 병목 구간을 확인할 수 있다.
- 실행계획에서 인덱스가 실제로 사용되는지 검증할 수 있다.

---

## 1. 복합 쿼리의 EXPLAIN 결과 구조

조인, 서브쿼리, 윈도우 함수가 함께 사용된 복합 쿼리는 여러 단계의 실행계획으로 표현된다. 실행계획의 모든 노드를 외우기보다 데이터가 어떤 순서로 읽히고 처리되는지 큰 흐름을 먼저 파악하는 것이 중요하다.

일반적인 처리 흐름은 다음과 같다.

```text
데이터 읽기
    ↓
조건에 맞는 데이터 필터링
    ↓
테이블 조인
    ↓
필요한 순서로 정렬
    ↓
윈도우 함수 계산
    ↓
최종 결과 반환
```

### 실행계획은 트리 구조로 읽는다

PostgreSQL의 `EXPLAIN` 결과는 실행할 작업을 트리 형태로 보여준다. 가장 안쪽에 있는 리프 노드에서 데이터 처리가 시작되고, 그 결과가 상위 노드로 전달된다. 따라서 실행 순서를 파악할 때는 실행계획의 아래쪽부터 위쪽으로 읽는다.

```text
WindowAgg                 ← 정렬된 결과에 윈도우 함수 적용
  └─ Sort                 ← 필요한 기준으로 정렬
      └─ Hash Join        ← 두 데이터셋 결합
          ├─ Seq Scan     ← 첫 번째 테이블 조회
          └─ Seq Scan     ← 두 번째 테이블 조회
```

각 노드는 하나의 연산을 담당한다.

| 노드 | 역할 | 확인할 내용 |
|---|---|---|
| `Seq Scan` | 테이블 전체를 순차적으로 읽음 | 많은 행을 불필요하게 읽는지 |
| `Index Scan` | 인덱스를 이용해 필요한 행을 조회 | 조건에 맞는 인덱스가 사용되는지 |
| `Bitmap Index Scan` | 인덱스로 필요한 행의 위치를 찾음 | 어떤 인덱스와 조건을 사용하는지 |
| `Bitmap Heap Scan` | 찾은 위치를 바탕으로 실제 테이블 행을 가져옴 | 실제로 가져오는 행의 수가 많은지 |
| `Join` | 두 데이터셋을 결합 | 조인 방식과 처리 행 수가 적절한지 |
| `Sort` | 데이터를 지정한 기준으로 정렬 | 메모리 또는 디스크 정렬인지 |
| `Aggregate` | 그룹 집계 수행 | 집계 전에 처리하는 데이터가 과도한지 |
| `WindowAgg` | 윈도우 함수 계산 | 정렬 비용과 처리 행 수가 큰지 |

실행계획은 데이터베이스 내부 구현을 모두 이해하기 위한 문서라기보다, 쿼리가 데이터를 어떻게 읽고 어디서 조인·정렬·집계를 수행하는지 확인하는 도구로 활용할 수 있다.

---

## 2. 주요 실행계획 요소 식별하기

특정 날짜 이후의 주문과 고객 정보를 조회하고 주문 날짜의 내림차순으로 정렬하는 쿼리를 생각해보자.

```sql
EXPLAIN
SELECT
    o.order_id,
    c.first_name,
    c.last_name,
    o.order_date
FROM orders AS o
JOIN customers AS c
    ON o.customer_id = c.customer_id
WHERE o.order_date >= '2018-01-01'
ORDER BY o.order_date DESC;
```

이 실행계획에는 다음과 같은 작업이 나타날 수 있다.

```text
orders와 customers 데이터 읽기
    ↓
조건에 맞는 주문 필터링
    ↓
두 테이블 조인
    ↓
order_date 기준 내림차순 정렬
    ↓
최종 결과 반환
```

처음 실행계획을 볼 때는 세부 알고리즘보다 다음 세 가지를 먼저 구분한다.

- `Scan`: 데이터를 읽는 작업
- `Join`: 두 데이터셋을 연결하는 작업
- `Sort`: 결과의 순서를 변경하는 작업

윈도우 함수가 포함되어 있다면 `WindowAgg`도 함께 확인한다. 윈도우 함수의 `PARTITION BY`와 `ORDER BY`를 계산하기 위해 별도의 정렬이 발생할 수 있기 때문이다.

### cost와 rows

실행계획의 각 노드에는 옵티마이저의 예상치가 표시된다.

```text
Sort  (cost=28399.54..28816.20 rows=166667 width=32)
```

| 항목 | 의미 |
|---|---|
| 첫 번째 `cost` | 결과를 처음 반환하기까지의 시작 비용 |
| 두 번째 `cost` | 해당 노드의 처리를 완료하기까지의 총비용 |
| `rows` | 노드가 반환할 것으로 예상한 행 수 |
| `width` | 행 하나의 예상 평균 크기(Byte) |

`cost`는 밀리초 단위의 실제 실행 시간이 아니다. PostgreSQL이 여러 실행 방법을 비교하기 위해 사용하는 상대적인 비용 값이다.

---

## 3. EXPLAIN과 EXPLAIN ANALYZE 비교

`EXPLAIN`은 옵티마이저가 만든 예상 실행계획만 보여준다. 반면 `EXPLAIN ANALYZE`는 쿼리를 실제로 실행한 뒤 예상치와 실제 측정치를 함께 보여준다.

| 명령 | 쿼리 실제 실행 | 확인할 수 있는 정보 |
|---|---:|---|
| `EXPLAIN` | 아니요 | 예상 비용, 예상 행 수, 실행 노드 |
| `EXPLAIN ANALYZE` | 예 | 예상 정보, 실제 시간, 실제 행 수, 반복 횟수, 전체 실행 시간 |

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE order_date >= '2018-01-01'
ORDER BY order_date DESC;
```

`EXPLAIN ANALYZE` 결과에서는 다음 항목을 중점적으로 확인한다.

- `actual time`: 해당 노드에서 실제로 측정된 시작 시간과 완료 시간
- `rows`: 한 번 실행할 때 실제로 반환한 행 수
- `loops`: 해당 노드가 반복 실행된 횟수
- `Execution Time`: 전체 쿼리 실행 시간
- `Rows Removed by Filter`: 조건 검사 후 제외된 행 수

`EXPLAIN ANALYZE`는 실제로 쿼리를 실행하므로 `INSERT`, `UPDATE`, `DELETE` 같은 변경 쿼리에 사용할 때는 데이터 변경에 주의해야 한다.

---

## 4. 병목 구간 찾기

예시 실행계획에서 약 40만 건을 순차 조회한 뒤 정렬한다고 가정한다.

```text
Sort
  (actual time=256.411..327.851 rows=400205 loops=1)
  Sort Key: order_date DESC
  Sort Method: external merge  Disk: 16248kB
  └─ Seq Scan on orders
       (actual time=0.019..74.576 rows=400205 loops=1)
       Filter: (order_date >= '2018-01-01'::date)
       Rows Removed by Filter: 99795

Execution Time: 350.346 ms
```

이 결과에서는 다음을 알 수 있다.

- `orders` 테이블을 읽고 필터링하는 데 약 75ms가 사용되었다.
- 필터를 통과한 약 40만 건이 정렬 대상으로 전달되었다.
- 전체 실행 시간은 약 350ms이다.
- `external merge`와 `Disk`가 표시되므로 정렬 데이터가 메모리에 다 들어가지 않아 디스크를 사용했다.
- 조회 이후의 정렬 작업이 전체 시간에서 큰 비중을 차지하므로 우선 확인할 병목 후보이다.

### 병목 확인 순서

1. 최상위 노드의 총비용과 전체 `Execution Time`을 확인한다.
2. 하위 노드별 비용과 `actual time`을 비교한다.
3. 많은 행을 읽거나 제거하는 Scan 노드를 찾는다.
4. 큰 데이터가 전달되는 Join·Sort·Aggregate 노드를 찾는다.
5. 예상 `rows`와 실제 `rows`의 차이가 큰 노드를 확인한다.
6. `loops`를 고려해 반복 실행 비용이 큰 노드를 확인한다.

예상 행 수와 실제 행 수가 크게 다르면 옵티마이저가 부정확한 통계를 바탕으로 비효율적인 스캔 방식이나 조인 순서를 선택했을 가능성이 있다. 따라서 비용이 큰 노드뿐 아니라 예상치와 실제치의 차이도 병목 분석의 중요한 단서가 된다.

---

## 5. 실행계획에서 인덱스 사용 여부 확인하기

인덱스는 생성 여부보다 실제 쿼리에서 사용되는지를 확인하는 것이 중요하다.

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE customer_id = 100
ORDER BY order_date DESC;
```

다음과 같은 실행계획이 나타날 수 있다.

```text
Sort
  Sort Key: order_date DESC
  Sort Method: quicksort  Memory: 25kB
  └─ Bitmap Heap Scan on orders
       Recheck Cond: (customer_id = 100)
       └─ Bitmap Index Scan on idx_orders_customer_id
            Index Cond: (customer_id = 100)
```

처리 흐름은 다음과 같다.

```text
customer_id 인덱스로 행의 위치 찾기
    ↓
orders 테이블에서 해당 행 가져오기
    ↓
order_date DESC 기준으로 정렬
    ↓
최종 결과 반환
```

| 실행계획 표현 | 해석 |
|---|---|
| `Bitmap Index Scan on idx_orders_customer_id` | 지정된 인덱스를 이용해 조건에 맞는 위치를 찾음 |
| `Index Cond: (customer_id = 100)` | 인덱스 탐색에 사용한 조건 |
| `Bitmap Heap Scan on orders` | 찾은 위치를 이용해 실제 테이블의 행을 가져옴 |
| `Sort Method: quicksort Memory: 25kB` | 조회 결과를 메모리에서 정렬함 |

예시에서는 6개의 주문을 조회하고 전체 실행 시간이 약 `0.125 ms`였다. 그러나 인덱스가 존재하더라도 데이터 분포, 조회 범위, 예상 반환 행 수에 따라 PostgreSQL은 `Seq Scan` 등 다른 방식을 선택할 수 있다. 그러므로 인덱스를 만든 뒤에는 실제 업무 쿼리에 `EXPLAIN` 또는 `EXPLAIN ANALYZE`를 적용하여 사용 여부를 확인해야 한다.

---

## 6. 실행계획 요소별 개선 방향

| 관찰한 현상 | 확인할 사항 | 개선 방향의 예 |
|---|---|---|
| 큰 테이블에서 `Seq Scan` 발생 | 조건의 선택도, 인덱스 존재 여부, 필터로 제거되는 행 수 | 조건 열에 적절한 인덱스 검토 |
| `Rows Removed by Filter`가 매우 큼 | 불필요한 행을 먼저 읽고 있는지 | 더 선택적인 조건과 인덱스 검토 |
| Join에서 많은 행 처리 | 조인 조건, 조인 전 필터링, 예상·실제 행 수 | 조인 키 인덱스와 필터 적용 시점 검토 |
| Sort 비용이 큼 | 정렬 대상 행 수, 정렬 키, 디스크 사용 여부 | 정렬 전 행 수 축소, 정렬 키 인덱스 검토 |
| `external merge Disk` 발생 | 정렬이 메모리 범위를 초과했는지 | 정렬 데이터 축소, 환경에 맞는 메모리 설정 검토 |
| 예상 rows와 실제 rows 차이가 큼 | 통계 정보와 데이터 분포 | 통계 갱신 및 쿼리 조건 검토 |
| 특정 노드의 `loops`가 큼 | 반복 실행되는 내부 작업의 비용 | 조인 방식, 인덱스, 쿼리 구조 검토 |

실행계획의 특정 노드만 보고 즉시 결론을 내리기보다, 데이터 규모와 분포, 반환 행 수, 전체 쿼리 흐름을 함께 고려해야 한다.

---

## 핵심 용어 정리

| 용어 | 의미 |
|---|---|
| 실행계획 | DBMS가 SQL을 처리하기 위해 선택한 작업 순서와 방법 |
| 노드 | Scan, Join, Sort처럼 실행계획에서 하나의 작업을 나타내는 단위 |
| `cost` | 옵티마이저가 계산한 상대적인 예상 비용 |
| `rows` | 노드가 반환할 것으로 예상하거나 실제 반환한 행 수 |
| `actual time` | `EXPLAIN ANALYZE`에서 측정한 실제 시작·완료 시간 |
| `loops` | 해당 노드가 반복 실행된 횟수 |
| `Execution Time` | 전체 쿼리의 실제 실행 시간 |
| `Seq Scan` | 테이블 전체를 순차적으로 읽는 방식 |
| `Index Scan` | 인덱스를 사용해 필요한 행을 찾는 방식 |
| `Bitmap Index Scan` | 인덱스로 조건에 맞는 행의 위치를 비트맵으로 찾는 작업 |
| `Bitmap Heap Scan` | 비트맵의 위치를 이용해 실제 테이블 행을 읽는 작업 |
| `Sort` | 입력 데이터를 지정된 기준으로 정렬하는 작업 |
| `WindowAgg` | 윈도우 함수를 계산하는 작업 |

---

## 오늘 배운 내용 정리

- 복합 쿼리의 실행계획은 아래쪽 리프 노드부터 위쪽으로 읽는다.
- 모든 노드를 외우기보다 데이터 조회, 필터링, 조인, 정렬, 윈도우 계산의 흐름을 파악한다.
- `cost`는 옵티마이저의 예상 비용이며 실제 실행 시간이 아니다.
- 실제 병목은 `EXPLAIN ANALYZE`의 `actual time`, 실제 `rows`, `loops`, `Execution Time`을 통해 확인한다.
- `external merge`와 `Disk`는 정렬 과정에서 디스크를 사용했다는 신호이다.
- 예상 행 수와 실제 행 수의 큰 차이는 실행계획 선택이 부정확할 수 있다는 단서이다.
- 인덱스는 생성한 것으로 끝나지 않고 실행계획에서 실제 사용 여부를 확인해야 한다.

## 한 문장 정리

> 실행계획은 아래에서 위로 데이터 처리 흐름을 읽고, `EXPLAIN ANALYZE`의 실제 시간과 행 수를 비교하여 비용이 큰 Scan·Join·Sort 노드를 찾는 성능 분석 도구이다.