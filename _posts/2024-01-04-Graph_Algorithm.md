---
title: Graph Algorithm in CT
author: Patrick
date: 2024-01-04 15:47:00 +0900
categories: [Algorithm, CT]
tags: [Graph, DFS, BFS]
---
## Coding Test Graph
### 1. 깊이 우선 탐색 (Depth-First Search, DFS)
- 개념: 
    - DFS는 가능한 한 깊게 노드를 탐색한 후, 더 이상 깊이 갈 수 없을 때 이전 노드로 돌아와 다른 경로를 탐색하는 방식
    - 주로 스택 자료구조를 활용하며, 재귀 함수로 구현하는 경우가 많음
- 적용 방식:
    - 사이클 판별: 그래프 내에 사이클이 존재하는지 확인할 때 사용
    -연결 요소 확인: 그래프의 연결된 구성 요소를 찾는 데 활용​

```python
def dfs(graph, node, visited):
    visited[node] = True
    # 노드 방문 시 수행할 작업
    for neighbor in graph[node]:
        if not visited[neighbor]:
            dfs(graph, neighbor, visited)
```

### 2. 너비 우선 탐색 (Breadth-First Search, BFS)
- 개념: 
    - BFS는 현재 노드와 인접한 노드들을 먼저 탐색한 후, 그 다음 레벨의 노드들을 탐색하는 방식
    - 큐 자료구조를 활용하여 구현
- 적용 방식:
    - 최단 경로 탐색: 가중치가 동일한 그래프에서 두 노드 간의 최단 경로를 찾을 때 사용
    - 레벨 탐색: 그래프의 각 레벨별로 노드를 탐색할 때 활용

```python
from collections import deque

def bfs(graph, start):
    visited = [False] * len(graph)
    queue = deque([start])
    visited[start] = True

    while queue:
        node = queue.popleft()
        # 노드 방문 시 수행할 작업
        for neighbor in graph[node]:
            if not visited[neighbor]:
                queue.append(neighbor)
                visited[neighbor] = True
```

### 3. 다익스트라 알고리즘 (Dijkstra's Algorithm)
- 개념: 
    - 가중치가 있는 그래프에서 특정 노드로부터 다른 모든 노드까지의 최단 경로를 찾는 알고리즘
    - 우선순위 큐를 활용하여 구현하며, 간선의 가중치가 음수가 아닌 경우에만 적용 가능
- 적용 방식:
    - 최단 경로 탐색: 가중치가 있는 그래프에서 최단 경로를 찾을 때 사용됩니다.​

```python
import heapq

def dijkstra(graph, start):
    distances = {node: float('inf') for node in graph}
    distances[start] = 0
    priority_queue = [(0, start)]

    while priority_queue:
        current_distance, current_node = heapq.heappop(priority_queue)

        if current_distance > distances[current_node]:
            continue

        for neighbor, weight in graph[current_node].items():
            distance = current_distance + weight

            if distance < distances[neighbor]:
                distances[neighbor] = distance
                heapq.heappush(priority_queue, (distance, neighbor))

    return distances
```

### 4. 벨만-포드 알고리즘 (Bellman-Ford Algorithm)
- 개념: 
    - 다익스트라 알고리즘과 유사하게 최단 경로를 찾지만, 음수 가중치가 있는 그래프에서도 사용 가능
    - 모든 간선을 반복적으로 확인하여 최단 거리를 갱신하는 방식
- 적용 방식:
    - 음수 가중치 처리: 음수 가중치가 존재하는 그래프에서 최단 경로를 찾을 때 사용됩니다.​

```python
def bellman_ford(graph, start):
    distances = {node: float('inf') for node in graph}
    distances[start] = 0

    for _ in range(len(graph) - 1):
        for node in graph:
            for neighbor, weight in graph[node].items():
                if distances[node] + weight < distances[neighbor]:
                    distances[neighbor] = distances[node] + weight

    # 음수 사이클 확인
    for node in graph:
        for neighbor, weight in graph[node].items():
            if distances[node] + weight < distances[neighbor]:
                raise ValueError("음수 사이클이 존재합니다.")

    return distances
```

### 5. 플로이드-워셜 알고리즘 (Floyd-Warshall Algorithm)
- 개념: 
    - 그래프 내의 모든 노드 쌍 간의 최단 경로를 찾는 알고리즘
    - 동적 계획법을 활용하여 구현하며, 음수 가중치도 처리할 수 있지만, 음수 사이클이 존재하면 올바른 결과를 보장하지 못함​
- 적용 방식:
    - 모든 쌍 최단 경로 탐색: 그래프의 모든 노드 쌍에 대한 최단 경로를 찾을 때 사용됩니다.​

```python
def floyd_warshall(graph):
    nodes = list(graph.keys())
    distances = {node: {neighbor: float('inf') for neighbor in nodes} for node in nodes}

    for node in nodes:
        distances[node][node] = 0
        for neighbor, weight in graph[node].items():
            distances[node][neighbor] = weight

    for k in nodes:
        for i in nodes:
            for j in nodes:
                if distances[i][j] > distances[i][k] + distances[k][j]:
                    distances[i][j] = distances[i][k] + distances[k][j]

    return distances
```