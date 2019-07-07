---
title: "7월 첫째 주 배운 것들"
date: "2019-07-07"
category: "tech"
cover: https://developer.apple.com/swift/
tags:
    - swift
---

[Data Structure & Algorithms in Swift](https://www.raywenderlich.com/977854-data-structures-algorithms-in-swift)를 읽고 배운 것을 정리. 아래 나오는 코드도 이 책에서 따왔다.

## AVL Tree vs Red-Black Tree

BST(Binary Search Tree, 이진 탐색 트리)는 대소관계를 가지는 값을 저장하고 검색하는 데에 최적화된 자료구조로, 삽입과 검색의 시간 복잡도가 O(logN)으로 우수하다. 단, 노드가 균형있게 좌우로 분산되지 않고 한 쪽으로 몰리게 되는 최악의 경우에는 O(N)이 되어버리고 만다.

AVL Tree와 Red-Black Tree 모두 이런 문제를 해결하는 자료구조다. 노드를 삽입/삭제할 때 매번 트리의 균형을 잡아준다. 흥미로웠던 것은 AVL Tree와 Red-Black Tree의 성능 차이인데, Red-Black Tree가 일반적으로 삽입/삭제 성능이 더 좋고, 그 이유는 **트리의 균형을 적당히만 잡기 때문이다.** AVL Tree는 최대 깊이와 최소 깊이가 1 이상 차이나는 경우를 허용하지 않는 반면, Red-Black Tree는 최대 깊이가 최소 깊이의 두 배까지 커지는 것을 허용한다. 그로 인해 트리의 균형을 잡기 위해 드는 비용이 AVL Tree에 비해 작다. 그렇다고 해서 삽입/삭제가 크게 느리지 않고 여전히 시간 복잡도는 로그 단위이다.

여러 알고리즘이 무엇을 포기하고 무엇을 얻었는지를 비교해볼 수 있어 흥미로웠다.

## 반복문을 재귀 함수로 바꾸기

아래는 BFS 알고리즘을 수행하는 Swift 코드이다.

```swift
public func breadthFirstSearch(from source: Vertex<Element>) -> [Vertex<Element>] {
    // 이 변수들이 루프 내에서 사용됨
    var queue = QueueDoubleStack<Vertex<Element>>()
    var enqueued = Set<Vertex<Element>>()
    var visited = [Vertex<Element>]()

    queue.enqueue(source)
    enqueued.insert(source)
    while let vertex = queue.dequeue() {
        visited.append(vertex)
        for edge in edges(from: vertex) {
            if !enqueued.contains(edge.destination) {
                queue.enqueue(edge.destination)
                enqueued.insert(edge.destination)
            }
        }
    }
    return visited
}
```

이렇게 바깥 변수에 상태를 저장하는 반복문을 재귀함수로 만들 때는, **이 변수들을 재귀 함수의 매개변수로 만드는 것이 일반적이다.**

단, 여기서 주의할 점은 Swift 언어의 value type의 동작 방식이다. Swift의 `inout` 매개변수를 사용해서 value type의 변수를 call by reference 넘길 수 있다. (만약 `inout` 매개변수가 아닌 보통의 매개변수를 사용한다면 call by value 방식으로 전달된다. 즉, 값이 복사된다.)

```swift
public func bfs(from source: Vertex<Element>) -> [Vertex<Element>] {
    var queue = QueueDoubleStack<Vertex<Element>>()
    var enqueued = Set<Vertex<Element>>()
    var visited = [Vertex<Element>]()
    queue.enqueue(source)
    enqueued.insert(source)

    // 세 변수의 참조를 call by reference 방식으로 넘겨주었다.
    bfs(queue: &queue, enqueued: &enqueued, visited: &visited)
    return visited
}

// 반복문에서 상태를 저장하기 위해 쓰이는 변수들을 재귀 함수의 매개변수로 바꿔주었다.
// 단, 꼭 inout 매개변수를 사용해야 한다. 아니면, 참조가 넘어가는 것이 아니라 값이 복사된다.
private func bfs(queue: inout QueueDoubleStack<Vertex<Element>>,
                  enqueued: inout Set<Vertex<Element>>,
                  visited: inout [Vertex<Element>]) {
    guard let vertex = queue.dequeue() else {
        return
    }
    // 나머지는 동일
    visited.append(vertex)
    for edge in edges(from: vertex) {
        if !enqueued.contains(edge.destination) {
            queue.enqueue(edge.destination)
            enqueued.insert(edge.destination)
        }
    }
    bfs(queue: &queue, enqueued: &enqueued, visited: &visited)
}
```

## Dijkstra 알고리즘

Dijkstra 알고리즘은 그래프의 두 정점 간 최단경로를 찾는 알고리즘이다. 이 알고리즘은 '최단경로'라는 것의 굉장히 중요한 성질을 활용하는데,

> A 부터 B 까지 가는 최단경로 상에 C가 있다면, 이 경로의 A 부터 C 까지 가는 부분경로가 바로 A 부터 C 까지의 최단경로이다.

가 그것이다. 조금만 생각해보면 증명할 수 있다. (만약 A 부터 C 까지의 다른 최단경로가 있다면, 그 쪽 경로를 통해서 A 부터 B 까지 더 짧은 경로가 존재한다는 모순에 이른다.)

Dijkstra 알고리즘은 이 성질을 바탕으로 **꼭 필요한 정보만을 기억한다.**

- 출발점부터 임의의 정점까지의 **최단거리**와, 그 최단거리에 도달하기 위해 거쳐온 **직전 정점**을 기억한다.

이를 바탕으로 최단경로를 구해낼 수 있다. 예를 들어, 알고리즘을 성공적으로 수행해서 아래 사실을 알아냈다고 하자.

1. A 부터 B 까지는 최단거리가 6이고, 직전에 E를 거쳐왔다.
2. A 부터 C 까지는 최단거리가 4이고, 직전에 G를 거쳐왔다.
3. A 부터 D 까지는 최단거리가 7이고, 직전에 E를 거쳐왔다.
4. A 부터 E 까지는 최단거리가 5이고, 직전에 C를 거쳐왔다.
5. A 부터 F 까지는 최단거리가 9이고, 직전에 A를 거쳐왔다.
6. A 부터 G 까지는 최단거리가 1이고, 직전에 A를 거쳐왔다.

위 사실로부터 A 부터 D 까지 가는 최단경로를 구해낼 수 있다.

- A에서 출발해서 최단경로를 통해 D에 가려면, E를 거쳐야 한다.
- A에서 출발해서 최단경로를 통해 E에 가려면, C를 거쳐야 한다.
- A에서 출발해서 최단경로를 통해 C에 가려면, G를 거쳐야 한다.
- A에서 출발해서 최단경로를 통해 G에 가려면, A를 거쳐야 한다.

즉, 최단경로는 `A -> G -> C -> E -> D` 이다.

어떤 성질이 성립한다는 것이 보장되어 있다면, 그 성질을 활용해서 최소한의 계산/기억만으로 답을 구해낼 수 있는 점이 흥미로웠다.