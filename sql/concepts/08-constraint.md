# SQL 기초: 제약조건(Constraint)

## 목차

1. [제약조건의 개념](#1-제약조건의-개념)
2. [제약조건의 종류](#2-제약조건의-종류)
3. [제약조건 정의 방법](#3-제약조건-정의-방법)
4. [FOREIGN KEY와 참조 무결성](#4-foreign-key와-참조-무결성)
5. [DML 적용과 제약조건 확인](#5-dml-적용과-제약조건-확인)

<br>

## 1. 제약조건의 개념

**제약조건(Constraint)**은 테이블에 저장되는 데이터가 정해진 규칙을 지키도록 하여 **데이터 무결성(정확성·일관성)**을 유지하는 장치이다.

데이터의 입력·수정·삭제 과정에서 잘못된 값이 저장되거나 참조 관계가 깨지는 것을 데이터베이스 수준에서 방지한다.

- 제약조건은 데이터 사전(Data Dictionary)에 저장된다.
- `EMP_ID_PK`, `EMP_DEPT_FK`처럼 의미 있는 이름을 붙이면 관리하기 쉽다. 같은 스키마에서 제약조건 이름은 중복할 수 없다.
- 이름을 생략하면 Oracle이 `SYS_Cn..` 형태의 이름을 자동으로 만든다.

<br>

## 2. 제약조건의 종류

| 제약조건      | 역할                     | 핵심 특징                                                                              |
| ------------- | ------------------------ | -------------------------------------------------------------------------------------- |
| `NOT NULL`    | 값 누락 방지             | `NULL`을 허용하지 않음                                                                 |
| `UNIQUE`      | 값의 중복 방지           | 중복은 불가하지만 `NULL`은 허용할 수 있음                                              |
| `PRIMARY KEY` | 행을 고유하게 식별       | `UNIQUE` + `NOT NULL`; 테이블당 하나의 PK 제약조건                                     |
| `FOREIGN KEY` | 테이블 간 참조 관계 유지 | 자식의 값은 부모의 PK/UNIQUE 값과 일치하거나 `NULL`이어야 함(별도 `NOT NULL`이 없다면) |
| `CHECK`       | 값의 범위·조건 제한      | 예: 급여는 0 이상 10,000 이하                                                          |

**`PRIMARY KEY`와 `UNIQUE`의 차이:** 둘 다 중복을 막지만 PK는 `NULL`도 허용하지 않는다. PK 제약조건은 테이블당 하나만 정의할 수 있어도 **여러 컬럼을 묶은 복합 기본 키**는 가능하다. `UNIQUE` 역시 여러 컬럼의 조합에 적용할 수 있다. Oracle은 PK·UNIQUE를 검사할 때 인덱스를 활용하며, 적절한 인덱스가 없다면 자동으로 생성할 수 있다.

`CHECK (SALARY >= 0)`은 `SALARY`가 `NULL`일 때도 위반으로 판단하지 않는다. **값을 반드시 입력받으려면 `NOT NULL`을 별도로 지정**해야 한다.

<br>

## 3. 제약조건 정의 방법

제약조건은 컬럼 정의 바로 뒤에 적는 **컬럼 레벨(Column Level)**과 컬럼 목록 다음에 적는 **테이블 레벨(Table Level)** 방식으로 정의할 수 있다.

- **컬럼 레벨:** 한 컬럼에 제약조건을 지정할 때 직관적이다. `NOT NULL`은 이 방식으로만 정의한다.
- **테이블 레벨:** 여러 컬럼을 묶은 복합 PK·UNIQUE·FK를 정의할 수 있다. `NOT NULL`을 제외한 제약조건에 사용한다.

아래는 두 방식을 함께 적용한 예제다. 기존 실습 테이블 `S_EMP`, `S_DEPT`를 변경하지 않도록 별도 테스트 테이블을 만든다.

```sql
-- 부모 테이블
CREATE TABLE DEPT_TEST (
    ID   NUMBER(5) CONSTRAINT DEPT_TEST_ID_PK PRIMARY KEY,
    NAME VARCHAR2(30 CHAR) NOT NULL
);

-- 자식 테이블
CREATE TABLE EMP_TEST (
    ID      NUMBER(5),
    NAME    VARCHAR2(30 CHAR) CONSTRAINT EMP_TEST_NAME_NN NOT NULL,
    EMAIL   VARCHAR2(100),
    DEPT_ID NUMBER(5),
    SALARY  NUMBER(9, 2),
    CONSTRAINT EMP_TEST_ID_PK PRIMARY KEY (ID),
    CONSTRAINT EMP_TEST_EMAIL_UK UNIQUE (EMAIL),
    CONSTRAINT EMP_TEST_DEPT_FK FOREIGN KEY (DEPT_ID) REFERENCES DEPT_TEST(ID),
    CONSTRAINT EMP_TEST_SALARY_CK CHECK (SALARY BETWEEN 0 AND 10000)
);
```

`DEPT_TEST.ID`와 `EMP_TEST.NAME`은 컬럼 레벨, `EMP_TEST.ID`·`EMAIL`·`DEPT_ID`·`SALARY`의 제약조건은 테이블 레벨로 정의했다.

제약조건은 나중에 `ALTER TABLE`로 추가·변경·삭제할 수도 있다.

<br>

## 4. FOREIGN KEY와 참조 무결성

**외래 키(FK)**는 자식 테이블에서 부모 테이블의 **PK 또는 UNIQUE 키**를 참조해 잘못된 관계가 저장되지 않도록 한다.

ex) S_EMP 테이블의 `S_EMP.DEPT_ID`가 `S_DEPT.ID`를 참조하는 관계

- **부모 테이블·참조 키가 먼저 존재**해야 하며, 두 컬럼은 서로 호환되는 자료형이어야 한다.
- 기본적으로 부모에 없는 값을 자식에 입력하거나 수정할 수 없다.
- 자식이 참조 중인 부모 행도 기본적으로 삭제할 수 없다.
- `ON DELETE CASCADE`를 FK에 지정하면 **부모 행 삭제 시 이를 참조하는 자식 행도 함께 삭제**된다. 편리하지만 의도하지 않은 연쇄 삭제에 주의해야 한다.

`CHECK`는 같은 행의 값에 대한 조건을 검사한다. **다른 테이블·행을 조회하는 서브쿼리나 `SYSDATE` 같은 비결정적 함수, `ROWNUM`·`NEXTVAL` 등은 CHECK 조건에서 사용할 수 없다.**

다른 테이블과의 관계는 `CHECK`가 아니라 주로 `FOREIGN KEY`로 검증한다.

<br>

## 5. DML 적용과 제약조건 확인

제약조건은 테이블을 만들 때뿐 아니라 데이터를 **입력·수정·삭제할 때도 검사**된다.

```sql
-- 정상: 부모 부서를 먼저 등록하고 직원을 추가
INSERT INTO DEPT_TEST (ID, NAME) VALUES (10, '개발부');
INSERT INTO EMP_TEST (ID, NAME, EMAIL, DEPT_ID, SALARY)
VALUES (1, '홍길동', 'hong@example.com', 10, 3000);

-- 오류: 부모 테이블에 없는 999번 부서를 참조
INSERT INTO EMP_TEST (ID, NAME, DEPT_ID, SALARY)
VALUES (2, '김철수', 999, 2000);
```

마지막 `INSERT`는 `FOREIGN KEY`를 위반하므로 실패한다. 이 밖에도 **중복된 PK·UNIQUE 값, `NULL`인 필수 컬럼, CHECK 범위 밖의 값**을 입력·수정하면 오류가 발생한다.

자식이 참조하는 부모 행을 삭제하는 경우에도 `ON DELETE CASCADE` 등의 설정이 없다면 참조 무결성 오류가 발생한다.

제약조건 이름과 종류는 데이터 사전에서 확인할 수 있다.

```sql
SELECT TABLE_NAME, CONSTRAINT_NAME, CONSTRAINT_TYPE
FROM USER_CONSTRAINTS
WHERE TABLE_NAME IN ('DEPT_TEST', 'EMP_TEST');
```

`CONSTRAINT_TYPE`의 대표 값은 `P`(PK), `R`(FK), `U`(UNIQUE), `C`(CHECK 및 NOT NULL)이다. 해당 제약조건에 속한 컬럼은 `USER_CONS_COLUMNS`에서 확인할 수 있다.

<br>

## 참고 자료

- 『SQL의 정석』 8장 — CONSTRAINT (8.1~8.7, 8.9)
