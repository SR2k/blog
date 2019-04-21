---
title: 【译】如何在 macOS 上使用 NSTouchBar
date: 2017-10-10 00:01:11
tags:
  - 翻译
  - Ray Wenderlich
  - macOS
  - Swift
---

> 本文翻译自 [raywenderlich.com](https://www.raywenderlich.com/) 的 [How to Use NSTouchBar on macOS](https://www.raywenderlich.com/147118/use-nstouchbar-macos)，已咨询对方网站，可至多翻译 10 篇文章。
> 各位若有英语阅读能力的话，还是~~先打赏然后~~去阅读英文原吧😉。
> 综上，此翻译版本仅供参考，**谢绝转载**。

![](http://upload-images.jianshu.io/upload_images/5477931-52bd2e529f437384.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 对了我跟着这个教程敲代码的时候发现文中所有的 `@available(OSX 10.12.1, *)` 其实应为 `@available(OSX 10.12.2, *)`，但是出于对原文的尊重没有修改，请各位注意～

等了好久好久~~终于等到今天~~之后，Apple 终于发布了新款的 MacBook Pro，它最惹人瞩目的应该就是那块（小小的）触屏了吧。

新款设备用全新的 Touch Bar 替代了原有的功能键，它们可扩展、支持多点触控，更重要的是，Touch Bar 对开发者们完全开放，这意味着你的 macOS app 可以获得一种全新的交互方式。

如果你是一个 macOS 开发者，你一定很希望自己的 app 能够立刻使用上这项前沿科技。在这个教程中，我会将向你们展示如何使用全新的 `NSTouchBar` API 来为你的 macOS app 创建一个动态的 Touch Bar。

> **注意：**这个教程需要 Xcode 8.1 或更高版本以及 macOS 10.12.1 (Build 16B2657) 或更高版本，否则的话你将无法运行 Touch Bar 模拟器。你可以在 **** → **关于本机**里点击数字版本号来查看 build 版本。
> ![](http://upload-images.jianshu.io/upload_images/5477931-f1e49c156d4e8aba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 如果你的版本不够，你可以在 [Apple 的网站上](https://support.apple.com/kb/dl1897) 下载。

# Touch Bar 是个啥？

如上文所述，Touch Bar 是一块安装在键盘上方的细长形的触摸屏，它允许用户使用一种全新的方式来与 app（以及 Mac）进行交互。

![](http://upload-images.jianshu.io/upload_images/5477931-1920310de9ddfda1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 Touch Bar 上有三个默认的部分：

* *系统按钮*：根据运行的 app，这里将会显示一个系统级的按钮，比如 esc；
* *App 区域*：你的 app 可以控制的显示区域，也就是我们的主舞台；
* *控制条*：这里用于显示你熟悉的控制按钮，比如亮度、音量等。

和其它许多 Apple 的新科技一样，Touch Bar 也拥有自己的《人机交互则例（Human Interface Guidelines）》，为了你的 app 和其它 Mac app 拥有统一的用户体验，你应当遵循这份则例。你可以[点击这里](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/OSXHIGuidelines/AbouttheTouchBar.html)阅读它。

概括说来，这份则例中比较重要的几点有：

* **App 中的某个功能不应该只能在 Touch Bar 中使用**：即使部份用户还未升级到最新的硬件，你的 app 也应该尽量为他们提供一样的功能。如果你决定为 Touch Bar 加入某些功能，请确保这个功能也能在 app 的其他某处也可以访问到。另外 Touch Bar 是可以被用户禁用的，所以也别太指望你的用户能一直看到它；
* **Touch Bar 是键盘的延伸，而不是一个显示屏**：诚然，Touch Bar 是一个显示屏，但是它不是显示器的延伸，而是一个与 app 交互的窗口。你不应该在 Touch Bar 上用滚来滚去的内容或 blingbling 的警告打扰用户的视线；
* **快速响应**：用户在键盘上按下一个真实的按键时，按键会立即给予一个反馈（也就是被按下去了）。同理，用户在 Touch Bar 上按下一个虚拟按钮时，你也应该给他们提供即时的反馈。

# 如何让你的 app 支持 Touch Bar

要让你的 app 支持 Touch Bar，你需要使用 Apple 提供的两个类：`NSTouchBar` 和  `NSTouchBarItem`（当然还有他们的子类）。

某些 `NSTouchBarItem` 的子类提供了这些功能：
* **Slider**：滑动调节某个值；
* **Popover**：把更多功能藏入一个二级菜单中；
* **Color Picker**：和名字一样，用来选取颜色的咯🤷‍♀️；
* **Custom**：这个子类是你的天下，你可以在它的里面塞入文本、按钮以及其他各种各样的控件。

从文字大小颜色到图片内容，你可以随意自定义你的 item，从而为你的用户提供一个传统键盘无法提供的、更加牛×的交互方式，但是请时刻请谨记《人机交互则例》。现在我们要开始动工啦。

# 准备开始

在开始敲代码之前，请先[点击这里](https://koenig-media.raywenderlich.com/uploads/2016/10/TouchBar-Starter-2.zip)下载初始项目的源代码。

我们要编写的 app 是一个简单的旅行记录 app。打开初始项目，如果你的设备不支持 Touch Bar，请点击 Xcode 菜单栏上的 **Window** → **Show Touch Bar**，Touch Bar 的模拟器就会出现在屏幕上。

![](http://upload-images.jianshu.io/upload_images/5477931-1ad812514b33ca1d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

编译并运行你的 app，你将会看到 Touch Bar 上除了 esc 按钮和控制条以外空空如也。

![](http://upload-images.jianshu.io/upload_images/5477931-1c112d10fa6ba8b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/5477931-eb56c213cf6eb050.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们要做的第一步是告诉系统我们的 app 需要自定义 Touch Bar。打开 **AppDelegate.swift**，将这些代码添加到 `applicationDidFinishLaunching(_:)` 方法中：

```swift
func applicationDidFinishLaunching(_ aNotification: Notification) {
  if #available(OSX 10.12.1, *) {
    NSApplication.shared().isAutomaticCustomizeTouchBarMenuItemEnabled = true
  }
}
```

这些代码将会帮你搞定启用 Touch Bar 所需要的各种操作，（在写这篇文章的时候）Xcode 还没有 macOS 10.12.1 的配套 SDK，所以你需要在 Touch Bar 相关的代码周边添加 `#available(OS X 10.12.1, *)`，当然如果你忘了这件事，Xcode 会给你一个温馨的提醒😉。

打开 **WindowController.swift**，找到 `makeTouchBar()`，这个方法用于检测 `ViewController` 是否含有一个可以被返回的 Touch Bar，如果有，它会把这个 Touch Bar 返回给 **Window**，然后呈现给用户。现在，我们还没有创建 Touch Bar，所以什么也不会发生。

在你开始创建自己的 Touch Bar 和 Touch Bar Item 之前，你需要注意这些类都需要独一无二的 identifier（标识符），打开 **TouchBarIdentifiers.swift**，你将能看到两个扩展定义了一些标识符：`NSTouchBarCustomizationIdentifier` 和 `NSTouchBarItemIdentifier`。

前往 **ViewController.swift**，并把这些带码添加到文件最后 `// MARK: - TouchBar Delegate` 注释的后方：

```swift
@available(OSX 10.12.1, *)
extension ViewController: NSTouchBarDelegate {
  override func makeTouchBar() -> NSTouchBar? {
    // 1
    let touchBar = NSTouchBar()
    touchBar.delegate = self
    // 2
    touchBar.customizationIdentifier = .travelBar
    // 3
    touchBar.defaultItemIdentifiers = [.infoLabelItem]
    // 4
    touchBar.customizationAllowedItemIdentifiers = [.infoLabelItem]
    return touchBar
  }
}
```

在此处，通过重写 `makeTouchBar()` 方法，你给你的 View 或 Window 创建了一个 Touch Bar，在这个方法中：

1. 创建一个新的 `TouchBar` 并设置它的 delegate；
2. 设置 customizationIdentifier（自定义标识符），每个 `TouchBar` 和 `TouchBarItem` 都需要一个独一无二的标识符；
3. 设置 Touch Bar 的默认 item 标识符，这将会告诉 Touch Bar 它将会容纳哪些 item；
4. 这一步是允许用户可以自定义的 item。作为参考，你可以打开 Finder，点击菜单栏上的 **显示** → **自定 Multi-Touch Bar**看看自定义 Touch Bar 是什么效果。

接下来，我们还需要设置 `.infoLabelItem` 是什么样子的，在同一个 extension 中添加：

```swift
func touchBar(_ touchBar: NSTouchBar, makeItemForIdentifier identifier: NSTouchBarItemIdentifier) -> NSTouchBarItem? {
    switch identifier {
    case NSTouchBarItemIdentifier.infoLabelItem:
      let customViewItem = NSCustomTouchBarItem(identifier: identifier)
      customViewItem.view = NSTextField(labelWithString: "\u{1F30E} \u{1F4D3}")
      return customViewItem
    default:
      return nil
    }
}
```

通过实现`touchBar(_:makeItemForIdentifier:)` 方法，你可以自定义你的 Touch Bar Item，在段代码里，你创建了一个 `NSCustomTouchBarItem`，并把它的 `view` 设置为了 `NSTextField`。

编译并运行你的 app，你可以看到 Touch Bar 多了一个 item。 

![](http://upload-images.jianshu.io/upload_images/5477931-96245a90a25d22dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

耶✌️！通过一番努力你得到了……两个 emoji…好吧，现在我们来添加一些别的控件。

![](http://upload-images.jianshu.io/upload_images/5477931-88d35baba7596185.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# Text Field 和 Scrubber

在 `makeTouchBar()` 里，把 `touchBar.defaultItemIdentifiers = [.infoLabelItem] ` 修改为：

```swift
touchBar.defaultItemIdentifiers = [.infoLabelItem, .flexibleSpace, .ratingLabel, .ratingScrubber]
```

这些代码会让 Touch Bar 显示三个 item：一个 label、一个 scrubber 以及一个 `.flexibleSpace`。这是一个动态的间距，它可以把各种 item 按组进行整洁的分类。此外你还可以使用 `.fixedSpaceSmall` 和 `.fixedSpaceLarge` 这两个固定间距。

和之前的那个 label 一样，你也需要自定义这些 item，把这些 `cases` 添加到 `touchBar(_:makeItemForIdentifier:)` 里的 `switch`：

```swift
case NSTouchBarItemIdentifier.ratingLabel:
  // 1
  let customViewItem = NSCustomTouchBarItem(identifier: identifier)
  customViewItem.view = NSTextField(labelWithString: "Rating")
  return customViewItem
case NSTouchBarItemIdentifier.ratingScrubber:
  // 2
  let scrubberItem = NSCustomTouchBarItem(identifier: identifier)   
  let scrubber = NSScrubber()
  scrubber.scrubberLayout = NSScrubberFlowLayout()
  scrubber.register(NSScrubberTextItemView.self, forItemIdentifier: "RatingScrubberItemIdentifier")
  scrubber.mode = .fixed
  scrubber.selectionBackgroundStyle = .roundedBackground
  scrubber.delegate = self
  scrubber.dataSource = self
  scrubberItem.view = scrubber
  scrubber.bind("selectedIndex", to: self, withKeyPath: #keyPath(rating), options: nil)    
  return scrubberItem
```

一步一步来看：

1. 为「评分」新建了一个新的 label item；
2. 然后创建一个新的 item 用来展示 `NSScrubber`。这是一个 Touch Bar 特有的控件，它允许你在若干个项目中进行选择。Scrubber 需要一个代理来处理事件，你需要做的是设置一个 `delegate`（初始项目的源代码已经在 **ViewController** 里实现过这个协议了）。

编译并运行你的 app，你将会看到 Touch Bar 里多出了两个新的 item，当你点击某个 scrubber 里的项目后，在 app 的主窗口里能看到数值的调整。

![](http://upload-images.jianshu.io/upload_images/5477931-de1447818a28d7be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# Segmented Control

接下来，我们来添加一个 Segmented Control。这个控件没有使用 Delegate 模式，因此这刚好可以让你体验一下怎么设置 Touch Bar 里的 **Target-Action（目标动作）**。回到 `makeTouchBar()` 中，把这三个 item 添加到 `defaultItemIdentifiers` 里：

```swift
touchBar.defaultItemIdentifiers = [.infoLabelItem, .flexibleSpace, .ratingLabel, .ratingScrubber, .flexibleSpace, .visitedLabelItem, .visitedItem, .visitSegmentedItem]
```

以及把最后的三个 `case` 添加到 `touchBar(_:makeItemForIdentifier:)` 中：

```swift
case NSTouchBarItemIdentifier.visitedLabelItem:
  // 1
  let customViewItem = NSCustomTouchBarItem(identifier: identifier)
  customViewItem.view = NSTextField(labelWithString: "Times Visited")
  return customViewItem
case NSTouchBarItemIdentifier.visitedItem:
  // 2
  let customViewItem = NSCustomTouchBarItem(identifier: identifier)
  customViewItem.view = NSTextField(labelWithString: "--")
  customViewItem.view.bind("value", to: self, withKeyPath: #keyPath(visited), options: nil)
  return customViewItem
case NSTouchBarItemIdentifier.visitSegmentedItem:
  // 3
  let customActionItem = NSCustomTouchBarItem(identifier: identifier)
  let segmentedControl = NSSegmentedControl(images: [NSImage(named: NSImageNameRemoveTemplate)!, NSImage(named: NSImageNameAddTemplate)!], trackingMode: .momentary, target: self, action: #selector(changevisitedAmount(_:)))
  segmentedControl.setWidth(40, forSegment: 0)
  segmentedControl.setWidth(40, forSegment: 1)
  customActionItem.view = segmentedControl
  return customActionItem
```

一步一步看：

1. 和之前一样，创建一个简单的 Label；
2. 创建另一个 Label，但这一次使用 **bind** 来把 Label 要显示的文本绑定到一个属性上；
3. 最后，创建一个 Segmented Control，并设置它的 action。你可以看到，为它设置一个 action 和其他常见控件是完全一样的。

编译并运行，除了 Scrubber，你现在还可以和 Segmented Control 交互了。此外，在 Touch Bar 里修改一个数值，app 的主窗口中也会显示出来，反之亦然。

![](http://upload-images.jianshu.io/upload_images/5477931-a7c5438cf524b6a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 彩色按钮

能让用户使用 Touch Bar 来进行「保存」操作是一个不错的点子，因为这个按钮和别的按钮都不一样，我们可以把它设置成绿色的。你可以使用 `NSButton` 的 `bezelColor` 属性来给它设置一个特殊的颜色。

打开 **TouchBarIdentifiers.swift**，在 `NSTouchBarItemIdentifier` extension 里，添加这些代码：

```swift
static let saveItem = NSTouchBarItemIdentifier("com.razeware.SaveItem")
```

这将会从头开始创建一个标识符，以此允许你在 Touch Bar 里添加一个 item。

返回 **ViewController.swift**，添加一个新的 `.flexSpace` 和 `.saveItem`  到 Touch Bar 的 `defaultItemIdentifiers` 里：

```swift
touchBar.defaultItemIdentifiers = [.infoLabelItem, .flexibleSpace, .ratingLabel, .ratingScrubber, .flexibleSpace, .visitedLabelItem, .visitedItem, .visitSegmentedItem, .flexibleSpace, .saveItem]
```

基本上完成了，最后一步是对按钮进行一些设置。在 `touchBar(_:makeItemForIdentifier:)` 中添加最后一个 `case`：

```swift
case NSTouchBarItemIdentifier.saveItem:
  let saveItem = NSCustomTouchBarItem(identifier: identifier)
  let button = NSButton(title: "Save", target: self, action: #selector(save(_:)))
  button.bezelColor = NSColor(red:0.35, green:0.61, blue:0.35, alpha:1.00)
  saveItem.view = button
  return saveItem
```

通过 `bezelColor`，你已经把这个按钮成功地修改成了绿色。

编译并运行，你会看到 Touch Bar 上有了一个绿色的按钮，它和窗口中的 **Save** 按钮拥有一样的功能。

![](http://upload-images.jianshu.io/upload_images/5477931-c9f44cf8bb30f2df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 现在该做些啥？

你可以[点击这里](https://koenig-media.raywenderlich.com/uploads/2016/10/TouchBar-Final-3.zip)下载最终完成的项目。 

这些就是 Touch Bar 的基础了。显然，Apple 希望 Touch Bar 的开发过程越简单越好，因此你可以尽快把这些特性尽快地带给新 MacBook Pro 的用户。

在这个教程中，你学到了：

1. 如何设置你的 app，让它能显示 Touch Bar；
2. 如何在 Touch Bar 里显示静止的 Label；
3. 如何使用 binding（绑定）来添加一个动态的 Label；
4. 如何在 Touch Bar 中添加控件，并处理它们的事件。

请不要在此停下！`NSTouchBar` 和 `NSTouchBarItem` 中还有很多值得探索的特性。你可以试着在 Touch Bar 里添加一个 Popover，或者也可以试试在你的 app 里格式化一个文本，你还可以在试着去在 Interface Builder 里创建一个 Touch Bar。

[How to Use NSTouchBar on macOS](https://www.raywenderlich.com/147118/use-nstouchbar-macos)