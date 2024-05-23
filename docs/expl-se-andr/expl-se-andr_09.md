# 第九章：向域添加服务

在上一章中，我们介绍了将文件对象放入正确域的过程。在大多数情况下，文件对象是目标。然而，在本章中，我们将：

+   强调标记进程——尤其是由 init 运行和管理的 Android 服务。

+   管理由 init 创建的辅助关联对象。

# Init —— 守护进程之王

在 Linux 系统中，init 进程至关重要，Android 在这方面也不例外。然而，Android 有其自己的 init 实现。Init 是系统上的第一个进程，因此具有**进程 ID**（**PID**）为 1。所有其他进程都是直接从 init 进行`fork()`的结果，因此所有进程最终都会直接或间接地成为 init 的子进程。Init 负责清理和维护这些进程。例如，任何父进程死亡的孩子进程都会被内核重新设置为 init 的子进程。这样，当进程退出时，init 可以调用`wait()`（更多详情请查看`man 2 wait`）来清理进程。

### 注意

已经终止但尚未调用`wait()`的进程是**僵尸进程**。在调用此函数之前，内核必须保留进程数据结构。如果做不到这一点，将会无限期地消耗内存。

由于 init 是所有进程的根，它还提供了一种通过其自己的脚本语言声明和执行命令的机制。使用这种语言来控制 init 的文件称为 init 脚本，我们已经修改了一些。在源代码树中，我们使用了`init.rc`文件，您可以通过导航到`device/fsl/imx6/etc/init.rc`来找到它，但在设备上，它与 ramdisk 一起打包在`/init.rc`，并可供 init 使用，init 也包含在 ramdisk 中的`/init`。

要向 init 脚本添加服务，您可以修改`init.rc`文件并添加一个声明，如下所示：

```kt
service <name> <path> [ <argument>... ]
```

在这里，`name`是服务名称，`path`是可执行文件的路径，而`argument`是要传递给可执行文件在它的`argv`数组中的以空格分隔的参数字符串。

例如，以下是`rild`的 service 声明，即**无线接口层守护进程**（**RILD**）：

```kt
Service ril-daemon /system/bin/rild
```

通常情况下，可以并且需要添加额外的服务选项。init 脚本的`service`语句支持丰富的选项集合。要查看完整列表，请参考位于`system/core/init/readme.txt`的信息文件。此外，我们在第三章中介绍了针对 Android 特定的 SE 更改，*Android Is Weird*。

继续剖析`rild`，我们看到在 UDO 的`init.rc`中的声明其余部分如下：

```kt
Service ril-daemon /system/bin/rild
 class main
 socket rild stream 660 root radio
 socket rild-debug stream 660 radio system
 socket rild-ppp stream 660 radio system
 user root
 group radio cache inet misc audio sdcard_rw log

```

这里需要注意的是，它会创建相当多的套接字。`init.rc`中的`socket`关键字由`readme.txt`文件描述：

### 注意

来自源代码树文件`system/core/init/readme.txt`：

```kt
socket <name> <type> <perm> [ <user> [ <group> [ <context> ] ] ]

```

创建一个名为 `/dev/socket/<name>` 的 Unix 域套接字，并将其 `fd` 传递给启动的进程。类型必须是 `dgram`、`stream` 或 `seqpacket`。`user` 和 `group` ID 默认为 `0`。套接字的 SELinux 安全上下文是 `context`。默认为服务安全上下文，由 `seclabel` 指定，或者基于服务可执行文件的 security context 计算。

让我们查看这个目录，看看我们发现了什么。

```kt
root@udoo:/dev/socket # ls -laZ | grep adb
srw-rw---- system system u:object_r:adbd_socket:s0 adbd

```

这引发了这样一个问题：“它是如何进入那个域的？”根据我们上一章的知识，我们知道 **/** `dev` 是一个 `tmpfs`，所以我们知道它不是通过 `xattrs` 进入这个域的。它必须是一个代码修改或类型转换。让我们检查是否是类型转换。如果是，我们预计会在扩展的 `policy.conf` 中看到一条声明。SELinux 策略基于 `m4` 宏语言。在构建期间，它被扩展到 `policy.conf`，然后编译。第十二章，*掌握工具链*，对此有更多细节。

我们可以通过使用 sesearch 来查找 `adbd_socket` 的类型转换来发现这一点：

```kt
$ sesearch -T -t adbd_socket $OUT/sepolicy

```

正如您从空输出中看到的，没有这样的行，所以这不是策略所做的事情，而是代码更改。

在 Linux 中，进程是通过 `fork()` 然后 `exec()` 创建的。因此，我们能够提供很好的关键字来搜索 init 守护进程。我们怀疑设置套接字的代码就在子进程中的 `fork()` 调用之后，在 `exec()` 调用之前：

```kt
$ grep -n fork system/core/init/init.c 
235: pid = fork();

```

因此，我们要找的 `fork` 在 `init.c` 的第 235 行；让我们在文本编辑器中打开 `init.c` 并查看。我们将找到以下代码段进行审查：

```kt
...
NOTICE("starting '%s'\n", svc->name);

  pid = fork();

  if (pid == 0) {
    struct socketinfo *si;
    struct svcenvinfo *ei;
    char tmp[32];
    int fd, sz;

    umask(077);
    if (properties_inited()) {
      get_property_workspace(&fd, &sz);
      sprintf(tmp, "%d,%d", dup(fd), sz);
      add_environment("ANDROID_PROPERTY_WORKSPACE", tmp);
    }

    for (ei = svc->envvars; ei; ei = ei->next)
      add_environment(ei->name, ei->value);

    for (si = svc->sockets; si; si = si->next) {
      int socket_type = (
        !strcmp(si->type, "stream") ? SOCK_STREAM :
          (!strcmp(si->type, "dgram") ? SOCK_DGRAM : SOCK_SEQPACKET));
      int s = create_socket(si->name, socket_type,
            si->perm, si->uid, si->gid, si->socketcon ?: scon);
      if (s >= 0) {
        publish_socket(si->name, s);
      }
...
```

根据 `man 2 fork`，子进程中的 `fork()` 返回代码是 `0`。子进程在此 `if` 语句内执行，父进程跳过它。函数 `create` **_** `socket()` 也似乎很有趣。它似乎接受服务名称、套接字类型、权限标志、`uid`、`gid` 和 `socketcon`。什么是 `socketcon`？让我们检查是否可以追溯到它的设置位置。

如果我们查看 `fork()` 之前的内容，我们可以看到父进程根据两个因素获取其 `scon`：

```kt
...
    if (svc->seclabel) {
      scon = strdup(svc->seclabel);
      if (!scon) {
        ERROR("Out of memory while starting '%s'\n", svc->name);
        return;
      }
      } else {
...
```

当 `svc->seclabel` 不为空时，通过 `if` 语句的第一个路径发生。这个 `svc` 结构用与服务相关的选项填充。从第三章，*安卓很奇怪* 中回想一下，`seclabel` 允许您显式设置服务的上下文，硬编码到 `init.rc` 中的值。`else` 子句要复杂和有趣得多。

在`else`子句中，我们通过调用`getcon()`获取当前进程的上下文。由于我们是在 init 中运行，这个函数应该返回`u:r:init:s0`并将其存储在`mycon`中。下一个函数`getfilecon()`传递了可执行文件的路径，并检查文件本身的上下文。第三个函数是这里的工作马：`security_compute_create()`。它接收`mycon`、`fcon`和`target`类别，并计算安全上下文`scon`。给定这些输入，它会尝试根据策略类型转换确定子进程的结果域。如果没有定义转换，`scon`将与`mycon`相同。

`create_socket()`函数内的条件表达式另外决定了传递的套接字上下文。变量`si`是一个结构体，其中包含了 init `service`部分中套接字语句的所有选项。如`readme.txt`文件所述，`si->socketcon`是套接字上下文参数。换句话说，套接字上下文可能来自以下三个地方（按优先级递减）：

+   `service`声明中套接字选项的`socketcon`选项

+   `service`关键字上的`seclabel`选项

+   从源和目标上下文动态计算

套接字上下文被传递给`create_socket()`。现在，让我们看看`create_socket()`。这个函数在`system/core/init/util.c:87`定义。围绕`socket()`的代码片段似乎很有趣：

```kt
...
  if (socketcon)
    setsockcreatecon(socketcon);

  fd = socket(PF_UNIX, type, 0);
  if (fd < 0) {
    ERROR("Failed to open socket '%s': %s\n", name, strerror(errno));
    return -1;
  }

  if (socketcon)
    setsockcreatecon(NULL);
...
```

`setsockcreatecon()`函数设置了进程的套接字创建上下文。这意味着通过`socket()`调用创建的套接字将具有通过`setsockcreatecon()`设置的上下文。创建后，进程通过使用`setsockcreatecon(NULL)`将其重置为原始上下文。

下一段有趣的代码是关于`bind()`的：

```kt
...
  filecon = NULL;
  if (sehandle) {
    ret = selabel_lookup(sehandle, &filecon, addr.sun_path, S_IFSOCK);
    if (ret == 0)
      setfscreatecon(filecon);
  }

  ret = bind(fd, (struct sockaddr *) &addr, sizeof (addr));
  if (ret) {
    ERROR("Failed to bind socket '%s': %s\n", name, strerror(errno));
    goto out_unlink;
  }

  setfscreatecon(NULL);
  freecon(filecon);
...
```

在这里，我们设置了文件创建的上下文。这些功能与`setsock_creation()`类似，但适用于文件系统对象。然而，`selabel_lookup()`函数会在`file_contexts`中查找文件的上下文。你可能遗漏的部分是，对于基于路径的套接字，`bind()`的调用会在`sockaddr_un struct`指定的路径上创建一个文件。因此，套接字对象和文件系统节点条目是截然不同的，并且可以具有不同的上下文。通常，套接字属于进程的上下文，而文件系统节点被赋予其他上下文。

# 动态域转换

我们看到 init 计算了 init 套接字的上下文，但在为子进程设置域时从未遇到过。在本节中，我们将深入探讨两种实现方法：使用 init 脚本显式设置和 sepolicy 动态域转换。

设置子进程域的第一种方式是在 init 脚本服务声明中使用`seclabel`语句。在`fork()`之后的子进程执行中，我们发现了这个语句：

```kt
if (svc->seclabel) {
if (is_selinux_enabled() > 0 && setexeccon(svc->seclabel) < 0) {
ERROR("cannot setexeccon('%s'): %s\n", svc->seclabel, strerror(errno));
_exit(127);
}
}
```

为了澄清，`svc`变量是包含服务选项和参数的结构，所以`svc->seclabel`就是`seclabel`。如果它被设置了，它会调用`setexeccon()`，后者为进程通过`exec()`执行的任何东西设置执行上下文。再往下，我们看到`exec()`函数调用。`exec()`系统调用在成功时永远不会返回；它只在失败时返回。

为子进程设置域的另一种方式，这种方式更为推荐，就是使用 sepolicy。之所以推荐，是因为策略不依赖于其他任何东西。通过在 init 中硬编码上下文，你就在 init 脚本和 sepolicy 之间耦合了一个依赖关系。例如，如果 sepolicy 移除了在 init 脚本中硬编码的类型，init `setcon`将失败，但两个系统都能正确编译。如果你移除了一个类型转换的类型，并留下了转换语句，你可以在编译时捕获错误。由于我们查看了`rild`服务语句，让我们看看位于`sepolicy`中的`rild.te`策略文件。我们应该在这个文件中使用`grep`搜索`type_transition`关键字：

```kt
$ grep -c type_transition rild.te 
0

```

没有找到`type_transition`的实例，但这个关键字必须存在，类似于文件。然而，它可能隐藏在一个未展开的宏中。SELinux 策略文件是用 m4 宏语言编写的，它们在编译之前会被展开。让我们查看`rild.te`文件，看看我们是否能找到一些宏。它们具有参数，看起来像函数。我们遇到的第一个宏是`init_daemon_domain(rild)`。现在，我们需要在`sepolicy`中找到这个宏的定义。m4 语言使用`define`关键字来声明宏，所以我们可以搜索这个：

```kt
$ grep -n init_daemon_domain * | grep define
te_macros:99:define(`init_daemon_domain', `

```

我们的宏在`te_macros`中声明，碰巧它包含了与**类型强制执行**（**TE**）相关的所有宏。让我们更详细地看看这个宏的作用。首先，它的定义是：

```kt
...
#####################################
# init_daemon_domain(domain)
# Set up a transition from init to the daemon domain
# upon executing its binary.
define(`init_daemon_domain', `
domain_auto_trans(init, $1_exec, $1)
tmpfs_domain($1)
')
...
```

上述代码中的注释行（以`#`开头的 m4 行），表明它设置了一个从 init 到守护进程域的转换。这似乎是我们想要的东西。然而，包含它们的语句都是宏，我们需要递归地展开它们。我们将从`domain_auto_trans()`开始：

```kt
...
#####################################
# domain_auto_trans(olddomain, type, newdomain)
# Automatically transition from olddomain to newdomain
# upon executing a file labeled with type.
#
define(`domain_auto_trans', `
# Allow the necessary permissions.
domain_trans($1,$2,$3)
# Make the transition occur by default.
type_transition $1 $2:process $3;
')
...
```

这里的注释表明我们正朝着正确的方向前进；然而，在搜索过程中，我们需要继续展开宏。根据注释，`domain_trans()`宏允许仅发生转换。请记住，在 SELinux 中几乎所有的操作都需要来自策略的明确许可才能进行，包括类型转换。宏中的最后一条语句是我们一直在寻找的：

```kt
type_transition $1 $2:process $3;
```

如果你展开这条语句，你会得到：

```kt
type_transition init rild_exec:process rild;
```

这条语句传达的意思是，如果你在一个类型为`rild_exec`的文件上执行`exec()`系统调用，并且执行域是 init，那么将子进程的域设置为`rild`。

# 通过 seclabel 显示上下文

设置上下文的另一种方法非常直接。就是在`service`声明中通过初始化脚本将它们硬编码。在`service`声明中，正如我们在第三章《*安卓很奇怪*》中所看到的，对 init 语言进行了修改。其中一个添加项是`seclabel`。这个选项只是让 init 明确地将服务的上下文更改为传递给`seclabel`的参数。以下是`adbd`的一个例子：

```kt
Service adbd /sbin/adbd
  class core
  socket adbd stream 660 system system
  disabled
  seclabel u:r:adbd:s0
```

那么为什么有些使用动态转换，而另一些使用`seclabel`呢？答案取决于你从哪里执行。像`adbd`这样的东西很早就从 ramdisk 中执行，因为 ramdisk 实际上不使用每个文件的标签，所以你不能正确设置转换——目标具有相同的上下文。

# 重新标记进程

既然我们现在拥有了动态进程转换功能，而且需要从初始化脚本中设置套接字上下文。让我们尝试重新标记那些处于不正确上下文中的服务。我们可以通过以下规则检查它们是否不正确：

+   除了 init，不应该有其他进程处于初始化上下文

+   没有长时间运行的进程应该处于`init_shell`域

+   除了 zygote，不应该有其他进程处于 zygote 域

### 注意

一个更全面的测试套件是 AOSP 上的 CTS 的一部分。更多详细信息请参考 Android CTS 项目：（git clone）[`android.googlesource.com/platform/cts`](https://android.googlesource.com/platform/cts)。注意`./hostsidetests/security/src/android/cts/security/SELinuxHostTest.java`和`./tests/tests/security/src/android/security/cts/SELinux.*.java`测试。

让我们运行一些基本的命令，并通过`adb`连接评估我们的 UDOO 的状态：

```kt
$ adb shell ps -Z | grep init
u:r:init:s0 root 1 0 /init
u:r:init:s0 root 2267 1 /sbin/watchdogd
u:r:init_shell:s0 root 2278 1 /system/bin/sh
$ adb shell ps -Z | grep zygote
u:r:zygote:s0 root 2285 1 zygote

```

我们有两个进程处于不正确的域中。第一个是`watchdogd`，第二个是`sh`进程。我们需要找到这些进程并将它们纠正。

我们将从神秘的`sh`程序开始。正如你在上一章中可以回忆起，我们的 UDOO 串行控制台进程具有`init_shell`的上下文，所以这是一个很好的嫌疑对象。让我们检查 PID 并找出。从 UDOO 串行控制台执行：

```kt
root@udoo:/ # echo $$ 
2278

```

我们可以将这个 PID 与`adb shell ps`输出的 PID 字段（PID 字段是第三个字段，索引为 2）进行比较，正如你所看到的，我们有一个匹配项。

接下来，我们需要找到这个服务的声明。我们知道它在`init.rc`中，因为它运行在`init_shell`中，根据 SELinux 策略，只能由 init 直接转换到这种类型的运行状态。另外，init 只通过服务声明开始处理事情，所以为了处于`init_shell`状态，你必须通过服务声明由 init 启动。

### 注意

使用`sesearch`查找编译后的 sepolicy 二进制文件上的此类信息：

```kt
$ sesearch -T -s init -t shell_exec -c process $OUT/root/sepolicy

```

如果我们在`udoo/device/fsl/imx6/etc`中的 UDOO 的`init.rc`文件中搜索`/system/bin/sh`这个有疑问的命令，可以使用`grep`来查找其内容。如果我们这样做，我们会发现：

```kt
$ grep -n "/system/bin/sh" init.rc 
499:service console /system/bin/sh
702:service wifi_mac /system/bin/sh /system/etc/check_wifi_mac.sh

```

让我们看看`499`，因为我们对 Wi-Fi 没有涉及任何事情：

```kt
service console /system/bin/sh
 class core
 console
 user root
 group root

```

如果这就是问题服务，我们应该能够禁用它，并验证我们的串行连接不再工作：

```kt
$ adb shell setprop ctl.stop console

```

我的实时串行连接在以下位置断开：

```kt
root@udoo:/ # avc: denied { set } for property=ctl.console scontext=u:r:shell:s0 tcontext=u:e

```

现在我们已经验证了它是什么，我们可以重新启动它：

```kt
$ adb shell setprop ctl.start console

```

当系统恢复到工作状态后，我们现在需要解决修正此服务标签的最佳方法。我们有两个选项：

+   在`init.rc`中使用明确的`seclabel`条目

+   使用类型转换

我们在这里将使用第一个选项。原因是 init 会不时执行 shell，我们不希望所有这些都在 console 进程域中。我们希望最小权限来隔离运行中的进程。通过使用明确的 seclabel，我们不会改变沿途中执行的其他 shell。

为此，我们需要修改`init.rc`中关于 console 的条目；添加：

```kt
service console /system/bin/sh
 class core
 console
 user root
 group root
 seclabel u:r:shell:s0

```

此可执行文件适当的域是`shell`，因为它应该与`adb shell`具有相同的权限集。在您进行此更改后，重新编译引导映像，刷新，然后重新启动。我们可以看到它现在处于 shell 域中。要从 UDOO 串行连接中验证，执行以下操作：

```kt
root@udoo:/ # id -Z 
uid=0(root) gid=0(root) context=u:r:shell:s0

```

或者，使用`adb`执行以下命令：

```kt
$ adb shell ps -Z | grep "system/bin/sh"
u:r:shell:s0 root 2279 1 /system/bin/sh

```

下一个我们需要处理的是`watchdogd`。`watchdogd`进程已经有了一个域，并且在`watchdog.te`中允许规则；所以我们只需要添加一个`seclabel`语句并将其放入适当的域中。修改`init.rc`：

```kt
# Set watchdog timer to 30 seconds and pet it every 10 seconds to get a 20 second margin
service watchdogd /sbin/watchdogd 10 20
  class core
  seclabel u:r:watchdogd:s0
```

要使用`adb`验证，执行以下命令：

```kt
$ adb shell ps -Z | grep watchdog
u:r:watchdogd:s0 root 2267 1 /sbin/watchdogd

```

在这一点上，我们已经对 UDOO 需要的实际策略进行了更正。然而，我们需要练习使用动态域转换。一个好的教学示例应该有一个在其自己域中的 shell 的子 shell。让我们从定义一个新域并设置转换开始。

我们将在`sepolicy`中创建一个名为`subshell.te`的新`.te`文件，并编辑其内容如下：

```kt
type subshell, domain, shelldomain, mlstrustedsubject;
# domain_auto_trans(olddomain, type, newdomain)
# Automatically transition from olddomain to newdomain
# upon executing a file labeled with type.
#
domain_auto_trans(shell, shell_exec, subshell)

```

现在，本书前面使用的`mmm`技巧可以用来仅编译策略。同时，使用`adb push`命令将新策略推送到`/data/security/current/sepolicy`，并执行`setprop`以重新加载策略，正如我们在第八章 *将上下文应用于文件*中所做的那样。

为了测试这一点，我们应该能够输入`sh`，并验证域转换。我们将从获取当前上下文开始：

```kt
root@udoo:/ # id -Z
uid=0(root) gid=0(root) context=u:r:shell:s0

```

然后通过执行以下命令来启动一个 shell：

```kt
root@udoo:/ # sh
root@udoo:/ # id -Z
uid=0(root) gid=0(root) context=u:r:subshell:s0

```

我们能够使用动态类型转换让一个新进程进入一个域。如果你将此与第八章中提出的给文件打标签相结合，你就有了一个强大的工具来控制进程权限。

# 对应用标签的限制

这些动态进程转换的一个基本限制是它们需要一个`exec()`系统调用来执行。只有这样，SELinux 才能计算出新域，并触发上下文切换。唯一的其他方法是通过修改代码，本质上当你指定`seclabel()`时，init 就是这样做的。init 代码为其进程设置了执行上下文，导致下一次`exec`进入指定的域。实际上，我们可以在`init.c`代码中看到这一点：

```kt
if (svc->seclabel) {
if (is_selinux_enabled() > 0 && setexeccon(svc->seclabel) < 0) {
ERROR("cannot setexeccon('%s'): %s\n", svc->seclabel, strerror(errno));
_exit(127);
}
}
```

在这里，子进程通过调用`setexeccon()`设置了其执行上下文，在`exec()`系统调用将控制权交给新的二进制映像之前。在安卓中，应用程序不是以这种方式生成的，并且在进程创建路径中不存在`exec()`系统调用；因此需要一个新的机制。

# 概述

在本章中，我们学习了如何通过类型转换以及通过`seclabel`语句来标记进程。我们还研究了 init 如何管理服务套接字，以及如何正确标记它们。然后，我们修正了串行控制台以及看门狗守护进程的进程上下文。

安卓中的应用程序在启动程序执行时，永远不会显式调用`exec()`。由于没有`exec()`，我们必须通过代码更改来标记应用程序。在下一章中，我们将介绍这是如何发生的，以及应用程序是如何被标记的。
