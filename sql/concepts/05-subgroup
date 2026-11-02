# SQL 기초: 데이터 그룹화(GROUP BY와 HAVING)

## 목차

1. [그룹화와 그룹 함수](#1-그룹화와-그룹-함수)
2. [GROUP BY: 그룹별 집계](#2-group-by-그룹별-집계)
3. [WHERE와 HAVING](#3-where와-having)
4. [여러 컬럼으로 그룹화](#4-여러-컬럼으로-그룹화)
5. [그룹화할 때 주의할 점](#5-그룹화할-때-주의할-점)
6. [SQL 작성 순서와 논리적 처리 순서](#6-sql-작성-순서와-논리적-처리-순서)

<br>

## 1. 그룹화와 그룹 함수

4장에서 배운 `COUNT`, `SUM`, `AVG` 같은 **그룹 함수(집계 함수)**는 여러 행의 값을 모아 하나의 결과를 계산한다.

예를 들어 `S_EMP`에서 직원 수와 평균 급여를 구하면 다음과 같다.

```sql
SELECT COUNT(*) AS EMP_COUNT,
       ROUND(AVG(SALARY), 2) AS AVG_SALARY
FROM S_EMP;
```

| EMP_COUNT | AVG_SALARY |
| --------- | ---------- |
| 25        | 1695.6     |

이 쿼리는 직원 전체를 하나의 집합으로 보고 집계한다. 그런데 **부서별 직원 수나 평균 급여**를 알고 싶다면 전체를 한 번에 계산하는 것만으로는 부족하다. 이때 사용하는 것이 `GROUP BY`다.

<br>

## 2. GROUP BY: 그룹별 집계

`GROUP BY`는 **같은 값을 가진 행들을 하나의 그룹으로 묶고, 그룹마다 집계 결과를 구할 때** 사용한다.

```sql
SELECT DEPT_ID,
       COUNT(*) AS EMP_COUNT,
       ROUND(AVG(SALARY), 2) AS AVG_SALARY
FROM S_EMP
GROUP BY DEPT_ID
ORDER BY DEPT_ID;
```

위 쿼리는 같은 `DEPT_ID`를 가진 직원들을 묶어서 부서별 직원 수와 평균 급여를 계산한다.

**결과 중 일부**

| DEPT_ID | EMP_COUNT | AVG_SALARY |
| ------- | --------- | ---------- |
| 110     | 4         | 1585       |
| 111     | 3         | 1465       |
| 113     | 5         | 1472       |
| 118     | 2         | 4100       |

- `GROUP BY DEPT_ID`: 부서 번호가 같은 직원들을 하나의 그룹으로 묶는다.
- `COUNT(*)`: 각 부서에 속한 직원 수를 센다.
- `AVG(SALARY)`: 각 부서의 평균 급여를 계산한다.
- `ORDER BY DEPT_ID`: 조회 결과를 부서 번호순으로 정렬한다.

`GROUP BY`로 묶인 여러 행은 **그룹당 한 행의 집계 결과**로 나타난다.

<br>

## 3. WHERE와 HAVING

`WHERE`와 `HAVING`은 모두 조건에 맞는 데이터를 필터링할 때 사용하지만, **조건을 적용하는 대상과 시점이 다르다.**

| 구분           | WHERE                   | HAVING                             |
| -------------- | ----------------------- | ---------------------------------- |
| 적용 시점      | `GROUP BY` 이전         | `GROUP BY` 이후                    |
| 적용 대상      | 개별 행(Row)            | 그룹(Group)                        |
| 목적           | 조건에 맞는 행을 필터링 | 조건에 맞는 그룹을 필터링          |
| 집계 함수 조건 | 직접 사용할 수 없음     | `COUNT`, `AVG`, `SUM` 등 사용 가능 |

`WHERE`는 일반적인 데이터 조회에서 사용하며, `GROUP BY`와 함께 사용하면 그룹화하기 전에 조건에 맞는 행만 선택한다. 반면 `HAVING`은 그룹화된 결과에 조건을 적용할 때 사용한다.

### WHERE: 조건에 맞는 행 선택

직책이 `사원`인 직원들만 대상으로 부서별 인원과 평균 급여를 구해 보자.

```sql
SELECT DEPT_ID,
       COUNT(*) AS EMP_COUNT,
       ROUND(AVG(SALARY), 2) AS AVG_SALARY
FROM S_EMP
WHERE TITLE = '사원'
GROUP BY DEPT_ID
ORDER BY DEPT_ID;
```

**결과**

| DEPT_ID | EMP_COUNT | AVG_SALARY |
| ------- | --------- | ---------- |
| 106     | 1         | 795        |
| 110     | 2         | 470        |
| 111     | 2         | 997.5      |
| 112     | 2         | 800        |
| 113     | 3         | 920        |

`WHERE TITLE = '사원'`으로 사원이 아닌 직원을 먼저 제외하고, **남은 직원들만 부서별로 그룹화하여 집계한다.**

### HAVING: 조건에 맞는 그룹 선택

이번에는 직책이 `사원`인 직원들 중 **사원이 3명 이상인 부서**만 조회해 보자.

```sql
SELECT DEPT_ID,
       COUNT(*) AS EMP_COUNT,
       ROUND(AVG(SALARY), 2) AS AVG_SALARY
FROM S_EMP
WHERE TITLE = '사원'
GROUP BY DEPT_ID
HAVING COUNT(*) >= 3
ORDER BY DEPT_ID;
```

**결과**

| DEPT_ID | EMP_COUNT | AVG_SALARY |
| ------- | --------- | ---------- |
| 113     | 3         | 920        |

1. `WHERE`에서 직책이 `사원`인 직원만 선택한다.
2. `GROUP BY`에서 선택된 직원들을 부서별로 묶는다.
3. `HAVING`에서 사원이 3명 이상인 부서만 남긴다.
4. `SELECT`에서 부서별 직원 수와 평균 급여를 출력한다.

즉, **`WHERE`는 개별 행을 필터링하고, `HAVING`은 그룹화한 결과를 필터링한다.** 두 절을 함께 사용하면 원하는 행만 먼저 선택한 뒤, 집계 결과에 추가 조건을 적용할 수 있다.

<br>

## 4. 여러 컬럼으로 그룹화

`GROUP BY`에 여러 컬럼을 지정하면 **컬럼들의 값 조합이 같은 행끼리** 그룹으로 묶는다.

```sql
SELECT DEPT_ID, TITLE, COUNT(*) AS EMP_COUNT
FROM S_EMP
WHERE DEPT_ID IN (110, 113)
GROUP BY DEPT_ID, TITLE
HAVING COUNT(*) >= 2
ORDER BY DEPT_ID, TITLE;
```

**결과**

| DEPT_ID | TITLE | EMP_COUNT |
| ------- | ----- | --------- |
| 110     | 사원  | 2         |
| 113     | 과장  | 2         |
| 113     | 사원  | 3         |

- `WHERE`: 부서 번호가 `110` 또는 `113`인 직원만 선택한다.
- `GROUP BY`: 선택된 직원들을 부서 번호와 직책의 조합으로 그룹화한다.
- `HAVING`: 각 그룹에서 직원 수가 2명 이상인 경우만 남긴다.

`GROUP BY DEPT_ID`만 사용하면 **부서별 인원**을 구하지만, `GROUP BY DEPT_ID, TITLE`을 사용하면 **각 부서 안에서 직책별 인원**까지 나누어 계산할 수 있다.

이처럼 여러 컬럼으로 그룹화하면 데이터를 더 세분화할 수 있고, `HAVING`을 사용하면 세분화한 그룹 중 필요한 결과만 조회할 수 있다.

<br>

## 5. 그룹화할 때 주의할 점

### SELECT에는 그룹 기준 컬럼이나 집계 결과를 사용

`GROUP BY`를 사용하면 조회 결과는 그룹 단위가 된다. 따라서 일반적으로 `SELECT`에는 **`GROUP BY`에 사용한 컬럼이나 집계 함수**를 작성해야 한다.

```sql
-- 오류: DEPT_ID로 묶었지만 NAME은 그룹별로 하나의 값을 정할 수 없다.
SELECT DEPT_ID, NAME, COUNT(*)
FROM S_EMP
GROUP BY DEPT_ID;
```

같은 부서에는 여러 직원이 있으므로, 그룹별 한 행에서 어떤 직원의 `NAME`을 보여줘야 할지 결정할 수 없다.

```sql
-- 올바른 예: 그룹 기준 컬럼과 집계 결과를 조회
SELECT DEPT_ID, COUNT(*) AS EMP_COUNT
FROM S_EMP
GROUP BY DEPT_ID;
```

### 집계 결과를 WHERE에서 비교할 수 없음

`WHERE`는 그룹을 만들기 전에 행을 필터링하므로 `AVG(SALARY)`처럼 **집계한 결과에 대한 조건**을 직접 사용할 수 없다.

```sql
-- 오류: WHERE에서는 AVG(SALARY)로 조건을 걸 수 없다.
SELECT DEPT_ID, AVG(SALARY)
FROM S_EMP
WHERE AVG(SALARY) >= 2000
GROUP BY DEPT_ID;
```

```sql
-- 올바른 예: 그룹별 평균에 대한 조건은 HAVING 사용
SELECT DEPT_ID, AVG(SALARY) AS AVG_SALARY
FROM S_EMP
GROUP BY DEPT_ID
HAVING AVG(SALARY) >= 2000;
```

또한 집계 함수는 일반적으로 `NULL`을 계산에서 제외한다. `COUNT(*)`는 행 자체를 세지만, `COUNT(SALARY)`와 `AVG(SALARY)`는 `SALARY`가 `NULL`인 행을 제외한다.

**숫자 `0`은 `NULL`이 아니므로 집계에 포함된다.**

<br>

## 6. SQL 작성 순서와 논리적 처리 순서

`GROUP BY`와 `HAVING`을 이해하려면 SQL을 **작성하는 순서**와 **논리적으로 처리하는 순서**를 구분하는 것이 중요하다.

### SQL 작성 순서

```sql
SELECT DEPT_ID, COUNT(*) AS EMP_COUNT, AVG(SALARY) AS AVG_SALARY
FROM S_EMP
WHERE TITLE = '사원'
GROUP BY DEPT_ID
HAVING AVG(SALARY) >= 900
ORDER BY AVG_SALARY DESC;
```

### SQL 논리적 처리 순서

```text
FROM       → 데이터를 가져올 테이블 결정
WHERE      → 조건에 맞는 개별 행 선택
GROUP BY   → 선택된 행을 기준별로 그룹화
HAVING     → 집계 조건을 만족하는 그룹 선택
SELECT     → 출력할 컬럼과 집계 결과 계산
ORDER BY   → 최종 결과 정렬
```

위 쿼리에 적용하면 다음 흐름이 된다.

1. **FROM:** `S_EMP`에서 직원 데이터를 가져온다.
2. **WHERE:** 직책이 `사원`인 직원만 남긴다.
3. **GROUP BY:** 남은 직원들을 `DEPT_ID`별로 묶는다.
4. **HAVING:** 평균 급여가 900 이상인 부서만 남긴다.
5. **SELECT:** 부서 번호, 직원 수, 평균 급여를 결과로 만든다.
6. **ORDER BY:** 평균 급여가 높은 부서부터 보여준다.

**핵심은 `WHERE`가 그룹화 이전에, `HAVING`이 그룹화 이후에 적용된다는 점이다.**

이 순서는 SQL의 의미를 이해하기 위한 **논리적 처리 순서**이며, 데이터베이스가 실제로 데이터를 읽고 처리하는 물리적인 실행 계획과 반드시 같지는 않다.

<br>

## 참고 자료

- 『SQL의 정석』 5장
