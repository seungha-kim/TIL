---
title: "Swift의 guard, optional chaning, defer"
date: "2019-06-15"
category: "tech"
cover: https://developer.apple.com/swift/
tags:
    - swift
---

[Data Structures and Algorithms in Swift](https://store.raywenderlich.com/products/data-structures-and-algorithms-in-swift)를 보면서 Swift 공부를 하던 중에 배운 것을 남긴다.

대부분의 현대적인 명령형 프로그래밍 언어에는 `if`, `try/finally`와 같은 제어 구문들이 있고, 이를 바탕으로 로직을 작성할 수 있다. 제어 구문의 활용도는 무궁무진하지만, 제어 구문을 활용해서 프로그래밍을 하다보면 반복해서 나타나는 패턴이 생긴다. Swift는 이런 패턴을 위한 별도의 제어 구문을 갖추고 있으며, 이 구문들을 활용하면 좀 더 간결한 코드를 작성할 수 있다.

## guard

함수를 작성하다 보면 아래와 같이 **특정 작업을 수행하기 위한 조건을 충족하지 못하면, 일찍 반환하는** 패턴이 많이 나타나게 된다.

```swift
func function(arg: Whatever?) -> Whatever? {
  if !someCondition {
    return nil
  }
  if !otherCondition {
    return nil
  }
  // ...나머지 긴 작업
}
```

위 `if` 구문 내부의 코드가 복잡하다면 return을 빠뜨리는 실수를 할 수도 있다. Swift는 이런 패턴을 위해 `guard` 구문을 지원한다.

```swift
func f(arg: Whatever?) -> Whatever? {
  guard someCondition else {
    return nil
  }
  guard otherCondition else {
    return nil
  }
  // ...나머지 긴 작업
}
```

`if` 구문을 사용하는 것과 모양은 같지만, `guard` 구문 내부에서 `return` 혹은 `throw` 같은 구문을 통해 함수를 종료시키지 않으면 Swift 컴파일러가 에러를 뱉는다.

Optional unwrapping을 하기 위해 `guard` 구문을 활용할 수도 있다. 예를 들어, TypeScript에서 nullable인 특정 변수가 null이 아닌 경우를 위한 type guard를 사용하려면 아래와 같이 `if` 구문을 써줘야 한다.

```typescript
function f(arg: Whatever?): Whatever? {
  if (arg !== null) {
    // 여기에서는 arg가 nullable이 아니다.
  }
  // 여기에서는 arg가 여전히 nullable이다.
}
```

arg에 type guard를 적용하기 위해 `if` 구문 내부에 코드를 작성해주어야 해서, 코드의 들여쓰기 깊이가 불필요하게 늘어나는 느낌이다. Swift에서는 이와 비슷한 처리를 `guard` 구문으로 해줄 수 있다.

```swift
func f(arg: Whatever?) -> Whatever? {
  // 여기에서는 arg가 optional이다.
  guard let arg = arg else {
    return nil
  }
  // 여기서부터는 arg가 optional이 아니다!
}
```

이로써, 이후의 코드가 더 간결해진다.

## optional chaning

TypeScript에서 null일지도 모르는 특정 객체의 속성을 반환하는 경우를 생각해보자.

```javascript
function f(arg: Whatever?): Value? {
  if (arg === null) {
    return null
  } else {
    return arg.value
  }
}
```

Swift의 optional chaning 문법을 사용하면, 위와 같은 코드를 줄여쓸 수 있다.

```swift
func f(arg: Whatever?) -> Value? {
  return arg?.value
}
```

위의 `arg?.value` 표현식은 만약 `arg`가 `nil`이면 `nil`로 평가되고, `arg`가 `nil`이 아니면 `arg.value`로 평가된다.

## defer

함수를 작성하다 보면, 어떤 자원을 얻어서 그 자원을 이용한 뒤, 더 이상 이용할 필요가 없다면 뒷처리를 하는 코드를 자주 작성하게 된다. 이런 경우 TypeScript에서는 보통 `try/finally` 구문을 사용한다.

```typescript
function f(arg: Whatever?): Value? {
  const someResource = someManager.acquire()
  const otherResource = otherManager.acquire()
  try {
    // ...자원을 이용해 작업을 수행하고, 값을 반환한다.
    return someValue
  } finally {
    otherResource.release()
    someResource.release()
  }
}
```

이 경우, 자원을 획득하는 코드와 뒷처리를 하는 코드가 떨어져있어서, 여러 자원을 사용해야 하는 경우 일부의 뒷처리를 빠뜨리기가 쉽다. 또한, 뒷처리를 하는 순서를 일일이 신경써주어야 해서 불편하다.

Swift는 이런 패턴을 위해 `defer` 구문을 제공한다.

```swift
func f(arg: Whatever?) -> Value? {
  let someResource = someManager.acquire()
  defer {
    someResource.release()
  }

  let otherResource = otherManager.acquire()
  defer {
    otherResource.release()
  }

  // ...자원을 이용해 작업을 수행하고, 값을 반환한다.
  return someValue
}
```

`defer` 구문 내부의 코드는 바로 실행되는 것이 아니라 `return` 혹은 `throw`등에 의해 함수가 종료되는 시점에 실행된다. 또한 나중에 등장한 `defer` 구문 내부의 코드가 먼저 실행된다. 이를 통해 자원의 획득과 뒷처리에 관련된 코드를 한 곳에 모아둘 수 있고, 또 뒷처리 순서를 일일이 신경써주지 않아도 된다.