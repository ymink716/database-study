# SQL 기초: 데이터 사전(Data Dictionary)

## 목차

1. [데이터 사전의 개념](#1-데이터-사전의-개념)
2. [데이터 사전 뷰의 종류](#2-데이터-사전-뷰의-종류)
3. [그 밖의 데이터 사전 뷰](#3-그-밖의-데이터-사전-뷰)
4. [데이터 사전 조회 실습](#4-데이터-사전-조회-실습)

<br>

## 1. 데이터 사전의 개념

**데이터 사전(Data Dictionary)**은 데이터베이스의 **구조와 관리 정보(메타데이터)**를 저장하는 시스템 정보다.

Oracle 서버가 생성·관리하며, 기본 데이터 사전 테이블은 `SYS` 스키마가 소유한다.

일반 사용자는 주로 **읽기 전용 데이터 사전 뷰(View)**를 통해 필요한 정보를 조회한다.

여기에는 **사용자와 권한, 테이블·뷰·시퀀스·인덱스 같은 객체, 제약조건, 감사(Auditing) 정보** 등이 포함된다.

<br>

## 2. 데이터 사전 뷰의 종류

접두어를 보면 **누구의 정보를 어느 범위까지 조회하는지** 구분할 수 있다.

| 접두어  | 조회 범위                                             | 대표 예시                          |
| ------- | ----------------------------------------------------- | ---------------------------------- |
| `USER_` | 현재 사용자가 소유한 객체 및 관련 정보                | `USER_OBJECTS`, `USER_CONSTRAINTS` |
| `ALL_`  | 현재 사용자에게 접근 권한이 있는 객체(본인 소유 포함) | `ALL_OBJECTS`                      |
| `DBA_`  | 데이터베이스 전체의 관리 정보(적절한 권한 필요)       | `DBA_OBJECTS`                      |
| `V$`    | 세션·잠금·성능 등 **현재 데이터베이스의 동적 상태**   | `V$SESSION`, `V$LOCK`              |

`USER_`, `ALL_`, `DBA_`는 주로 **정적 데이터 사전 뷰**의 구분이고, `V$`는 **동적 성능 뷰**다. 일반 실습 계정에서는 `USER_`부터 조회하고, `DBA_`나 일부 `V$` 뷰는 권한이 없어 접근할 수 없을 수 있다.

<br>

## 3. 그 밖의 데이터 사전 뷰

| 이름               | 용도                                                              |
| ------------------ | ----------------------------------------------------------------- |
| `DICTIONARY`       | 조회할 수 있는 데이터 사전 뷰와 설명을 찾아보는 목록(일명 `DICT`) |
| `TABLE_PRIVILEGES` | 객체에 부여된 권한 정보                                           |
| `IND`              | `USER_INDEXES`의 동의어(Synonym)로, 인덱스 정보 조회              |

실제 객체 권한을 조회할 때는 Oracle에서 제공하는 **`USER_TAB_PRIVS`**, **`ALL_TAB_PRIVS`**도 알아두면 좋다.

**동의어(Synonym)**는 기존 객체를 다른 이름으로 참조할 수 있게 하는 별칭이다.

<br>

## 4. 데이터 사전 조회 실습

### 필요한 데이터 사전 뷰 찾기

```sql
SELECT TABLE_NAME, COMMENTS
FROM DICTIONARY
WHERE TABLE_NAME LIKE 'USER%'
ORDER BY TABLE_NAME;
```

`DICTIONARY`에서 `USER`로 시작하는 데이터 사전 이름을 검색한다. **일반적인 객체 이름은 대문자로 저장되므로** 문자열 비교에도 `'USER%'`, `'S_EMP'`처럼 대문자를 사용한다.

### 내 테이블 목록 확인

```sql
SELECT OBJECT_NAME, OBJECT_TYPE
FROM USER_OBJECTS
WHERE OBJECT_TYPE = 'TABLE'
ORDER BY OBJECT_NAME;
```

현재 사용자 소유 객체 중 테이블만 조회한다. `USER_OBJECTS`는 테이블 이외에도 뷰·시퀀스·인덱스 등 다양한 객체 정보를 제공한다. **테이블만 확인**하는 용도라면 `USER_TABLES`도 사용할 수 있다.

### 테이블의 제약조건과 적용 컬럼 확인

```sql
-- S_EMP에 정의된 제약조건 확인
SELECT CONSTRAINT_NAME, CONSTRAINT_TYPE,
       SEARCH_CONDITION, R_CONSTRAINT_NAME
FROM USER_CONSTRAINTS
WHERE TABLE_NAME = 'S_EMP';

-- 각 제약조건이 적용된 컬럼 확인
SELECT CONSTRAINT_NAME, COLUMN_NAME, POSITION
FROM USER_CONS_COLUMNS
WHERE TABLE_NAME = 'S_EMP'
ORDER BY CONSTRAINT_NAME, POSITION;
```

- `USER_CONSTRAINTS`: 제약조건의 **이름·종류·검사 조건·참조 제약조건** 등을 확인한다.
- `USER_CONS_COLUMNS`: 제약조건이 **어떤 컬럼**에 적용됐는지 확인한다.
- `CONSTRAINT_TYPE`: `P`(PK), `R`(FK), `U`(UNIQUE), `C`(CHECK·NOT NULL) 등이 대표적이다.

<br>

## 참고 자료

- 『SQL의 정석』 10장 — DICTIONARY
