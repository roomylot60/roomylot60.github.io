---
title: MySQL Initial Setting for Password
author: Patrick
date: 2024-09-17 17:33:00+0900
tags: [Error]
categories: [MySQL, Database]
render_with_liquid: false
---
### MySQL 최초 설치 시  발생하는 에러 (Server version: 8.0.41-0ubuntu0.22.04.1 (Ubuntu))
#### [ERROR 1698 (28000)](https://dev.mysql.com/doc/relnotes/workbench/en/news-6-1-7.html)
- MySQL error code 1698 (ER_ACCESS_DENIED_NO_PASSWORD_ERROR) is now handled as an authentication error. (Bug #18711142, Bug #72536)

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

### [Ref.](https://taltal-dev-note.tistory.com/370)
#### Why is MySQL's Authentication Method Plugin auth_socket?
- In recent versions of MySQL, particularly on Debian-based systems like Ubuntu, the default authentication method for the root user is often set to auth_socket for security reasons.
- This is intended to improve security by requiring users to connect using the Unix socket file rather than a password.
- This way, MySQL ensures that the user is authenticated through the operating system's user management.

#### What are auth_socket and mysql_native_password?
- `auth_socket`
    - Mechanism: Uses Unix socket file for authentication.
    - Usage: Authenticates the MySQL user based on the Unix system user running the MySQL client. This means the user must have a corresponding Unix user account.
    - Pros: More secure because it eliminates the need for a password in certain cases.
    - Cons: Less flexible for remote connections or scripts that require password authentication.

- `mysql_native_password`
    - Mechanism: Uses a password for authentication.
    - Usage: Standard method for authenticating MySQL users, requiring a username and password.
    - Pros: More versatile and works well for remote connections and scripts.
    - Cons: Passwords can be a security risk if not managed properly.

#### Other Authentication Methods
- MySQL supports several other authentication plugins. Here are a few:
- `caching_sha2_password`
    - Mechanism: Uses SHA-256 hashing for passwords.
    - Usage: Default authentication plugin from MySQL 8.0.
    - Pros: Stronger security compared to mysql_native_password.
    - Cons: May require additional configuration for older clients.
- `sha256_password`
    - Mechanism: Uses SHA-256 hashing for passwords.
    - Usage: Provides strong security with some of the advanced features of caching_sha2_password.
    - Pros: Strong encryption.
    - Cons: Requires SSL/TLS for connections unless RSA is used for password exchange.
- `pam`
    - Mechanism: Pluggable Authentication Modules (PAM).
    - Usage: Allows integration with various PAM-supported authentication methods, like LDAP, Kerberos, etc.
    - Pros: Flexible and can integrate with existing authentication infrastructures.
    - Cons: Complexity in configuration and dependency on the PAM configuration.
- `dialog`
    - Mechanism: Provides a mechanism for interactive client/server password authentication.
    - Usage: Can be used to implement custom authentication dialogs.
    - Pros: Flexibility in implementing custom authentication schemes.
    - Cons: More complex to implement and manage.
 
#### Summary
- `auth_socket`: Authenticates via Unix socket file, tying MySQL user to the Unix user.
- `mysql_native_password`: Standard username/password authentication.
- Other methods like `caching_sha2_password`, `sha256_password`, `pam`, and `dialog` offer various levels of security and flexibility.