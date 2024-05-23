# 前言

*Android Canvas 学习* 提供了 Android Canvas 图形和编程的基本知识和理解。目标读者被假定为对 Canvas 以及 Android 应用程序开发中的图形处理没有任何先验知识。将读者从基本的图形和 Canvas 编程知识带到中级 Android Canvas 编程知识。本书仅关注 2D 图形，不包括 3D 或动画，但为过渡到动画和 3D 图形学习提供了一个非常坚实的基础。它提供了从图形基础到不同图形对象和技术的实践逐步指导，再到更复杂的交互式图形丰富的 Android 应用程序。

# 这本书涵盖的内容

第一章, *Android Canvas 入门*, 提供了关于 Android Canvas、2D 图形、显示屏幕及其基本理解的一些背景知识，还介绍了图形丰富应用程序中的重要文件。

第二章，*绘图线程*, 有助于理解线程的需求、角色和用途，以及与线程相关的问题和解决方案。

第三章, *Android Canvas 中的绘图和 Drawable*, 向读者介绍了一些 Drawable 以及在 canvas、view 和 surface view 上绘图。

第四章, *NinePatch 图片*, 解释了切片的基本概念，`NinePatch`图片，重复和非重复区域，以及使用它们创建背景。

第五章, *触摸事件和在 Canvas 上绘图*, 解释了捕捉触摸事件并相应地做出响应。还涵盖了创建自定义`View`类及其实现，包括触摸事件实现。

第六章, *整合应用*, 讲述了如何规划一个应用程序，从零开始创建具有复杂用户界面的应用程序，并将之前学到的所有知识付诸实践。

# 你需要为这本书准备什么

你需要以下软件来运行本书中的示例：

+   一台具有合理处理能力和 RAM 的计算机，Intel Core i3 就能胜任这项工作

+   Java 运行时环境

+   Eclipse 经典版

+   Android SDK，最新版本将是最佳选择

# 这本书适合的读者群体

熟悉 Java 编码和一些基本的 Android 开发知识的开发者。这本书适合那些具备基本的 Android 开发知识但对 Android Canvas 开发一无所知的开发者，也适合对图形丰富的应用程序和游戏开发感兴趣的开发者。

# 编写约定

在这本书中，你将发现多种文本样式，用于区分不同类型的信息。以下是一些样式示例及其含义的解释。

文本中的代码字如下所示："我们将把我们的应用程序命名为`MyFirstCanvasApp`。"

代码块设置如下：

```kt
class OurGameView extends SurfaceView implements SurfaceHolder.Callback {
// Our class functionality comes here
}
```

当我们希望您关注代码块的某个部分时，相关的行或项目会以粗体显示：

```kt
android:background="@drawable/myfirstninepatch"
android:text="@string/buttonwith9patchimage"

```

任何命令行输入或输出都如下编写：

```kt
C:\learningandroidcanvasmini\chapter1\firstapp

```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的内容，例如菜单或对话框中的，会在文本中这样显示："为此，在 Eclipse 中，我们将导航至**文件** | **新建** | **Android 应用程序项目**。"

### 注意

警告或重要说明会以这样的框显示。

### 提示

提示和技巧会像这样出现。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或可能不喜欢的内容。读者的反馈对我们来说非常重要，可以帮助我们开发出您真正能从中受益的图书。

如果要给我们发送一般性反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在邮件的主题中提及书名。

如果您在某个主题上有专业知识，并且有兴趣撰写或参与图书编写，请查看我们在[www.packtpub.com/authors](http://www.packtpub.com/authors)的作者指南。

# 客户支持

既然您已经成为了 Packt 图书的骄傲拥有者，我们有许多方式帮助您充分利用您的购买。

## 下载示例代码

您可以从您的账户下载您购买的所有 Packt 图书的示例代码文件，访问地址为[`www.packtpub.com`](http://www.packtpub.com)。如果您在别处购买了这本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)进行注册，我们会将文件通过电子邮件直接发送给您。

## 勘误

尽管我们已经尽力确保内容的准确性，但错误仍然会发生。如果您在我们的书中发现了一个错误——可能是文本或代码中的错误——如果您能报告给我们，我们将不胜感激。这样做，您可以避免其他读者的困扰，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情。一旦您的勘误被验证，您的提交将被接受，勘误将在我们网站的相应位置上传，或添加到现有勘误列表中。任何现有的勘误可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题来查看。

## 盗版

互联网上对版权材料的海盗行为是所有媒体都面临的持续问题。在 Packt，我们非常重视对我们版权和许可的保护。如果您在网上以任何形式遇到我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

如果您发现疑似盗版材料，请通过`<copyright@packtpub.com>`联系我们，并提供该材料的链接。

我们感谢您帮助保护我们的作者，以及我们为您带来有价值内容的能力。

## 问题

如果您在书的任何方面遇到问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决。
