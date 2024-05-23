# 第六章：探索 SELinuxFS

在前面的几章中，我们看到 SELinuxFS 在许多场合出现。从它在 `/proc/filesystems` 中的条目到 init 守护进程中的策略加载，在启用了 SELinux 的系统中经常使用。SELinuxFS 是内核到用户空间的接口，也是构建更高用户空间习惯用法和 `libselinux` 的基础。在本章中，我们将探索这个文件系统的功能，以更深入地了解系统的工作原理。具体来说，我们将：

+   确定如何找到 SELinux 文件系统的挂载点

+   提取有关我们当前 SELinux 系统状态的信息

+   在 shell 中即时修改我们的 SELinux 系统状态，并通过代码进行修改

+   调查 ProcFS 接口

# 定位文件系统

我们需要做的第一件事是定位文件系统的挂载点。`libselinux` 在两个地方之一挂载文件系统：默认为 `/selinux` 或 `/sys/fs/selinux`。然而，这不是一个严格的要求，可以通过调用 void `set_selinuxmnt(char *mnt)` 来更改，它设置 SELinux 挂载点的位置。然而，在大多数情况下，这应该发生，不需要任何调整。

在系统中找到挂载点的最佳方式是运行 mount 命令并找到文件系统的位置。在串行控制台，发出以下命令：

```kt
root@udoo:/ # mount | grep selinux
selinuxfs /sys/fs/selinux selinuxfs rw,relatime 0 0

```

如你所见，挂载点是 `/sys/fs/selinux`。让我们通过在串行终端提示符下发出以下命令，前往那个目录：

```kt
root@udoo:/ # cd /sys/fs/selinux
root@udoo:/sys/fs/selinux #

```

你现在处于 SELinux 文件系统的根目录。

# 询问文件系统

你可以询问 SELinuxFS 以找出内核支持的最高策略版本是什么。当你开始使用不是从源代码构建的系统时，这很有用。当你没有直接访问 KConfig 文件时也很有用。需要注意的是，DAC 和 MAC 权限都适用于此文件系统。关于 MAC 和 SELinux，这方面的访问向量在策略文件 `external/sepolicy/access_vectors` 中的 security 类别中枚举：

```kt
root@udoo:/sys/fs/selinux # echo 'cat policyvers'
23

```

### 提示

在前面的命令中，以及接下来几个命令中，我们不仅仅使用 `cat` 命令打印文件。这是因为这些文件在文件末尾没有换行符。没有换行符，命令执行后的命令提示符将位于输出最后一行的末尾。将 `cat` 命令用 `echo` 包裹可以保证有换行符。获取同样效果的另一种方法是使用 `cat policyvers ; echo`。

正如我们所预期的，支持的版本是 23。正如你所记得的，我们在 第四章 *在 UDOO 上安装* 中配置内核以使用 `make menuconfig` 启用 SELinux 时设置了此值，从 `kernel_imx` 目录中。这也可以通过 `libselinux` API 访问：

```kt
int security_policyvers(void);

```

它不应需要任何提升的权限，系统上的任何人都可以读取。

## 强制节点

在前面的章节中，我们讨论了 SELinux 在两种模式下运行，**强制**和**宽容**。这两种模式都会记录策略违规，但是强制模式会导致内核拒绝访问资源，并向调用用户空间进程返回错误（例如，`EACCESS`）。SELinuxFS 有一个接口来查询此状态——文件节点`enforce`。从该文件读取会根据我们是运行在宽容模式还是强制模式，返回状态`0`或`1`：

```kt
root@udoo:/sys/fs/selinux # echo 'cat enforce' 
0

```

如你所见，我们的系统处于宽容模式。Android 有一个 toolbox 命令用于打印此状态。这个命令会根据我们是运行在宽容模式还是强制模式，返回`Permissive`或`Enforcing`状态：

```kt
root@udoo:/sys/fs/selinux # getenforce
Permissive

```

你也可以写入到`enforce`文件。此文件系统的 DAC 权限为：

```kt
Owner: root read, write
Group: root read
Others: read

```

任何人都可以获取强制状态，但要设置它，你必须要是 root 用户。进行此操作所需的 MAC 权限为：

```kt
class: security 
vector: setenforce

```

一个名为`setenforce`的命令可以更改此状态：

```kt
root@udoo:/sys/fs/selinux # setenforce 0

```

要查看命令的作用，可以在`strace`中运行它：

```kt
root@udoo:/sys/fs/selinux # strace setenforce 0

...
open("/proc/self/task/3275/attr/current", O_RDONLY) = 4
brk(0x41d80000) = 0x41d80000
read(4, "u:r:init_shell:s0\0", 4095) = 18
close(4) = 0
open("/sys/fs/selinux/enforce", O_RDWR) = 4
write(4, "0", 1) 
...

```

如我们所见，写入`enforce`的接口非常简单，只需写入`0`或`1`。在`libselinux`中执行此操作的功能是`int security_setenforce(int value)`。上述命令的另一个有趣之处是我们可以看到访问了`procfs`。SELinux 在`procfs`中也有一些额外的条目。这些将在本章中进一步介绍。

## 禁用文件接口

在运行时，也可以使用`disable`文件接口禁用 SELinux。但是，内核必须使用`CONFIG_SECURITY_SELINUX_DISABLE=y`进行构建。我们的内核没有使用此选项构建。此文件只能由所有者写入，并且没有与之关联的特定 MAC 权限。我们建议保持此选项禁用。此外，可以在加载策略之前禁用 SELinux。即使启用了该选项，一旦加载了策略，它也会被禁用。

## 策略文件

`policy`文件允许你读取当前加载到内核中的 SELinux 策略文件。这可以读取并保存到磁盘：

```kt
root@udoo:/sys/fs/selinux # cat policy > /sdcard/policy

```

通过启用`adb`接口，你现在可以从设备中提取它，并在宿主上使用标准的 SELinux 工具进行分析。此文件的 DAC 权限为所有者：`root`，`read`。对此文件没有特定的 SELinux 权限。

`policy`文件的对应文件是`load`文件。我们已经看到当通过`libselinux` API 加载策略文件时，会出现这个文件：

```kt
int security_load_policy(void *data, size_t len);

```

## `null`文件

当域转换发生时，SELinux 使用`null`文件来重定向未授权的文件访问。记住，域转换是指从一种上下文转换到另一种上下文。在大多数情况下，这发生在程序执行 fork 和 exec 函数时，但也可能是程序化发生的。在任一情况下，进程都有无法再访问的文件引用，为了帮助防止进程崩溃，它们只需从 SELinux 空设备写入/读取。

## `mls`文件

我们系统的一个功能是当前策略正在使用**多级安全**（**MLS**）支持。这是基于加载的策略文件是否使用它，要么是`0`要么是`1`。由于我们已经启用它，我们预计会从这个文件看到`1`：

```kt
root@udoo:/sys/fs/selinux # echo 'cat mls'
1

```

`mls`文件对所有用户可读，并有一个相应的 SELinux API：

```kt
int is_selinux_mls_enabled(void)

```

## 状态文件

`version`文件允许你了解 SELinux 内部发生的更新。例如，当策略重新加载时。一个**用户空间对象管理器**可以缓存决策结果，并使用`reload`事件作为触发器来刷新其缓存。`status`文件是只由所有人读取的，没有特定的 MAC 权限。`libselinux` API 接口是：

```kt
int selinux_status_open(int fallback);
void selinux_status_close();
int selinux_status_updated(void);
int selinux_status_getenforce(void);
int selinux_status_policyload(void);
int selinux_status_deny_unknown(void);

```

通过检查状态结构，你可以检测变化并刷新缓存。然而，目前你的`libselinux`中缺少这个 API，但我们在第七章，*利用审计日志*中会纠正这个问题。

文件树中有许多 SELinuxFS 文件；我们这里只介绍了几个文件，因为它们的重要性或与我们已做工作及未来方向的相关性。我们没有涵盖：

+   `access`

+   `checkreqprot`

+   `commit_pending_bools`

+   `context`

+   `create`

+   `deny_unknown`

+   `member`

+   `reject_unknown`

+   `relabel`

使用这些文件并不简单，通常是由使用`libselinux` API 的用户空间对象管理器来完成，以抽象化复杂性。

## 访问向量缓存

SELinuxFS 还有一些你可以探索的目录。第一个是`avc`。它代表“访问向量缓存”，可以用来获取内核中安全服务器统计信息：

```kt
root@udoo:/sys/fs/selinux # cd avc/
root@udoo:/sys/fs/selinux/avc # ls
cache_stats
cache_threshold
hash_stats

```

所有这些文件都可以使用`cat`命令读取：

```kt
root@udoo:/sys/fs/selinux/avc # cat cache_stats
lookups hits misses allocations reclaims frees
285710 285438 272 272 128 128
245827 245409 418 418 288 288
267511 267227 284 284 192 193
214328 213883 445 445 288 298

```

`cache_stats`文件对所有用户可读，不需要特殊的 MAC 权限。

下一个要查看的文件是`hash_stats`：

```kt
root@udoo:/sys/fs/selinux/avc # cat hash_stats
entries: 512
buckets used: 284/512
longest chain: 7

```

访问向量缓存的基础数据结构是一个哈希表；`hash_stats`列出了当前的属性。从前一条命令的输出中可以看出，表中我们有 512 个槽位，其中 284 个正在使用中。在冲突处理中，最长的链有 7 个条目。这个文件是全局可读的，不需要特殊的 MAC 权限。你可以通过`cache_threshold`文件修改此表中的条目数。

`cache_threshold`文件用于调整`avc`哈希表中的条目数。它是全局可读的，所有者可写的。它需要 SELinux 权限`setsecparam`，并且可以使用以下简单的命令分别进行写入和读取：

```kt
root@udoo:/sys/fs/selinux/avc # echo "1024" > cache_threshold 
root@udoo:/sys/fs/selinux/avc # echo 'cat cache_threshold'
1024

```

你可以通过写入`0`来禁用缓存。然而，在基准测试之外，这不鼓励这样做。

## 布尔值目录

第二个要查看的目录是`booleans`。SELinux 的`boolean`允许策略声明通过`boolean`条件动态更改。通过改变`boolean`的状态，您可以影响已加载策略的行为。当前策略没有定义任何布尔值；因此这个目录是空的。在定义了布尔值的策略中，该目录将填充以每个布尔值命名的文件。然后，您可以读取和写入这些文件来改变`boolean`的状态。Android 工具箱已经进行了修改，包含了`getsebool`和`setsebool`命令。`libselinux` API 也公开了这些功能：

```kt
int security_get_boolean_names(char ***names, int *len);
int security_get_boolean_pending(const char *name);
int security_get_boolean_active(const char *name);
int security_set_boolean(const char *name, int value);
int security_commit_booleans(void);
int security_set_boolean_list(size_t boolcnt, SELboolean * boollist, int permanent);

```

布尔值是事务性的。这意味着它是一组“全有或全无”的设置。当您使用`security_set_boolean*`时，必须调用`security_commit_booleans()`使其生效。与 Linux 桌面系统不同，永久布尔值是不支持的。更改运行时值不会在重启后保留。另外，在 Android 上，如果您尝试达到 Android **兼容性测试套件** (**CTS**) 的合规性，布尔值将导致测试失败。布尔值可以根据目标具有不同的 DAC 权限，但它们总是需要 SELinux 权限，即`setbool`。

### 提示

您必须通过 Android 兼容性测试套件才能使用 Android 品牌。关于 CTS 的更多信息可以在[`source.android.com/compatibility/cts-intro.html`](https://source.android.com/compatibility/cts-intro.html)找到。

## 类目录

下一个要查看的目录是`class`。`class`目录包含了在`access_vectors` SELinux 策略文件中定义的所有类，或者通过 SELinux 策略语言中的`class`关键字定义的类。对于策略中定义的每个类，都存在一个同名的目录。例如，在串行终端运行以下命令：

```kt
root@udoo:/sys/fs/selinux/class # ls -la
...
dr-xr-xr-x root root 1970-01-02 01:58 peer
dr-xr-xr-x root root 1970-01-02 01:58 process
dr-xr-xr-x root root 1970-01-02 01:58 property_service
dr-xr-xr-x root root 1970-01-02 01:58 rawip_socket
dr-xr-xr-x root root 1970-01-02 01:58 security
...

```

如您从前面的命令中看到的，有不少目录。让我们检查一下`property_service`目录。选择这个目录是因为它在 Android 上只有一个定义。然而，每个目录中存在的文件是相同的，包括`index`和`perms`：

```kt
root@udoo:/sys/fs/selinux/class/property_service # ls
index
perms

```

字符串和 SELinux 内核模块中定义的某些任意整数之间的映射是`index`。包含该类的所有可能权限的目录是`perms`：

```kt
root@udoo:/sys/fs/selinux/class/property_service # cd perms/
root@udoo:/sys/fs/selinux/class/property_service/perms # ls
set

```

如您所见，`property_service`类中可使用`set`访问向量。`class`目录可以非常有助于观察系统中已加载的策略文件。

## `initial_contexts`目录

下一个要查看的目录条目是`initial_contexts`。这是初始安全上下文的静态映射，更广为人知的是**安全标识符**（**sid**）。这个映射告诉 SELinux 系统应该使用哪个上下文来启动每个内核对象：

```kt
root@udoo:/sys/fs/selinux/initial_contexts # ls
any_socket
devnull
file
...

```

我们可以通过执行以下操作来查看`file`的初始 sid：

```kt
root@udoo:/sys/fs/selinux/initial_contexts # echo 'cat file'
u:object_r:unlabeled:s0

```

这对应于`external/sepolicy/initial_sid_contexts`中的条目：

```kt
...
sid file u:object_r:unlabeled:s0
...
```

## `policy_capabilities`目录

最后需要查看的目录是`policy_capabilities`。这个目录定义了策略可能具有的任何附加功能。对于我们当前的设置，我们应该有：

```kt
root@udoo:/sys/fs/selinux/policy_capabilities # ls
network_peer_controls
open_perms

```

每个文件条目都包含一个布尔值，指示功能是否启用：

```kt
root@udoo:/sys/fs/selinux/policy_capabilities # echo 'cat open_perms'
1

```

这些条目对所有人可读，对任何人不可写。

## ProcFS

我们之前提到了一些正在导出的 procfs 接口。讨论的大部分内容是安全上下文，这意味着 shell 应该与某些安全上下文相关联...但我们应该如何实现这一点？由于这是所有 LSMs 使用的通用机制，因此安全上下文通过 procfs 进行读取和写入：

```kt
root@udoo:/sys/fs/selinux/policy_capabilities # echo 'cat /proc/self/attr/current'
u:r:init_shell:s0

```

你也可以获取每个线程的上下文：

```kt
root@udoo:/sys/fs/selinux/policy_capabilities # echo '/proc/self/task/2278/attr/current'
u:r:init_shell:s0

```

只需将`2278`替换为你想要的线程 ID。

当前文件上的 DAC 权限对所有人都是读写权限，但这些文件通常受到 MAC 权限的严格限制。通常，只有拥有 procfs 条目的进程可以读取这些文件，并且你必须拥有标准的写权限以及`setcurrent`的组合权限。注意，使用**dyntransition**必须允许“从”和“到”域。要读取，你必须拥有`getattr`。所有这些权限都来自`process`安全类。`libselinux` API 函数`getcon`和`setcon`允许你操作`current`。

`prev`文件可以用来查找你之前切换的上下文。这个文件是不可写的：

```kt
root@udoo:/proc/self/attr # echo 'cat prev'
u:r:init:s0

```

我们串行终端的前一个域或安全上下文是`u:r:init:s0`。

`exec`文件用于为子进程设置标签。这在进行 exec 之前设置。所有这些文件的权限与实际设置它们时使用的 MAC 权限相同。尝试设置此项的调用者还必须持有来自`process`类的`setexec`。可以使用 libselinux API `int setexeccon(security_context_t context)`和`int getexeccon(security_context_t *context)`来设置和检索标签。

`fscreate`、`keycreate`和`sockcreate`文件执行类似操作。当进程创建任何对应的对象时，如`fs`对象（文件、命名管道或其他对象）、密钥或套接字，这里设置的值将被使用。调用者还必须持有来自`process`类的`setfscreate`、`setsockcreate`和`setkeycreate`。以下 SELinux API 用于更改这些：

```kt
int set*createcon(security_context_t context);
int get*createcon(security_context_t *con);

```

其中`*`可以是`fs`、`key`或`socket`。

需要注意的是，这些特殊的`process`类权限可以让你更改`proc/attr`文件。你仍然需要通过 DAC 权限以及文件对象上设置的任何 SELinux 权限。只有在完成这些之后，你才需要额外的权限，如`setfscreate`。

# Java SELinux API

对于之前讨论的 C API，Java 也有类似的 API。在这种情况下，假设你将使用平台来构建代码，因为这些 API 并未随 Android SDK 一起公开提供。该 API 位于`frameworks/base/core/java/android/os/SELinux.java`。然而，这只是 API 的一个非常有限的部分。

# 总结

在本章中，我们探讨了内核与用户空间之间关于 SELinux 的接口，并强化了访问向量类和安全上下文的概念。在下一章中，我们将对我们的系统进行一些升级，并查看审计日志，使我们离最终目标更近一步——在 SELinux 强制模式下可操作的设备。我们之所以说它是可操作的，是因为我们现在可以将其设置为强制模式。然而，如果你现在通过在 UDOO 上执行`setenforce 1`这样做，你的设备可能会变得不稳定。例如，在我们的系统上，如果我们这样做，浏览器将无法启动。
