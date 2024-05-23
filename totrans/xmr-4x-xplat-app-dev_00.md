# 前言

Xamarin 为 C#开发 iOS 和 Android 应用程序打造了优秀的产品：Xamarin Studio，Visual Studio 的插件，Xamarin.iOS 和 Xamarin.Android。Xamarin 让你直接访问每个平台的本地 API，并具有共享 C#代码的灵活性。使用 Xamarin 和 C#，相比于 Java 或 Objective-C，你可以获得更高的生产效率，并且与 HTML 或 JavaScript 解决方案相比，仍然保持出色的性能。

在本书中，我们将开发一个现实世界的示例应用程序，以展示你可以使用 Xamarin 技术做什么，并在 iOS 和 Android 的核心平台概念上进行构建。我们还将涵盖高级主题，如推送通知、获取联系人、使用相机和 GPS 定位。随着 Xamarin 3 的推出，引入了一个名为 Xamarin.Forms 的新框架。我们将介绍 Xamarin.Forms 的基础知识以及如何将其应用于跨平台开发。最后，我们将介绍提交应用程序到 Apple App Store 和 Google Play 需要做些什么。

# 本书涵盖的内容

第一章，*Xamarin 设置*，是关于安装适合进行跨平台开发的 Xamarin 软件和本地 SDK 的指南。指导 Windows 用户如何在本地网络中连接 Mac，以便在 Visual Studio 中进行 iOS 开发。

第二章， *平台你好！*，带你一步步在 iOS 和 Android 上创建一个简单的计算器应用程序，同时也涵盖了每个平台的一些基本概念。

第三章，*iOS 和 Android 之间的代码共享*，介绍了可以使用 Xamarin 的代码共享技术和项目设置策略。

第四章， *XamSnap - 一个跨平台应用*，介绍了一个示例应用程序，我们将在整本书中构建它。在本章中，我们将为该应用程序编写所有共享代码，并完成单元测试。

第五章， *iOS 的 XamSnap*，展示了如何为 XamSnap 实现 iOS 用户界面，并涵盖了各种 iOS 开发概念。

第六章， *安卓的 XamSnap*，展示了如何实现 XamSnap 的 Android 版本，并介绍了 Android 特定的开发概念。

第七章， *在设备上部署和测试*，带你经历将第一个应用程序部署到设备的痛苦过程。我们还讨论为什么在真实设备上测试应用程序很重要。

第八章，*联系人、相机和位置*，介绍了库 Xamarin.Mobile，作为跨平台方式访问用户的联系人、相机和 GPS 位置，并将这些功能添加到我们的 XamSnap 应用程序中。

第九章，*带有推送通知的 Web 服务*，展示了如何使用 Windows Azure 实现 XamSnap 的真实后端 Web 服务，利用 Azure Functions 和 Azure Notification Hubs。

第十章，*第三方库*，涵盖了使用 Xamarin 的各种第三方库选项，以及如何甚至利用原生 Java 和 Objective-C 库。

第十一章，*Xamarin.Forms*，帮助我们探索 Xamarin 的最新框架 Xamarin.Forms，以及如何利用它构建跨平台应用程序。

第十二章，*应用商店提交*，将引导我们完成将你的应用提交到苹果 App Store 和 Google Play 的过程。

# 你需要为这本书准备什么

对于这本书，你需要一台运行至少 OS X 10.10 的 Mac 电脑。苹果要求 iOS 应用程序必须在 Mac 上编译，因此 Xamarin 也有同样的要求。你可以使用 Xamarin Studio（最适合 Mac）或 Visual Studio（最适合 Windows）作为 IDE。在 Windows 上的开发人员可以通过连接到本地网络上的 Mac 来在 Visual Studio 上开发 iOS 应用程序。访问[`xamarin.com/download`](https://xamarin.com/download)或[`visualstudio.com/download`](https://visualstudio.com/download)以下载合适的软件。

# 这本书适合谁

这本书适合已经熟悉 C#并希望学习使用 Xamarin 进行移动开发的开发人员。如果你在 ASP.NET、WPF、WinRT、Windows Phone 或 UWP 方面有过工作经验，那么使用这本书来开发原生 iOS 和 Android 应用程序将会非常得心应手。

# 约定

在这本书中，你会发现多种文本样式，用于区分不同类型的信息。以下是一些样式示例及其含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 处理方式如下所示："使用`await`关键字在 C#中运行异步代码。"

一段代码如下设置：

```kt
class ChuckNorris
{
    void DropKick()
    {
        Console.WriteLine("Dropkick!");
    }
}
```

当我们希望引起您对代码块中特定部分的注意时，相关的行或项目会以粗体显示：

```kt
class ChuckNorris
{
    void DropKick()
    {
        Console.WriteLine("Dropkick!");
    }
}
```

任何命令行输入或输出都如下编写：

```kt
# xbuild MyProject.csproj

```

**新术语**和**重要词汇**以粗体显示。你在屏幕上看到的词，例如菜单或对话框中的，文本中会像这样显示："为了下载新模块，我们将转到**文件** | **设置** | **项目名称** | **项目解释器**。"

### 注意

警告或重要提示会以这样的框显示。

### 提示

提示和技巧会像这样显示。

# 读者反馈

我们欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢或不喜欢什么。读者的反馈对我们很重要，因为它帮助我们开发出你真正能从中获得最大收益的标题。要给我们发送一般反馈，只需发送电子邮件到 feedback@packtpub.com，并在邮件的主题中提及书籍的标题。如果你对某个主题有专业知识，并且有兴趣撰写或为书籍做贡献，请查看我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然你现在拥有一本 Packt 的书，我们有很多方法可以帮助你充分利用你的购买。

## 下载示例代码

你可以从你的账户在[`www.packtpub.com`](http://www.packtpub.com)下载本书的示例代码文件。如果你在其他地方购买了这本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)注册，我们会直接将文件通过电子邮件发送给你。

你可以通过以下步骤下载代码文件：

1.  使用你的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标悬停在顶部**支持**标签上。

1.  点击**代码下载 &勘误**。

1.  在**搜索**框中输入书籍名称。

1.  选择你想要下载代码文件的那本书。

1.  从下拉菜单中选择你购买本书的地方。

1.  点击**代码下载**。

文件下载后，请确保你使用最新版本的软件解压或提取文件夹：

+   对于 Windows 系统，使用 WinRAR / 7-Zip。

+   对于 Mac 系统，使用 Zipeg / iZip / UnRarX。

+   对于 Linux 系统，使用 7-Zip / PeaZip。

本书的代码包也托管在 GitHub 上，地址为[`github.com/PacktPublishing/Xamarin 4x-Cross-Platform-Application-Development-Third-Edition`](https://github.com/PacktPublishing/Xamarin%204x-Cross-Platform-Application-Development-Third-Edition)。我们还有其他丰富的书籍和视频代码包，可以在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。请查看！

## 下载本书的色彩图片

我们还为你提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的色彩图片。色彩图片将帮助你更好地理解输出的变化。你可以从[`www.packtpub.com/sites/default/files/downloads/Xamarin4xCrossPlatformApplicationDevelopmentThirdEdition_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/Xamarin4xCrossPlatformApplicationDevelopmentThirdEdition_ColorImages.pdf)下载这个文件。

## 勘误

尽管我们已经竭尽全力确保内容的准确性，但错误仍然在所难免。如果你在我们的书中发现了一个错误——可能是文本或代码中的错误，如果你能向我们报告，我们将不胜感激。这样做可以避免其他读者产生困扰，并帮助我们改进本书后续版本。如果你发现任何勘误信息，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择你的书籍，点击**勘误提交表单**链接，并输入你的勘误详情。一旦你的勘误信息被核实，你的提交将被接受，并且勘误信息将被上传到我们的网站，或者添加到该书标题下的现有勘误列表中。

要查看之前提交的勘误信息，请前往[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将显示在**勘误**部分。

## 盗版

在互联网上对版权材料的盗版是所有媒体持续存在的问题。在 Packt，我们非常重视保护我们的版权和许可。如果你在互联网上以任何形式遇到我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

如果你怀疑有盗版材料，请通过 copyright@packtpub.com 联系我们，并提供相关链接。

我们感谢你帮助保护我们的作者和我们为你提供有价值内容的能力。

## 问题

如果你对本书的任何方面有问题，可以通过 questions@packtpub.com 联系我们，我们将尽力解决问题。
