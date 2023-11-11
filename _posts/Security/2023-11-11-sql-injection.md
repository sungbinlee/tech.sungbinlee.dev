---
title: "SQL Injection 문제와 방어 전략"
categories:
  - Security
tags:
  - Security
  - Backend
  - Database
  - System Protection
toc: true
toc_sticky: true
toc_label: "SQL Injection 문제와 방어 전략"
toc_icon: "magnifier"
---

## 서론
보안 분야에서 자주 다루는 이슈 중 하나는 SQL Injection입니다. 이것이 정확히 무엇이며, 어떤 이유로 발생할 수 있는지에 대해 알아보겠습니다. 더불어 이러한 공격으로부터 시스템을 어떻게 보호할 수 있는지를 살펴보겠습니다.

## SQL Injection의 개념과 발생 원인
SQL Injection은 데이터베이스의 SQL 명령어를 악의적으로 조작하여 비정상적인 방식으로 데이터베이스 엑세스를 시도하는 공격입니다. 

## 공격 사례와 위험성

### 사례1: 참 값을 반환하는 조작
공격 사례:
```sql
' OR '1' = 1'
```
위와 같이 조작된 password 인자를 쿼리에 주입하여, 해당 부분이 항상 참이 되도록 조작하는 SQL Injection 공격입니다.
- 일반적으로 로그인 폼에서 사용자가 입력한 아이디와 비밀번호를 검증하기 위한 SQL 쿼리에서 발생합니다.
- 사용자가 입력한 비밀번호에 위와 같이 ' OR '1' = 1'을 포함시키면, 쿼리는 다음과 같이 해석됩니다.

```sql
SELECT * FROM users WHERE username = '입력한_아이디' AND password = '' OR '1' = '1';
```
- 이렇게 되면 항상 참이 되는 조건이 도출되어, 권한이 없는 사용자라도 로그인이 성공적으로 이루어집니다.

위험성:
- 권한이 없는 사용자도 로그인할 수 있게 되므로, 민감한 정보에 접근할 수 있는 보안 취약점이 발생합니다.
- 데이터베이스의 중요한 정보가 노출될 수 있으며, 해당 정보를 이용한 다양한 공격이 가능해집니다.

### 사례2: 쿼리문 주석 처리를 이용한 공격
공격 사례:
```sql
'; -- 주석
DELETE FROM users WHERE username = 'admin'
```
주석 처리를 이용하여 기존 쿼리를 무력화하고 새로운 쿼리를 주입하는 공격입니다. 위 예시에서는 'admin' 사용자의 정보를 삭제하는 쿼리가 주입됩니다.

위험성:
- 무단 데이터 삭제나 수정이 가능해지며, 시스템의 무결성이 손상됩니다.
- 공격자에 의해 데이터 손실이 발생할 수 있습니다.

### 사례2-1: DROP 테이블을 이용한 치명적인 공격
공격 사례:
```sql
'; DROP TABLE users; --
```
DROP 테이블 쿼리를 주입하여 데이터베이스의 특정 테이블을 삭제하는 공격입니다.

위험성:

- 데이터베이스의 특정 테이블이 삭제되면 해당 테이블에 저장된 모든 데이터가 소실됩니다.
- 시스템의 가용성을 침해하여 서비스 중단을 초래할 수 있습니다.

## SQL Injection 방어 전략
### 1. ORM 사용
- 현대의 백엔드에서는 ORM을 활용하여 직접적인 SQL 명령어 실행을 피하고, 허용되지 않은 값을 사용할 경우 ORM에서 차단하는 방식을 택합니다.
- 예시:
```python
# Django ORM 사용 예시
User.objects.filter(username=username, password=password).first()
```

### 2. Procedure 활용

- 프로시저를 통해 SQL 명령어를 묶어서 실행함으로써, 입력 값에 대한 타입 검증 및 SQL Injection에 대한 일정 수준의 방어가 가능합니다.
- 예시:
```sql
-- 프로시저 정의
CREATE PROCEDURE InsertUser(IN username VARCHAR(255), IN password VARCHAR(255))
BEGIN
  -- 타입 검증 및 데이터베이스 등록 로직
END;
```
### 3. Parameterized Query 사용

- 쿼리에 파라미터를 사용하여 사용자 입력을 안전하게 처리하고 SQL Injection을 방지합니다.
- 예시:
```python
# 파이썬 SQLite 라이브러리를 사용한 Parameterized Query 예시
cursor.execute("SELECT * FROM users WHERE username = ? AND password = ?", (username, password))
```
### 4. SQL Injection 코드 패턴 필터링

- 자주 사용되는 SQL Injection 코드 패턴을 미리 걸러내는 방법을 통해 시스템을 더욱 안전하게 운영할 수 있습니다.

## 결론
SQL Injection은 보안에 심각한 위협을 가하는 공격 중 하나입니다. 현대의 백엔드 개발에서는 ORM 기술을 통해 상당 부분 방어되고 있지만, 여전히 다양한 프로젝트와 환경에서 일할 때는 SQL Injection에 대한 경계가 필요합니다. 이런 상황에서는 훌륭한 개발자라면 다양한 방어책을 알고 적절히 활용하는 것이 중요합니다.
