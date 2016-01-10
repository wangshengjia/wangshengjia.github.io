---
layout: post
title:  "Have fun with properties in Swift"
date:   2016-01-09
categories: Swift
banner_image: ""
featured: false
comments: true
---

For years as a developer who using Obj-C every day, we are so familiar with the code below to define a property.

Usually I’d like to suggest to the beginner to always define a property in this way by default (Well.. in recent days, we may also take care a about nullability annotation). Because this is the best practise for most cases, there is no reason to use another approach with “modern” Xcode, such as iVar with synthesized keyword.

<!--more-->
Now with Swift, things are all changed. We totally say goodbye to instance variable. Instead, we now have so many different ways to create and initialise a property. In this blog post, I’d like to walkthrough all the interesting cases about all type of property in Swift. You may use this post as a mini-cheatsheet if you want.

__As always, you can checkout the whole Gist from Github.__

__OK, let’s begin. Enjoy and have fun 🍻🎉🎉__

## A little preparation
Before all, let’s do some preparations first. In order to view clearly how are the properties initialised differently, we make a `Property` type which `print` the value in it’s init() method.

```swift
class Property {
    let content: String
    init(_ newContent: String) {
        content = newContent
        print("\(self.content)", terminator: ", ")
    }
}

class PropertyCheatSheet { }
```

## I. Stored property
- It can be either initialized in `init()`, in this case it require an explicit type.
- Or it can be set with a value directly, then let the compiler infer the right type.
- Stored property can also be inited in a `lazy` way, which means property4 will be inited only at the first time be accessed

```swift
var property1: Property // mutable property with Property type
let property2: Property // immutable property with Property type

// Property("3") will be called immediatelly, even before property1 & property2 be initialized
let property3 = Property("3")
// A lazy property can only be mutable.
lazy var property4 = Property("4")

init() {
    // need to be set a value in initializer
    property1 = Property("1")
    // even with an immutable property, you still need to set a value for once in initializer
    property2 = Property("2")
}

PropertyCheatSheet()
cheatsheet.property4
// => 3, 1, 2, 4,
```

A stored property can also be inited with a closure value. The syntax is a little bit similar as a `computed property` in some cases, but they have totally different behaviours. We'll walkthrough it later.

```swift
// Stored property with closure default value
var property5: Property = {
    return Property("5")
}()
// Computed property
var property5: Property {
    return Property("5")
}
```

As a stored property with a closure default value:

- Closure will be executed only once at the very beginning.
- It can be immutable, not as `Computed property`.
- We can not access `self` inside closure. Quite make sense since the moment closure executed there is even no `self`.
- It can also be `lazy`, of course in this case it need to be mutable. As return, we can access `self` in the closure.
- Be careful, if you try to access property itself in its own closure, you'll get a infinit loop 😜

```swift
...
let property5 = {
    // print(self) -> compile error
    return Property("5")
}()

lazy var property6: Property = {
    // print(self.property1) -> this is OK
    // print(self.property6) -> infinit loop
    return Property("6")
}()
...
var cheatsheet = PropertyCheatSheet()
cheatsheet.property4
cheatsheet.property6
// => 3, 5, 1, 2, 4, 6,
```

## II. Computed properties
As the name, `computed property` is kind of property you want to re-compute the value based on the context every time you access it, or set other properties and values indirectly.

- Computed property must have an explicit type.
- Computed property is kind of `lazy`, so it can only use `var` keyword. But it do not means you can actually `assign` the property.
- The `get` block of computed property will be executed every time property be accessed
- Without `set` block, the property is `read-only`
- We can access `self` inside since it is kind of `lazy`. Same as an stored property with closure, if you try to access property itself, then you'll get an infinit loop.

```swift
var property7: Property {
  // get {
    // print(self.property7) -> infinit loop
    return Property("7")
  // }
}
```

- If there is a setter, we must give also a getter.
- `newValue` as the default shorthand, can be changed with ohter custom value name.
- Should also be careful with the infinit loop issue.

```swift
// 1. If there is a setter, we must have a getter
// 2. "newValue" as default shorthand, can be changed with set(/* customValueName */) {}
var property8: Property {
    get {
        return Property(property1.content)
    }
    set {
        // self.property8 = Property("8") -> infinit loop
        property1 = Property(property1.content + "_" + newValue.content)
    }
}
```

## III. Property observing
