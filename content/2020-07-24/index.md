---
title: "UIView.isUserInteractionEnabled"
date: "2020-07-24"
category: "tech"
cover: https://graphql.org/
tags:
    - Swift
    - UIKit
---

UI 요소의 인터랙션을 임시로 (혹은 영원히) 막고 싶을 때, `isUserInteractionEnabled` 속성에 `false`를 대입해주면 된다. 모든 서브뷰가 이 속성에 영향을 받는다.


```swift
import UIKit

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        let textField = UITextField()
        textField.frame = CGRect(x: 10, y: 100, width: 100, height: 30)
        textField.text = "Hello by UITextField"
        view.addSubview(textField)
        // 아래 주석을 풀면, 텍스트 필드를 터치해도 아무 반응이 없다.
        // textField.isUserInteractionEnabled = false
        
        let button = UIButton()
        button.frame = CGRect(x: 10, y: 300, width: 100, height: 30)

        button.setTitle("Hello", for: .normal)
        button.setTitleColor(.black, for: .normal)
        button.addTarget(self, action: #selector(tapButton), for: .touchUpInside)
        view.addSubview(button)
        // 아래 주석을 풀면, 버튼을 터치해도 아무 반응이 없다.
        // button.isUserInteractionEnabled = false
        
        // 아래 주석을 풀면, view의 모든 subview에 대한 인터랙션이 막힌다.
        // view.isUserInteractionEnabled = false
    }
    
    @objc func tapButton() {
        print("button tapped!")
    }
}
```