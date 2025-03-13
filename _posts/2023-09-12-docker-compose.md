---
title: Docker-Compose Basic
author: Patrick
date: 2023-09-12 18:30:00 +0900
tags: [Grammer]
categories: [Docker]
render_with_liquid: false
---
# Docker-compose
## Def. Docker Compose
- Docker Compose는 다중 컨테이너 애플리케이션을 정의 공유할 수 있도록 개발된 도구로 단일 명령을 사용하여 모두 실행 또는 종료할 수 있도록 개발된 도구

## Command Line
### 1. Network
- `docker network create wordpress_net` : "wordpress_net"(Network) 생성
### 2. Container
- `docker run --name "db" -v "$(pwd)/db_data:/var/lib/mysql" -e "MYSQL_ROOT_PASSWORD=123456" -e "MYSQL_DATABASE=wordpress" -e "MYSQL_USER=wordpress_user" -e "MYSQL_PASSWORD=123456" --network wordpress_net mysql:5.7` : "db"(MySQL Container) 생성 및 "wordpress_net" 연결
    - `docker run {details} mysql:5.7` : mysql 5.7 버전의 container 생성
    - `--name "db"` : Container 명을 "db"로 설정
    - `-v "$(pwd)/db_data:/var/lib/mysql"` : Host의 "db_data" 디렉토리와 Container 내부의 "/var/lib/mysql/"을 연결하는 Volume 생성
    - `-e "env_details"` : 각각의 환경 변수(environmental variable) 설정
    - `--network wordpress_net` : "wordpress_net"이라는 이름을 가진 Network와 연결
- `docker run --name app -v "$(pwd)/app_data:/var/www/html" -e "WORDPRESS_DB_HOST=db" -e "WORDPRESS_DB_USER=wordpress_user" -e "WORDPRESS_DB_NAME=wordpress" -e "WORDPRESS_DB_PASSWORD=123456" -e "WORDPRESS_DEBUG=1" -p 8080:80 --network wordpress_net wordpress:latest` : "app"(Wordpress Container) 생성 및 "wordpress_net" 연결
    - `-p 8080:80` : Host의 8080 포트와 Container의 80 포트를 연결
    - `-e "WORDPRESS_DB_HOST=db"` : "db"와 연결하기 위한 환경 변수 설정

## yml 파일
- `docker-compose up` : `docker-compose.yml`을 통해 자동으로 Network와 내부 Container들을 생성
- `docker-compose down` : Network 종료

<br>

**docker-compose.yml**
```
version: "3.7"

services:
  db:
    image: mysql:5.7
    volumes:
      - ./db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress_user
      MYSQL_PASSWORD: 123456
  
  app:
    depends_on: 
      - db
    image: wordpress:latest
    volumes:
      - ./app_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: wordpress_user
      WORDPRESS_DB_PASSWORD: 123456
```

- `version:3.7` : docker 버전 별 각각 사용 가능한 docker-compose.yml 버전의 차이가 있음
    - [Release Note](https://docs.docker.com/compose/release-notes/)
- `service:{Image Container Documents}` : 생성할 Container들을 정의
    - `db:`, `app:` : 생성할 Container의 명칭
    - `depends_on:\ - db` : "db" Container 가 생성된 이후에 생성
    - `image:mysql:5.7` : 5.7버전의 mysql image로 Container를 생성
    - `volumes:\ -./db_data:/var/lib/mysql` : Host의 "db_data" 디렉토리와 Container 내부의 "/var/lib/mysql/" 을 연결하는 Volume 생성
    - `ports:\ -"8080:80"` : Host의 8080 포트와 "app" Container의 80 포트를 연결
    - `environment:\ {env_details}` : 환경 변수 설정
        - `WORDPRESS_DB_HOST:db:3306` : "db" Container의 3306 포트로 접속
    - `restart: always` : 특정 요인으로 인해 Container가 실행 중 중단되었을 경우, 해당 Container를 자동으로 재시작 해줌
<br>

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.html path="assets/img/flowchart.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>

## Ref.
[생활코딩:Docker-compose](https://bit.ly/docker-compose-sample)