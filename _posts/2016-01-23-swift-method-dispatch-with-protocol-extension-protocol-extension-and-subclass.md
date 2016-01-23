---
layout: post
title:  "Method dispatch in Swift with protocol, protocol extension, extension and subclass"
date:   2016-01-23
categories: Swift
banner_image: ""
featured: false
comments: true
---

The protocol extension introduced in Swift 2.0 is certainly a powerful tool, meanwhile things are also becoming more complex. __Alexandros Salazar__ has wrote a post ["The Ghost of Swift Bugs Future"](http://nomothetis.svbtle.com/the-ghost-of-swift-bugs-future) to discuss it. There is also a [Flow Diagram](https://d262ilb51hltx0.cloudfront.net/max/1600/1*SIcSsfmBCp4tNzLxGJAbdw.png) made by [Omar Abdelhafith](https://medium.com/@NSomar) which showing how the methods dispatch works in Swift.

In this post, I try to discover a little bit more in case there is an extension or a subclass.

**Tested with Swift 2.1 & Xcode 7.2**

First of all, let's replay the conclusion of Alexandros.

<!--more-->

```swift
protocol Draggable {
    func drag()
}

extension Draggable {
    func drag() {
        print("drag called from extension Draggable")
    }
    func drag(with style: String) {
        print("dragWithStyle called from extension Draggable")
    }
}

class MyDraggableView: Draggable {
    func drag() {
        print("drag called from MyDraggableView")
    }
    func drag(with style: String) {
        print("dragWithStyle called from MyDraggableView")
    }
}

let view = MyDraggableView()

view.drag()                         // drag called from MyDraggableView
(view as Draggable).drag()          // drag called from MyDraggableView

view.drag(with: "Style")                // dragWithStyle called from MyDraggableView
(view as Draggable).drag(with: "Style") // dragWithStyle called from extension Draggable
```

Well, we have the expected result. Since `dragWithStyle` is not declared in `Draggable`, and the instance inferred as `Draggable` because the protocol is explicitly specified, it will call the default implementation of protocol.

So far so good. Then I'd like to go a little bit further by adding one more level: an extension of UIView which conform `Draggable`, and make `MyDraggableView` as a subclass of UIView. Will that change the conclusion rules of Alexandros?

```swift
extension UIView: Draggable {}

class MyDraggableView: UIView {
    func drag() {
        print("drag called from MyDraggableView")
    }
    func drag(with style: String) {
        print("dragWithStyle called from MyDraggableView")
    }
}

let view = MyDraggableView()

view.drag()                         // drag called from MyDraggableView
(view as Draggable).drag()          // drag called from extension Draggable

view.drag(with: "Style")                // dragWithStyle called from MyDraggableView
(view as Draggable).drag(with: "Style") // dragWithStyle called from extension Draggable
```
Oups, it seems the result is not as we expected which supposed to followed rules:

> - IF the inferred type of a variable is the protocol:
  - AND the method is defined in the original protocol
  - THEN the runtime type’s implementation is called, irrespective of whether there is a default implementation in the extension.

`drag()` get called with the default implementation instead of runtime type's implementation. We run into the issue so called *"Why does the method that I wrote overriding protocol extension X never get called?"* again even we declared `drag()` in the protocol.

What's changed? Is that because we have an extension of `UIView` and make `UIView` to conform `Draggable` protocol? Let's refactor a little our example:

```swift
class MyDraggableView: Draggable {}

class MyDraggableButton: MyDraggableView {
    func drag() {
        print("drag called from MyDraggableButton")
    }
    func drag(with style: String) {
        print("dragWithStyle called from MyDraggableButton")
    }
}

let view = MyDraggableButton()

view.drag()                         // drag called from MyDraggableButton
(view as Draggable).drag()          // drag called from extension Draggable

view.drag(with: "Style")                // dragWithStyle called from MyDraggableButton
(view as Draggable).drag(with: "Style") // dragWithStyle called from extension Draggable
```

Now we removed the suspicion of extension, but we still have the same result. Ok, what if we move `drag()` into the superclass:

```swift
class MyDraggableView: Draggable {
    func drag() {
        print("drag called from MyDraggableView")
    }
}

class MyDraggableButton: MyDraggableView {
    func drag(with style: String) {
        print("dragWithStyle called from MyDraggableButton")
    }
}

let view = MyDraggableButton()

view.drag()                         // drag called from MyDraggableView
(view as Draggable).drag()          // drag called from MyDraggableView

view.drag(with: "Style")                // dragWithStyle called from MyDraggableButton
(view as Draggable).drag(with: "Style") // dragWithStyle called from extension Draggable
```

Then we can get the expected result which respected the rules of Alexandros.
By now, things are pretty clear:

- The class which **INDIRECTLY** conform to the original protocol,
  - will always call default implementation when inferred as a protocol,
  - will always call dynamic type's implementation when inferred as it's real runtime type. Irrespective of whether it's defined in protocol or not.

Now if we modify a little flow diagram based on what we've learned, it should be like this:
![diagram](/media/swift_method_dispatch.jpg)

And the rules:

- IF the inferred type of a variable is the protocol:
  - IF the method is in a class conform directly to the original protocol:
      - AND the method is defined in the original protocol
          - THEN the runtime type’s implementation is called, irrespective of whether   there is a default implementation in the extension.
      - AND the method is not defined in the original protocol,
          - THEN the default implementation is called.
  - ELSE IF the method is in a subclass of another class who conform to the original protocol
      - THEN the default implementation of protocol is called.
- ELSE IF the inferred type of the variable is the type
  - THEN the type’s implementation is called.

If you have any question or another explanation of this issue, please ping me on Twitter by [@wangshengjia](http://twitter.com/wangshengjia)

Enjoy 🎉🍻
