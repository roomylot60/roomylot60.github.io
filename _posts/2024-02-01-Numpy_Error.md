---
title: Numpy Version Error
author: Patrick
date: 2024-02-01 13:18:00 +0900
tags: []
categories: []
render_with_liquid: false
---
## Error
### AttributeError
- numpy importing 문제 발생

```powershell
module 'numpy' has no attribute 'object'.
```

#### 해결
- numpy의 버전으로 인한 문제로 추정
	- 버전 설정해서 재 설치 `pip install numpy==1.23.4`

### ImportError
- matplotlib importing 문제 발생

```powershell
DLL load failed while importing _imaging: 지정된 모듈을 찾을 수 없습니다.
```

#### 생각할 수 있는 문제점
1. 시스템 환경 변수 문제
2. 설치된 모듈의 버전 문제
#### 해결
1. 시스템 환경 변수 편집 >> 환경변수 >> Path 편집 >> `C:\Users\dhsmf\anaconda3\Library\bin` 추가
2. `conda remove [모듈명]` >> `conda install [모듈명]` 혹은 `pip install [모듈명]`
3. `pip install -U Pillow`로 해결
	- 이미지를 분석하고 처리하는 Library
	- matplotlib 에서 그래프 생성과 같은 이미지 처리 기능이 있으므로 이와 관련된 것으로 추정
### UnicodeDecodeError
- json 데이터 파일 loading 문제 발생

```powershell
'utf-8' codec can't decode byte 0xbf in position 12: invalid start byte
```

#### 해결
- codec 문제 >> 'UTF8'로 decode 하는 과정에서 발생
	- `json.load(open(DATA_IN_PATH + DATA_CONFIGS, 'r', encoding='cp949'))`로 encoding 설정 변경으로 해결

### LookupError

```powershell
Resource stopwords not found. Please use the NLTK Downloader to obtain the resource: >>> import nltk >>> nltk.download('stopwords')
```

- nltk를 활용하기 위해서는 해당 모듈에서 제공하는 data를 다운 받아야 함

```python
import nltk
nltk.download()
```

## Parser
### BeautifulSoup
- HTML tag를 Parsing하여 image, text, contents를 사용자가 편리하게 가져올 수 있게 제공하는 python library
### HTML Parser
- BeautifulSoup은 HTML parser가 아니므로 지원하는 parser를 설치 및 사용해야 함

| Parser | Usage | Adventage | Disadventage |
| :--: | :--: | :--: | :--: |
| html.parser | `BeautifulSoup(markup,'html.parser')` | 알맞은 속도 Lenient(python 2.7.3 및 3.2 기준) | lxml만큼 빠르지 않고, html5lib보다 덜 관대함 |
| lxml | `BeautifulSoup(markup,'lxml')` | 매우 빠르고, 허술함 | 외부 C library의존성 |
| lxml-xml | `BeautifulSoup(markup,'lxml-xml')`, `BeautifulSoup(markup,'xml')` | 매우 빠르고, 현재 지원하는 유일 XML parser | 외부 C library의존성 |
| html5lib | `BeautifulSoup(markup,'html5lib')` | 매우 관대하고 웹 브라우저와 동일한 방식으로 페이지 구분 분석, 유효한 HTML5 문법 제공 | 아주 느리고, 외부 python library의존성 |

