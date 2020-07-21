---
title: "Cannot use mutating member on immutable value: 'self' is immutable"
date: "2020-07-22"
category: "tech"
cover: https://graphql.org/
tags:
    - Swift
---

오늘 예상치 못한 에러를 만났다.

```swift
protocol MyProtocol {
    var prop: Int { get set }
}

extension MyProtocol {
    mutating func updateSomething() {
        prop += 1
    }
}

class MyClass: MyProtocol {
    var prop: Int = 0
    
    private func haha() {
        updateSomething() // 컴파일 에러 - Cannot use mutating member on immutable value: 'self' is immutable
    }
}
```

저런 에러가 나는 이유는 MyClass가 참조 타입이기 때문! (참조는 immutable 하므로...)

이 에러를 해결하려면, `mutating`을 떼고 MyProtocol을 class only protocol로 만들어주면 된다.

```swift
protocol MyProtocol: class { // class only protocol
    var prop: Int { get set }
}

extension MyProtocol {
    func updateSomething() { // mutating 제거 (값 자체가 변경되는 것이 아님)
        prop += 1
    }
}

class MyClass: MyProtocol {
    var prop: Int = 0
    
    private func haha() {
        updateSomething() // 이제 컴파일이 잘 된다.
    }
}
```

## 참고글

- https://www.bignerdranch.com/blog/protocol-oriented-problems-and-the-immutable-self-error/

