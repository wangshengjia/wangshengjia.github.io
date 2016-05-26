---
layout: post
title:  "Build UI without UIView"
date:   2016-05-12
categories: Swift
banner_image: ""
featured: false
comments: true
---

Encouraged by John Sundell's great video at Scale, I've decided to write more about component driven UI, about why we should and how to do it (for iOS).
In this post, I'll try to focus on talking only the very basic idea.

<!--more-->

#### Before all, let's take one minute…

Just think about how we setup UI typically in iOS.

Either with or without Interface Builder, we always need to work with a `UIView` instance. Either setup view's frame manually or use auto layout, we always need to add a `UIView` instance as a subview. Either deal with standard auto layout API or use a fancy third party auto layout helper library, we setup auto layout constraints always to the `UIView` instance.

**The thing is: When dealing with UI in iOS, there is always the `UIView` instance, which is heavy, mutable and main-thread required.**

As the consequences: 
- We have to deal with a bunch of verbose code or a bunch of IB files, or both.
- The code is hardly to be reused, have to repeat the similar code or IB files once and once again.
- We have to often separate UI configuration logic from the same component into different places: a part be in IB file, the rest be in code.

Along with the increasing complexity, these problems will finally cause the nightmare of the maintenance.

## Begin to reuse everything from UI by getting rid of UIView instance
Before talking about how to get rid of `UIView` instance, let's look a little bit closer into UI component.

#### Anatomy of UI component

Almost all UI components, such as a view, a label, an image view or somethings else, can be decoupled and described in 5 parts:

1. Type - which kind of class? UIView or it's subclass. This is the only thing we must have to define a UI component.
2. Name - what's the name when component be reused? A component need a name to be reusable. A name can represent the nature of this component. Such as "title", "subtitle", "avatar", "likeButton", "favoriteButton", "subscriberButton", "date", etc…
3. Style - how the component itself looks like? It could be either the visual appearances, such as `backgroundColor`, `numberOfLines` or the non-visual view properties, such as `userInteractionEnabled`, `scrollEnabled`. Styles are almost always key/value convertible, can easily be put into a dictionary.
4. Children components - what's inside the parent component? It surely will be another UI component, so it also can be described in the same way recursively. It means all we need is just the names of children components in order to be capable to retrieve it later.
5. Layout - how to arrange the children? If there are more than one sub-component, we'll need a layout for them. This is the tricky part. We can not avoid `UIView` instance by setting up frame manually, neither by using auto layout. Fortunately Apple also give us another way: Visual Format Language. VFL give us the possibility to define the whole layout with pure ASCII.6. 

```
let layout = ["H:|-20-[title]-10-[avatar]-20-|", "V:|-20-[title]-10-[subtitle]-20-|"]
```

You may haven't noticed yet, we already have everything we need to define the whole UI, **without `UIView` instance**, only value type here. Further more, every component would be reusable by their name. All styles and layouts would also be reusable.

<p align="center"><img src="/media/diag.png" width="300"/></p>

And … that's it. That's the whole thing: we now have our UI without single `UIView` instance. 🎉

## What do we have now ?

- We can define the whole UI from anywhere you want, all UI logics can be centralised in one place.
- All components are available in the stock, so we can reuse them whenever we need.
- Since every thing is value type now, which means they are totally thread-safe. So we can compute and manipulate UI hierary easily on background.
- Since every thing is somehow key/value convertible, it also give a capability to drive naive UI from your backend via JSON payload.

#### Think about Lego's bricks
Now what we need to do is analysing all the screen of our app, decouple and name them carefully into small pieces of UI component. The idea is just as same as Lego bricks.
<p align="center"><img src="/media/lego.png"/></p>
Every component act exactly as the same role as Lego's brick. It can be a simple brick, which do not composited with others.
<p align="center"><img src="/media/legoRobot.png"/></p>
It can also be the super complex unit, which composited by many different simple bricks with different structure.

## What's next ?

As mentioned at the beginning, in [another post](), I'll present you how to do component driven UI in details with codes and examples with [LeeGo](github.com/wangshengjia/LeeGo).
