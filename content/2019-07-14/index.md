---
title: "그리디 알고리즘"
date: "2019-07-14"
category: "tech"
cover: https://developer.apple.com/swift/
tags:
    - swift
    - algorithm
---

많은 수의 알고리즘이 어떤 자료구조를 특정한 순서대로 방문하면서 순차적인 계산을 하는 방식으로 답을 구해낸다. 이 때 **계산 순서**를 결정하기가 어려운 경우가 있는데, 배열같은 선형 자료구조는 별로 고민할 거리가 없지만, 그래프같이 선형이 아닌 자료구조를 다룰 때에는 계산 순서를 결정하기가 쉽지 않다.

이 때 가장 쉽게 시도해볼 수 있는 방법은 **이제까지 알아낸 사실만 가지고 최적으로 보이는 길을 선택하는** 것이다. 이런 방식을 그리디(greedy) 알고리즘이라 부른다.

당연히 부분 최적해가 전체 최적해일 것이란 보장은 없다. 하지만 전체 최적해를 구하는데 비용이 너무 많이 들고, 부분 최적해만으로도 충분히 쓸만한 경우에는 그리디 알고리즘이 충분히 유용할 수 있다.

또, 부분 최적해를 찾아나가는 과정을 마치면 이것이 바로 전체 최적해가 되는 경우들도 있다. 다익스트라 알고리즘과 프림 알고리즘이 대표적인 예이다.

## 다익스트라 알고리즘

다익스트라 알고리즘은 가중 유향 그래프의 특정 정점으로부터 다른 모든 정점까지의 최단 경로를 구해내는 알고리즘이다. 계산 과정은 다음과 같다.

1. 시작 정점을 방문한다.
2. **마지막에 방문한** 정점으로부터 뻗어나가는 간선 중에 **가중치가 가장 낮은** 간선을 통해 다음 정점을 방문한다... (세부사항 생략)
3. 더 이상 방문할 정점이 남아있지 않을 때까지 2의 과정을 반복한다.

## 프림 알고리즘

프림 알고리즘은 가중 무향 그래프에서 최소 신장 트리 (Minimum Spanning Tree)를 구해내는 알고리즘이다. 계산 과정은 다음과 같다.

1. 임의의 정점을 방문한다.
1. **이제까지 방문한** 모든 정점으로부터 뻗어나가는 간선 중에 **가중치가 가장 낮은** 간선을 통해 다음 정점을 방문한다... (세부사항 생략)
1. 더 이상 방문할 정점이 남아있지 않을 때까지 2의 과정을 반복한다.

이렇게 위 두 알고리즘은 부분 최적해를 지속적으로 구해나가는 과정을 반복하지만, 결과적으로 전체 최적해에 도달하게 된다는 사실이 증명되어 있다.