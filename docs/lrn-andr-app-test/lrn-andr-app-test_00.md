# 前言

无论您在 Android 设计上投入多少时间，甚至在编程时多么小心，错误是不可避免的，bug 也会出现。这本书将帮助您最小化这些错误对您的 Android 项目的影响，并提高您的开发生产力。它将向您展示那些容易避免的问题，帮助您快速进入测试阶段。

《Android 应用测试指南》是第一本也是唯一一本提供实用介绍的书，旨在通过使用各种方法提高您的 Android 应用程序开发，介绍最常见的技术、框架和工具。

作者在将应用测试技术应用于实际项目方面的经验，使他能够分享关于创建专业级 Android 应用程序的见解。

本书涵盖了测试框架支持的基础知识，以及如测试驱动开发（Test-driven Development）等架构和技术，这是软件开发过程中的一个敏捷组成部分，也是一项早期解决错误的技术。从应用于示例项目最基本的单元测试到更复杂的性能测试，这本书以食谱式的方法提供了 Android 测试领域中最广泛使用技术的详细描述。

作者在其职业生涯中参与了各种开发项目，拥有丰富的经验。所有这些研究和知识都有助于创作一本对任何开发者在 Android 测试世界中导航都有用的资源书。

# 本书内容涵盖

第一章，*测试入门*，介绍了不同类型的测试及其一般适用于软件开发项目，特别是 Android。然后继续讲述 Android 平台上的测试、单元测试和 JUnit、创建 Android 测试项目并运行测试。

第二章，*使用 Android SDK 理解测试*，开始更深入地识别可用于创建测试的构建块。它涵盖了断言、TouchUtils（用于测试用户界面）、模拟对象、测试仪器和 TestCase 类层次结构。

第三章，*使用测试食谱烘焙*，提供了在应用前面描述的纪律和技术时通常会遇到的常见情况的实用示例。这些示例以食谱风格呈现，以便您可以根据自己的项目进行改编和使用。这些食谱涵盖了 Android 单元测试、活动、应用程序、数据库和 ContentProviders、服务、UI、异常、解析器、内存泄漏，以及使用 Espresso 进行测试的探讨。

第四章，*管理你的 Android 测试环境*，提供了不同的条件来运行测试。它从创建 Android 虚拟设备（AVD）开始，为被测应用程序提供不同的条件和配置，并使用可用选项运行测试。最后，它引入了猴子作为生成用于测试的模拟事件的方式。

第五章，*探索持续集成*，介绍了这种敏捷软件工程和自动化技术，旨在通过持续集成和频繁测试来提高软件质量并减少集成更改所需的时间。

第六章，*实践测试驱动开发*，介绍了测试驱动开发这一纪律。它从一般性复习开始，随后转移到与 Android 平台紧密相关 concepts 概念和技巧。这是一个代码密集型的章节。

第七章，*行为驱动开发*，介绍了行为驱动开发以及一些概念，例如使用通用词汇表达测试，以及将业务参与者包括在软件开发项目中。

第八章，*测试和性能分析*，介绍了一系列与基准测试和性能分析相关的概念，从传统的日志语句方法到创建 Android 性能测试并使用分析工具。

第九章，*替代测试策略*，涵盖了添加代码覆盖率以确保你知道哪些已测试哪些未测试，以及在宿主的 Java 虚拟机上测试，研究 Fest、Spoon 以及 Android 测试的未来，以构建和扩展你的 Android 测试范围。

# 你需要为这本书准备的内容。

为了能够跟随不同章节中的示例，你需要安装一组常见的软件和工具，以及每个章节特别描述的其他组件，包括它们的下载位置。

所有示例基于以下内容：

+   Mac OSX 10.9.4，已完全更新

+   Java SE 版本 1.6.0_24（构建版本 1.6.0_24-b07）

+   Android SDK 工具，版本 24

+   Android SDK 平台工具，版本 21

+   SDK 平台 Android 4.4，API 20

+   Android 支持库，版本 21

+   Android Studio IDE，版本：1.1.0

+   Gradle 版本 2.2.1

+   Git 版本 1.8.5.2

# 本书的目标读者

如果你是一个希望测试应用程序或优化应用开发流程的 Android 开发者，那么这本书就是为你而写的。不需要有应用程序测试的先前经验。

# 约定

在这本书中，你会发现多种文本样式，用于区分不同类型的信息。以下是一些样式示例及其含义的解释。

文本中的代码字词如下所示："要调用`am`命令，我们将使用`adb` `shell`命令"。

代码块设置如下：

```kt
dependencies {
    compile project(':dummylibrary')
}
```

当我们希望引起你对代码块中某个特定部分的注意时，相关的行或项目会以粗体显示：

```kt
fahrenheitEditNumber
.addTextChangedListener(
newFehrenheitToCelciusWatcher(fahrenheitEditNumber, celsiusEditNumber));
}

```

任何命令行输入或输出都如下写出：

```kt
junit.framework.ComparisonFailure: expected:<[]> but was:<[123.45]>
at com.blundell.tut.EditNumberTests.testClear(EditNumberTests.java:31)
at java.lang.reflect.Method.invokeNative(Native Method)
at android.test.AndroidTestRunner.runTest(AndroidTestRunner.java:191)
```

**新术语**和**重要词汇**以粗体显示。你在屏幕上看到的内容，例如菜单或对话框中的单词，会像这样出现在文本中："第一个测试执行了对 Forwarding Activity 的**Go**按钮的点击。"

### 注意

警告或重要说明会像这样出现在一个框中。

### 提示

提示和技巧会像这样出现。

# 读者反馈

我们非常欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢或不喜欢的地方。读者的反馈对我们很重要，因为它能帮助我们开发出你真正能从中获益的图书。

如需向我们发送一般反馈，只需将邮件发送至`<feedback@packtpub.com>`，并在邮件的主题中提及书籍的标题。

如果你在一个主题上有专业知识，并且有兴趣撰写或参与书籍编写，请查看我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然你现在拥有了 Packt 的一本书，我们有一些事情可以帮助你最大限度地利用你的购买。

## 下载示例代码

你可以从你在[`www.packtpub.com`](http://www.packtpub.com)的账户下载你所购买的所有 Packt Publishing 书籍的示例代码文件。如果你在其他地方购买了这本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)注册，我们会直接将文件通过电子邮件发送给你。

## 勘误

尽管我们已经尽力确保内容的准确性，但错误仍然会发生。如果你在我们的书中发现了一个错误——可能是文本或代码中的错误——如果你能向我们报告，我们将不胜感激。这样做，你可以避免其他读者的困扰，并帮助我们改进本书后续版本。如果你发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择你的书籍，点击**Errata Submission Form**链接，并输入你的勘误详情。一旦你的勘误被验证，你的提交将被接受，勘误将被上传到我们的网站或添加到该标题下现有勘误列表中。

若要查看之前提交的勘误信息，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，在搜索栏中输入书籍名称。所需信息将在**勘误**部分显示。

## 盗版

在互联网上，版权材料的盗版问题在所有媒体中持续存在。Packt 出版社非常重视我们版权和许可的保护。如果您在任何形式的互联网上发现我们作品非法复制的版本，请立即提供其位置地址或网站名称，以便我们可以采取补救措施。

如果您发现任何涉嫌盗版的材料，请通过`<copyright@packtpub.com>`联系我们，并提供相关链接。

我们感谢您帮助保护我们的作者以及我们为您提供有价值内容的能力。

## 问题咨询

如果您对这本书的任何方面有疑问，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决问题。

## 问题咨询

如果您在阅读本书的任何部分遇到问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽我们所能予以解决。
