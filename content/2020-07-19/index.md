---
title: "iOS 앱 번들에 이미지를 포함시키고 불러오기"
date: "2020-07-19"
category: "tech"
cover: https://graphql.org/
tags:
    - UIImage
---

(Programming iOS 13 읽으며 내용 정리)

# 앱 번들에 포함시키기

## 직접 포함시키기

빌드 세팅에서 직접 포함시킬 수는 있겠지만... 별로..

## Image set (in Asset catalog)

각각의 용도가 정해져있는 정적인 이미지들을 포함시킬 때 편한 방법. Xcode UI 상에서 High-resolution variant, UI style(light/dark), size class 등 다양한 경우의 수에 대해 적절한 이미지들을 개별적으로 설정해줄 수 있다. (물론 코드상으로도 가능하지만, 이 쪽이 더 편하다)

## Folder reference

다수의 이미지 파일이 들어있는 폴더를 앱 번들에 포함시킬 때, 'Create folder references' 옵션을 선택하면 group이 아니라 folder reference 형태로 포함된다. 이렇게 하면 앱 번들에 폴더가 통째로 들어간다. 즉 Xcode를 통해서가 아니라 외부 프로그램(파인더 등)에 의해 폴더에 추가된 파일들도 자동으로 앱 번들에 들어간다.

Folder reference는 namespace 역할도 해줄 수 있다. (후술)

# 코드에서 불러오기

## 파일 경로로 불러오기

```swift
let image = UIImage(contentsOfFile: Bundle.main.path(forResource: "...", ofType: "png")!)
```

## 이름으로 불러오기 - `UIImage(named:)`

`UIImage(named:)` 생성자는 두 가지 방법으로 이미지를 검색한다.

- Asset catalog의 image set 이름을 검색한다.
- 앱 번들의 최상위 폴더를 검색한다. `.png` 확장자는 생략해도 무방하다.

Asset catalog에 만들어둔 image set의 이름으로 이미지를 불러올 수 있다.

```swift
let image = UIImage(named: "MyImageSet")
```

Image set이 asset catalog의 폴더 안에 들어있고 해당 폴더의 'Provides Namespace' 옵션이 켜져있는 경우, 앞에 폴더 이름을 붙여주어야 한다.

```swift
let image = UIImage(named: "MyFolder/MyImageSet")
```

앱 번들의 최상위 폴더로부터 시작하는 상대 경로를 통해 불러올 수도 있다. Xcode에 folder reference를 추가하면 자동으로 앱 번들에 포함되므로, 편하게 folder reference 이름과 이미지 파일 이름을 적어주면 된다.

```swift
let image = UIImage(named: "MyFolderReferenceName/myImage.png") // .png를 생략해도 문제없이 불러온다.
```
