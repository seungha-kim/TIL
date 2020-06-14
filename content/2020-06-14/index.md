---
title: "UIViewController 라이프사이클"
date: "2020-06-14"
category: "tech"
cover: https://graphql.org/
tags:
    - Swift
    - UIKit
---

# loadView()

뷰 컨트롤러 인스턴스의 view 속성에 저장할 UIView 인스턴스를 만들어야 하는 시점에 호출된다. 기본 구현은 연결된 nib 파일 또는 스토리보드에서 연결된 뷰를 불러온다. 뷰를 직접 만들고 싶은 경우, 이 메소드를 오버라이드해서 view 속성에 직접 만든 뷰를 넣어주어야 한다.

# viewDidLoad()

위 loadView의 실행이 끝난 시점에 호출된다. 이 시점에서는 뷰 컨트롤러의 상태가 올바르게 설정되어 있다고 생각해도 무방하다. (e.g. outlet 등이 주입되어 있다.)

내장 데이터베이스, 혹은 네트워크를 통해 데이터를 불러오는 작업을 시작하기에 적합한 메소드이다.

# viewWillAppear(_ animated: Bool)

뷰 컨트롤러의 view가 화면에 표시되기 직전에 호출된다.

# viewDidAppear(_ animated: Bool)

뷰 컨트롤러의 view가 화면에 표시된 직후에 호출된다.

# viewWillDisappear(_ animated: Bool)

뷰 컨트롤러의 view가 화면에서 사라지기 직전에 호출된다.

# viewDidDisappear(_ animated: Bool)

뷰 컨트롤러의 view가 화면에서 사라진 직후에 호출된다.

# didReceiveMemoryWarning()

기기의 메모리가 충분하지 않을 때 호출된다. iOS는 기기의 메모리 자원이 부족할 때 백그라운드 앱을 종료시키므로, 이 메소드를 신경써서 구현하는 것은 백그라운드 종료 현상을 줄이는데 도움이 될 수 있다.

# 호출 순서

<img src="./g19fw.png" />

이미지 출처: https://stackoverflow.com/questions/5562938/looking-to-understand-the-ios-uiviewcontroller-lifecycle