# SQL 기초: 데이터 조회(SELECT)

## 목차

1. [SELECT 기본 조회 (3.1~3.3)](#1-select-기본-조회-3133)
2. [조회 결과 가공: 산술식, 별칭, 문자열 연결 (3.4~3.6)](#2-조회-결과-가공-산술식-별칭-문자열-연결-3436)
3. [DISTINCT: 중복 제거 (3.7)](#3-distinct-중복-제거-37)
4. [ORDER BY: 결과 정렬 (3.8)](#4-order-by-결과-정렬-38)
5. [WHERE: 조건에 맞는 행 조회 (3.9~3.12)](#5-where-조건에-맞는-행-조회-39312)
6. [BETWEEN, IN, LIKE: 조건을 표현하는 방법 (3.14)](#6-between-in-like-조건을-표현하는-방법-314)
7. [AND, OR와 연산자 우선순위 (3.15)](#7-and-or와-연산자-우선순위-315)

<br>

## 1. SELECT 기본 조회

`SELECT`는 테이블에 저장된 데이터 중 필요한 정보를 조회하는 명령어다. `SELECT`에서 **어떤 컬럼을 볼지** 정하고, `FROM`에서 **어느 테이블을 조회할지** 지정한다.

```sql
-- 부서 테이블의 모든 컬럼 조회
SELECT *
FROM S_DEPT;

-- 직원의 이름, 입사일, 급여만 조회
SELECT NAME, START_DATE, SALARY
FROM S_EMP;
```

<br>

## 2. 조회 결과 가공: 산술식, 별칭, 문자열 연결

`SELECT`에서는 저장된 값을 그대로 보여주는 것뿐 아니라 **계산하거나, 컬럼에 표시할 이름을 붙이거나, 문자열을 이어 붙일 수 있다.**

이러한 표현식은 조회 결과를 만드는 것이며, 원본 테이블의 값을 변경하지 않는다.

### 산술식

숫자 컬럼에는 `+`, `-`, `*`, `/`와 괄호를 이용한 계산식을 사용할 수 있다.

```sql
SELECT NAME, SALARY, SALARY * 12 AS SALARY_X12
FROM S_EMP;
```

직원별 급여에 12를 곱한 값을 `SALARY_X12`라는 컬럼으로 보여준다. 실제 `SALARY` 값이 바뀌는 것은 아니다.

### 컬럼 별칭(ALIAS)

별칭은 조회 결과에 표시되는 컬럼 이름을 바꾸는 기능이다. `AS`는 생략할 수 있으며, 별칭에 공백이 있거나 대소문자를 그대로 표시하려면 큰따옴표(`" "`)를 사용한다.

```sql
SELECT NAME AS "직원 이름", SALARY * 12 AS "급여 12배"
FROM S_EMP;
```

### 문자열 연결(CONCATENATION)

Oracle에서는 `||` 연산자로 문자열이나 컬럼 값을 하나로 연결할 수 있다.

```sql
SELECT NAME || ' - ' || TITLE AS "직원 정보"
FROM S_EMP;
```

`NAME`과 `TITLE` 사이에 `-`를 넣어 `박지수 - 사장`과 같은 형태로 보여준다.

<br>

## 3. DISTINCT: 중복 제거

`SELECT`는 조건에 맞는 행을 그대로 반환하므로, 선택한 컬럼의 값이 여러 행에서 같으면 결과에도 반복해서 나타난다. **`DISTINCT`는 조회 결과에서 중복되는 값을 제거한다.**

```sql
SELECT DISTINCT NAME
FROM S_DEPT;
```

`S_DEPT`에는 `영업부`, `기획부`처럼 이름이 같은 부서가 여러 지역에 존재한다. `DISTINCT NAME`을 사용하면 같은 부서명이 한 번씩만 표시된다.

여러 컬럼을 함께 선택하면 **각 컬럼의 조합이 같은 행**을 중복으로 판단한다.

```sql
SELECT DISTINCT NAME, REGION_ID
FROM S_DEPT;
```

따라서 `NAME`만 조회할 때와 `NAME, REGION_ID`를 함께 조회할 때 중복 제거 결과가 달라질 수 있다.

<br>

## 4. ORDER BY: 결과 정렬

`ORDER BY`는 조회 결과를 특정 컬럼이나 표현식 기준으로 정렬한다. 정렬 기준을 지정하지 않으면 **결과가 항상 같은 순서로 나온다고 보장할 수 없다.**

- `ASC`: 오름차순. 생략하면 기본으로 적용된다.
- `DESC`: 내림차순.
- 정렬 기준을 여러 개 지정하면 앞의 기준부터 적용한다.

```sql
SELECT NAME, SALARY
FROM S_EMP
ORDER BY SALARY DESC, NAME ASC;
```

급여가 높은 직원부터 보여주고, 급여가 같을 때는 이름순으로 정렬한다. `ORDER BY`는 일반적인 `SELECT` 문에서 마지막에 작성한다.

<br>

## 5. WHERE: 조건에 맞는 행 조회

`SELECT`가 **어떤 컬럼을 보여줄지** 정한다면, `WHERE`는 **어떤 행을 가져올지** 결정한다. 조건을 만족하는 행만 조회 결과에 포함된다.

```sql
SELECT NAME, TITLE, DEPT_ID
FROM S_EMP
WHERE DEPT_ID = 110;
```

이 쿼리는 부서 번호가 `110`인 직원만 조회한다.

### 비교 연산자

| 연산자     | 의미              | 조건 예시        |
| ---------- | ----------------- | ---------------- |
| `=`        | 같다              | `DEPT_ID = 110`  |
| `<>`, `!=` | 같지 않다         | `DEPT_ID <> 110` |
| `>`, `>=`  | 크다, 크거나 같다 | `SALARY >= 1000` |
| `<`, `<=`  | 작다, 작거나 같다 | `SALARY < 2000`  |

```sql
-- 급여가 1,000 이상인 직원 조회
SELECT NAME, SALARY
FROM S_EMP
WHERE SALARY >= 1000;

-- 2016년 이전에 입사한 직원 조회
SELECT NAME, START_DATE
FROM S_EMP
WHERE START_DATE < TO_DATE('2016-01-01', 'YYYY-MM-DD');
```

숫자뿐 아니라 날짜도 비교 조건에 사용할 수 있다. TO_DATE()는 문자열을 DATE 타입으로 변환하는 함수로, 'YYYY-MM-DD'는 변환할 날짜의 형식을 지정한다.

### 부정 조건과 NULL 비교

조건을 반대로 표현할 때는 `<>`, `NOT` 등을 사용한다. 예를 들어 `NOT BETWEEN`, `NOT IN`, `NOT LIKE`로 해당 범위나 목록, 패턴에 속하지 않는 값을 찾을 수 있다.

`NULL`은 값이 없거나 알려지지 않은 상태를 의미하므로 `= NULL`, `<> NULL`로 비교하지 않고 **`IS NULL`, `IS NOT NULL`**을 사용한다.

```sql
SELECT NAME, START_DATE
FROM S_EMP
WHERE START_DATE IS NOT NULL;
```

<br>

## 6. BETWEEN, IN, LIKE: 조건을 표현하는 방법

같은 `WHERE` 조건이라도 원하는 데이터의 형태에 따라 더 간결하게 표현할 수 있다.

| 연산자            | 용도                              | 특징                                             |
| ----------------- | --------------------------------- | ------------------------------------------------ |
| `BETWEEN a AND b` | 범위 조회                         | 양 끝값 `a`, `b`도 포함한다.                     |
| `IN (a, b, ...)`  | 여러 값 중 하나와 일치하는지 검사 | `OR`로 여러 동등 조건을 연결한 것과 비슷하다.    |
| `LIKE`            | 문자열 패턴 검색                  | `%`는 0개 이상의 문자, `_`는 한 글자를 나타낸다. |

### BETWEEN: 범위에 포함되는 값

```sql
SELECT NAME, SALARY
FROM S_EMP
WHERE SALARY BETWEEN 1000 AND 1500;
```

급여가 1,000 이상 1,500 이하인 직원을 조회한다.

### IN: 여러 값 중 하나에 해당하는 값

```sql
SELECT NAME, DEPT_ID
FROM S_EMP
WHERE DEPT_ID IN (110, 113);
```

부서 번호가 `110` 또는 `113`인 직원을 조회한다.

### LIKE: 문자열 패턴 검색

```sql
SELECT NAME, TITLE
FROM S_EMP
WHERE NAME LIKE '김%';
```

이름이 `김`으로 시작하는 직원을 조회한다. 예를 들어 `%김%`은 `김`이 포함된 문자열, `김_`은 `김` 다음에 한 글자가 오는 문자열과 일치한다.

<br>

## 7. AND, OR와 연산자 우선순위

여러 조건을 함께 적용할 때는 논리 연산자를 사용한다.

- `AND`: 모든 조건을 만족해야 한다.
- `OR`: 조건 중 하나 이상을 만족하면 된다.
- `NOT`: 조건의 결과를 반대로 바꾼다.

일반적으로 비교 연산자가 먼저 평가되고, 논리 연산자 중에는 **`NOT` → `AND` → `OR`** 순으로 우선순위가 적용된다. 여러 조건을 섞어 쓸 때는 괄호로 의도를 명확하게 표현하는 것이 좋다.

```sql
SELECT NAME, DEPT_ID, SALARY
FROM S_EMP
WHERE (DEPT_ID = 110 OR DEPT_ID = 113)
  AND SALARY >= 2000;
```

먼저 부서 번호가 `110` 또는 `113`인지 확인하고, **그중 급여가 2,000 이상인 직원**만 조회한다. 괄호를 생략하면 `AND`가 `OR`보다 먼저 적용되어 의도와 다른 결과가 나올 수 있다.

<br>

## 참고 자료

- 『SQL의 정석』 3장
