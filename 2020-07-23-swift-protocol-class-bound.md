# A protocol can be bounded to a specific class

특정 클래스를 상속받은 클래스만이 프로토콜을 따를 수 있도록, 프로토콜에 class bound를 지정할 수 있다.

```swift
class Person {}

protocol Programming: Person {}

class Programmer: Person, Programming {}

// 컴파일 에러 - 'Programming' requires that 'Dog' inherit from 'Person'
// 강아지는 프로그래밍을 못 한다..
class Dog: Programming {}

let someone: Programming = Programmer()
// Programming을 따르려면 Person의 서브클래스여야 한다는 사실을 인식해서, 에러가 나지 않는다!
let person: Person = someone
```