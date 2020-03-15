---
title: "Delegation pattern"
date: "2020-03-15"
category: "tech"
cover: https://graphql.org/
tags:
    - Swift
    - Design Pattern
---

객체지향을 설명하는 책이나 글을 보면 'composition over inheritance', 상속보다 합성을 사용하라는 격언(?)이 종종 등장한다. 상속은 유연함이 떨어지고, 기능 재사용이 어렵다는 이유에서인데, 이는 널리 사용되는 OOP 언어(C#, Java 등)에서 제공하는 상속 기능이 다중 상속 지원을 하지 않는다는 한계점에 기인한다.

Delegation 패턴을 적용하면, 컴파일 타임에 결정되는 정적인 상속 관계 대신, 런타임에 객체를 합성함으로써 동적인 관계를 만들어 위와 같은 한계점을 뛰어넘을 수 있다.

예를 들어, '가게 주인(StoreOwner)은 손님에게 인사를 해야 한다'는 요구사항이 있다고 해보자. '인사를 한다'는 기능을 가게 주인의 메소드로 만들 수도 있겠지만, '인사를 한다'는 기능은 어디서나 많이 필요한 기능이기 때문에 재사용성을 높이기 위해 별도로 작성하는 것이 좋을 것이다.

이를 위해 `Greeter` 클래스를 만들어서 `StoreOwner`가 상속을 받게 만들어볼 수 있다.

```swift
class Greeter {
    func greet() {
        print("Hello!")
    }
}

class StoreOwner: Greeter {
    func guestDidCome() {
        greet()
    }
}
```

하지만 `StoreOwner`는 인사뿐만 아니라 다른 여러가지 책임도 지고 있다. (장보기, 회계, 청소, ...) Swift에서는 다중 상속을 허용하지 않기 때문에, 이렇게 많은 클래스들을 상속받는 것은 불가능하다.

```swift
class StoreOwner: Greeter, Supplier, Accountant, Cleaner {
    // Error! 다중 상속이 허용되지 않는다.
}
```

이 때, 가게 주인이 직접 인사를 하는 것이 아니라, 인사를 할 수 있는 사람(Greetable)을 데려와서 인사를 시키면 어떨까?

```swift
// '인사를 할 수 있다'를 protocol로 만든다.
protocol Greetable {
    func greet()
}

// 위 protocol을 따르는 인사꾼 클래스를 만든다.
class Greeter: Greetable {
    func greet() {
        print("Hello!")
    }
}

class StoreOwner {
    let greeter: Greetable

    // 가게 주인 인스턴스를 만들 때, 인사꾼을 인수로 받는다.
    init(greeter: Greetable) {
        self.greeter = greeter
    }

    func guestDidCome() {
        greeter.greet()
    }
}

let storeOwner = StoreOwner(greeter: Greeter())
storeOwner.guestDidCome() // Hello!

```

비슷하게, '재료공급원', '회계사', '청소부'를 고용해서 그들에게 업무를 위임(delegate)할 수 있다.

```swift
class StoreOwner {
    // ...

    let supplier: Suppliable
    let accountant: Accountable
    let cleaner: Cleanable

    func beforeOpening() {
        supplier.supply()
    }

    func afterClosing() {
        accountant.settleAccount()
        cleaner.clean()
    }
}
```

위와 같이 Delegation 패턴을 잘 활용하면, 특정 타입에게 요구되는 많은 역할과 책임을 나누어 작성할 수 있고, 또 나뉘어진 각각의 부분을 같은 기능이 요구되는 다른 부분에 쉽게 적용할 수 있다.

---

다만 delgation 패턴은 상속 대비 아래와 같은 단점을 지닌다.

- (상속 대비) 코드가 복잡해진다.
- (dynamic dispatch로 인해) 성능이 저하되는 문제가 발생할 수 있다.

지난 글에서 Swift의 extension 기능을 활용하면 위와 같은 문제를 해결할 수 있다고 했다. 하지만 extension 역시 delegation 패턴 대비 아래와 같은 단점을 가지고 있다.

- Swift extension으로는 타입에 stored property를 추가할 수 없다. ([몇몇 workaround](https://medium.com/@valv0/computed-properties-and-extensions-a-pure-swift-approach-64733768112c)가 있긴 하다) Delegation 패턴이라면 delegate 객체에 얼마든지 stored property를 추가할 수 있다.
- 런타임에 extension을 갈아끼울 수 없다. Delegation 패턴이라면 얼마든지 런타임에 delegate 객체를 교체하는 것이 가능하다.

따라서 Delegation 패턴이나 extension 중 어느 한 쪽이 항상 좋다고 말할 수는 없으며, 기술적 요구사항을 정확히 파악한 후 적절한 방식을 선택하는 것이 중요하다.