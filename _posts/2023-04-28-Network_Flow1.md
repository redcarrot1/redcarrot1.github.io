---
title: Network Flow 1-최대 유량 알고리즘(Edmonds-Karp)
date: 2023-04-28 12:00:00 +09:00
categories: [알고리즘]
tags:
  [
    Network Flow,
    Edmonds-Karp,
    Maximum Flow
  ]
---

## 개요
네트워크 플로우 알고리즘(Network Flow Algorithm)은 그래프 이론에서 사용되며, 여러 목적에 맞게 사용될 수 있다.<br>
그 중 이번 포스팅에서 알아볼 것은 최대 유량(maximum flow)을 찾는 알고리즘이다. 이때 유량(flow)은 한 노드에서 다른 노드로의 데이터 전송량을 의미한다.

예를 들어, 기름을 전송하는 파이프(pipe)를 떠올려보자. 각 파이프는 지름 등의 요인으로 인해 한 번에 전송할 수 있는 용량(Capacity)를 가진다.<br>
여러 파이프가 연결된 경우를 생각해보자. 연결된 긴 파이프의 용량은 몇인가?<br>
**연결된 파이프들의 용량 중 가장 작은 값과 같다.**

네트워크 유량의 특징들이 존재한다. 잘 정리되어 있는 [블로그](https://iknoom.tistory.com/13){:target="_blank"}에서 확인할 수 있다. 알고리즘 증명은 [갓사과님 블로그](https://koosaga.com/133){:target="_blank"}에서 확인할 수 있다.


## Edmonds-karp
> 특정 시작 노드에서 도착 노드까지의 최대 용량을 구하자.
{: .prompt-info }

두 노드간의 최대 유량을 구하는 알고리즘 중 하나가 에드몬드 카프(Edmonds-Karp) 알고리즘이다. 이 알고리즘은 너비 우선 탐색(Breadth-First Search, BFS) 알고리즘을 사용하여 최대 용량을 찾는다. 시간 복잡도는 O(VE^2)이다.

에드몬드-카프 알고리즘의 동작 방식은 다음과 같다.

1. 초기에는 최대 용량을 0으로 설정합니다.
2. BFS 알고리즘을 사용하여 시작 노드에서 도착 노드까지의 경로를 찾습니다.
3. 경로에서 최소 용량을 찾습니다.
4. 해당 경로 상의 각 간선에 대해 최소 용량을 빼고, 그 값을 최대 용량에 더합니다.
5. 2번부터 4번까지 반복합니다.

### 코드

먼저 전체 코드를 본 후, 하나씩 분석해보자.

```c++
int n, capacity[100][100], flow[100][100], parent[100];
vector<int> edge[100];

int maxFlow(int start, int end) {
    int result = 0;
    while (1) {
        fill(parent, parent + 100, -1);
        queue<int> q;
        q.push(start);
        while (!q.empty()) {
            int here = q.front();
            q.pop();
            for (int i = 0; i < edge[here].size(); ++i) {
                int there = edge[here][i];
                if (parent[there] == -1 && capacity[here][there] > flow[here][there]) {
                    q.push(there);
                    parent[there] = here;
                    if (there == end) break;
                }
            }
        }

        if (parent[end] == -1) break;
        int minCapacity = MAXINT;
        int here = end;
        while (here != start) {
            minCapacity = min(minCapacity, capacity[parent[here]][here] - flow[parent[here]][here]);
            here = parent[here];
        }

        here = end;
        while (here != start) {
            flow[parent[here]][here] += minCapacity;
            flow[here][parent[here]] -= minCapacity;
            here = parent[here];
        }
        result += minCapacity;
    }
    return result;
}
```
전역 변수의 의미는 아래와 같다.
- `n`: 노드 개수
- `capacity[][]`: 각 네트워크(파이프)의 용량
- `flow[][]`: 현재 해당 네트워크에 흘러보내고 있는 유량
- `parent[]`: 흘러보낼 경로를 역추적하며 파악하기 위해 선언. -1이면 아직 탐색하지 않는 노드.
- `vector<int> edge[]`: 그래프 엣지

  
<br/>

  
```c++
while (1) {
    fill(parent, parent + 100, -1);
    queue<int> q;
    q.push(start);
    ...
}
```

에드몬드 카프 알고리즘은 BFS를 여러번 수행한다.(`while(1)`)<br>
먼저 `parent[]`를 -1로 초기화 후, 시작노드를 queue에 넣어 탐색할 준비를 한다.

<br/>

```c++
while (!q.empty()) {
    int here = q.front();
    q.pop();
    for (int i = 0; i < edge[here].size(); ++i) {
        int there = edge[here][i];

        // 방문하지 않고, 유량을 추가로 흘러보낼 수 있는 노드 탐색
        if (parent[there] == -1 && capacity[here][there] > flow[here][there]) {
            q.push(there);
            parent[there] = here;
            if (there == end) break;
        }
    }
}
```

가장 일반적인 BFS 구현이다. 이때 if문에 capacity보다 flow가 작아야 탐색을 하게 되는 부분만 주의하면 된다.<br>
또한 `parent[]`에 값을 넣어 경로를 기록하는 것도 확인하자.

<br/>

```c++
if (parent[end] == -1) break;
```

BFS 수행 때 end 노드를 한번도 방문하지 않았다는 것은 더 이상 유량을 흘러보낼 방법이 없다는 것이다.<br>
따라서 최대 유량을 구했을 것을 보장하므로 break를 통해 `while(1)`을 빠져나와 함수를 종료하게 된다.

<br>

```c++
int minCapacity = MAXINT;
int here = end;
while (here != start) {
    minCapacity = min(minCapacity, capacity[parent[here]][here] - flow[parent[here]][here]);
    here = parent[here];
}
```

연결된 경로의 네트워크(파이프) 중 현재 추가로 흘러보낼 수 있는 최소 유량을 구한다.<br>
이때 `parent[]`를 이용해 end 노드부터 역추적하며 경로를 따라간다.


<br/>

  
```c++
here = end;
while (here != start) {
    flow[parent[here]][here] += minCapacity;
    flow[here][parent[here]] -= minCapacity;
    here = parent[here];
}
result += minCapacity;
```

 minCapacity을 구했다면 다시 역추적하며 네트워크에 흘러보낼 유량을 더한다.<br>
 result는 우리가 구하고자 하는 최대 유량을 뜻하는 변수이다.


## 마치며
네트워크 플로우는 다른 알고리즘에 비해 증명이 난해하고, 기본 특징을 알아야 코드를 이해할 수 있다. 따라서 쉽게 포기하지 말고, 다른 블로그 포스팅도 참고해 이해하도록 노력하자.<br>
네트워크 플로우는 최대 유량 문제 외에도, 최소 컷(minimum cut), [이분 매칭](https://redcarrot1.github.io/posts/%EC%9D%B4%EB%B6%84_%EB%A7%A4%EC%B9%AD/){:target="_blank"}, MCMF 등 여러 문제로 나눌 수 있다.
또한 이 게시물에서 본 최대 유량 문제(maximum flow)도 여러 알고리즘이 존재하고, 현재도 연구 중이라고 한다.<br>
다른 알고리즘에 대해서는 추후에 시리즈로 작성할 예정이다.
  
## Reference

[https://blog.naver.com/ndb796/221237111220](https://blog.naver.com/ndb796/221237111220){:target="_blank"}

