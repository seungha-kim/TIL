---
title: "AVCaptureSession"
date: "2020-08-01"
category: "tech"
cover: https://graphql.org/
tags:
    - Swift
    - AVFoundation
---

회사에서 음성인식 API를 연동하는 미션을 받아서, sample buffer를 얻어오는 방법을 찾아보는 중.

Audio Toolbox API를 써서도 할 수 있는 것 같은데, 너무 로우레벨인 것 같아서 다른 방법을 찾아보다가 `AVCaptureSession` 활용하는 방법을 발견했다.


```swift
import UIKit
import AVFoundation

class ViewController: UIViewController {
    let captureSession = AVCaptureSession()

    override func viewDidLoad() {
        configureCaptureSession()
        captureSession.startRunning()
    }

    func configureCaptureSession() {
        captureSession.beginConfiguration()
        guard let audioCaptureDevice: AVCaptureDevice = AVCaptureDevice.default(for: .audio) else {
            return
        }
        let audioInput = try! AVCaptureDeviceInput(device: audioCaptureDevice)
        if captureSession.canAddInput(audioInput) {
            captureSession.addInput(audioInput)
        }

        let audioOutput = AVCaptureAudioDataOutput()

        if captureSession.canAddOutput(audioOutput) {
            captureSession.addOutput(audioOutput)
        }

        audioOutput.setSampleBufferDelegate(self, queue: DispatchQueue.global())

        captureSession.commitConfiguration()
    }
}

extension ViewController: AVCaptureAudioDataOutputSampleBufferDelegate {
    public func captureOutput(_ output: AVCaptureOutput, didOutput sampleBuffer: CMSampleBuffer, from connection: AVCaptureConnection) {
        for c in connection.audioChannels {
            print("averagePowerLevel \(c.averagePowerLevel) peakHoldLevel \(c.peakHoldLevel)")
        }
    }
}
```