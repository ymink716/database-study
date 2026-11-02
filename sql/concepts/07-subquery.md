# SQL 기초: 서브쿼리(Subquery)

## 목차

1. [서브쿼리의 개념과 기본 형태](#1-서브쿼리의-개념과-기본-형태)
2. [단일 행 서브쿼리(Single Row Subquery)](#2-단일-행-서브쿼리single-row-subquery)
3. [다중 행 서브쿼리(Multi Row Subquery)](#3-다중-행-서브쿼리multi-row-subquery)
4. [다중 컬럼 서브쿼리(Multi-column Subquery)](#4-다중-컬럼-서브쿼리multi-column-subquery)
5. [FROM절 서브쿼리: 인라인 뷰(Inline View)](#5-from절-서브쿼리-인라인-뷰inline-view)
6. [HAVING절에서의 서브쿼리](#6-having절에서의-서브쿼리)
7. [CREATE TABLE에서의 서브쿼리](#7-create-table에서의-서브쿼리)
8. [DML에서의 서브쿼리](#8-dml에서의-서브쿼리)
9. [서브쿼리를 사용할 때 알아둘 점](#9-서브쿼리를-사용할-때-알아둘-점)

<br>

## 1. 서브쿼리의 개념과 기본 형태

**서브쿼리(Subquery)**는 하나의 SQL문 안에 포함된 또 다른 `SELECT`문이다. **메인 쿼리(Main Query)**에서 필요한 값이나 조회 대상 데이터를 서브쿼리로 구해 사용할 수 있다.

예를 들어 **김정미가 속한 부서의 직원들을 조회**하려고 한다면, 먼저 김정미의 부서 번호를 알아낸 다음 그 번호로 직원들을 검색해야 한다. 이 과정을 하나의 SQL문으로 표현할 수 있다.

```sql
SELECT NAME, DEPT_ID
FROM S_EMP
WHERE DEPT_ID = (
    SELECT DEPT_ID
    FROM S_EMP
    WHERE NAME = '김정미'
);
```

- **안쪽 서브쿼리:** 김정미의 `DEPT_ID`인 `111`을 구한다.
- **바깥쪽 메인 쿼리:** `DEPT_ID = 111`인 직원들을 조회한다.

서브쿼리를 이해할 때는 먼저 **안쪽 쿼리가 무엇을 반환하는지(값 하나, 여러 행, 여러 컬럼)**를 확인한 다음, 그 결과를 바깥쪽에서 어떻게 사용하는지 살펴보면 좋다.

<br>

## 2. 단일 행 서브쿼리(Single Row Subquery)

**단일 행 서브쿼리**는 서브쿼리가 **한 행을 반환**하는 경우다. 그 행에서 **한 컬럼만 반환한다면** 하나의 값처럼 `=`, `>`, `<`, `>=`, `<=`, `<>` 같은 단일 값 비교 연산자와 함께 사용할 수 있다.

### 같은 부서에 속한 직원 조회

```sql
SELECT ID, NAME, TITLE, DEPT_ID
FROM S_EMP
WHERE DEPT_ID = (
    SELECT DEPT_ID
    FROM S_EMP
    WHERE NAME = '김정미'
)
ORDER BY ID;
```

**결과**

| ID  | NAME   | TITLE | DEPT_ID |
| --- | ------ | ----- | ------- |
| 7   | 김정미 | 과장  | 111     |
| 18  | 서정주 | 사원  | 111     |
| 19  | 안창환 | 사원  | 111     |

안쪽에서 김정미의 부서 번호 `111`을 반환하고, 바깥쪽에서 해당 부서의 직원 3명을 조회한다. **서브쿼리가 반환하는 값 자체가 조회 조건**이 되는 예제다.

### 전체 평균 급여보다 많이 받는 직원 조회

```sql
SELECT NAME, SALARY
FROM S_EMP
WHERE SALARY > (
    SELECT AVG(SALARY)
    FROM S_EMP
)
ORDER BY SALARY DESC;
```

안쪽 쿼리의 결과는 **전체 평균 급여 `1695.6`**이다. 따라서 평균보다 급여가 높은 직원만 조회한다. `AVG()`처럼 여러 행을 집계하더라도 **결과가 한 값이면 단일 행 서브쿼리로 비교**할 수 있다.

<br>

## 3. 다중 행 서브쿼리(Multi Row Subquery)

**다중 행 서브쿼리**는 서브쿼리에서 **여러 행이 반환**되는 경우다. 결과가 여러 값일 수 있으므로 `=` 대신 주로 **`IN`, `NOT IN`** 같은 다중 행 비교 연산자를 사용한다.

### 특정 지역에 속한 부서의 직원 조회

충청/강원 지역(`REGION_ID = 3`)에 있는 부서들을 먼저 찾고, 그 부서에 속한 직원을 조회해 보자.

```sql
SELECT ID, NAME, DEPT_ID
FROM S_EMP
WHERE DEPT_ID IN (
    SELECT ID
    FROM S_DEPT
    WHERE REGION_ID = 3
)
ORDER BY ID;
```

**서브쿼리 결과:** 부서 번호 `104`, `112`

**최종 결과**

| ID  | NAME   | DEPT_ID |
| --- | ------ | ------- |
| 8   | 신기웅 | 112     |
| 13  | 김석우 | 104     |
| 20  | 윤정숙 | 112     |
| 21  | 장현철 | 112     |

- 안쪽 쿼리는 지역 번호가 `3`인 **여러 부서 번호**를 반환한다.
- 바깥쪽 쿼리는 그중 어느 하나에 속하는 직원을 조회한다.

<br>

## 4. 다중 컬럼 서브쿼리(Multi-column Subquery)

**다중 컬럼 서브쿼리**는 **여러 컬럼을 하나의 조합으로 비교**하는 방식이다. 값 하나가 아니라 `(급여, 부서 번호)`처럼 컬럼들의 묶음이 일치하는지 확인할 수 있다.

### 부서별 최저 급여를 받는 직원 조회

```sql
SELECT ID, NAME, DEPT_ID, SALARY
FROM S_EMP
WHERE (SALARY, DEPT_ID) IN (
    SELECT MIN(SALARY), DEPT_ID
    FROM S_EMP
    GROUP BY DEPT_ID
)
ORDER BY DEPT_ID, ID;
```

**결과 중 일부**

| ID  | NAME   | DEPT_ID | SALARY |
| --- | ------ | ------- | -----: |
| 16  | 이기철 | 110     |      0 |
| 19  | 안창환 | 111     |    795 |
| 20  | 윤정숙 | 112     |    750 |
| 22  | 장혜숙 | 113     |    800 |

서브쿼리는 부서마다 `(최저 급여, 부서 번호)`의 조합을 반환한다. 바깥쪽의 `(SALARY, DEPT_ID)`가 이 조합 중 하나와 **둘 다 일치하는 직원**을 찾는 것이다.

급여만 비교하면 **다른 부서에서 같은 급여를 받는 직원**까지 잘못 포함될 수 있으므로, 부서 번호도 함께 비교하는 것이 중요하다. 같은 부서에서 최저 급여가 동률이라면 해당 직원들이 모두 조회된다.

<br>

## 5. FROM절 서브쿼리: 인라인 뷰(Inline View)

**FROM절의 서브쿼리**는 안쪽 조회 결과를 **하나의 임시적인 테이블처럼 사용**하는 방법이다. 이렇게 `FROM`에 작성한 서브쿼리를 **인라인 뷰(Inline View)**라고 부른다.

### 사원만 먼저 조회한 뒤 부서 정보와 연결

```sql
SELECT e.NAME AS EMP_NAME,
       e.TITLE,
       d.NAME AS DEPT_NAME
FROM (
    SELECT NAME, TITLE, DEPT_ID
    FROM S_EMP
    WHERE TITLE = '사원'
) e
JOIN S_DEPT d
    ON e.DEPT_ID = d.ID
ORDER BY e.NAME;
```

**결과 중 일부**

| EMP_NAME | TITLE | DEPT_NAME |
| -------- | ----- | --------- |
| 강연범   | 사원  | 영업부    |
| 구원상   | 사원  | 기획부    |
| 서정주   | 사원  | 기획부    |
| 안창환   | 사원  | 기획부    |

1. 안쪽 쿼리에서 `TITLE = '사원'`인 직원의 필요한 컬럼만 조회한다.
2. 이 결과에 `e`라는 별칭을 붙여 `FROM`에서 테이블처럼 사용한다.
3. `S_DEPT`와 조인하여 부서명을 함께 조회한다.

인라인 뷰는 복잡한 조회를 단계별로 나누어 이해하기 쉽게 만들거나, 필요한 행과 컬럼을 먼저 조회한 결과를 다른 테이블과 연결할 때 유용하다.

<br>

## 6. HAVING절에서의 서브쿼리

`HAVING`은 **그룹화한 결과를 필터링**하는 절이다. 비교할 기준값도 다른 쿼리로 구하고 싶다면 `HAVING`에 서브쿼리를 사용할 수 있다.

### 113번 부서보다 평균 급여가 높은 부서 조회

```sql
SELECT DEPT_ID,
       ROUND(AVG(SALARY), 2) AS AVG_SALARY
FROM S_EMP
GROUP BY DEPT_ID
HAVING AVG(SALARY) > (
    SELECT AVG(SALARY)
    FROM S_EMP
    WHERE DEPT_ID = 113
)
ORDER BY DEPT_ID;
```

**서브쿼리 결과:** 113번 부서의 평균 급여 `1472`

**최종 결과**

| DEPT_ID | AVG_SALARY |
| ------- | ---------: |
| 101     |       3000 |
| 102     |       2975 |
| 104     |       2650 |
| 110     |       1585 |
| 118     |       4100 |

여기서 `WHERE DEPT_ID = 113`은 **비교 기준이 되는 부서의 행을 선택**하고, 바깥쪽의 `HAVING`은 **각 부서의 평균 급여와 그 기준값을 비교**한다.

### 응용: 평균 급여가 가장 낮은 직책 찾기

```sql
SELECT TITLE,
       ROUND(AVG(SALARY), 2) AS AVG_SALARY
FROM S_EMP
GROUP BY TITLE
HAVING AVG(SALARY) = (
    SELECT MIN(AVG(SALARY))
    FROM S_EMP
    GROUP BY TITLE
);
```

**결과:** `사원` / 평균 급여 `809`

안쪽에서는 직책별 평균 급여를 계산하고, 그중 **가장 작은 평균값**을 구한다. 바깥쪽에서는 그 값과 평균 급여가 일치하는 직책을 조회한다. `MIN(AVG(SALARY))`는 그룹 함수를 중첩하여 사용한 예다.

<br>

## 7. CREATE TABLE에서의 서브쿼리

기존 테이블을 조회한 결과로 새 테이블을 만들 수 있다.

```sql
CREATE TABLE EMP_113 AS
SELECT ID, NAME, MAILID, START_DATE
FROM S_EMP
WHERE DEPT_ID = 113;
```

이 문장을 실행하면 `EMP_113` 테이블이 새로 만들어지고, **113번 부서 직원 5명의 데이터**가 복사된다. 원본인 `S_EMP`는 변경되지 않는다.

```sql
SELECT COUNT(*) AS EMP_COUNT
FROM EMP_113;

-- 결과: 5
```

- `CREATE TABLE`: 새로운 테이블을 생성한다.
- `AS SELECT`: 새 테이블에 들어갈 컬럼과 데이터를 기존 조회 결과로 지정한다.
- **데이터 없이 구조만** 복사하려면 `WHERE 1 = 0`처럼 항상 거짓인 조건을 사용할 수 있다.
- 원본의 **PK·FK 제약조건과 인덱스가 그대로 복사되는 것은 아니다.**

<br>

## 8. DML에서의 서브쿼리

서브쿼리는 조회뿐 아니라 **`INSERT`, `UPDATE`처럼 데이터를 변경하는 문장**에서도 활용할 수 있다.

### INSERT ... SELECT: 조회 결과를 여러 행 추가

```sql
INSERT INTO EMP_113 (ID, NAME, MAILID, START_DATE)
SELECT ID, NAME, MAILID, START_DATE
FROM S_EMP
WHERE DEPT_ID = 110;
```

`SELECT`로 110번 부서 직원 **4명**을 조회하고 그 행들을 `EMP_113`에 추가한다. 따라서 7번에서 생성한 테이블의 기존 5행에 4행이 더해져 **9행**이 된다.

이때 `INSERT ... SELECT`는 `VALUES`로 하나씩 값을 지정하는 대신, **SELECT 결과를 한 번에 삽입**하는 방식이다.

### UPDATE: 서브쿼리의 결과로 값 변경

```sql
UPDATE EMP_113
SET START_DATE = (
    SELECT START_DATE
    FROM S_EMP
    WHERE ID = 1
)
WHERE ID = 9;
```

안쪽에서 1번 직원 박구곤의 입사일을 구하고, 그 날짜를 복사 테이블의 9번 직원에게 설정한다. `SET`에 사용하는 이 서브쿼리는 **한 개의 값을 반환**해야 한다.

<br>

## 참고 자료

- 『SQL의 정석』 7장 — SUBQUERY
