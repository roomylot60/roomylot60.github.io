---
title: LRU Algorithm
author: Patrick
date: 2025-01-01 17:20:00+0900
tags: [LRU]
categories: [Algorithm]
render_with_liquid: false
---
## LRU (Least Recently Used) 알고리즘
- LRU(Least Recently Used, 가장 오랫동안 사용되지 않은 것) 알고리즘은 캐시 교체 알고리즘 중 하나로, 가장 오랫동안 사용되지 않은 데이터를 제거하여 새로운 데이터를 저장하는 방식

### LRU 알고리즘 동작 방식
- 캐시에 데이터가 존재하면(hit) 해당 데이터를 캐시에서 제거 후 다시 추가하여 가장 최근에 사용된 것으로 갱신
- 캐시에 데이터가 없으면(miss) 새로운 데이터를 캐시에 추가
- 캐시가 가득 찬 경우, 가장 오랫동안 사용되지 않은 데이터(맨 앞의 데이터)를 제거한 후 새로운 데이터를 추가

### LRU 알고리즘 구현 방법
- LRU를 구현하는 방식 : 가장 대표적인 방법은 Deque (이중 연결 리스트) + HashSet (또는 HashMap) 사용

#### Deque (Queue 역할)
- 캐시의 입출력 순서를 유지하며, 오래된 데이터를 쉽게 제거할 수 있음
- `collections.deque`를 사용하면 효율적으로 관리 가능
- `HashSet` 또는 `HashMap` (빠른 조회)
- `in` 연산자를 사용하여 해당 값이 캐시에 존재하는지 빠르게 확인 가능
- Python의 경우 `list.remove(value)` 연산이 \\( O(N) \\)이지만, deque를 사용하면 속도 개선 가능

#### LRU 캐시 동작 예시
- 캐시 크기 3, 요청 순서 ["A", "B", "C", "D", "A", "E", "C"]

|요청|캐시 상태 (최신 → 오래된)|Hit/Miss|실행 시간|
|--|-------|-----|---|
|A|A|Miss|5|
|B|B, A|Miss|5|
|C|C, B, A|Miss|5|
|D|D, C, B|Miss|5|
|A|A, D, C|Miss|5|
|E|E, A, D|Miss|5|
|C|C, E, A|Miss|5|

- 총 실행시간 = 5 + 5 + 5 + 5 + 5 + 5 + 5 = 35

### LRU 알고리즘의 장점과 단점
- 장점
    - 최근에 사용된 데이터를 남겨두므로, 캐시 효율성이 높음
    - 실제 컴퓨터 메모리 관리에서도 많이 활용됨
- 단점
    - 캐시 크기가 작으면 효율이 떨어질 수 있음
    - LRU 관리 비용이 발생(데이터를 갱신해야 하므로 속도 저하 가능)

### 응용 분야
- 운영체제(OS)에서 페이지 교체 알고리즘
- 데이터베이스 캐싱
- CPU 캐시 메모리 관리
- 웹 브라우저 캐싱
- 네트워크 캐시 시스템
