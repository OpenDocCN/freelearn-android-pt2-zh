# 第七章：其他 Robotium 实用工具

本章包括 Robotium 框架中的各种实用工具。这些实用工具包括 `RobotiumUtils` 类、XPath 的使用、Robotium 对已安装的 Android 应用程序的用途，以及在应用程序签名和解签名操作期间涉及的签名过程，以执行测试。让我们逐一了解本章中的这些实用工具。

# `RobotiumUtils` 类

`RobotiumUtils` 类在 Robotium v4.2 中引入。它包含实用方法。让我们看看 `RobotiumUtils` 类的 API 集合。

![`RobotiumUtils` 类](img/8010OS_07_01.jpg)

## API 集合

`RobotiumUtils` 类提供了用于排序、订购、过滤视图等方法。提供这些功能的不同方法如下：

+   下面的方法基于作为参数传递的类过滤视图：

    ```kt
    RobotiumUtils.filterViews(Class classToFilterBy, Iterable viewList)
    ```

+   下面的方法过滤掉所有不在 `classSet` 参数中的视图：

    ```kt
    RobotiumUtils.filterViewsToSet(Class<android.view.View>[] classSet, Iterable<android.view.View> viewList)
    ```

+   下面的方法检查视图是否与 `stringToMatch` 变量匹配，并返回找到的匹配数量：

    ```kt
    RobotiumUtils.getNumberOfMatches(String stringToMatch, TextView targetTextView, Set<TextView> textviewList)
    ```

+   下面的方法移除了所有可见性为不可见的视图：

    ```kt
    RobotiumUtils.removeInvisibleViews(Iterable viewList)
    ```

+   下面的方法根据屏幕上的位置对所有视图进行排序：

    ```kt
    RobotiumUtils.sortViewsByLocationOnScreen(List viewList)
    ```

+   下面的方法过滤视图集合，并返回与指定正则表达式匹配的视图。

    ```kt
    RobotiumUtils.filterViewsByText(Iterable viewList, string regex)
    ```

### 注意

你可以通过以下链接获取这些方法的更多信息：

[`robotium.googlecode.com/svn/doc/com/jayway/android/robotium/solo/RobotiumUtils.html`](http://robotium.googlecode.com/svn/doc/com/jayway/android/robotium/solo/RobotiumUtils.html)

# XPath API 及语法

可以使用类似 XPath 的表达式定位网页元素。这帮助测试人员在保持测试对应用程序 UI 的抵抗力的同时找到 DOM 的元素。请注意，这仅适用于 WebView 应用程序内的网页元素。

![XPath API 及语法](img/8010OS_07_02.jpg)

XPath 主要用于在 XML 文档中导航属性和元素。可以使用简单的路径表达式通过 XPath 导航 XML 文档。

让我们通过一个简单的例子来了解如何使用 XPath：

以下是我们必须使用 XPath 解析的 XML 文件：

```kt
<?xml version="1.0"?>
<catalog>
  <book id="z24">
  <author>Hrushikesh</author>
  <title>Robotium Automated Testing For Android</title>
  <genre>Computer</genre>	
  <description>Efficiently automate test cases for Android applications using Robotium </description>
  </book>

  <book id="az24">
  <author>Bharati</author>
  <title>Mathematics MHTCET</title>
  <genre>Objective</genre>
  <description>A comprehensive guide to Mathematics for MHTCET</description>
  </book>
</catalog>
```

假设你想从上一个 XML 文件中提取作者的名字。使用 XPath 完成此操作的功能如下：

```kt
private void parseDataUsingXPath() {
  // create an InputSource object from /res/xml
  InputSource inputSrc = new InputSource(getResources().openRawResource(R.xml.data));

  // Instantiate the XPath Parser instance
  XPath xPath = XPathFactory.newInstance().newXPath();

  // Create the xpath expression
  String xPathExpression = "//author";

  // list of nodes queried
  NodeList nodes = (NodeList)xpath.evaluate(expression, inputSrc, XPathConstants.NODESET);

  // if author node found, then add to authorList i.e. the list of authors instantiated in the activity where this function is used
  if(nodes != null && nodes.getLength() > 0) {
    int len = nodes.getLength();
    for(int i = 0; i < len; ++i) {
      Node node = nodes.item(i);
      authorList.add(node.getTextContent());
    }
  }
}
```

你可以使用以下 XPath 表达式示例查找网页元素：

```kt
solo.getWebElement(By.xpath("//*[@id='nameOfelementID']/tbody/tr[11]/td[2]/input[1]"), 0);
where By.xpath(<path expression>, index);
```

# Robotium 用于预装应用程序

Robotium 允许编写和运行针对 Android 上预装应用程序的测试用例。为此，你需要使用测试项目的调试密钥对目标应用程序执行重新签名操作。这需要访问设备上的 `/system/app` 文件夹。要访问此文件夹，你必须有一个已获得根权限的设备。

![Robotium 用于预装应用程序](img/8010OS_07_03.jpg)

### 注意

请注意，一些经过重新签名操作预装的应用程序可能无法正常工作。这些应用在用新的调试密钥重新签名后甚至不会显示。

要重新签名 APK 文件，请按照以下步骤操作：

1.  以 root 用户登录：`adb root`。

1.  使用以下代码重新挂载：

    ```kt
    adb remount
    adb pull /system/app/ApplicationName.apk 
    ```

1.  重新签名 `ApplicationName.apk` 文件，使其具有与 `adb pull /data/system/packages.xml` 测试项目相同的证书调试密钥签名。

1.  打开 `packages.xml` 并删除以下代码：

    ```kt
    <package name="com.ApplicationName">
    .....
    </package>
    ```

1.  将 `packages.xml` 推回设备 `adb push packages.xml /data/system`。

1.  重启你的设备。

1.  将重新签名的 `ApplicationName.apk` 文件推回设备 `adb push ApplicationName.apk /system/app`。

## 仅测试 APK

要测试的 APK 文件应具有与测试项目相同的证书签名。如果签名不同，将会发生不匹配。

![仅测试 APK](img/8010OS_07_04.jpg)

+   如果你知道 APK 证书签名，请使用相同的签名对测试项目进行签名。

+   如果 APK 尚未签名，请使用 Android 调试密钥签名 APK。

+   如果你不知道 APK 证书签名，删除其证书签名，并为 APK 和你的测试项目提供调试密钥签名。

## 签名过程

在 Android 操作系统中，所有应用程序都使用持有私钥的证书进行签名，该私钥由应用程序开发者持有。

有两种构建模式可以构建你的 Android 应用程序。它们如下：

+   调试模式

+   发布模式

Android 构建过程会根据构建模式类型对应用程序进行签名。在开发测试应用程序时通常使用调试模式，而发布模式则用于将发布版本的应用程序分发给用户，或者在如 **Google Play Store** 的市场中发布应用程序。

![签名过程](img/8010OS_07_05.jpg)

当你使用构建模式时，`Keytool` 实用工具用于创建调试密钥。调试密钥的别名和密码为 SDK 构建工具所知。这就是为什么工具在每次编译程序时不会提示你输入调试密钥的别名和密码。

当你在发布模式下构建应用程序时，会使用私钥。如果你没有私钥，`Keytool` 实用工具会为你创建一个。当你的应用程序在发布模式下编译时，构建工具会使用你的私钥以及 `jarsigner` 实用工具来签名应用程序 APK 文件。

当你使用安装了 ADT 插件的 Eclipse 调试或运行应用程序时，调试密钥签名过程会在后台自动进行。使用启用了调试选项的 Ant 脚本构建应用程序时，也会遵循相同的签名过程。《Eclipse 导出向导》可以帮助自动化发布签名过程或修改 Ant 构建脚本，以及使用发布选项构建应用程序。

要取消已签名的 APK 的签名，然后使用调试密钥重新签名，请执行以下步骤：

1.  使用任何 ZIP 解压工具解压 APK 文件。

1.  删除`Meta-INF`文件夹。

1.  将提取的文件重新压缩成 APK 文件，并将扩展名从`appname.apk.zip`改为`appname.apk`。这样，你就可以取消已签名的 APK 的签名！

1.  要使用 Android 调试签名密钥为这个 APK 签名，请在命令提示符中运行以下命令：

    ```kt
    > jarsigner -keystore ~/.android/debug.keystore -storepass android -keypass android appname.apk androiddebugkey	

    > zipalign 4 appname.apk tempappname.apk
    ```

`jarsigner`工具可以在 JDK 二进制文件中找到，而*zipalign*是 Android SDK 的一部分。

*zipalign*工具用于优化 Android APK 文件。其基本目的是使未压缩的数据与文件的开始位置相对齐。

借助 Eclipse，你也可以为你的 Android 应用签名并导出。执行以下步骤：

1.  在 Android 项目上右键点击，导航到**Android Tools** | **导出已签名的应用程序包**。

1.  在**导出**Android 应用程序向导中，选择你想要导出的项目，然后点击**下一步**。

1.  将会出现**密钥库选择**屏幕。

1.  如果你没有现有的密钥库，请创建一个新的密钥库。你应该可以通过输入位置和密码来创建一个密钥库。

1.  在导航到你想使用的文件夹后，在文件浏览窗口中的**文件名：**字段里输入一个名字，例如，`hrushikesh.keystore`。然后，你应该可以继续进行密钥的创建。

有关 APK 签名的更多信息，请参考以下链接：

[APK 签名指南](http://developer.android.com/tools/publishing/app-signing.html)

# 概述

在本章中，你了解到了 Robotium Framework 中不同的工具及其使用方法。在下一章中，我们将看到 Robotium 与 Maven 的集成机制，以及一些示例。
