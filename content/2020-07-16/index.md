---
title: "Kadane’s algorithm - 연속된 부분배열의 합이 최대가 되는 경우를 찾기"
date: "2020-07-16"
category: "tech"
cover: https://graphql.org/
tags:
    - Algorithm
    - Swift
---

짝꿍분과 어제 같이 보았던 문제. https://leetcode.com/problems/best-time-to-buy-and-sell-stock/

놀랍게도 O(n)으로 풀 수 있다고 한다. Kadane’s algorithm 이라는 이름으로 알려져 있다. 아이디어는 점화식 느낌인데...

## 1. k번째 요소로 끝나는 부분배열만 놓고 정답을 생각해본다.

k-1번째 요소까지만 놓고 같은 문제를 풀었을 때의 정답을 알고 있다면, 여기에 k번째 요소를 포함시켜서 문제를 푸는 것은 아주 쉽다.

아래 둘 중에 큰 놈을 고르면 된다.

- k번째 요소 하나만 포함하는 배열이거나
- (k > 1 이라면) k-1번째 요소까지만 봤을 때의 정답 + k번째 요소

## 2. 모든 문제를 풀었을 때의 정답

1번째 요소로 끝나는 배열에 대한 정답, 2번째 요소로 끝나는 배열에 대한 정답, ... n번째 요소로 끝나는 배열에 대한 정답 중 가장 큰 놈을 고르면, (다른 가능성이 없으므로) 이 놈이 전체 문제의 정답이 된다.

이걸 코드로 옮기면...

```swift
func solution(_ input: [Int]) -> Int {
    assert(input.count > 0)
    var bestEndingAtK = 0
    var bestOverall = 0
    for k in 0..<input.count {
        bestEndingAtK = max(input[k], bestEndingAtK + input[k])
        bestOverall = max(bestOverall, bestEndingAtK)
    }
    return bestOverall
}

print(solution([-2, -3, 4, -1, -2, 1, 5, -3]))
// 7
```

애매한 부분은 초기값인데, 저걸 0으로 두면 배열의 모든 요소가 음수일 때 틀린 답(0)이 출력되고, `Int.min`으로 두면 `bestEndingAtK + input[k]`을 계산할 때 문제가 생긴다. (`Int.min`보다 더 내려갈 수 있으므로)

이걸 해결하려면 초기값을 배열의 첫 요소로 두면 된다. (https://www.geeksforgeeks.org/largest-sum-contiguous-subarray/)

```swift
func solution(_ input: [Int]) -> Int {
    assert(input.count > 0)
    var bestEndingAtK = input[0] // 여기가 달라짐
    var bestOverall = input[0] // 여기가 달라짐
    for k in 1..<input.count {
        bestEndingAtK = max(input[k], bestEndingAtK + input[k])
        bestOverall = max(bestOverall, bestEndingAtK)
    }
    return bestOverall
}

print(solution([-2, -3, -5, -1, -2, -7, -4, -3]))
// -1
```

여기에 조금만 더 추가하면 정답인 부분배열의 위치까지 알아낼 수 있지만 졸려서 오늘은 이만.