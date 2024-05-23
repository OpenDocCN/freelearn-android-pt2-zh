# 前言

Android 6 主要关注改善整体用户体验，并引入了一些功能，例如重新设计的权限模型，在该模型中，应用程序在安装时不再自动获得其指定权限，设备在用户未操作时采用 Doze 电源方案以延长电池寿命，以及原生支持指纹识别。

如果你已经是 Android 开发者，只需几步之遥就能利用你现有的开发经验，无论用户在任何时间、任何地点需要或想要你的 app，你都能触达他们。

作为一名专业的 Android 开发人员，你必须为用户创建可用于生产的 app。这本书将为你提供作为公司开发团队的一员、独立 app 开发者或仅作为使用 Android 开发最佳实践的程序员发布精致 app 所需要的一切。

在本书的最后，你将能够识别应用程序中需要改进的关键领域，并实施必要的更改和完善，以确保在发布前符合 Android 核心应用指南。

# 本书涵盖的内容

第一章，*Android Marshmallow 权限*，讨论了 Android 权限系统和模型非常广泛，并且进行了一些可以帮助应用开发者和应用程序获得更多吸引力、安装量，以及让用户决定你的应用程序何时能够使用每个依赖权限功能的更改。但请记住，这只是一个起点，Android Marshmallow 仍需要获得市场份额并得到 OEM 厂商的采用，使用户能够自由选择。作为应用开发者，你必须提前做好准备，确保你的应用开发面向未来，让新用户尽快享受最新的更新，同时保持你的应用程序的高性能水平。

第二章，*应用链接*，讲述了应用链接在 Android Marshmallow 中变得多么强大。这允许你，应用开发者，帮助系统更好地决定如何行动。处理网页 URL 将为你提供更广泛的曝光，更多进入你 app 的渠道，以及你可以为用户提供更好的体验（这都有助于提高评分和增加下载量，反之亦然）。

应用链接实现简单，易于理解，在当今的移动/网络世界中是必备功能。尽管应用链接为使用你应用程序的用户提供了更好的操作处理，但用户可能会有多个设备，期望在每个设备上都有相同的行为，如果他们的数据和操作处理能够全方位的话，他们会更加投入。

第三章，*应用自动备份*，告知你 Android Marshmallow 带来了一项为应用备份的出色功能，这减少了用户迁移到新设备时的摩擦。

在一个充满各种不同应用的世界里，最大化自动备份的好处可以带来极佳的用户体验。这一功能的目标是减轻负担，缩短设置新设备所需的时间，并安装用户喜爱的应用。如果需要，在全新安装后，仅通过密码提示就让用户进入你的应用，可以提供很好的体验。试试看吧！

第四章，*变革展开*，概述了 Android Marshmallow 中的一些变化。所有这些变化都值得关注，并将在你的应用开发周期中提供帮助。未来章节将讨论更多变化，并采用更详细的方法。

第五章，*音频、视频和相机功能*，涵盖了 Android API 的许多变化和新增内容。Android Marshmallow 更多的是帮助我们开发者实现更好的媒体支持，在使用音频、视频或相机 API 时展示我们的创意。

第六章，*工作用 Android*，涵盖了 Android Marshmallow 为工作用 Android 世界带来的诸多变化。作为开发者，我们需要始终与组织的需要保持可行的联系。确保我们通过 Marshmallow 的变化了解工作用 Android 的世界，可以帮助我们构建并针对企业工作流程，同时享有更简单 API 的好处。

第七章，*Chrome 自定义标签页*，讨论了新增加的功能，Chrome 自定义标签页，它允许我们开发者将网页内容嵌入到我们的应用中，修改 UI，并根据我们应用的主题和色彩以及观感进行调整。这帮助我们让用户留在我们的应用中，同时提供良好的 UI 和整体体验。

第八章, *认证*, 讨论了 Android Marshmallow 如何通过指纹 API 为我们提供了一个新的用户认证接口。我们可以使用传感器，甚至在我们的应用程序内对用户进行认证，并在需要时保存，以避免使用 Android Marshmallow 引入的凭据宽限期功能来登录用户。我们还介绍了一种方法，通过仅使用 HTTPS 来提高应用程序的安全性。借助 usesCleartextTraffic 标志实施的 StrictMode 策略，可以确保我们连接到外部世界的所有节点都经过检查，以确认是否需要安全连接。

# 你需要为这本书准备什么

对于这本书，你需要具备 Android 平台、API 以及应用程序开发过程的前期知识。你还需要设置工作环境，至少具备以下条件：

+   可以从[`developer.android.com/sdk/index.html`](https://developer.android.com/sdk/index.html)下载的 Android Studio

+   最新的 Android SDK 工具和平台。请确保升级到最新版本，如果缺少的话，添加 Android 6.0（Marshmallow）平台。

+   安卓设备很有帮助，但如果你愿意，也可以使用模拟器，或者你可以使用 Genymotion 的出色解决方案作为模拟器，访问[`www.genymotion.com/`](https://www.genymotion.com/)

# 这本书的目标读者

这本书是为希望轻松地将应用程序迁移到下一个 Android 版本的安卓开发者准备的。在本书的章节中，作者将 Android 6 称为 Android Marshmallow。你应该对 Java 和之前的 Android API 有很好的了解，并且应该能够使用 Marshmallow 之前的 API 编写应用程序。

# 约定

在这本书中，你会发现多种文本样式，用于区分不同类型的信息。以下是一些样式示例及其含义的解释。

文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 处理程序如下所示："添加了`setTorchMode()`方法来控制闪光灯手电筒模式。"

代码块设置如下：

```kt
<?xml version="1.0" encoding="utf-8"?>
<full-backup-content>
  <exclude domain="database" path="sensitive_database_name.db"/>
  <exclude domain="sharedpref" path="androidapp_shared_prefs_name"/>
  <exclude domain="file" path="some_file.file_Extension"/>
  <exclude domain="file" path="some_file.file_Extension"/>
</full-backup-content>
```

任何命令行输入或输出都如下所示：

```kt
$ adb shell sm set-force-adoptable true

```

**新术语**和**重要词汇**以粗体显示。你在屏幕上看到的内容，例如菜单或对话框中的，会像这样出现在文本中："现在当你前往**设置** | **更多** | **VPN**时，你可以查看 VPN 应用。"

### 注意

警告或重要提示会像这样出现在一个框里。

### 提示

提示和技巧会像这样出现。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢或不喜欢什么。读者的反馈对我们很重要，因为它帮助我们开发出你真正能从中获得最大收益的标题。

如需向我们提供一般性反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及书籍标题。

如果您在某个主题上有专业知识，并且有兴趣撰写或参与书籍编写，请查看我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然您已成为 Packt 图书的骄傲拥有者，我们有一系列措施帮助您充分利用您的购买。

## 下载示例代码

您可以从您的账户[`www.packtpub.com`](http://www.packtpub.com)下载您购买的所有 Packt Publishing 书籍的示例代码文件。如果您在别处购买了这本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)注册，我们会将文件直接通过电子邮件发送给您。

### 下载本书的颜色图像

我们还为您提供了一份 PDF 文件，其中包含本书中使用的屏幕截图/图表的颜色图像。颜色图像可以帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites`](https://www.packtpub.com/sites)/[default/files/downloads/4412OS_ColoredImages.pdf](http://default/files/downloads/4412OS_ColoredImages.pdf)下载此文件。

## 勘误

尽管我们已经尽力确保内容的准确性，但错误仍然会发生。如果您在我们的书中发现了一个错误——可能是文本或代码中的错误——如果您能报告给我们，我们将不胜感激。这样做，您可以避免其他读者感到沮丧，并帮助我们改进该书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情。一旦您的勘误被验证，您的提交将被接受，勘误信息将被上传到我们的网站或添加到该标题勘误部分下的现有勘误列表中。

若要查看之前提交的勘误信息，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将显示在**勘误**部分下。

## 盗版

网络上对版权材料的盗版是所有媒体面临的持续问题。在 Packt，我们非常重视保护我们的版权和许可。如果您在互联网上以任何形式发现我们作品非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

如发现疑似盗版材料，请通过`<copyright@packtpub.com>`联系我们，并提供相关链接。

我们感谢您帮助保护我们的作者以及我们提供有价值内容的能力。

## 问题

如果您对这本书的任何方面有问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决问题。
