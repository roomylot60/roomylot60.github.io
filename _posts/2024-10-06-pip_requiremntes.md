---
title: 
author: Patrick
date: 2024-10-06 13:18:00 +0900
tags: []
categories: []
render_with_liquid: false
---
## pip

### 개발환경 관리

- 'requirements.txt' 를 사용
- `pip freeze > requirements.txt` : 현재 개발환경에 적용된 패키지 다운로드 주소를 포함한 정보들을 저장
- `pip list --format=freeze > requirements.txt` : `pip list`를 사용하여 해당 패키지들의 버전 정보만을 저장
- `pip install -r requirements.txt` : 해당 텍스트 파일에 저장된 패키지 버전 정보를 활용하여 `pip`에서 설치