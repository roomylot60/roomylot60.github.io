---
title: Docker Desktop Basic
author: Patrick
date: 2023-09-11 16:28:00 +0900
tags: [Grammer]
categories: [Docker]
render_with_liquid: false
---
# Docker
## Docker hub : 프로그램을 다운로드 받는 App store와 같은 개념
|index|Docker|Computer|
|-|--|--|
|1|docker hub|app store|
|2|image|program|
|3|container|process|
- pull : Download `images` from `docker hub`
- run : Excute `images` to make `container` 
- [Docker hub](https://hub.docker.com/)

## Docker images
- `docker pull image_name` 을 통해 이미지를 다운
- `docker images` image 설치 여부 확인

**HTTPD**
- 'Apache HTTP Server Project' 를 위한 image

## Docker command
- `docker run [option] image_name [command]` : 특정 image를 실행하여 container 생성
- `docker stop [option] container_name [container]` : run을 통해 실행한 image의 container를 정지
- `docker ps` : docker 상의 실행 중인 image에 대한 container들의 상태를 확인
- `docker ps -a` : 모든(a : all) container에 대한 상태 확인
- `docker start container_name` : stop 상태의 container를 실행(log 제외)
- `docker logs container_name` : container의 log 확인
- `docker logs -f container_name` : container의 실시간 log를 확인
- `docker rm container_name` : container 제거(실행 중인 container는 제거 불가)
- `docker rm --force container_name` : 실행 여부에 관계 없이 container 제거
- `docker rmi image_name` : docker 상에서 image 제거

## Network Basic
- Docker Host 내부에는 여러 개의 Container가 존재
- Container에 Web server와 File system 내부에 실행 시킬 파일이 존재
- User가 사용하는 컴퓨터의 Web browser를 통해 접속을 시도
- Web browser에서 정해진 path를 통해 특정 port에 접근
- 유저의 컴퓨터와 서버의 컴퓨터가 따로 있을 경우, 해당 포트로 연결함으로써 해결<br>
**Port Forwarding**
<div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/port_forwarding.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
- Docker 상의 Container 내부에 Web server가 존재할 경우, Container에 접속하기 이전에 Host 상에 있는 port를 거쳐야 함
- 따라서 `docker run -p hst_p_num:ctn_p_num img_name` 해당 명령어를 통해 Host의 port와 Container의 port를 연결해 주어야 함(p : publish)

## Command Line in Docker Container
- `docker exec [options] container_name command` : (exec : execute)
- `docker exec ctn_name /bin/sh` : container의 shell을 실행 (지속적인 연결이 되지 않아 바로 정지됨)
- `docker exec -it ctn_name /bin/sh` : (i : keep STDIN open even if no attached, t : allocate a pseudo-TTY)
- `docker exec -it ctn_name /bin/bash` : powershell이 아닌 bash shell을 실행
- `exit` : Host 상의 shell로 돌아옴
- Container 내부의 File system에 위치한 파일들을 직접 수정하는 것은 위험
    - Container가 제거될 경우 수정된 파일도 함께 제거됨
    - Host의 File system 상에서 파일을 수정하고 이를 Container 상의 파일로 반영되도록 하면 위험성 감소, 버전 관리에 용이
    - `docker run -p hst_p_num:ctn_p_num -v ~/hst_dir_path:ctn_dir_path/ img_name` : (v : volume) Host의 파일 디렉토리와 Container의 디렉토리를 Volume으로 연결
**apt, nano**
- apt : Ubuntu 관련 Linux 배포에 deb package를 install, update, remove, maintain을 위한 cmd utility
- nano : HTML 파일을 포함해 파일 내용을 수정할 수 있는 에디터(vim과 같은)