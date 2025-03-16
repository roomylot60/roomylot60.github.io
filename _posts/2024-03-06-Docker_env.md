---
title: 
author: Patrick
date: 2024-03-06 13:18:00 +0900
tags: []
categories: []
render_with_liquid: false
---

```bash
docker run -i -t --name container_name ubuntu:latest /bin/bash
```

- `/bin/bash` : bash를 기본 쉘로 지정하여 컨테이너를 실행

![](Attatched/Pasted%20image%2020240306053848.png)
- 원인 : Windows나 Mac 등에서 텍스트 편집기를 통해 작성한 스크립트에 CR+LF 가 리눅스 쉘 스크립트 파서에서 오류를 발생시킴
- 해결 : Git bash 혹은 WSL이 아닌 powershell에서 실행하여 해결함

```docker
ENV LANG=C.UTF-8
ENV TZ=Asia/Seoul
ENV DEBIAN_FRONTEND=noninteractive
```

1. `ENV LANG=C.UTF-8`: 이 환경 변수는 컨테이너 내에서 사용되는 기본 문자 인코딩을 설정합니다. `C.UTF-8`은 UTF-8 인코딩을 사용하는 로케일로 설정됩니다. 이는 Unicode 문자 집합을 처리하기 위한 표준 인코딩으로 사용됩니다.
    
2. `ENV TZ=Asia/Seoul`: 이 환경 변수는 컨테이너의 타임존을 설정합니다. `Asia/Seoul`은 대한민국의 시간대인 한국 표준시(KST)를 나타냅니다. 이를 통해 컨테이너의 시간 설정이 한국 표준시로 맞추어집니다.
    
3. `ENV DEBIAN_FRONTEND=noninteractive`: 이 환경 변수는 Debian 패키지 관리자를 비대화식(non-interactive) 모드로 설정합니다. 이 설정은 패키지 설치 중에 대화식 프롬프트를 피하기 위한 것으로, 사용자 입력을 요구하지 않고 기본값을 사용하여 패키지를 설치하게 됩니다. 이는 Docker 이미지를 빌드할 때 패키지 설치 중에 무한정 대기하지 않도록 하기 위한 것입니다.

```docker
RUN apt-get update && \
    apt-get install -y --no-install-recommends tzdata g++ git curl
```

1. `apt-get update`: 패키지 매니저를 사용하여 패키지 목록을 업데이트합니다. 이 명령은 컨테이너의 패키지 매니저가 최신의 패키지 목록을 가지고 있도록 합니다.
    
2. `apt-get install -y --no-install-recommends tzdata g++ git curl`: 패키지 매니저를 사용하여 `tzdata`, `g++`, `git`, `curl` 패키지를 설치합니다. 이 때, `-y` 옵션은 패키지 설치 중 사용자에게 확인을 요구하지 않고 기본값으로 설치를 진행하도록 합니다. `--no-install-recommends` 옵션은 해당 패키지가 의존하는 다른 패키지 중 권장(recommended) 패키지를 설치하지 않도록 합니다. 이렇게 함으로써 최소한의 패키지만 설치되고, 불필요한 의존성을 줄일 수 있습니다.
	- `tzdata` : timezone 데이터를 제공하는 패키지로, 컨테이너의 시간대를 설정하고, 시간 관련 작업을 수행할 때 필요한 시간대 데이터를 확보