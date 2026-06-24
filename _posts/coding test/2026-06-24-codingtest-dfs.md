---

title: 코딩테스트 DFS 
date: 2026-06-24 09:00:00 +0900
categories: [Codingtest]
tags: [Codingtest, DFS]

---

# 코딩테스트 DFS 완전 정리

---

## 1. DFS란?

DFS = 한 방향으로 갈 수 있는 끝까지 탐색 후, 돌아와서 다음 방향 탐색

```
        [S]
       /   \
     [1]   [5]
    /  \      \
  [2]  [4]   [6]
  /
[3]

방문 순서: S → 1 → 2 → 3 → 4 → 5 → 6
끝까지 파고든 뒤 되돌아오는 방식
```

BFS와 달리 **최단거리 보장이 없다.**  
대신 **모든 경우 탐색, 가능 여부 판단, 조합/순열** 문제에 적합하다.

---

## 2. DFS 원리 개념

### Stack(재귀)이 핵심인 이유

DFS는 **재귀 호출(Call Stack)** 또는 **Stack(LIFO)** 덕분에 깊이 우선으로 동작한다.

```
Stack = 나중에 들어온 것이 먼저 나온다

Queue를 쓰면 → BFS (너비 우선)
Stack을 쓰면 → DFS (깊이 우선)   ← 이게 핵심
```

---

### 단계별 탐색 시각화

```
그래프 구조:
    1
   / \
  2   3
 / \
4   5
```

```
STEP 1  1 방문
재귀 스택: [1]

STEP 2  1의 첫 인접 노드 2 방문
재귀 스택: [1, 2]

STEP 3  2의 첫 인접 노드 4 방문
재귀 스택: [1, 2, 4]

STEP 4  4의 인접 없음 → 되돌아가기
재귀 스택: [1, 2]

STEP 5  2의 다음 인접 노드 5 방문
재귀 스택: [1, 2, 5]

STEP 6  5의 인접 없음 → 되돌아가기
재귀 스택: [1]

STEP 7  1의 다음 인접 노드 3 방문
재귀 스택: [1, 3]
```

방문 순서: `1 → 2 → 4 → 5 → 3`

---

### BFS와 비교

```
같은 그래프, 같은 시작점 BFS vs DFS

       1
      / \
     2   3
    /
   4

BFS(1): 1 → 2 → 3 → 4     너비 우선 (층별)
DFS(1): 1 → 2 → 4 → 3     깊이 우선 (끝까지)

"3에 도달 가능한가?" 라는 문제라면?
BFS → 가능 ✅ (최단거리도 앎)
DFS → 가능 ✅ (최단거리는 모름)

"모든 경로를 탐색하라" 라는 문제라면?
BFS → 부적합
DFS → 적합 ✅
```

---

## 3. 문제 받았을 때 유추 흐름

```
문제 읽기
    │
    ▼
입력이 격자(2D) 또는 그래프?  ── No ──▶ 다른 유형
    │ Yes
    ▼
"가능 여부" or "모든 경우" 요구?  ── No ──▶ BFS 고려
    │ Yes
    ▼
순서/조합/백트래킹 필요?
    │ Yes
    ▼
         DFS 확정 ✅
```

### 문제에서 보이는 신호 단어들

| 신호 | 예시 표현 |
|---|---|
| 가능 여부 | "연결되어 있는지", "갈 수 있는지", "탈출 가능한지" |
| 모든 경우 | "모든 경로", "경우의 수", "가능한 조합" |
| 완전 탐색 | "모든 칸을 방문", "전체 탐색" |
| 연결 요소 | "섬의 개수", "덩어리 개수", "연결된 그룹" |
| 백트래킹 | "조건에 맞는 경우만", "N개 중 M개 선택" |

---

## 4. DFS 풀이 5단계 흐름

```
① 그래프/격자 파악      지도 크기, 시작 위치 확인
        ↓
② 탈출 조건 정의        언제 멈출지 (목적지 도달, 범위 초과 등)
        ↓
③ visited 배열 선언     방문 체크 (무한루프 방지)
        ↓
④ 시작점에서 DFS 호출   재귀 or 스택으로 시작
        ↓
⑤ 인접 노드 탐색        조건 맞으면 재귀 호출 → 복귀 시 되돌리기(백트래킹)
```

---

## 5. 기본 DFS 코드 구조 (C# 암기용)

### 방법 1 — 재귀 (일반적으로 더 많이 씀)

```csharp
bool[] visited = new bool[N + 1];
List<int>[] graph = new List<int>[N + 1];

void DFS(int node)
{
    // ① 현재 노드 처리
    visited[node] = true;
    Console.WriteLine(node);

    // ② 인접 노드 탐색
    foreach (int next in graph[node])
    {
        if (visited[next]) continue; // 방문했으면 스킵

        DFS(next); // 재귀 호출
    }
    // ③ 재귀 종료 시 자동으로 이전 노드로 복귀
}
```

---

### 방법 2 — 스택 (재귀 깊이 제한 우려 시)

```csharp
void DFS(int start)
{
    bool[] visited = new bool[N + 1];
    var stack = new Stack<int>();

    stack.Push(start);

    while (stack.Count > 0)
    {
        int node = stack.Pop();

        if (visited[node]) continue;
        visited[node] = true;

        Console.WriteLine(node);

        foreach (int next in graph[node])
        {
            if (!visited[next])
                stack.Push(next);
        }
    }
}
```

> 재귀 방식이 코드가 더 짧고 직관적이어서 코테에서 주로 사용.  
> 단, 깊이가 매우 깊을 경우 (노드 수 10만 이상) 스택 오버플로우 우려 → 스택 방식 사용.

---

## 6. 격자 DFS 코드 구조 (4방향 응용)

```csharp
int[] dr = { -1, 1, 0, 0 };
int[] dc = {  0, 0,-1, 1 };
bool[,] visited;
string[] maps;
int rows, cols;

void DFS(int r, int c)
{
    // ① 방문 처리
    visited[r, c] = true;

    // ② 4방향 탐색
    for (int d = 0; d < 4; d++)
    {
        int nr = r + dr[d];
        int nc = c + dc[d];

        // ③ 범위 + 벽 + 방문 체크
        if (nr < 0 || nr >= rows) continue;
        if (nc < 0 || nc >= cols) continue;
        if (visited[nr, nc])      continue;
        if (maps[nr][nc] == 'X')  continue; // 벽 조건

        DFS(nr, nc); // 재귀
    }
}
```

---

## 7. 백트래킹 패턴 (DFS의 핵심 응용)

백트래킹 = DFS로 탐색하다가 조건에 맞지 않으면 **되돌아가는 것**

```csharp
bool[] used = new bool[N + 1];
List<int> path = new List<int>();

void DFS(int depth)
{
    // ① 목표 깊이 도달 시 결과 저장
    if (depth == M)
    {
        Console.WriteLine(string.Join(" ", path));
        return;
    }

    for (int i = 1; i <= N; i++)
    {
        if (used[i]) continue;

        // ② 선택
        used[i] = true;
        path.Add(i);

        DFS(depth + 1); // 재귀

        // ③ 되돌리기 (백트래킹 핵심)
        used[i] = false;
        path.RemoveAt(path.Count - 1);
    }
}
```

```
선택 → 재귀 → 되돌리기
이 3단계가 백트래킹의 전부
```

---

## 8. DFS 응용 패턴 3가지

### 패턴 1 — 연결 요소 개수 세기

```csharp
int count = 0;
for (int i = 0; i < rows; i++)
    for (int j = 0; j < cols; j++)
        if (!visited[i, j] && maps[i][j] != 'X')
        {
            DFS(i, j); // 연결된 덩어리 전체 방문
            count++;   // 덩어리 하나 카운트
        }
```

예시: 섬의 개수, 바이러스 감염 영역

---

### 패턴 2 — 경로 존재 여부

```csharp
bool found = false;

void DFS(int r, int c)
{
    if (r == endR && c == endC) { found = true; return; }
    if (found) return; // 찾으면 조기 종료

    visited[r, c] = true;
    // 4방향 탐색 ...
}
```

예시: 출구 도달 가능 여부, 연결 여부 확인

---

### 패턴 3 — 모든 경로 탐색 (백트래킹)

```csharp
void DFS(int r, int c)
{
    if (r == endR && c == endC) { result++; return; }

    visited[r, c] = true;       // 선택
    // 4방향 탐색 ...
    visited[r, c] = false;      // 되돌리기
}
```

예시: 미로에서 가능한 경로 수, N과 M 조합

---

## 9. 자주 하는 실수 & 체크리스트

```
□ 재귀 탈출 조건을 반드시 먼저 작성
  (없으면 무한 재귀 → 스택 오버플로우)

□ 백트래킹 시 visited 되돌리기 잊지 않기
  (되돌리지 않으면 다른 경로 탐색 불가)

□ 범위 체크를 visited보다 먼저
  (배열 인덱스 초과 에러 방지)

□ 노드 수가 매우 많을 때 재귀 깊이 주의
  (C# 기본 스택 깊이 제한 있음 → 스택 방식 고려)

□ 제출 전 Console.WriteLine 디버그 코드 제거
```

---

## 10. 문제 유형별 한눈에 정리

| 유형 | 방법 | visited |
|---|---|---|
| 연결 요소 개수 | DFS 반복 호출 | 2D 배열 |
| 경로 존재 여부 | DFS + found 플래그 | 2D 배열 |
| 모든 경로 탐색 | DFS + 백트래킹 | 2D 배열 (되돌리기) |
| 조합/순열 | DFS + 백트래킹 | 1D used 배열 |

---

## 11. BFS vs DFS 한눈에 비교

| 항목 | BFS | DFS |
|---|---|---|
| 자료구조 | Queue | Stack (재귀) |
| 탐색 방식 | 가까운 것부터 | 깊은 곳부터 |
| 최단거리 보장 | ✅ | ❌ |
| 메모리 | 상대적으로 많음 | 상대적으로 적음 |
| 적합한 문제 | 최단거리, 최소 횟수 | 가능 여부, 모든 경우, 백트래킹 |

---

## 12. 문제 받으면 판단 순서

```
격자/그래프?
    │
    ▼
최솟값/최단거리 요구?  ── Yes ──▶ BFS
    │ No
    ▼
가능 여부 / 모든 경우 / 연결 요소?  ── Yes ──▶ DFS
    │
    ▼
조건부 선택 필요?  ── Yes ──▶ DFS + 백트래킹
```
