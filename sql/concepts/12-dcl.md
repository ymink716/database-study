# SQL 기초: 데이터 제어어(DCL)

## 목차

1. [DCL과 데이터베이스 보안](#1-dcl과-데이터베이스-보안)
2. [시스템 권한(System Privilege)](#2-시스템-권한system-privilege)
3. [객체 권한(Object Privilege)](#3-객체-권한object-privilege)
4. [역할(Role)의 생성과 부여](#4-역할role의-생성과-부여)
5. [권한과 역할의 데이터 사전](#5-권한과-역할의-데이터-사전)

<br>

## 1. DCL과 데이터베이스 보안

**DCL(Data Control Language, 데이터 제어어)**은 데이터베이스의 **접근 권한을 부여하거나 회수**하여 보안을 관리하는 SQL 명령어다. 관리자는 사용자를 생성하고 필요한 권한이나 역할을 부여해, 각 사용자가 허용된 작업만 수행하도록 관리한다.

- **`GRANT`**: 사용자나 역할에 권한을 부여한다.
- **`REVOKE`**: 부여한 권한이나 역할을 회수한다.
- **권한(Privilege)**: 특정 SQL 작업을 수행하거나 객체에 접근할 수 있는 권리다.
- **역할(Role)**: 여러 권한을 묶어서 관리하는 단위다.

`CREATE USER`, `CREATE ROLE`은 객체를 생성하는 **DDL**이고, 권한을 부여·회수하는 `GRANT`, `REVOKE`가 **DCL**이다. Oracle에서

DCL은 암묵적 `COMMIT`을 발생시키므로 일반적인 `ROLLBACK`으로 취소할 수 없다.

<br>

## 2. 시스템 권한(System Privilege)

**시스템 권한**은 데이터베이스에서 **특정 작업을 수행할 수 있는 권한**이다. 일반적으로 DBA 등 권한을 가진 관리자가 사용자나 역할에 부여한다.

| 시스템 권한      | 의미                        |
| ---------------- | --------------------------- |
| `CREATE SESSION` | 데이터베이스에 접속         |
| `CREATE TABLE`   | 자신의 스키마에 테이블 생성 |
| `CREATE USER`    | 사용자 계정 생성            |

```sql
-- 관리자 또는 해당 권한의 부여 권한을 가진 계정에서 실행
GRANT CREATE SESSION, CREATE TABLE TO USER_B;
REVOKE CREATE TABLE FROM USER_B;
```

`WITH ADMIN OPTION`을 지정하면 **받은 시스템 권한을 다른 사용자나 역할에도 부여·회수**할 수 있다. 예를 들어 `GRANT CREATE SESSION TO USER_B WITH ADMIN OPTION;`과 같이 사용한다.

<br>

## 3. 객체 권한(Object Privilege)

**객체 권한**은 **특정 테이블·뷰·시퀀스 등 데이터베이스 객체에 접근하거나 작업할 수 있는 권한**이다. 일반적으로 객체 소유자 또는 적절한 부여 권한을 가진 사용자가 부여한다.

| 객체 권한                        | 대표적인 작업                                |
| -------------------------------- | -------------------------------------------- |
| `SELECT`                         | 테이블·뷰 조회, 시퀀스 사용                  |
| `INSERT` / `UPDATE` / `DELETE`   | 테이블·뷰의 데이터 추가·수정·삭제            |
| `ALTER` / `INDEX` / `REFERENCES` | 테이블 구조 변경·인덱스 생성·외래 키 참조 등 |
| `EXECUTE`                        | 프로시저·함수 실행                           |

객체 종류에 따라 사용할 수 있는 권한은 다르다. 예를 들어 `S_EMP` 테이블의 소유자는 다음과 같이 다른 사용자에게 권한을 줄 수 있다.

```sql
-- S_EMP 소유자(STUDENT)에서 실행하는 예시
GRANT SELECT ON S_EMP TO USER_B;
GRANT UPDATE (SALARY) ON S_EMP TO USER_B;
REVOKE SELECT ON S_EMP FROM USER_B;
```

- `ON S_EMP`는 **권한을 적용할 객체**를 지정한다.
- `UPDATE (SALARY)`처럼 일부 권한은 **특정 컬럼으로 제한**할 수 있다.
- `WITH GRANT OPTION`을 지정하면 **받은 객체 권한을 다른 사용자에게도 부여**할 수 있다.
- `TO PUBLIC`은 **모든 사용자에게 권한을 부여**하므로 신중하게 사용해야 한다.

**두 옵션의 차이:** `WITH ADMIN OPTION`은 **시스템 권한이나 역할을 다른 사용자에게도 부여할 수 있도록 허용**하고, `WITH GRANT OPTION`은 **특정 객체에 대한 권한을 다른 사용자에게도 부여할 수 있도록 허용**한다. 단, `WITH GRANT OPTION`은 역할(Role)에 객체 권한을 부여할 때는 사용할 수 없다.

<br>

## 4. 역할(Role)의 생성과 부여

**역할(Role)**은 여러 권한을 묶어 **사용자별 권한 부여와 회수를 간단하게 관리**하도록 만든 것이다. 한 사용자에게 여러 역할을 줄 수 있고, 하나의 역할을 여러 사용자나 다른 역할에 부여할 수도 있다.

**역할 사용 순서:** ① 역할 생성 → ② 역할에 권한 부여 → ③ 사용자에게 역할 부여

```sql
-- ① 역할 생성: CREATE ROLE 권한 필요 (DDL)
CREATE ROLE EMP_READER;

-- ② 역할에 객체 권한 부여: S_EMP 소유자가 실행
GRANT SELECT ON S_EMP TO EMP_READER;

-- ③ 사용자에게 역할 부여: 역할 부여 권한 필요
GRANT EMP_READER TO USER_B;

-- 사용자에게 부여했던 역할 회수
REVOKE EMP_READER FROM USER_B;
```

예를 들어 여러 직원에게 조회 권한을 각각 부여하는 대신 **`EMP_READER` 역할 하나를 부여**하면 관리가 편리하다. 역할에는 시스템 권한도 포함할 수 있다.

<br>

## 5. 권한과 역할의 데이터 사전

권한과 역할의 부여 현황은 **데이터 사전 뷰**에서 확인할 수 있다.

| 데이터 사전 뷰        | 조회 내용                                     |
| --------------------- | --------------------------------------------- |
| `ROLE_SYS_PRIVS`      | 역할에 부여된 시스템 권한                     |
| `ROLE_TAB_PRIVS`      | 역할에 부여된 객체 권한                       |
| `USER_ROLE_PRIVS`     | 현재 사용자에게 부여된 역할                   |
| `USER_TAB_PRIVS_MADE` | 현재 사용자 소유 객체에 부여된 객체 권한      |
| `USER_TAB_PRIVS_RECD` | 현재 사용자가 받은 객체 권한                  |
| `USER_COL_PRIVS_MADE` | 현재 사용자 소유 객체의 컬럼별 권한 부여 내역 |
| `USER_COL_PRIVS_RECD` | 현재 사용자가 받은 컬럼별 객체 권한           |

```sql
-- 내게 부여된 역할 확인
SELECT GRANTED_ROLE, ADMIN_OPTION
FROM USER_ROLE_PRIVS;

-- 내 테이블 S_EMP에 부여된 객체 권한 확인
SELECT GRANTEE, TABLE_NAME, PRIVILEGE
FROM USER_TAB_PRIVS_MADE
WHERE TABLE_NAME = 'S_EMP';
```

<br>

## 참고 자료

- 『SQL의 정석』 12장 — 데이터 제어어(DCL)
