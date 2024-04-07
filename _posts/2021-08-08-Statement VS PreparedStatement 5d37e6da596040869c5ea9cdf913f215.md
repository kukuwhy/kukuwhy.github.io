---
layout: post
title:  "Statement VS PreparedStatement"
tags: SQL Injection
---

## Statement

```java
String sqlstr = "SELECT name, memo FROM TABLE WHERE name =" + num
Statement stmt = conn.createStatement();
ResultSet rst = stmt.executeQuery(sqlstr);
```

- 쿼리문을 수행할 떄마다 SQL 실행단계 1~3단계를 거친다.
- SQL 문을 수행하는 과정에서 매번 컴파일을 하기 때문에 성능상의 이슈가 발생한다.
- 실행되는 SQL문을 확인할 수 있다.

SQL 실행 단계는 아래와 같습니다.

1. 쿼리 문장 분석
2. 컴파일
3. 실행

## Prepared Statement

```java
String sqlstr = "SELECT name, memo FROM TABLE WHERE num = ?"
PreparedStatement stmt = conn.preparedStatement();
stmt.setInt(1, num);
ResultSet rst = stmt.executeQuery(sqlstr);
```

- 컴파일이 미리 되어 있어서 Statement에 비해 좋은 성능
- 특수문자를 자동으로 파싱해주기 때문에 SQL 인젝션을 방어할 수 있다.
- “?” 부분에만 변화를 주어 쿼리문을 수행하므로 실행되는 SQL문을 파악하기 어렵다.

### 왜 사용해요?

1) 사용자 입력값으로 쿼리문 실행 시 특수문자로 들어오는 문자도 알아서 파싱하기 때문에 에러를 막을 수 있고, 공격도 방어가 됩니다.

2) 쿼리 반복 수행 작업 일 경우

## 무슨 차이가 있어요?

`캐시 사용 여부` 에 차이가 있습니다.

Statment를 사용하면 매번 쿼리를 수행할 때마다 단계를 계속 거치지만,

PreparedStatemnt의 경우 처음 한 번만 세 단계를 거친 후 캐시에 담아 재사용합니다.

만약 동일한 쿼리를 반복적으로 수행한다면 DB에 부하가 줄게 되며, 성능에도 좋습니다.