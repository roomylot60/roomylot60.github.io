---
title: MySQL Basic
author: Patrick
date: 2023-09-27 17:42:00+0900
tags: [Grammer]
categories: [Docker]
render_with_liquid: false
---
## Docker 명령어

```bash
docker run --platform linux/amd64
-p 3306:3306
--name [컨테이너 이름]
-e MYSQL_ROOT_PASSWORD=[루트 유저 비밀번호]
-e MYSQL_DATABASE=[데이터베이스 이름]
-e MYSQL_USER=[유저 이름]
-e MYSQL_PASSWORD=[비밀번호]
-d mysql
```
### -f 옵션
일반적으로 `docker-compose.yml` 파일이 현 디렉토리 내에 있을 경우에 `docker-compose up` 명령어를 통해 실행할 수 있다.
다른 파일명이나 다른 경로의 파일을 통해 실행하고자 할 경우에 `-f`옵션을 사용한다.
```bash
$ docker-compose -f docker-compose-local.yml up
```

- `docker-compose-local.yml` 파일을 `docker-compose up`

```bash
$ docker-compose -f docker-compose.yml -f docker-compose-test.yml up
```

- `-f` 옵션을 통해 복수의 설정파일의 실행또한 가능하다. 이때, `docker-compose.yml` 파일 실행 후 `docker-compose-test.yml` 을 실행한다.

### -d 옵션
- `docker-compose up` 을 통해 설정파일을 실행할 때, `Crtl + c` 를 통해 모든 서비스를 종료할 수 있는데, 이를 막기 위해 설정파일을 백그라운드로 실행하기 위한 옵션이다.

```bash
$ docker-compose up -d
```

## docker-compose.yml 파일 내에서의 명령어

```bash
version: '3'
services:
  db:
    image: mysql:[mysql버전]
    container_name: [컨테이너 이름]
    restart: always
    port:
      - [로컬 포트]:3306
    environment:
      MYSQL_ROOT_PASSWORD: [루트 유저 비밀번호]
      TZ: Asia/Seoul
    volumes:
      - ./db/mysql/data:/var/lib/mysql
      - ./db/mysql/init:/docker-entrypoint-initdb.d
    platform: linux/x86_64
```

## ubuntu django 설치 및 가상 환경 실행 & mysql 연결
1. 컨테이너 접속 및 shell 생성
- `docker exec -it [ubuntu container name] bin/sh` 를 사용해 `docker shell`을 생성하고 container 에 접근

```bash
docker exec -it django bin/sh
```

- `sh`는 `Bourne shell`로 `apt` 실행은 가능하나 내부에서 `source` 명령어 실행이 안되어 중간에 변경

```bash
!/bin/bash
```

- `bash`는 `Brourne Again Shell`의 약자로 기능이 몇가지 추가된 형태

2. 모듈 설치
- apt update & python 설치

```bash
apt update && apt install -y python
```

- 가상환경 설치

```bash
apt install -y python3-venv
```

- vim 설치

```bash
apt install -y vim
```

3. Django 디렉토리 생성 & 가상 환경 활성화
- 디렉토리 생성 및 이동

```bash
mkdir django_app
cd django_app
```

- 가상환경 생성 및 활성화

```bash
python3 -m venv venv
source venv/bin/activate
```

- 가상 환경 내 django 모듈 설치

```bash
pip install django
```

- mysqlclient 설치

```bash
apt-get update
apt-get install python3-venv -y
apt-get install libmysqlclient-dev -y
apt-get install mysql-client-core-8.0 -y
apt-get install gcc -y
apt-get install python3.10-dev -y
pip install mysqlclient
```

- 원래 위 코드를 활용하여 `mysqlclient`를 다운 받을 수 있었으나 다음의 에러가 발생

#### mysqlclient download error

```bash
(venv) root@906860cb7779:/django_app# pip install mysqlclient                  
Collecting mysqlclient
  Using cached mysqlclient-2.2.0.tar.gz (89 kB)
  Installing build dependencies ... done
  Getting requirements to build wheel ... error
  error: subprocess-exited-with-error

  × Getting requirements to build wheel did not run successfully.
  │ exit code: 1
  ╰─> [24 lines of output]
      /bin/sh: 1: pkg-config: not found
      /bin/sh: 1: pkg-config: not found
      Trying pkg-config --exists mysqlclient
      Command 'pkg-config --exists mysqlclient' returned non-zero exit status 127.
      Trying pkg-config --exists mariadb
      Command 'pkg-config --exists mariadb' returned non-zero exit status 127. 
      Traceback (most recent call last):
        File "/django_app/venv/lib/python3.10/site-packages/pip/_vendor/pep517/in_process/_in_process.py", line 363, in <module>
          main()
        File "/django_app/venv/lib/python3.10/site-packages/pip/_vendor/pep517/in_process/_in_process.py", line 345, in main
          json_out['return_val'] = hook(**hook_input['kwargs'])
        File "/django_app/venv/lib/python3.10/site-packages/pip/_vendor/pep517/in_process/_in_process.py", line 130, in get_requires_for_build_wheel
          return hook(config_settings)
        File "/tmp/pip-build-env-ftqygab2/overlay/lib/python3.10/site-packages/setuptools/build_meta.py", line 355, in get_requires_for_build_wheel
          return self._get_build_requires(config_settings, requirements=['wheel'])
        File "/tmp/pip-build-env-ftqygab2/overlay/lib/python3.10/site-packages/setuptools/build_meta.py", line 325, in _get_build_requires
          self.run_setup()
        File "/tmp/pip-build-env-ftqygab2/overlay/lib/python3.10/site-packages/setuptools/build_meta.py", line 341, in run_setup
          exec(code, locals())
        File "<string>", line 154, in <module>
        File "<string>", line 48, in get_config_posix
        File "<string>", line 27, in find_package_name
      Exception: Can not find valid pkg-config name.
      Specify MYSQLCLIENT_CFLAGS and MYSQLCLIENT_LDFLAGS env vars manually     
      [end of output]

  note: This error originates from a subprocess, and is likely not a problem with pip.
error: subprocess-exited-with-error

× Getting requirements to build wheel did not run successfully.
│ exit code: 1
╰─> See above for output.

note: This error originates from a subprocess, and is likely not a problem with pip.
```

- 2023-06-22 릴리스로 인해 `mysqlclient` 설치에 `pkg-config`가 필요

```bash
apt install default-libmysqlclient-dev pkg-config -y
```

- 해당 코드를 사용하여 `pkg-config`를 다운로드 받고 나서 `mysqlclient`를 다운 받을 수 있음

4. Django 세팅
- 프로젝트 생성

```bash
django-admin startproject [pj_name] [생성 위치]
cd [생성된 위치]
```

- my_settings.py 생성 : 프로젝트 최상위 폴더에 생성

```bash
touch my_settings.py
```
#### **파일 내용**

```bash
DATABASES = {
    'default' : {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'DATABASE 명',
        'USER': 'DB접속 계정명',
        'PASSWORD': 'DB접속용 비밀번호',
        'HOST': 'MySQL 컨테이너 이름',
        'PORT': 'MySQL 컨테이너 내부 포트번호',
    }
}

SECRET_KEY = '시크릿키' 
# settings.py에 있는 secret_key를 사용
```
- settings.py 설정

```bash
from my_settings.py import DATABASES, SECRET_KEY # my_settings.py에서 값을 불러옴
...
SECRET_KEY = SECRET_KEY # 해당 값들을 입력
DATABASES = DATABASES
...
ALLOWED_HOSTS = ['*'] # 모든 IP의 접속을 허용
```

### Linux command
- `pwd` : 현재 경로 확인

