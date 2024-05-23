# 前言

能够与计算机交谈的想法长期以来吸引着许多人。然而，直到最近，这看起来还像是科幻小说的内容。现在情况已经改变，拥有智能手机或平板电脑的人可以使用语音在设备上执行许多任务——你可以发送短信、更新日历、设置闹钟，以及提出你之前会输入到搜索框中的问题。在小型设备上，语音输入通常更为方便，尤其是在物理限制使得打字和点击更加困难的情况下。

本书为 Android 设备开发语音应用提供了实用指南，使用谷歌语音 API 进行文本到语音（TTS）和自动语音识别（ASR）以及其他开源软件。尽管有许多书籍涵盖了 Android 编程的通用内容，但没有一个资源全面处理为 Android 开发基于语音的应用程序。

开发语音用户界面与开发更传统界面的许多特性是相同的，但也存在一些特定的要求使得语音应用开发与众不同，开发者进入这一领域时，了解常见的陷阱和困难是很重要的。本书提供了一些入门材料，以覆盖那些可能不为主流计算背景的专业人士所熟悉的方面。然后详细展示了如何从简单的程序逐步构建起复杂的应用程序。通过在书中示例的基础上进行实践，尝试描述的技术，你将能够为你的 Android 应用带来语音的力量，使它们更智能、更直观，提升用户的移动体验。

# 本书涵盖的内容

第一章，*Android 设备上的语音*，讨论了如何在 Android 设备上使用语音，并概述了涉及的技术。

第二章，*文本到语音合成*，涵盖了文本到语音合成技术以及如何使用谷歌 TTS 引擎。

第三章，*语音识别*，概述了语音识别技术以及如何使用谷歌语音识别引擎。

第四章，*简单语音交互*，展示了如何构建用户与应用程序可以进行对话的简单交互，以获取一些信息或执行操作。

第五章，*表单填充对话框*，说明了如何创建与传统网页应用中表单填充类似的语音对话框。

第六章，*对话语法*，介绍了如何使用语法来解释用户的输入，这些输入超出了单个词和短语。

第七章，*多语言和多模态对话*，探讨了如何构建使用不同语言和模态的应用程序。

第八章，*与虚拟个人助手的对话*，展示了如何构建一个支持语音的个人助手。

第九章，*进一步探索*，展示了如何开发更高级的虚拟个人助手。

# 您需要这本书的内容

要运行代码示例并开发您自己的应用程序，您需要安装 Android SDK 和平台工具。一个包含必要的 Android SDK 组件以及内置 ADT（Android 开发者工具）版本的 Eclipse IDE 和教程的完整捆绑包可以在[`developer.android.com/sdk/`](http://developer.android.com/sdk/)下载。

您还需要一个 Android 设备来构建和测试示例，因为 Android ASR（语音识别）在虚拟设备（模拟器）上无法工作。

# 本书的目标读者

本书面向所有对语音应用开发感兴趣的人，包括语音技术和移动计算专业的学生。我们假设您具有一般的编程背景，特别是在 Java 方面。我们还假设您对 Android 编程有一定的了解。

# 约定

在这本书中，您会发现多种文本样式，这些样式区分了不同类型的信息。以下是一些样式示例，及其含义的解释。

文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 处理程序会以下列形式显示："以下代码行创建了一个实现`onInitListener`接口的`onInit`方法的`TextToSpeech`对象。"

代码块设置如下：

```kt
TextToSpeech tts = new TextToSpeech(this, new OnInitListener(){ 
    public void onInit(int status){ 
       if (status == TextToSpeech.SUCCESS) 
             speak("Hello world", TextToSpeech.QUEUE_ADD, null); 
    }
}
```

当我们需要引起您对代码块中特定部分的注意时，相关的行或项目会以粗体显示：

```kt
Interpret field i:
       Play prompt of field i
       Listen for ASR result
       Process ASR result:
               If the recognition was successful, then save recognized
	       keyword as value for the field i and move to the next field
               If there was a no match or no input, then interpret field i
               If there is any other error, then stop interpreting
Move to the next field:
       If the next field has already a value assigned, then move to the next one
       If the last field in the form is reached,thenendOfDialogue=true
```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的词，例如菜单或对话框中的，会在文本中以这样的形式出现：**“请说出专辑名称的一个词”**。

### 注意

警告或重要注意事项会以这样的框显示。

### 提示

提示和技巧会像这样出现。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或可能不喜欢的地方。读者的反馈对我们来说非常重要，帮助我们开发出您真正能够充分利用的标题。

如果您想要给我们发送一般性反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在邮件的主题中提及书名。

如果您在某个主题上有专业知识，并且有兴趣撰写或参与书籍编写，请查看我们在[www.packtpub.com/authors](http://www.packtpub.com/authors)上的作者指南。

# 客户支持

既然您已经自豪地拥有了一本 Packt 图书，我们有许多方法可以帮助您充分利用您的购买。

## 下载示例代码

您可以从您的账户中下载您购买的所有 Packt 图书的示例代码，访问地址为[`www.packtpub.com`](http://www.packtpub.com)。如果您在别处购买了这本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)进行注册，我们会将文件通过电子邮件直接发送给您。

## 图书网页

本书有一个网页[`lsi.ugr.es/zoraida/androidspeechbook`](http://lsi.ugr.es/zoraida/androidspeechbook)，其中包含额外的资源，包括练习和项目的想法，进一步阅读的建议，以及有用的网页链接。

## 勘误

尽管我们已经竭尽全力确保内容的准确性，但错误仍然会发生。如果您在我们的书中发现了一个错误——可能是文本或代码中的错误——如果您能将其报告给我们，我们将不胜感激。这样做可以节省其他读者的时间，并帮助我们在后续版本中改进这本书。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情。一旦您的勘误被验证，您的提交将被接受，勘误将在我们网站的相应标题下的勘误部分上传或添加到现有的勘误列表中。任何现有的勘误都可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题来查看。

## 盗版

互联网上版权材料的盗版是一个所有媒体都面临的持续问题。在 Packt，我们非常重视保护我们的版权和许可。如果您在互联网上以任何形式遇到我们作品非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

如果您发现了疑似盗版材料，请通过`<copyright@packtpub.com>`联系我们，并提供相关链接。

我们感谢您保护我们的作者，以及我们为您带来有价值内容的能力。

## 问题

如果您在书的任何方面遇到问题，可以通过`<questions@packtpub.com>`联系我们，我们会尽力解决。
