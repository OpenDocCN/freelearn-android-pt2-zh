# 前言

移动设备上的自动化测试已经存在了好几年，但真正得到发展是在 Robotium 框架出现之后。

在自动化测试用例的帮助下，业务组件得到了广泛的重用，有助于执行复杂的测试用例。由于 Robotium 框架添加了许多不同的关键特性，它已经成为世界上领先的 Android 测试自动化框架，大多数行业专家和专业人士都在使用这个框架来测试他们的 Android 业务应用。

将这本书推向市场的目的是为用户提供关于 Robotium 框架及其特性的详细知识。阅读之后，你应该能够开始创建自动化测试用例，并为你的 Android 项目运行它们！

欢迎来到 Android 的 Robotium 自动化测试！

# 这本书涵盖的内容

第一章，*Robotium 入门*，讨论了 Robotium 框架，并帮助我们一步步在 Windows 上安装和设置 Android 环境。

第二章，*使用 Robotium 创建测试项目*，指导你创建一个测试项目，并帮助你使用 Eclipse 运行它。

第三章，*Robotium API*，介绍 `Solo` 类以及框架中存在的 API 信息。它还将教会你关于国际化的知识。

第四章，*Robotium 中的 Web 支持*，简要介绍如何使用 Robotium 的 Web 支持在 Android 中访问网页元素。

第五章，*与其他框架的比较*，旨在基于某些参数提供 Robotium 与其他测试框架之间的比较。

第六章，*Robotium 中的远程控制*，介绍软件自动化框架支持以及 Android 中远程控制的工作原理。

第七章，*Robotium 的其他工具*，包含了 Robotium 框架中各种现成的工具。这些工具包括 `RobotiumUtils` 类、XPath 的使用、Robotium 在已安装的 Android 应用中的使用，以及在应用签名和取消签名操作期间涉及的签名过程，以执行测试。

第八章，*Maven 中的 Robotium*，简要介绍 Maven 工具，该工具帮助你将 Android 项目连接到构建过程。这一章还解释了你需要使用 Maven 配置 Robotium 的不同方法。

# 你需要为这本书准备什么

对于这本书，你需要有 Windows XP（或更新版本）、Linux 或 Mac OS X 操作系统。

你需要下载并安装 Android SDK 和 Eclipse IDE（参考第一章中的*设置 Android 环境*部分，*Robotium 入门*）。

# 本书适合的读者对象

Robotium 是针对 Android 应用程序的自动化测试用例开发者的框架。本书旨在帮助初学者熟悉 Robotium SDK。你需要对 Java 和 Android 编程有基本的了解，以及基本的命令行熟悉度。

# 约定

在这本书中，你会发现多种文本样式，用于区分不同类型的信息。以下是一些样式示例，以及它们的含义解释。

文本中的代码词汇如下所示："我们可以通过使用`include`指令包含其他上下文。"

网站参考链接如下所示：[`github.com/jayway/robotium/tree/master/robotium-solo`](https://github.com/jayway/robotium/tree/master/robotium-solo)

代码块如下设置：

```kt
Activity activity = solo.getCurrentActivity();

ImageView imageView = (ImageView) solo.getView(act.getResources().getIdentifier("appicon", "id", act.getPackageName()));
```

任何命令行输入或输出如下所示：

```kt
# adb push app.apk <path>

```

# 读者反馈

我们始终欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢或可能不喜欢的内容。读者的反馈对我们开发能让你们充分利用的标题非常重要。

如果要给我们发送一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在邮件的主题中提及书名。

如果你对某个主题有专业知识，并且有兴趣撰写或参与书籍编写，请查看我们在[www.packtpub.com/authors](http://www.packtpub.com/authors)上的作者指南。

# 客户支持

既然你现在拥有了 Packt 的一本书，我们有一些事情可以帮助你最大限度地利用你的购买。

## 下载示例代码

你可以从你在[`www.packtpub.com`](http://www.packtpub.com)的账户下载你所购买的所有 Packt 图书的示例代码文件。如果你在其他地方购买了这本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)注册，我们会直接将文件通过电子邮件发送给你。

## 勘误

尽管我们已经竭尽全力确保内容的准确性，但错误仍然会发生。如果您在我们的书中发现了一个错误——可能是文本或代码中的错误——如果您能报告给我们，我们将不胜感激。这样做，您可以避免其他读者感到沮丧，并帮助我们在后续版本中改进这本书。如果您发现了任何勘误信息，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击 **errata** **submission** **form** 链接，并输入您的勘误详情。一旦您的勘误信息得到验证，您的提交将被接受，并且勘误信息将被上传到我们的网站，或添加到该标题勘误部分现有的勘误列表中。任何现有的勘误信息可以通过选择您的标题从 [`www.packtpub.com/support`](http://www.packtpub.com/support) 进行查看。

## 盗版

互联网上对版权材料的盗版行为是所有媒体持续面临的问题。在 Packt，我们非常重视对我们版权和许可的保护。如果您在网上以任何形式遇到我们作品的非法副本，请立即提供其位置地址或网站名称，以便我们可以寻求补救措施。

如果您发现了疑似盗版材料，请通过 `<copyright@packtpub.com>` 联系我们，并提供该材料的链接。

我们感谢您帮助保护我们的作者，以及我们为您带来有价值内容的能力。

## 问题

如果您在书的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决。
