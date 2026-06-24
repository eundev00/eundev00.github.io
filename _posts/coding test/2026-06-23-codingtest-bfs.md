---

## title: 코딩테스트 BFS
date: 2026-06-23 09:00:00 +0900
categories: [Codingtest]
tags: [Codingtest, BFS]

---

# 코딩테스트 BFS 완전 정리

## 1. BFS란?

BFS = 가까운 것부터 순서대로 탐색

출발점에서 거리 1인 곳 전부 → 거리 2인 곳 전부 → 거리 3 ...

```
        [S]
       /   \
     [1]   [1]      ← 거리 1인 노드들 먼저
    /  \  /  \
  [2] [2][2] [2]    ← 그 다음 거리 2
```

이 특성 때문에 **처음 목적지 도달 = 무조건 최단거리** 가 보장된다.

DFS는 이 보장이 없어서 최단거리엔 BFS를 써야 한다.

---

## 2. BFS 원리 개념

### Queue가 핵심인 이유

BFS는 **Queue(FIFO)** 구조 덕분에 가까운 것부터 자동으로 처리된다.

```
Queue = 먼저 들어온 것이 먼저 나온다 (줄 서기)

Stack을 쓰면 → DFS (깊이 우선)
Queue를 쓰면 → BFS (너비 우선)   ← 이게 핵심
```

---

### 단계별 탐색 시각화

```
그래프 구조:
    1
   / \
  2   3
 / \   \
4   5   6
```

```
STEP 0  시작
Queue: [1]
visited: 1

STEP 1  1을 꺼냄 → 인접한 2, 3 삽입
Queue: [2, 3]
visited: 1-2-3

STEP 2  2를 꺼냄 → 인접한 4, 5 삽입
Queue: [3, 4, 5]
visited: 1-2-3-4-5

STEP 3  3을 꺼냄 → 인접한 6 삽입
Queue: [4, 5, 6]
visited: 1-2-3-4-5-6

STEP 4  4, 5, 6 차례로 꺼냄 → 인접 없음
Queue: []
탐색 종료
```

방문 순서: `1 → 2 → 3 → 4 → 5 → 6`

거리별로 정확하게 층층이 탐색된다.

---

### 거리 보장 원리

```
Queue에 들어오는 순서 = 거리 순서

거리 0 │ 1
거리 1 │ 2  3
거리 2 │ 4  5  6

Queue는 거리 0 → 거리 1 → 거리 2 순으로 처리
→ 목적지에 처음 도달하는 순간 = 반드시 최단거리
```

---

### DFS와 비교

```
같은 그래프, 같은 시작점 BFS vs DFS

       1
      / \
     2   3
    /
   4

BFS(1): 1 → 2 → 3 → 4     너비 우선 (층별)
DFS(1): 1 → 2 → 4 → 3     깊이 우선 (끝까지)

최단거리 목적지가 3이라면?
BFS → 2번 만에 도달 ✅ (최단 보장)
DFS → 4까지 갔다가 돌아와서 3 도달 ❌ (최단 보장 없음)
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
"최소 횟수/시간/비용" 요구?  ── No ──▶ DFS 고려
    │ Yes
    ▼
이동 비용이 모두 같은가?  ── No ──▶ 다익스트라
    │ Yes
    ▼
         BFS 확정 ✅
```

### 문제에서 보이는 신호 단어들

| 신호 | 예시 표현 |
| --- | --- |
| 최단거리 | "최소 시간", "최대한 빠르게", "가장 적은 횟수" |
| 격자 탐색 | "상하좌우 이동", "한 칸씩 이동", "격자", "미로" |
| 균일 비용 | "한 번에 1칸", "1초에 한 칸" |
| 연결 탐색 | "인접한", "연결된", "붙어있는" |

---

## 4. BFS 풀이 5단계 흐름

```
① 그래프/격자 파악      지도 크기, 시작/끝 위치 확인
        ↓
② 이동 방향 정의        상하좌우 or 대각선 등
        ↓
③ visited 배열 선언     방문 체크 (무한루프 방지)
        ↓
④ Queue에 시작점 삽입   (위치, 거리) 함께 저장
        ↓
⑤ Queue가 빌 때까지     꺼내기 → 조건 체크 → 인접칸 삽입
```

---

## 5. 기본 BFS 코드 구조 (C# 암기용)

```csharp
void BFS(int start)
{
    // ① visited 선언
    bool[] visited = new bool[N + 1];

    // ② Queue 생성 + 시작점 삽입
    var queue = new Queue<int>();
    queue.Enqueue(start);
    visited[start] = true;

    // ③ 탐색 루프
    while (queue.Count > 0)
    {
        // ④ 꺼내기
        int node = queue.Dequeue();
        Console.WriteLine(node); // 현재 노드 처리

        // ⑤ 인접 노드 탐색
        foreach (int next in graph[node])
        {
            if (visited[next]) continue; // 이미 방문했으면 스킵

            visited[next] = true;
            queue.Enqueue(next);
        }
    }
}
```

### 그래프 입력 세팅

```csharp
int N = 5; // 노드 수
var graph = new List<int>[N + 1];
for (int i = 0; i <= N; i++)
    graph[i] = new List<int>();

// 간선 연결 (양방향)
graph[1].Add(2); graph[2].Add(1);
graph[1].Add(3); graph[3].Add(1);
graph[2].Add(4); graph[4].Add(2);
```

### 실행 흐름 예시

```
그래프:  1 - 2 - 4
         |
         3 - 5

BFS(1) 실행하면:

Queue 상태          방문 순서
[1]             →   1 출력
[2, 3]          →   2 출력
[3, 4]          →   3 출력
[4, 5]          →   4 출력
[5]             →   5 출력
```

> 격자 문제는 이 구조에서 `foreach`를 `4방향 for문`으로 바꾼 것뿐이다.
> 
> 
> 디폴트 구조만 외우면 나머지는 응용이다.
> 

---

## 6. 격자 BFS 코드 구조 (4방향 응용)

```csharp
int[] dr = { -1, 1, 0, 0 };
int[] dc = {  0, 0,-1, 1 };

int BFS((int r, int c) start, (int r, int c) end)
{
    bool[,] visited = new bool[rows, cols];

    var queue = new Queue<(int r, int c, int dist)>();
    queue.Enqueue((start.r, start.c, 0));
    visited[start.r, start.c] = true;

    while (queue.Count > 0)
    {
        var (r, c, dist) = queue.Dequeue();

        if (r == end.r && c == end.c) return dist;

        for (int d = 0; d < 4; d++)
        {
            int nr = r + dr[d];
            int nc = c + dc[d];

            if (nr < 0 || nr >= rows) continue;
            if (nc < 0 || nc >= cols) continue;
            if (visited[nr, nc])      continue;
            if (maps[nr][nc] == 'X')  continue; // 벽 조건

            visited[nr, nc] = true;
            queue.Enqueue((nr, nc, dist + 1));
        }
    }

    return -1; // 도달 불가
}
```

---

## 7. BFS 응용 패턴 3가지

### 패턴 1 — 단순 최단거리

```
S ──────────────────▶ E
        BFS 1번
```

예시: 미로 최단거리, 섬 탐색

---

### 패턴 2 — 경유지 있는 최단거리

```
S ──────▶ L ──────▶ E
   BFS1       BFS2

정답 = BFS1 결과 + BFS2 결과
```

예시: 레버 당기고 탈출, 중간 아이템 획득 후 목적지

```csharp
int d1 = BFS(S, L);
if (d1 == -1) return -1;

int d2 = BFS(L, E);
if (d2 == -1) return -1;

return d1 + d2;
```

---

### 패턴 3 — 상태를 같이 들고 다니는 BFS

```csharp
// visited를 3D로 선언
bool[,,] visited = new bool[rows, cols, 상태수];

// Queue에 상태도 함께 저장
queue.Enqueue((r, c, 상태, dist));
```

예시: 벽 한 번 부수고 이동, 열쇠 획득 후 문 열기

→ 상태가 바뀌면 같은 칸도 다시 방문 가능

---

## 8. 자주 하는 실수 & 체크리스트

```
□ visited 체크를 Queue에 넣을 때 해야 함
  (꺼낼 때 하면 중복 삽입 발생 → 시간 초과)

□ 범위 체크를 visited보다 먼저
  (배열 인덱스 초과 에러 방지)

□ 경유지 있으면 BFS를 나눠서
  (visited를 공유하면 안 됨, 각각 초기화)

□ 제출 전 Console.WriteLine 디버그 코드 제거
```

---

## 9. 문제 유형별 한눈에 정리

| 유형 | 방법 | visited |
| --- | --- | --- |
| 단순 최단거리 | BFS 1번 | 2D 배열 |
| 경유지 포함 | BFS N번 분리 | 매번 새로 초기화 |
| 상태 포함 | BFS 1번 + 상태 | 3D 배열 |
| 비용 다를 때 | 다익스트라 | 거리 배열 |

---

## 10. 문제 받으면 판단 순서

```
격자/그래프?  →  최솟값?  →  비용 균일?  →  경유지/상태?
                                              ↓
                                   BFS 몇 번 쓸지 결정
                                              ↓
                                       템플릿 적용
```
