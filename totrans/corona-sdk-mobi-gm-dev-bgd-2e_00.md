# 前言

本书旨在介绍你在 iOS 和 Android 平台使用 Corona SDK 的基本标准。通过按部就班地构建三款独特的游戏，你将增强学习体验。除了开发游戏，你还将学习社交网络集成、应用内购买，以及将应用程序发布到苹果 App Store 和/或谷歌 Play 商店。

# 本书涵盖内容

第一章，*开始使用 Corona SDK*，首先教你如何在 Mac OS X 和 Windows 操作系统上安装 Corona SDK。你将学会如何仅用两行代码创建你的第一个程序。最后，我们将介绍构建和加载应用程序到 iOS 或 Android 设备的过程。

第二章，*Lua 速成与 Corona 框架*，深入探讨用于 Corona SDK 开发的 Lua 编程语言。我们将介绍 Lua 中变量、函数和数据结构的基础知识。本章还将介绍如何在 Corona 框架内实现各种显示对象。

第三章，*制作我们的第一款游戏——打砖块*，讨论了制作你的第一款游戏，打砖块的前半部分。你将学习如何在 Corona 项目中构建游戏文件，并创建将在屏幕上显示的游戏对象。

第四章，*游戏控制*，继续讨论制作你的第一款游戏，打砖块的后半部分。我们将涵盖游戏对象移动以及场景中对象之间的碰撞检测。你还将学习如何创建一个计分系统，该系统将实现游戏的胜利和失败条件。

第五章，*让我们的游戏动起来*，解释了如何使用精灵表来动画化游戏。本章将深入探讨在创建新游戏框架时管理动作和过渡。

第六章，*播放声音和音乐*，提供了如何在应用程序中应用声音效果和音乐的信息。在增强游戏开发感官体验方面，包含某种类型的音频至关重要。你将学习如何通过加载、执行和循环技术，利用 Corona 音频系统融入音频。

第七章，*物理现象——下落物体*，涵盖了如何在 Corona SDK 中使用显示对象实现 Box2D 引擎。你将能够自定义构建物体，并处理下落物体的物理行为。在本章中，我们将应用动态/静态物体的使用，并解释碰撞后处理的目的。

第八章，*操作编排器*，讨论如何使用 Composer API 管理所有游戏场景。我们还将详细介绍菜单设计，例如创建暂停菜单和主菜单。此外，你将学习如何在游戏中保存高分。

第九章，*处理多设备和网络应用*，提供了将你的应用程序与如 Twitter 或 Facebook 等社交网络集成的信息。这将使你的应用程序能够全球范围内触及更多受众。

第十章，*优化、测试和发布你的游戏*，解释了针对 iOS 和 Android 设备的应用提交过程。本章将指导你如何为 Apple App Store 设置分发供应配置文件，并在 iTunes Connect 中管理你的应用信息。Android 开发者将学习如何为发布签署他们的应用程序，以便提交到 Google Play Store。

第十一章，*实现应用内购买*，介绍了如何通过创建可消耗、不可消耗或订阅购买来为你的游戏实现货币化。你将使用 Corona 的商店模块在 Apple App Store 申请应用内购买。我们还将查看在设备上测试购买，以检查是否使用沙盒环境应用了交易。

附录，*弹出式测验答案*，包含了本书所有弹出式测验部分的答案。

# 你需要为本书准备以下物品

在使用 Corona SDK for Mac 开发游戏之前，你需要准备以下物品：

+   如果你正在安装适用于 Mac OS X 的 Corona，请确保你的系统具备以下条件：

    +   Mac OS X 10.9 或更高版本

    +   运行 Lion、Mountain Lion、Mavericks 或 Yosemite 的基于 Intel 的系统

    +   64 位 CPU（Core 2 Duo）

    +   OpenGL 2.0 或更高版本的图形系统

+   你必须注册 Apple Developer Program

+   XCode

+   文本编辑器，如 TextWrangler、BBEdit 或 TextMate

在使用 Corona SDK for Windows 开发游戏之前，你需要准备以下物品：

+   如果你使用的是 Microsoft Windows，请确保你的系统具备以下条件：

    +   Windows 8、Windows 7、Vista 或 XP（Service Pack 2）操作系统

    +   1 GHz 处理器（推荐）

    +   80 MB 磁盘空间（最低要求）

    +   1 GB 内存（最低要求）

    +   OpenGL 2.1 或更高版本的图形系统（大多数现代 Windows 系统中可用）

    +   Java 开发工具包（JDK）的 32 位（x86）版本

    +   使用 Corona 在 Mac 或 Windows 上创建 Android 设备构建时，不需要 Android SDK

+   Java 6 SDK

+   文本编辑器，如 Notepad ++

如果你想为 Android 设备提交和发布应用，你必须注册为 Google Play Developer。

游戏教程需要使用本书提供的资源文件，也可以从 Packt Publishing 网站下载。

最后，你需要 Corona SDK 的最新稳定版本。这适用于所有订阅级别。

# 本书适合的对象

这本书适合任何想要尝试为 Android 和 iOS 创建商业上成功的游戏的人。你不需要游戏开发或编程经验。

# 部分

在这本书中，你会发现有几个经常出现的标题（动手时间、刚刚发生了什么？、小测验和动手英雄）。

为了清楚地说明如何完成一个过程或任务，我们使用以下部分：

# 动手时间——标题

1.  操作 1

1.  操作 2

1.  操作 3

指令通常需要一些额外的解释以确保其意义明确，因此它们后面会跟着这些部分：

## *刚刚发生了什么？*

本节解释了你刚刚完成的工作或指令的运作方式。

你在书中还会发现一些其他的学习辅助工具，例如：

## 小测验——标题

这些是简短的选择题，旨在帮助你测试自己的理解。

## 动手英雄——标题

这些是实践挑战，为你提供实验所学知识的想法。

# 约定

你还会发现文本中有多种样式，用于区分不同类型的信息。以下是一些样式的例子及其含义的解释。

文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 处理程序如下显示："我们可以通过使用`include`指令包含其他上下文。"

代码块如下设置：

```kt
textObject = display.newText( "Hello World!", 160, 80, native.systemFont, 36 )
textObject: setFillColor ( 1, 1, 1 )
```

当我们希望引起你注意代码块中的特定部分时，相关的行或项目会以粗体设置：

```kt
    local buyLevel2 = function ( product ) 
      print ("Congrats! Purchasing " ..product)

     -- Purchase the item
      if store.canMakePurchases then 
        store.purchase( {validProducts[1]} ) 
      else
        native.showAlert("Store purchases are not available, please try again later",  { "OK" } ) – Will occur only due to phone setting/account restrictions
      end 
    end 
    -- Enter your product ID here
 -- Replace Product ID with a valid one from iTunes Connect
 buyLevel2("com.companyname.appname.NonConsumable")

```

任何命令行输入或输出都如下书写：

```kt
keytool -genkey -v -keystore my-release-key.keystore -alias aliasname -keyalg RSA -validity 999999

```

**新** **术语**和**重要** **词汇**以粗体显示。你在屏幕上看到的词，例如菜单或对话框中的，会在文本中这样出现："点击**立即注册**按钮，并按照苹果的指示完成流程。"

### 注意

警告或重要提示会以这样的框显示。

### 提示

技巧和窍门会像这样出现。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢或不喜欢什么。读者的反馈对我们很重要，因为它帮助我们开发出你真正能从中获得最大收益的标题。

要给我们发送一般反馈，只需发送电子邮件到`<feedback@packtpub.com>`，并在邮件的主题中提及书籍的标题。

如果你有一个擅长的主题并且有兴趣撰写或参与书籍编写，请查看我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然你现在拥有了 Packt Publishing 的一本书，我们有许多方法帮助你最大限度地利用你的购买。

## 下载示例代码

您可以从您的账户[`www.packtpub.com`](http://www.packtpub.com)下载所有您购买过的 Packt Publishing 书籍的示例代码文件。如果您在别处购买了这本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)注册，我们会直接将文件通过电子邮件发送给您。

## 下载本书的彩色图像

我们还为您提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。彩色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/9343OT_ColoredImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/9343OT_ColoredImages.pdf)下载此文件。

## 勘误

尽管我们已经尽力确保内容的准确性，但错误仍然会发生。如果您在我们的书中发现了一个错误——可能是文本或代码中的错误——如果您能报告给我们，我们将不胜感激。这样做，您可以避免其他读者感到沮丧，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**Errata Submission Form**链接，并输入您的勘误详情来报告。一旦您的勘误被验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题下的现有勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书名。所需信息将在**Errata**部分出现。

## 盗版

在互联网上，盗版受版权保护的材料是所有媒体都面临的持续问题。在 Packt，我们非常重视保护我们的版权和许可。如果您在互联网上以任何形式遇到我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请在`<copyright@packtpub.com>`联系我们，并提供疑似盗版材料的链接。

我们感谢您保护我们的作者以及我们为您带来有价值内容的能力。

## 问题

如果您对本书的任何方面有问题，可以联系我们`<questions@packtpub.com>`，我们将尽力解决问题。
