# 前言

本书是学习高级 Android 应用开发的实用指南。本书帮助掌握 Android 的核心概念，并在实际项目中快速应用知识。在整本书中，创建了一个应用，并在每一章中进化，以便读者可以轻松地跟随并吸收概念。

本书共分为十二章。前三章主要关注应用的设计，解释了设计的基本概念以及在 Android 中使用的编程模式。接下来的章节旨在改进应用，访问服务器端以下载应用中要显示的信息。应用功能完成后，将使用 Material Design 组件和其他第三方库进行改进。

在结束之前，为应用添加了额外服务，如位置服务、分析、崩溃报告和盈利化。最后，导出应用，解释不同的构建类型和证书，并将其上传到 Play 商店，准备进行分发。

# 本书内容涵盖

第一章，*入门*，解释了 Android 6 Marshmallow 的基础知识以及 Material Design 的重要概念。我们将设置开始开发所需的工具，并可选地安装一个比 Android 默认模拟器更快的超快模拟器，这将帮助我们在阅读本书的过程中测试我们的应用。

第二章，*设计我们的应用*，介绍了创建应用的第一步——设计导航——以及不同的导航模式。我们将应用标签页模式与滑动屏幕，解释并使用 Fragments，这是 Android 应用开发的一个关键组件。

第三章，*从云端创建和访问内容*，涵盖了在我们应用中显示来自互联网信息所需的一切。这些信息可以在外部服务器或 API 上。我们将使用 Parse 创建自己的服务器，并使用 Volley 和 OKHttp 进行高级网络请求来访问它，处理信息并使用 Gson 将其转换为可用的对象。

第四章，*并发与软件设计模式*，讨论了 Android 中的并发问题及处理它的不同机制，如 AsyncTask、服务、Loader 等。本章的第二部分讲述了在 Android 中最常见的编程模式。

第五章，*列表和网格*，讨论了列表和网格，从 ListViews 开始。它解释了这一组件如何在 RecyclerView 中演变，并以示例展示了如何创建带有不同类型元素的列表。

第六章，*CardView 和材料设计*，主要从用户界面角度出发，提升应用的设计感，并引入材料设计，讲解并实现如 CardView、Toolbar 和 CoordinatorLayout 等功能。

第七章，*图像处理和内存管理*，主要讨论如何在我们的应用中显示从互联网上下载的图片，使用不同的机制，例如 Volley 或 Picasso。它还涵盖了不同类型的图像，如矢量可绘制图像和 Nine patch。最后，它讨论了内存管理以及如何预防、检测和定位内存泄漏。

第八章，*数据库和加载器*，本质上是解释 Android 中数据库的工作原理，内容提供者是什么，以及如何使用 CursorLoaders 让数据库直接与视图通信。

第九章，*推送通知和分析*，讨论如何使用 Google Cloud Messaging 和 Parse 实现推送通知。章节的第二部分讨论了分析，这对于理解用户如何与我们的应用互动、捕获错误报告以及保持应用无 bug 至关重要。

第十章，*位置服务*，通过在应用中实现一个示例来介绍 MapView，从开发者控制台的初始设置到应用中最终展示位置标记的地图视图。

第十一章，*安卓上的调试和测试*，主要讨论测试。它涵盖了单元测试、集成测试和用户界面测试。还讨论了使用市场上不同的工具和最佳实践，通过自动化测试开发可维护的应用。

第十二章，*营利化、构建过程和发布*，展示了如何使应用盈利，并解释了广告盈利化的关键概念。它展示了如何导出具有不同构建类型的应用，并最终如何在 Google Play 商店上传和推广此应用。

# 阅读本书所需条件

您的系统需要安装以下软件以执行本书中提到的代码：

+   Android Studio 1.0 或更高版本

+   Java 1.7 或更高版本

+   Android 4.0 或更高版本

# 本书的目标读者

如果您是一位有 Gradle 和项目开发经验的 Java 开发者，并希望成为专家，那么这本书适合您。对 Gradle 的基本了解是必需的。

# 约定

在本书中，您会发现多种文本样式，用于区分不同类型的信息。以下是一些样式示例及其含义的解释。

文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名和虚拟 URL 会以如下形式显示："我们可以通过使用`include`指令包含其他上下文。"

代码块如下设置：

```kt
<uses-permission android:name="android.permission.INTERNET" /> <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" /> <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" /> 
```

当我们希望引起你注意代码块中的特定部分时，相关的行或项目会以粗体设置：

```kt
<uses-permission android:name="android.permission.INTERNET" /> <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" /> <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" /> 
```

**新术语**和**重要词汇**会用粗体显示。你在屏幕上看到的词，例如菜单或对话框中的，会在文本中以这样的形式出现："点击**下一步**按钮，你会进入下一个屏幕。"

### 注意

警告或重要注意事项会像这样出现在一个框里。

### 提示

提示和技巧会像这样出现。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们你对这本书的看法——你喜欢或不喜欢的地方。读者的反馈对我们很重要，因为它能帮助我们开发出你真正能从中获益的图书。

要发送给我们一般反馈，只需通过电子邮件`<feedback@packtpub.com>`联系我们，并在邮件的主题中提及书籍的标题。

如果你有一个擅长的主题，并且有兴趣撰写或参与编写一本书，请查看我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然你现在拥有了 Packt 的一本书，我们有一些事情可以帮助你最大限度地利用你的购买。

## 下载示例代码

你可以从你的账户[`www.packtpub.com`](http://www.packtpub.com)下载你所购买的 Packt Publishing 图书的示例代码文件。如果你在其他地方购买了这本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)注册，我们会直接将文件通过电子邮件发送给你。

## 下载本书的彩色图像

我们还为你提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。彩色图像将帮助你更好地理解输出的变化。你可以从[`www.packtpub.com/sites/default/files/downloads/4221OS_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/4221OS_ColorImages.pdf)下载这个文件。

## 错误更正

尽管我们已经尽力确保内容的准确性，但错误仍然会发生。如果你在我们的书中发现了一个错误——可能是文本或代码中的错误——如果你能报告给我们，我们将不胜感激。这样做，你可以避免其他读者的困扰，并帮助我们改进本书的后续版本。如果你发现任何错误更正，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择你的书籍，点击**错误更正提交表单**链接，并输入你的错误更正详情。一旦你的错误更正被验证，你的提交将被接受，错误更正将被上传到我们的网站或添加到该标题错误更正部分下的现有错误更正列表中。

要查看先前提交的勘误信息，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书名。所需信息将显示在**勘误**部分下。

## 盗版

互联网上对版权材料的盗版行为是所有媒体持续面临的问题。在 Packt，我们非常重视保护我们的版权和许可。如果您在互联网上以任何形式遇到我们作品的非法副本，请立即提供其位置地址或网站名称，以便我们可以寻求补救措施。

如果您发现疑似盗版材料，请通过`<copyright@packtpub.com>`联系我们，并提供相关链接。

我们感谢您帮助保护我们的作者以及我们向您提供有价值内容的能力。

## 问题咨询

如果您对这本书的任何方面有疑问，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决问题。
