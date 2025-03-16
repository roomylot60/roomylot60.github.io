---
title: 
author: Patrick
date: 2024-03-21 13:18:00 +0900
tags: []
categories: []
render_with_liquid: false
---
## Korean NLP Chatbot

### Data Preprocessing
#### Normalization(정규화)
- **Regular Expression(정규 표현식)** : 특정 문자열 패턴(규칙)을 검색, 치환, 제거하여 데이터를 정제할 때 사용
- `re` 패키지에서 `compile()` 매서드를 활용하여 정규식 객체를 생성

```python
import re
RE = re.compile("PATTERN")
```

| 표현               | 설명               | 예시          | 예시 설명                |
| ---------------- | ---------------- | ----------- | -------------------- |
| `^`              | 이 패턴으로 시작        | `^abc`      | "abc"로 시작해야 함        |
| `$`              | 이 패턴으로 종료        | `xyz$`      | "xyz"로 종료되어야 함       |
| `+`              | 앞 패턴 하나 이상이어야 함  | `a-z+`      | "a~z" 중 하나는 포함       |
| `[문자들]`          | 가능한 문자들의 집합을 정의  | `[Pp]ython` | "Python" 혹은 "python" |
| `[^문자들]`         | 피해야할 문자들의 집합을 정의 | `[^aeiou]`  | 소문자 모음이 아닌 문자들       |
| `\d`             | 숫자 0~9           | `\d\d\d`    | 숫자 3개(000,123 등)     |
| `\w`             | 영어 대소문자, 숫자, '_' | `\w\w\w`    | 문자 3개(ABC, xy_ 등)    |
| `\s`, `\t`, `\n` | 공백               | `\s\s`      | 스페이스 문자 2개           |

#### Tokenize to Morpheme(형태소화)
- `Konlpy.tag` 패키지에서 `Okt()`를 사용
	- Open Korean Tokenizer : 형태소 단위로 문장을 문해

#### Vocabulary Dictionary
