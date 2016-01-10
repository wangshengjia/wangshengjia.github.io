---
layout: post
title:  "Quizzes about properties in Swift"
date:   2016-01-08
categories: Swift
banner_image: ""
featured: false
comments: true
---

Come on, I know you guys love both Quiz and Swift. This post brings you some quizzes about properties in Swift, try to tell the answer without actually run it, could you get them all ? Just try it.

The quizzes may be easy for some of you, may be not. The most important thing here, I hope you guys all **have fun** from it, cheers 🎉🎉🍻

<!--more-->

You can find the answers and some explications [HERE](http://www.allblue.me/swift/2016/01/08/quiz-answers)

**Tested with Swift 2.1 & Xcode 7.2**

OK, here we go!

## Beginner quiz

### Quiz #1:
What's the order of numbers will be printed in the output?

```swift
class Property {
    let content: String
    init(_ newContent: String) {
        content = newContent
        print("\(self.content)", terminator: ", ")
    }
}
class PropertyQuiz {
    var property1: Property
    let property2: Property

    let property3 = Property("3")
    lazy var property4 = Property("4")

    init() {
        property1 = Property("1")
        property2 = Property("2")
    }
}
let property4 = PropertyQuiz().property4
// => ?
```

### Quiz #2:
What's the result in two different cases respectively?

```swift
class PropertyQuiz {
    var property = "1" {
        willSet {
            self.property = "2"
        }
        didSet {
            self.property = "3"
        }
    }

    init() {
        self.property = "4"
        // config()
    }

    func config() {
        self.property = "5"
    }
}
let quiz = PropertyQuiz()
print(quiz.property)

// Without calling `config()` => ?
// With calling `config()` => ?
// (1, 2, 3, 4, 5, compiler error or runtime crash)
```

### Quiz #3:
Respectively under these five cases, what's the result?

```swift
class PropertyQuiz {
    // case 1
    var property: String {
        get {
            self.property = "2"
            return "1"
        }
        set { }
    }

    // case 2
    var property: String {
        get {
            print(self.property)
            return "1"
        }
        set { }
    }

    // case 3
    var property: String {
        get {
            return "1"
        }
        set { self.property = "4" }
    }

    // case 4
    var property: String {
        get {
            return "1"
        }
        set { print(self.property) }
    }

    // case 5
    var property: String {
        get {
            self.property = "2"
            return "1"
        }
        set { print(self.property) }
    }
}
let quiz = PropertyQuiz()
quiz.property = "3"
print(quiz.property)

// => ?
// (1, 2, 3, compiler error or runtime crash)
```

### Quiz #4:
Respectively under these four cases, what's the result?

```swift
class PropertyQuiz {
    let property1 = {
        print(self.property2) // case: 1
        return "1"
    }()

    lazy var property2: String = {
        print(self.property1) // case: 2
        print(self.property2) // case: 3
        self.property2 = "3" // case: 4
        return "2"
    }()
}
let quiz = PropertyQuiz()
print(quiz.property2)

// => ?
// (1, 2, 3, compiler error or runtime crash) ?
```

## Advanced quiz

### Quiz #5:
Still remember Quiz #2? Now with inheritance, we actually added property observing to computed property, is there still the same result?

```swift
class PropertyQuiz {
    var property: String {
        get {
            return "1"
        }
        set { }
    }
    init() { }
}

class PropertySubQuiz: PropertyQuiz {

    override var property: String {
        willSet {
            self.property = "2"     // case 1
            // super.property = "2" // case 2
        }
        didSet {
            self.property = "3"     // case 1
            // super.property = "3" // case 2
        }
    }

    override init() {
        super.init()
        self.property = "4"
        // config()
    }

    func config() {
        self.property = "5"
    }
}

let subQuiz = PropertySubQuiz()
print(subQuiz.property)

// case1 => ?
// case2 => ?
// (1, 2, 3, 4, 5, compiler error or runtime crash) ?
```

### Quiz #6:
Compare with the Quiz #5, what if we do it vice versa? What's the result respectively in two cases?

```swift
class PropertyQuiz {
    var property: String = "1" {
        willSet {
            self.property = "2"
        }
        didSet {
            self.property = "3"
        }
    }

    init() { }
}

class PropertySubQuiz: PropertyQuiz {

    override var property: String {
        get {
            return super.property
        }
        set { } // case 1
        set { super.property = "6" } // case 2
    }

    override init() {
        super.init()
        self.property = "4"
        config()
    }

    func config() {
        self.property = "5"
    }
}

let subQuiz = PropertySubQuiz()
print(subQuiz.property)

// => ?
```

Feel free to ping me on twitter by @wangshengjia if there is something wrong or you have an question or anything 😁
Enjoy 🎉🍻
