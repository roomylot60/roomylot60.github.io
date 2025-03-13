---
title: MySQL Basic
author: Patrick
date: 2023-09-12 16:22:00 +0900
tags: [Grammer]
categories: [MySQL]
render_with_liquid: false
---
# MySQL
## Relational Database(관계형 데이터베이스)
### Table(표)의 형태로 표현
- row(행) : 각 특성(Attribute)에 대한 Value
- column(열) : Attribute(특성은 각각 다른 type을 갖음)
- schema : 서로 연관이 있는 Table들을 grouping 해놓은 집합
- Table에 있는 행과 열의 관계를 통해 새로운 Table의 작성 가능

**SQL(Structured Query Language)**
- 표의 형태로 구조화(Structured)하기 위한 요청(Query)을 표현하는 언어(Language)

## Docker image를 통해 mysql 접속
### Mysql 설치
- [Docker Hub of mysql](https://hub.docker.com/_/mysql) : 해당 사이트를 통해 설치 및 상세 내용 확인 가능
- `docker pull mysql:latest` : 최신 버전 pull

### Mysql 실행
- `docker run --name mysql_container -e MYSQL_PASSWORD=my_password -d -p hst_p_num:ctn_p_num mysql:ver_num` : 
- `docker exec -it mysql_container bash` : container 상의 mysql의 bash를 실행
- `SET PASSWORD = PASSWORD('new_password');` : 사용하는 user의 password 재설정(ERROR 1820(HY000) 해결법)

## 명령어 정리
- [Mysql Cheatsheet](https://www.mysqltutorial.org/mysql-cheat-sheet.aspx)
### Schema 명령어
- `CREATE DATABASE schema_name;` : 생성
- `DROP DATABASE schema_name;` : 삭제
- `SHOW {DATABASES | SCHEMAS};` : 생성된 목록 확인
- `USE schema_name;` : 사용할 schema 선택
- `exit` : 종료

### TABLE 명령어
- `CREATE TABLE table_name(column_list);` : 생성
**column_list**
- `column_name data_type(length) {NOT NULL | NULL} ...`
    - `column_name` : 생성하고자 하는 column명
    - `data_type(length)` : 해당 데이터의 종류 및 노출시키고자 하는 길이
        **String**
        - VARCHAR
        - CHAR
        - TEXT
        **Number**
        - INT
        - LONGINT
        - DECIMAL
        **date**
        - DATETIME
    - `NOT NULL | NULL` 값의 유무 허용여부 표기
    - `AUTO_INCREMENT` : 값을 자동으로 하나씩 증가하도록 설정(식별자로 사용할 때 주로 사용)
- `PRIMARY KEY(column_name)` : 식별자로 사용하고자 하는 column 설정 = 중복 제거
- `SHOW TABLES;` : 생성된 목록 확인
- `DESC table_name` : 생성된 table의 구성요소 묘사(describe)
