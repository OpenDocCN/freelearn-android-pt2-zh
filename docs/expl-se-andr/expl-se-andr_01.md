# 第一章：Linux 访问控制

Android 是一个由两个不同组件组成的操作系统。第一个组件是分叉的 Linux 主线内核，几乎与 Linux 共享所有内容。第二个组件，将在后面讨论，是用户空间部分，这部分非常定制且特定于 Android。由于 Linux 内核支撑这个系统并且负责大多数访问控制决策，所以从逻辑上讲，这是深入研究 Android 的一个很好的起点。

在本章中我们将：

+   检查自主访问控制的基础

+   介绍 Linux 权限标志和能力

+   在验证访问策略时跟踪系统调用

+   论证更强大的访问控制技术的必要性

+   讨论利用自主访问控制问题的 Android 漏洞

Linux 的默认且熟悉的访问控制机制称为**自主访问控制**（**DAC**）。这只是一个术语，意味着关于访问对象的权限由其创建者/所有者自行决定。

在 Linux 中，当一个进程调用大多数系统调用时，会执行一个权限检查。例如，希望打开一个文件的进程会调用`open()`系统调用。当调用这个系统调用时，会执行上下文切换，操作系统代码开始执行。操作系统有能力决定是否应该向请求的进程返回文件描述符。在做出这个决定的过程中，操作系统会检查请求进程以及它希望获得文件描述符的目标文件的访问权限。根据权限检查通过或失败，返回的将是文件描述符或 EPERM。

Linux 在内核中维护数据结构以管理这些权限字段，这些字段可以从用户空间访问，并且对于 Linux 和*NIX 用户来说应该是熟悉的。第一组访问控制元数据属于进程，构成了其凭据集的一部分。常见的凭据是用户和组。通常，我们使用组这个术语来指代主要组以及可能的次要组。你可以通过运行`ps`命令来查看这些权限：

```kt
$ ps -eo pid,comm,user,group,supgrp
PID COMMAND         USER     GROUP    SUPGRP
1 init            root     root     -
...
 2993 system-service- root     root     root 
 3276 chromium-browse bookuser sudo fuse bookuser 
...

```

如你所见，我们有以`root`和`bookuser`用户身份运行的进程。你还可以看到，他们的主要组只是等式的一部分。进程还有一组辅助组，称为补充组。这个集合可能是空的，由`SUPGRP`字段中的破折号表示。

我们希望打开的文件，被称为目标对象、目标或对象，同时也维护一组权限。该对象维护`USER`和`GROUP`，以及一组权限位。在目标对象的上下文中，`USER`可以被称为*所有者*或*创建者*。

```kt
$ ls -la
total 296
drwxr-xr-x 38 bookuser bookuser  4096 Aug 23 11:08 .
drwxr-xr-x  3 root     root      4096 Jun  8 18:50 ..
-rw-rw-r--  1 bookuser bookuser   116 Jul 22 13:13 a.c
drwxrwxr-x  4 bookuser bookuser  4096 Aug  4 16:20 .android
-rw-rw-r--  1 bookuser bookuser   130 Jun 19 17:51 .apport-ignore.xml
-rw-rw-r--  1 bookuser bookuser   365 Jun 23 19:44 hello.txt
-rw-------  1 bookuser bookuser 19276 Aug  4 16:36 .bash_history
...

```

如果我们查看前面命令的输出，我们可以看到`hello.txt`的`USER`是`bookuser`，`GROUP`是`bookuser`。我们还可以看到输出左侧的权限位或标志。还有七个字段需要考虑。每个空字段都用破折号表示。当使用`ls`打印时，第一个字段可能会因语义而变得混乱。因此，让我们使用`stat`来调查文件权限：

```kt
$ stat hello.txt
 File: `hello.txt'
 Size: 365         Blocks: 8          IO Block: 4096   regular file
Device: 801h/2049d  Inode: 1587858     Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1000/bookuser)   Gid: ( 1000/bookuser)
Access: 2014-08-04 15:53:01.951024557 -0700
Modify: 2014-06-23 19:44:14.308741592 -0700
Change: 2014-06-23 19:44:14.308741592 -0700
 Birth: -

```

第一行访问信息是最有力的。它包含了所有访问控制的重要信息。第二行只是一个时间戳，告诉我们文件最后被访问的时间。正如我们所见，对象的`USER`或`UID`是`bookuser`，`GROUP`也是`bookuser`。权限标志（`0664/-rw-rw-r--`）标识了两种表示权限标志的方式。第一种是八进制形式`0664`，将每个三标志字段压缩为一个三基数（八进制）数字。第二种是友好形式，`-rw-rw-r--`，等同于八进制形式，但视觉上更容易解读。在任何情况下，我们可以看到最左边的字段是 0，我们的其余讨论将忽略它。该字段用于`setuid`和`setgid`功能，这对于本讨论不重要。如果我们把剩下的八进制数字 664 转换为二进制，我们得到 110 110 100。这个二进制表示直接关联到友好形式。每个三重映射到读、写和执行权限。通常你会看到这个权限三重表示为`RWX`。第一个三重是给`USER`的权限，第二个是给`GROUP`的权限，第三个是给`OTHERS`的权限。翻译成常规英语就是，“用户`bookuser`有权从`hello.txt`中读取和写入。组`bookuser`有权从`hello.txt`中读取和写入，而其他人只有权从`hello.txt`中读取。”让我们通过一些现实世界的例子来测试这一点。

# 更改权限位

让我们以`bookuser`用户的身份测试示例运行过程中的访问控制。大多数进程在调用它们的用户的上下文中运行（不包括`setuid`和`getuid`程序），所以任何我们调用的命令都应该继承我们用户的权限。我们可以通过发出以下命令来查看：

```kt
$ groups bookuser
bookuser : bookuser sudo fuse

```

我的用户，`bookuser`，是`USER bookuser`，`GROUP bookuser`以及`SUPGRP sudo`和`fuse`。

要测试读取权限，我们可以使用`cat`命令，它打开文件并将其内容打印到`stdout`：

```kt
$ cat hello.txt 
Hello, "Exploring SE for Android"
Here is a simple text file for
your enjoyment.
...

```

我们可以通过运行`strace`命令并查看输出来自省执行的系统调用：

```kt
$ strace cat hello.txt 
...
open("hello.txt", O_RDONLY)                   = 3
...
read(3, "Hello, \"Exploring SE for Android\"\n"..., 32768) = 365
...

```

输出可能会相当冗长，因此我只展示了相关部分。我们可以看到`cat`调用了`open`系统调用并获得了文件描述符`3`。我们可以使用该描述符通过其他系统调用查找其他访问。稍后我们会看到在文件描述符`3`上发生了一个读取操作，它返回了`365`，即读取的字节数。如果我们没有从`hello.txt`读取的权限，打开操作将会失败，我们也永远不会得到该文件的有效的文件描述符。我们还会在`strace`输出中看到失败的信息。

既然已经验证了读取权限，让我们尝试写入。一个简单的方法是编写一个简单的程序，将内容写入现有文件。在本例中，我们将写入`my new text\n`（参考`write.c`文件）。

使用以下命令编译程序：

```kt
$ gcc -o mywrite write.c

```

现在使用新编译的程序运行：

```kt
$ strace ./mywrite hello.txt

```

在验证时，你会看到：

```kt
...
open("hello.txt", O_WRONLY)                   = 3
write(3, "my new text\n", 12)           = 12
...

```

如你所见，写入操作成功，并返回了`12`，即写入到`hello.txt`的字节数。没有报告错误，所以权限似乎到目前为止是检查无误的。

现在尝试执行`hello.txt`，看看会发生什么。我们预期会看到错误。像执行普通命令那样执行它：

```kt
$ ./hello.txt
bash: ./hello.txt: Permission denied

```

这正是我们所预期的，但让我们用`strace`来更深入地了解究竟哪里出了问题：

```kt
$ strace ./hello.txt
...
execve("./hello.txt", ["./hello.txt"], [/* 39 vars */]) = -1 EACCES (Permission denied)
...

```

`execve`系统调用，它用于启动进程，由于`EACCESS`错误而失败。这正是当没有执行权限时所希望看到的情况。Linux 的访问控制按预期工作！

现在我们将在另一个用户的上下文中测试访问控制。首先，我们将使用`adduser`命令创建一个名为`testuser`的新用户：

```kt
$ sudo adduser testuser
[sudo] password for bookuser: 
Adding user `testuser' ...
Adding new group `testuser' (1001) ...
Adding new user `testuser' (1001) with group `testuser' ...
Creating home directory `/home/testuser' ...
...

```

验证`testuser`的`USER`、`GROUP`和`SUPGRP`：

```kt
$ groups testuser
testuser : testuser

```

由于`USER`和`GROUP`与`a.S`上的任何权限都不匹配，所有的访问都将受到`OTHERS`权限检查，正如你所记得的，这是只读的（`0664`）。

首先临时作为`testuser`工作：

```kt
$ su testuser
Password: 
testuser@ubuntu:/home/bookuser$ 

```

如你所见，我们仍然在 bookuser 的主目录中，但当前用户已经变更为`testuser`。

我们将先用`cat`命令测试`read`：

```kt
$ strace cat hello.txt
...
open("hello.txt", O_RDONLY)                   = 3
...
read(3, "my new text\n", 32768)         = 12
...

```

与前面的示例类似，正如预期的那样，`testuser`可以顺利地读取数据。

现在让我们进行写入测试。预期没有适当的权限这将失败：

```kt
$ strace ./mywrite hello.txt
...
open("hello.txt", O_WRONLY)                   = -1 EACCES (Permission denied)
...

```

如预期的那样，系统调用操作失败了。当我们尝试以`testuser`的身份执行`hello.txt`时，也应该失败：

```kt
$ strace ./hello.txt
...
execve("./hello.txt", ["./hello.txt"], [/* 40 vars */]) = -1 EACCES (Permission denied)
...

```

现在我们需要测试组访问权限。我们可以通过向`testuser`添加一个补充组来实现这一点。为此，我们需要退出到有权限执行`sudo`命令的`bookuser`：

```kt
$ exit
exit
$ sudo usermod -G bookuser testuser

```

现在让我们检查`testuser`的组：

```kt
$ groups testuser
testuser : testuser bookuser

```

由于之前的`usermod`命令，`testuser`现在属于两个组：`testuser`和`bookuser`。这意味着当`testuser`访问具有`bookuser`组的文件或其他对象（如套接字）时，将应用`GROUP`权限，而不是`OTHERS`。在`hello.txt`的背景下，`testuser`现在可以读取和写入文件，但不能执行它。

通过执行以下命令切换到`testuser`：

```kt
$ su testuser

```

通过执行以下命令测试`read`：

```kt
$ strace cat ./hello.txt
...
open("./hello.txt", O_RDONLY)                 = 3
...
read(3, "my new text\n", 32768)         = 12
...

```

与之前一样，`testuser`能够读取文件。唯一的区别是现在它可以通过`OTHERS`和`GROUP`的访问权限来`read`文件。

通过执行以下命令测试`write`：

```kt
$ strace ./mywrite hello.txt
...
open("hello.txt", O_WRONLY)                   = 3
write(3, "my new text\n", 12)           = 12
...

```

这一次，`testuser`不仅能够写入文件，而不是像之前那样遇到`EACCESS`权限错误。

尝试执行文件应该仍然会失败：

```kt
$ strace ./hello.txt
execve("./hello.txt", ["./hello.txt"], [/* 40 vars */]) = -1 EACCES (Permission denied)
...

```

这些概念是 Linux 访问控制权限位、用户和组的基础。

# 更改所有者和组

在前面的章节中，我们使用`hello.txt`进行探索性工作，展示了对象的所有者如何通过管理对象的权限位来允许各种形式的访问。更改权限是通过使用`chmod`系统调用完成的。更改用户和/或组是通过`chown`系统调用来完成的。在本节中，我们将研究这些操作的具体细节。

让我们从仅向`hello.txt`文件的所有者`bookuser`授予读和写权限开始。

```kt
$ chmod 0600 hello.txt
$ stat hello.txt
 File: `hello.txt'
 Size: 12          Blocks: 8          IO Block: 4096   regular file
Device: 801h/2049d  Inode: 1587858     Links: 1
Access: (0600/-rw-------)  Uid: ( 1000/bookuser)   Gid: ( 1000/bookuser)
Access: 2014-08-23 12:34:30.147146826 -0700
Modify: 2014-08-23 12:47:19.123113845 -0700
Change: 2014-08-23 12:59:04.275083602 -0700
 Birth: -

```

如我们所见，现在文件权限设置为只允许`bookuser`读取和写入。一个细致的读者可以执行本章前面部分提到的命令来验证权限是否按预期工作。

更改组也可以通过`chown`以类似的方式进行。让我们将组更改为`testuser`：

```kt
$ chown bookuser:testuser hello.txt
chown: changing ownership of `hello.txt': Operation not permitted

```

这并没有按照我们的预期工作，但问题出在哪里呢？在 Linux 中，只有特权进程可以更改对象的`USER`和`GROUP`字段。在对象创建时，初始的`USER`和`GROUP`字段是从有效的`USER`和`GROUP`中设置的，在尝试执行该进程时会进行检查。只有进程可以创建对象。特权进程有两种形式：作为全能的`root`运行和设置了其功能的进程。我们稍后会详细介绍功能。现在，让我们关注`root`。

让我们切换到`root`用户，以确保执行`chown`命令可以更改该对象的组：

```kt
$ sudo su
# chown bookuser:testuser hello.txt 
Now, we can verify the change occurred successfully:
# stat hello.txt
 File: `hello.txt'
 Size: 12          Blocks: 8          IO Block: 4096   regular file
Device: 801h/2049d  Inode: 1587858     Links: 1
Access: (0600/-rw-------)  Uid: ( 1000/bookuser)   Gid: ( 1001/testuser)
Access: 2014-08-23 12:34:30.147146826 -0700
Modify: 2014-08-23 12:47:19.123113845 -0700
Change: 2014-08-23 13:08:46.059058649 -0700
 Birth: -

```

# 更多情况的考虑

你可以看到`GROUP`（`GID`）现在是`testuser`，事情看起来相当安全，因为要更改对象的用户和组，你需要具备特权。只有在你拥有对象时，你才能更改对象的权限位，`root`用户除外。这意味着如果你以`root`身份运行，即使没有权限，你也可以对系统进行任何操作。这种绝对权威就是为什么以 root 运行的进程遭到成功攻击或错误时可能对系统造成严重损害的原因。此外，非 root 进程遭到成功攻击也可能通过无意中更改权限位造成损害。例如，假设你的 SSH 私钥上有一个非预期的`chmod 0666`命令。这将使你的密钥对所有系统用户暴露，这几乎肯定是你绝对不希望发生的事情。能力模型部分解决了 root 的限制。

# 能力模型

在 Linux 上执行许多操作时，对象权限模型并不完全适用。例如，更改`UID`和`GID`需要一种被称为`root`的神奇`USER`。假设你有一个需要利用这些功能的长运行服务。也许这个服务监听内核事件并为你创建设备节点？这样的服务确实存在，它被称为`ueventd`或用户事件守护进程。这个守护进程传统上以`root`身份运行，这意味着如果它被攻破，它可能会从你的主目录读取你的私钥并将其发送给攻击者。这可能是一个极端的例子，但它旨在展示以`root`身份运行进程可能很危险。假设你可以以`root`用户身份启动一个服务，并让进程将其`UID`和`GID`更改为不具备特权的用户，但保留一些较小的特权能力以完成其工作？这正是 Linux 中的能力模型所做的。

Linux 中的能力模型试图将`root`所具有的权限集分解为更小的子集。这样，进程可以被限制在执行预期功能所需的最小权限集中。这就是所谓的最小权限，这是在保护系统时减少成功攻击可能造成的损害的关键理念。在某些情况下，它甚至可以通过阻止其他开放的攻击向量来防止成功攻击的发生。

能力有很多种。能力的手册页是事实上的文档。让我们看看`CAP_SYS_BOOT`能力：

```kt
$ man capabilities
...
CAP_SYS_BOOT
 Use reboot(2) and kexec_load(2).

```

这意味着具有此能力的进程可以重启系统。但是，该进程不能像以`root`身份运行或具有`CAP_DAC_READ_SEARCH`时那样任意更改`USERS`和`GROUP`。这限制了攻击者可以执行的操作：

```kt
<FROM MAN PAGE>
CAP_DAC_READ_SEARCH
 Bypass file read permission checks and directory read and execute permission checks.

```

现在假设我们的重启进程运行时带有`CAP_CHOWN`权限。假设它使用这个功能确保在接收到重启请求时，在重启之前备份每个用户的家目录下的一个文件到服务器。假设这个文件是`~/backup`，权限是 0600，`USER`和`GROUP`分别是该家目录的相应用户和组。在这种情况下，我们已经尽可能最小化了权限，但进程仍然可以访问用户的 SSH 密钥，并且可能由于错误或攻击而上传这些密钥。另一种方法是设置组为`backup`，并以`GROUP backup`运行进程。然而，这也有局限性。假设你想与另一个用户共享这个文件。该用户需要`backup`的辅助组，但现在用户可以读取*所有*备份文件，而不仅仅是预期的那些。一个敏锐的读者可能会考虑到`bind`挂载，然而执行`bind`挂载和文件权限的进程也带有某些权限，因此也受到这个粒度问题的影响。

主要问题，以及另一个访问控制系统的案例可以用一个词来概括，那就是*粒度*。DAC 模型没有足够的粒度来安全处理复杂的访问控制模型，或者最小化一个进程可能造成的损害。这在 Android 上尤为重要，因为整个隔离系统都依赖于这种控制，一个恶意 root 进程可能会破坏整个系统。

# Android 对 DAC 的使用

在 Android 沙盒模型中，每个应用程序都以其自己的`UID`运行。这意味着每个应用都可以将其存储的数据与其他应用隔离开来。用户和组设置为该应用程序的`UID`和`GID`，因此没有应用可以在应用程序显式对其对象执行`chmod`的情况下访问另一个应用程序的私有文件。此外，Android 中的应用程序不能拥有权限，因此我们不必担心如`CAP_SYS_PTRACE`这样的权限，即调试另一个应用程序的能力。在 Android 中，在一个完美的世界里，只有系统组件会运行带权限，应用程序不会意外地对所有用户执行`chmod`操作以读取私有文件。由于应用程序兼容性，当前 AOSP SELinux 策略没有纠正这个问题，但可以通过 SELinux 来解决。在 Android 上，应用程序之间共享数据的正确方式是通过 binder 和共享文件描述符。对于较小的数据量，提供者模型就足够了。

# 浏览 Android 漏洞

利用我们对 DAC 权限模型及其一些局限性的新理解，让我们来看一些针对它的 Android 攻击方式。我们将仅涵盖一些攻击方式，以了解 DAC 模型是如何失败的。

## Skype 漏洞

CVE-2011-1717 在 2011 年发布。在这个漏洞中，Skype 应用程序留下了一个 SQLite3 数据库，该数据库可以被全世界读取（类似于 0666 权限）。这个数据库包含了用户名和聊天日志，以及如姓名和电子邮件等个人数据。一个名为 Skypwned 的应用程序能够演示这一功能。这是一个改变对象权限可能导致严重后果的例子，特别是在将权限从`READ`开放给`OTHERS`的情况下。

## GingerBreak

CVE-2011-1823 展示了对 Android 系统的 root 攻击。Android 上的卷管理守护进程（vold）负责外部 SD 卡挂载和卸载。该守护进程通过 NETLINK 套接字监听消息。守护进程从未检查过消息的来源，任何应用都可以打开并创建 NETLINK 套接字向 vold 发送消息。攻击者一旦打开 NETLINK 套接字，就会发送一个精心构造的消息来绕过健全性检查。该检查测试了一个有符号整数的最小界限，但从未检查过它是否为负数。然后它被用来索引一个数组。这种负数访问将导致内存破坏，如果消息恰当，可能导致执行任意代码。GingerBreak 的实现使得任意用户获得了 root 权限，这是一个典型的权限执行攻击。设备一旦被 root，沙盒就不再有效。

## Rage against the cage

CVE-2010-EASY 是通过 fork 炸弹攻击实现的`setuid`耗尽。它成功攻击了 Android 上的`adb`守护进程，该进程最初以 root 权限启动，如果不需要 root 权限则降级权限。这种攻击使`adb`保持为`root`，并向用户返回一个 root shell。在 Linux 内核 2.6 中，当运行进程数达到`RLIMIT_NPROC`时，`setuid`系统调用返回错误。`adb`守护进程代码没有检查`setuid`的返回值，这为攻击者留下了一个小的竞争窗口。攻击者需要分叉足够多的进程以达到`RLIMIT_NPROC`，然后杀死守护进程。`adb`守护进程降级到 shell `UID`，攻击者以 shell `USER`身份运行程序，因此 kill 命令将成功执行。此时，`adb`服务会被重新启动，如果`RLIMIT_NPROC`已达到最大值，`setuid`将失败，`adb`将保持以 root 权限运行。然后，从主机运行`adb shell`会向用户返回一个很好的 root shell。

## MotoChopper

CVE-2013-2596 是高通视频驱动程序`mmap`功能中的一个漏洞。应用程序通过`mmap`访问 GPU 以进行高级图形渲染，例如 OpenGL 调用。`mmap`中的漏洞允许攻击者`mmap`内核地址空间，此时攻击者能够直接改变他们的内核凭据结构。这个漏洞是一个例子，其中 DAC 模型并没有出错。实际上，除了修补代码或移除直接图形访问权限之外，只有对`mmap`边界的编程检查才能防止这种攻击。

# 总结

DAC 模型非常强大，但其缺乏细粒度控制以及使用异常强大的`root`用户，仍有所不足。随着移动设备使用的敏感性增加，提高系统安全性的需求是有根据的。幸运的是，Android 建立在 Linux 之上，因此受益于一个由众多工程师和研究人员构成的庞大生态系统。自从 Linux 内核 2.6 起，一种名为**强制访问控制（MAC）**的新访问控制模型被加入。这是一个框架，通过它可以将模块加载到内核中，以提供一种新的访问控制模型。第一个模块被称为 SELinux。它被 Red Hat 等公司用于保护敏感的政府系统。因此，找到了一个解决方案，以实现对 Android 的这种访问控制。
