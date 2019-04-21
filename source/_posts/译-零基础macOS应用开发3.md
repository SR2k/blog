---
title: 【译】 零基础 macOS 应用开发（三）
date: 2017-09-23 03:54:25
tags:
  - 翻译
  - Ray Wenderlich
  - macOS
  - Swift
---

> 本文翻译自 [raywenderlich.com 的 macOS 开发经典入门教程](https://www.raywenderlich.com/151741/macos-development-beginners-part-1) ，已咨询对方网站，可至多翻译 10 篇文章。
> 希望各位有英语阅读能力的话，还是~~先打赏然后~~去阅读英文原吧，毕竟无论是 Xcode，抑或是官方的文档，还是各种最前沿的资讯都只有英文版本。  
> 综上，此翻译版本仅供参考，**谢绝转载**。 

欢迎回到我们的零基础 macOS 应用开发教程的最后一部分（共三部分）！

在第一部分中，你已经学会了如何安装 Xcode 和如何创建一个示例 app；在第二部分中你为一个更加复杂的 app 创建了 UI，但因为你还没有编写任何代码，所以它还不能工作。在这个部分中，你将会编写所有 Swift 代码并让你的 app 真正活起来！

# 开始

如果你还没有完成第二部分，或你希望从一个更加纯净的情况继续学习，你可以下载[第二部分中已经完成了 UI 布局的工程文件](https://koenig-media.raywenderlich.com/uploads/2017/01/EggTimer-Part2-3.zip)。打开你下载的或你跟着第二部分完成的工程文件，并运行一下它，确认一下是否所有的 UI 都能正确显示，打开偏好设置窗口看看它是否能正常显示。

![](http://upload-images.jianshu.io/upload_images/5477931-08b68c653acde683.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 沙盒机制

在你开始编写代码之前，请花一些时间来了解一下 macOS 的沙盒机制。如果你是一个 iOS 开发者，你已经了解了这个概念，如果你不曾了解过，继续往下阅读。

一个沙盒化了的 app 拥有自己独立的存储空间，沙盒会禁止你的 app 访问另一个 app 创建的文件以及其他的许可和限制。对于 iOS app，使用沙盒是必须的，而对于 macOS app，这只是一个可选项；但如果你希望通过 Mac App Store 进行分发和销售，你的 app 必须沙盒化，由于沙盒带来的诸多限制，你的 app 可能会出现一些问题。

要为你的 app 启用沙盒，在 **Project Navigator（项目导航器）**中选择项目文件，也就是文件列表里最顶上的蓝色图标。在 **Targets** 列表中选择 **EggTimer**(其实 Targets 列表里也只有一个项目可以选择)，然后在上方的标签中点击 **Capabilities（功能）**标签，点击 **App Sandbox（应用沙盒）**那一栏的开关，这个视图将会展开并显示你的 app 可以申请的许多权限。这个例子中的 app 不需要任何特殊的权限，因此它们都不需要打开。

![](http://upload-images.jianshu.io/upload_images/5477931-a963c427566b4d64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 管理你的文件

看一眼你的 **Project Navigator（项目导航器）**，所有的文件都堆在一起，缺乏组织，这个 app 不会有很多文件，但把文件整理的井井有条始终都会是个好习惯，也能帮助我们更快速地定位到你需要的文件，这一点对于大型项目尤其有用。

![](http://upload-images.jianshu.io/upload_images/5477931-31a03fd62646fa8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

按住 Shift 的同时分别点击两个 View Controller 文件，把他们同时选中，右键点击并选择 **New Group from selection（用所选项目创建新的分组）**，给新建的分组起名为 **View Controllers**。

这个项目将会包含一些 Model 文件，所以右键点击 **EggTimer** 分组，选择 **New Group*（新建分组）*，把这个分组命名为 **Model**。

最后，选中 **Info.plist** 和 **EggTimer.entitlements**，把它们扔掉一个叫 **Supporting Files** 的文件夹里。

拖动分组和文件调整他们的顺序，直到你的项目看起来像这样：

![](http://upload-images.jianshu.io/upload_images/5477931-75542da7b8b457c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# MVC

这个 app 将会应用 MVC 模式：Model View Controller（模型 - 视图 - 控制器）。

> 译者注：请参见 [MVC 设计模式的维基百科词条](https://zh.wikipedia.org/wiki/MVC)，以及[这篇简书文章](http://www.jianshu.com/p/add73330d106)。
> 以及下文会经常出现的名词，下文就不再翻译啦～
> **Model**：模型
> **View**：视图
> **Controller**：控制器
> **Delegate** and **Protocol**：代理与协议

我们要给 app 创建的第一个 Model 对象名叫 `EggTimer`。这个类将会拥有一些关于计时器的开始时间、倒计时的时长和以及过去的时间的属性。还有一个叫做 `Timer` 的对象，每过一秒它都会被激活，并更新自己的状态，并用自己的方法来开始、暂停、恢复或把 `EggTimer` 归零。

`EggTimer` Model 类还会保存数据并执行动作，但它不能用来显示数据。Controller（在这个项目中就是 `ViewController`）则能与 `EggTimer`（也就是 Model）通信，它拥有一个 `View` 并用它来显示数据。

为了能和 `ViewController` 通信，`EggTimer` 使用一个代理协议（Delegate Protocol），每当某些数据发生改变时，`EggTimer` 向它的 `delegate` 发送一条消息，`ViewController` 则让自己去担任 `EggTimer` 的这个所谓的 `delegate`，所以它能接收到这条消息，并把新的数据显示在界面上。

# 编写 EggTimer 类

在**项目导航器**中选中 **Model** 分组，并点击 Xcode 菜单栏上的 **File** → **New** → **File…**，选择 **macOS** → **Swift File**，并点击 **Next**，给这个文件起名为 **EggTimer.swift** 并点击 **Create** 来创建它。

在这个文件中加入以下代码：

```swift
class EggTimer { 
	var timer: Timer? = nil 
	var startTime: Date? 
	var duration: TimeInterval = 360 // 默认的计时时间是 6 分钟
	var elapsedTime: TimeInterval = 0 
}
```

这样 `EggTimer` 类和它的属性们就设置好了。`TimeInterval` 其实就是 `Double` 类型，但一般我们在表示秒数时都会使用它而不是 Double。

第二件事是在类中添加两个计算属性（Computed Properties），这两个属性是用来决定 `EggTimer` 属性的捷径。将以下代码写在刚刚添加的属性之后：

```swift
var isStopped: Bool {
	return timer == nil && elapsedTime == 0 
}

var isPaused: Bool { 
	return timer == nil && elapsedTime > 0 
}
```

在 **EggTimer.swift** 文件 `EggTimer` 类以外的地方添加代理协议的定义 —— 我更喜欢把代理协议写在文件顶部 import 部分的后边。

```swift
protocol EggTimerProtocol { 
	func timeRemainingOnTimer(_ timer: EggTimer, timeRemaining: TimeInterval) 
	func timerHasFinished(_ timer: EggTimer) 
}
```

你可以理解为：这个协议制定了一份合同，任何宣布遵守 `EggTimerProtocol` 协议（也就是签订了这份合同）的对象都需要实现这两个方法。

现在你定义了一个协议，`EggTimer` 可以通过定义一个 `delegate`（代理）属性来履行这份协议，这个属性的类型可以是任何类型（Any）。`EggTimer` 并不知道也不关心代理的类型是什么，因为很明显既然这个代理源自 `EggTimerProtocol` 协议，它拥有这两个方法。

将这些代码属性添加到 `EggTimer` 类：

```swift
var delegate: EggTimerProtocol?
```

让 `EggTimer` 的 timer 对象开始运行会导致一个方法每秒钟被调用一次，继续添加以下代码来定义这个方法，`dynamic` 关键字是让 `Timer` 能发现它的关键。

```swift
dynamic func timerAction() { 
	// 1
	guard let startTime = startTime else { 
	return 
} 

	// 2 
	elapsedTime = -startTime.timeIntervalSinceNow 

	// 3
	let secondsRemaining = (duration - elapsedTime).rounded() 

	// 4
	if secondsRemaining <= 0 { 
		resetTimer() 
			delegate?.timerHasFinished(self) 
	} else { 
		delegate?.timeRemainingOnTimer(self, timeRemaining: secondsRemaining) 
	}
}
```

…所以这些代码到底是在做些什么？

1. `startTime` 是个可选的 `Date`，当它是 `nil` 时，timer 将无法运行，所以这时什么都不会发生；
2. 重新计算 `elapsedTime` 属性，`startTime` 比当前的时间还要早，所以 **timeIntervalSinceNow** 会产生一个负值，这个负值会使得 `elapsedTime` 成为一个正值；
3. 计算 timer 的剩余时间，并进行取整；
4. 如果 timer 已经结束，就把它重设，并告知 `delegate` 计时结束了；否则，告诉 `delegate` 计时器还剩多少秒。另外，由于 `delegate` 是一个可选值，所以需要用 `?` 来进行解包，也就是说，如果 `delegate` 还没有被赋值，除了那些方法不会被调用，没有别的坏事会发生。

你会看到 Xcode 提示我们出现了一些错误，不过当我们完成了 `EggTimer` 类的代码之后，它们就会消失了，这是因为我们还没有添加用于开始计时、暂停计时、恢复计时和重启计时器的方法。

```swift
// 1 
func startTimer() { 
	startTime = Date() 
	elapsedTime = 0 

	timer = Timer.scheduledTimer(timeInterval: 1,
								 target: self, selector: #selector(timerAction), 
								 userInfo: nil,
								 repeats: true) 
	timerAction() 
} 

// 2 
func resumeTimer() {
	startTime = Date(timeIntervalSinceNow: -elapsedTime) 
	timer = Timer.scheduledTimer(timeInterval: 1, 
								 target: self, 
								 selector: #selector(timerAction), 
								 userInfo: nil, 
								 repeats: true) 
	timerAction() 
} 

// 3 
func stopTimer() { 
	// really just pauses the timer 
	timer?.invalidate() 
	timer = nil 
	timerAction() 
} 

// 4 
func resetTimer() { 
	// 停止计时器 & 重设所有属性
	timer?.invalidate() 
	timer = nil 
	startTime = nil 
	duration = 360 
	elapsedTime = 0 
	timerAction() 
}
```

这些代码是做什么的？

1. 通过调用 `Date()` 方法 `startTimer` 设置开始时间为当前时间，然后它会设置一个一直重复运行的 `Timer`；
2. `resumeTimer` 是计时器已经暂停并需要继续时会被调用的方法，它还会根据已经过去的时间重新设置开始时间；
3. `stopTimer` 会停止重复运行的 timer；
4. `resetTimer` 会停止 timer，并把相关属性恢复原始设置。

以上的这些方法都会调用 `timerAction`，所以一旦它们被调用，界面上显示的内容都会被更新。

# ViewController

现在 `EggTimer` 对象已经业已正常运转了，我们该回到 **ViewController.swift** 中让数据的变化能及时反映到界面上了。

`ViewController` 已经拥有了 `@IBOutlet` 属性，但现在你需要让它拥有一个类型为 `EggTimer` 的属性：

```swift
var eggTimer = EggTimer()
```

将 `viewDidLoad` 方法中的注释行替换成这一行：

```swift
eggTimer.delegate = self
```

写完上面的代码以后会出现一个错误，因为 `ViewController` 还没有遵从 `EggTimerProtocol` 协议。当我们要让一个类遵从某个协议时，如果我们单独创建一个 Extension（扩展）来盛放协议需要的方法，你的代码将会看起来整洁许多。在 `ViewController` 类以外的地方输入以下代码：

```swift
extension ViewController: EggTimerProtocol {

	func timeRemainingOnTimer(_ timer: EggTimer, timeRemaining: TimeInterval) {
		updateDisplay(for: timeRemaining)
	}

	func timerHasFinished(_ timer: EggTimer) {
		updateDisplay(for: 0)
	}
}
```

因此我们还需要为 `ViewController` 添加另一个 Extension，用来盛放关于屏幕显示的方法。

```swift
extension ViewController {

	// MARK: - 显示
	func updateDisplay(for timeRemaining: TimeInterval) {
		timeLeftField.stringValue = textToDisplay(for: timeRemaining)
		eggImageView.image = imageToDisplay(for: timeRemaining)
	}

	private func textToDisplay(for timeRemaining: TimeInterval) -> String {
	if timeRemaining == 0 {
		return "Done!"
	}

	let minutesRemaining = floor(timeRemaining / 60)
	let secondsRemaining = timeRemaining - (minutesRemaining * 60)

	let secondsDisplay = String(format: "%02d", Int(secondsRemaining))
	let timeRemainingDisplay = "\(Int(minutesRemaining)):\(secondsDisplay)"

	return timeRemainingDisplay
}

private func imageToDisplay(for timeRemaining: TimeInterval) -> NSImage? {
	let percentageComplete = 100 - (timeRemaining / 360 * 100)

	if eggTimer.isStopped {
		let stoppedImageName = (timeRemaining == 0) ? "100" : "stopped"
		return NSImage(named: stoppedImageName)
	}

	let imageName: String
	switch percentageComplete {
		case 0 ..< 25:
			imageName = "0"
		case 25 ..< 50:
			imageName = "25"
		case 50 ..< 75:
			imageName = "50"
		case 75 ..< 100:
			imageName = "75"
		default:
			imageName = "100"
		}

		return NSImage(named: imageName)
	}

}
```

`updateDisplay` 使用一个 Private 方法来根据剩余的时间来获取文本和图像，并将它们显示在界面上的 Text Field 和 Image View 中。

`textToDisplay` 把剩余的时间格式化成「分：秒」的格式。`imageToDisplay` 计算出鸡蛋有多熟的百分比，然后选择合适的图片来显示在界面上。

所以 `ViewController` 用一个 `EggTimer` 对象的方法来接收 `EggTimer` 传来的数据并显示在屏幕上，但是界面上的按钮还没有任何实质性的代码。在第二部分中，你已经为按钮设置了 `@IBAction`。

这里是这些 IBAction 的方法，你可以用它们来替代之前的 IBAction。

```swift
@IBAction func startButtonClicked(_ sender: Any) {
	if eggTimer.isPaused {
		eggTimer.resumeTimer()
	} else {
		eggTimer.duration = 360
		eggTimer.startTimer()
	}
}

@IBAction func stopButtonClicked(_ sender: Any) {
	eggTimer.stopTimer()
}

@IBAction func resetButtonClicked(_ sender: Any) {
	eggTimer.resetTimer()
	updateDisplay(for: 360)
}
```

这里的三个 IBAction 将会调用你之前添加的 `EggTimer` 方法。

现在编译并运行你的 app，并点击 **Start** 按钮。你还可以用 **Timer** 菜单来控制这个 app，试着去用键盘快捷键来操作你的 app。

现在我们还需要完善一些功能：Stop 和 Reset 按钮始终是被禁用的，而且你只可以定 6 分钟的时。

如果你有足够的耐心，你将会看到鸡蛋的颜色随着时间渐渐改变，并在完成时显示一个「DONE！」。

![](http://upload-images.jianshu.io/upload_images/5477931-5c5c7e7bcdaab0df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 按钮和菜单

界面上的按钮以及菜单里的菜单项应该随着 timer 的状态自动启用或禁用。

把这个方法添加到 `ViewController` 中盛放用于显示相关方法的 Extension 扩展中：

```swift
func configureButtonsAndMenus() {
	let enableStart: Bool
	let enableStop:  Bool
	let enableReset: Bool

	if eggTimer.isStopped {
		enableStart = true
		enableStop  = false
		enableReset = false
	} else if eggTimer.isPaused {
		enableStart = true
		enableStop  = false
		enableReset = true
	} else {
		enableStart = false
		enableStop  = true
		enableReset = false
	}

	startButton.isEnabled = enableStart
	stopButton.isEnabled  = enableStop
	resetButton.isEnabled = enableReset

	if let appDel = NSApplication.shared().delegate as? AppDelegate {
		appDel.enableMenus(start: enableStart, stop: enableStop, reset: enableReset)
	}
}
```

这个方法使用 `EggTimer` 的状态（还记得你添加到 `EggTimer` 里的计算属性吗）来计算出哪个按钮应该启用。

在第二部分中，你创立了一个 Timer menu item 作为 `AppDelegate` 的属性，所以我们应该在 `AppDelegate` 中来编辑这些代码。

切换到 **AppDelegate.swift**，在其中添加这个方法：

```swift
func enableMenus(start: Bool, stop: Bool, reset: Bool) {
	startTimerMenuItem.isEnabled = start
	stopTimerMenuItem.isEnabled  = stop
	resetTimerMenuItem.isEnabled = reset
}
```

为了让你的你的 app 能在初次启动时自动配置按钮的启用状态，在 `applicationDidFinishLaunching` 方法中添加这些代码：

```swift
enableMenus(start: true, stop: false, reset: false)
```

每当用户按下了任何一个按钮或菜单项的时候，`EggTimer` 的状态会发生改变，按钮或菜单项的状态也需要随之更新。返回到 *ViewController.swift* 中并把这一行添加到三个按钮的 IBAction 方法中：

```swift
configureButtonsAndMenus()
```

再次编译并运行你的 app，你可以看到按钮们如预期地启用和禁用了。点击菜单里的菜单项试试，它们应该拥有和按钮一样的功能。

![](http://upload-images.jianshu.io/upload_images/5477931-cad654f61e1616ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 偏好设置窗口

这个 app 还有一个很重要的问题：如果你希望煮鸡蛋的时间不是 6 分钟呢？

在第二部分中，你已经设计好了一个偏好设置窗口来允许用户来选择需要的倒计时时间，这个窗口是由 `PrefsViewController` 控制的，但它还需要一个 Model 对象来处理和查询数据。

用户的设置可以通过一个叫 `UserDefaults` 的东西来存储，它会在你 app 的沙盒容器中的 Preferences 文件夹中用键值对来存储零碎的小数据。

在 **Project Navigator（项目导航器）** 中，右键点击 **Model** 分组，并选择 Xcode 菜单上的 **New File…**，选择 **macOS** → **Swift File**，然后点击 **Next**，把文件起名为 **Preferences.swift** 并点击 **Create**。把这些代码添加到 **Preferences.swift** 文件中：

```swift
struct Preferences {

	// 1
	var selectedTime: TimeInterval {
	get {
		// 2
		let savedTime = UserDefaults.standard.double(forKey: "selectedTime")
			if savedTime > 0 {
				return savedTime
			}
			// 3
			return 360
		}
		set {
			// 4
			UserDefaults.standard.set(newValue, forKey: "selectedTime")
		}
	}

}
```

所以这些代码又干了些啥？

1. 定义了一个名叫 `selectedTime` 的 `TimeInterval` 计算属性；
2. 当别的代码请求访问这个变量的值的时候时，`UserDefaults` 的单例将会去查找键「selectedTime」对应的 `Double` 值；如果这个值从没被定义过，`UserDefaults` 将会返回 0；但如果存在这个值，且它大于 0，就将这个值返回，并设置为 `selectedTime`；
3. 如果 `selectedTime` 还没有被定义过，就使用默认值 360（6 分钟）；
4. 只要 `selectedTime` 的值发生了改变，把新的值用键「selectedTime」存入 `UserDefaults`。

通过使用 getter 和 setter，`UserDefaults` 的数据存储将能够自动进行。

现在切换回 **PrefsViewController.swift**，我们需要把用户修改的设置内容在界面上显示出来。

第一步，在 IBOutlet 之下添加这些代码：

```swift
var prefs = Preferences()
```

这一步中你创建了一个 `Preferences` 的实例，所以你现在可以自由访问 `selectedTime` 计算变量了。

接下来，添加这些方法：

```swift
func showExistingPrefs() {
	// 1
	let selectedTimeInMinutes = Int(prefs.selectedTime) / 60

	// 2
	presetsPopup.selectItem(withTitle: "Custom")
	customSlider.isEnabled = true

	// 3
	for item in presetsPopup.itemArray {
		if item.tag == selectedTimeInMinutes {
			presetsPopup.select(item)
			customSlider.isEnabled = false
			break
		}
	}

	// 4
	customSlider.integerValue = selectedTimeInMinutes
	showSliderValueAsText()
}

// 5
func showSliderValueAsText() {
	let newTimerDuration = customSlider.integerValue
	let minutesDescription = (newTimerDuration == 1) ? "minute" : "minutes"
	customTextField.stringValue = "\(newTimerDuration) \(minutesDescription)"
}
```

好像是很大一坨代码🙄️…所以我们一点一点来看：

1. 访问 `prefs` 对象的 `selectedTime` 属性，并把它转化成整数的分钟数；
2. 把默认的计时时间设置为「Custom」，以防止没有找到人寰预设的数据；
3. 遍历 `presetsPopup` 里的菜单项并检查他们的 tag，还记得在第二部分中你把每个项目的 tag 都设置成了各自选项的分钟数了吗？如果找到了用户选择的菜单项，就把这个菜单项启用，并跳出这个循环；
4. 设置滑动条的数值，并调用 `showSliderValueAsText` 方法；
5. `showSliderValueAsText` 把数字加上「minute」或「minutes」并将它显示在界面上的 Text Field 中。

现在，把这行代码添加到 `viewDidLoad` 中：

```swift
showExistingPrefs()
```

在 View 加载的时候，会调用这个方法，把用户的设置加载到界面上，在 MVC 模式中，`Preferences` Model 完全不知道它伫立的数据会怎样被显示出来 —— 界面显示是 `PrefsViewController` 的事儿。

所以，尽管现在你的 app 已经可以显示用户设置的时间了，然而偏好设置里的下拉框还是不能工作，你需要为它编写一个方法来让它能存储新的的设置，并告诉所有相关对象数据发生了改变。

在 `EggTimer` 对象中，你使用了 delegate 模式来把数据传递到需要它的地方，这一次，你需要通过发送一个 `Notification`（通知）来告诉大家数据改变了（其实用 delegate 还是可以的，这里只是为了演示 Notification 的用法）。任何对象在表明自己对这个通知感兴趣之后，都可以接收到这个通知，并在接收时采取行动。

在 `PrefsViewController` 中添加以下方法：

```swift
func saveNewPrefs() {
	prefs.selectedTime = customSlider.doubleValue * 60
	NotificationCenter.default.post(name: Notification.Name(rawValue: "PrefsChanged"),
                                object: nil)
}
```

这个方法将会获取 customSlider 滑动条的数值，并转化成分钟数，赋值予 `selectedTime`，因为我们之前编写的 setter，它会自动使用 `UserDefaults` 来存储新的数据。然后 `NotificationCenter`（通知中心）会将一个名叫「PrefsChanged」通知发送出去。

接下来，我们来让 `ViewController` 能够接收到这个 `Notification`，并采取行动：

在 `PrefsViewController` 中要编写的最后一部分代码是为第二部分中你添加的 `@IBAction` 们添加真正的代码：

```swift
// 1
@IBAction func popupValueChanged(_ sender: NSPopUpButton) {
	if sender.selectedItem?.title == "Custom" {
		customSlider.isEnabled = true
		return
	}

	let newTimerDuration = sender.selectedTag()
	customSlider.integerValue = newTimerDuration
	showSliderValueAsText()
	customSlider.isEnabled = false
}

// 2
@IBAction func sliderValueChanged(_ sender: NSSlider) {
	showSliderValueAsText()
}

// 3
@IBAction func cancelButtonClicked(_ sender: Any) {
	view.window?.close()
}

// 4
@IBAction func okButtonClicked(_ sender: Any) {
	saveNewPrefs()
	view.window?.close()
}
```

1. 当用户在下拉框中选择了一个新的菜单项，这段代码会检测这个项是不是 Custom：
	- 如果是的，就启用滑动条，并直接终止这个方法；
	- 如果不是，就通过这个项的 tag 来获取用户选择的计时时间；
2. 每当滑动条的数据更新时，更新界面上的文本；
3. 点击 Cancel 按钮会把窗口关闭，且不会存储数据；
4. 点击 OK 按钮会先调用 `saveNewPrefs`，然后关闭这个窗口。

编译并运行你的 app，前往 **Preferences**，试着在下拉框中选择不同的选项，观察一下滑动条和文本有没有根据你的选择而正确显示。选择 Custom 选项，然后自己选择一个时间，点击 **OK**，然后再次前往 **Preferences**，看看你刚刚选择的时间是不是还能正常显示。

现在试着退出你的 app 并重新打开它，返回 **Preferences**，看看你的 app 是否保存了你的设置。

![](http://upload-images.jianshu.io/upload_images/5477931-cfea9ce49a01ee81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 让用户的设置生效

现在偏好设置窗口看起来还不错了 —— 它可以存储并读取用户的设置，但当你回到主窗口，你看到的时间会还是 6 分钟！ ☹️

所以你需要编辑 **ViewController.swift**，让它能使用存储了的数据，并侦听关于数据变化了的通知，从而及时更新或重设 Timer。

把这个 Extension 添加到 **ViewController.swift** 中类定义以外的部分 —— 这样一来我们的代码会被分成若干个承担不同职能的部分，看起来会更整洁。

```swift
extension ViewController {

	// MARK: - 设置
	func setupPrefs() {
		updateDisplay(for: prefs.selectedTime)

		let notificationName = Notification.Name(rawValue: "PrefsChanged")
		NotificationCenter.default.addObserver(forName: notificationName,
											   object: nil, queue: nil) {
			(notification) in
			self.updateFromPrefs()
		}
	}

	func updateFromPrefs() {
		self.eggTimer.duration = self.prefs.selectedTime
		self.resetButtonClicked(self)
	}

}
```

这些代码会报错，因为 `ViewController` 内部还没有一个叫做 `prefs` 的对象。在 `ViewController` 类的定义中（也就是你定义 `eggTimer` 的地方），添加这行代码：

```swift
var prefs = Preferences()
```

现在 `PrefsViewController` 和 `ViewController` 内部都有了一个 prefs 属性 —— 这是个问题吗？不！原因如下：

1. `Preferences` 是一个 struct（结构体），所以它是一个数据型的对象而非一个关系型的对象。每一个 View Controller 都可以拥有一份它的副本；
2. `Preferences` 结构体是使用了 `UserDefaults` 的单例，所以这俩副本其实是在调用同一个 `UserDefaults`，因此拿到的数据也是完全一样的。

在 ViewController 最后的 `viewDidLoad` 方法中，添加这一行代码，它会设置好自己和 `Preferences` 的连接：

```swift
setupPrefs()
```

现在还有最后的一系列步骤需要做。之前我们把默认的时间，也就是 360 秒，直接写进了代码里（也就是硬编码，hard-coded），现在因为 `ViewController` 已经可以访问 `Preferences` 了，你需要修改一下这种写法。

在 **ViewController.swift** 中找到「360」（你应该能找到 3 个 360），并把它们修改成 `prefs.selectedTime`。

编译并运行你的 app，如果你之前修改过设置里的计时时间，你选择的时间现在应该能正常显示在界面上了。前往 **Preferences**，选择另一时间，点击 **OK** —— 因为 `ViewController` 接收到了通知，你新选择的时间应该马上就能显示出来了。

![](http://upload-images.jianshu.io/upload_images/5477931-ea3e30e4e2da720e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

启动计时器，然后前往 **Preferences**，在主窗口中，倒计时还在继续，修改一个时间然后点击 **OK**，计时器应用了新的时间，但是也停止并重设了倒计时。我觉得这没什么问题，但是如果能添加一个提示，询问用户是否真的希望停止计时，这样会不会更好呢？

在 ViewController 中负责处理设置的 Extension 中，添加这些代码：

```swift
func checkForResetAfterPrefsChange() {
	if eggTimer.isStopped || eggTimer.isPaused {
		// 1
		updateFromPrefs()
	} else {
		// 2
		let alert = NSAlert()
		alert.messageText = "Reset timer with the new settings?"
		alert.informativeText = "This will stop your current timer!"
		alert.alertStyle = .warning

		// 3
		alert.addButton(withTitle: "Reset")
		alert.addButton(withTitle: "Cancel")

		// 4
		let response = alert.runModal()
		if response == NSAlertFirstButtonReturn {
			self.updateFromPrefs()
		}
	}
}
```

所以这些代码是干啥的？

1. 如果计时器已经停止或暂停了，不做任何操作直接修改时间；
2. 创建一个 `NSAlert`，它是一个用来显示一个对话框的类，并设置它的文字和样子；
3. 添加两个按钮：Reset 和 Cancel，它们将会根据你添加的顺序从右往左显示在对话框中，且右边的将会是默认选项；
4. 把警告以一个模态的窗口显示出来，并等待用户的选择，如果用户点击了第一个按钮（Reset），就重设计时器。

在 `setupPrefs` 方法中，把 `self.updateFromPrefs()` 这一行改成：

```swift
self.checkForResetAfterPrefsChange()
```

编译并运行你的 app，开始计时，前往 **Preferences**，修改一下时间，然后点击 **OK**，你将会看见一个对话框询问你是否要重设时间。

![](http://upload-images.jianshu.io/upload_images/5477931-129ab28735fa7b83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 音效

现在这个 app 中唯一未完成的功能就是音效了。如果没有「叮～～」的一声的话，煮蛋计时器还能叫做煮蛋计时器吗？

在第二部分中，你已经下载了一个包含了所有资产的文件夹，其中的内容绝大多数都是图片，你也已经用过它们了，但是其实这里面还有一个音效文件：**ding.mp3**。如果你找不到这个文件了，你可以单独下载[这个音效文件](https://koenig-media.raywenderlich.com/uploads/2017/02/ding.mp3.zip)。

把 **ding.mp3** 拖动到 **Project Navigator（项目导航器）**中的 **EggTimer** 分组下方 —— 看起来就放在 **Main.storyboard** 下边是一个不错的想法。勾选 **Copy items if needed（如果需要的话把文件拷贝到项目中）**，在 **Add to targets（添加到目标中）** 中勾选 EggTimer，然后点击 **Finish**。

![](http://upload-images.jianshu.io/upload_images/5477931-83a5759cff376d34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

你需要一个叫 `AVFoundation` 的库来播放声音。当代理告诉 `ViewController` 计时器结束了的时候，`ViewController` 就会负责播放这个音效，所以我们切换到 **ViewController.swift** 中，在最顶部你会看到这个文件引用了 `Cocoa` 库（`import Cocoa`）。

在那一行引用的下方，添加：

```swift
import AVFoundation
```

`ViewController` 需用一个 `AVAudioPlayer` 来播放声音，所以我们为它添加一个属性：

```swift
var soundPlayer: AVAudioPlayer?
```

我们应该为 `ViewController` 新建一个单独的 Extension 来处理和声音相关的方法，所以在 **ViewController.swift** 类定义以外的地方添加：

```swift
extension ViewController {
	
	// MARK: - 声音
		
	func prepareSound() {
		guard let audioFileUrl = Bundle.main.url(forResource: "ding",
											 withExtension: "mp3") else {
			return
		}
		do {
			soundPlayer = try AVAudioPlayer(contentsOf: audioFileUrl)
			soundPlayer?.prepareToPlay()
		} catch {
			print("Sound player not available: \(error)")
		}
	}
	
	func playSound() {
		soundPlayer?.play()
	}

}
```

`prepareSound` 方法会负责处理绝大多数的事情 —— 它会先检查 ding.mp3 是否存在于 app 的包中，如果这个文件存在，它就会试图去用这个文件的 URL 来实例化一个 `AVAudioPlayer`，并准备好它以备播放。这将会预先加载这个音频文件，所以一旦需要，就可以立即播放。

如果 `soundPlayer` 存在，`playSound` 会调用它的 `play()` 方法；但如果 `prepareSound` 运行失败了，`soundPlayer` 将会为空（nil），因此它什么也不会做。

声音文件只在 Start 按钮被点击时需要被准备，所以把这行代码插入到 `startButtonClicked` 方法的最后：

```swift
prepareSound()
```

在 **EggTimerProtocol** Extension 的 **timerHasFinished** 方法中，追加这行代码：

```swift
playSound()
```

编译并运行之，选择一个短一点的时间并开始计时，一声清脆的「叮🔔」会在计时结束的时候响起。

![](http://upload-images.jianshu.io/upload_images/5477931-34ba46713c905890.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 现在该做些什么？

你可以[下载这个项目的源代码](https://koenig-media.raywenderlich.com/uploads/2017/02/EggTimer-Final2.zip)。

在这个 macOS 开发教程中，你已经掌握了开发 macOS app 的基本技能，但真正要学习的还有很多！

Apple 编写了许多很棒的[文档](https://developer.apple.com/library/content/navigation/#section=Platforms&topic=macOS)，他们覆盖了 macOS 开发的方方面面。

我同时强烈建议你去看看我们（原作者）的网站 [raywenderlich.com](https://www.raywenderlich.com/category/macos) 上的其他 macOS 教程。

如果你还有任何问题，欢迎在[原文下方](https://www.raywenderlich.com/151748/macos-development-beginners-part-3)参与讨论！
