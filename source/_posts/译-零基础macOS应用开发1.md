---
title: 【译】 零基础 macOS 应用开发（一）
date: 2017-09-17 18:51:26
tags:
  - 翻译
  - Ray Wenderlich
  - macOS
  - Swift
---

> 本文翻译自 [raywenderlich.com 的 macOS 开发经典入门教程](https://www.raywenderlich.com/151741/macos-development-beginners-part-1)，已咨询对方网站，可至多翻译 10 篇文章。
> 希望各位有英语阅读能力的话，还是~~先打赏然后~~去阅读英文原吧，毕竟无论是 Xcode，抑或是官方的文档，还是各种最前沿的资讯都只有英文版本。

你想要学习开发你自己的 macOS app 吗？

有一个好消息要告诉你！在这个教程中你将会发现，Apple 让开发 macOS app 这件事变得无比简单。你将会学习到如何去创建你的第一个 macOS app——即使你是一个完全的小白。

* 在第一部分中，你将学会如何获取 macOS 开发所需要的工具们。然后，通过创建一个简单的「Hello, World!」app，你将会粗略地认识 Xcode——如何运行一个 app、编辑代码、设计 UI 以及调试你的代码。
* 在第二、三部分中，你将会创建一个复杂的多的「Egg Timer」app，并了解一个 macOS app 的各个组成部分——从一个 app 是如何启动的，到 UI 是如何被构建的，一直到处理交互。

所以你还在等什么呢？桌面级 app 的世界正在等着你！

>**注意：**关于如何开始学习这个系列，这里有一些提示：
>* 如果你从未学习过 Swift，这个系列涉及了一些 Swift 知识，所以请先看看我们的 [Swift 教程](https://www.raywenderlich.com/category/swift)。
>* 如果你已经有过 iOS 的开发经验，第一部分的内容则可以看作一个复习。保险起见，请快速地浏览一下这些内容，然后直接跳转到下一个部分。
>* 这个课程是为完全的小白而准备的——你不需要拥有任何 iOS 或 macOS 开发经验！

# 开始

要成为一个 macOS 开发者，你需要两个东西：

1. 一台运行着 macOS Sierra 的 Mac：macOS 操作系统只能在苹果电脑上运行，因此无论是开发还是运行 macOS app，你都需要一台 Mac。
2. Xcode：这是创建 macOS app 的 IDE（集成开发环境）。稍后你将学会如何在 Mac 上安装它。

当你完成 app 的开发，希望将它上传至 App Store 进行销售时，你需要一个 Apple 的开发者账户。但直到你已经准备好让你的 app 飞向整片世界的蓝天，这都不是一个硬性要求。甚至你已经决定发布你的 app，也只有在你准备通过 Mac App Store 进行销售时才会需要一个开发者账户。如果你已经是一个 iOS 开发者，那你已经搞定了一切——Apple 已经将各种开发者账户融合为一个，因此你只需要一个账户就可以为各种 Apple 设备分发 app。

不同于有些其他平台，为 macOS 开发 app 只需要一个工具：Xcode。Xcode 这个 IDE 包含了创建 macOS、iOS、watchOS 和 tvOS app 时所需要的一切。

如果你还没有安装好 Xcode，点击你 Mac 左上角的 Apple 图标，并选择 **App Store** 来打开 App Store。即便 Xcode 是免费的，你仍需要一个 App Store 的账户来下载 Xcode。

![](http://upload-images.jianshu.io/upload_images/5477931-e82075190fa370a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在请搜索 Xcode，然后点击**获取**按钮来开始下载。当它下载并安装好（这可能得花点时间，它不是个小软件）之后，你可以在 **应用程序**文件夹中打开它。

![](http://upload-images.jianshu.io/upload_images/5477931-1352711309fe2cd7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# Hello World!

遵循长久以来学习一门新编程语言或平台的传统，你将会学习如何为 macOS 开发一个「Hello World!」app。

如果 Xcode 现在还没打开，请打开它。你会看到一个「Welcome to Xcode」的窗口。如果你没有看到这个窗口。点击 **Window** 菜单中的 **Welcome to Xcode**。

点击 **Create a new Xcode project** 并在接下来的对话框中，在顶部的标签中选择 **macOS**，然后在 **Application** 部分中选择 **Cocoa Application**，并点击 **Next**。

![](http://upload-images.jianshu.io/upload_images/5477931-4247ae116c5942d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

给你的 app你取个名字——**Hello World**，并确保 **Language** 设置为 **Swift**，以及 **Use Storyboards** 被勾选。其他选项全部取消勾选。

![](http://upload-images.jianshu.io/upload_images/5477931-0b8c71ba1aa3d4c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击 **Next** 和 **Create** 来保存你的新项目。

# 运行你的 app

Xcode 已经为你的 app 应用了一个基础的模版，并添加了其运行所需要的所有文件。因此，什么都不用做而直接运行它其实还挺有趣的。

点击工具栏上的 **播放按钮️** 或使用键盘快捷键 **⌘R** 来运行你的 app。Xcode 现在会将所有的代码编译成机器语言，将 app 运行所需要的所有资源文件打包，然后运行它。

![](http://upload-images.jianshu.io/upload_images/5477931-9eacc5f13d7eaaf0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> **注意：**当你第一次在 Xcode 中运行一个 app 的时候，你可能会被询问是否要 **Enable Developer Mode on this Mac（在这台 Mac 上启用开发者模式）**，请放心地点击 **Enable**，这可能会要求你输入 Mac 的开机密码。开发者模式允许 Xcode 附加调试器到你的 app 的进程——这对于开发一个 app 来说极其有用！


你现在应该能看到一个空白的窗口了，但请不要灰心——你现在已经能做到这些了：

* 这个窗口可以调整大小、最小化和全屏；
* 这个 app 拥有已经拥有了一套完整的菜单选项，这其中有很多都已经可以正常使用而不用你做任何事情。
* Dock 栏上的 app 图标也拥有了常见的菜单。

![](http://upload-images.jianshu.io/upload_images/5477931-e0c710c4703481ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是现在是时候来把这个空空的屏幕变得更有趣了，所以请退出这个 app 并返回到 Xcode。

# Xcode 的界面

Xcode 将许多功能融合在同一个窗口中，因此许多东西并不能同时显示出来。要想成为一个 Xcode 高手，你需要知道你需要的功能在哪里，以及如何打开它们。

当你在 Xcode 里打开一个新的工程，你已经拥有了一个带有工具栏和三个主要面板的窗口。

![](http://upload-images.jianshu.io/upload_images/5477931-86776d3b5504b44a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

左边的窗口是 **Navigator（导航器）**，在这个面板的顶部有八个标签将它分成了八个部分。你所最常用到的是第一个——**Project（项目）**——它列出了项目中所有的文件，你可以点击这些文件来编辑它们。

中间的面板是 **Editor（编辑器）**，这里会显示你在 **Project Navigator（项目导航器）**里选择的文件。

右侧的面板是 **Utilities（工具集）**面板，这里显示的内容会会随着你在 **Editor** 里进行的操作而变化。

# 添加 UI

你可以使用 Storyboard 来设计 app 的 UI（用户交互界面）。你的 app 已经有了一个 storyboard，所以在 **Project Navigator（工程导航器）** 中点击 **Main.stroyboard** 来将之显示在编辑器中。

你的屏幕自动切换了，是不是很神奇！在编辑器中，你现在可以看到文档的线框图，以及可视化的 UI 编辑器。

看一看你可视化编辑器中的东西。这儿有三个主要的区域，每一个都有着一个文本占位符：

* **Application Scene**：顶部的菜单栏；
* **Window Controller Scene**：配置窗口会有怎样的表现；
* **View Controller Scene**：放置 UI 控件的地方。

在 **Utilities** 面板里，你将会看见上半部分有八个标签页，而下半部分有四个标签页
下半部分的区域呈现了你可以插入到你的项目里的东西。现在你需要插入 UI 控件，所以点击第三个图标 **Object Library**。

在底部的过滤器（输入框）中，输入「text」，然后找到并拖动一个 **Text Field** 到你的 **View Controller Scene**。

![](http://upload-images.jianshu.io/upload_images/5477931-7bb8e8a9ba23fbab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在搜索「button」并拖动一个 **Push Button** 到 **View Controller Scene** 中。最后，再添加一个 **Label**。

现在，点击**播放按钮️**或按下 **⌘R** 来编译并运行你的 app，你就能看到这三个 UI 元素了。试着在输入框中打些字——它现在已经支持所有标准的键盘快捷键了：复制、粘贴、剪切、撤销、重做…但是界面上的按钮还是什么都做不了，文本也只显示了一个「Label」，所以现在我们该把它们连接起来。

![](http://upload-images.jianshu.io/upload_images/5477931-1d03dc6fde4e0fcc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 配置 UI

返回 **Main.storyboard** 并点击界面上的 Button，在右侧的 **Utilities** 面板中，点击第四个标签页 **Attributes Inspector**。

把按钮的标题改为「Say Hello」，此时的按钮可能不够宽，所以请点击菜单栏上的 **Editor**，然后点击 **Size to Fit Content**。（如果 Size to Fit Content 选项不可用，先点击空白处取消选中这个按钮，再重新选择它，然后再试一次）。

![](http://upload-images.jianshu.io/upload_images/5477931-809c555b71a331f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在点击并选中 Text Field。在这个 app 中，用户将在这里输入他们的名字，当他们点击按钮时，app 会显示「Hello」+他们的名字。为了提示用户，我们在 **Attributes Inspector** 中为输入框中添加一个占位符。

把输入框稍微拖大一些，以便能放下一些较长的名字，然后把按钮拖动到它的右边。当你在 **View Controller Scene** 中拖动元素时，会出现一些蓝色的虚线来帮助你根据 Apple’s Human Interface Guidelines（Apple 人机交互指南）对齐并放置元素。

![](http://upload-images.jianshu.io/upload_images/5477931-9dd6729b3c989e3f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

把 Label 放置在输入框的下方。因为这个 Label 十分重要，我们把它的字体改大一些，选中这个 Label，在 **Attributes Inspector** 中把 **Font** 更改为 **System Regular 30**。

![](http://upload-images.jianshu.io/upload_images/5477931-4b5f1a07bdc7cff3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不如来些更刺激的吧！把文字的颜色改成红色。

![](http://upload-images.jianshu.io/upload_images/5477931-1bb391c64ee5700a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

你不知道用户的名字会有多长，所以我们需要重新调整输入框的大小，以适应字体的高度，并把宽度调整为和窗口的宽度差不多。

编译并运行你的 app 来检查一下你的 UI 设置是否生效了。一旦你觉得 Label 中的文字效果令你满意，把 Label 的 **Title** 删除，这样一来就清空了 Label 里的文字。

![](http://upload-images.jianshu.io/upload_images/5477931-5bda3bf99a4a06b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 连接 UI 与代码 

你的 app 现在无法按照你预期的那样工作，你仍需要添加代码来让你的代码能与 UI 通讯。为了建立这种联系，你需要使用 Xcode 的 **Assistant Editor**，先在导航器中选中 **Main.storyboard**，按住 Option⌥ 键的同时点击 **ViewController.swift**，这个操作将会在不关闭之前打开的文件的情况下，为 ViewController.swift 打开第二个编辑器面板。

现在屏幕上的东西可能看起来有些拥挤（当然这也取决于你的显示器尺寸），所以点击工具栏最右上方的按钮来隐藏 Utilities 面板。如果你的显示器空间还是不够，把 Navigator 面板也一并隐藏了。

![](http://upload-images.jianshu.io/upload_images/5477931-63a7464df5d8e17f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选中输入框，按住 Control⌃ 键的同时把文本框拖动到 `class ViewController : NSViewController {` 和   `override func viewDidLoad() {` 之间的那一行，松开鼠标时会弹出一个小窗口，在「name」中输入 `nameField`，然后点击 **Connect**。

![](http://upload-images.jianshu.io/upload_images/5477931-09098b371d0e4b48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对 Label 做同样的事情，并把它命名为 `helloLabel`。
看一看 Xcode 自动生成的代码，你将会看见它们都以 `@IBOutlet` 开头，这是「Interface Builder Outlet」的缩写，也就是告诉 Storyboard 编辑器这些对象的名称是如何与视觉元素关联起来的。

对于那个按钮，代码中不需要给它起名字，但它需要知道用户何时点击了这个按钮。与 `@IBOutlet` 不同，这种连接叫做 `@IBAction`。

和先前一样，选中按钮，按住 Control⌃ 键的同时把文本框拖动到 **ViewController.swift** 中。这一次把 **Connection** 选项设置为 **Action**，并把 **name** 设置为 `sayButtonClicked`。这将会创建一个按钮被电击时会调用的一个方法。

![](http://upload-images.jianshu.io/upload_images/5477931-a1dcd7e8ada4cb90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在万事俱备，只差代码了！点击右侧编辑器面板右上角的「×」关闭 **Assistant Editor**，回到 **ViewController.swift**。如果你隐藏了 **Navigator**，点击右上角的按钮或按下 **⌘1** 来直接跳转到 **Project Navigator**。

把如下代码输入到 `sayButtonClicked` 方法中：

```swift
var name = nameField.stringValue
if name.isEmpty {
	name = "World"
}
let greeting = "Hello \(name)!"
helloLabel.stringValue = greeting
```

在删除了顶部自动生成的版权信息后，完成了的 **ViewController.swift** 中的代码看起来应该像下边截图中的一样。行号左边的小气泡表明这是一个 UI 与代码的连接点。

![](http://upload-images.jianshu.io/upload_images/5477931-232d57854832c0e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在编译并运行你的 app。

什么也要不输入，并点击 **Say Hello** 按钮，你将会看到「Hello World!」；而输入了你的名字之后再点击，就能看到你专属的问候语了。

![](http://upload-images.jianshu.io/upload_images/5477931-796ee8240876fbd2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 调试

有时，我们开发者会犯错——相信我，真的会的。当我们犯了错，就需要调试我们的代码。Xcode 允许我们在代码的任意一处暂停，然后一行一行地运行代码，让你检查每一行运行后各个变量的值，以此来找到错误。

在  **ViewController.swift** 中找到 `sayButtonClicked`，点击  `var name =`  左边的行号，一个蓝色的小旗子将会出现，这是一个激活了的断点，当你点击你 app 界面上的按钮时，调试器会让程序在这里暂停。再次点击它，它会变成浅浅的蓝色，这是一个未激活的断点，它将不会暂停代码，也不会启动调试器。要彻底移除一个断点，把它从行号那一条中拖出去即可。

![](http://upload-images.jianshu.io/upload_images/5477931-7286002933ce6d55.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再次添加一个断点并运行你的 app，点击 **Say Hello** 按钮，Xcode 会移动到最顶层，并把有断点的那一行高亮显示。在 **Editor** 面板的最底部，将会有两个部分：**Variables（变量）**和 **Console（控制台）**。**Variables** 部分展示了此函数中用到的所有变量以及  `self`（也就是 View Controller）和 `sender`（也就是按钮）。

![](http://upload-images.jianshu.io/upload_images/5477931-b02096c9aed0c4c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 **Variables** 部分的上方有几个控制调试器的按钮。请把鼠标挨个放在这些按钮上，根据弹出的工具提示看看他们是做什么的。点击 **Step Over** 按钮来运行下一行代码。

在 **Variables** 部分中，你可以看到 `name` 是一个空的字符串，所以再连续点击 **Step Over** 按钮两次，调试器会移动到 `if`语句中，并把 `name` 变量设置为「World」。

在 **Variables** 部分中选中 `Name` 变量，点击下方的 **Quick Look** 按钮来查看具体的内容，电击 **Print Description** 按可以把它的信息输出到 **Console** 部分中。如果「World」没有被正确地设置，你应该能在这里看到，并想出对应的对策。

![](http://upload-images.jianshu.io/upload_images/5477931-1d3489b617788b79.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当你检查完所有的变量内容后，点击 **Continue program execution** 按钮来停止调试，并让程序继续运行。你可以使用界面右上角的按钮来隐藏调试器。

# 图像

除了代码和 UI，你的 app 还需要一些图像。根据屏幕的不同（Apple 设备上分为 Retina 显示屏和非 Retina 显示屏），你经常需要为一套「资产」提供多个版本。为了简化这个流程，Xcode 使用 **Asset Libraries** 来存储和管理这些 app 需要的资产。

在 **Project Navigator** 中点击 **Assets.xcassets**，到目前为止里边只有一个 **AppIcon**，它包含了不同分辨率下的 app 的图标。点击 **AppIcon**，你会看到它需要 10 个不同的图片来覆盖所有的情况，但如果你只提供了一个，那么 Xcode 会尽量让它发挥最大功效，但这并不是一个正确的做法——你需要尽力为你的 app 提供所有尺寸的图标，但在这个简单的上手教程中，一个图标就够了，当你真正开始制作自己的 app 时，这些图标最好一个都别少。

下载 512×512 像素的[示例图标](https://koenig-media.raywenderlich.com/uploads/2017/02/rw_logo.png.zip)，把它拖动到 **Mac 512pt 1x** 的方框中。

![](http://upload-images.jianshu.io/upload_images/5477931-9813e1e8d75fb779.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

编译并运行你的 app，查看 Dock 栏中的图标，如果你看到的还是默认图标，请在 **Product** 菜单中选择 **Clean**，然后再次运行。

![](http://upload-images.jianshu.io/upload_images/5477931-f51cb8591231f806.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 获得帮助

除了一个编辑器，Xcode 还包含了你编写 macOS app 所需要的所有文档。

在 **Help** 菜单中选择 **Documentation and API Reference**。搜索 **NSButton**，请确保当前选中的语言是 Swift，只需点击顶部的搜索结果，就可以找到所有关于按钮的信息了。

![](http://upload-images.jianshu.io/upload_images/5477931-7508d1e9c3fe8bd7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

还有一种方法，可以在你的代码中直接查看相关的文档，返回 **ViewController.swift**，找到 `sayButtonClicked` 所在的行，按住 Option⌥ 键的同时点击 `stringValue`，一个快速的查询面板将会弹出，在底部还有一个 **Property Reference** 的链接，点击它就可以在文档中查看更多信息。

**Option⌥ +鼠标点击**是一个特别好的学习方式，你甚至可以为你自己编写的函数添加自己编写的文档，以便今后快速查阅。

**Help** 菜单中还包括了 **Xcode Help**，在这里你可以了解到更多的 Xcode 信息。
