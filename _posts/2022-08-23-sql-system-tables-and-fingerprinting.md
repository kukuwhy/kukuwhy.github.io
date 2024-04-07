---
layout: post
title:  "SQL System tables & fingerprinting"
tags: SQL Injection
---

## MySQL

`데이터 베이스 종류`

- information_schema
- myslq
- performance_schema
- sys
- mysql

`테이블 정보`, `스키마 정보`

information_schema.tables

```sql
#스키마 정보
select TABLE_SCHEMA from information_schema.tables group by TABLE_SCHEMA

#테이블 정보
select TABLE_SCHEMA, TABLE_NAME from informcation_schema.Tables;
```

`컬럼 정보` 

information_schema.columns

```sql
#컬럼 정보
select TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME from information_schema.COLUMNS;
```

`실시간 실행 쿼리 정보`

information_schema.PROCESSLIST

```sql
#실시간 실행 쿼리 조회
select info from information_schema.PROCESSLIST;

#MySQL 계정 및 실시간 실행 쿼리 조회
select user, current_statement from sys.sessions
```

`DBMS 계정 정보`

information_schema.USER_PRIVILEGES, myql.user

```sql
#MySQL DBMS 권한 및 계정 정보
select GRANTEE, PRIVILEGE_TYPE, IS_GRANTABLE from information_schema.USER_PRIVILEGES;

#MySQL DBMS 계정 정보
select User, authentication_string from mysql.user
```

`버전`

@@verison

version()

SQL 쿼리 에러가 발생하는 경우 에러 검색으로 DB 종류를 알 수 있다.

에러메시지 - `ERROR 1222 (21000): The used SELECT statements have a different number of columns`

## MSSQL

초기 `데이터베이스 종류`

- master
- tempdb
- model
- msdb
- userTable(임의로 만든 데이터베이스)

`. .. 차이`

기본적으로 .은 선 객체 내에 속한 후 객체 접근을 의미합니다.

예를 들어 스키마.테이블은 해당 스키마에 속한 테이블을 나열합니다.

..의 경우에는 MSSQL은 데이터베이스.스키마명.테이블명으로 쿼리를 구성할 수 있는데

해당 유저의 기본 스키마를 사용한다면 중간의 스키마명을 빼고, 데이터베이스명..테이블명으로 작성할 수 있습니다.

`데이터베이스 정보`

master..sysdatabase, DB_NAME();

```sql
select name from master..sysdatabases;

select DB_NAME(1);

```

`테이블 정보`

sysobjects(xtype), 데이터베이스.information_schema.tables;

```sql
#xtype='U'는 사용자 정의 테이블을 의미
select name from userTable..sysobjects where xtype='U';
#                      (DB)       (Schema)          (table)
select table_name from userTable.information_schema.tables;
```

`컬럼 정보`

syscolumns, 데이터베이스.information_schema.columns;

```sql
SELECT name FROM syscolumns WHERE id = (SELECT id FROM sysobjects WHERE name = 'users');
SELECT table_name, column_name FROM userTable.information_schema.columns;
```

`DBMS 계정 정보`

```sql
select name, password_hash from master.sys.sql_logins;
select * from master..syslogins;
```

`버전`

@@version

에러 메시지 - 

Msg 205, Level 16, State 1, Server e2cb36ec2593, Line 1
All queries combined using a UNION, INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.asdf

## PostgreSQL

- postgres
- template1
- template0

`스키마(카탈로그) 정보`

pg_catalog, information_schema

```sql
select nspname from pg_catalog.pg_namespace;
```

`테이블 정보`

```sql
select table_name from information_schema.tables where table_schema='pg_catalog';
select table_name from information_schema.tables where table_schema='information_schema';
select table_schema, table_name from information_schema.tables;
```

`컬럼 정보`

```sql
select table_schema, table_name, column_name from information_schema.columns;
```

`DBMS 계정정보`

```sql
select usename, passwd from pg_catalog.pg_shadow;
```

`DBMS 설정정보`

```sql
select name, setting from pg_catalog.pg_settings;
```

`실시간 실행 쿼리 확인`

```sql
select usename, query from pg_catalog.pg_stat_activity;
```

`버전`

version()

에러 메시지 - ERROR:  each UNION query must have the same number of columns

## Oracle

`데이터베이스 정보`

```sql
select DISTINCT owner FROM all_tables
```

`테이블 정보`

```sql
select owner, table_name FROM all_tables
select TABLE_NAME FROM tabs;
```

`컬럼 정보`

```sql
SELECT columna_name FROM all_tab_columns where table_name='users';
SELECT columna_name FROM COLS where table_name='users';
SELECT CNAME from COL WHERE TNAME='users';
```

`DBMS 계정 정보`

```sql
select * from all_users
```

`버전`

```sql
SELECT banner FROM v$version WHERE banner LIKE 'Oracle%';
SELECT banner FROM v$version WHERE banner LIKE 'TNS%';
SELECT version FROM v$instance;
```

 

## SQLite

`시스템 테이블`

```sql
select * from sqlite_master;
```

`버전`

sqlite_version()

에러 메시지 - `Error: SELECTs to the left and right of UNION do not have the same number of result columns`