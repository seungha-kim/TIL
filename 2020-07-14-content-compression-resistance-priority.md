# iOS Auto Layout - Content Compression Resistance Priority

이름도 참 길다... content compression resistance priority... 내용압축저항우선순위...

일단은 constraint에 부등식을 먹였을 때 의미를 가지는 놈인 것 같은데... 내일 자세히 살펴봐야지

TODO: https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/ViewswithIntrinsicContentSize.html#//apple_ref/doc/uid/TP40010853-CH13-SW1

```swift
import UIKit

class ViewController: UIViewController {
    let container = UIView()
    let textField1 = UITextField()
    let textField2 = UITextField()

    override func viewDidLoad() {
        super.viewDidLoad()
        
        setupViews()
        setupConstraints()
    }
    
    private func setupViews() {
        // container
        container.backgroundColor = .red
        view.addSubview(container)
        
        // textField1
        container.addSubview(textField1)
        textField1.backgroundColor = .white
        
        // textField2
        container.addSubview(textField2)
        textField2.backgroundColor = .white
    }
    
    private func setupConstraints() {
        container.translatesAutoresizingMaskIntoConstraints = false
        textField1.translatesAutoresizingMaskIntoConstraints = false
        textField2.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            // container horizontal
            container.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor, constant: 48),
            container.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor, constant: -48),
            
            // container vertical
            container.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 48),
            container.heightAnchor.constraint(equalToConstant: 160),
            
            // textField1 horizontal
            textField1.leadingAnchor.constraint(equalTo: container.leadingAnchor, constant: 16),
            textField1.widthAnchor.constraint(greaterThanOrEqualToConstant: 100),
            
            // textField1 vertical
            textField1.centerYAnchor.constraint(equalTo: container.centerYAnchor),
            
            // textField2 horizontal
            textField2.trailingAnchor.constraint(equalTo: container.trailingAnchor, constant: -16),
            textField2.widthAnchor.constraint(greaterThanOrEqualToConstant: 100),
            
            // textField2 vertical
            textField2.centerYAnchor.constraint(equalTo: container.centerYAnchor),
            
            // textField1 - textField2
            textField2.leadingAnchor.constraint(greaterThanOrEqualTo: textField1.trailingAnchor, constant: 16),
        ])
        
        // textField2의 content compression resistance priority를 textField1 보다 높게 설정.
        // 이렇게 되면 textField2의 내용이 길어졌을 때 textField1을 밀어내면서 길어진다.
        // 이를 생략하면 반대로 textField1의 내용이 길어졌을 때 textField2를 밀어냄 (같은 값일 때 무엇이 우선순위를 갖는지는 검색해봤는데 찾지 못했다.)
        textField2.setContentCompressionResistancePriority(
            textField1.contentCompressionResistancePriority(for: .horizontal) + 1,
            for: .horizontal
        )
    }
}
```