# 附录 A.开发环境

为了构建 UDOO 提供的 Android 4.3 源码，你需要一个带有 Oracle Java 6 的 Ubuntu Linux 系统。虽然可能可以使用这种设置的变体，但 Google 为 Android 4.3 的标准目标开发平台是 Ubuntu 12.04。因此，我们将使用此设置以确保在探索 Linux、SE Linux、Android、UDOO 和 SE for Android 时获得最高的成功率。

在本附录中，我们将执行以下操作：

+   使用**虚拟机**（**VM**）下载并安装 Ubuntu 12.04

+   通过安装 VirtualBox 扩展包和 VirtualBox 客户机增强功能来提高我们的 VM 性能

+   搭建适合构建 Linux 内核和 UDOO 源的开发环境

+   安装 Oracle Java 6

### 提示

如果你已经使用 Ubuntu Linux 12.04，可以跳转到*构建环境*部分。如果你打算直接安装 Ubuntu（不在虚拟机中），应该跳转到*Ubuntu Linux 12.04*部分，并按照那些说明操作，忽略 VirtualBox 的步骤。

# VirtualBox

有许多虚拟化产品可用于运行客户操作系统，如 Ubuntu Linux，但在此设置中我们将使用**VirtualBox**。VirtualBox 是一个广泛使用的开源虚拟化系统，适用于 Mac、Linux、Solaris 和 Windows 主机（及其他）。它支持各种客户操作系统。VirtualBox 还允许使用许多现代/常见处理器家族的硬件虚拟化来提高性能，为每个虚拟机提供其自己的私有地址空间。

VirtualBox 的文档为各种平台提供了出色的安装说明，我们建议你为自己的主机平台参考这些说明。你可以在[`www.virtualbox.org/manual/ch02.html`](http://www.virtualbox.org/manual/ch02.html)找到有关为你的主机操作系统安装和运行 VirtualBox 的信息。

# Ubuntu Linux 12.04（精确的穿山甲）

要安装 Ubuntu Linux 12.04，首先需要下载合适的发行版镜像。可以在[`releases.ubuntu.com/12.04/`](http://releases.ubuntu.com/12.04/)找到这些镜像。那里有许多可接受的镜像，我们将安装 64 位桌面版的发行版——[`releases.ubuntu.com/12.04/ubuntu-12.04.5-desktop-amd64.iso`](http://releases.ubuntu.com/12.04/ubuntu-12.04.5-desktop-amd64.iso)。本例中使用的主机是一台运行 OS X 10.9.2 的 64 位 Macbook Pro，因此我们也目标是 64 位的客户机。如果你使用的是 32 位机器，我们涵盖的基本操作将相同；只有一些细节会有所不同，这部分留给你自行探索和解决。

在你的主机上启动 VirtualBox，等待**VM 管理器**窗口出现，并执行以下步骤：

1.  点击**新建**。

1.  对于**名称**和**操作系统**设置，做出以下选择：

    +   **名称**：**SE for Android Book**

    +   **类型**：**Linux**

    +   **版本**：**Ubuntu (64 位)**

1.  将**内存大小**设置为至少`16` GB。低于这个值将导致构建失败。

1.  要设置硬盘，选择**立即创建虚拟硬盘**。将此值设置为至少`80` GB。

1.  选择**硬盘文件类型**，**VDI (VirtualBox 磁盘映像)**。

1.  确保物理硬盘上的存储设置为**动态分配**。

1.  当提示输入**文件位置和大小**时，将新的虚拟硬盘命名为**SE for Android Book**，并将其大小设置为 80 GB。

确保在左侧窗格中选择了**SE for Android Book** VM。点击绿色启动箭头以首次启动 VM。将出现一个对话框，要求你选择一个虚拟光驱文件。点击小文件夹图标，找到你之前下载的`ubuntu-12.04.5-desktop-amd64.iso` CD 映像。然后点击**启动**。

当屏幕变黑并在 VM 窗口底部中央显示键盘图像时，按任意键开始 Ubuntu 安装。只要你这样做，语言选择屏幕就会出现。选择对你来说最合适的语言，但在这个例子中，我们将选择**英语**。然后选择**安装 Ubuntu**。

有时，你可能会在虚拟机窗口上看到一条看起来不寻常的错误信息——类似于**SMBus 基址未初始化**。显示此消息是因为 VirtualBox 不支持 Ubuntu 12.04 默认加载的特定内核模块。然而，这不会造成任何困难，仅是外观上的困扰。几分钟后，一个不错的图形界面安装屏幕将出现，等待你再次选择语言。我们将再次选择**英语**。

在接下来的**准备安装 Ubuntu**屏幕上，显示了三个清单项目。由于你的虚拟驱动器远大于 Ubuntu 的最小要求，你应该已经满足第一个项目。为了满足其他项目，确保你的主系统已连接电源并建立了网络连接。尽管这对于我们的目的来说完全是多余的，但在继续之前，我们几乎总是勾选**安装时下载更新**和**安装此第三方软件**的复选框。

在**安装类型**屏幕上，我们将选择简单的方式，并选择**擦除磁盘并安装 Ubuntu**。请记住，这只会擦除你虚拟机的虚拟硬盘上的磁盘，而主系统将保持完整。在**擦除磁盘并安装 Ubuntu**的屏幕上，你的虚拟硬盘应该已经被选中，所以你只需点击**立即安装**。

在此之后的 Ubuntu 安装过程中，将同时进行两项任务：在后台线程中，安装程序将准备虚拟驱动器以安装基本系统；其次，你将配置新系统的某些基本方面。但首先，你需要通过点击世界地图上的适当位置来标识你的时区，然后继续。接下来，确认你的键盘布局并继续。

设置你的第一个用户账户。在这种情况下，它将是我们在本书中进行工作的账户，因此我们将输入以下信息：

+   **您的姓名**：**书籍用户**

+   **计算机名称**：**SE-for-Android**

+   **选择一个用户名**：**bookuser**

+   **密码字段**：（按您的喜好填写）

我们还将选择**自动登录**。虽然出于安全考虑我们通常不会这样做，但为了方便，我们将在本地 VM 中这样做；但你可以按照你喜欢的任何方式保护此账户。

Ubuntu 安装完成后，会出现一个提示你重启计算机的对话框。点击**立即重启**按钮，几分钟后，终端提示会通知你移除所有安装介质并按*Enter*。要移除虚拟安装 CD，请使用 VirtualBox 菜单栏的**设备** | **CD/DVD 设备** | **从虚拟驱动器中移除磁盘**。然后按*Enter*重启 VM，但通过关闭 VM 窗口中断引导过程。它会询问你是否想要关闭机器。只需点击**确定**。

# VirtualBox 扩展包和客户端附加组件

为了使你的 Ubuntu 客户端虚拟机获得最佳性能并访问与 UDOO 一起工作所需的虚拟 USB 设备，你需要安装 VirtualBox 扩展包和客户端附加组件。

## VirtualBox 扩展包

从 VirtualBox 网站下载扩展包，网址为 [`www.virtualbox.org/wiki/Downloads`](http://www.virtualbox.org/wiki/Downloads)。那里会有一个针对**所有支持的平台**的下载链接。下载此文件后，你需要安装它。每个宿主系统的安装过程都不同，但非常直接。对于 Linux 和 Mac OS X 宿主系统，只需双击下载的扩展包文件即可。对于 Windows 系统，你需要运行下载的安装程序。

## VirtualBox 客户端附加组件

完成扩展包的安装后，通过在左侧面板中选择虚拟机并在工具栏上点击**启动**来从 VirtualBox 启动你的 Ubuntu Linux 12.04 虚拟机。当你的 Ubuntu 桌面激活后，你会注意到它并不适合你的虚拟机窗口。请调整虚拟机窗口大小以使其更大，但虚拟机屏幕大小将保持不变。安装 VirtualBox 客户端附加组件可以解决这些问题，包括其他性能问题。你可能会看到在虚拟桌面上弹出一个窗口，提示有新的 Ubuntu 版本可用。不要升级，只需关闭该窗口。

通过 VirtualBox 菜单栏，转到**设备** | **插入客户添加光盘镜像…**。不久之后，会出现一个对话框，询问你是否想要运行刚刚插入的新介质上的软件。点击**运行**按钮。然后你需要通过输入用户密码（在设置过程中输入的密码）来验证你的用户身份。用户验证通过后，将自动构建并更新几个内核模块。脚本完成后，通过点击屏幕右上角的齿轮，选择**关机…**，并在随后出现的对话框中点击**重启**来重启虚拟机。

当虚拟机重启后，你首先应该注意到的是虚拟机屏幕现在可以适应虚拟机窗口了。此外，如果你调整虚拟机窗口的大小，虚拟机屏幕也会随之调整。这是确定你已经成功安装 VirtualBox 客户添加功能的简单方法。

# 使用共享文件夹节省时间

在为 UDOOU 开发镜像时，你可以通过设置宿主系统和 Ubuntu Linux 客户系统之间的共享文件夹来提升整体性能。这样，一旦你为 UDOOU 构建了新的 SD 卡镜像，就可以通过共享文件夹直接让宿主系统访问该镜像。然后宿主系统可以通过长时间运行的命令来烧录 SD 卡，而不会因为虚拟化层降低对宿主读卡器的访问速度而增加时间消耗。在我们编写这本书所使用的系统中，每烧录一个镜像可以节省大约 10 分钟的时间。

要设置共享文件夹，你需要在 VirtualBox 管理器打开的情况下，并且 Ubuntu 虚拟机已关闭。点击工具栏上的**设置**图标。然后在打开的**设置**对话框中选择**共享文件夹**标签页。点击右侧的**添加共享文件夹**图标。输入你想要共享的宿主系统上的文件夹的**文件夹路径**。在我们的例子中，我们创建了一个名为`vbox_share`的新文件夹与虚拟机客户共享。VirtualBox 将生成**文件夹名称**，但在点击**确定**之前，请确保选择了**自动挂载**。从现在开始启动 Ubuntu 虚拟机时，共享文件夹可以在你的客户虚拟机中以`/media/sf_<folder_name>`访问。但是，如果你尝试从客户机列出该目录中的文件，你可能会被拒绝。为了使我们的`bookuser`完全访问这个文件夹（即读写权限），我们需要将那个 UID 添加到`vboxsf`组中：

```kt
$ sudo usermod -a -G vboxsf bookuser

```

退出并重新登录客户系统，或者重启客户虚拟机以完成该过程。

# 构建环境

为了准备构建 Linux 内核、Android 以及 Android 应用程序的系统，我们需要安装并设置一些关键软件。点击屏幕左侧启动栏顶部的 Ubuntu 仪表盘图标。在出现的搜索栏中，输入`term`并按*Enter*。将打开一个终端窗口。然后执行以下命令：

```kt
$ sudo apt-get update
$ sudo apt-get install apt-file git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev libc6-dev lib32ncurses5-dev ia32-libs x11proto-core-dev libx11-dev ia32-libs dialog liblzo2-dev libxml2-utils minicom

```

当系统询问你是否希望继续时，输入`y`并按下*Enter*键。

# Oracle Java 6

从 Oracle Java 归档网站下载最新版的 Java 6 SE 开发工具包（版本 6u45），网址为[`www.oracle.com/technetwork/java/javase/archive-139210.html`](http://www.oracle.com/technetwork/java/javase/archive-139210.html)。你需要`jdk-6u45-linux-x64.bin`版本以满足谷歌的目标开发环境。下载完成后，执行以下命令来安装 Java 6 JDK：

```kt
$ chmod a+x jdk-6u45-linux-x64.bin
$ sudo mkdir -p /usr/lib/jvm
$ sudo mv jdk-6u45-linux-x64.bin /usr/lib/jvm/
$ cd /usr/lib/jvm/
$ sudo ./jdk-6u45-linux-x64.bin
$ sudo update-alternatives --install "/usr/bin/java" "java" "/usr/lib/jvm/jdk1.6.0_45/bin/java" 1
$ sudo update-alternatives --install "/usr/bin/jar" "jar" "/usr/lib/jvm/jdk1.6.0_45/bin/jar" 1
$ sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/lib/jvm/jdk1.6.0_45/bin/javac" 1
$ sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/lib/jvm/jdk1.6.0_45/bin/javaws" 1
$ sudo update-alternatives --install "/usr/bin/jar" "jar" "/usr/lib/jvm/jdk1.6.0_35/bin/jar" 1
$ sudo update-alternatives --install "/usr/bin/javadoc" "javadoc" "/usr/lib/jvm/jdk1.6.0_45/bin/javadoc" 1
$ sudo update-alternatives --install "/usr/bin/jarsigner" "jarsigner" "/usr/lib/jvm/jdk1.6.0_45/bin/jarsigner" 1
$ sudo update-alternatives --install "/usr/bin/javah" "javah" "/usr/lib/jvm/jdk1.6.0_45/bin/javah" 1
$ sudo rm jdk-6u45-linux-x64.bin

```

# 总结

在本附录中，我们讨论了谷歌为 Android 设定的目标开发环境，并展示了如何在一个虚拟机中创建一个兼容的环境。你可以自由修改系统的其他元素，但安装本附录所列元素将为你提供一个能够执行第四章《在 UDOO 上的安装》及以后所有步骤的最低限度的可行环境。
