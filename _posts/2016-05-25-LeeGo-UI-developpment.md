---
layout: post
title:  "Put your whole UI into struct & enum with LeeGo"
date:   2016-05-25
categories: Swift
banner_image: ""
featured: true
comments: true
---

We all know that MVC pattern have some serious problems when dealing with a complex iOS project. Fortunately there are also a bunch of approaches that aim to fix the problems, most of them mainly address the `Controller` part, such as MVP, MVVM, MVSM or VIPER. But there is barely a thing which address the `View` part. Is that means we just run out of all the problems in the `View` part ?

<!--more-->

I think [the answer is NO](http://allblue.me/swift/2016/05/12/reuse-everything-of-your-UI-components/), especially when we face a complex UI presentation and meanwhile we need our app to be full responsive.

## LeeGo

I've talked the idea on [dotSwift](http://dotpost.com/2016/01/victor-wang-build-ios-ui-in-the-way-of-lego-bricks) at Paris early this year. After working on it by couple of month, I'm glad to introduce you [LeeGo](https://github.com/wangshengjia/LeeGo), the framework helps you to make UI development in a way declarative, configurable and highly reusable.
<p align="center"><img src="/media/leego.png" width="600"/></p>
Basically, it do only one thing: 

**Let you make UI development without UIView**. 

I've talked also once about [why this is important](http://allblue.me/swift/2016/05/12/reuse-everything-of-your-UI-components/). Actually almost [all the benefits](https://github.com/wangshengjia/LeeGo) of LeeGo are based on this fact.

#### Prepare the whole UI during the compile-time 

Most complexity of building UI is usually introduced by mutating components based on different given data at the runtime. Why not just prepare the whole screen during the compile-time ? LeeGo try to bring you the compile-time UI development, as IB file, but without all the inconveniences.

#### Brick 

In LeeGo, the basic concept is `Brick`. A brick represent the description of a UI component, usually UIView or it's subclass.

Generally it tells:

- what the kind of component's class is.
- how the component looks like.
- what's inside the component and how to lay them out.

A brick can have zero or more bricks inside recursively, it represent the same hierarchy as UIView and it's subviews. Brick is designed to be lightweight and pure value type. It is basically built with only simple JSON data types, such as `Int`, `Float`, `String`, etc, nothing more. 

The code below create the brick represent a header view which contains a label and an image view. 

```
// 1. I want a label with `titleFont` & `titleColor` and auto newline
// -> Create a brick with name "title"
let title = "title".build(UILabel).style([.font(titleFont), .textColor(titleColor), .numberOfLines(0)])

// 2. I want a image named "avatar", width = 80, height = 50
// -> Create a brick with name "avatar"
let avatar = "avatar".build(UIImageView).width(80).height(60)

// 3. I want a view named "header" which have "title" and "avatar" as children inside, layout as horizontal flow.
// -> Create a brick with name "header"
let header = "header".build().bricks(title, avatar) {
	title, avatar in
		Layout(["H:|[\(title)]-(>=10)-[\(avatar)]|",
        	    "V:|[\(title)]|", 
            	"V:|[\(avatar)]|")
}

```

Remember, the brick is a pure value type object, you can create it on site. But the better way is to create and keep it from the very beginning on background thread. Just treat them as same as your model object. And whenever we need to configure a view, we just pick up the right brick and configure the view as this brick.

```
// Configure your view
headerView.lg_configureAs(header)
```

#### Use Brick to replace your custom UIView and Xib files 

In the app of "Le Monde", a heavy content based app like NYT, do not have custom collection view cells and theirs IB files at all. Just because we isolated all UI logic out of cell by using brick, it allows us to have a cell totally generic. A standard `UICollectionViewCell` used for all different presentations.

#### Put your whole UI into struct & enum

One of my personal favorite thing in Swift is the new enumeration. By using `brick`, we can put the whole UI into a Swift enum. Let's take a Twitter's timeline cell as an example.

1, Decouple the complex view into small pieces of bricks
<p align="center"><img src="/media/tw_screenshot0.png" width="200"/><img src="/media/tw_screenshot3.png" width="200"/></p>

2, Group them and name them carefully.

It's important to give your brick a meaningful name, a name represent the nature of your brick. Such as “title”, “subtitle”, “avatar”, “likeButton”, “favoriteButton”, “subscriberButton”, “date”, etc…

3, Define the enum cases which fit the different bricks

```
enum Twitter: BrickBuilderType {

    // leaf bricks
    case username, account, avatar, tweetText, tweetImage, date, replyButton, retweetButton, likeButton

    // complex bricks
    case accountHeader, toolbarFooter, retweetHeader

    // root bricks
    case tweet
    
    static let brickClass: [Twitter: AnyClass] = [
        username: UILabel.self,
        account: UILabel.self,
        avatar: UIImageView.self,
        tweetText: UITextView.self,
        tweetImage: UIImageView.self,
        date: UILabel.self,
        replyButton: UIButton.self,
        retweetButton: UIButton.self,
        likeButton: UIButton.self,
    ]
}
```

4, Implement the concrete method which build and return the different brick instances based on different enum cases.

Well I'm not going to paste the whole "stylesheet" here, there is nothing more complicated than a configuration file. You may checkout this entire [demo project on Github](https://github.com/wangshengjia/LeeGo) for the details.

## Backend driven native UI

`Brick` is JSON convertible, every `Brick` instance can be turned into a JSON object and vice versa. Which means we can define and manipulate all bricks server side, powered the whole UI from JSON payload. Just treat your bricks like CSS, embed them with model object.

## Where to go from here

Please checkout [LeeGo on Github](https://github.com/wangshengjia/LeeGo), try the demo project. Begin to think how your project can benefit from LeeGo, it's totally smooth to begin with integrating only a small part, also free to drop all without any side effect since LeeGo is designed to be UIKit friendly and non-intrusive at all.

If you have any suggestion or question, please fire an issue on Github or directly ping me on [Twitter](twitter.com/wangshengjia) by @wangshengjia

Enjoy. 🎉🎉





