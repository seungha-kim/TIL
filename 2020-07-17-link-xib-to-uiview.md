# xib 파일을 UIView에 연결하기

오늘은 xib 파일을 UIView에 연결하는 활동을 해보았따.

# MyView.swift

```swift

import UIKit

class MyView: UIView {
    @IBOutlet private var contentView: UIView!
    @IBOutlet private weak var label: UILabel!
    @IBOutlet private weak var button: UIButton!

    var labelText: String? {
        get {
            label.text
        }
        set {
            label.text = newValue
        }
    }

    var buttonText: String? {
        get {
            button.title(for: .normal)
        }
        set {
            button.setTitle(newValue, for: .normal)
        }
    }

    override init(frame: CGRect) {
        super.init(frame: frame)
        commonInit()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        commonInit()
    }
    
    private func commonInit() {
        Bundle.main.loadNibNamed("MyView", owner: self, options: nil)
        addSubview(contentView)
        contentView.frame = self.bounds
        contentView.translatesAutoresizingMaskIntoConstraints = false
    }
}
```

`MyView.xib` 파일의 `File's Owner`를 `MyView`로 설정해줘야 outlet이 잘 잡힌다. ([File's Owner란 무엇인가?](https://stackoverflow.com/questions/15251370/what-is-the-files-owner-in-interface-builder))

# ViewController.swift

```swift
import UIKit

class ViewController: UIViewController {
    @IBOutlet weak var myView: MyView!

    override func viewDidLoad() {
        super.viewDidLoad()
        
        myView.labelText = "Hello"
        myView.buttonText = "World"
        
    }
}
```

# 참고글

- https://medium.com/better-programming/swift-3-creating-a-custom-view-from-a-xib-ecdfe5b3a960