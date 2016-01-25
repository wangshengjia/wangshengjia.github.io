---
layout: post
title:  "Two weird things about protocol, extension and subclass in Swift"
date:   2016-01-24
categories: Swift
banner_image: ""
featured: false
comments: true
---

As I wrote a post about [method dispatch in Swift](http://allblue.me/swift/2016/01/23/swift-method-dispatch-with-protocol-extension-protocol-extension-and-subclass/) yesterday, in the meantime, I found there are two weird problems.

1. Override extension method behave weird. It seems that we can only override the methods in extension which is full compatible with Objective-C. Otherwise we'll have an compile error: **Declaration from extensions cannot be overridden yet**

<!--more-->

See the code below:
- This works
```swift
typealias Style = String

protocol Draggable {}

extension UIView: Draggable {
    func drag() {}
    func drag(with style: Style) {}
}

class MyDraggableView: UIView {
    override func drag() {}
    override func drag(with style: Style) {}
}
```
- This do not work
```swift
struct Style {}

protocol Draggable {}

extension UIView: Draggable {
    func drag() {}
    func drag(with style: Style) {}
}

class MyDraggableView: UIView {
    override func drag() {}
    override func drag(with style: Style) {} // compiler error: Declaration from extensions cannot be overridden yet
}
```

On StackOverflow, there is also a [question relevant](http://stackoverflow.com/questions/27109006/can-you-override-between-extensions-in-swift-or-not-compiler-seems-confused). But I still don't know what's the reason behind, what's the original purpose if there was one.

The second one is probably a bug, the code below will crash everytime the source editor in Playground, and give a `segment error` in Xcode 7.2
```swift
protocol Draggable {
    func drag()
}

extension Draggable {
    func drag(with style: String) {} // just rename drag to another name will fix the problem
}

extension UIView: Draggable {}
```

If you have any idea, please tell me on twitter [@wangshengjia](http://twitter.com/wangshengjia).

Thanks.
