# SQL 기초: Advanced SQL

## 목차

1. [NVL, DECODE, CASE: 값 변환과 조건 처리](#1-nvl-decode-case-값-변환과-조건-처리)

2. [PIVOT: 행 데이터를 열로 집계](#2-pivot-행-데이터를-열로-집계)

3. [ROLLUP과 CUBE: 소계와 총계](#3-rollup과-cube-소계와-총계)

4. [RANK: 순위 계산](#4-rank-순위-계산)

5. [상관 서브쿼리와 다중 행 비교 연산자](#5-상관-서브쿼리와-다중-행-비교-연산자)

6. [ROWID와 ROWNUM: 의사 컬럼](#6-rowid와-rownum-의사-컬럼)

7. [인덱스를 효율적으로 사용하는 조건](#7-인덱스를-효율적으로-사용하는-조건)

8. [TRIGGER: 자동으로 실행되는 프로그램](#8-trigger-자동으로-실행되는-프로그램)

<br>

## 1. NVL, DECODE, CASE: 값 변환과 조건 처리

`NVL`, `DECODE`, `CASE`는 **NULL을 처리하거나 조건에 따라 다른 값을 표시**할 때 사용한다.

| 구분                                    | 역할                                   | 기억할 점                                                     |
| --------------------------------------- | -------------------------------------- | ------------------------------------------------------------- |
| `NVL(값, 대체값)`                       | 값이 `NULL`이면 지정한 값으로 대체     | 두 인수는 서로 호환되는 자료형이어야 함                       |
| `DECODE(값, 비교값, 결과, ..., 기본값)` | 값이 **일치하는지** 비교하여 결과 반환 | Oracle 전용 함수. 일치하지 않으면 기본값, 생략 시 `NULL`      |
| `CASE ... END`                          | 조건에 따라 결과 반환                  | 값 비교뿐 아니라 `>=`, `BETWEEN` 등 **범위 조건**도 표현 가능 |

```sql
SELECT NAME,
       NVL(COMMISSION_PCT, 0) AS COMMISSION,
       DECODE(TITLE, '사원', '일반직', '과장', '관리직', '기타') AS TITLE_GROUP,
       CASE
           WHEN SALARY >= 3000 THEN '높음'
           WHEN SALARY >= 2000 THEN '보통'
           ELSE '낮음'
       END AS SALARY_GROUP
FROM S_EMP;
```

- **`DECODE`**는 `TITLE = '사원'`처럼 **값이 같은지** 비교하는 데 편리하다.
- **`CASE`**는 `CASE TITLE WHEN '사원' THEN ...`처럼 값으로 비교하거나, 위 예제처럼 `CASE WHEN 조건 THEN ...`으로 범위를 비교할 수 있다.

단순한 값 비교에는 `DECODE`도 가능하지만, **여러 조건을 읽기 쉽게 표현할 때는 `CASE`가 더 직관적**이다.

<br>

## 2. PIVOT: 행 데이터를 열로 집계

**`PIVOT`**은 여러 행에 흩어진 값을 **열로 펼쳐 집계**할 때 사용한다.

예를 들어 부서별 직원 직책을 행으로 나열하는 대신, **부서마다 사원과 과장이 몇 명인지** 한 줄에서 볼 수 있다.

```sql
SELECT *
FROM (SELECT DEPT_ID, TITLE FROM S_EMP)
PIVOT (
    COUNT(*)
    FOR TITLE IN ('사원' AS STAFF, '과장' AS MANAGER)
)
ORDER BY DEPT_ID;
```

| DEPT_ID | STAFF | MANAGER |
| ------- | ----: | ------: |
| 110     |     2 |       1 |
| 111     |     2 |       1 |
| 112     |     2 |       1 |
| 113     |     3 |       2 |

...생략

- `FOR TITLE`: 어떤 컬럼의 값을 열로 펼칠지 지정한다.
- `IN (...)`: **새 열로 만들 값**을 명시한다.
- `COUNT(*)`: 각 부서와 직책 조합에 해당하는 직원 수를 계산한다.

`PIVOT` 없이 **`DECODE`와 그룹 함수**를 조합해 같은 형태로 집계할 수도 있다.

```sql
SELECT DEPT_ID,
       COUNT(DECODE(TITLE, '사원', 1)) AS STAFF,
       COUNT(DECODE(TITLE, '과장', 1)) AS MANAGER
FROM S_EMP
GROUP BY DEPT_ID
ORDER BY DEPT_ID;
```

<br>

## 3. ROLLUP과 CUBE: 소계와 총계

일반 `GROUP BY`는 **그룹별 집계 결과**를 구하지만, `ROLLUP`과 `CUBE`는 **그룹별 소계와 전체 총계까지 함께 계산**한다.

- **`ROLLUP`**: 지정한 컬럼 순서에 따라 **단계적으로 소계와 총계**를 계산한다.
- **`CUBE`**: 지정한 컬럼들의 **가능한 모든 조합에 대한 소계와 총계**를 계산한다.

### ROLLUP: 부서별 소계와 전체 총계

```sql
SELECT DEPT_ID, TITLE, COUNT(*) AS CNT
FROM S_EMP
WHERE DEPT_ID IN (106, 112, 113)
GROUP BY ROLLUP(DEPT_ID, TITLE)
ORDER BY DEPT_ID, TITLE;
```

| DEPT_ID | TITLE        | CNT |
| ------- | ------------ | --: |
| 106     | 사원         |   1 |
| 106     | 영업대표이사 |   1 |
| 106     | NULL         |   2 |
| 112     | 과장         |   1 |
| 112     | 사원         |   2 |
| 112     | NULL         |   3 |
| 113     | 과장         |   2 |
| 113     | 사원         |   3 |
| 113     | NULL         |   5 |
| NULL    | NULL         |  10 |

`DEPT_ID = 106`인 직원은 사원 1명과 영업대표이사 1명이므로 **부서 소계는 2명**이다. 같은 방식으로 112번 부서는 3명, 113번 부서는 5명이며, 마지막 행은 **전체 총계 10명**을 나타낸다.

즉, `ROLLUP(DEPT_ID, TITLE)`은 **부서·직책별 인원 → 부서별 소계 → 전체 총계**를 계산한다.

### CUBE: 부서별·직책별 소계와 전체 총계

```sql
SELECT DEPT_ID, TITLE, COUNT(*) AS CNT
FROM S_EMP
WHERE DEPT_ID IN (106, 112, 113)
GROUP BY CUBE(DEPT_ID, TITLE)
ORDER BY DEPT_ID, TITLE;
```

**실행 결과**

| DEPT_ID | TITLE        | CNT |
| ------- | ------------ | --: |
| 106     | 사원         |   1 |
| 106     | 영업대표이사 |   1 |
| 106     | NULL         |   2 |
| 112     | 과장         |   1 |
| 112     | 사원         |   2 |
| 112     | NULL         |   3 |
| 113     | 과장         |   2 |
| 113     | 사원         |   3 |
| 113     | NULL         |   5 |
| NULL    | 과장         |   3 |
| NULL    | 사원         |   6 |
| NULL    | 영업대표이사 |   1 |
| NULL    | NULL         |  10 |

`CUBE`는 `ROLLUP`의 결과에 **직책별 소계**까지 추가한다.

예를 들어 `DEPT_ID = NULL`, `TITLE = '사원'`, `CNT = 6`인 행은 **세 부서에 속한 사원이 총 6명**이라는 의미다.

즉, `CUBE(DEPT_ID, TITLE)`은 **부서·직책별 인원 → 부서별 소계 + 직책별 소계 → 전체 총계**를 계산한다.

**핵심 차이:** `ROLLUP`은 **지정한 컬럼 순서에 따른 단계별 소계**, `CUBE`는 **모든 컬럼 조합에 대한 소계**를 계산한다.

### GROUPING(): 소계와 총계 구분하기

`ROLLUP`과 `CUBE`의 결과에서 **소계·총계 계산으로 제외된 컬럼은 `NULL`로 표시**된다.

`GROUPING()` 함수는 해당 컬럼이 집계 기준에서 제외되었다면 `1`, 그렇지 않다면 `0`을 반환한다. 이를 `CASE`와 함께 사용하면 **집계용 `NULL`을 '부서 소계', '전체 총계'처럼 알아보기 쉽게 표시**할 수 있다.

<details>

<summary>💡 GROUPING() 활용 예제</summary>

```sql
SELECT
    CASE
        WHEN GROUPING(DEPT_ID) = 1 THEN '전체 부서'
        ELSE TO_CHAR(DEPT_ID)
    END AS 부서,
    CASE
        WHEN GROUPING(DEPT_ID) = 1
             AND GROUPING(TITLE) = 1 THEN '전체 총계'
        WHEN GROUPING(TITLE) = 1 THEN '부서 소계'
        ELSE TITLE
    END AS 직책,
    COUNT(*) AS 인원
FROM S_EMP
WHERE DEPT_ID IN (106, 112, 113)
GROUP BY CUBE(DEPT_ID, TITLE)
ORDER BY GROUPING(DEPT_ID), DEPT_ID,
         GROUPING(TITLE), TITLE;
```

**실행 결과**

| 부서      | 직책          |   인원 |
| --------- | ------------- | -----: |
| 106       | 사원          |      1 |
| 106       | 영업대표이사  |      1 |
| 106       | **부서 소계** |  **2** |
| 112       | 과장          |      1 |
| 112       | 사원          |      2 |
| 112       | **부서 소계** |  **3** |
| 113       | 과장          |      2 |
| 113       | 사원          |      3 |
| 113       | **부서 소계** |  **5** |
| 전체 부서 | 과장          |      3 |
| 전체 부서 | 사원          |      6 |
| 전체 부서 | 영업대표이사  |      1 |
| 전체 부서 | **전체 총계** | **10** |

</details>

<br>

## 4. RANK: 순위 계산

**`RANK()`**는 데이터의 순위를 계산하는 **분석 함수(Analytic Function)**다. `GROUP BY`와 달리 **각 행을 그대로 유지하면서 순위를 계산**할 수 있다.

```sql
SELECT DEPT_ID, NAME, SALARY,
       RANK() OVER (
           PARTITION BY DEPT_ID
           ORDER BY SALARY DESC
       ) AS SALARY_RANK
FROM S_EMP
WHERE DEPT_ID IN (112, 113)
ORDER BY DEPT_ID, SALARY_RANK, ID;
```

- **`PARTITION BY DEPT_ID`**: 부서별로 데이터를 나누어 **각 부서의 순위를 따로 계산**한다. 생략하면 조회 대상 전체에서 순위를 계산한다.
- **`ORDER BY SALARY DESC`**: 각 부서 내에서 급여가 높은 순서대로 순위를 매긴다.
- **`RANK()`**: 같은 급여는 같은 순위를 부여하며, **동점자 수만큼 다음 순위가 건너뛰어진다.**

예를 들어 113번 부서에서 급여가 `2300`인 직원 두 명은 모두 **1위**가 되고, 다음 급여가 `1100`인 직원은 **3위**가 된다.

또한 112번 부서와 113번 부서는 **각각 독립적으로 1위부터 순위가 계산**된다.

참고로 **`DENSE_RANK()`**는 동점 이후에도 순위를 건너뛰지 않는다.

- `RANK()`: 1위, 1위, 3위
- `DENSE_RANK()`: 1위, 1위, 2위

**주의:** `OVER` 내부의 `ORDER BY`는 **순위 계산 기준**이고, SELECT문 마지막의 `ORDER BY`는 **최종 조회 결과의 정렬 순서**를 지정한다.

<br>

## 5. 상관 서브쿼리와 다중 행 비교 연산자

### 상관 서브쿼리(Correlated Subquery)

일반 서브쿼리와 상관 서브쿼리는 **메인 쿼리와 서브쿼리의 관계**에 차이가 있다.

- **일반 서브쿼리**: 서브쿼리의 결과를 먼저 구하고, 그 결과를 메인 쿼리에서 사용하는 방식으로 이해할 수 있다.
- **상관 서브쿼리**: 메인 쿼리의 행을 기준으로, **그 행의 값을 서브쿼리에서 참조**하여 조건을 판단한다.

예를 들어 직원의 급여를 평균 급여와 비교한다고 가정해 보자.

**① 일반 서브쿼리: 전체 직원의 평균 급여와 비교**

```sql
SELECT e.NAME, e.SALARY, e.DEPT_ID
FROM S_EMP e
WHERE e.SALARY < (
    SELECT AVG(s.SALARY)
    FROM S_EMP s
);
```

서브쿼리에서 **전체 직원의 평균 급여**를 구하고, 메인 쿼리에서 각 직원의 급여와 비교한다.

**② 상관 서브쿼리: 자신이 속한 부서의 평균 급여와 비교**

```sql
SELECT e.NAME, e.SALARY, e.DEPT_ID
FROM S_EMP e
WHERE e.SALARY < (
    SELECT AVG(s.SALARY)
    FROM S_EMP s
    WHERE s.DEPT_ID = e.DEPT_ID
);
```

일반 서브쿼리와 달리 `WHERE s.DEPT_ID = e.DEPT_ID` 조건이 추가되었다.

여기서 `e.DEPT_ID`는 **메인 쿼리에서 현재 처리 중인 직원의 부서 번호**를 의미한다. 서브쿼리가 이 값을 참조하므로 직원마다 자신의 부서 평균 급여를 기준으로 비교할 수 있다.

**논리적인 처리 순서**

1. **메인 쿼리**에서 직원 한 명을 기준으로 잡는다.

2. **서브쿼리**에서 해당 직원의 부서 번호를 참조하여 부서 평균 급여를 구한다.

3. 직원의 급여가 부서 평균보다 낮으면 결과에 포함한다.

4. 다른 직원도 같은 방식으로 비교한다.

즉, 일반 서브쿼리는 **독립적으로 구한 결과를 메인 쿼리에서 사용**하지만, 상관 서브쿼리는 **메인 쿼리의 값에 의존하여 결과를 구한다**는 차이가 있다.

> **참고:** 위 처리 순서는 개념을 이해하기 위한 **논리적인 설명**이다. 실제 SQL 실행 순서는 옵티마이저가 선택한 실행계획에 따라 달라질 수 있다.

### 다중 행 비교 연산자

| 연산자          | 의미                                                       |
| --------------- | ---------------------------------------------------------- |
| `IN` / `NOT IN` | 서브쿼리 결과 목록에 **포함되는지 / 포함되지 않는지** 비교 |
| `> ANY` 등      | 반환된 값 중 **하나라도** 비교 조건을 만족하면 참          |
| `> ALL` 등      | 반환된 **모든 값에 대해** 비교 조건을 만족해야 참          |
| `EXISTS`        | 서브쿼리 결과가 **한 행이라도 존재**하면 참                |
| `NOT EXISTS`    | 서브쿼리 결과가 **한 행도 없으면** 참                      |

예를 들어 `SALARY > ANY (750, 850)`은 둘 중 **하나보다만 커도** 되고, `SALARY > ALL (750, 850)`은 **둘 다보다 커야** 한다.

```sql
-- 다른 직원의 관리자로 지정된 직원만 조회
SELECT e.ID, e.NAME, e.TITLE
FROM S_EMP e
WHERE EXISTS (
    SELECT 1
    FROM S_EMP s
    WHERE s.MANAGER_ID = e.ID
)
ORDER BY e.ID;
```

`EXISTS`는 **반환하는 값이 아니라 행이 존재하는지**만 확인한다. 따라서 `SELECT 1`처럼 작성해도 된다. 같은 조건에서 `NOT EXISTS`로 바꾸면 **다른 직원의 관리자로 지정되지 않은 직원**을 찾는다.

**`NOT IN`과 NULL 주의:** 결과 목록에 `NULL`이 포함되면 비교 결과가 `UNKNOWN`이 되어 **예상과 달리 조회 결과가 없을 수 있다.** 존재 여부를 비교할 때는 `NOT EXISTS`를 활용하거나 서브쿼리에서 `NULL`을 제외하는 방식이 안전하다.

<br>

## 6. ROWID와 ROWNUM: 의사 컬럼

**의사 컬럼(Pseudo column)**은 실제 테이블 컬럼으로 저장되지는 않지만, SQL에서 컬럼처럼 조회할 수 있는 값이다.

| 구분     | 의미                                                 | 핵심 용도                                                    |
| -------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| `ROWID`  | 행이 저장된 위치를 나타내는 주소 정보                | 특정 행에 직접 접근. 데이터 객체·파일·블록·행 위치 정보 포함 |
| `ROWNUM` | Oracle이 조회 과정에서 행을 선택하면서 부여하는 순번 | 조회 행 수 제한                                              |

```sql
SELECT ID, NAME, ROWID
FROM S_EMP
WHERE ID <= 3;
```

`ROWID`는 인덱스가 찾은 행의 위치로 테이블에 접근할 때 활용된다. 다만 **행 이동·테이블 재구성 등으로 값이 달라질 수 있으므로 PK 대신 사용하지는 않는다.**

### 정렬한 다음 상위 5행 조회하기

`ROWNUM`은 **행을 조회하는 과정에서 부여되는 순번**이므로, `ORDER BY`로 정렬한 순위를 의미하지 않는다.

따라서 `ROWNUM`과 `ORDER BY`를 같은 SELECT문에서 사용하면 **먼저 선택된 5행만 정렬되는 상황**이 발생할 수 있다.

```sql
-- 먼저 5행을 선택한 뒤, 선택된 행만 정렬될 수 있음
SELECT NAME, SALARY
FROM S_EMP
WHERE ROWNUM <= 5
ORDER BY SALARY DESC;
```

**급여가 높은 직원 5명**을 정확히 조회하려면, 전체 데이터를 급여순으로 정렬한 다음 상위 5행을 선택해야 한다.

**① 인라인 뷰를 사용하는 방법**

```sql
SELECT NAME, SALARY
FROM (
    SELECT NAME, SALARY
    FROM S_EMP
    ORDER BY SALARY DESC
)
WHERE ROWNUM <= 5;
```

- **내부 쿼리**: 전체 직원을 급여가 높은 순서로 정렬한다.
- **외부 쿼리**: 정렬된 결과에서 처음 5행을 선택한다.

**② FETCH FIRST를 사용하는 방법**

```sql
SELECT NAME, SALARY
FROM S_EMP
ORDER BY SALARY DESC
FETCH FIRST 5 ROWS ONLY;
```

`FETCH FIRST`를 사용하면 인라인 뷰 없이도 **정렬된 결과에서 상위 N행을 간단하게 조회**할 수 있다.

`ROWNUM`은 조회 과정에서 부여되는 순번이므로, **정렬된 결과에서 상위 N행을 구하려면 인라인 뷰 또는 `FETCH FIRST`를 사용**한다.

<br>

## 7. 인덱스를 효율적으로 사용하는 조건

인덱스가 있어도 **검색 조건을 어떻게 작성했는지, 조회 대상이 얼마나 많은지**에 따라 인덱스를 활용하지 않을 수 있다.

```sql
-- 인덱스 컬럼 SALARY에 연산을 적용한 경우
SELECT ID, NAME, SALARY
FROM S_EMP
WHERE SALARY * 12 = 24000;
-- 컬럼은 그대로 두고 비교값을 계산한 경우
SELECT ID, NAME, SALARY
FROM S_EMP
WHERE SALARY = 24000 / 12;
```

일반적인 `SALARY` 인덱스를 가정하면 **컬럼 자체를 가공하지 않은 두 번째 조건이 인덱스를 활용하기에 더 유리**하다. 컬럼을 가공해야 한다면 함수 기반 인덱스(Function-Based Index)가 대안이 될 수도 있다.

- `LIKE '영업%'`처럼 **앞부분이 정해진 검색**은 일반적인 B-Tree 인덱스의 범위 탐색에 활용될 수 있다.
- `LIKE '%부장'`처럼 **앞에 `%`가 붙은 검색**은 일반적인 인덱스 범위 탐색이 어렵다.
- `<>`, `NOT`, `IS NULL` 조건도 인덱스 활용이 제한될 수 있지만, **무조건 인덱스를 사용하지 않는다고 단정할 수는 없다.**

<br>

## 8. TRIGGER: 자동으로 실행되는 프로그램

**트리거(Trigger)**는 사용자가 직접 호출하는 것이 아니라, 테이블에 **`INSERT`·`UPDATE`·`DELETE` 같은 이벤트가 발생했을 때 DBMS가 자동으로 실행하는 프로그램**이다. 변경 이력을 기록하거나 특정 처리 규칙을 적용할 때 사용할 수 있다.

| 구분          | 종류                    | 의미                                                    |
| ------------- | ----------------------- | ------------------------------------------------------- |
| **실행 시점** | `BEFORE` / `AFTER`      | DML 실행 전 / 후                                        |
| **실행 단위** | 문장 트리거 / 행 트리거 | SQL문당 한 번 / 영향을 받은 행마다 실행(`FOR EACH ROW`) |

```sql
CREATE TABLE ADV_TRG_TEST (
    ID         NUMBER PRIMARY KEY,
    CREATED_AT DATE
);

CREATE OR REPLACE TRIGGER ADV_TRG_BI
BEFORE INSERT ON ADV_TRG_TEST
FOR EACH ROW
BEGIN
    IF :NEW.CREATED_AT IS NULL THEN
        :NEW.CREATED_AT := SYSDATE;
    END IF;
END;
/

INSERT INTO ADV_TRG_TEST (ID) VALUES (1);
SELECT ID, CREATED_AT FROM ADV_TRG_TEST;
-- 입력 시각이 자동으로 설정됨

DROP TABLE ADV_TRG_TEST;
```

- `BEFORE INSERT`: 행이 삽입되기 전에 실행한다.
- `FOR EACH ROW`: 삽입되는 **각 행마다** 실행한다.
- `:NEW`: 새로 입력될 행의 값을 참조한다. (`UPDATE`·`DELETE` 등에서는 `:OLD`도 사용한다.)

트리거는 자동 실행되므로 **동작이 눈에 보이지 않아 관리가 복잡해질 수 있다.** 단순한 기본값이나 데이터 검증은 `DEFAULT`, `CHECK` 같은 기능으로 해결할 수 있는지도 먼저 살펴보자.

<br>

## 참고 자료

- 『SQL의 정석』 14장 — ADVANCED SQL
