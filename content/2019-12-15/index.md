---
title: "TypeScript - 제네릭 타입 인수의 추론"
date: "2019-12-15"
category: "tech"
cover: https://typescriptlang.org
tags:
    - TypeScript
    - Lookup Type

---

# Type Argument Inference

제네릭은 타입 추론에서 빠질 수 없는 중요한 기능이다. 특히 여러 타입에 대응되는 함수를 만들 때, 일일이 타입을 지정해주지 않고도 인수로 들어온 값의 타입이 제네릭 타입으로 사용된다.

```typescript
// https://www.typescriptlang.org/docs/handbook/generics.html 에서 가져옴

function identity<T>(arg: T): T {
    return arg;
}

// 타입 파라미터를 명시적으로 지정 - output 변수의 타입은 string
let output = identity<string>("myString");

// 인수의 타입이 제네릭 타입으로 사용됨 - 위와 동일하게 output 변수의 타입은 string
let output = identity("myString");
```

# 값 인수를 통해 추론할 수 없는 타입 인수

함수의 인수와는 상관없이, 별개의 타입 인수를 받는 제네릭 함수를 만들 수도 있다.

하지만 이 때에는 값 인수를 통해 추론을 할 수 없으므로, 명시적으로 타입 인수를 지정해주어야 한다.

```typescript
function requestState<Data, Error>() {
  return {
    data: null as Data | null,
    error: null as Error | null
  }
}

// 명시적으로 타입 인수를 넘겨줌
const state = requestState<number, string>()
```

# 추론할 수 있는 타입 인수와, 추론할 수 없는 타입 인수를 함께 쓸 때의 어려움

아래는 값 인수를 통해 추론할 수 있는 타입 인수와, 추론할 수 없는 타입 인수를 같이 쓰는 예제이다.

TypeScript에서는 명시적으로 타입 인수를 넘겨줄 때 (default type argument가 지정되어있지 않는 한) 

1. 모든 타입 파라미터에 대해 타입 인수를 넘겨준다.
2. 아예 넘겨주지 않는다.

와 같은 두 가지 경우만 허용한다. 예를 들어, 아래와 같은 코드는 허용하지 않는다.

```typescript
function requestState<Data, Error, Name extends string>(name: Name) {
  return {
    name,
    data: null as Data | null,
    error: null as Error | null
  }
}

// 타입 파라미터는 세 개지만, 두 개의 타입 인수만을 넘겨주었다. 이런 경우를 허용하지 않는다.
const state = requestState<number, string>('myState')

// 에러: Expected 3 type arguments, but got 2.
```

타입 추론을 할 수 있는 상황임에도 불구하고, 굳이 아래와 같이 모든 타입 인수를 넘겨주어야 한다.

```typescript
// 세 번째 타입 인수를 넘겨주었다. 코드 중복이 발생했다.
const state = requestState<number, string, 'myState'>('myState')
```

# 해결책 - 또다른 제네릭 클래스를 활용하기

해결책은 어쨌든 값을 통한 추론을 활용하는 것이다. 제네릭 클래스를 만들어서, 이 클래스의 인스턴스를 타입 추론에 활용해볼 수 있다.

```typescript
// 타입 파라미터 외에는 아무 기능도 없는 클래스를 만든다.
class Either<Data, Error> {}

// 위 클래스의 인스턴스를 값으로 받는다. 이렇게 되면 값 인수를 통한 타입 인수의 추론이 가능해진다.
function requestState<Data, Error, Name extends string>(name: Name, either: Either<Data, Error>) {
  return {
    name,
    data: null as Data | null,
    error: null as Error | null
  }
}

// 'myState'에 대한 코드 중복이 없어졌다.
const state = requestState('myState', new Either<number, string>())
```

위 코드의 `Either` 클래스 내부에서 `Data`, `Error` 타입 파라미터를 사용하지 않았기 때문에 `All type parameters are unused` 경고가 뜬다. 이를 해결하기 위해 `// @ts-ignore`를 써줘도 되지만, 조금 더 우아한 방법이 있다.

아이디어는 non-null assertion을 적용한 속성을 사용하는 것이다. 이렇게 하면 `requestState` 함수가 끝나고 나서도 `Data`, `Error`에 적용된 타입 인수를 가져오는 것이 가능해진다.

```typescript
class Either<Data, Error> {
  // non-null assertion (!) 을 적용하지 않으면 컴파일 에러가 난다.
  // 에러: Property 'DataType' has no initializer and is not definitely assigned in the constructor.
  DataType!: Data
  ErrorType!: Error
}

function requestState<Data, Error, Name extends string>(name: Name, either: Either<Data, Error>) {
  return {
    name,
    data: null as Data | null,
    error: null as Error | null,
    // 테스트를 위해 반환되는 객체에 either를 추가해본다.
    either
  }
}

const state = requestState('myState', new Either<number, string>())

// lookup type - 결과적으로 MyType은 number가 된다.
type MyType = (typeof state.either)['DataType']
```
