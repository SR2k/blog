---
title: 【译】macOS 上的 Core Graphics 入门教程
date: 2017-09-25 17:54:11
tags:
  - 翻译
  - Ray Wenderlich
  - macOS
  - Swift
---

> 本文翻译自 [raywenderlich.com](https://www.raywenderlich.com/) 的 [Core Graphics on macOS Tutorial](https://www.raywenderlich.com/128614/core-graphics-os-x-tutorial)，已咨询对方网站，可至多翻译 10 篇文章。
> 希望各位有英语阅读能力的话，还是**先打赏**然后去阅读英文原吧，毕竟无论是 Xcode，抑或是官方的文档，还是各种最前沿的资讯都只有英文版本。  
> 综上，此翻译版本仅供参考，**谢绝转载**。
> 
> 也欢迎你点击我的头像查看我翻译的其他 macOS 开发教程💋

![](http://upload-images.jianshu.io/upload_images/5477931-5204a02b7332fac7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> **更新于 2016-9-22：**此教程已更新至 Xcode 8 和 Swift 3。

你肯定见过许多拥有漂亮的界面和华丽的自定义视图的 app，它们肯定在你的心里留下了深刻的印象，因为它们是**那！么！好！看！**

**Core Graphics** 是 Apple 提供的 2D 绘图引擎，也几乎是 macOS 和 iOS 所有框架中最酷的了。它可以用来绘制你能想到的所有图形，从简单的几何形状到复杂的阴影和渐变等视觉效果。

在这个 macOS Core Graphics 教程中，你将会创造一个叫做 **DiskInfo** 的自定义视图，它能用一张饼图和一个条状图来显示出你 Mac 上的硬盘可用空间。这个教程将会让你拥有把平淡单调的 UI 变得精彩纷呈的能力：

![](http://upload-images.jianshu.io/upload_images/5477931-f0a35f95c87f254e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这个教程中你将学会：

* 创建并配置一个自定义视图，这是绘制图形元素的必要条件；
* 实现实时渲染预览功能，有了它你不需要编译和运行，就能在 Interface Builder 里看到你对图形的各种修改；
* 用代码绘制路径、填充图形、创建剪切蒙版剪辑和渲染文本；
* 使用 **AppKit** 里的 **Cocoa Drawing** 工具提供的高级类和方法。

在第一部分中，你将会通过 Core Graphics 来实现绘制一个饼图，稍后你将会学习如何用 Cocoa Drawing 实现相同的效果。

所以拿起你的小画刷，我们要开始作画啦～

# 准备开始

[点击这里](https://koenig-media.raywenderlich.com/uploads/2016/06/DiskInfo-Starter.zip)下载 DiskInfo 的起步工程，编译并运行它：

![](http://upload-images.jianshu.io/upload_images/5477931-6aed1359cd471275.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个 app 会罗列出你的所有硬盘，点击任何一个即可查看他的详细信息。

在操作之前，我们先来熟悉一下这个项目的结构：

![](http://upload-images.jianshu.io/upload_images/5477931-8f84eb5ad169695c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **ViewController.swift**：app 的主要 View Controller；
- **VolumeInfo.swift**：实现了用于处理硬盘信息的 `VolumeInfo` 类，以及用于分析不同文件类型所占空间的 `FilesDistribution` 结构体；
- **NSColor+DiskInfo.swift** 和 **NSFont+DiskInfo.swift**：扩展了 NSColor，定义了 app 中会用到的颜色和字体；
- **CGFloat+Radians.swift**：扩展了 CGFloat，提供了转换角度值和弧度制的 helper 方法；
- **MountedVolumesDataSource.swift** 和 **MountedVolumesDelegate.swift**：实现了显示硬盘信息所必需的各类方法。

> **注意：**这个 app 可以显示你真正的硬盘用量信息，但在这个教程中，它将会生成随机的数据。
> 每次启动 app 时都计算一次硬盘上所有文件的类型会很耗时，也会消磨完你所有的乐趣，没人愿意在这上面浪费时间，对吧😉？

# 创建一个自定义视图

你要做的第一件事是创建一个名叫 `GraphView` 的自定义视图。这将会是你绘制饼状图和条形图的地方。这个部分中你需要做两件事：

1. 创建一个 `NSView` 的子类；
2. 重写 `draw(_:)` 方法，加入一些用于绘制的代码。

## 创建 NSView 的子类

选中项目导航器的 **Views** 分组，点击去 Xcode 菜单上的 **File** → **New** → **File…**，然后点击 **macOS** → **Source** → **Cocoa Class** 模版。

点击 **Next**，把新的类命名为 `GraphView`，并让它继承自 `NSView`，把语言选择为 **Swift**。

点击 **Next** 和 **Create** 来保存你的文件。

打开 **Main.storyboard**，在 **View Controller Scene** 中，从**控件库**里拖入一个 **Custom View**。

![](http://upload-images.jianshu.io/upload_images/5477931-4fc358f76ed26670.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选中这个 Custom View，在**身份检查器**里，把它的类名设置为 `GraphView`。

![](http://upload-images.jianshu.io/upload_images/5477931-3dd1e7d056a92c9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

你需要一些约束，所以保持 Graph View 的选中状态，点击 Auto Layout 工具栏上的 **Pin** 按钮，把它的 **Top**、**Bottom**、**Leading** 和 **Trailing** 约束设置为 0，然后点击 **Add 4 Constrains** 按钮。

![](http://upload-images.jianshu.io/upload_images/5477931-80fe225f08e1c3f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击 Auto Layout 工具栏上的三角形的 **Resolve Auto Layout Issues** 按钮，然后在 **Selected Views** 部分中点击 **Update Frames**。如果这个选项不可用，你可能需要先点击空白处取消选中 Graph View，然后再次选中它。

![](http://upload-images.jianshu.io/upload_images/5477931-8316bd45e283b331.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 重写 `draw(_:)`

打开 **GraphView.swift**，你能看到 Xcode 自动为我们创建了一个 `draw(_:)` 的实现。把那行注释替换成以下代码，并确保你别不小心删掉了它调用父类此方法的那一行哦。

```swift
NSColor.white.setFill() 
NSRectFill(bounds)
```

第一行代码把填充颜色设置为了白色，然后通过调用 `NSRectFill` 方法，你把整个视图的背景设成了白色。

编译并运行：

![](http://upload-images.jianshu.io/upload_images/5477931-3a9a9a76b687046c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

你已经把自定义视图的背景从默认的灰色改成了白色。

![](http://upload-images.jianshu.io/upload_images/5477931-e86ac3d6a8e288b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

哈哈，我们的画布已经就绪！就是这么简单～

# 实时渲染预览：`@IBDesignable` 和 `@IBInspectable`

Xcode 6 为我们带来了一个牛×的功能：实时渲染预览。它允许你在 Interface Builder 里查看你自定义的视图的样子，而不用每次都编译和运行。

要启用这个功能，你需要用 `@IBDesignable` 来修饰你的类；并实现 `prepareForInterfaceBuilder()` 方法来提供一些示例数据（实现这个方法不是必须的）。

打开 **GraphView.swift**，在类的定义之前加入：

```swift
@IBDesignable
```

现在你需要提供一些示例数据，把这些代码添加到 `GraphView` 类中：

```swift
var fileDistribution: FilesDistribution? {
  didSet {
    needsDisplay = true
  }
}

override func prepareForInterfaceBuilder() {
  let used = Int64(100000000000)
  let available = used / 3
  let filesBytes = used / 5
  let distribution: [FileType] = [
    .apps(bytes: filesBytes / 2, percent: 0.1),
    .photos(bytes: filesBytes, percent: 0.2),
    .movies(bytes: filesBytes * 2, percent: 0.15),
    .audio(bytes: filesBytes, percent: 0.18),
    .other(bytes: filesBytes, percent: 0.2)
  ]
  fileDistribution = FilesDistribution(capacity: used + available,
                                       available: available,
                                       distribution: distribution)
}
```

这将会定义一个 `fileDistribution` 属性用于存储硬盘的信息。当这个属性发生改变，它会设置这个视图的 `needsDisplay` 属性为 `true`，从而让视图重绘自己的内容。

然后，它实现了  `prepareForInterfaceBuilder()` 方法，以此创建了一个各种文件类型的例子，用于给 Xcode 预览这个视图。

> **注意：**你甚至可以在 Interface Builder 里实时修改视觉属性。这要求你用 `@IBInspectable` 来修饰这个属性。

下一步：用 `@IBInspectable` 修饰所有的视觉属性，把这些代码添加到 `GraphView` 的声明中：

```swift
// 1
fileprivate struct Constants {
  static let barHeight: CGFloat = 30.0
  static let barMinHeight: CGFloat = 20.0
  static let barMaxHeight: CGFloat = 40.0
  static let marginSize: CGFloat = 20.0
  static let pieChartWidthPercentage: CGFloat = 1.0 / 3.0
  static let pieChartBorderWidth: CGFloat = 1.0
  static let pieChartMinRadius: CGFloat = 30.0
  static let pieChartGradientAngle: CGFloat = 90.0
  static let barChartCornerRadius: CGFloat = 4.0
  static let barChartLegendSquareSize: CGFloat = 8.0
  static let legendTextMargin: CGFloat = 5.0
}

// 2
@IBInspectable var barHeight: CGFloat = Constants.barHeight {
  didSet {
    barHeight = max(min(barHeight, Constants.barMaxHeight), Constants.barMinHeight)
  }
}
@IBInspectable var pieChartUsedLineColor: NSColor = NSColor.pieChartUsedStrokeColor
@IBInspectable var pieChartAvailableLineColor: NSColor = NSColor.pieChartAvailableStrokeColor
@IBInspectable var pieChartAvailableFillColor: NSColor = NSColor.pieChartAvailableFillColor
@IBInspectable var pieChartGradientStartColor: NSColor = NSColor.pieChartGradientStartColor
@IBInspectable var pieChartGradientEndColor: NSColor = NSColor.pieChartGradientEndColor
@IBInspectable var barChartAvailableLineColor: NSColor = NSColor.availableStrokeColor
@IBInspectable var barChartAvailableFillColor: NSColor = NSColor.availableFillColor
@IBInspectable var barChartAppsLineColor: NSColor = NSColor.appsStrokeColor
@IBInspectable var barChartAppsFillColor: NSColor = NSColor.appsFillColor
@IBInspectable var barChartMoviesLineColor: NSColor = NSColor.moviesStrokeColor
@IBInspectable var barChartMoviesFillColor: NSColor = NSColor.moviesFillColor
@IBInspectable var barChartPhotosLineColor: NSColor = NSColor.photosStrokeColor
@IBInspectable var barChartPhotosFillColor: NSColor = NSColor.photosFillColor
@IBInspectable var barChartAudioLineColor: NSColor = NSColor.audioStrokeColor
@IBInspectable var barChartAudioFillColor: NSColor = NSColor.audioFillColor
@IBInspectable var barChartOthersLineColor: NSColor = NSColor.othersStrokeColor
@IBInspectable var barChartOthersFillColor: NSColor = NSColor.othersFillColor

// 3
func colorsForFileType(fileType: FileType) -> (strokeColor: NSColor, fillColor: NSColor) {
  switch fileType {
  case .audio(_, _):
    return (strokeColor: barChartAudioLineColor, fillColor: barChartAudioFillColor)
  case .movies(_, _):
    return (strokeColor: barChartMoviesLineColor, fillColor: barChartMoviesFillColor)
  case .photos(_, _):
    return (strokeColor: barChartPhotosLineColor, fillColor: barChartPhotosFillColor)
  case .apps(_, _):
    return (strokeColor: barChartAppsLineColor, fillColor: barChartAppsFillColor)
  case .other(_, _):
    return (strokeColor: barChartOthersLineColor, fillColor: barChartOthersFillColor)
  }
}
```

这一大坨代码的作用是：

1. 声明了带有许多常量的结构体 —— 你得在这个 app 的各个地方用到它们；
2. 用 `@IBInspectable` 修饰所有可配置的属性。并使用 **NSColor+DiskInfo.swift** 中的值为它们赋值。注意：要使一个属性「inspectable」（可以在 Interface Builder 里直接编辑），你必须声明它的类型，即使大多数情况下 Swift 会帮你做这件事儿；
3. 定义一个 Helper 方法，为不同的文件类型返回它的笔触颜色和填充颜色。你在绘制「什么文件占了多大地儿」的图表的时候会用到它。

打开 **Main.stroyboard**，你应该能注意到 Graph View 已经从默认的灰色变成了白色，这意味着实时渲染预览已经起作用了。

![](http://upload-images.jianshu.io/upload_images/5477931-466bd5ddecfe519d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选中 Graph View，打开**属性检查器**，你会发现所有的「inspectable」属性已经出现在这里了。

![](http://upload-images.jianshu.io/upload_images/5477931-ea983d52c0f02849.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在开始，要查看调整好的效果，你可以直接编译和运行，也可以直接在 Interface Builder 里查看。

万事俱备，是时候开始真正的绘制啦！

# Graphics Context（图形上下文）

在使用 Core Graphics 的时候，你并不是直接在视图中绘画，而是使用一个叫 **Graphics Context（图形上下文）**的东西，它是系统渲染图形与把图形显示在视图中的中间层。

![](http://upload-images.jianshu.io/upload_images/5477931-077fb51fe0aae885.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Core Graphics 使用一个叫「Painter’s Model（画家模式）」的模式，你可以想像成自己拿着笔，唰唰地在画布上绘图样子。你需要放置一个路径，然后去填充它，你没办法去改变已经布置好的像素们，但你可以在它们之上继续画图。

这个「上下文」非常重要，因为他决定了你最终得到的效果。

![](http://upload-images.jianshu.io/upload_images/5477931-8be0aad94ef83a2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 绘制路径

要使用 Core Graphics 绘制一个路径，你需要定义一个 **Path（路径）**，也就是 `CGPathRef` 和可变的 `CGMutablePathRef`。

路径准备好以后，把它添加到图形上下文里，就可以根据路径和绘制属性渲染出你要的图形了。

## 为饼图…绘制一个路径

条形图的基本元素是圆角矩形，所以我们从这里开始入手。

打开 **GraphView.swift**，把这个扩展添加在文件底部类定义以外的地方：

```swift
// MARK: - 用于绘制的 extension

extension GraphView {
  func drawRoundedRect(rect: CGRect, inContext context: CGContext?,
                       radius: CGFloat, borderColor: CGColor, fillColor: CGColor) {
    // 1
    let path = CGMutablePath()
    
    // 2
    path.move( to: CGPoint(x:  rect.midX, y:rect.minY ))
    path.addArc( tangent1End: CGPoint(x: rect.maxX, y: rect.minY ), 
                 tangent2End: CGPoint(x: rect.maxX, y: rect.maxY), radius: radius)
    path.addArc( tangent1End: CGPoint(x: rect.maxX, y: rect.maxY ), 
                 tangent2End: CGPoint(x: rect.minX, y: rect.maxY), radius: radius)
    path.addArc( tangent1End: CGPoint(x: rect.minX, y: rect.maxY ), 
                 tangent2End: CGPoint(x: rect.minX, y: rect.minY), radius: radius)
    path.addArc( tangent1End: CGPoint(x: rect.minX, y: rect.minY ), 
                 tangent2End: CGPoint(x: rect.maxX, y: rect.minY), radius: radius)
    path.closeSubpath()
    
    // 3
    context?.setLineWidth(1.0)
    context?.setFillColor(fillColor)
    context?.setStrokeColor(borderColor)
    
    // 4
    context?.addPath(path)
    context?.drawPath(using: .fillStroke)
  }
}
```

太长不看🙄️：以上**就**是绘制一个圆角矩形的方法，用人类能理解的语言解释一遍就是：

1. 创建一个可以改变的路径；
2. 一步一步勾勒出一个圆角矩形：
	- 移动到矩形底边的中点，这里将是我们的起点；
	- 使用 `addArc(tangent1End:tangent2End:radius)` 方法绘制右下角的线段，这个方法会绘制出底部的水平线以及右下角的圆角；
	- 添加右边的线段和右上角的圆角；
	- 添加顶部的线段和左上角的圆角；
	- 添加左边的线段和左下角的圆角；
	- 闭合路径，也就是从上一步的重点连接到起点；
3. 设置绘制属性：线宽、填充颜色和边框颜色；
4. 把路径添加到图形上下文，并使用 `.fillStroke` 参数绘制着个路径，这个参数将会告诉 Core Graphics 这条路径需要填充颜色并绘制边框。

![](http://upload-images.jianshu.io/upload_images/5477931-11f0c635e8e2e781.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 计算位置

使用 Core Graphics 进行绘制的过程其实就是计算各个视觉元素在视图中位置的过程。所以我们需要关心的就是把不同的元素放置在哪，以及当视图的大小发生变化时它们该如何应对。

我们准备这样布局我们的视图：

![](http://upload-images.jianshu.io/upload_images/5477931-c03336233ad3ca43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打开 **GraphView.swift** 并添加这个扩展：

```swift
// MARK: - 用于计算的 extension

extension GraphView {
  // 1
  func pieChartRectangle() -> CGRect {
    let width = bounds.size.width * Constants.pieChartWidthPercentage - 2 * Constants.marginSize
    let height = bounds.size.height - 2 * Constants.marginSize
    let diameter = max(min(width, height), Constants.pieChartMinRadius)
    let rect = CGRect(x: Constants.marginSize,
                      y: bounds.midY - diameter / 2.0,
                      width: diameter, height: diameter)
    return rect
  }
  
  // 2
  func barChartRectangle() -> CGRect {
    let pieChartRect = pieChartRectangle()
    let width = bounds.size.width - pieChartRect.maxX - 2 * Constants.marginSize
    let rect = CGRect(x: pieChartRect.maxX + Constants.marginSize,
                      y: pieChartRect.midY + Constants.marginSize,
                      width: width, height: barHeight)
    return rect
  }
  
  // 3
  func barChartLegendRectangle() -> CGRect {
    let barchartRect = barChartRectangle()
    let rect = barchartRect.offsetBy(dx: 0.0, dy: -(barchartRect.size.height + Constants.marginSize))
    return rect
  }
}
```

以上代码做了这些必要的计算：

1. 从计算饼图的位置开始入手，它将会垂直居中，并占据视图 1/3 的宽度；
2. 计算条形图的位置，它将会占据 2/3 的宽度，并处于视图中部偏上的位置；
3. 根据饼图的最小 Y 值和边距来计算图例的位置。

现在我们来把把它绘制到你的视图中去，在 `GraphView` 的用于绘制的扩展中加入：

```swift
func drawBarGraphInContext(context: CGContext?) {
  let barChartRect = barChartRectangle()
  drawRoundedRect(rect: barChartRect, inContext: context,
                  radius: Constants.barChartCornerRadius,
                  borderColor: barChartAvailableLineColor.cgColor,
                  fillColor: barChartAvailableFillColor.cgColor)
}
```

你需要一个 Helper 方法来绘制条形图，它会绘制一个圆角矩形，并使用画笔颜色和填充颜色在空白处绘制图形，你可以在 **NSColor+DiskInfo** 扩展中找到这个些颜色。

把 `draw(_:)` 方法里的所有代码替换成：

```swift
super.draw(dirtyRect)
      
let context = NSGraphicsContext.current()?.cgContext
drawBarGraphInContext(context: context)
```

这段代码会真正地把图形绘制到视图中去。首先，通过调用 `NSGraphicsContext.current()`，我们获取到了当前视图的图形上下文，然后我们调用刚刚编写的方法绘制出了条形图。

编译并运行，你可以看到条形图已经就位了：

![](http://upload-images.jianshu.io/upload_images/5477931-0d2de8a91a09b0bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
现在，打开 **Main.storyboard** 并选中 **View Controller Scene**， 你会看到这个：

![](http://upload-images.jianshu.io/upload_images/5477931-b066eae14cf2949c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Interface Builder 为你实时渲染了预览。你可以试着去修改一下颜色，它也会实时响应你的修改，是不是很棒棒呀😄～

# 剪切一部分区域（也就是蒙版）

现在我们来制作文件分布图，也就是这个家伙：

![](http://upload-images.jianshu.io/upload_images/5477931-81d4bc8182f1d863.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

先暂停一下下，我们来理理思路。显而易见的是，每种文件都有自己的专属颜色，我们的 app 只需要根据这种文件占硬盘空间大小的百分比来计算每个方块的宽度，然后用对应的颜色把它绘制出来。

你需要绘制一个不规则的图形，比如一个!@#¥%%。然而，我们可以通过一个叫 **clipping areas（蒙板）** 的技术来避免编写重复代码。

> 👆这一段的第一句实在没看懂，求指正：You could create a special shape, such as a filled rectangle with two lines at bottom and top of the rectangle。

你可以把蒙版想象成「在一张纸上剪了个窟窿」，你只能透过这个窟窿看到部分的图形。这个「窟窿」就叫做「Clipping Mask（剪切蒙版）」，你需要在 Core Graphics 里定义它。

在这个条形图的例子里，你需要为每种文件分类创建一个完整的圆角矩形，然后通过剪切蒙版来使它们只显示正确的部分：

![](http://upload-images.jianshu.io/upload_images/5477931-a87e7ca09027a440.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

理论说完了，我们来动手吧～

开支绘制之前，你需要为选中的硬盘设置 `fileDistribution`。打开 **Main.storyboard**，我们来创建一个 Outlet 连接。

在项目导航器里按住 **Option⌥ 键**的同时点击 **ViewController.swift**，使它显示在右半边的**协助编辑器**里，然后按住 **Control⌃ 键**的同时把 Graph View 拖动到 View Controller 的代码里。

![](http://upload-images.jianshu.io/upload_images/5477931-bfc8259a9457f83f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在弹出的小气泡里，把这个 Outlet 命名为 `graphView`，并点击 **Connect**。

![](http://upload-images.jianshu.io/upload_images/5477931-a1d8f29e9ccf8ff1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打开 **ViewController.swift** 并把这行代码添加到 `showVolumeInfo(_:)` 的末尾：

```swift
graphView.fileDistribution = volume.fileDistribution
```

这行代码设置了 `fileDistribution` 的值，从而让 Graph View 能获取各类文件占的百分比。

打开 **GraphView.swift**，把这些代码添加到 `drawBarGraphInContext(context:)` 的末尾来绘制条形图：

```swift
// 1
if let fileTypes = fileDistribution?.distribution, let capacity = fileDistribution?.capacity, capacity > 0 {
  var clipRect = barChartRect
  // 2
  for (index, fileType) in fileTypes.enumerated() {
    // 3
    let fileTypeInfo = fileType.fileTypeInfo
    let clipWidth = floor(barChartRect.width * CGFloat(fileTypeInfo.percent))
    clipRect.size.width = clipWidth
        
    // 4
    context?.saveGState()
    context?.clip(to: clipRect)

    let fileTypeColors = colorsForFileType(fileType: fileType)
    drawRoundedRect(rect: barChartRect, inContext: context,
                    radius: Constants.barChartCornerRadius,
                    borderColor: fileTypeColors.strokeColor.cgColor,
                    fillColor: fileTypeColors.fillColor.cgColor)
    context?.restoreGState()
        
    // 5
    clipRect.origin.x = clipRect.maxX
  }
}
```

这些代码做了这些事儿：

1. 先确保了 Graph View 拥有一个有效的 `fileDistribution`；
2. 遍历 `fileDistribution` 里的每一种文件类型；
3. 根据文件的占比计算蒙版的大小；
4. 存储图形上下文的状态，设置蒙版的大小，用文件类型对应的颜色绘制圆角矩形，并恢复图形上下文的状态；
5. 把剪切蒙版的 **x** 移动到正确的位置。

你可能会奇怪：为什么要先存储，再恢复图形上下文？还记得「painter’s model」吗？你添加到图形上下文里的所有东西都会被保存在上下文中，就像你画在纸上的画，会一直在那里。

如果你添加了多个剪切蒙版，事实上你是只创建了一个剪切蒙版，并应用到所有矩形上。要避免这种情况，你需要在添加新的剪切蒙版之前存储上下文的状态，等你使用完了这个蒙版，再把它恢复出来，再处理新的蒙版。

此时，**Xcode** 会弹出一个警告，因为 `index` 从没被使用过。别担心，它的待会儿就会派上用场。

编译并运行，或者直接打开 **Main.storyboard**：

![](http://upload-images.jianshu.io/upload_images/5477931-53339621371d124f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

哈哈，DiskInfo 功能似乎已经渐渐完善了呢～除了图例，这个条形图已经基本完工了👍。

## 绘制文本

在自定义视图里绘制文本特别简单，你需要为这个文本的各种属性创建一个字典，包含了字体、尺寸、颜色和对齐，把它传入 `String` 的 `draw(in:withAttributes:)` 方法。这些属性将会在我们计算矩形大小和位置的时候派上用场。

打开 **GraphView.swift**，把这个属性添加到类的定义里：

```swift
fileprivate var bytesFormatter = ByteCountFormatter()
```

这将会创建一个 **ByteCountFormatter**。它会帮我们完成「把字节转化成人话」这个高深而繁重的工作。

现在，在 `drawBarGraphInContext(context:)` 方法的 `for (index,fileType) in fileTypes.enumerated()` 循环里加入这些代码：

```swift
// 1
let legendRectWidth = (barChartRect.size.width / CGFloat(fileTypes.count))
let legendOriginX = barChartRect.origin.x + floor(CGFloat(index) * legendRectWidth)
let legendOriginY = barChartRect.minY - 2 * Constants.marginSize
let legendSquareRect = CGRect(x: legendOriginX, y: legendOriginY,
                              width: Constants.barChartLegendSquareSize,
                              height: Constants.barChartLegendSquareSize)

let legendSquarePath = CGMutablePath()
legendSquarePath.addRect( legendSquareRect )
context?.addPath(legendSquarePath)
context?.setFillColor(fileTypeColors.fillColor.cgColor)
context?.setStrokeColor(fileTypeColors.strokeColor.cgColor)
context?.drawPath(using: .fillStroke)

// 2
let paragraphStyle = NSMutableParagraphStyle()
paragraphStyle.lineBreakMode = .byTruncatingTail
paragraphStyle.alignment = .left
let nameTextAttributes = [
  NSFontAttributeName: NSFont.barChartLegendNameFont,
  NSParagraphStyleAttributeName: paragraphStyle]

// 3
let nameTextSize = fileType.name.size(withAttributes: nameTextAttributes)
let legendTextOriginX = legendSquareRect.maxX + Constants.legendTextMargin
let legendTextOriginY = legendOriginY - 2 * Constants.pieChartBorderWidth
let legendNameRect = CGRect(x: legendTextOriginX, y: legendTextOriginY,
                            width: legendRectWidth - legendSquareRect.size.width - 2 *
                              Constants.legendTextMargin,
                            height: nameTextSize.height)

// 4
fileType.name.draw(in: legendNameRect, withAttributes: nameTextAttributes)

// 5
let bytesText = bytesFormatter.string(fromByteCount: fileTypeInfo.bytes)
let bytesTextAttributes = [
  NSFontAttributeName: NSFont.barChartLegendSizeTextFont,
  NSParagraphStyleAttributeName: paragraphStyle,
  NSForegroundColorAttributeName: NSColor.secondaryLabelColor]
let bytesTextSize = bytesText.size(withAttributes: bytesTextAttributes)
let bytesTextRect = legendNameRect.offsetBy(dx: 0.0, dy: -bytesTextSize.height)
bytesText.draw(in: bytesTextRect, withAttributes: bytesTextAttributes)
```

看起来这一大堆代码还挺唬人的，其实很简单：

1. 你已经很熟悉这一段代码了：计算图例的彩色方块的位置，为它创建一条路径，并用对应的颜色填充；
2. 创建一个字典，包含了两个属性：字体和 `NSMutableParagraphStyle`。后者会定义这些文本会怎样在给定的矩形里被绘制出来。在这个例子中，文本会显示为左对齐，且若文本超出了矩形范围，系统会在他的末尾加上省略号；
3. 计算用于绘制文本的矩形的位置和大小；
4. 调用 `draw(in:withAttributes:)`，绘制文本；
5. 使用 `bytesFormatter` 获取文本，并设置「文件大小」的文本的属性。这和之前唯一的区别是：这个文本用 `NSFontAttributeName` 设置了一个不同的颜色。

编译并运行，或者前往 **Main.storyboard**。

![](http://upload-images.jianshu.io/upload_images/5477931-bf60da1d58848b16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

热烈祝贺条形图杀青成功！你现在可以调整一下窗口的大小，看看图例里的文本时如何给自己加上省略号来适应狭小的空间的。

给自己鼓个掌吧👏～

# Cocoa Drawing

**macOS** 还提供了使用 **AppKit** 的框架来进行绘制的选项。它将会提供更高层的抽象绘图法。它使用各种类来代替 **C 语言**的函数，它还包含了许多 Helper 方法来更轻松地应对常见绘图任务。在两个框架中，图形上下文是一样的。如果你对 Core Graphics 很熟悉的话，你应该能很轻松地掌握 Cocoa Drawing。

和 Core Graphics 一样，你需要创建并绘制路径，但在 Cocoa Drawing，我们使用 `NSBezierPath`，它和 `CGPathRef` 是一样的。

我们要绘制的饼图是这样的：

![](http://upload-images.jianshu.io/upload_images/5477931-e246d4795e5dec1b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

你需要分三步来绘制它：

![](http://upload-images.jianshu.io/upload_images/5477931-bda1ff3ab8d8e064.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 创建一条圆形的路径，用于显示总硬盘空间，然后用定义好的颜色绘制它；
2. 为已用空间创建一条路径，并绘制它；
3. 为已用空间的路径绘制一个渐变填充。

打开 **GraphView.swift**，把这个方法添加到用于绘制的 extension 里：

```swift
func drawPieChart() {
  guard let fileDistribution = fileDistribution else {
    return
  }
  
  // 1
  let rect = pieChartRectangle()
  let circle = NSBezierPath(ovalIn: rect)
  pieChartAvailableFillColor.setFill()
  pieChartAvailableLineColor.setStroke()
  circle.stroke()
  circle.fill()
  
  // 2
  let path = NSBezierPath()
  let center = CGPoint(x: rect.midX, y: rect.midY)
  let usedPercent = Double(fileDistribution.capacity - fileDistribution.available) /
    Double(fileDistribution.capacity)
  let endAngle = CGFloat(360 * usedPercent)
  let radius = rect.size.width / 2.0
  path.move(to: center)
  path.line(to: CGPoint(x: rect.maxX, y: center.y))
  path.appendArc(withCenter: center, radius: radius,
                                         startAngle: 0, endAngle: endAngle)
  path.close()
  
  // 3
  pieChartUsedLineColor.setStroke()
  path.stroke()
}
```

我们来分析一下这段代码：

1. 用 `init(ovalIn:)` 构造方法创建一条圆形的路径，设置它的填充颜色和笔触颜色，然后绘制这条路径；
2. 为已用空间创建一条路径：
	- 根据已用空间计算扇形的角度；
	- 移动到大圆的圆心；
	- 添加一条从圆心到圆的右顶点的线段；
	- 根据之前计算的角度添加一条圆弧；
	- 闭合图形，也就是从圆弧的终点连接到圆心；
3. 调用 `stroke()` 方法，设置笔触颜色；

你应该能发现这段代码和之前的区别：

- 代码中没有提到过图形上下文，因为我们调用的方法会自动获取当前的上下文，在这个例子中，就是视图自己的图形上下文；
- 角是以角度制计算，而不是弧度制。**CGFloat+Radians.swift** 扩展了 CGFloat 类来进行了自动转换。

现在把这行代码添加到 `draw(_:)` 方法中来绘制饼图：

```swift
drawPieChart()
```

编译并运行：

![](http://upload-images.jianshu.io/upload_images/5477931-5169614dde1ef08a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

进展不错！

## 绘制渐变

**Cocoa Drawing** 使用 `NSGradient` 来绘制渐变。

你需要在已使用的扇形里绘制渐变，该怎么实现呢……🤔？

没错，用剪切蒙版啊！

你已经创建了一条路径来绘制已用空间的扇形，在我们绘制渐变之前，先来把它用作剪切蒙版。

把这些代码添加到 `drawPieChart()` 方法中：

```swift
if let gradient = NSGradient(starting: pieChartGradientStartColor,
                             ending: pieChartGradientEndColor) {
  gradient.draw(in: path, angle: Constants.pieChartGradientAngle)
}
```

第一行代码会试着去创建一个两种颜色构成的渐变。如果创建成功了，就会调用 `draw(in:angle:)` 方法来绘制它。在大括号里，这个方法会设置蒙版，并在蒙版区域内绘制渐变。是不是特别棒～

编译并运行：

![](http://upload-images.jianshu.io/upload_images/5477931-27ddddbb77a21564.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 练习/挑战：绘制饼图的图例

现在我们的自定义视图已经越来越完美了，但还有一个待办事项：绘制饼图的图例，也就是其内部的文字说明。

你已经知道该怎么去做了，准备好接受挑战了嘛💪？

一些小提示：

1. 使用 `bytesFormatter` 来获取硬盘的可用空间（`fileDistribution.available` 属性）和总空间（`fileDistribution.capacity` 属性）；
2. 计算文本的位置，确保你的文本显示在各个扇形的中央；
3. 在计算好的位置用以下属性绘制文本：
	- Font：`NSFont.pieChartLegendFont`；
	- Used space text color：`NSColor.pieChartUsedSpaceTextColor`；
	- Available space text color：`NSColor.pieChartAvailableSpaceTextColor`。  

# 答案：绘制图例

把这些代码添加到 `drawPieChart()` 方法中：

```swift
// 1
let usedMidAngle = endAngle / 2.0
let availableMidAngle = (360.0 - endAngle) / 2.0
let halfRadius = radius / 2.0

// 2
let usedSpaceText = bytesFormatter.string(fromByteCount: fileDistribution.capacity)
let usedSpaceTextAttributes = [
  NSFontAttributeName: NSFont.pieChartLegendFont,
  NSForegroundColorAttributeName: NSColor.pieChartUsedSpaceTextColor]
let usedSpaceTextSize = usedSpaceText.size(withAttributes: usedSpaceTextAttributes)
let xPos = rect.midX + CGFloat(cos(usedMidAngle.radians)) *
  halfRadius - (usedSpaceTextSize.width / 2.0)
let yPos = rect.midY + CGFloat(sin(usedMidAngle.radians)) *
  halfRadius - (usedSpaceTextSize.height / 2.0)
usedSpaceText.draw(at: CGPoint(x: xPos, y: yPos),
                   withAttributes: usedSpaceTextAttributes)

// 3
let availableSpaceText = bytesFormatter.string(fromByteCount: fileDistribution.available)
let availableSpaceTextAttributes = [
  NSFontAttributeName: NSFont.pieChartLegendFont,
  NSForegroundColorAttributeName: NSColor.pieChartAvailableSpaceTextColor]
let availableSpaceTextSize = availableSpaceText.size(withAttributes: availableSpaceTextAttributes)
let availableXPos = rect.midX + cos(-availableMidAngle.radians) *
  halfRadius - (availableSpaceTextSize.width / 2.0)
let availableYPos = rect.midY + sin(-availableMidAngle.radians) *
  halfRadius - (availableSpaceTextSize.height / 2.0)
availableSpaceText.draw(at: CGPoint(x: availableXPos, y: availableYPos),
                        withAttributes: availableSpaceTextAttributes)
```

代码含义：

1. 计算两个区域的角度；
2. 创建已用空间的文本的属性，并计算其 *x* 和 *y*，然后绘制它；
3. 创建总空间的文本的属性，并计算其 *x* 和 *y*，然后绘制它；

现在，编译并运行你的 app，好好欣赏一下你的杰出作品：

![](http://upload-images.jianshu.io/upload_images/5477931-2b67c2e23ffc4526.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
恭喜你！你使用 Core Graphics 和 Cocoa Drawing 创建了一个美丽的 app！

# 接下来该做啥？

你可以[点击这里](https://koenig-media.raywenderlich.com/uploads/2016/06/DiskInfo-Final.zip)下载完整的工程文件。

这个 macOS Core Graphics 教程覆盖了 macOS 中用于绘制自定义视图的不同框架的基本知识：

- 如何使用 Core Graphics 和 Cocoa Drawing 创建和绘制路径；
- 如何剪切一个区域；
- 如何绘制文本串；
- 如何绘制渐变。

之后的日子里，当你需要创建一些整洁、优美的用户界面的时候，你应该能自信地拿出 Core Graphics 和 Cocoa Drawing 挥洒创意了。

如果你还想继续深入，可以参考这些资源：

- Apple 的 [Introduction to Cocoa Drawing Guide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CocoaDrawingGuide/Introduction/Introduction.html)
- Apple 的 [Quartz 2D Programming Guide](https://developer.apple.com/library/mac/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/Introduction/Introduction.html)