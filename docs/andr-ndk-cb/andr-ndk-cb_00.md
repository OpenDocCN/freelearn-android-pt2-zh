# 序言

自 2008 年首次发布以来，安卓已成为世界上最大的移动平台。预计到 2013 年中，Google Play 中的应用总数将达到 100 万。大多数安卓应用都是使用安卓软件开发工具包（SDK）的 Java 编写的。许多开发者即使有 C/C++经验，也只编写 Java 代码，没有意识到他们放弃了一个多么强大的工具。

安卓**原生开发工具包**（**NDK**）于 2009 年发布，旨在帮助开发者编写和移植原生代码。它提供了一套交叉编译工具和一些库。使用 NDK 编程有两个主要优点。首先，你可以用原生代码优化你的应用程序并提升性能。其次，你可以复用大量的现有 C/C++代码。《*Android Native Development Kit*》是一本实用的指南，帮助你使用 NDK 编写安卓原生代码。我们将从基础内容开始，例如**Java 原生接口**（**JNI**），并构建和调试一个原生应用程序（第 1 至 3 章）。然后，我们将探讨 NDK 提供的各种库，包括 OpenGL ES、原生应用程序 API、OpenSL ES、OpenMAX AL 等（第 4 至 7 章）。之后，我们将讨论如何使用 NDK 将现有应用程序和库移植到安卓（第 8 和 9 章）。最后，我们将展示如何使用 NDK 编写多媒体应用程序和游戏（附加章节 1 和 2）。

# 本书涵盖内容

第一章，*Hello NDK*，介绍了如何在 Windows、Linux 和 MacOS 中设置安卓 NDK 开发环境。我们将在本章末尾编写一个“Hello NDK”应用程序。

第二章，*Java Native Interface*，详细描述了 JNI 的使用方法。我们将从 Java 代码中调用原生方法，反之亦然。

第三章，*Build and Debug NDK Applications*，展示了如何从命令行和 Eclipse IDE 构建原生代码。我们还将探讨使用`gdb`、`cgdb`、eclipse 等调试原生代码的方法。

第四章，*Android NDK OpenGL ES API*，阐述了 OpenGL ES 1.x 和 2.0 API。我们将涵盖 2D 绘图、3D 图形、纹理映射、EGL 等内容。

第五章，*Android Native Application API*，讨论了安卓原生应用程序 API，包括管理原生窗口、访问传感器、处理输入事件、管理资源等。在本章中，我们将看到如何编写一个纯粹的原生应用程序。

第六章，*Android NDK Multithreading*，描述了安卓多线程 API。我们将涵盖创建和终止原生线程、各种线程同步技术（互斥量、条件变量、信号量以及读写锁）、线程调度和线程数据管理。

第七章，*其他 Android NDK API*，讨论了一些额外的 Android 库，包括`jnigraphics`图形库，动态链接库，`zlib`压缩库，OpenSL ES 音频库，以及 OpenMAX AL 媒体库。

第八章，*使用 Android NDK 移植和使用现有库*，描述了使用 NDK 移植和使用现有 C/C++库的各种技术。在章节的最后，我们将移植`boost`库。

第九章，*使用 NDK 将现有应用移植到 Android*，提供了分步指南，用于使用 NDK 将现有应用移植到 Android。我们以一个开源的图像调整大小程序为例。

*第一章附录*，*使用 NDK 开发多媒体应用*，演示了如何使用`ffmpeg`库编写多媒体应用。我们将移植`ffmpeg`库，并使用库 API 编写一个帧抓取器应用。

*第二章附录*，*使用 NDK 开发游戏*，讨论了使用 NDK 编写游戏。我们将移植《德军总部 3D》游戏，以展示如何为游戏设置显示、添加游戏控制以及启用音频效果。

你可以从以下链接下载附录章节：[`www.packtpub.com/sites/default/files/downloads/Developing_Multimedia_Applications_with_NDK.pdf`](http://www.packtpub.com/sites/default/files/downloads/Developing_Multimedia_Applications_with_NDK.pdf) 和 [`www.packtpub.com/sites/default/files/downloads/Developing_Games_with_NDK.pdf`](http://www.packtpub.com/sites/default/files/downloads/Developing_Games_with_NDK.pdf)。

# 阅读本书所需的知识

需要一台安装了 Windows、Ubuntu Linux 或 MacOS 的计算机（推荐使用 Linux 或 MacOS）。虽然我们可以使用模拟器运行 Android 应用，但对于 Android 开发来说既慢又低效。因此，建议使用 Android 设备。

本书假设读者具备 C 和 C++编程语言的基础知识。你也应该熟悉 Java 和 Android SDK。

请注意，除非另有说明，本书的示例代码基于 Android ndk r8，因为这是本书撰写时的最新版本 NDK。到本书出版时，应该会有更新的版本。代码也应该在任何新版本上运行。因此，我们可以安装 NDK r8 或更高版本。

# 本书适合的读者

本书面向任何对为 Android 编写原生代码感兴趣的人。章节从基础到中级再到高级排列，相对独立。对于 NDK 的新手，建议从头到尾阅读，而熟悉 NDK 的读者可以选择任何特定的章节，甚至是特定的食谱。

# 编写约定

在这本书中，你会发现多种文本样式，用于区分不同类型的信息。以下是一些样式示例，以及它们含义的解释。

文本中的代码字如下所示："Windows NDK 带有一个新的 `ndk-build.cmd` 构建脚本。"

代码块设置如下：

```kt
#include <string.h>
#include <jni.h>

jstring 
Java_cookbook_chapter1_HelloNDKActivity_naGetHelloNDKStr(JNIEnv* pEnv,  jobject pObj)
{
    return (*pEnv)->NewStringUTF(pEnv, "Hello NDK!");
}
```

当我们希望引起你注意代码块的某个部分时，相关的行或项目将以粗体显示：

```kt
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE  := framegrabber
LOCAL_SRC_FILES := framegrabber.c
#LOCAL_CFLAGS := -DANDROID_BUILD
LOCAL_LDLIBS := -llog -ljnigraphics -lz  
LOCAL_STATIC_LIBRARIES := libavformat_static libavcodec_static libswscale_static libavutil_static
include $(BUILD_SHARED_LIBRARY)
$(call import-module,ffmpeg-1.0.1/android/armv5te)

```

命令行输入或输出将如下书写：

```kt
$sudo update-java-alternatives -s <java name>

```

**新术语**和**重要词汇**以粗体显示。你在屏幕上看到的词，例如菜单或对话框中的，会在文本中以这样的形式出现："转到**控制面板** | **系统和安全** | **系统** | **高级系统设置**。"

### 注意

警告或重要说明将如下框所示。

### 提示

提示和技巧如下所示。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢或可能不喜欢的内容。读者的反馈对我们开发能让你们充分利用的标题非常重要。

要给我们发送一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在邮件的主题中提及书名。

如果你对某个主题有专业知识，并且有兴趣撰写或参与书籍编写，请查看我们在[www.packtpub.com/authors](http://www.packtpub.com/authors)的作者指南。

# 客户支持

既然你现在拥有了 Packt 的一本书，我们有很多方法可以帮助你充分利用你的购买。

## 下载示例代码

你可以从你在[`www.packtpub.com`](http://www.packtpub.com)的账户下载你所购买的所有 Packt 图书的示例代码文件。如果你在其他地方购买了这本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)注册，我们会直接将文件通过电子邮件发送给你。

## 错误更正

尽管我们已经尽力确保内容的准确性，但错误仍然会发生。如果你在我们的书中发现了一个错误——可能是文本或代码中的错误——如果你能报告给我们，我们将不胜感激。这样做，你可以让其他读者免受挫折，并帮助我们改进本书的后续版本。如果你发现任何错误更正，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择你的书，点击**错误更正** **提交** **表单**链接，并输入你的错误更正详情。一旦你的错误更正得到验证，你的提交将被接受，并且错误更正将在我们网站的相应位置上传，或添加到现有错误更正列表中。任何现有的错误更正可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择你的标题来查看。

## 盗版

网络上的版权材料盗版问题在所有媒体中持续存在。在 Packt，我们非常重视对我们版权和许可的保护。如果您在网上以任何形式遇到我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

如果您发现疑似盗版材料，请通过`<copyright@packtpub.com>`联系我们，并提供链接。

我们感谢您在保护我们的作者以及我们为您提供有价值内容的能力方面的帮助。

## 问题咨询

如果您在阅读本书的任何方面遇到问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决。
