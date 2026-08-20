# 2026-08-19 SQL 테이블 관계, ERD와 테이블 생성 및 데이터 입력

## 학습 목표

- 테이블 간 `1:1`, `1:N`, `N:M` 관계를 구분한다.
- ERD(Entity Relationship Diagram)의 기본 구성과 관계 기호를 이해한다.
- PK와 FK를 따라 필요한 테이블의 JOIN 경로를 찾는다.
- DDL과 DML의 차이를 구분한다.
- `CREATE TABLE`로 테이블을 생성하고 제약 조건을 설정한다.
- `DROP TABLE`로 테이블을 삭제한다.
- `INSERT INTO`로 데이터를 입력하고 여러 행을 한 번에 추가하는 방법을 익힌다.

---

## 1. 테이블 간 관계

실무 데이터베이스는 여러 테이블이 서로 연결되어 구성되기 때문에, 테이블 간 관계를 이해해야 원하는 데이터를 제대로 조회할 수 있다.

### 1:1 관계

`1:1`은 한 테이블의 레코드 하나가 다른 테이블의 레코드 하나와 정확히 대응되는 관계이다.

예를 들어 회원과 회원 상세정보가 있다.

- `users` : 회원 정보
- `user_details` : 회원 상세정보

회원 한 명마다 하나의 상세정보가 대응되므로 `users : user_details = 1:1`이다.

### 1:N 관계

`1:N`은 한 테이블의 레코드 하나가 다른 테이블의 여러 레코드와 연결되는 관계이다. 실무에서 자주 등장하는 관계 유형이다.

대표적인 예시는 **회원과 주문**이다.

- 한 회원은 여러 주문을 할 수 있다.
- 하나의 주문은 한 회원에게 속한다.

따라서 `users : orders = 1:N`이다.

게시글과 댓글도 같은 구조이다. 하나의 게시글에는 여러 댓글이 달릴 수 있지만 하나의 댓글은 하나의 게시글에 속한다.

### N:M 관계

`N:M`은 양쪽 테이블의 레코드가 서로 여러 개와 연결될 수 있는 관계이다.

예를 들어 상품과 카테고리, 학생과 강의가 있다.

- 하나의 상품은 여러 카테고리에 포함될 수 있다.
- 하나의 카테고리에는 여러 상품이 포함될 수 있다.

N:M 관계는 데이터베이스에서 직접 표현하지 않고 **중간 테이블(Junction Table)**을 사용하여 두 개의 `1:N` 관계로 분리한다.

예를 들어 학생과 강의를 연결한다면:

```text
학생(student_id)
      ↓ 1:N
수강(student_id, course_id)
      ↑ N:1
강의(course_id)
```

상품과 카테고리라면 `product_categories(product_id, category_id)`와 같은 중간 테이블을 사용할 수 있다.

### 관계 유형 정리

| 관계 | 의미 | 예시 |
|---|---|---|
| `1:1` | 1개 ↔ 1개 | 회원 : 회원 상세정보 |
| `1:N` | 1개 ↔ 여러 개 | 회원 : 주문, 게시글 : 댓글 |
| `N:M` | 여러 개 ↔ 여러 개 | 학생 : 강의, 상품 : 태그 |

---

## 2. ERD 읽기

ERD(Entity Relationship Diagram)는 데이터베이스의 테이블과 테이블 간 관계를 시각적으로 표현한 다이어그램이다.

ERD의 모든 표기법을 암기하는 것보다 다음 내용을 파악하는 것이 중요하다.

1. 필요한 데이터가 어느 테이블에 있는가?
2. 각 테이블의 PK는 무엇인가?
3. 다른 테이블에서 해당 PK를 참조하는 FK는 무엇인가?
4. PK와 FK를 따라 어떤 테이블끼리 연결할 수 있는가?
5. 어떤 컬럼을 기준으로 JOIN해야 하는가?

### PK와 FK를 따라 JOIN 경로 찾기

예를 들어 고객의 이름과 주문 상품을 함께 조회해야 한다면 다음과 같은 테이블이 있다고 생각할 수 있다.

```text
customers
  customer_id (PK)
  name

orders
  order_id (PK)
  customer_id (FK)
  order_date

order_items
  order_item_id (PK)
  order_id (FK)
  product_name
  quantity
```

`customers`와 `order_items`는 직접 연결되는 공통 관계가 없으므로 PK와 FK를 따라간다.

```text
customers → orders → order_items
```

즉, ERD는 복잡한 데이터베이스에서 **필요한 데이터까지 어떤 테이블을 거쳐 갈 수 있는지 찾는 지도**처럼 활용할 수 있다.

예시 JOIN:

```sql
SELECT c.name,
       oi.product_name,
       oi.quantity
FROM customers AS c
JOIN orders AS o
  ON c.customer_id = o.customer_id
JOIN order_items AS oi
  ON o.order_id = oi.order_id;
```

회원과 주문만 연결한다면 다음처럼 PK와 FK를 기준으로 JOIN할 수 있다.

```sql
SELECT u.name,
       o.order_id,
       o.order_date
FROM users AS u
JOIN orders AS o
  ON u.user_id = o.user_id;
```

### ERD의 기본 구성

- **Entity**: 데이터베이스의 테이블에 해당
- **Attribute**: 테이블의 컬럼에 해당
- **Relationship**: 테이블 간 연결

### Crow's Foot Notation

ERD에서는 관계의 수량을 기호로 표현할 수 있다.

- `|` : 최소 1개, 필수
- `O` : 0개, 선택적
- `<` : 여러 개(N)
- `||` : 정확히 1개, 필수

---

## 3. DDL과 DML

SQL 명령어는 역할에 따라 여러 그룹으로 나뉘며, 이번 학습에서는 특히 DDL과 DML의 차이를 배웠다.

### DDL

**DDL(Data Definition Language)**은 데이터베이스의 구조를 정의하거나 변경·삭제하는 명령어이다.

대표적인 명령어:

- `CREATE`
- `ALTER`
- `DROP`

예를 들어 새로운 테이블을 만드는 `CREATE TABLE`은 DDL이다.

### DML

**DML(Data Manipulation Language)**은 테이블 내부의 데이터를 조회하거나 삽입·수정·삭제하는 명령어이다.

대표적인 명령어:

- `SELECT`
- `INSERT`
- `UPDATE`
- `DELETE`

쉽게 비유하면:

```text
DDL → 데이터를 담을 그릇을 만드는 것
DML → 그릇 안의 데이터를 다루는 것
```

---

## 4. CREATE TABLE

`CREATE TABLE`은 새로운 테이블의 구조를 정의하는 DDL 명령이다.

기본 문법은 다음과 같다.

```sql
CREATE TABLE 테이블명
(
    열이름1 데이터타입 제약조건,
    열이름2 데이터타입 제약조건,
    ...
);
```

예를 들어 회원 테이블을 만들 수 있다.

```sql
CREATE TABLE users
(
    user_id INT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    created_at DATE NOT NULL
);
```

각 컬럼에는 이름과 데이터 타입을 지정하고, 필요한 경우 제약 조건을 추가한다.

### 주요 제약 조건

| 제약 조건 | 의미 |
|---|---|
| `PRIMARY KEY` | 중복 불가, NULL 불가 |
| `NOT NULL` | NULL 저장 불가 |
| `UNIQUE` | 중복 값 저장 불가 |
| `DEFAULT` | 값이 없을 때 기본값 자동 적용 |
| `REFERENCES` | 외래키가 참조할 테이블 지정 |

예를 들어 기본값을 설정할 수 있다.

```sql
CREATE TABLE ddl_test_members
(
    member_id INT PRIMARY KEY,
    member_name VARCHAR(50) NOT NULL,
    active BOOLEAN DEFAULT TRUE,
    created_at DATE DEFAULT CURRENT_DATE
);
```

`active`를 입력하지 않으면 `TRUE`, `created_at`을 입력하지 않으면 현재 날짜가 기본값으로 사용된다.

### REFERENCES로 테이블 연결

외래키 관계는 `REFERENCES`를 이용해 실제 테이블 생성 시 설정할 수 있다.

```sql
CREATE TABLE ddl_test_orders
(
    order_id INT PRIMARY KEY,
    customer_id INT REFERENCES ddl_test_customers(customer_id),
    order_date DATE NOT NULL
);
```

여기서 `customer_id`는 `ddl_test_customers`의 `customer_id`를 참조한다.

---

## 5. DROP TABLE

`DROP TABLE`은 테이블 자체를 삭제하는 DDL 명령이다.

```sql
DROP TABLE test_events;
```

테이블만 삭제되는 것이 아니라 테이블 내부의 데이터도 함께 삭제된다. 또한 한 번 삭제하면 복구할 수 없으므로 주의해야 한다.

특히 다른 테이블이 외래키로 해당 테이블을 참조하고 있다면 참조 관계를 고려하여 삭제 순서를 결정해야 한다.

예를 들어 `ddl_test_employees.department_id`가 `ddl_test_departments.department_id`를 참조한다면, 참조하는 직원 테이블을 먼저 삭제한 뒤 부서 테이블을 삭제해야 한다.

---

## 6. INSERT INTO

`INSERT INTO`는 테이블에 새로운 행(Row)을 추가하는 DML 명령이다.

권장되는 기본 형태는 열 이름을 직접 지정하는 방식이다.

```sql
INSERT INTO employees
(
    employee_id,
    employee_name,
    department
)
VALUES
(
    101,
    '김민수',
    '데이터팀'
);
```

중요한 점은 **열 이름의 순서와 VALUES에 작성하는 값의 순서를 맞추는 것**이다.

```text
employee_id → employee_name → department
101         → '김민수'      → '데이터팀'
```

열 이름을 지정하면 어떤 값이 어느 컬럼에 들어가는지 명확하기 때문에 더 안전하다.

모든 열에 순서대로 값을 입력하는 방식도 가능하다.

```sql
INSERT INTO 테이블명
VALUES
(
    값1,
    값2,
    값3,
    값4
);
```

---

## 7. 여러 행 입력

여러 개의 데이터를 입력할 때는 한 번의 `INSERT INTO`에서 여러 행을 입력할 수 있다.

```sql
INSERT INTO ddl_test_products
(
    product_id,
    product_name,
    list_price
)
VALUES
    (1, 'Mountain Bike', 1200.00),
    (2, 'Road Bike', 1800.00);
```

이처럼 여러 행을 한 번에 입력하면 데이터를 추가하는 작업을 효율적으로 처리할 수 있다.

---

## 8. 오늘 배운 핵심 흐름

이번 학습에서 SQL을 사용할 때 다음과 같은 흐름을 이해하는 것이 중요했다.

```text
ERD 확인
   ↓
필요한 데이터가 있는 테이블 찾기
   ↓
PK / FK 확인
   ↓
JOIN 경로 결정
   ↓
테이블 구조가 필요하면 CREATE TABLE
   ↓
데이터 입력은 INSERT INTO
   ↓
불필요한 실습 테이블은 DROP TABLE
```

특히 ERD에서는 모든 기호를 먼저 외우기보다 **필요한 데이터가 어디에 있고 PK와 FK가 어떻게 연결되어 있는지 확인하는 것**이 중요하다.

---

## 9. 오늘의 핵심 코드

### 테이블 생성

```sql
CREATE TABLE courses
(
    course_id INT PRIMARY KEY,
    course_name VARCHAR(100) NOT NULL,
    max_people INT NOT NULL,
    is_open BOOLEAN DEFAULT TRUE
);
```

### 외래키 설정

```sql
CREATE TABLE ddl_test_orders
(
    order_id INT PRIMARY KEY,
    customer_id INT REFERENCES ddl_test_customers(customer_id),
    order_date DATE NOT NULL
);
```

### 데이터 입력

```sql
INSERT INTO employees
(
    employee_id,
    employee_name,
    department
)
VALUES
(
    101,
    '김민수',
    '데이터팀'
);
```

### 여러 행 입력

```sql
INSERT INTO ddl_test_products
(
    product_id,
    product_name,
    list_price
)
VALUES
    (1, 'Mountain Bike', 1200.00),
    (2, 'Road Bike', 1800.00);
```

### 테이블 삭제

```sql
DROP TABLE test_events;
```

---

## 10. 느낀 점

오늘은 데이터베이스의 이론적인 구조를 이해하는 것에서 한 단계 더 나아가, 실제 SQL을 사용해 테이블을 만들고 데이터를 입력하는 방법을 배웠다.

특히 테이블 간 관계에서 `1:1`, `1:N`, `N:M`을 단순히 기호로 암기하는 것보다 **각 테이블의 데이터가 실제로 어떻게 연결되는지 생각하는 것이 중요하다**는 점을 알게 되었다. `N:M` 관계에서는 중간 테이블을 사용하여 두 개의 `1:N` 관계로 나눈다는 것도 기억해야 할 핵심 내용이었다.

ERD를 읽을 때는 복잡한 관계를 한 번에 전부 이해하려 하기보다, 먼저 필요한 데이터가 어느 테이블에 있는지 찾고 PK와 FK를 따라가며 JOIN 경로를 찾는 방식이 실무적으로 중요하다는 것을 배웠다.

또한 `CREATE TABLE`을 이용해 테이블 구조를 만들고, `PRIMARY KEY`, `NOT NULL`, `UNIQUE`, `DEFAULT`, `REFERENCES` 같은 제약 조건으로 저장할 수 있는 데이터를 제한할 수 있다는 점도 이해했다.

`INSERT INTO`에서는 컬럼 이름과 값의 순서를 맞추는 것이 중요했고, 여러 행을 한 번에 입력할 수 있다는 것도 알게 되었다. 반대로 `DROP TABLE`은 테이블과 데이터를 함께 삭제하므로 특히 주의해서 사용해야 한다.

결국 오늘 배운 내용은

> **ERD로 테이블 관계를 파악하고 → PK/FK를 따라 JOIN 경로를 찾고 → DDL로 테이블 구조를 만들고 → DML로 데이터를 다루는 SQL의 기본 흐름**

을 처음 직접 경험했다는 점에서 의미가 있었다.