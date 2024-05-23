# 第一章：开始使用 Corona SDK

> *在我们开始编写一些简单的游戏之前，我们需要安装并运行必要的程序，这些程序将使我们的应用程序变得生动。**Corona SDK**主要是一个 2D 开发引擎。如果你有 iOS 或 Android 开发的经验，你会发现使用 Corona 的经历令人耳目一新。它也非常易于使用。很快，你就能创建出可以在 Apple App Store 和 Google Play Store 发布的成品。*

在本章中，我们将：

+   在 Mac OS X 和 Windows 上设置 Corona SDK

+   为 Mac OS X 安装 Xcode

+   用两行代码创建一个 Hello World 程序

+   在 iOS Provisioning Portal 中添加设备

+   将应用程序加载到 iOS 设备上

+   将应用程序加载到 Android 设备上

# 下载并安装 Corona

你可以选择 Mac OS X 或 Microsoft Windows 操作系统进行开发。请记住运行此程序所需的以下系统要求。本书中最兼容的版本是 Build 2014.2511。

如果你是在 Mac OS X 上安装 Corona，请确保你的系统具备以下特性：

+   Mac OS X 10.9 或更高版本

+   运行 Lion、Mountain Lion、Mavericks 或 Yosemite 的基于 Intel 的系统

+   一个 64 位的 CPU（Core 2 Duo）

+   OpenGL 2.0 或更高版本的图形系统

如果你使用的是 Microsoft Windows，请确保你的系统具备以下特性：

+   Windows 8、Windows 7、Vista 或 XP（Service Pack 2）操作系统

+   1 GHZ 处理器（推荐）

+   80 MB 的磁盘空间（最低）

+   1 GB 的 RAM（最低）

+   需要 OpenGL 2.1 或更高版本的图形系统（大多数现代 Windows 系统都可用）

+   **Java 开发工具包**（**JDK**）的 32 位（x86）版本

+   使用 Corona 在 Mac 或 Windows 上创建 Android 设备构建时，不需要 Android SDK

# 动手操作——在 Mac OS X 上设置并激活 Corona

让我们从在桌面上设置 Corona SDK 开始：

1.  如果你还没有下载 SDK，请从[`www.coronalabs.com/downloads/coronasdk`](http://www.coronalabs.com/downloads/coronasdk)下载。在访问 SDK 之前，你需要注册成为用户。

1.  任何 Mac 程序的文件扩展名应以`.dmg`结尾；这被称为 Apple 磁盘映像。下载磁盘映像后，双击磁盘映像文件进行挂载。名称应类似于`CoronaSDK-XXXX.XXXX.dmg`。挂载后，你应该能看到如下截图所示的已挂载磁盘映像文件夹：![动手操作——在 Mac OS X 上设置并激活 Corona](img/9343OT_01_01.jpg)

1.  接下来，将`CoronaSDK`文件夹拖到`Applications`文件夹中。这将把 Corona 文件夹的内容复制到`/Applications`。如果你不是账户的主要管理员，系统会提示你输入管理员密码。成功安装后，你可以在`/Applications`中看到`CoronaSDK`文件夹。为了方便访问文件夹内容，你可以将`CoronaSDK`文件夹拖到 Mac 桌面的 dock 上创建别名：![行动时间——在 Mac OS X 上设置和激活 Corona](img/9343OT_01_02.jpg)

第一次使用 Corona SDK 的用户需要完成一次快速简单的一次性授权过程才能使用。你需要连接到互联网才能完成授权过程。

1.  在 SDK 文件夹中启动 Corona 模拟器。

1.  假设这是你第一次操作，系统会展示一个**最终用户许可协议**（**EULA**）。接受协议后，输入你用来注册 Corona 的电子邮件和密码以激活 SDK。否则，点击**注册**创建一个账户。

    ### 注意

    如果你以独立开发者的身份注册 Corona，那么在 iOS 和/或 Android 设备上进行开发是免费的。

    ![行动时间——在 Mac OS X 上设置和激活 Corona](img/9343OT_01_03.jpg)

1.  登录成功后，你会看到一个确认对话框，表明 SDK 已经可以使用：![行动时间——在 Mac OS X 上设置和激活 Corona](img/9343OT_01_04.jpg)

1.  点击**继续**按钮，你将看到欢迎来到 Corona 的屏幕：![行动时间——在 Mac OS X 上设置和激活 Corona](img/9343OT_01_05.jpg)

## *刚才发生了什么？*

在你的 Mac 操作系统上设置 Corona SDK 就像安装其他任何专门的 Mac 程序一样简单。在你在机器上授权 SDK 并用你的电子邮件和密码登录后，它就可以使用了。从现在开始，每次你启动 Corona，它都会自动登录到你的账户。当这种情况发生时，你会注意到屏幕上会出现 Corona SDK 的欢迎界面。

# 行动时间——在 Windows 上设置和激活 Corona

让我们按照以下步骤在桌面上安装 Corona SDK：

1.  从[`www.coronalabs.com/downloads/coronasdk`](http://www.coronalabs.com/downloads/coronasdk)下载 Corona SDK。在访问 SDK 之前，你需要注册成为用户。

1.  Corona 在 Windows 版本的文件扩展名应为`.msi`，这是微软制作的 Windows 安装程序的一部分，用于安装程序。双击该文件。文件名应该类似于`CoronaSDK.msi`。

1.  按照屏幕上的指示进行安装。

1.  Corona 默认会直接安装到您的`Programs`文件夹中。在 Microsoft Windows 上，您可以从开始菜单的程序列表中选择**Corona Simulator**，或者双击桌面上的 Corona 图标。成功激活后，您应该会看到以下屏幕：![行动时间 - 在 Windows 上设置和激活 Corona](img/9343OT_01_06.jpg)

1.  启动 Corona SDK 的激活过程应该与 Mac 上的操作相同，这是您第一次启动 Corona 时的步骤。

    ### 注意

    如果您遇到图像显示不正常的问题，请检查您是否使用的是最新的 OpenGL 图形驱动程序，至少是 2.1 版本。

    请注意，Windows 上的 Corona SDK 只能为 Android 设备构建，不能为 iOS 设备（iPhone、iPad 或 iPod Touch）构建。而 Mac 不仅可以为 iOS 构建，也可以为 Android 设备构建 Corona。

1.  要创建设备构建，您需要在 PC 上安装 Java 6 SDK。您需要访问 Oracle 网站 [`www.oracle.com/technetwork/java/javasebusiness/downloads/java-archive-downloads-javase6-419409.html`](http://www.oracle.com/technetwork/java/javasebusiness/downloads/java-archive-downloads-javase6-419409.html) 下载 JDK，并点击**Java SE Development Kit 6u45**链接。

1.  在下一页，选择**接受许可协议**的单选按钮，然后点击**Windows x86**链接下载安装程序。如果您还没有 Oracle 网站的用户账户，系统会要求您登录或创建一个。

1.  一旦下载了 JDK，请运行安装程序。安装完成后，您就可以在 PC 上为 Android 创建设备构建了。

## *刚才发生了什么？*

在 Windows 上安装 SDK 的过程与在 Mac OS X 上的设置过程不同。执行安装文件时，Windows 会自动提供一个指定的位置来安装应用程序，比如`Programs`文件夹，这样您就不必手动选择目的地。安装成功后，您会在桌面上看到 Corona SDK 的图标以便快速访问，或者在您首次访问时，它可能会在开始菜单的程序列表中突出显示。当您在计算机上授权 Corona 并使用您的登录信息登录后，它就可以使用了，并且每次启动时都会自动登录。

# 在 Mac 和 Windows 上使用模拟器

在 Mac OS X 上，可以通过选择`Applications`目录中的 Corona 终端或 Corona 模拟器来启动 Corona SDK。这两个选择都可以访问 SDK。Corona 模拟器只打开模拟器，而 Corona 终端会同时打开模拟器和终端窗口。终端有助于调试您的程序，并显示模拟器错误/警告和`print()`消息。

在 Microsoft Windows 上，选择 `Corona SDK` 文件夹，并从开始菜单中的程序列表中点击 **Corona Simulator**，或者双击桌面上的 Corona 图标。如果你使用的是 Windows，模拟器和终端将始终一起打开。

让我们回顾一下 `Corona SDK` 文件夹（在 Mac 上的 `Applications/Corona SDK`，在 Windows 上的 `Start/All Apps/Corona SDK`）中有用的内容：

+   **调试器（Mac）/Corona 调试器（Windows）**：这是一个工具，用于查找并隔离代码中的问题。

+   **Corona 模拟器**：这是用于启动你的应用程序进行测试的环境。它在你本地计算机上模拟你正在开发的移动设备。在 Windows 上，它将同时打开模拟器和终端。

+   **Corona 终端**：这会启动 Corona 模拟器并打开一个终端窗口，以显示错误/警告消息和 `print()` 语句。这对于调试代码非常有帮助，但仅在 Mac 上可用。

+   **模拟器**：这具有与 Corona 终端相同的属性，但它是从命令行调用的，并且仅在 Mac 上可用。

+   **示例代码**：这是一组示例应用程序，帮助你开始使用 Corona。它包含代码和艺术资源以供使用。

启动模拟器时，Corona SDK 窗口会自动打开。你可以在模拟器中打开 Corona 项目，创建设备构建以进行测试或分发，并查看一些示例游戏和应用，以便熟悉 SDK。

# 动手时间——在模拟器中查看示例项目

让我们在模拟器中看看 `HelloPhysics` 示例项目：

1.  点击 `Corona SDK` 文件夹中的 **Corona Simulator**。

1.  当 Corona SDK 窗口启动时，点击 **Samples** 链接。在出现的 **Open** 对话框中，导航到 `Applications/CoronaSDK/SampleCode/Physics/HelloPhysics`（Mac）或 `C:\Program Files (x86)\Corona Labs\Corona SDK\Sample Code\Physics\HelloPhysics`（Windows）。在 Mac 上，点击 **Open**，它将自动打开 `main.lua`。在 Windows 上，双击 `main.lua` 打开文件。`HelloPhysics` 应用程序将在模拟器中打开并运行。

## *刚才发生了什么？*

通过 Corona 终端或 Corona 模拟器访问 SDK 是个人偏好的问题。许多 Mac 用户更喜欢使用 Corona 终端，这样他们可以追踪输出到终端的消息。当你通过 Corona 模拟器启动 SDK 时，将显示模拟器，但不会显示终端窗口。当 Windows 用户启动 Corona 模拟器时，它将同时显示模拟器和终端窗口。当你想要尝试 Corona 提供的示例应用程序时，这种方式很方便。

`main.lua` 文件是一个特殊的文件名，它告诉 Corona 在项目文件夹中从哪里开始执行。这个文件还可以加载其他代码文件或程序资源，如声音或图形。

当你在 Corona 中启动`HelloPhysics`应用程序时，你会观察到模拟器中的盒子对象从屏幕顶部落下并与地面对象碰撞。从启动`main.lua`文件到在模拟器中查看结果的过程几乎是立即的。

## 动手试试——使用不同的设备壳。

当你开始熟悉 Corona 模拟器时，无论是在 Windows 还是 Mac OS X 中，启动应用程序时总是使用默认设备。Windows 使用 Droid 作为默认设备，而 Mac OS X 使用常规 iPhone。尝试在不同的设备壳中启动示例代码，以查看模拟器所有可用设备之间的屏幕分辨率差异。

当将构建版本移植到多个平台时，你需要考虑 iOS 和 Android 设备中各种屏幕分辨率。构建是你所有源代码的编译版本，转换成一个文件。让你的游戏构建适用于多个平台可以扩大你的应用程序受众。

# 选择文本编辑器

Corona 没有指定的程序编辑器用于编写代码，因此你需要找到一个符合你需求的编辑器。

对于 Mac OS，TextWrangler 是一个不错的选择，而且它是免费的！你可以在[`www.barebones.com/products/textwrangler/download.html`](http://www.barebones.com/products/textwrangler/download.html)下载它。其他文本编辑器如 BBEdit（[`www.barebones.com/thedeck`](http://www.barebones.com/thedeck)）和 TextMate（[`macromates.com/`](http://macromates.com/)）也非常好，但使用它们需要购买。TextMate 还兼容 Corona TextMate Bundle，可在[`www.ludicroussoftware.com/corona-textmate-bundle/index.html`](http://www.ludicroussoftware.com/corona-textmate-bundle/index.html)获取。

对于 Microsoft Windows，推荐使用 Notepad++，可以从[`notepad-plus-plus.org/`](http://notepad-plus-plus.org/)下载。

以下文本编辑器兼容 Mac OS 和 Microsoft Windows：

+   Sublime Text ([`www.sublimetext.com`](http://www.sublimetext.com))

+   Lua Glider ([`www.mydevelopersgames.com/Glider/`](http://www.mydevelopersgames.com/Glider/))

+   Outlaw ([`outlawgametools.com/outlaw-code-editor-and-project-manager/`](http://outlawgametools.com/outlaw-code-editor-and-project-manager/))

操作系统自带的任何文本编辑器，如 Mac 的 TextEdit 或 Windows 的 Notepad，都可以使用，但使用专为编程设计的编辑器会更容易。对于 Corona 来说，使用支持 Lua 语法高亮的编辑器在编码时会更加高效。语法高亮通过为关键字和标点添加格式化属性，使读者更容易区分代码与文本。

# 在设备上开发

如果你只想使用 Corona 模拟器，则无需下载 Apple 的开发工具包 Xcode 或 Android SDK。若要在 iOS 设备（iPhone、iPod Touch 和 iPad）上构建和测试你的代码，你需要注册成为 Apple 开发者并创建和下载配置文件。如果你想在 Android 上开发，除非你想使用 ADB 工具帮助安装构建版本和查看调试信息，否则不需要下载 Android SDK。

Corona SDK 入门版本允许你为 iOS 构建 Adhoc（测试版）和调试版本（Android），以便在你的设备上进行测试。Corona Pro 用户还能享受特殊功能，如访问每日构建版本、高级功能、所有插件和高级支持。

# 操作时间——下载和安装 Xcode

要开发任何 iOS 应用程序，你需要加入 Apple 开发者计划，这需要每年支付 99 美元，并在 Apple 网站 [`developer.apple.com/programs/ios/`](http://developer.apple.com/programs/ios/) 上按照以下步骤创建一个账户：

1.  点击**立即注册**按钮，并按照 Apple 的说明完成流程。在添加程序时，选择**iOS 开发者计划**。

1.  完成注册后，点击标记为**开发中心**的部分下的 iOS 链接。

1.  滚动到**下载**部分，下载当前的 Xcode，或者你也可以从 Mac App Store 下载 Xcode。

1.  完全下载 Xcode 后，从`/Applications/Xcode`目录中双击 Xcode。系统会要求你作为管理员用户进行身份验证：![操作时间——下载和安装 Xcode](img/9343OT_01_07.jpg)

1.  输入您的凭据后，点击**确定**按钮完成安装。你将看到以下屏幕：![操作时间——下载和安装 Xcode](img/9343OT_01_08.jpg)

1.  安装完 Xcode 开发者工具后，你可以通过启动 Xcode 并选择**帮助**菜单中的任何项目来访问文档。像 Xcode 和 Instruments 这样的开发者应用程序安装在`/Applications/Xcode`目录下。你可以将这些应用程序图标拖到你的 Dock 中以便快速访问。

## *刚才发生了什么？*

我们刚才走过了如何在 Mac OS X 上安装 Xcode 的步骤。通过加入 Apple 开发者计划，你可以在网站上访问最新的开发工具。记住，要继续作为 Apple 开发者，*你必须支付*每年 99 美元的费用以保持订阅。

Xcode 文件相当大，所以下载需要一些时间，具体取决于你的互联网连接速度。安装完成后，Xcode 就可以使用了。

# 操作时间——用两行代码创建一个 Hello World 应用程序

既然我们已经设置好了模拟器和文本编辑器，让我们开始制作我们的第一个 Corona 程序吧！我们将要制作的第一款程序叫做`Hello World`。这是一个传统程序，许多人在开始学习一门新的编程语言时都会学习它。

1.  打开你喜欢的文本编辑器，并输入以下几行：

    ```kt
    textObject = display.newText( "Hello World!", 160, 80, native.systemFont, 36 )
    textObject: setFillColor ( 1, 1, 1 )
    ```

1.  接下来，在桌面上创建一个名为`Hello World`的文件夹。将前面的文本保存为名为`main.lua`的文件，保存在你的项目文件夹位置。

1.  启动 Corona。你会看到 Corona SDK 的界面。点击**打开**，导航到你刚刚创建的`Hello World`文件夹。你应该在这个文件夹中看到你的`main.lua`文件：![动手时间——用两行代码创建一个 Hello World 应用](img/9343OT_01_09.jpg)

1.  在 Mac 上，点击**打开**按钮。在 Windows 上，选择`main.lua`文件并点击**打开**按钮。你会在 Corona 模拟器中看到你的新程序运行：![动手时间——用两行代码创建一个 Hello World 应用](img/9343OT_01_10.jpg)

### 提示

**下载示例代码**

你可以从你在 [`www.packtpub.com`](http://www.packtpub.com) 的账户下载你所购买的所有 Packt Publishing 图书的示例代码文件。如果你在其他地方购买了这本书，可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 注册，我们会直接将文件通过电子邮件发送给你。

# 动手时间——修改我们的应用程序

在我们深入更复杂的示例之前，通过执行以下步骤，让我们对程序进行一些小修改：

1.  让我们将`main.lua`的第二行更改为如下显示：

    ```kt
    textObject = display.newText( "Hello World!", 160, 80, native.systemFont, 36 )
    textObject:setFillColor( 0.9, 0.98 ,0 )
    ```

1.  保存你的文件，回到 Corona 模拟器。模拟器将检测文件中的更改并自动重新启动并应用更改。如果保存文件后模拟器没有自动重新启动，请按 *Command* + *R* (Mac) / *Ctrl* + *R* (Windows)。你将在屏幕上看到以下输出：![动手时间——修改我们的应用程序](img/9343OT_01_11.jpg)

### 注意

当你继续学习更多 Corona 函数时，你会注意到一些文本值是可选的。在这种情况下，我们需要使用五个值。

# 动手时间——将新字体名称应用到我们的应用程序

现在，通过执行以下步骤，让我们来玩转字体名称：

1.  将第一行更改为以下代码行：

    ```kt
    textObject = display.newText( "Hello World!", 160, 80, "Times New Roman", 36 )
    ```

1.  在对`main.lua`文件进行任何修改后，请确保保存文件；然后在 Corona 中按下 *Command* + *R* (Mac) / *Ctrl* + *R* (Windows) 重新启动模拟器以查看新字体。如果你使用的是 Mac，通常在保存文件后模拟器会自动重新启动，或者它可能会询问你是否想重新启动程序。你可以在模拟器中看到新字体：![动手时间——将新字体名称应用到我们的应用程序](img/9343OT_01_12.jpg)

## *刚才发生了什么？*

现在你已经制作了你的第一个完整的移动应用程序！更令人惊叹的是，这是一个完整的 iPhone、iPad 和 Android 应用程序。这个两行程序实际上可以安装并在你的 iOS/Android 设备上运行，如果你创建了一个构建。现在你已经了解了 Corona 中的基本工作流程。

如果你查看`main.lua`文件中的第 2 行，你会注意到`setFillColor`改变了**Hello World!**的文本颜色。

颜色由三组 RGB 数字组成，分别代表颜色中包含的红、绿、蓝的数量。它们用三个数字表示，数值范围从 0 到 1。例如，黑色为（0,0,0），蓝色为（0,0,1），白色为（0.6, 0.4, 0.8）。

继续尝试不同的颜色值，以查看不同的结果。当你保存`main.lua`文件并重新启动 Corona 时，你可以在模拟器中看到代码的更改。

当你查看`main.lua`文件的第一行时，你会注意到`newText()`是由显示对象调用的。返回的引用是`textObject`。`newText()`函数返回一个将在屏幕上表示文本的对象。`newText()`函数是显示库的一部分。

当你需要访问`newText`的显示属性时，输入`display.newText`。`Hello World!`后面的两个数字控制了文本在屏幕上的水平和垂直位置，单位为像素。接下来的项指定了字体。我们使用了`native.systemFont`这个名字，默认情况下，它指的是当前设备上的标准字体。例如，iPhone 的默认字体是 Helvetica。你可以使用任何标准字体名称，比如前面示例中使用的 Times New Roman。最后一个数字是字体大小。

## 尝试英雄——添加更多文本对象。

既然你现在开始对编程有了初步了解，请尝试在你的当前项目文件中按照以下步骤操作：

1.  创建一个新的显示对象，并使用不同的字体和文字颜色。确保它显示在`Hello World!`文字下方。同时，请确保你的新显示对象拥有一个不同的对象名称。

1.  继续改变当前显示对象`textObject`的值。更改*x*和*y*坐标、字符串文本、字体名称，甚至字体大小。

1.  虽然`object:setFillColor( r,g,b )`设置了文本的颜色，但你可以添加一个可选参数来控制文本的不透明度。尝试使用`object:setFillColor( r, g, b [, a] )`。`a`的值也在 0 到 1 之间（1 是不透明，这是默认值）。观察你的文本颜色的结果。

# 在 iOS 设备上测试我们的应用程序

如果你只想在 Android 设备上测试应用程序，可以跳过本章节的这部分内容，直接阅读*在 Android 设备上测试我们的应用程序*。在我们能够将第一个 Hello World 应用程序上传到 iOS 设备之前，我们需要登录我们的 Apple 开发者账户，这样我们才能在开发机上创建和安装签名证书。如果你还没有创建开发者账户，请访问[`developer.apple.com/programs/ios/`](http://developer.apple.com/programs/ios/)进行创建。记住，成为 Apple 开发者每年需要支付 99 美元的费用。

### 注意

Apple 开发者账户仅适用于在 Mac OS X 上开发的用户。确保你的 Xcode 版本与手机上的操作系统版本相同或更新。例如，如果你安装了 iPhone OS 版本 5.0，你将需要包含 iOS SDK 版本 5.0 或更高版本的 Xcode。

# 行动时间——获取 iOS 开发者证书

确保你已经注册了开发者计划；你将需要使用位于`/Applications/Utilities`的钥匙串访问工具，以便创建一个证书请求。在 Apple 设备上进行任何测试之前，必须使用有效的证书对所有的 iOS 应用程序进行签名。以下步骤将向你展示如何创建 iOS 开发者证书：

1.  前往**钥匙串访问** | **证书助手** | **从证书授权中心请求证书**:![行动时间——获取 iOS 开发者证书](img/9343OT_01_13.jpg)

1.  在**用户电子邮件地址**字段中，输入你注册 iOS 开发者时使用的电子邮件地址。在**常用名称**中，输入你的名字或团队名称。确保输入的名称与注册 iOS 开发者时提交的信息相匹配。**CA 电子邮件地址**字段无需填写，你可以将其留空。我们不将证书通过电子邮件发送给**证书授权中心**（**CA**）。勾选**保存到磁盘**和**让我指定密钥对信息**。点击**继续**后，系统会要求你选择一个保存位置。将你的文件保存到一个你能轻松找到的位置，比如桌面。![行动时间——获取 iOS 开发者证书](img/9343OT_01_14.jpg)

1.  在以下窗口中，确保已选择**2048 位**作为**密钥大小**，选择**RSA**作为**算法**，然后点击**继续**。这将生成密钥并将其保存到你指定的位置。在下一个窗口中点击**完成**。![行动时间——获取 iOS 开发者证书](img/9343OT_01_15.jpg)

1.  接下来，访问苹果开发者网站[`developer.apple.com/`](http://developer.apple.com/)，点击**iOS 开发者中心**，并登录到你的开发者账户。在屏幕右侧选择**iOS 开发者计划**下的**证书、标识符和配置文件**，然后导航到**iOS 应用**下的**证书**。点击页面右侧的**+**图标。在**开发**下，点击**iOS 应用开发**单选按钮。点击**继续**按钮，直到你到达生成证书的屏幕：![行动时间——获取 iOS 开发者证书](img/9343OT_01_16.jpg)

1.  点击**选择文件**按钮，找到你保存到桌面上的证书文件，然后点击**生成**按钮。

1.  点击**生成**后，你会收到来自钥匙串访问的 CA 请求表单中指定的电子邮件通知，或者你可以直接从开发者门户下载。创建证书的人将收到此电子邮件，并通过点击**批准**按钮来批准请求。![行动时间——获取 iOS 开发者证书](img/9343OT_01_17.jpg)

1.  点击**下载**按钮，并将证书保存到一个容易找到的位置。完成此操作后，双击该文件，证书将自动添加到钥匙串访问中。![行动时间——获取 iOS 开发者证书](img/9343OT_01_18.jpg)

## *刚才发生了什么？*

现在我们有了 iOS 设备的有效证书。iOS 开发证书仅用于开发目的，有效期为大约一年。密钥对由你的公钥和私钥组成。私钥允许 Xcode 为 iOS 应用程序签名。私钥只对密钥对创建者可用，并存储在创建者的机器的系统钥匙串中。

## 添加 iOS 设备

在 iPhone 开发者计划中，你可以分配最多 100 个设备用于开发和测试。要注册一个设备，你需要**唯一设备识别(UDID)**号码。你可以在 iTunes 和 Xcode 中找到这个信息。

### Xcode

要查找设备的 UDID，请将设备连接到 Mac 并打开 Xcode。在 Xcode 中，导航到菜单栏，选择**窗口**，然后点击**组织者**。**标识符**字段中的 40 个十六进制字符字符串就是你的设备 UDID。打开**组织者**窗口后，你应该能在左侧的**设备**列表中看到你的设备名称。点击它，并用鼠标选择标识符，复制到剪贴板。

![Xcode](img/9343OT_01_19.jpg)

通常，当你第一次将设备连接到**组织者**时，你会收到一个按钮通知，上面写着**用于开发**。选择它，Xcode 将为你的设备在 iOS 预配门户中完成大部分预配工作。

### iTunes

在连接设备的情况下，打开 iTunes 并点击设备列表中的您的设备。选择 **摘要** 选项卡。点击 **序列号** 标签以显示 **标识符** 字段和 40 个字符的 UDID。按 *Command* + *C* 将 UDID 复制到剪贴板。

![iTunes](img/9343OT_01_20.jpg)

# 行动时间 - 添加/注册您的 iOS 设备

要添加用于开发/测试的设备，请执行以下步骤：

1.  在开发者门户中选择 **设备** 并点击 **+** 图标以注册新设备。选择 **注册设备** 单选按钮以注册一个设备。

1.  在 **名称** 字段中为您的设备创建一个名称，并通过按 *Command* + *V* 将您保存在剪贴板上的设备 UDID 粘贴到 **UDID** 字段中。

1.  完成后点击 **继续** 并在验证设备信息后点击 **注册**。![行动时间 - 添加/注册您的 iOS 设备](img/9343OT_01_21.jpg)

# 行动时间 - 创建一个 App ID

现在，您已经在门户中添加了一个设备，接下来需要创建一个 App ID。App ID 具有一个由 Apple 生成的唯一 10 个字符的 Apple ID 前缀和一个由配置门户中的团队管理员创建的 Apple ID 后缀。App ID 可能如下所示：`7R456G1254.com.companyname.YourApplication`。要创建新的 App ID，请使用以下步骤：

1.  在门户的 **标识符** 部分点击 **App IDs** 并选择 **+** 图标。![行动时间 - 创建一个 App ID](img/9343OT_01_22.jpg)

1.  在 **App ID** **描述** 字段中填写您的应用程序名称。

1.  您已经分配了一个 Apple ID 前缀（也称为团队 ID）。

1.  在 **App ID 后缀** 字段中，为您的应用指定一个唯一标识符。您可以根据自己的意愿来标识应用，但建议您使用反向域名风格的字符串，即 `com.domainname.appname`。点击 **继续** 然后点击 **提交** 以创建您的 App ID。

### 注意

您可以在捆绑标识符中创建一个通配符字符，以便在共享同一钥匙串访问的一组应用程序之间共享。为此，只需创建一个带有星号 (*) 结尾的单个 App ID。您可以将此字符单独放在捆绑标识符字段中，或者作为字符串的结尾，例如，`com.domainname.*`。关于此主题的更多信息可以在 iOS 配置门户的 App IDs 部分找到，链接为[`developer.apple.com/ios/manage/bundles/howto.action`](https://developer.apple.com/ios/manage/bundles/howto.action)。

## *刚才发生了什么？*

所有设备的 UDID 都是唯一的，我们可以在 Xcode 和 iTunes 中找到它们。当我们在 iOS 配置门户中添加设备时，我们获取了由 40 个十六进制字符组成的 UDID，并确保我们创建了一个设备名称，以便我们可以识别用于开发的设备。

现在我们有了想要安装在设备上的应用程序的 App ID。App ID 是 iOS 用来允许您的应用程序连接到 Apple Push Notification 服务、在应用程序之间共享钥匙串数据以及与您希望与 iOS 应用程序配对的外部硬件配件进行通信的唯一标识符。

## 配置文件

**配置文件**是一组数字实体，它将应用程序和设备独特地绑定到一个授权的 iOS 开发团队，并使设备能够用来测试特定的应用程序。配置文件定义了应用程序、设备和开发团队之间的关系。它们需要为应用程序的开发和分发方面进行定义。

# 行动时间 - 创建配置文件

要创建配置文件，请访问开发者门户的**配置文件**部分，并点击**+**图标。执行以下步骤：

1.  在**开发**部分下选择**iOS App Development**单选按钮，然后选择**继续**。

1.  在下拉菜单中选择您为应用程序创建的**App ID**，然后点击**继续**。

1.  选择您希望包含在配置文件中的证书，然后点击**继续**。

1.  选择您希望授权此配置文件的设备，然后点击**继续**。

1.  创建一个**配置文件名称**，完成后点击**生成**按钮：![行动时间 - 创建配置文件](img/9343OT_01_23.jpg)

1.  点击**下载**按钮。在文件下载时，如果 Xcode 尚未打开，请启动 Xcode，并在键盘上按*Shift* + *Command* + *2*打开**组织者**。

1.  在**库**下，选择**配置文件**部分。将您下载的`.mobileprovision`文件拖到**组织者**窗口中。这将自动将您的`.mobileprovision`文件复制到正确的目录中。![行动时间 - 创建配置文件](img/9343OT_01_24.jpg)

## *刚才发生了什么？*

在配置文件中有权限的设备只要证书包含在配置文件中，就可以用于测试。一个设备可以安装多个配置文件。

## 应用程序图标

当前，我们的应用程序在设备上没有图标图像显示。默认情况下，如果应用程序没有设置图标图像，一旦构建被加载到您的设备上，您将看到一个浅灰色框以及下面的应用程序名称。因此，启动您喜欢的创意开发工具，让我们创建一个简单的图像。

标准分辨率 iPad2 或 iPad mini 的应用程序图标图像文件为 76 x 76 px PNG。图像应始终保存为`Icon.png`，并且必须位于您当前的项目文件夹中。支持视网膜显示的 iPhone/iPod touch 设备需要一个额外的 120 x 120 px 高分辨率图标，而 iPad 或 iPad mini 需要一个 152 x 152 px 的图标，命名为`Icon@2x.png`。

您当前项目文件夹的内容应如下所示：

```kt
Hello World/       name of your project folder
 Icon.png           required for iPhone/iPod/iPad
 Icon@2x.png   required for iPhone/iPod with Retina display
 main.lua

```

为了分发你的应用，App Store 需要一张 1024 x 1024 像素的应用图标。最好先以更高分辨率创建你的图标。参考*Apple iOS Human Interface Guidelines*获取最新的官方 App Store 要求，访问[`developer.apple.com/library/ios/#documentation/userexperience/conceptual/mobilehig/Introduction/Introduction.html`](http://developer.apple.com/library/ios/#documentation/userexperience/conceptual/mobilehig/Introduction/Introduction.html)。

创建应用程序图标是你应用程序名称的视觉表示。一旦你编译了构建，你将能在设备上查看该图标。该图标也是启动你应用程序的图像。

# 为 iOS 创建 Hello World 版本构建

现在我们准备为我们的设备构建 Hello World 应用程序。由于我们已经有了配置文件，从现在开始的构建过程非常简单。在创建设备版本之前，请确保你已连接到互联网。你可以为 Xcode 模拟器或设备测试你的应用。

# 动手操作时间——创建 iOS 版本

按以下步骤在 Corona SDK 中创建一个新的 iOS 版本：

1.  打开 Corona 模拟器并选择**打开**。

1.  导航至你的 Hello World 应用程序，并选择你的`main.lua`文件。

1.  当应用程序在模拟器中启动后，请导航至 Corona 模拟器的菜单栏，选择**文件** | **构建** | **iOS**，或者在你的键盘上按*Command* + *B*。以下对话框将会出现：![动手操作时间——创建 iOS 版本](img/9343OT_01_25.jpg)

1.  在**应用名称**字段中为你的应用创建一个名称。我们可以保持名称为`Hello World`。在**版本**字段中，保持数字为`1.0`。为了在 Xcode 模拟器中测试应用，从**构建为**下拉菜单中选择**Xcode Simulator**。如果你想为设备构建，选择**Device**以构建应用包。接下来，从**支持设备**下拉菜单中选择目标设备（iPhone 或 iPad）。从**代码签名标识**下拉菜单中选择你为特定设备创建的配置文件。它与 Apple 开发者网站上 iOS Provisioning Portal 中的**Profile Name**相同。在**保存到文件夹**部分，点击**浏览**并选择你希望保存应用的位置。

    如果对话框中的所有信息都已确认，请点击**构建**按钮。

### 提示

将你的应用程序设置为保存到桌面会更加方便；这样容易找到。

## *刚才发生了什么？*

恭喜你！现在你已经创建了可以上传到设备的第一款 iOS 应用程序文件。随着你开始开发用于分发的应用程序，你将希望创建应用程序的新版本，以便跟踪每次新构建所做的更改。你的所有信息从你的供应配置文件在 iOS 供应门户中创建，并应用于构建。Corona 编译完构建后，应用程序应该位于你保存它的文件夹中。

# 行动时间 – 在你的 iOS 设备上加载应用程序

选择你创建的 Hello World 构建版本，并选择 iTunes 或 Xcode 将你的应用程序加载到 iOS 设备上。它们可以用来传输应用程序文件。

如果使用 iTunes，将你的构建版本拖到 iTunes 资料库中，然后像以下屏幕截图所示正常同步你的设备：

![行动时间 – 在你的 iOS 设备上加载应用程序](img/9343OT_01_26.jpg)

将应用程序安装到设备上的另一种方式是使用 Xcode，因为它提供了一种方便的方法来安装 iOS 设备应用程序。执行以下步骤：

1.  连接设备后，通过菜单栏的 **窗口** | **Organizer** 打开 Xcode 的 **Organizer**，然后在左侧的 **设备** 列表中找到你的连接设备。

1.  如果建立了正确的连接，你会看到一个绿色指示灯。如果几分钟之后变成黄色，尝试关闭设备然后再打开，或者断开设备连接并重新连接。这通常可以建立正确的连接。![行动时间 – 在你的 iOS 设备上加载应用程序](img/9343OT_01_27.jpg)

1.  只需将你的构建文件拖放到 **Organizer** 窗口的 **应用程序** 区域，它就会自动安装到你的设备上。

## *刚才发生了什么？*

我们刚刚学习了两种将应用程序构建加载到 iOS 设备上的不同方法：使用 iTunes 和使用 Xcode。

使用 iTunes 可以轻松实现拖放功能到你的资料库，并且只要设备同步，就可以传输构建的内容。

Xcode 方法可能是将构建加载到设备上最简单且最常见的方式。只要你的设备在 Organizer 中正确连接并准备好使用，你只需将构建拖放到应用程序中，它就会自动加载。

# 在 Android 设备上测试我们的应用程序

在 Android 设备上创建和测试构建版本不需要像苹果为 iOS 设备那样需要开发者账户。为 Android 构建所需的唯一工具是 PC 或 Mac、Corona SDK、安装的 JDK6 和一个 Android 设备。如果你打算将应用程序提交到 Google Play 商店，你需要注册成为 Google Play 开发者，注册地址为 [`play.google.com/apps/publish/signup/`](https://play.google.com/apps/publish/signup/)。如果你想在 Google Play 商店上发布软件，需要支付一次性的 25 美元注册费。

# 为 Android 创建 Hello World 构建版本

构建我们的 Hello World 应用程序相当简单，因为我们不需要为调试构建创建唯一的密钥库或密钥别名。当你准备将应用程序提交到 Google Play 商店时，你需要创建一个发布构建并生成自己的私钥来签名你的应用。我们将在本书的后面详细讨论发布构建和私钥。

# 动手操作——创建一个 Android 构建

按照以下步骤在 Corona SDK 中创建一个新的 Android 构建：

1.  启动 Corona 模拟器，并选择**模拟器**。

1.  导航到你的 Hello World 应用程序，并选择你的`main.lua`文件。

1.  当你的应用程序在模拟器中运行时，转到**Corona Simulator**菜单栏，然后导航至**文件** | **构建为** | **Android**（Windows）/在键盘上按*Shift* + *Command* + *B*（Mac）。以下对话框将出现：![动手操作——创建一个 Android 构建](img/9343OT_01_31.jpg)

1.  在**应用程序名称**字段中为你的应用创建一个名称。我们可以保持相同的名称，即**Hello World**。在**版本代码**字段中，如果默认数字不是 1，则将其设置为**1**。这个特定的字段必须始终是一个整数，并且对用户不可见。在**版本名称**字段中，保持数字为**1.0**。这个属性是显示给用户的字符串。在**包**字段中，你需要指定一个使用传统 Java 方案的名称，这基本上是你的域名反转格式；例如，**com.mycompany.app.helloworld**可以作为包名称。**项目路径**显示你的项目文件夹的位置。**最低 SDK 版本**目前支持运行在 ArmV7 处理器上的 Android 2.3.3 及更新设备。在**目标应用商店**下拉菜单中，默认商店可以保持为 Google Play。在**密钥库**字段中，你将使用 Corona 提供的`Debug`密钥库来签名你的构建。在**密钥别名**字段中，如果尚未选择，请从下拉菜单中选择`androiddebugkey`。在**保存到文件夹**部分，点击**浏览**并选择你希望保存应用程序的位置。

1.  如果对话框中的所有信息都已确认，请点击**构建**按钮。

### 提示

有关 Java 包名称的更多信息，请参阅 Java 文档中关于*唯一包名称*的部分，链接为：[`java.sun.com/docs/books/jls/third_edition/html/packages.html#40169`](http://java.sun.com/docs/books/jls/third_edition/html/packages.html#40169)。

## *刚才发生了什么？*

你已经创建了你的第一个 Android 构建！看看这有多简单？由于 Corona SDK 已经在引擎中提供了`Debug`密钥库和`androiddebugkey`密钥别名，所以大部分签名工作已经为你完成。你唯一需要做的是填写应用程序的构建信息，然后点击**构建**按钮来创建一个调试构建。你的 Hello World 应用程序将保存为`.apk`文件，保存在你指定的位置。文件名将显示为`Hello World.apk`。

# 动手时间——在 Android 设备上加载应用程序

有多种方法可以将你的 Hello World 构建加载到 Android 设备上，这些方法并不需要你下载 Android SDK。以下是一些简单的方法。

一个方便的方法是通过 Dropbox。你可以在[`www.dropbox.com/`](https://www.dropbox.com/)创建一个账户。Dropbox 是一个免费服务，允许你在 PC/Mac 和移动设备上上传/下载文件。以下是通过 Dropbox 加载 Hello World 构建的步骤：

1.  下载 Dropbox 安装程序，并在你的电脑上安装它。同时，在设备上从 Google Play 商店（也是免费的）下载移动应用并安装。

1.  在你的电脑和移动设备上登录你的 Dropbox 账户。从电脑上，上传你的`Hello World.apk`文件。

1.  上传完成后，在设备上打开 Dropbox 应用，选择你的`Hello World.apk`文件。你将看到一个询问你是否要安装该应用程序的屏幕。选择**安装**按钮。假设它安装正确，另一个屏幕将出现，显示**应用程序已安装**，你可以通过按**打开**按钮来启动你的 Hello World 应用。

将`.apk`文件上传到设备上的另一种方法是，通过 USB 接口将其传输到 SD 卡。如果你的设备没有配备某种文件管理应用程序，你可以从 Google Play 商店下载一个很好的 ASTRO 文件管理器，它的下载地址是[`play.google.com/store/apps/details?id=com.metago.astro`](https://play.google.com/store/apps/details?id=com.metago.astro)。你总是可以通过设备上的 Google Play 应用正常搜索前面提到的应用程序或类似的 apk 安装器。要将`.apk`文件传输到 SD 卡，请执行以下步骤：

1.  在设备的**设置**中，选择**应用程序**，然后选择**开发**。如果 USB 调试模式未激活，请点击**USB 调试**。

1.  回到前几屏的**应用程序**部分。如果**未知来源**尚未激活，请启用它。这将允许你安装任何非市场应用程序（即调试版本）。设置完毕后，选择设备上的主页按钮。

1.  使用 USB 电缆将设备连接到电脑。你会看到一个新通知，一个新的驱动器已经连接到你的 PC 或 Mac。访问 SD 驱动器并创建一个新文件夹。给你的 Android 构建容易识别的名字。将 `Hello World.apk` 文件从桌面拖放到文件夹中。

1.  从桌面上弹出驱动器并断开设备与 USB 电缆的连接。启动 ASTRO 文件管理器或使用你在 Google Play 商店下载的任何应用。在 ASTRO 中，选择**文件管理器**，找到你添加到 SD 卡的文件夹并选择它。你会看到你的 `Hello World.apk` 文件。选择该文件，会出现一个提示询问你是否安装它。选择**安装**按钮，你应该会在设备的**应用**文件夹中看到你的 Hello World 应用程序。![行动时间——在 Android 设备上加载应用程序](img/9343OT_01_32.jpg)

最简单的方法之一是通过 Gmail。如果你还没有 Gmail 账户，可以在[`mail.google.com/`](https://mail.google.com/)创建一个。按照以下步骤在 Gmail 账户上发送 `.apk` 文件：

1.  登录你的账户，撰写一封新邮件，并将你的 `Hello World.apk` 文件作为附件添加到消息中。

1.  将消息的收件人地址设为你自己的电子邮件地址并发送。

1.  在你的 Android 设备上，确保你的电子邮件账户已经关联。一收到消息，就打开邮件。你将有机会在设备上安装该应用程序。将会有一个**安装**按钮或类似的显示。

## *刚才发生了什么？*

我们刚刚学习了几种将 `.apk` 文件加载到 Android 设备上的方法。前面的方法是最简单的方式之一，可以快速加载应用程序而不会遇到任何问题。

使用文件管理器方法可以轻松访问你的 `.apk` 文件，无需任何运营商数据或 Wi-Fi 连接。使用与你的设备兼容的 USB 电缆并将其连接到电脑是一个简单的拖放过程。

一旦在电脑和移动设备上设置好，Dropbox 方法是最方便的。你需要做的就是将 `.apk` 文件拖放到你的账户文件夹中，任何安装了 Dropbox 应用程序的设备都可以立即访问。你也可以通过下载链接分享你的文件，这也是 Dropbox 提供的另一个很棒的功能。

如果你不想在设备和电脑上下载任何文件管理器或其他程序，设置一个 Gmail 账户并将你的 `.apk` 文件作为附件发送给自己很简单。唯一要记住的是，你不能在 Gmail 中发送超过 25 MB 的附件。

## 小测验——了解 Corona

Q1. 关于使用 Corona 模拟器，以下哪个是正确的？

1.  你需要一个 `main.lua` 文件来启动你的应用程序。

1.  Corona SDK 只能在 Mac OS X 上运行。

1.  Corona 终端不会启动模拟器。

1.  以上都不是。

Q2. 在 iPhone 开发者计划中，你可以使用多少个 iOS 设备进行开发？

1.  `50`。

1.  `75`。

1.  `5`。

1.  `100`。

Q3. 在 Corona SDK 中为 Android 构建时，版本代码必须是什么？

1.  一个字符串。

1.  一个整数。

1.  它必须遵循 Java 方案格式。

1.  以上都不是。

# 概要

在本章中，我们介绍了一些开始使用 Corona SDK 开发应用程序所必需的工具。无论你是在 Mac OS X 还是 Microsoft Windows 上工作，你都会注意到在两个操作系统上工作的相似性，以及运行 Corona SDK 是多么简单。

为了进一步熟悉 Corona，尝试执行以下操作：

+   花时间查看 Corona 提供的示例代码，以了解 SDK 的功能。

+   请随意修改任何示例代码，以更好地理解 Lua 编程。

+   无论你是在 iOS（如果你是注册的 Apple 开发者）还是 Android 上工作，尝试在你的设备上安装任何示例代码，看看应用程序在模拟器环境之外是如何工作的。

+   访问 Corona 实验室论坛 [`forums.coronalabs.com/`](http://forums.coronalabs.com/)，浏览一下 Corona SDK 开发者和工作人员关于 Corona 开发的最新讨论。

既然你已经了解了如何在 Corona 中显示对象的过程，我们将能够深入探讨其他有助于创建可操作的移动游戏的函数。

在下一章中，我们将进一步了解 Lua 编程语言的细节，你将学习到类似于 Corona 示例代码的简单编程技巧。你将对 Lua 语法有更深入的理解，并注意到与其他编程语言相比，它是多么快速和容易学习。那么，让我们开始吧！
