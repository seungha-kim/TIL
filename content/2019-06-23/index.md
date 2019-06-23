---
title: "재귀 함수를 반복문으로 바꾸는 간단한(?) 방법"
date: "2019-06-23"
category: "tech"
cover: https://developer.apple.com/swift/
tags:
    - swift
    - recursion
---

재귀 함수는 반복문보다 알고리즘을 간결하게 표현할 수 있다는 장점을 가지고 있지만, 구동환경에 따라 재귀 호출 횟수에 제약이 있는 경우가 있기 때문에 재귀 함수를 반복문으로 변환하는 작업을 해야 할 때가 있다. 컴파일러단에서 tail call optimization 등의 최적화를 해주기도 하지만, 자동 최적화를 할 수 없는 경우 사람이 직접 반복문으로 바꿔줘야 한다. 이런 작업을 해야할 때 빠르고 더럽게(?) 반복문을 작성할 수 있는 기법이 떠올라 기록해둔다. (우연히 떠올라서 혹시나 하고 찾아봤는데 역시나 널리 쓰이는 기법이었다..)

아이디어는 호출 스택을 그대로 흉내내는 것이다. 호출 스택은 아래와 같은 변화를 거친다

- 함수를 호출할 때, 호출 스택에 **매개변수와 지역변수**가 push 된다.
- 함수가 종료될 때, 호출 스택에서 pop이 일어나고 **반환값**이 **이전 위치**로 전달된다.

보통 이런 과정은 구동 환경에 의해 자동으로 관리된다. 여기에서는 이 과정을 직접 구현해볼 것이다.

아래는 Swift 언어로 작성된 이진 탐색 트리의 insert 메소드이다.

```swift
private func insert(from node: BinaryNode<Element>?, value: Element) -> BinaryNode<Element> {
    // 이 순간을 initial이라 하자.
    // 매개변수: node, value
    guard let node = node else {
        return BinaryNode(value: value)
    }

    if value < node.value {
        // 여기서 재귀 호출이 한 번 (insert가 호출된 순간을 leftComplete라 하자.)
        node.leftChild = insert(from: node.leftChild, value: value)
    } else {
        // 여기서 또 한번 (insert가 호출된 순간을 rightComplete라 하자.)
        node.rightChild = insert(from: node.rightChild, value: value)
    }
    return node
}
```

이 재귀 함수를 반복문을 사용하는 함수로 바꿔본다. 호출 스택을 흉내내기 위해서 

- pop이 일어났을 때 어디로 돌아가야할지를 나타내는 `ReturnAddr`
- 매개 변수와 지역 변수를 담을 구조체 `StackFrame`

를 만든다.

```swift
enum ReturnAddr {
    case initial
    case leftComplete
    case rightComplete
}

struct StackFrame<Element> {
    let node: BinaryNode<Element>?
    var addr: ReturnAddr
}
```

위 `insert` 함수의 `value` 매개변수는 재귀 호출 과정 내내 같은 값이 사용되므로, 굳이 StackFrame에 넣지 않아도 된다. 이제 이를 이용해서 반복문을 짤 수 있다.

```swift
private func insertLoop(from root: BinaryNode<Element>?, value: Element) -> BinaryNode<Element> {
    // 호출 스택을 흉내내는 배열
    var stack = [StackFrame<Element>]()

    // 반환값이 저장될 변수
    var returned: BinaryNode<Element>?

    // 첫 실행을 나타내는 프레임을 push 한다. 이 때 initial 위치부터 실행되게 한다.
    stack.append(StackFrame(node: root, addr: .initial))
    while let popped = stack.popLast() {
        switch popped.addr {
        // 실행과정의 시작
        case .initial:
            guard let node = popped.node else {
                // 현재 실행과정의 반환값을 다음 실행과정에서 사용할 수 있도록 변수에 저장
                returned = BinaryNode(value: value)
                continue
            }
            if value < node.value {
                // 현재 실행과정이 leftComplete 위치에서 재개되어야 함을 스택에 등록
                stack.append(StackFrame(node: node, addr: .leftComplete))

                // 다음 실행과정을 스택에 등록
                stack.append(StackFrame(node: node.leftChild, addr: .initial))
            } else {
                // 현재 실행과정이 rightComplete 위치에서 재개되어야 함을 스택에 등록
                stack.append(StackFrame(node: node, addr: .rightComplete))

                // 다음 실행과정을 스택에 등록
                stack.append(StackFrame(node: node.rightChild, addr: .initial))
            }
        // 실행과정이 leftComplete 위치에서 재개되어야 하는 경우
        case .leftComplete:
            popped.node?.leftChild = returned
            returned = popped.node
        // 실행과정이 rightComplete 위치에서 재개되어야 하는 경우
        case .rightComplete:
            popped.node?.rightChild = returned
            returned = popped.node
        }

    }
    return returned!
}
```

다만, 위처럼 시간 복잡도가 logN으로 떨어지는 경우 딱히 재귀 호출 횟수때문에 문제가 생길 일은 없으므로 그냥 재귀 함수를 사용해도 무방하긴 하다.