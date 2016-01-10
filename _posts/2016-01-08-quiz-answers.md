---
layout: post
title:  "The answers - Quiz about properties in Swift"
date:   2016-01-08
categories: Swift
banner_image: ""
featured: false
comments: true
---


### Before get into the answers, you may want to checkout the quizzes from [HERE](http://www.allblue.me/swift/2016/01/08/quiz-about-properties-in-swift)

<!--more-->

*****

**Tested with Swift 2.1 & Xcode 7.2**

### Answer #1

Before `init()` get called, `property3` is already initialized. Then `init()` get called and initialize property1 & property2 inside. Finally, we access the property explicitly to initialize property4.

```swift
// => 3, 1, 2, 4,
```

### Answer #2
1. Assign a new value inside `init()` will not fire `willSet` & `didSet`. So property value changed from `1` to `4`, then printed `4`.
2. Assign a new value inside `config()` will fire `willSet` & `didSet`. So actually the value path is: `1 -> 4 -> 2 -> 5 -> 3`, then printed `3`.
3. Should notice that we can actually access `self` inside property observing, it will not cause infinite loop.

```swift
// Without calling `config()` => 4
// With calling `config()` => 3
```

### Answer #3

1. In `case 1`, assign the property will not fire getter, so technically we can do this even in his own getter.
2. In `case 2`, on the other hand, we fire getter in his own getter. So yes, infinite loop.
3. `Case 3` is as same as `case 2`, try to assign the property in setter will cause infinite loop.
4. `Case 4` is as same as `case 1`, call getter in his own setter is OK.
5. `Case 5` is a combination of `case 1` and `case 4`, we try to call getter from setter and vice versa. It will cause an infinite loop between getter & setter.
6. Should notice that usually we don't have this kind need. We'll get a warning if we try to access the property in his own getter or setter without explicitly using `self.`.

```swift
// case 1 => 1
// case 2 => runtime crash (infinit loop)
// case 3 => runtime crash (infinit loop)
// case 4 => 1
// case 5 => runtime crash (infinit loop)
```

### Answer #4

1. We can not access `self` in non-lazy stored property. Quite obviously.
2. We can access `self` when stored property is `lazy`. So it's safe to access other property.
3. When try to access property getter in his own initializer, it will cause an infinite loop. Also quite obviously.
4. Only assign the value will not called getter, so it is safe from infinite loop. Even this approach is kind of meaningless ¯\\\_(ツ)\_/¯

```swift
// case 1 => compiler error
// case 2 => 1, 2
// case 3 => runtime crash (infinit loop)
// case 4 => 2
```

### Answer #5

1. Case 1: Override and re-assign the property value in his own `willSet` & `didSet` will cause infinite loop.
2. Case 2: We can safely access `super` in property observing. In this case the value actually from super's getter, so it printed `1`.
3. Should notice that assignment of an overridden property in subclass's `init()` will actually fire `willSet` & `didSet`.

```swift
// case 1 => runtime crash (infinit loop)
// case 2 => 1
```

### Answer #6

1. The property is override by a computed property with an empty setter, which means `willSet` & `didSet` won't get fired since it can not be assigned with a new value. So it will always return value from getter: `super.property` which is `1`.
2. In `case 2`, we assign a new value in the setter, it will fire `willSet` & `didSet`, so the final value will be `3`.

```swift
// case 1 => 1
// case 2 => 3
```

### Conclusion

Well, I hope you guys had a good time from this quizzes. Compare with Objective-C's property/iVar, Swift brings us a way more powerful property system, a lot more possibilities. There are some circumstances you may never have in production, otherwise it probably a signal that you should refactor your codes. But hey, there is still a lot of fun to discover more details of the edge cases, right?

One more thing, Swift is still moving forward fast. Or even faster after been open source. So things may change, for instance, if the [proposal about "Property behaviors"](https://gist.github.com/jckarter/f3d392cf183c6b2b2ac3) of [Joe Groff](https://twitter.com/jckarter) is adapted, some of those quiz/answer may be obsoleted very soon. So, keep moving, keep learning 😉

Feel free to ping me on twitter by @wangshengjia if I am wrong about something I've mentioned or you have any question/anything.
Enjoy 🎉🍻
