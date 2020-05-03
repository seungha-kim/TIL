---
title: "Core Animation, Core Graphics"
date: "2020-05-03"
category: "tech"
cover: https://graphql.org/
tags:
    - iOS
---

다니는 회사에서 팀을 옮기게 됐다. ProtoPie Studio로 만든 프로토타입을 web, iOS, Android 위에서 재생해주는 플레이어 부분을 개발하게 됐다. 플레이어가 하는 일은

- 입력된 모델 파일과 미디어 리소스를 기반으로 플랫폼별 API를 사용해서 의도에 맞게 UI(회사에서는 object라고 부른다)를 출력하기
- 특정 이벤트에 (trigger라고 부른다) 미리 설정된 인터랙션(response라고 부른다)을 보여주기

UI 애니메이션이 부드러워야 하니 성능에도 신경을 써야 하고, 변수와 표현식같은 개념도 있어서 (간단하지만) 자체 언어를 위한 인터프리터 같은 것도 만들어져 있다. 거기다가 지원해야 할 플랫폼이 여러 개라 플랫폼별 특성도 파악해야 하고... 배워야 할게 산더미 같다!

팀장님(?)한테 과제를 달라고 말씀드렸더니 여러 플랫폼에서 직접 바닥부터 구현해보라는 미션을 주셔서, 일단 iOS에서 어떻게 구현할 수 있을지 조사 중이다.

일단은 화면을 그리는 방법부터. ProtoPie에서 프로토타입을 만들 때는 iOS에서 기본적으로 제공하는 UI 컨트롤을 쓰지 않기 때문에, iOS UI를 위한 UIKit으로 화면을 그릴 일은 거의 없을 것으로 보인다.

더 low level에서 2D 뭐시기를 그리는 API에는 Core Animation, Core Graphics 두 가지가 있는데, 역할이 조금씩 다르다.

Core Graphics는 웹의 Canvas 2D API와 비슷하게 2D 그리기를 위한 API이다. 벡터 그리기를 하거나, 비트맵을 표시하거나, 텍스트를 그리는 등의 작업을 할 수 있다. 이 작업들은 주로 CPU 자원을 소모한다고 한다.

반면에 Core Animation은 이미 만들어진 비트맵을 가지고 변환(transform)을 하거나, 변환을 이용한 애니메이션을 보여줘야 할 때 필요한 기능들을 제공한다. 예를 들어 Core Graphics를 가지고 그린 비트맵을 가지고 Core Animation으로 평행이동, 회전, 크기 변경 등을 빠른 속도로 할 수 있다. 2D 비트맵을 3D 공간 안에서 다룬다고 하며, 이런 작업을 주로 GPU를 이용해 처리한다고 한다. 일반적인 그래픽 툴과 유사하게 '레이어'라는 추상화를 해놓아서 이해하기 쉬웠다. 이외에도 UI를 그릴때 필수적인 몇몇 기능(border, border radius, shadow 등)을 자체적으로 가지고 있다.

정리하자면,

- Core Graphics: 일반적인 2D 그리기를 위한 API. CPU를 많이 사용
- Core Animation: 비트맵에 대한 변환, 애니메이션, UI 그리기를 위한 편의 기능을 제공하는 API. GPU를 많이 사용
- UIKit: iOS UI 구현을 위한 프레임워크. 내부적으로 Core Animation, Core Graphics를 사용한다.