---
title: 【译】NSCollectionView 入门教程
date: 2017-09-24 22:34:11
tags:
  - 翻译
  - Ray Wenderlich
  - macOS
  - Swift
---

> 本文翻译自 [raywenderlich.com](https://www.raywenderlich.com/) 的 [NSCollectionView Tutorial](https://www.raywenderlich.com/145978/nscollectionview-tutorial)，已咨询对方网站，可至多翻译 10 篇文章。
> 希望各位有英语阅读能力的话，还是~~先打赏~~然后去阅读英文原吧，毕竟无论是 Xcode，抑或是官方的文档，还是各种最前沿的资讯都只有英文版本。  
> 综上，此翻译版本仅供参考，**谢绝转载**。

**更新信息：** 此 NSCollectionView 教程已由 Gabriel Miro 更新至 Xcode 8 和 Swift 3.
 
Collection View 是现实一系列相同类型数据的最佳方式。Mac 中自带的 Finder 和 Photos 就是使用了它：通过一个 Collection View 来展示所有的文件和图片。

`NSCollectionView` 最早在 OS X 10.5 被推出，它可以非常方便地布局一组具有相同大小的 item，并把它们展示在一个可以滑动的 Scroll View 中。

在 OS X 10.11 El Capitan 中，参照 iOS 上的 `UICollectionView`，`NSCollectionView` 被全面进行了升级。

macOS 10.12 Sierra 则给予了它「收起分区」（就像 Finder 里那样）和「固定标题」两项新功能，使得它和 iOS 的差距进一步减小了。

在这个 NSCollectionView 的入门教程中，你将会创造一个叫 **SlideMagic** 的 app，它是一个只属于你的网格状的图片浏览 app。

这个教程假定你已经基本了解过了 macOS app 的开发，如果你还不曾了解过，raywenderlich.com 上提供了很多很棒的 [macOS 开发教程](http://www.raywenderlich.com/category/macos)，你可以先去看看那些。
> 当然还有[我自己翻的《零基础 macOS 应用开发教程》](http://www.jianshu.com/p/a3f16178a213)系列

# 准备开始

你将会编写的 **SlidesMagic** app 是一个简单的图片浏览器，它很酷，但是，可别因为它太酷了而一不小心在把玩的时候把自己 Mac 上的照片删了哦😛～

这个 app 会从获取一个文件夹里的所有图片，然后用一个极其优雅的 Collection View 来把它们显示出来。完成了的 app 长这样：

![](http://upload-images.jianshu.io/upload_images/5477931-6db9bff0a7b673ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[下载这个项目的起步代码](https://koenig-media.raywenderlich.com/uploads/2016/10/SlidesMagic8V2-Starter.zip)，编译并运行:

![](http://upload-images.jianshu.io/upload_images/5477931-213e3900e938a253.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此时，这个 app 看起来只是一个空荡荡的窗口，但这些起步代码包含了一些「隐藏功能」，这是后面使它成为一个图片浏览器的基础。

**SlidesMagic** 启动的时候，会自动加载系统中 **Desktop Pictures** 目录下的所有图片，在 Xcode 的控制台输出中，我们可以看到这些文件的名字。

![](http://upload-images.jianshu.io/upload_images/5477931-283174b11475b7cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

控制台中输出的列表表明，起步代码中 Model 的加载逻辑代码已经可以正常工作了，你可以在这个 app 的 **File** → **Open Another Folder…** 菜单中打开另一个目录。

## 起步代码

起步代码提供了一些与 Collection Views 无直接关联的代码。

![](http://upload-images.jianshu.io/upload_images/5477931-ef6b2cfd384dd183.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Model

- **ImageFile.swift**: 用于描述一个图片文件
- **ImageDirectoryLoader.swift**: 用来把图片从硬盘中加载出来的 Helper 类

## Controller

这个 app 拥有两个主要的 Controller：

- **WindowController.swift**：
	- `windowDidLoad()`：在左半边的屏幕上设置主窗口的大小；
	- `openAnotherFolder(_:)`：提供一个标准的「打开」对话框来供用户选择文件夹；
- **ViewController.swift**：
	- `viewDidLoad()` 打开 Desktop Pictures 目录作为默认目录。

# Collection View 幕后探秘

`NSCollectionView` 是今天的主角，它将会在几个关键的组成部分的帮助下，显示许多 item。

## 布局

`NSCollectionViewLayout`：明确了 Collection View 的布局方式，它是一个抽象的类，所有用来表示 CollectionView 布局的实类都继承自它。

`NSCollectionViewFlowLayout`：提供了一个灵活的网格状的布局。对于绝大多数 app，这种布局方式都适用。

`NSCollectionViewGridLayout`：为了兼容 OS X 10.11 和以前的版本所保留的布局方式，对于新创建的 app 不推荐使用。

![](http://upload-images.jianshu.io/upload_images/5477931-cc118c34a1553151.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Section 和 `IndexPath`**：前者允许你把 item 分成若干个 section（分区），每个 section 包含了一组有序的 item。每个 item 都和一个索引相关联，这个索引是一个由一对整数（section，item）构成的 `IndexPath` 实例。默认情况下，当你不需要给 item 分区时，这个 Collection View 仍然会拥有一个 section。

![](http://upload-images.jianshu.io/upload_images/5477931-bd89941692bf5bba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Collection View Item

就像其他许多 **Cocoa** 框架一样，Collection View 中的 item 也遵守着 **MVC** 设计模式。

**Model 和 View**：这个 item 的内容来自 Model 的数据对象。每个单独的对象都通过在 Collection View 中创建自己的 View 来把自己显示出来。这些 View 的结构由一个单独的 nib 文件（文件扩展名是 .xib）来定义。

**Controller**：上面提到的 nib 文件是一个由 **NSCollectionViewItem** 管理的 `NSViewController` 的子类。它负责与 Model 对象进行通信并控制 Collection View 的显示。通常情况下，你会编写一个 `NSCollectionViewItem` 的子类。当你需要不同类型的 item 的时候，你需要为每个分支定义一个不同的子类，并创建一个 nib。

## 额外的 View

要在 Collection View 中显示不同于普通 item 额外的信息，你需要额外的 item。

最直观的例子就是分区的标题和脚注。

## Collection View 的数据源和代理（Data Source and Delegates）

* `NSCollectionViewDataSource`：用 item 和额外的 item 来填充 Collection View。
* `NSCollectionViewDelegate`：处理拖放相关的事件，以及选中状态和高亮。
* `NSCollectionViewDelegateFlowLayout`：允许你自定义你的网格视图。

> **注意**：填充一个 Collection View 的方法有二：数据源和 Cocoa 绑定。这个教程将会使用数据源。

# 创建 Collection View

打开 **Main.storyboard**。前往**控件库**，向 **View Controller Scene** 中拖动一个 **Collection View**。

![](http://upload-images.jianshu.io/upload_images/5477931-a96ea28b4e1256e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>**注意**：你或许注意到了，**Interface Builder** 为我们添加了三个 View，而不是一个。这是因为 **Collection View** 是嵌入在一个 **Scroll View** 中的，而后者又自带一个 **Clip View** 子视图。这仨视图各不相同，因此当本教程需要你选择 **Collection View** 的时候，切记不要错选了 **Scroll View** 或 **Clip View**。

调整 **Bordered Scroll View** 的大小，使它填满它的父视图的所有空间。然后选择 Xcode 菜单栏上的 **Editor** → **Resolve Auto Layout Issues** → **Add Missing Constraints** 来添加 **Auto Layout** 约束条件。

![](http://upload-images.jianshu.io/upload_images/5477931-6fe7039d2d1a1b21.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

你需要在 **ViewController** 中添加一个 Outlet 来访问界面上的 Collection View。打开 **ViewController.swift**，在 `ViewController` 类的定义中添加以下代码：

```swift
@IBOutlet weak var collectionView: NSCollectionView!
```

打开 **Main.storyboard**，并选择 **View Controller Scene** 中的 **View Controller**。

打开**连接检查器**，在 **Outlets** 部分中找到 **collectionView**，**拖动**它旁边的小圆圈到画布中的 Collection View 上。

![](http://upload-images.jianshu.io/upload_images/5477931-705deb60d6892d5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 调整 Collection View 的布局

你现在有两种选择：在 **Interface Builder** 中设置好主要的布局属性，或者在代码中手动编写。

在 SlidesMagic 这个项目中，我们选择手动编写代码。

打开 **ViewController.swift**，把这些方法添加到 `ViewController` 中：

```swift
  fileprivate func configureCollectionView() {
    // 1
    let flowLayout = NSCollectionViewFlowLayout()
    flowLayout.itemSize = NSSize(width: 160.0, height: 140.0)
    flowLayout.sectionInset = EdgeInsets(top: 10.0, left: 20.0, bottom: 10.0, right: 20.0)
    flowLayout.minimumInteritemSpacing = 20.0
    flowLayout.minimumLineSpacing = 20.0
    collectionView.collectionViewLayout = flowLayout
    // 2
    view.wantsLayer = true
    // 3
    collectionView.layer?.backgroundColor = NSColor.black.cgColor
  }
```

这些代码的作用是：

1. 创建一个 `NSCollectionViewFlowLayout`，配置它的基本属性，并设置 `NSCollectionView` 的 `collectionViewLayout`；
2. 一般情况下，`NSCollectionView` 是基于层的，所以你需要把它的父视图的 `wantsLayer` 设置为 `true`；
3. 把 Collection View 的背景颜色设置为黑色。

你需要在试图加载完成时调用这个方法，所以在 `viewDidLoad()` 方法的最后插入：

```swift
configureCollectionView()
```

编译并运行：

![](http://upload-images.jianshu.io/upload_images/5477931-b9359a0cde32671d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此时，你的 Collection View 已经拥有了一个黑色的背景，并配置好了布局。

# 创建一个 Collection View Item

先在你需要创建一个 `NSCollectionViewItem` 的子类并把 Model 里的数据们显示出来。

点击 Xcode 菜单栏上的 **File** → **New** → **File…**，选择 **macOS** → **Source** → **Cocoa Class** 并点击 **Next**。

把 **Class** 填写 **CollectionViewItem**，**Subclass of** 填写 `NSCollectionViewItem`，并勾选 **Also create XIB for user interface**。

![](http://upload-images.jianshu.io/upload_images/5477931-7edf2b2db9985f31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
点击 **Next**，然后在对话框中的 **Group** 中选择 **Controllers**，并点击 **Create**。

打开 **CollectionViewItem.swift**，把里边的内容全部替换为：

```swift
import Cocoa
class CollectionViewItem: NSCollectionViewItem {

  // 1
  var imageFile: ImageFile? {
    didSet {
      guard isViewLoaded else { return }
      if let imageFile = imageFile {
        imageView?.image = imageFile.thumbnail
        textField?.stringValue = imageFile.fileName
      } else {
        imageView?.image = nil
        textField?.stringValue = ""
      }
    }
  }
  
  // 2
  override func viewDidLoad() {
    super.viewDidLoad()
    view.wantsLayer = true
    view.layer?.backgroundColor = NSColor.lightGray.cgColor
  }
}
```

这些代码的功能是：

1. 定义了 `imageFile` 属性，用来访问需要展示的 Model 对象。当你为 `imageFile` 属性赋值时，它的 `didSet` 属性观察器会设置这个 item 的 Image 和 Label；
2. 改变此 item 的 View 的背景颜色。

# 向 View 中添加 Control

你在 **CollectionViewItem.swift** 时勾选了「Also create a XIB（同时创建一个 XIB）」，为了更清楚地整理文件，把 **CollectionViewItem.xib** 拖动到 **Main.storyboard** 下方的 **Resources** 分组中。

Nib 文件中的 **View** 就是每个 item 所显示出来的根视图，你需要添加一个 Image View 来显示图片，以及一个 Label 来显示文件名。

打开 **CollectionViewItem.xib**，添加一个 `NSImageView`：

1. 从**控件库**中拖动一个 **Image View** 到画布上的 **View** 中；
![](http://upload-images.jianshu.io/upload_images/5477931-da794ce70a3f8239.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. 在 **Auto Layout** 工具栏中点击 **Pin** 按钮来设置它的约束条件；
3. 设置它的 **top**、**leading** 和 **trailing** 约束为 0，**bottom** 为 30。点击 **Update Frames: Items of New Constraints** 然后点击 **Add 4 Constraints**。
![](http://upload-images.jianshu.io/upload_images/5477931-2cebd9643e49bae7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
再来添加一个 Label：

1. 从**控件库**中拖动一个 **Label** 到画布上的 **Image View** 的下方；
![](http://upload-images.jianshu.io/upload_images/5477931-8662823f27453bff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. 在 **Auto Layout** 工具栏中点击 **Pin** 按钮，设置它的 **top**、**bottom**、**leading** 和 **trailing** 约束都为 0。点击 **Update Frames: Items of New Constraints** 然后点击 **Add 4 Constraints**。
![](http://upload-images.jianshu.io/upload_images/5477931-3589ffea0a0ecce4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选中 **Label**，在**属性检查器**中设置如下属性：

1. 设置 **Alignment** 为 **center**
2. 设置 **Text Color** 为 **white**
3. 设置 **Line Break** 为 **Truncate Tail**
![](http://upload-images.jianshu.io/upload_images/5477931-dd778a91d505a2e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 向 Nib 中添加 CollectionViewItem 并连接 Outlets

尽管 Nib 文件的 **File’s Owner** 现在是 `CollectionViewItem`，它还只是一个占位符。当 Nib 文件被实例化时，它还会需要一个「真正的」`NSCollectionViewItem` 的实例。

从**控件库**中拖动一个 **Collection View Item** 到**文档大纲**中，选中它，在 **身份检查器**中把它的 **Class** 设置为 `CollectionViewItem`。

![](http://upload-images.jianshu.io/upload_images/5477931-865306100dff432a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 xib 中，你需要把 View 的层次关系连接到 **CollectionViewItem** 的 Outlet 中，在 **CollectionViewItem.xib** 中：

1. 选中 **Collection View Item** 并前往 **Connections Inspector**；
2. 把 `view` 的 outlet 拖动到**文档大纲**中的 **View** 上；
3. 用同样的方法，把 `imageView` 和 `textField` 的 outlet 连接到**文档大纲**中的 **Image View** 和 **Label** 中。
![](http://upload-images.jianshu.io/upload_images/5477931-111e8db6987f691b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 填充 Collection View

你需要实现 Collection View 的数据源协议，说人话就是：

1. Collection 中有几个分区？
2. 每个分区分别有多少个 item？
3. 某个索引路径（Index Path）对应的是哪个 Item？

打开 **ViewController.swift** 并在文件的末尾添加这些扩展代码：

```swift
extension ViewController : NSCollectionViewDataSource {
  
  // 1
  func numberOfSections(in collectionView: NSCollectionView) -> Int {
    return imageDirectoryLoader.numberOfSections
  }
  
  // 2
  func collectionView(_ collectionView: NSCollectionView, numberOfItemsInSection section: Int) -> Int {
    return imageDirectoryLoader.numberOfItemsInSection(section)
  }
  
  // 3
  func collectionView(_ itemForRepresentedObjectAtcollectionView: NSCollectionView, itemForRepresentedObjectAt indexPath: IndexPath) -> NSCollectionViewItem {
    
    // 4
    let item = collectionView.makeItem(withIdentifier: "CollectionViewItem", for: indexPath)
    guard let collectionViewItem = item as? CollectionViewItem else {return item}
    
    // 5
    let imageFile = imageDirectoryLoader.imageFileForIndexPath(indexPath)
    collectionViewItem.imageFile = imageFile
    return item
  }
  
}
```

1. 如果你的 app 不需要用到分区，那么你可以删除这个方法，因为一个分区就够了；
2. 这是两个 `NSCollectionViewDataSource` 协议必须实现的方法之一，在这个方法中你需要返回某个分区容纳的 item 的数量；
3. 另一个必须实现的方法，这个方法会针对某个 `indexPath` 返回一个 item；
4. 这个方法会从 nib 中实例化一个 item，这个 item 的名字是 `identifier` 参数，它会根据所需要的 item 的类型来试图重复使用一个 item，如果没有 item 可供重复使用，它会新建一个 item；
5. 根据 `IndexPath` 获取 Model 对象，设置 Image 和 Label 的内容。

> **注意**: Collection View 具有一项能力：循环使用已经生成了的 Item，以此来减轻数据源过大时的内存压力。从界面上移出去的 item 就是被重复使用的 item。

# 设置数据源

接下来我发需要定义数据源：

打开 **Main.storyboard**，选中 Collection View。

打开**连接检查器**，在 **Outlets** 部分中找到 **dataSource**，**拖动**它旁边的小圆圈到**文档大纲**里的 **View Controller** 上。
![](http://upload-images.jianshu.io/upload_images/5477931-7fb81eeed145f612.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

编译并运行，你的 Collection View 现在应该能显示 **Desktop Pictures** 目录中的图片了：

![](http://upload-images.jianshu.io/upload_images/5477931-1422559e2aea84ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

哈哈哈，折腾了半天都是值得的✌️～

## 故障排除

如果你还看不见任何图片，你可能遗漏了一些小细节：

1. 你在**连接检查器**中正确地设置了所有的连接吗？
2. 你设置 `dataSource` 的 Outlet 了吗？
3. 你在**身份检查器**中应用正确的自定义类了吗？
4. 你添加顶层的 `NSCollectionViewItem`，并把它的类设置为 `CollectionViewItem` 了吗？
5. `makeItemWithIdentifier` 中的 `identifier` 参数的值和 nib 的名字一样吗？

# Model 发生改变时重新加载 Item 们

要显示另一个目录中的图片，你可以在 app 的菜单栏上点击 **File** → **Open Another Folder…**，然后选择一个存有 **JPG** 或 **PNG** 格式的图片的目录。

但时此时窗口中的东西似乎什么变化都没有，还是显示着 **Desktop Pictures** 目录中的图片。尽管 Xcode 里的控制台中已经打印出了新目录里的文件名称。

你需要调用 Collection View 的 `reloadData()` 方法来刷新它的 item。

打开 **ViewController.swift** 并把这些代码添加到 `loadDataForNewFolderWithUrl(_:)` 方法中：

```swift
collectionView.reloadData()
```

编译并运行，现在你应该能看到窗口中已经能显示正确的照片了。

# 添加分区

SlidesMagic app 现在已经可以做一些很神奇的事儿了，但我们要更进一步 —— 为 Collection View 加入分区。

首先，你需要在主视图的最底部加入一个复选框，来允许你切换是否启用分组。

打开 **Main.storyboard**，然后在**文档大纲**中选中 Scroll View 的约束条件，在**尺寸检查器**中吧它的 **Constant** 修改为 30.

这会把 Collection View 抬高一些些，腾出地方来放置复选框。

![](http://upload-images.jianshu.io/upload_images/5477931-b811bef73ed6329e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在，从**控件库**中拖动一个 **Check Box Button**到画布中 Collection View 下方的空间里，选中它，在**属性检查器**中把它的 **Title** 设置为 **Show Sections**，然后把 **State** 设置为 **Off**。

![](http://upload-images.jianshu.io/upload_images/5477931-14190a9187e4912b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来，点击 **Pin** 按钮更新它的 **Auto Layout** 约束条件：**Top** 设置为 8，**Leading** 设置为 20。然后点击 **Update Frames: Items of New Constraints** 和 **Add 2 Constraints**

编译并运行，现在 app 的底部看起来应该是这样的：

![](http://upload-images.jianshu.io/upload_images/5477931-a2538c0d623bc2e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当你点击这个复选框的时候，你的 app 需要改变 Collection View 的外观。

打开 **ViewController.swift** 在 `ViewController` 类的最后添加这些代码：

```swift
  @IBAction func showHideSections(sender: NSButton) {
    let show = sender.state
    // 1
    imageDirectoryLoader.singleSectionMode = (show == NSOffState)
    // 2
    imageDirectoryLoader.setupDataForUrls(nil)
    // 3
    collectionView.reloadData()
  }
```

这些代码会：

1. 根据复选框的状态切换单一分组/多个分组；
2. 根据当前选择的模式来调整 Model，此时传递的 `nil` 参数表示跳过图像加载 —— 毕竟图片还是那些图片，只是布局发生了改变；
3. Model 发生了改变，所以需要刷新数据。

如果你好奇图片是按照是按照什么规则进行分组的，在 `ImageDirectoryLoader` 中找到 `sectionLengthArray`，这个数组里的数字设置了各个分组的里最多可以容纳多少个 item。这个数组是随机生成的，只是用来用作演示。

现在，打开 **Main.storyboard**。在**文档大纲**中**按住 Control⌃ 键的同时**把 **Show Sections** 拖动到 **View Controller** 上。在弹出的黑色窗口中点击 **showHideSections:**。你可以在**连接检查器**里查看你是否连接成功了。

![](http://upload-images.jianshu.io/upload_images/5477931-6656f84c5d2c2726.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

编译并运行，勾选 **Show Sections** 来查看布局的变化。

![](http://upload-images.jianshu.io/upload_images/5477931-15e1753a88dfc587.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

为了更好地区分各个分区，打开 **ViewController.swift**，编辑 `configureCollectionView()` 方法里的 `sectionInset` 属性。

把这一行：

```swift
flowLayout.sectionInset = EdgeInsets(top: 10.0, left: 20.0, bottom: 10.0, right: 20.0)
```

替换成这个：

```swift
flowLayout.sectionInset = EdgeInsets(top: 30.0, left: 20.0, bottom: 30.0, right: 20.0)
```

编译并运行，勾选 **Show Sections**，可以看到各个分区之间已经有了分隔。

![](http://upload-images.jianshu.io/upload_images/5477931-fb004f710558b192.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 添加分区标题

另一种区分各个分区边界的方法是为每个分区添加一个标题或脚注。

你需要一个自定义的 `NSView` 类，并实现相应的数据源方法来为 Collection View 添加一个标题

要创建一个标题，在 Xcode 的菜单栏点击 **File** → **New** → **File…**。选择 **macOS** → **User Interface** → **View**，并点击 **Next**。

文件名输入 **HeaderView.xib**，**Group** 选择 **Resources**。

点击 **Create**。

![](http://upload-images.jianshu.io/upload_images/5477931-d7f8fd4c5b749b7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打开 **HeaderView.xib** 并选中 **Custom View**。在**尺寸检查器**中把 **Width** 设置为 500，**Height** 设置为 40。

从 **Object Library** 拖动一个 Label 到 **Custom View** 的左半边。打开**属性检查器**，设置它的 **Title** 为 **Section Number**，设置 **Font Size** 为 **16**。

再拖动一个 Label 到 **Custom View** 的右半边。设置它的 **Title** 为 **Image Count**，设置 **Alignment** 为 **Right**。

选中 **Section Number** Label，点击 **Pin** 按钮，设置它的 **Top** 约束为 12，**Leading** 约束为 20。点击 **Update Frames: Items of New Constraints** 和 **Add 2 Constraints**。

接下来，设置 **Image Count** Label 的 **Top** 约束为 11，**Trailing** 约束为 20，别忘了点击 **Update Frames: Items of New Constraints** 和 **Add 2 Constraints**。

现在我们的标题应该看起来像这样：

![](http://upload-images.jianshu.io/upload_images/5477931-219024e618e0e957.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在我们的标题 UI 已经准备好了，我们还需要为它创建一个子类。

在 Xcode 的菜单栏点击 **File** → **New** → **File…**。选择 **macOS** → **Source** → **Cocoa Class**，并点击 **Next**。把它的类名设置为 `HeaderView`，并让它继承自 `NSView`，点击 **Next**，并在 **Group** 中选择 **Views**。点击 **Create**。

打开 **HeaderView.swift** 然后把里边的所有内容替换为：

```swift
// 1
@IBOutlet weak var sectionTitle: NSTextField!
@IBOutlet weak var imageCount: NSTextField!

// 2
override func draw(_ dirtyRect: NSRect) {
  super.draw(dirtyRect)
  NSColor(calibratedWhite: 0.8 , alpha: 0.8).set()
  NSRectFillUsingOperation(dirtyRect, NSCompositingOperation.sourceOver)
}
```

这里的代码做了这些事儿：

1. 设置你需要用来连接 nib 元素的 outlet；
2. 绘制一个灰色的背景。

要把 outlet 连接至 Label，打开 **HeaderView.xib** 并选中 **Custom View**。在 **Identity Inspector** 中把 **Class** 设置为 **HeaderView**。

![](http://upload-images.jianshu.io/upload_images/5477931-7293eacf2ad306dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在**文档大纲**视图中，按住 **Control⌃ 键**的同时点击 **Header View**。在弹出的黑色窗口中，拖动 **imageCount** 到 **Images Count** 上来连接 outlet。 

对第二个 Label 进行同样的操作，拖动 **sectionTitle** 到画布中的 **Section Number** Label 上。

![](http://upload-images.jianshu.io/upload_images/5477931-68e5abf430d026b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 实现数据源和代理方法

你的标题已经完全准备好上战场了，你需要实现 `collectionView(_:viewForSupplementaryElementOfKind:at:)`，把这个标题视图传递给 Collection View：

打开 **ViewController.swift** 并把这些方法添加到 `NSCollectionViewDataSource` extension 中：

```swift
func collectionView(_ collectionView: NSCollectionView, viewForSupplementaryElementOfKind kind: String, at indexPath: IndexPath) -> NSView {
  // 1
  let view = collectionView.makeSupplementaryView(ofKind: NSCollectionElementKindSectionHeader, withIdentifier: "HeaderView", for: indexPath) as! HeaderView
  // 2
  view.sectionTitle.stringValue = "Section \(indexPath.section)"
  let numberOfItemsInSection = imageDirectoryLoader.numberOfItemsInSection(indexPath.section)
  view.imageCount.stringValue = "\(numberOfItemsInSection) image files"
  return view
}
```

Collection View 会在它需要数据源的时候调用这个方法，并为每个分区设置标题。这个方法做了这些：

1. 调用 `makeSupplementaryViewOfKind(_:withIdentifier:for:)` 来从 nib 文件实例化一个名字是 `withIdentifier` 的 `HeaderView` 对象；
2. 设置各个 Label 的值。

在 **ViewController.swift** 的最后，添加这个 `NSCollectionViewDelegateFlowLayout` 扩展：

```swift
extension ViewController : NSCollectionViewDelegateFlowLayout {
  func collectionView(_ collectionView: NSCollectionView, layout collectionViewLayout: NSCollectionViewLayout, referenceSizeForHeaderInSection section: Int) -> NSSize {
    return imageDirectoryLoader.singleSectionMode ? NSZeroSize : NSSize(width: 1000, height: 40)
  }
}
```

上面这个方法其实是不是必须的，但当你需要设置标题的时候就必须写上了，因为 Flow Layout（流式布局）的代理需要你提供各个分区的标题的大小。

如果没有实现这个方法，你将看不到标题，因为它们的尺寸都是 0。此外，它还会忽略你指定的宽度，而是把标题的宽度设置为 Collection View 的宽度。

在这个例子中，当 Collection View 只有一个分区的时候，这个方法返回的标题尺寸是 0，否则他会返回 40.

对于使用了 `NSCollectionViewDelegateFlowLayout` 的 Collection View，你需要把 `ViewController` 连接到 `NSCollectionView` 的 `delegate`。

打开 **Main.storyboard** 并选中 Collection View。打开**连接检查器**，在 **Outlets** 部分中找到 **delegate**。**拖动**他旁边的小圆点到**文档大纲**中的 View Controller 上。

![](http://upload-images.jianshu.io/upload_images/5477931-e295106e936e58cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

编译并运行，勾选 **Show Sections**，可以看到一个个标题把分区区分开来：

![](http://upload-images.jianshu.io/upload_images/5477931-d7c86df41a0aac39.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 固定标题

macOS 10.12 中的 `NSCollectionViewFlowLayout` 新加入了两个属性：`sectionHeadersPinToVisibleBounds` 和 `sectionFootersPinToVisibleBounds`。

当 `sectionHeadersPinToVisibleBounds` 设置为 `true`，最上端的分区的标题将会固定在顶端，而不会移出界面以外。当你继续向下滚动时，下一个标题会把它顶走。这种效果一般被称为「sticky headers（固定标题）」或「floating headers（浮动标题）」。

把 `sectionFootersPinToVisibleBounds` 设置为 `true` 则会把脚注固定在底部。

打开 **ViewController.swift**，在 `configureCollectionView()` 方法的底部加入这个方法：

```swift
flowLayout.sectionHeadersPinToVisibleBounds = true
```

编译并运行，勾选 **Show Sections** 并向下滚动一些，你可以看到第一个区域已经有一些图片被移出屏幕了，但标题还是固定在顶部：

![](http://upload-images.jianshu.io/upload_images/5477931-93f6bc53d834485e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> **注意**：如果你的 app 需要支持 OS X 10.11 或更老的版本，你需要通过重写 `layoutAttributesForElements(in:)` 方法来「手动」实现固定标题。你可以查看[Advanced Collection Views in OS X Tutorial]这篇教程（我正在翻译～）。

# Collection View 的选择功能

为了显示一个 item 的被选中状态，你需要设置一个白色的边框，没有被选中的项目将不会显示这个边框。

首先，你需要让我们的 Collection View 支持选中。打开 **Main.storyboard**，选中 **Collection View** 并在**属性检查器**里，勾选 **Selectable**。

![](http://upload-images.jianshu.io/upload_images/5477931-c413394c21316342.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

勾选 **Selectable** 开启了选择功能，意味着你可以通过点击一个 item 来选中它。如果你点击另一个 item，将会取消选择之前的那个 item 并选中新的 item。

当你选中一个 item：

1. 它的 `IndexPath` 会被添加到 `NSCollectionView` 的 `selectionIndexPaths` 属性；
2. 它的 `isSelected` 属性会被设置为 `true`。

打开 **CollectionViewItem.swift**。在 `viewDidLoad()` 方法的最后追加：

```swift
// 1 
view.layer?.borderColor = NSColor.white.cgColor 
// 2 
view.layer?.borderWidth = 0.0
```

这段代码：

1. 设置了一个白色的边框；
2. 把 `borderWidth` 设置为 `0.0` 来确保边框不可见 —— 也就是没被选中。

要在每次 `isSelected` 被设置时改变 `borderWidth`，我们需要把这些代码添加到 `CollectionViewItem` 类中：

```swift
 override var isSelected: Bool {
    didSet {
      view.layer?.borderWidth = isSelected ? 5.0 : 0.0
    }
  }
```

每次 `isSelected` 发生了改变，`didSet` 将会根据新的值来设置边框的宽度。

编译并运行。点击一个项目来选中它，你将会看见它周围出现了边框。哈哈哈，神奇✨！

![](http://upload-images.jianshu.io/upload_images/5477931-f84ecab7bba30633.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 下一步该做些啥？

[点击这里](https://koenig-media.raywenderlich.com/uploads/2016/10/SlidesMagic8V3-Final.zip)下载最终完成了的 **SlideMagic**。

在这个 NSCollectionView 入门教程中，你了解了如何创建你的第一个 Collection View，了解了错综复杂的数据源 API 和如何处理分区。至此你已经学到了很多，但其实这仅仅是个开始，Collection View 还有很多功能等待你去发掘。这里有很多值得去探索的东西：

* 通过 Cocoa 的数据绑定构建「免数据源」的 Collection View
* 不同类型的 Item
* 追加和移除 Item
* 自定义布局
* 拖放手势（Drag and drop）
* 动画
* 修改 `NSCollectionViewFlowLayout`
* 收起某个分区（macOS 10.12 Sierra 的新功能）

你可以在我们的《NSCollectionView 进阶教程》中了解更多。