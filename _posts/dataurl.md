---
title: 
author: Patrick
date: 2024-06-09 17:32:00 +0900
tags: []
categories: []
render_with_liquid: false
---

# SQL Query vs ORM

## ORM(Object Relational Mapping)
- DB내의 테이블들을 객체화
- 각 DBMS에 대해서 CRUD 등을 공통된 접근 기법으로 사용

## SQLAlchemy
- Python에서 ORM 라이브러리 중 가장 많이 사용하는 package
- DB와 연결 시 사용
- SQLAlchemy를 사용하면 sql 쿼리를 작성하지 않아도 프로그램이 언어로 객체간의 관계를 표현

## DataURL
```docker
dialect+driver://username:password@host:port/database
```

|구성 요소|	의미	|예시 값|
|--------|--------|-------|
|dialect	|사용할 데이터베이스 종류	|mysql, postgresql, sqlite, oracle 등|
|driver	|Python에서 DB와 통신할 드라이버	|pymysql, psycopg2, asyncpg 등|
|username	|데이터베이스 사용자 계정	|root, admin 등|
|password	|사용자 비밀번호	|mypass123|
|host	|DB 서버 주소 (도커라면 서비스명)	|localhost, db, db_stream 등|
|port	|DB 접속 포트 번호	|MySQL/MariaDB: 3306, PostgreSQL: 5432|
|database	|접속할 데이터베이스 이름	|mydb, project_db 등|
