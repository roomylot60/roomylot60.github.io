---
title: 
author: Patrick
date: 2024-06-09 10:23:00 +0900
tags: []
categories: []
render_with_liquid: false
---

# Dockerfile 정의

## Dockerfile 구성
- Dockerfile은 생성할 Docker 컨테이너의 Image를 구성하는 설계도
- os를 비롯한 기본 환경을 구성하여 격리된 IDE를 구성한다

## 환경 설정

### 1. 기존 Image 활용
```bash
# FROM image_name
# 예시
FROM nvidia/cuda:11.8.0-cudnn8-runtime-ubuntu22.04
```

- `FROM` : Docker HUB에서 제공하는 기존의 image를 자동으로 pull하여 사용하도록 하는 명령어
  - `FROM nvidia/cuda:11.8.0-cudnn8-runtime-ubuntu22.04`:
    - nvidia/cuda에서 제공하는 ubuntu 22.04 os 기반의 환경
    - cuda 11.8.0과 관련된 toolkit 등이 설치된 환경

### 2. 기반 os 별 package manager
- 기반으로 사용되는 os(e.g. ubuntu, debian, alpine)에 맞게 설치 방법 차이 발생
- Package manager: `apt`, `apk`, `yum` 등

|베이스 이미지|	OS	|패키지 매니저|
|------------|------|------------|
|ubuntu:22.04|Ubuntu|	apt|
|debian:bullseye|	Debian|	apt|
|alpine:3.18|	Alpine	|apk|
|centos:7	|CentOS	|yum or dnf|

### 3. Package 설치
```bash
COPY requirements.txt .
RUN pip3 install --no-cache-dir -r requirements.txt
```

- `COPY requirements.txt .`:
  - `requirements.txt`에서 설정한 패키지 별 버전 로드
- `RUN pip3 install --no-cache-dir -r requirements.txt`:
  - 각 패키지의 버전에 맞게 설치를 진행
