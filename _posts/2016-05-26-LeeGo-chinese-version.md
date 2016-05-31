---
layout: post
title:  "用 struct 和 enum 来构建你的整套 UI"
date:   2016-05-26
categories: Swift
banner_image: ""
featured: false
comments: true
---

之前分享过一次 [LeeGo](github.com/wangshengjia/LeeGo)，有同学说不是很明白什么时候用，以及具体有什么好处。我觉得有必要再安利一次，好处往简单了说就一句话，可以让大家「脱离 UIView 做 UI 开发」。还写过[另一篇文章](http://allblue.me/swift/2016/05/12/reuse-everything-of-your-UI-components/)讨论过为什么。其他的诸如声明式，高可用性UI，可服务器端远程更新等等一切好处其实都或多或少是源于这一条。
<!--more-->
我们大家都知道 MVC 模式确实存在很多问题，尤其是在面对一个UI相对复杂的项目时。不过好在我们也有很多办法去改善这些问题，大部分都是针对`Controller`那部分的，比如 MVP，MVSM，MVVM 或者 VIPER之类的，但我们很少看到有什么东西是针对`View`这部分的。确实相对于`Controller`，`View`的部分相对要简单不少，很多时候比较直白。但当我们面对一个相对复杂的视图混杂着不同的业务逻辑和不同的响应式布局时，越来越多的各种自定义视图和IB文件，常常会让项目变得不可维护。

## [LeeGo](github.com/wangshengjia/LeeGo)

年初的时候，我在 dotSwift 的 talk 上谈过这个想法，努力了几个月以后总算是初步实现了基本功能（还有很多issue躺在那里，需要继续努力）。[LeeGo](github.com/wangshengjia/LeeGo) 是一个 Swift 框架，旨在提供一种更好的方式处理UI，让UI开发变的声明式，可配置和可重用。

<p align="center"><a href="http://github.com/wangshengjia/LeeGo"><img src="/media/leego.png" width="600"/></a></p>

#### 在编译期间完成所有 UI

简单来说，LeeGo 只做了一件最最基本的事：**可以让你脱离 UIView 打造 UI**

也正是基于这点，让我们有可能在**编译期间**就完成所有的视图相关开发。一般来说，视图部分的复杂度大多来源于需要根据不同的数据对象来动态调整视图和布局。而 LeeGo 提供的一个根本性的解决方案就是尽量少的在运行时更改视图，就像 IB 文件一样，只不过更强大且没有副作用。

#### Brick

LeeGo 里的一个最基本的概念是`Brick`，一个`Brick`代表了对一个UI组件的完整描述，通常是`UIView`或者它的子类。简单的说，一个`Brick`描述了三件事：

- **UI组件是什么类**：是`UILabel`，`UIImageView`或者是一个完全自定义的`UIView`子类
- **UI组件长什么样**：针对各种不同UI组件的不同属性，典型的比如背景颜色，如果是`UILabel`的话可以是行数，字体和颜色等等
- **UI组件里面有什么以及如何布局**：描述如何布局内部的子组件

一个Brick可以有0个或者多个其它`Brick`嵌套其中，整体结构类似于现有的视图和子视图的结构。不同的是`Brick`是完全`value type`的，基本都是由支持 JSON 转化的数据格式，比如 Int，Float，String 之类的。

下面这段代码展示了如何创建一个代表一个`header view`的 brick，`header view`里面有一个 title 和一个 avatar

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

之前说了，`Brick`是完全value type的，你可以在用的时候创建，但更好的方式是在一开始就创建好所有的UI，无论是在主线程还是在后台处理都行。就像对待 model 对象一样。然后当我们要用的时候，只要从仓库里找到对应的`Brick`拿出来用就好了：

```
// Configure your view
headerView.lg_configureAs(header)
```

#### 用 Brick 代替项目里的自定义 UIView 和 IB 文件
在“Le Monde”（法国世界报，重内容的新闻类应用，类似网易新闻这样的）的app里，我们没有使用任何自定义的`collection view cell`和 IB 文件，因为抽离了所有的UI逻辑，使得我们的`collection view cell`变的完全通用，一个标准的`UICollectionViewCell`就可以对应所有的不同显示方式。

#### 用 struct 和 enum 来构建你的整套 UI 吧
在 Swift 里，一个我最喜欢的地方便是 Swift 的 `enum`。通过使用Brick，我们可以把整套UI都放进一个Swift的enum里面。这里用一个推特app里时间线的cell来举个例子：

1, 把一个复杂的UI组件合理的分解成简单的组件
<p align="center"><img src="/media/tw_screenshot0.png" width="200"/><img src="/media/tw_screenshot3.png" width="200"/></p>
2, 合理的分类和命名这些组件

给 brick 一个合理的名字是很重要的一点，这是之后重用的关键。一个名字代表了这个 brick 最基本的属性和作用。比如title, subtitle, avatar, header, footer之类的。

3, 以同样的分类来创建enum，代表不同的brick

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

4, 实现具体的方法，使得可以通过不同的 enum case 来得到相对应的 brick 实例

具体实现上，我就不把整个方法贴在这里了。基本接近于配置文件，详情可以下载查看[Github上的demo项目](github.com/wangshengjia/LeeGo)

## 通过服务器端驱动原生UI
Brick和JSON对象可以互相转换。每个Brick实例都可以转化成一个JSON实例，反之亦然。也就是说我们完全可以像对待我们的数据流对象一样，从后台通过JSON的方式加载到本地，然后转化为Brick来驱动原生UI。

## 下一步 ？
更多详细内容，和Facebook ComponentKit的比较等等，请移[Github](github.com/wangshengjia/LeeGo)。尝试着跑一下demo项目，试着思考一下可以从哪里开始着手用在自己的项目里。LeeGo完全非侵入性，UIKit友好的API设计，使得完全不用什么改动就可以直接用在项目里，如果哪天想要替换掉，也不会有任何多余的副作用。

如果你有任何疑问或是建议，可以直接在Github上开一个issue，也可以在Weibo上@ShengjiaWang

谢谢～🎉

