---
title: 전체쌍 최단경로 문제는 Floyd-Warshall 알고리즘!
categories:
  - Dev
tags:
  - floyd-warshall
  - algorithms
last_modified_at: 2018-03-24T09:00:00-00:00
---

3가지 부분에 대해서 살펴보려고 한다

1) 두 정점을 연결하는 최단경로 구하기

2) 그래프에서 정점을 삭제

3) 두 정점을 연결하는 최단경로 구하기

전체쌍 최단경로 문제는 Floyd-Warshall 알고리즘

All pairs shortest path problem

2개 테이블 사용 

하나는 모든 경로에 대한 비용을 저장하는 테이블

하나는 각 정점까지 가기 직전의 정점을 저장한 테이블

점 X에서 0번을 통해 점 Y로 가는 최단거리를 찾아보자

1번 정점에서 0번 정점을 거쳐 다른 정점 y로 갈때 거리는

그냥 1->y보다 모두 길기때문에 1->0->y는 모두 무시

\\

2번 정점에서 0번 정점을 거쳐 다른 정점 y로 갈때 거리는

그냥 2->y보다 모두 길기때문에 2->0->y는 모두 무시

\\

3번 정점에서 0번 정점을 거쳐 다른 정점 y로 갈때 거리는

그냥 3->y보다 모두 길기때문에 3->0->y는 모두 무시

\\

4번 정점에서 0번 정점을 거쳐 다른 정점 y로 갈때 거리는

그냥 4->y보다 모두 길기때문에 4->0->y는 모두 무시

// 초기화

for (int i =1; i <= n; i++){

for (int j =1; j <= n; j++){

d\[i\]\[j\] = (i==j) ? 0 : INF;

}

}

// 두 정점 연결하는 간선 가중치 입력

for (int i =1; i <= n; i++){

for (int j =i+1; j <= n; j++){

int length = sc.nextInt();

if(length == 0) length = INF;

d\[i\]\[j\] = d\[j\]\[i\] = length;

}

}

// 플로이드 - 워셜 알고리즘으로 전체경로 대해 최단경로 찾기

for(int k =0; k < N; k++)

for(int i =0; i < N; i++)

for(int j =0; j < N; j++)

if(d\[i\]\[j\] > d\[i\]\[k\] + d\[k\]\[j\]) d\[i\]\[j\] = d\[i\]\[k\] + d\[k\]\[j\];

// 그래프에서 정점을 삭제

void deleteVertex(int index)

 while(AdjList