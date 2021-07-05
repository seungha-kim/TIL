# DispatchQueue

애플의 Dispatch 프레임워크는 멀티스레딩 구현을 위한 표준적인 방법으로 권장되고 있다. DispatchQueue 클래스의 역할은 웹 브라우저의 작업 큐와 비슷하게, 스레드 관리 및 작업 스케줄링을 알아서 해줌으로써 여러 디테일을 숨겨준다. 반면에 웹 브라우저의 작업 큐와 비교하면

- 직접 프로그래밍이 가능하다
- 단일 스레드가 아니라 여러 개의 스레드를 활용해서 작업을 실행시킨다

는 차이점이 있다.

가령, 스레드를 멈추게 만드는 동기식 HTTP 요청을 해야 한다고 해보자. 이 작업을 UI 스레드에서 실행하면, 요청을 하는 동안데 UI가 반응하지 않을 것이다. 이를 피하기 위해서 HTTP 요청은 UI 스레드가 아닌 별도의 스레드에서 수행해야 한다. 다만, 응답 결과를 가지고 화면을 갱신하는 작업은 UI 스레드에서 해야 한다. (이는 iOS를 비롯한 많은 UI 프레임워크들이 가지고 있는 제약이다.)

```swift
func handleButtonTouch() {
    // 스케줄링 우선순위가 default인 DispatchQueue 위에서 작업을 실행한다.
    // 이 DispatchQueue는 작업을 실행할 때 메인 스레드가 아닌 별도의 스레드를 사용한다.
    DispatchQueue.global(qos: .default).async {
        // 스레드를 블록하는 HTTP 요청을 수행. UI 스레드는 문제없이 작업을 수행할 수 있다.
        let res = someHTTPRequest()
        // DispatchQueue.main 은 UI 스레드에서 작업을 실행한다.
        DispatchQueue.main.async {
            someUI.content = res.body.content
        }
    }
}
```

## serial queue, concurrent queue

DispatchQueue는 serial 방식과, concurrent 방식의 두 가지 모드를 가진다. 예를 들어, 위 코드에서 `DispatchQueue.main`은 serial 방식으로 동작하고, `DispatchQueue.global(...)`은 concurrent 방식으로 동작한다.

serial 방식은 큐에 여러 개의 작업이 쌓여있다고 하더라도, 가장 먼저 들어온 작업이 끝나기 전까지는 큐에 쌓여있는 다른 작업을 실행시키지 않는다. 이 때문에 serial 방식은 스레드 안전이 보장되지 않는 자원에 접근할 때 사용된다. (큐를 사용하지 않으면, 한 번에 하나의 스레드만 공유 자원에 접근할 수 있도록 락을 걸어주는 등의 부가적인 처리가 필요할 것이고, 이는 데드락 등의 문제를 발생시킬 확룔을 높일 것이다.)

concurrent 방식은 현재 실행중인 작업이 끝나기 전에 다른 작업을 실행시킬 수 있다. 코어를 최대한으로 활용할 수 있기 때문에, 계산을 많이 해야 하는 작업에 활용된다.