# 前言

本书是一本实用的、手把手指导开发 Android 应用程序的指南，使用 Android Ice Cream Sandwich（Android 4.0）的新特性，采用逐步讲解和清晰解释的示例代码。你将通过这些示例代码学习 Android 4.0 的新 API。

# 本书涵盖内容

第一章, *为所有人设计的操作栏*, 介绍了操作栏，并展示如何使用和配置操作栏。

第二章, *新布局 - GridLayout*, 介绍了 GridLayout，并展示如何使用和配置 GridLayout。GridLayout 是随 Android Ice Cream Sandwich 推出的一种新布局。这个布局是一个优化过的布局，可以替代 LinearLayout 和 RelativeLayout。

第三章, *社交 API*, 讲解了随 Android Ice Cream Sandwich 推出的社交 API。这个 API 使得集成社交网络变得简单。此外，在 Ice Cream Sandwich 发布后，现在可以使用高分辨率照片作为联系人的照片。这一章通过示例展示了社交 API 的使用。

第四章, *日历 API*, 讲解了与 Android Ice Cream Sandwich 一同推出的日历 API，用于管理日历。事件、参与者、提醒和备忘数据库可以通过这些 API 进行管理。这些 API 使我们能够轻松地将日历与 Android 应用程序集成。这一章展示了如何通过示例使用日历 API。

第五章, *碎片*, 介绍了碎片的基础知识以及如何使用它们。

第六章, *支持不同的屏幕尺寸*, 介绍了设计支持不同屏幕尺寸的用户界面的方法。

第七章, *Android 兼容性包*, 介绍了 Android 兼容性包，并展示如何使用它。Android 兼容性包是为了将新 API 移植到 Android 平台的旧版本。

第八章, *新的连接 API - Android Beam 和 Wi-Fi Direct*, 介绍了 Android Beam，它使用设备的 NFC 硬件和无需使用无线接入点的 Wi-Fi Direct，允许设备之间相互连接。这一章将教我们如何使用 Android Beam 和 Wi-Fi Direct。

*第九章*, *多 APK 支持*, 介绍了多 APK 支持，这是 Google Play（Android 市场）中的一个新选项，通过它可以为单一应用程序上传多个 APK 版本。

本章可以在以下链接下载：[`www.packtpub.com/sites/default/files/downloads/Multiple_APK_Support.pdf`](http://www.packtpub.com/sites/default/files/downloads/Multiple_APK_Support.pdf)。

*第十章*，*Android Jelly Bean 的 API*，涵盖了 Android Jelly Bean 及其内部的新 API。

本章可以在 [`www.packtpub.com/sites/default/files/downloads/Android_JellyBean.pdf`](http://www.packtpub.com/sites/default/files/downloads/Android_JellyBean.pdf) 下载。

# 你需要这本书的内容

要跟随本书中的示例，需要设置并准备好 Android 开发工具。所需的软件列表如下：

+   带有 ADT 插件的 Eclipse

+   Android SDK 工具

+   Android 平台工具

+   最新的 Android 平台

**可以使用以下操作系统：**

+   Windows XP（32 位）、Vista（32 位或 64 位）或 Windows 7（32 位或 64 位）

+   Mac OS X 10.5.8 或更高版本（仅限 x86）

+   Linux（在 Ubuntu Linux，Lucid Lynx 上测试）

    +   需要使用 GNU C 库（glibc）2.7 或更高版本

    +   在 Ubuntu Linux 上，需要版本 8.04 或更高版本

    +   64 位发行版必须能够运行 32 位应用程序

Eclipse IDE 的使用规范如下：

+   Eclipse 3.6.2（Helios）或更高版本（Eclipse 3.5（Galileo）不再支持最新版本的 ADT）

+   Eclipse JDT 插件（包含在大多数 Eclipse IDE 包中）

+   JDK 6（仅 JRE 不够）

+   Android Development Tools 插件（推荐）

# 本书适合的读者群体

本书适合那些对 Android 平台有经验，但可能不熟悉 Android 4.0 的新特性和 API 的开发者。

想要了解如何支持多种屏幕尺寸和多个 Android 版本的 Android 开发者；这本书也适合你。

# 约定

在这本书中，你会发现多种文本样式，用以区分不同类型的信息。以下是一些样式示例，以及它们含义的解释。

文中的代码字显示如下："实现 `onCreateOptionsMenu` 和 `onOptionsItemSelected` 方法。"

代码块设置如下：

```kt
<?xml version="1.0" encoding="utf-8"?>
<menu  >
    <item android:id="@+id/settings" android:title="Settings">
    </item>
    <item android:id="@+id/about" android:title="About">
    </item>

</menu>
```

当我们希望引起你注意代码块中的特定部分时，相关的行或项目会以粗体设置：

```kt
@Override
public void onPrepareSubMenu(SubMenu subMenu) {
 //In order to add submenus, we should override this method we dynamically created submenus
    subMenu.clear();
    subMenu.add("SubItem1").setOnMenuItemClickListener(this);
    subMenu.add("SubItem2").setOnMenuItemClickListener(this);
  }
```

**新术语**和**重要词汇**以粗体显示。你在屏幕上看到的词，例如菜单或对话框中的，文本中会这样出现："点击 **插入** 按钮然后点击 **列表** 按钮"。

### 注意

警告或重要提示会以这样的框显示。

### 提示

技巧和窍门会像这样出现。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢或可能不喜欢的地方。读者的反馈对我们开发能让你们充分利用的标题很重要。

要给我们发送一般反馈，只需发送电子邮件至 `<feedback@packtpub.com>`，并在邮件的主题中提及书名。

如果你在一个主题上有专业知识，并且有兴趣撰写或参与书籍编写，请查看我们在 [www.packtpub.com/authors](http://www.packtpub.com/authors) 的作者指南。

# 客户支持

既然你现在拥有了 Packt 的一本书，我们有一些事情可以帮助你最大限度地利用你的购买。

## 下载示例代码

你可以从你在 [`www.PacktPub.com`](http://www.PacktPub.com) 的账户下载你购买的所有 Packt 图书的示例代码文件。如果你在其他地方购买了这本书，可以访问 [`www.PacktPub.com/support`](http://www.PacktPub.com/support) 注册，我们会直接将文件通过电子邮件发送给你。

源代码也可以在作者的网站 [www.ottodroid.net](http://www.ottodroid.net) 上获取。

## 错误更正

尽管我们已经竭尽全力确保内容的准确性，但错误仍然可能发生。如果你在我们的书中发现了一个错误——可能是文本或代码中的错误——我们非常感激你能向我们报告。这样做可以节省其他读者的时间，并帮助我们对本书的后续版本进行改进。如果你发现任何错误，请访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 报告，选择你的书籍，点击 **错误更正提交表单** 链接，并输入错误的详细信息。一旦你的错误报告被验证，你的提交将被接受，并且错误更正将会被上传到我们的网站，或在相应标题的“错误更正”部分添加到现有的错误更正列表中。你可以通过访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 选择你的标题来查看任何现有的错误更正。

## 盗版

互联网上版权材料的盗版问题在所有媒体中持续存在。在 Packt，我们非常重视保护我们的版权和许可。如果你在互联网上以任何形式遇到我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

如果你发现了疑似盗版材料，请通过 `<copyright@packtpub.com>` 联系我们，并提供链接。

我们感谢你帮助保护我们的作者，以及我们为你提供有价值内容的能力。

## 问题咨询

如果你在这本书的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决。
