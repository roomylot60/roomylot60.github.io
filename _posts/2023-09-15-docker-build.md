---
title: Docker image making
author: Patrick
date: 2023-09-15 16:22:00+0900
tags: [Grammer]
categories: [Docker]
render_with_liquid: false
---
# Docker Image 만들기
1. commit : 기존에 만들어서 사용하던 Container를 통해 image 생성
2. build : dockerfile 생성해서 image를 제작

# Web server image 제작
## Ubuntu 기반
### 1. Commit
- `docker run --name web-server -it ubuntu:20.04` : ubuntu 20.04 버전을 통해 실시간으로 통신하는 'web-server'라는 Container 생성
- `docker commit web-server web-server-commit` : 'web-server' Container를 commit 하여 'web-server-commit"이라는 image 생성
- apt : linux 명령어 기반 설치 시스템
    - `apt update`
    - `apt install python3`
- `mkdir -p /var/www/html` : 하위 디렉토리까지 한번에 생성
- `cd /var/www/html`
- `echo "Hello, <strong>Docker</strong>" > index.html` : "Hello, <strong>Docker</strong>"의 내용을 갖는 'index.html' 파일을 생성

### 2. Build
- `Dockerfile` 생성
    - ```
        FROM ubuntu:20.04 # ubuntu 20.04 버전을 기반으로 생성 선언
        RUN apt update && apt install -y python3
        WORKDIR /var/www/html
        COPY ["index.html", "."]
        CMD ["python3", "-u","-m", "http.server"]
        ```
        - `FROM`
        - `RUN` : 운영체제 명령어 실행 build 되는 시점에서 동작하여 image에 반영. 실행할 때 마다 layer를 생성하므로 효율을 위해 한번에 연결시켜 실행을 추천
        - `CMD` : Container가 동작할 때 실행하는 명령어로 Container에 반영
        - `-y` : 설치 여부를 묻는 경우를 생략하기 위한 parameter
        - `WORKDIR` : 하위 디렉토리 생성
        - `RUN echo "Hello, <strong>Docker</strong>" > index.html` : 해당 명령어를 통해 하위 디렉토리에 index.html 파일을 생성
        - `COPY ["index.html", "."]` : Host의 'index.html'을 하위 디렉토리로 복사하라는 명령어
        - `CMD ["python3", "-u","-m", "http.server"]` : 'python3', '-u' 를 통해 동작 명령, '-m', 'http.server'를 통해 http 서버를 생성
- `docker build -t web-server-build .`: 'web-server-build'라는 이미지(t : tag)를 생성하면서 현 디렉토리에 생성할 image와 관련된 파일을 생성할 것임을 선언
- `docker run -p 8888:8000 --name web-server web-server-build` : 'web-server'라는 'web-server-build' image 기반으로 생성된 Container의 8000번 포트와 host 8888번 포트를 연결
- `docker run -p 8888:8000 --name web-server web-server-build pwd` : Overriding 하여 pwd가 먼저 실행하므로 앞의 명령은 실행되지 않음

## Push
- commit 혹은 build로 생성한 image를 Docker hub와 같은 Registry로 load 하는 것
