---
title: "Retroactive Modeling"
date: "2020-03-08"
category: "tech"
cover: https://graphql.org/
tags:
    - Swift
    - Rust
---

객체지향을 설명하는 책이나 글을 보면 'composition over inheritance', 즉 상속보다 합성을 사용하라는 격언(?)이 종종 등장한다. 상속은 유연함이 떨어지고, 기능 재사용이 어렵다는 이유에서인데, 이는 널리 사용되는 OOP 언어(C#, Java 등)에서 제공하는 상속 기능이 가지는 한계점에 기인한다.

- (대개는) 다중 상속 지원을 하지 않는다.
- 자식 클래스 선언부에서만 상속할 수 있다. (예를 들어, 내가 만든 클래스 A를, 외부 라이브러리에 들어있는 클래스 B가 상속받게 만들 수 없음)

delegation 패턴이나 adapter 패턴 등을 적용해 이 문제들을 완화할 수는 있지만, (상속 대비) 코드가 복잡해지고 (dynamic dispatch로 인해) 성능이 저하되는 문제가 발생할 수 있다.

이를 근본적으로 해결하기 위해, 상대적으로 최근에 만들어진 Swift, Rust에는 '타입 정의가 끝난 이후에도 타입을 확장할 수 있는' 기능이 포함되어있다. 이를 통해 retroactive(소급적용되는) modeling이 가능해진다.

예를 들어, 외부 라이브러리에 정의되어있는 Circle 타입에 area 속성을 추가하고 싶다면, 아래와 같이 하면 된다. (여기에서, `protocol`과 `trait`는 다른 언어에서 interface 라고 부르는 것과 비슷한 것이다.)

```swift
// Swift

// Circle 타입을 갖는 외부 라이브러리를 불러오기
import ExternalLibrary

// 아래 타입을 사용할 수 있다.
// struct Circle {
//     let radius: Double
// }

// area 속성을 가지는 protocol을 선언
protocol ShapeArea {
  var area: Double { get }
}

// 위 protocol을 적용해 Circle 타입을 확장
extension Circle: ShapeArea {
  var area: Double {
    return self.radius * self.radius * Double.pi
  }
}

print("area:", Circle(radius: 1.0).area)
// area: 3.141592653589793

// ShapeArea를 변수의 타입으로 지정하는 것도 가능
let shape: ShapeArea = Circle(radius: 1.0)
```

```rust
// Rust

// Circle 타입을 갖는 외부 라이브러리를 불러오기
use external_library::Circle;

// 아래 타입을 사용할 수 있다.
// struct Circle {
//     radius: f64,
// }

// area 메소드를 가지는 trait을 선언
trait ShapeArea {
    fn area(self) -> f64;
}

// 위 trait을 적용해 Circle 타입을 확장
impl ShapeArea for Circle {
    fn area(self) -> f64 {
        return self.radius * self.radius * std::f64::consts::PI;
    }
}

fn main() {
    let circle = Circle { radius: 1.0 };
    println!("area: {}", circle.area());
    // area: 3.141592653589793

    // ShapeArea를 변수의 타입으로 지정하는 것도 가능
    let shape: &dyn ShapeArea = &Circle { radius: 1.0 };
}
```

위와 같이, 외부 라이브러리에 있는 타입도 얼마든지 확장할 수 있기 때문에 adapter 패턴을 적용할 필요가 없다.
또한 위 두 언어는 확장 횟수에 제한을 두지 않는다. 따라서 (말하자면) 다중 상속을 할 수 있기 때문에, 상속 기능을 대체하기 위해 delegation 패턴을 적용할 필요가 없어진다.
