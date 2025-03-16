---
title: 
author: Patrick
date: 2024-03-11 13:18:00 +0900
tags: []
categories: []
render_with_liquid: false
---
### Docker ubuntu 컨테이너에서 GPU 사용
- ubuntu 22.04
`ubuntu-drivers devices`

ubuntu-drivers: command not found

``` bash
apt update
apt install -y ubuntu-drivers-common
```

ERROR:root:aplay command not found

```bash
apt-get install -y alsa-utils
```

### Ref.
https://donghyun99.tistory.com/18
https://velog.io/@hanakim120/NVIDIA-graphics-driver-%EC%84%A4%EC%B9%98-Ubuntu


### Docker volume 연결
- `-v` 속성을 사용

```bash
docker run -it -v /$(pwd)/data:/data image_name bash
```

- `$(pwd)` : 현재 명령어를 실행하는 위치
- `bash` : 실행할 image에 대해 bash를 실행

```bash
# Install JVM for Konlpy  
RUN apt-get update && \  
    apt-get upgrade -y && \  
    apt-get install -y \  
    openjdk-8-jdk wget curl git python3-dev \  
    language-pack-ko
```

- Konlpy에서 사용할 JVM 설치

```bash
RUN locale-gen en_US.UTF-8 && \  
    update-locale LANG=en_US.UTF-8
```

- 언어팩 설정