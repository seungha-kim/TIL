---
title: "Core Animation - presentation tree"
date: "2020-07-12"
category: "tech"
cover: https://graphql.org/
tags:
    - Swift
    - UIView
---

```swift
// https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/CoreAnimationBasics/CoreAnimationBasics.html#//apple_ref/doc/uid/TP40004514-CH2-SW3

// Core Animation은 세 벌의 레이어 트리를 관리한다.
// model layer tree - 응용 개발자가 직접 조작하는 트리. 애니메이션의 목표(끝 값)을 저장한다.
// presentation tree - 화면에 보이는 레이어의 현재 상태를 나타낸다. 읽기 전용. presentation() 메소드로 복사본을 얻을 수 있다.
// render tree - Core Animation이 화면을 그릴때 이걸 사용한다. private

import UIKit

class ViewController: UIViewController {
    let rect = CALayer()
    var tickTimer: Timer?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        setupActions()
        setupLayers()
    }
    
    private func setupActions() {
        let tapRec = UITapGestureRecognizer()
        view.addGestureRecognizer(tapRec)
        tapRec.addTarget(self, action: #selector(handleTap))
    }
    
    @objc func handleTap() {
        startAnimation()
    }
    
    private func setupLayers() {
        rect.backgroundColor = UIColor.red.cgColor
        rect.frame = CGRect(origin: .zero, size: CGSize(100, 100))
        
        view.layer.addSublayer(rect)
    }
    
    private func startAnimation() {
        CATransaction.begin()
        CATransaction.setAnimationDuration(1)
        CATransaction.setCompletionBlock { [weak self] in
            self?.tickTimer?.invalidate()
        }
        
        // NOTE: setAnimationDuration, setCompletionBlock 등의 세팅이 완료된 후에 property 변경이 와야 함
        // 속성 변경이 된 이후에는 setAnimationDuration, setCompletionBlock 등이 아무 효과가 없다.
        rect.position.y = 200
        CATransaction.commit()
        
        // NOTE: Timer.scheduledTimer는 iOS 10.0 버전에서 추가됐다.
        tickTimer = Timer.scheduledTimer(withTimeInterval: 0.1, repeats: true) { [weak self] (timer) in
            guard let this = self else {
                timer.invalidate()
                return
            }
            print(this.rect.presentation()?.position.y ?? -1)
        }
    }
}

// 출력 결과

//71.32366225123405
//94.92525309324265
//137.3231053352356
//156.07120096683502
//170.37702202796936
//182.80697762966156
//191.13014936447144
//196.3515430688858
//199.15319979190826
//200.0

```