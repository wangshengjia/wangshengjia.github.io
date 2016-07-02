---
layout: post
title:  "Build your own Xcode 8 Source Editor Extension"
date:   2016-06-28
categories: Xcode
banner_image: ""
featured: false
comments: true
---

Since WWDC 2016, I was so excited about the announcement of the new Xcode source editor extension released in Xcode 8. Some of you may know I once wrote another [Xcode plugin called VWInstantRun](https://github.com/wangshengjia/VWInstantRun), an  plugin made with some runtime hacks, a.k.a. old/unofficial way.

So naturally I'd love to port this plugin to the brand new Xcode source editor extension. I gave a try this weekend and I failed to implement `InstantRun` with the new approach. Basically there is two major limit to achieve it:

<!--more-->

1. The only thing which expose to us from API is the text of current file inside a string buffer object. There is no way to access some UI relevant component at all. Which it's not possible to output the result directly into debug area as `VWInstantRun` do.
2. Since we suppose build thing inside an extension now, which is a sandbox. Which means there is no way to run build command in CLI during runtime.

Well, therefore I give it up. But I'm not saying the new Xcode extension is not worth to look at. In contrast, I think it's a very good beginning, by giving access of the most important part (the source editor) which allows to make [lots of useful features](https://github.com/cyanzhong/xTextHandler) and meanwhile eliminating any potential privacy and safety threat.

## Build one

I'll try to give a simple tutorial here to show you how easy to build an Xcode extension yourself. Let's begin to do it by borrowing some ideas from AppCode, such as quickly deleting/duplicating selected lines. 

#### I. Open Xcode 8.

(current version is beta 1)

Create a new macOS project and activate related scheme.

<p align="center"><img src="/media/new_mac_app.png" width="300"/></p>

Create a new target with Xcode source editor extension. (activate scheme proposed by Xcode)

<p align="center"><img src="/media/new_xcode_extension.png" width="300"/></p>

#### II. Configure.

Open `NSExtension` pair in `Info.plist` file, `XCSourceEditorCommandDefinitions` is an array contains a couple of `item`, an `item` represent a custom command. In each `item`, there are 3 pairs.

1. `XCSourceEditorCommandClassName`: The name of the class which you use to implement `XCSourceEditorCommand` protocol.
2. `XCSourceEditorCommandIdentifier`: The identifier you choose to identify the unique command
3. `XCSourceEditorCommandName`: The name of the command, which will show on Xcode menu.

Here we need to add two new commands: `delete_lines` & `duplicate_lines`.

<p align="center"><img src="/media/edit_plist.png" width="600"/></p>

#### III. Implementation.

Directly dive into `SourceEditorCommand`, implement protocol method:

```
public func perform(with invocation: XCSourceEditorCommandInvocation, completionHandler: (NSError?) -> Swift.Void)
```

Yes, there is only one method to implement. According to the description, this method will:

> Perform the action associated with the command using the information in an invocation. Xcode will pass the code a completion handler that it must invoke to finish performing the command, passing nil on success or an error on failure.

In other words, what you need to do is manipulating a `text buffer` based on the given information for different commands. You'll find all you need inside `invocation` instance. Once you finish, call completion handler, either pass `nil` or `error`.

Please checkout the details of the implementation directly from [code source on Github](https://github.com/wangshengjia/XcodeEditorPlus) which is quite well documented.

#### IV. We are done ! 🎉🎉

Now run your project and test it with Xcode-beta. You'll find a one more gray color Xcode instance is created, that's the one you'll test and debug with.

<p align="center"><img src="/media/debug_xcode.png" width="300"/></p>

In your menu (the gray Xcode's menu), you'll find the commands you've added in `Info.plist`. Just try them.

<p align="center"><img src="/media/use_plugin.png" width="300"/></p>

You can also bind with hot key from Xcode settings.

## Where to go from here?

With Xcode 8 beta 1, there are still bunch of bugs (such as sometimes it just act as disabled), but this is definitely a good beginning. There is also a [WWDC video](https://developer.apple.com/videos/play/wwdc2016/414/) this year which dedicate this new feature, don't hesitate to check it out. 

I've put the [project on Github](https://github.com/wangshengjia/XcodeEditorPlus), if you have any questions or feedback, feel free to file an issue or ping me on Twitter by @wangshengjia.

Enjoy~ 🍻



