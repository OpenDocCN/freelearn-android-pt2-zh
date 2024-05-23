# 前言

《Eclipse 下的 Android 开发工具》将向你展示如何使用 Eclipse 的 ADT（Android 开发工具）快速建立 Android 项目，创建应用程序 UI，调试以及导出签名（或未签名）的.apk 发布包。本书从 ADT 的安装开始，讨论重要的工具，并从零开始指导你进行 Android 应用开发，演示不同的概念和实现方法，最终帮助你进行分发。

# 本书涵盖的内容

第一章，《安装 Eclipse、ADT 和 SDK》，指导你安装进行 Android 应用开发所需的 Eclipse 和 ADT（Android 开发工具）。

第二章，《IDE 的重要特性》，描述了 Eclipse 和 ADT 环境中几个对开发原生 Android 应用有用的特性。

第三章，《创建一个新的 Android 项目》，指导你创建一个新项目并演示简单部件的使用。它还指导编译、调试和运行应用程序。

第四章，《合并多媒体元素》，将教你如何包含多媒体元素并在应用中处理多个屏幕。

第五章，《添加 RadioButton、CheckBox、Menu 和 Preferences》，涉及添加菜单和偏好设置屏幕以及单选按钮和复选框的使用。

第六章，《处理多种屏幕类型》，教你如何应对不同的屏幕类型和方向。

第七章，《添加外部库》，指导你添加外部库，即 AdMob 库并在应用中添加广告。

第八章，《签名和分发 APK》，展示了签名和分发 Android 应用的步骤。

# 你需要为这本书准备的东西

建议使用以下规格的笔记本电脑或 PC 以获得更好的开发性能：

+   4 GB 内存

+   Windows 7 操作系统

+   双核/i 系列处理器

# 本书适合的读者

《Eclipse 下的 Android 开发工具》面向初学者和希望了解更多关于 Android 开发的现有开发者。假设你有 Java 编程经验并且使用过 IDE 进行开发。

# 约定

在这本书中，你将发现多种文本样式，用于区分不同类型的信息。以下是一些样式的例子及其含义的解释。

文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 处理都如下所示："我们可以通过使用`include`指令包含其他上下文。"

代码块设置如下：

```kt
[default]
exten => s,1,Dial(Zap/1|30)
exten => s,2,Voicemail(u100)
exten => s,102,Voicemail(b100)
exten => i,1,Voicemail(s0)
```

当我们希望引起你对代码块中某个特定部分的注意时，相关的行或项目会以粗体显示：

```kt
[default]
exten => s,1,Dial(Zap/1|30)
exten => s,2,Voicemail(u100)
exten => s,102,Voicemail(b100)
exten => i,1,Voicemail(s0)
```

任何命令行输入或输出都如下所示：

```kt
# cp /usr/src/asterisk-addons/configs/cdr_mysql.conf.sample
     /etc/asterisk/cdr_mysql.conf
```

**新术语**和**重要词汇**会以粗体显示。你在屏幕上看到的内容，例如菜单或对话框中的单词，会在文本中以这样的形式出现："点击**下一步**按钮，你会进入下一个屏幕"。

### 注意

警告或重要提示会以这样的方框显示。

### 小贴士

小技巧会以这样的形式出现。

# 读者反馈

我们始终欢迎读者的反馈。让我们知道你对这本书的看法——你喜欢或可能不喜欢的内容。读者的反馈对我们开发能让你们充分利用的标题非常重要。

要向我们发送一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在邮件的主题中提及书名。如果你有专业知识的话题，并且有兴趣撰写或为书籍做出贡献，请查看我们在[www.packtpub.com/authors](http://www.packtpub.com/authors)上的作者指南。

# 客户支持

既然你现在拥有了 Packt 的一本书，我们有一些事情可以帮助你最大限度地利用你的购买。

## 下载示例代码

你可以从你在[`www.packtpub.com`](http://www.packtpub.com)的账户下载你购买的所有 Packt 书籍的示例代码文件。如果你在其他地方购买了这本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)注册，我们会直接将文件通过电子邮件发送给你。

## 勘误

尽管我们已经竭尽所能确保内容的准确性，但错误仍然在所难免。如果你在我们的书中发现了一个错误——可能是文本或代码中的错误——如果你能向我们报告，我们将不胜感激。这样做，你可以避免其他读者的困扰，并帮助我们改进本书后续版本。如果你发现任何勘误信息，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择你的书籍，点击**勘误提交表单**链接，并输入你的勘误详情。一旦你的勘误信息被核实，你的提交将被接受，并且勘误信息将被上传到我们的网站，或添加到该标题下的现有勘误列表中。任何现有的勘误信息可以通过访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并选择你的标题来查看。

## 盗版

互联网上版权资料的盗版问题在所有媒体中持续存在。在 Packt，我们非常重视对我们版权和许可的保护。如果您在网上以任何形式遇到我们作品的非法副本，请立即向我们提供位置地址或网站名称，以便我们可以寻求补救措施。

如果您有疑似盗版资料的链接，请联系`<copyright@packtpub.com>`。

我们感谢您在保护我们作者权益方面所提供的帮助，以及我们向您提供有价值内容的能力。

## 咨询问题

如果您在书的任何方面遇到问题，可以联系`<questions@packtpub.com>`，我们将尽力解决。