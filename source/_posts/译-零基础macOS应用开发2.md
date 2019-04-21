---
title: 【译】 零基础 macOS 应用开发（二）
date: 2017-09-22 04:39:26
tags:
  - 翻译
  - Ray Wenderlich
  - macOS
  - Swift
---

> 本文翻译自 [raywenderlich.com 的 macOS 开发经典入门教程](https://www.raywenderlich.com/151741/macos-development-beginners-part-1) ，已咨询对方网站，可至多翻译 10 篇文章。
> 还是~~先打赏然后~~去阅读英文原吧，毕竟无论是 Xcode，抑或是官方的文档，还是各种最前沿的资讯都只有英文版本。  
> 综上，此翻译版本仅供参考，**谢绝转载**。 

欢迎回到我们的零基础 macOS 应用开发教程的第二部分（共三部分）！

在第一部分（[原文](https://www.raywenderlich.com/151741/macos-development-beginners-part-1) / [译文](http://www.jianshu.com/p/a3f16178a213)），你了解了如何安装 Xcode、如何创建一个新的 app、添加 UI、连接代码与 UI、运行 app、调试 app，以及寻求帮助。如果你对上述内容还有任何不确定之处，再回去浏览一遍第一部分吧。

在这一部分，你将会创建一个界面更加复杂的 app。你将会学到如何应对可调整大小的窗口，以及设计第二个页面——偏好设置页面，并让你的 app 能跳转到这个新创建的页面。

# 准备开始

和第一部分一样，打开 Xcode 并在欢迎页面点击 **Create a new Xcode project**，或选择 **File** 菜单中的 **New Project…**。接下来在 **macOS** 下的 **Application** 标签页中选择 **Cocoa Application**，并点击 **Next**，把你的 app 命名为 **EggTimer**，确保 Language 选项为 **Swift** 以及 **Use Storyboards** 被选中。点击 **Next** 然后找一个合适的地方存储这个项目。

编译并运行你的 app 来确保一切正常。

# EggTimer App

你将会开发的 app 叫做 **EggTimer**，它能帮助用户倒计时，并显示剩余的时间。App 的界面上会有一个鸡蛋的图标，随着时间的临近，鸡蛋会被慢慢煮熟；当你的鸡蛋煮熟时，还会有一个提示音。App 内还有一个页面会显示此 app 的偏好设置。

![](http://upload-images.jianshu.io/upload_images/5477931-08716dfcefbc0d2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 **Project Navigator（项目导航器）**中打开 **Main.storyboard**，正如第一部分所说的，你已经拥有了三个场景：

* **Application Scene（应用场景）**包含了只要 app 运行着就会显示的菜单栏。
* **Window Controller（窗口控制器）**是定义了 app 窗口有怎样行为（怎么调整大小、窗口怎么出现、app 是否会记住上次调整的大小和位置等等）的部分；其实一个 Window Controller 可以管理多个窗口，但如果它们的属性不同，你就需要多个 Windows Controller 了。
* **View Controller** 则显示了窗口中的用户界面——也就是你对 UI 进行布局的地方。

你注意到了吗？有一个箭头指向了 **Window Controller**，这表明它是 app 启动时的 initial display（初始页面）。你可以在 **Document Outline（文档大纲）**中选择 **Window Controller**，然后在 **Attributes Inspector（属性检查器）**中查看这个设置。取消勾选 **Is Initial Controller（用作首要控制器）**后，箭头就消失了。因为你需要它来作为首要的控制器，请把这个选项勾选上。

# Window Controller（窗口控制器）

在你开始着手 UI 前，请确保你已经在 **Project Navigator（项目导航器）**中选择了 **Main.storyboard**。点击来选择其中的 window（窗口）。在可视化编辑器中，Window Controller显示了「View Controller」这行字，因为它包含着一个 View Controller（视图控制器）。在这个 app 中，我们不希望用户将窗口调整到 346 × 471 像素以下，这两个数值也将会是 app 启动时窗口的默认大小。

![](http://upload-images.jianshu.io/upload_images/5477931-19c468e3d35e28ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 **Utilities（工具集）**面板中前往 **Size Inspector（尺寸检查器）**，设置 **Content Size Width（内容宽度）**和 **Content Size Height（内容高度）**分别为 346；勾选 **Minimum Content Size（最小内容尺寸）**的复选框，并确保其数值与内容尺寸相同。现在，在可视化编辑器里的 Window Controller 的尺寸应该已经发生了变化，因此可能会盖住其他元素，你可能需要重新排列一下它们。

虽然不是必须的，但当你把 View Controller 和承放它的 Window Controller 调整到一样的大小后，后续操作会直观很多。点击并选中 **View Controller**，在 **Size Inspector（尺寸检查器）**中设置 Width 和 Height 分别为 346 和 471。如果需要的话，重新排列一下界面上的各个元素防止它们重叠。现在可视化编辑器里的 Window Controller 和 View Controller 都有着一样的尺寸了。

选中 WindowController 里的 Window，在 **Attributes Inspector（属性检查器）**中把它的名字更改为 **Egg Timer**，设置 **Autosave name** 为 **EggTimerMainWindow**，这样用户上次调整的窗口的尺寸和大小在 app 下次启动时就都会被记住了。

![](http://upload-images.jianshu.io/upload_images/5477931-c7d85f2f0d6d0904.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果你是一个 iOS 开发者，你应该已经处理过不同设备、屏幕尺寸、屏幕旋转方向下的适配问题了。在 macOS 开发过程中，你必须处理无限多种窗口尺寸和长宽比，因此我在这里把窗口的默认尺寸调整成了这么一个奇怪的数字。不过好消息是，Auto Layout 帮助你解决了这个问题。

# 布局 UI（一）

基本的 UI 包含了两个 Stack View（堆叠视图）：第一部分包含显示剩余时间的 Label 和鸡蛋的图标。第二个则在底部包含三个按钮，我们先来制作按钮：

* 在 Object Library（控件库）中搜索「Button」
* 向 View Controller 中拖动一个 Gradient button
* 使用 Attributes Inspector（属性检查器）删除它的图像，并把它 Title 设置为 Start
* 把字体改成 System 24
	![](http://upload-images.jianshu.io/upload_images/5477931-4ae1393f269f082a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 把按钮拖大一些让文字能显示完全
* 选中 Start 按钮，按下 **Command⌘ + D** 两次来复制两份
* 把新复制的按钮向右拖动一些，并把他们的 Title 改成 Stop 和 Reset
* 同时选中三个按钮，然后点击菜单栏上的 **Editor** → **Embed In** → **Stack View**
	![](http://upload-images.jianshu.io/upload_images/5477931-f6aea6afee6f6ab0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

为了让按钮填满整个 Stack View，选择新创建的 **Stack View**，然后在 **Attributes Inspector（属性检查器）**里做出如下更改：

![](http://upload-images.jianshu.io/upload_images/5477931-2b2511e23d04a185.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* Distribution（分配）: Fill Equally（均分填满）
* Spacing（间距）: 0

在可视化编辑器的底部点击 **Add New Constraints（添加新的约束条件）**，设置左边、右边、底部和高度如上图所示。将 **Update Frames** 设置为 **Items of New Constraints**，然后点击 **Apply 4 Constraints（应用四条约束）**。

![](http://upload-images.jianshu.io/upload_images/5477931-f85403067df43c12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Stack View 现在已经放置好了，但这些按钮似乎比 Stack View 矮了一些，在 **Document Outline（文档大纲）** 中，按住 Control⌃ 键的同时拖动 **Start** 按钮到 **Stack View** 上，并选择  **Equal Heights（高度等同）**。对另外两个按钮进行同样的操作。

![](http://upload-images.jianshu.io/upload_images/5477931-ab9ac2de341eaff3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在，包含了按钮的 Stack View 已经完全是我们想要的样子了。

编译并运行你的 app，尝试着改变窗口的大小，这些按钮会紧紧地贴在窗口的底部，并自动填满整个窗口的宽度。

![](http://upload-images.jianshu.io/upload_images/5477931-74146b94cd847984.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后一步，在 **Attributes Inspector（属性检查器）**中取消选中 **Enabled** 来禁用 Stop 按钮和 Reset 按钮，在计时开始前，这两个按钮不应该被启用。

# 布局 UI（二）

第二个 Stack View 包含剩余的时间以及鸡蛋的图片。向 View Controller 拖入一个 Label，把它的 **Title（标题）**设置为 6:00 并把 **Alignment（对齐）**设置为 center（居中）。默认的字体（San Francisco）会自动调整字符的间距——这意味着：随着倒计时的时间的变化，数字会来回跳跃——这会很烦人。

把字体改成 **Helvetica Neue** 来避免这样，然后把 Size（字体大小）设置为 **100**。这会使文字超出 Label 的范围，所以调整 Label 的大小直到你可以看到完整的文字。

![](http://upload-images.jianshu.io/upload_images/5477931-3ebc934ae850da45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在我们需要添加一张图片，在 **Object Library（控件库）**的搜索框中输入「image」，这会过滤出很多长得很像的控件，你需要的是 **Image View**，将它拖动到 **View Controller** 中剩余时间 Label 的下方。

下载[这个工程需要的资源文件](https://koenig-media.raywenderlich.com/uploads/2017/01/EggTimerAssets-1.zip)（一些图片和一个声音文件），解压碎并打开 **Egg Images** 文件夹。回到 Xcode，在 **Project Navigator（项目导航器）**中点击 **Assets.xcassets**。

把这六个图片拖动到 Assets Library（资产库）中，然后它们就可以在你的 app 中使用了。因为图片的文件名中带有「@2x」，它们会被自动分配到各个图像资产的 @2x 区域。

> **译注**：关于 @2x 请自行搜索 Mac/iOS 的 HiDPI适应

![](http://upload-images.jianshu.io/upload_images/5477931-97b495f9dec58409.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

前往 **Main.storyboard**，选中你刚刚添加的 **Image View**，在 **Attributes Inspector（属性检查器）**中点击 **Image（图像）**下拉框，在这个下拉框中你可以看到很多内置的图片以及你刚刚添加的图片，选中 **stopped**。

现在我们来制作第二个 Stack View：选中显示剩余时间的 Label 和刚刚调整好的 Image View，在菜单栏上点击 **Editor** → **Embed In** → **Stack View**。然后我们来调整一下这个 Stack View，使它填满剩余的空间。点击可视化编辑器底部的 **Add New Constrains（添加新的约束）**，然后添加下图所示的约束条件：

![](http://upload-images.jianshu.io/upload_images/5477931-615f609fb882c324.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

正我们所需要的，Stack View 填满了剩下的空间，但这个 Image View 太小了，选中它，将它左右两您的约束按照下图设置为 **User Standard Value（使用标准值）**：

![](http://upload-images.jianshu.io/upload_images/5477931-39834cef032ced63.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 **Attributes Inspector（属性检查器）**中，设置 **Scaling（缩放）** 为 **Proportionally Up or Down（等比放大或缩小）**。

编译并运行 app，调整窗口的大小，看看 UI 元素是否如预期地自动地适应了窗口尺寸。

![](http://upload-images.jianshu.io/upload_images/5477931-5ea6f7acd31b9674.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 连接 UI 和代码

在第一部分（[原文](https://www.raywenderlich.com/151741/macos-development-beginners-part-1) / [译文](http://www.jianshu.com/p/a3f16178a213)）中，你已经知道了你需要使用 `@IBOutlets` 和 `@IBActions` 来连接你的 UI 和代码。在这个例子中，你需要为以下元素设置 `@IBOutlets`：

- 剩余时间的 Label
- 显示鸡蛋的 Image View
- 三个按钮

此外，这三个按钮还需要 `@IBAction` 来响应用户的点击操作。在 **Project Navigator（项目导航器）**中，先选中 **Main.storyboard**，再按住 Option⌥，点击 **ViewController.swift**，这样就可以在 **Assistant Editor（辅助编辑器，即左右分栏）**中打开它了。如果你的屏幕空间不足，点击右上角的按钮来隐藏 **Utilities（工具集）** 和 **Navigator（导航器）** 面板。

和第一部分一样，选中剩余时间的 Label，按住 Control⌃ 键的同时拖动它到 `ViewController` 类中，将名称设置为 `timeLeftField`，对鸡蛋的 Image View 进行一样的操作，它的名字设置为 `eggImageView`。然后是三个按钮，他们的名字分别设置成 `startButton`、`stopButton` 和 `resetButton`。

这三个按钮还需要 `@IBAction`，按住 Control⌃ 并拖动 Start 按钮到代码中，但这次在弹出窗口中将 **Connection** 设置为 **Action**，并把它的名字设置为 `startButtonClicked`，对另外两个按钮重复以上动作，为它们添加名为 `stopButtonClicked` 和 `resetButtonClicked` 的 @IBAction。

如果你和我一样也经常忘了把 Connection 修改为 Action 的话，你会得到 `@IBOutlets` 而不是 `@IBAction`，如果你要删除这些多余的 `@IBOutlet`，先把这一行代码删除，再转到 **Utilities（工具集）**里的 **Connections Inspector（连接检查器）**，你会看到 **Referencing Outlets** 部分里有两个项目，点击错误连接旁的 **×** 来删除它，然后重新连接你的 `@IBAction`（这一次别忘了修改 Action 哦😛）。

![](http://upload-images.jianshu.io/upload_images/5477931-13272c42cf46d1e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`ViewController` 里的代码现在看起来应该像这样：

![](http://upload-images.jianshu.io/upload_images/5477931-6b623748347d67e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在第三部分中，你将会给这些动作添加代码，现在我们关闭 **Assistant Editor（辅助编辑器）**，如果你刚刚隐藏了 **Navigator（导航器）** 和 **Utilities（工具集）**，重新打开它们。

# 菜单

在 **Main.storyboard**，在 **Application Scene（应用场景）** 中点击 **menu bar（菜单栏）**，Xcode 提供的模版已经内置了一些默认的按钮，但是在这个 app 中，很多按钮都派不上用场，最简单的办法来浏览菜单里的项目是使用 **Document Outline（文档大纲）**，点击左侧的小三角来显示 **View** 菜单和里边的内容。

![](http://upload-images.jianshu.io/upload_images/5477931-2c9bae3bcd38361d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

菜单栏的结构是由一些 Menus（菜单）和 Menu Items（菜单项目）组成的，切换到 **Utilities（工具集）**里的 **Identity Inspector（身份检查器）**，这样你就能看到，当你点击各个菜单项目的时候，真正触发的东西是什么了。**Main Menu** 是 `NSMenu` 的实例，它包含了 `NSMenuItem` 组成的数组：View 菜单就是这个数组的一个成员。

View 菜单包含一个子菜单（`NSMenu`），里面包含了它自己的 `NSMenuItems`，值得注意的是，`Separator`（分割线）是 `NSMenuItem` 的特殊形式。

我们要做的第一件事是删除这个 app 里不需要的菜单项，在 **Document Outline（文档大纲）**中选中 **File** 菜单，按下键盘上的 **Delete⌫** 键把它删掉。如果你是在 **Visual Editor（可视化编辑器）**中选中它并删除的话，你会把 **File** 菜单里的菜单项都删掉，然后只剩下一个空空如也的菜单，如果你不小心这样做了，选中那个空白然后再次按下 **Delete⌫** 来移除它。

> 这段话的后半段没有看太明白，望各位指正，原文是： If you select it in the **Visual Editor** and delete, you will only have deleted the menu inside the **File** menu item, so you will be left with a space in the menu bar. If this happens, select the space and press **Delete** again to remove it. 

继续删除其他的 Menu，直到你只剩下了 EggTimer、Window 和 Help 菜单。

![](http://upload-images.jianshu.io/upload_images/5477931-f513732e08e2b756.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在你需要添加一个新的菜单来模拟三个按钮的操作，在 **Object Library（控件库）** 中搜索 「menu」，请注意，菜单栏中的每一个项目（如 File、Help ）都是 Menu Item，点击它之后打开的才是 Menu，拖动一个 **Menu Item** 到菜单栏中 **Egg Timer** 和 **Window** 之间的地方，它会显示成一个蓝色的方框，这是因为它内部还没有一个带有标题的 Menu。

现在想那个蓝色的方框拖入一个 **Menu**，如果你觉得那个方框太小拖不准的话，你也可以拖到 **Document Outline（文档大纲）**里我们刚刚创建的 **Item** 的下方。新的 Menu 还没有标题，但他已经有了三个菜单项。

![](http://upload-images.jianshu.io/upload_images/5477931-34421f6271f65999.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选中 Menu（不是 item），切换到 **Attributes Inspector（属性检查器）**并把 Title（标题）设置为 **Timer**，选中 **Item 1**，双击它（或在属性检查器里）把他的标题改为 **Start**。

在 **Attributes Inspector（属性检查器）**中点击 **Key Equivalent（快捷键）**，然后按下 **Command⌘ + S** 来设置快捷键。通常情况下 Command⌘ + S 是保存的快捷键，但是你已经删除了 File 菜单，这样的设置并不会有冲突，但在一般情况下我们还是不要复用常见快捷键的比较好。

用同样的方法把第二、三个 item 的标题分别设置为设置为 **Stop** 和 **Reset**，快捷键分别为 **Command⌘ + X** 和 **Command⌘ + R**。

![](http://upload-images.jianshu.io/upload_images/5477931-18b590d315a6d327.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在可视化编辑器中 Menu Bar 的上方，你能看到三个按钮，切换到 **Identity Inspector（身份检查器）**，然后依次点击这三个按钮，你可以看到，他们分别连接着 **Application**、**First Responder**、和 **AppDelegate**。First Responder 通常就是当前处于最前端的 View Controller，它可以从 Menu Item 那里接收动作。

按住 Option⌥ 的同时点击 **ViewController.swift**，然后在你刚刚为按钮们添加的 `@IBAction` 的下方添加以下代码：

``` swift
// MARK: - IBActions - menus
@IBAction func startTimerMenuItemSelected(_ sender: Any) {_
startButtonClicked(sender)
}

@IBAction func stopTimerMenuItemSelected(_ sender: Any) {
stopButtonClicked(sender)
}

@IBAction func resetTimerMenuItemSelected(_ sender: Any) {
resetButtonClicked(sender)
}
````

这些方法会在菜单项被点击时被调用，然后它们会去调用界面上那三个按钮的 IBAction 方法。其实，你可以让 Menu Item 直接调用按钮的方法（即把 Menu Item 的 IBAction 也拖到之前定义的按钮的 IBAction 上），但在这里我选择这种方式，因为这样的话，调试时这一系列事件会更加清晰明了。现在保存文件并退出 Assistant Editor（协助编辑器）。

按住 Control⌃ 键的同时，把 **Start** 菜单项拖动到象征着 **First Responder** 的橙色立方体上，一个包含了很多选项的窗口会弹出来，在键盘上输入「sta」来快速滚动到我们所需要的 **startTimerMenuItemSelected** 并选择它。

![](http://upload-images.jianshu.io/upload_images/5477931-36c02651cd888ae9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

用同样的方法把 **Stop** 菜单项和 **Reset** 菜单项分别连接至 `stopTimerMenuItemSelected` 和 `resetTimerMenuItemSelected`。现在当 EggTimer 窗口处于最前端时，点击菜单上的按钮将会调用这些方法。

但是还有一个问题：这三个按钮在同一时刻并非同时都是可用的，而且菜单栏上的按钮需要体现出他们是否可用。这个功能无法在 `ViewController` 里实现，因为它不会一直处于 First Responder 的状态，所以我们需要转战 `AppDelegate`。

打开 **Main.storyboard** 并确保这几个 Menu 都处于可见状态，在 **Project Navigator（项目导航器）**中按住 Option⌥ 键的同时点击 **AppDelegate.swift**，按住 Control⌃ 键的同时把 **Start** 菜单项拖动到 `AppDelegate` 中，创建一个名为 `startTimerMenuItem` 的 IBOutlet。

用同样的方法为另外两个菜单项创建名为 `stopTimerMenuItem` 的 `resetTimerMenuItem` IBOutlet。

![](http://upload-images.jianshu.io/upload_images/5477931-5ab7401f4b92e6cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在第三部分中你将了解到如何用代码来根据需要启用和禁用这些菜单项，但是现在你需要做的是关闭自动启用与禁用，因为在一般情况下，app 会检测当前的 First Responder 是否包含了 menu item 所连接的 action（一般就是 IBAction），如果没有，就会将他们禁用。在这个 app 中，我们希望自己来控制这件事情，所以选中 **Timer** 菜单项，在 **Attributes Inspector（属性检查器）** 中取消选中 **Auto Enables Item（自动启用项目）**。

# 偏好设置窗口

![](http://upload-images.jianshu.io/upload_images/5477931-d57913491417e0ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在 EggTimer app 的主界面已经看起来很好了，但他还需要一个偏好设置窗口来让用户选择他们想把鸡蛋煮多熟。

偏好设置界面将会显示在一个单独的窗口中，且拥有它自己的 Window Controller。这是因为尽管在一个 Window Controller 中显示多个 View Controller 是完全可行的，但它们会共享这个 Window Controller 的所有属性，而偏好设置窗口的默认大小与主窗口不一样，而且不可以调整大小。

打开 **Main.storyboard**，如果 Assistant Editor（辅助编辑器）还处于打开状态，把它关闭。在 **Objects Library（控件库）**中搜索「window」，向 **Visual Editor（可视化编辑器）**中拖入一个 **Window Controller**，Xcode 会自动为你创建一个 **View Controller** 来盛放需要现实的内容。重新排列一下窗口中的内容以便你能更清楚地看清所有内容，并让你新创建的 Window Controller 更靠近菜单栏。

打开菜单栏上的 **EggTimer** 菜单，按住 Control⌃ 键的同时拖动  **Preferences…** 菜单项到我们新创建的 Window Controller 上，这将会创建一个 Segue（转场），也就是说用户点击 **Preferences…** 时，这个 Window Controller 就会把我们新创建的 View Controller 给显示出来。

![](http://upload-images.jianshu.io/upload_images/5477931-38281070aa691081.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

偏好设置面板需要显示一个新的 View Controller，所以你需要为它创建一个新的 View Controller 类。在 **Project Navigator（项目导航器）**中，选中已经存在了的 **ViewController.swift** 文件，这会确保新建的文件存储在与之相同的组中（Xcode 用 Group 来管理文件），然后在 Xcode 的菜单上点击 **File** → **New** → **File…**。

![](http://upload-images.jianshu.io/upload_images/5477931-9d4f11612ef000b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选择 **macOS** → **Cocoa Class** 然后点击 **Next**，设置类名为 `PrefsViewController`，父类为 `NSViewController`，语言选择 **Swift**，不要勾选 **Also create XIB file for user interface（同时为 UI 创建 XIB 文件）**，然后点击 **Next** 和 **Create** 来保存文件。

![](http://upload-images.jianshu.io/upload_images/5477931-b8c33ef95a870bf6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

回到 **Main.storyboard**，选中我们新创建的 View Controller，请确保你选择的是 View Controller 而不是它的 View，在 **Document Outline（文档大纲）**里选择会更容易些。在 **Identity Inspector（身份检查器）**中，把它的 Class 设置为 `PrefsViewController`。

![](http://upload-images.jianshu.io/upload_images/5477931-f8c00a8af7e5cced.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选择偏好设置窗口里的 Window Controller，前往 **Attributes Inspector（属性检查器）**把它的标题设置为 ** Preferences**。Autosave Name 保持留空，这样每次窗口出现的时候都会出现在桌面的中央。取消选中 **Minimize** 和 **Resize** 复选框，这样窗口的大小就是固定的了。

![](http://upload-images.jianshu.io/upload_images/5477931-f7c1891dd5a4b659.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

前往 **Size Inspector（属性检查器）**，把 **Content Size** 设置为宽 416 高 214。在 **Initial Position（默认位置）** 下方的两个下拉框中分别选择 **Center Horizontally** 和 **Center Vertically**。

![](http://upload-images.jianshu.io/upload_images/5477931-2f67ec6ea8e6e5bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选中 **View** 中的 **PrefsViewController**，在 **Size Inspector（尺寸检查器）**中把它的宽和高分别设置为 416 和 214。

`PrefsViewController` 需要显示一个用来选择时间的下拉框和一个用来选择时间的滑块，它们都包含各自的 Label 用于显示标题，还有两个按钮：Cancel 和 OK，以及一个用于显示当前选择时间的 Label。

![](http://upload-images.jianshu.io/upload_images/5477931-118f4b02dadb21be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

拖动以下控件到 View Controller 中并按下表设置它们的属性：

|||
| ------------- |-------------|
|**Label**|设置 title 为「Preset Egg Timings:」|
|**Pop Up Button**||
|**Label**| 设置 title 为「Custom Egg Timing:」|
|**Label**| 设置 title 为 「6 minutes」|
|**Horizontal Slider**||
|**Push Button**| 设置 title 为 「Cancel」|
|**Push Button**|设置 title 为「OK」|

因为这个窗口不能调整大小，控件们会按照你的布局进行显示，所以我们不需要给它设置 Auto-layout 约束条件。把界面上的元素排列好，Xcode 自动会用蓝色的参考线来帮助你进行对齐。将显示「6 minutes」的 Label 的右边拖动到几乎与边界持平，因为它可能需要显示更多内容。双击 **Pop Up Button**，你会看到三个子项，把他们的标题分别设置为：

* For runny soft-boiled eggs (barely set whites): 3 minutes
* For slightly runny soft-boiled eggs: 4 minutes
* For custardy yet firm soft-boiled eggs: 6 minutes

再从 ** Objects Library（控件库）**中拖动两个 **Menu Item**、一个 **Separator Menu Item** 和另一个 **Menu Item** 到下拉框中，如果你觉得直接拖动它们到界面上有点难度的话，也可以拖动到 **Document Outline（文档大纲）**的相应位置。

给刚刚拖入的三个子项分别设置标题：

* For firm yet still creamy hard-boiled eggs: 10 minutes
* For very firm hard-boiled eggs: 15 minutes
* Custom

![](http://upload-images.jianshu.io/upload_images/5477931-a18bf7365a1fb3e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我其实一点都不知道怎么煮鸡蛋，所以我从 [The Kitchn](http://www.thekitchn.com/how-to-boil-eggs-perfectly-every-time-cooking-lessons-from-the-kitchn-202415) 找到了以上的时间和描述。

选中下拉框本体（而不是那几个子项），把它的 **Selected Item（已选中项目）**设置为那个「6 minute」的选项。

现在你需要让你的 app 知道用户在下拉框中到底选择了哪个子项，依次选中下拉菜单中的每一个自子项，在 **Attributes Inspector（属性检查器）**中设置它们的 Tag 分别为对应的分钟数：3、4、6、10、15（Custom 子项设置为 0）。
![](http://upload-images.jianshu.io/upload_images/5477931-168ccf7361b49bc7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在选择 **Slider**，在 **Attributes Inspector（属性检查器）**中将 **Tick marks（刻度数）** 设置为 25，**Minimum Valve** 设置为 1，**Maximum Valve** 设置为 25，并勾选 **Only stop on tick marks（只能选择刻度）**，你需要将滑块向下移动一些来适应新出现的刻度。因为只有当用户在下拉框种选择了 **Custom** 的时候才会启用，所以我们还需要取消选择 **Enabled**。

![](http://upload-images.jianshu.io/upload_images/5477931-250dc7eb52fcc9ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 连接偏好设置里的元素和代码

在 **Project Navigator（项目导航器）**中按住 Option⌥ 键的同时点击 **PrefsViewController**，如果你需要更多空间的话隐藏掉侧边面板。你需要为下拉框、滑动条和显示“6 minutes”的 Label 添加 `@IBOutlet`。把它们依次拖动到 `PrefsViewController.swift` 中，并给这些 IBOutlet 分别起名： 

* Popup: `presetsPopup`
* Slider: `customSlider`
* Label: `customTextField`

接下来，按住 Control⌃ 键的同时拖动以下项目来创建 `@IBAction`（别忘了把 **Connection** 改成 **Action** 哦～）：

* Popup: `popupValueChanged`
* Slider: `sliderValueChanged`
* Cancel button: `cancelButtonClicked`
* OK button: `okButtonClicked`

现在你的代码看起来应该像这样：

![](http://upload-images.jianshu.io/upload_images/5477931-fd6157d641407125.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至此，偏好设置面板窗口的布局就已经完成了，编译并运行你的 app 并在 **EggTimer** 菜单中选择 **Preferences** 菜单，检查一下打开的偏好设置窗口，然后点击红色的关闭按钮来关闭这个窗口。

![](http://upload-images.jianshu.io/upload_images/5477931-5f2525c0145a1a43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# App 图标

UI 部分还剩一步：为你的 app 添加图标。你之前已经下载了一个包含了这个 app 所需要的所有文件的资产文件夹，而且已经添加了一些图片到 Assets.xcassets 中，现在我们需要打开这个文件夹并找到 **egg-icon.png** 文件。

在 **Project Navigator（项目导航器）**中选择 **Assests.xcassets**，点击 **AppIcon** 并拖动 **egg-icon.png** 到 **Mac 256pt 1x** 的方框中。正如第一部分所说的，在真正的产品中你需要提供所有尺寸的图标，但在这个 app 中，一个图标就足够了。

![](http://upload-images.jianshu.io/upload_images/5477931-e432e5e3eb797654.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

编译并运行你的 app，看看新的图标有没有出现在 Dock 中，如果没有，你可能需要在 Xcode 的 **Product** 菜单中点击 **Clean**，然后再试一次。

![](http://upload-images.jianshu.io/upload_images/5477931-f55108ac9559467e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
