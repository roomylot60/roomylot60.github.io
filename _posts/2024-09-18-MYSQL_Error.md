---
title: MySQL Initial Setting for Password
author: Patrick
date: 2024-09-18 16:25:00+0900
tags: [Error]
categories: [MySQL, Database]
render_with_liquid: false
---
#### ERROR 1170 (42000)
- `ERROR 1170 (42000) at line 6: BLOB/TEXT column 'url' used in key specification without a key length`
- UNIQUE 로 설정한 문자열에 대해서는 길이를 지정해야 함
- 지정되어 있던 형식 TEXT 에서 VARCHAR 로 변경하면서 길이를 명시