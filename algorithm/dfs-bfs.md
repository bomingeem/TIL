## 1. 깊이 우선 탐색(DFS, Depth-First Search)

### 깊이 우선 탐색이란?
깊이 우선 탐색은 탐색을 함에 있어서 보다 깊은 것을 우선적으로 하여 탐색하는 알고리즘  
루트 노드(혹은 다른 임의의 노드)에서 시작해서 다음 분기로 넘어가기 전에 **해당 분기를 완벽하게 탐색**하는 방식  

 - 모든 노드를 방문하고자 하는 경우에 이 방법을 선택
 - 깊이 우선 탐색(DFS)이 너비 우선 탐색(BFS)보다 좀 더 간단함
 - 검색 속도 자체는 너비 우선 탐색(BFS)에 비해서 느림


## 2. 너비 우선 탐색(BFS, Breadth-First Search)

### 너비 우선 탐색이란?
최대한 넓게 이동한 다음, 더 이상 갈 수 없을 때 아래로 이동  
루트 노드(혹은 다른 임의의 노드)에서 시작해서 인접한 노드를 먼저 탐색하는 방법으로, 시작 정점으로부터 가까운 정점을 먼저 방문하고 멀리 떨어져 있는 정점을 나중에 방문하는 순회 방법  

 - 주로 두 노드 사이의 **최단 경로**를 찾고 싶을 때 이 방법을 선택


## 3. 깊이 우선 탐색(DFS)과 너비 우선 탐색(BFS) 비교
|DFS|BFS|
|------|------|
|현재 정점에서 갈 수 있는 점들까지 들어가면서 탐색|현재 정점에 연결된 가까운 점들부터 탐색|
|스택 또는 재귀함수로 구현|큐를 이용해서 구현|


#### 깊이 우선 탐색(DFS)과 너비 우선 탐색(BFS) 활용한 문제 유형/응용
1) 그래프의 **모든 정점을 방문**하는 것이 주요한 문제
   단순 모든 정점을 방문하는 것이 중요한 문제의 경우 DFS, BFS 두 가지 방법 가능
2) **경로의 특징**을 저장해둬야 하는 문제
   a부터 b까지 가는 경로를 구하는데 경로에 같은 숫자가 있으면 안된다는 문제 등 각각의 경로마다 특징을 저장해둬야 할 때는 DFS를 사용
3) **최단거리**를 구해야 하는 문제
   미로 찾기 등 최단거리를 구해야 하는 경우 BFS 사용

   
   
   
