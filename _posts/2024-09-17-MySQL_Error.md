---
title: MySQL Initial Setting for Password
author: Patrick
date: 2024-09-17 17:33:00+0900
tags: [Error]
categories: [MySQL, Database]
render_with_liquid: false
---
### MySQL 최초 설치 시  발생하는 에러 (Server version: 8.0.41-0ubuntu0.22.04.1 (Ubuntu))
#### ERROR 1698 (28000)

```bash
$ mysql -u root -p
Enter password: 
ERROR 1698 (28000): Access denied for user 'root'@'localhost'
```
- 최초 설치 시, 일반적으로 password의 기본값은 `enter` 일텐데 로그인이 계속해서 실패
- 해결하기 위해 강제적으로 재설정을 시도

```bash
$ sudo mysql -u root
```
- `sudo` 상에서 `mysql` 을 실행하고 재설정

```mysql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'your_password';
mysql> FLUSH PRIVILEGES;
mysql> EXIT;
```

- 재설정 후 로그인 시도

```bash
$ mysql -u root -p
Enter password: 
ERROR 1698 (28000): Access denied for user 'root'@'localhost'
```
- 같은 오류가 발생

#### 해결 방법

- `sudo`를 통해 `mysql`에 재접속 후 다음 내용 확인

```mysql
mysql> SELECT User, Host, plugin FROM mysql.user;
+------------------+-----------+-----------------------+
| User             | Host      | plugin                |
+------------------+-----------+-----------------------+
| debian-sys-maint | localhost | caching_sha2_password |
| mysql.infoschema | localhost | caching_sha2_password |
| mysql.session    | localhost | caching_sha2_password |
| mysql.sys        | localhost | caching_sha2_password |
| root             | localhost | auth_socket           |
+------------------+-----------+-----------------------+
```
- `root`에서의 `plugin` 값이 `auth_socket`으로 되어 있음
- 이를 `mysql_native_password`로 변경해주어야 함

```mysql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'your_password';
Query OK, 0 rows affected (0.02 sec)
```
- 간혹 위 과정에서 `ERROR 1819 (HY000): Your password does not satisfy the current policy requirements` 와 같은 에러 발생
- `mysql> SHOW VARIABLES LIKE 'validate_passwords';` 명령어를 통해 현재 적용중인 policy의 값을 확인하고 이를 변경