# SQL 기초: 데이터 정의어(DDL)

## 목차

1. [DDL의 개념과 주요 명령어](#1-ddl의-개념과-주요-명령어)
2. [CREATE TABLE: 테이블 생성](#2-create-table-테이블-생성)
3. [DROP TABLE: 테이블 삭제](#3-drop-table-테이블-삭제)
4. [ALTER TABLE: 테이블 구조와 제약조건 변경](#4-alter-table-테이블-구조와-제약조건-변경)
5. [RENAME, TRUNCATE, COMMENT](#5-rename-truncate-comment)

<br>

## 1. DDL의 개념과 주요 명령어

**DDL(Data Definition Language, 데이터 정의어)**은 테이블과 같은 **데이터베이스 객체의 구조를 생성·변경·삭제**하는 SQL 명령어다.

| 명령어           | 역할                                           |
| ---------------- | ---------------------------------------------- |
| `CREATE TABLE`   | 새 테이블 생성                                 |
| `ALTER TABLE`    | 기존 테이블의 컬럼·제약조건 변경               |
| `DROP TABLE`     | 테이블 구조와 데이터 삭제                      |
| `TRUNCATE TABLE` | 구조는 유지하고 모든 행 삭제 및 저장 공간 관리 |
| `RENAME`         | 객체 이름 변경                                 |
| `COMMENT`        | 테이블·컬럼에 설명(주석) 등록                  |

**Oracle의 일반적인 DDL은 실행 전에 암묵적 `COMMIT`이 발생하고, 성공하면 다시 `COMMIT`된다.** 따라서 일반적인 `ROLLBACK`으로 DDL을 되돌릴 수 없으며, 앞서 실행한 미확정 DML까지 확정될 수 있다.

<br>

## 2. CREATE TABLE: 테이블 생성

`CREATE TABLE`은 **컬럼 이름·자료형·제약조건 등을 정의하여 새로운 테이블을 생성**한다. 별도로 데이터를 넣지 않으면 처음에는 빈 테이블이다.

```sql
CREATE TABLE DDL_EMP_TEST (
    ID     NUMBER(5) CONSTRAINT DDL_EMP_ID_PK PRIMARY KEY,
    NAME   VARCHAR2(30 CHAR) NOT NULL,
    SALARY NUMBER(9, 2) DEFAULT 0
);
```

- `NUMBER(5)`: 최대 5자리 정수, `VARCHAR2(30 CHAR)`: 최대 30문자 문자열, `NUMBER(9, 2)`: 전체 9자리 중 소수점 이하 2자리 숫자.
- `PRIMARY KEY`, `NOT NULL`: 테이블 생성과 함께 제약조건을 지정한다.
- `DEFAULT 0`: `INSERT`에서 `SALARY` 값을 생략하면 기본값 `0`을 적용한다.
- `CREATE TABLE ... AS SELECT ...`로 **조회 결과를 이용해 새 테이블과 데이터를 함께 생성**할 수도 있다. (7장 참고)

<br>

## 3. DROP TABLE: 테이블 삭제

`DROP TABLE`은 **테이블의 구조와 모든 행을 삭제**하는 명령어다.

- **모든 데이터와 테이블 정의를 제거**하며, 테이블에 속한 인덱스·트리거 등 종속 객체도 함께 제거된다.
- **암묵적 `COMMIT`이 발생**하므로 일반적인 `ROLLBACK`으로 취소할 수 없다.
- **`CASCADE CONSTRAINTS`**는 삭제할 테이블의 PK·UNIQUE 키를 참조하는 **다른 테이블의 외래 키 제약조건**까지 제거한다. (자식 테이블과 그 데이터가 삭제되는 것은 아니다.)

```sql
-- 기본 문법
DROP TABLE 테이블명;
DROP TABLE 테이블명 CASCADE CONSTRAINTS;
```

<br>

## 4. ALTER TABLE: 테이블 구조와 제약조건 변경

`ALTER TABLE`은 **기존 테이블을 유지하면서 컬럼이나 제약조건을 변경**한다.

### 컬럼 추가·수정

```sql
-- ADD: 새로운 컬럼 추가
ALTER TABLE DDL_EMP_TEST
ADD (EMAIL VARCHAR2(100));

-- MODIFY: 컬럼 크기·기본값 변경
ALTER TABLE DDL_EMP_TEST
MODIFY (NAME VARCHAR2(50 CHAR) DEFAULT '미입력');
```

- **`ADD`**: 컬럼을 추가하며 `DEFAULT`·제약조건도 지정할 수 있다.
- **`MODIFY`**: 자료형·크기·기본값·`NOT NULL` 여부 등을 변경한다.
- **`DEFAULT` 변경**은 이후 입력에 적용되며, 기존 값을 일괄 수정하지 않는다.
- **변경 제한**: 기존 데이터에 따라 자료형 변경이나 크기 축소가 제한될 수 있고, `NULL`이 있는 컬럼에는 바로 `NOT NULL`을 지정할 수 없다.

### 제약조건 추가·삭제·활성화·비활성화

```sql
-- 제약조건 추가
ALTER TABLE DDL_EMP_TEST
ADD CONSTRAINT DDL_EMP_SAL_CK CHECK (SALARY >= 0);

-- 검사 비활성화 / 활성화
ALTER TABLE DDL_EMP_TEST DISABLE CONSTRAINT DDL_EMP_SAL_CK;
ALTER TABLE DDL_EMP_TEST ENABLE CONSTRAINT DDL_EMP_SAL_CK;

-- 제약조건 삭제
ALTER TABLE DDL_EMP_TEST DROP CONSTRAINT DDL_EMP_SAL_CK;
```

| 명령                 | 의미               |
| -------------------- | ------------------ |
| `ADD CONSTRAINT`     | 제약조건 추가      |
| `DROP CONSTRAINT`    | 제약조건 삭제      |
| `DISABLE CONSTRAINT` | 제약조건 검사 중지 |
| `ENABLE CONSTRAINT`  | 제약조건 검사 재개 |

`ENABLE` 시 **기존 데이터가 제약조건을 위반하면 활성화에 실패**할 수 있다. PK·UNIQUE 제약조건을 제거·비활성화할 때는 참조 중인 외래 키에 영향을 줄 수 있으므로, 필요한 경우 `CASCADE` 옵션을 사용한다.

<br>

## 5. RENAME, TRUNCATE, COMMENT

- **`RENAME`**: 테이블 등 객체의 이름을 변경한다.
- **`TRUNCATE TABLE`**: **테이블의 모든 행을 삭제하지만, 컬럼·제약조건 등 테이블 구조는 유지한다.** 데이터 삭제와 함께 저장 공간을 정리할 수 있다.
- **`COMMENT`**: 테이블·컬럼의 설명을 데이터 사전에 등록한다.

```sql
-- 테이블 이름 변경
RENAME DDL_EMP_TEST TO DDL_STAFF_TEST;

-- 테이블과 컬럼 설명 등록
COMMENT ON TABLE DDL_STAFF_TEST IS 'DDL 실습용 테이블';
COMMENT ON COLUMN DDL_STAFF_TEST.SALARY IS '급여';

-- 데이터 삽입 후 전체 행 삭제
INSERT INTO DDL_STAFF_TEST (ID, NAME, SALARY)
VALUES (1, '홍길동', 1000);
TRUNCATE TABLE DDL_STAFF_TEST;

SELECT COUNT(*) FROM DDL_STAFF_TEST;
-- 결과: 0 (테이블 구조는 유지)

-- 테이블 자체 삭제
DROP TABLE DDL_STAFF_TEST;
```

### DELETE · TRUNCATE · DROP 비교

| 구분       | `DELETE`                      | `TRUNCATE`                 | `DROP TABLE`                                   |
| ---------- | ----------------------------- | -------------------------- | ---------------------------------------------- |
| 분류       | DML                           | DDL                        | DDL                                            |
| 삭제 대상  | 조건에 맞는 행 (`WHERE` 가능) | 모든 행 (구조 유지)        | 테이블 구조와 모든 데이터                      |
| 저장 공간  | 할당 공간은 일반적으로 유지   | HWM 초기화, 공간 반환 가능 | 테이블 객체 제거(휴지통 설정에 따라 보관 가능) |
| `ROLLBACK` | `COMMIT` 전 가능              | 불가                       | 불가                                           |

`DELETE`는 **조건에 맞는 행을 삭제하는 DML**이며, `COMMIT` 전에는 `ROLLBACK`할 수 있다. 반면 `TRUNCATE`는 **모든 행을 삭제하고 저장 공간도 정리하는 DDL**이므로 암묵적 `COMMIT`이 발생하며, `ROLLBACK`할 수 없다.

<br>

## 참고 자료

- 『SQL의 정석』 11장 — 데이터 정의어(DDL)
