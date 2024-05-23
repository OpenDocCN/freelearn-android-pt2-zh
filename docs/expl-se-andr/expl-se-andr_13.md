# 第十三章：进入强制模式

作为一名工程师，你拿到一部 Android 设备，要求你应用 SE for Android 控制来增强设备的安全态势。到目前为止，我们已经看到了需要配置的所有部分以及它们如何工作来启用这样一个系统。在本章中，我们将运用所学的所有技能，让 UDO 进入强制模式。我们将：

+   运行、评估并响应来自 CTS 的审核日志

+   为 UDO 开发安全策略

+   切换到强制模式

# 更新到 SEPolicy master

自 4.3 版本以来，AOSP `master`分支的 `sepolicy` 目录发生了许多变化。在撰写本文时，`external/sepolicy`项目的 `master` 分支在 Git 提交 SHA `b5ffb`上。作者建议尝试使用最新的提交。然而，为了说明目的，我们将展示如何选择性地检出提交 `b5ffb`，以便你可以准确地跟随本章中的示例。

首先，你需要克隆 `external/sepolicy` 项目。在这些说明中，我们假设你的工作目录中包含 UDO 源码的 `./udoo` 目录：

```kt
$ git clone https://android.googlesource.com/platform/external/sepolicy
$ cd sepolicy

```

如果你想要精确地跟随本章的示例，你需要使用以下命令检出提交 `b5ffb`。如果你跳过这一步，你最终将使用 `master` 分支中的最新提交：

```kt
$ git checkout b5ffb

```

现在，我们将用从谷歌获取的 sepolicy 替换 UDO 4.3 的 sepolicy：

```kt
$ cd ..
$ rm -rf udoo/external/sepolicy
$ cp -r sepolicy udoo/external/sepolicy

```

可选地，你可以使用以下命令从新复制的 sepolicy 中删除 `.git` 文件夹，但这不是必须的：

```kt
$ rm –rf udoo/external/sepolicy/.git

```

同时，复制 `audit.te` 文件并恢复它。

此外，从 NSA Bitbucket `seandroid` 仓库恢复 `auditd` 提交。作为参考，它的提交 SHA 是 `d270aa3`。

之后，从 `udoo/build/core/Makefile` 中删除所有对 `setool` 的引用。这个命令将帮助你找到它们：

```kt
$ grep -nw setool udoo/build/core/Makefile

```

# 清除设备

在这一点上，我们的 UDO 很混乱，所以让我们重新刷新它，包括数据目录，并重新开始。我们希望只有代码和 `init` 脚本更改，而没有额外的 sepolicy。然后我们可以适当地编写一个策略，并应用我们遇到的所有的技术和工具。我们将从重置到一个类似于完成第四章，*UDO 上的安装*的状态开始。然而，主要区别是我们需要构建一个 `userdebug` 版本而不是工程 (`eng`) 版本用于 CTS。版本在设置脚本中选择，最终调用 `lunch`。要构建此版本，请从 UDO 工作区执行以下命令：

```kt
$ . setup udoo-userdebug
$ make -j8 2>&1 | tee logz

```

使用以下命令刷新系统，引导到 SD 卡，并清除 `userdata`，假设 SD 卡已插入主机且 `userdata` 未挂载：

```kt
$ mkdir ~/userdata
$ sudo mount /dev/sdd4 ~/userdata
$ cd ~/userdata/
$ sudo rm -rf *
$ cd ..
$ sudo umount ~/userdata

```

# 设置 CTS

如果你的组织寻求 Android 品牌认证，你必须通过 CTS。然而，即使你不是，运行这些测试也是一个好主意，以确保设备将符合应用程序的要求。根据你的安全目标和愿望，如果你不是在寻求 Android 品牌认证，你可能会在 CTS 的某些部分失败。对于我们的情况，我们将 CTS 视为一种锻炼系统并发现阻止 UDOO 正常工作的政策问题的方式。其源代码位于 `cts/` 目录中，但我们建议直接从 Google 下载二进制文件。你可以从 [`source.android.com/compatibility/cts-intro.html`](https://source.android.com/compatibility/cts-intro.html) 和 [`source.android.com/compatibility/android-cts-manual.pdf`](https://source.android.com/compatibility/android-cts-manual.pdf) 获取更多信息及 CTS 二进制文件本身。

从 **下载** 选项卡下载 CTS 4.3 二进制文件。然后选择 CTS 二进制文件。**兼容性定义文档**（**CDD**）也值得一读。它涵盖了 CTS 和兼容性要求的高级详细信息。

从 [`source.android.com/compatibility/downloads.html`](https://source.android.com/compatibility/downloads.html) 下载 CTS 并解压。选择与你的 Android 版本匹配的 CTS 版本。如果你不知道你的设备正在运行哪个版本，你可以通过 UDOO 使用 `getprop ro.build.version.release` 命令检查 `ro.build.version.release` 属性：

```kt
$ mkdir ~/udoo-cts
$ cd ~/udoo-cts
$ wget https://dl.google.com/dl/android/cts/android-cts-4.3_r2-linux_x86-arm.zip
$ unzip android-cts-4.3_r2-linux_x86-arm.zip

```

# 运行 CTS

CTS 对设备上的许多组件进行锻炼，并帮助测试系统的各个部分。一个好的通用政策应允许 Android 正常工作并通过 CTS。

按照 Android CTS 用户手册中的说明设置你的设备（见 *第 3.3 节*，*设置你的设备*）。通常，如果你没有精确地遵循所有步骤，你可能会看到一些失败，因为你可能无法获取到所有需要的资源或权限。然而，CTS 仍然会执行一些代码路径。至少，我们建议复制媒体文件并激活 Wi-Fi。设置好设备后，确保 `adb` 处于活动状态并启动测试：

```kt
$ ./cts-tradefed
11-30 10:30:08 I/: Detected new device 0123456789ABCDEF
cts-tf > run cts --plan CTS
cts-tf > 
time passes here
11-30 10:30:28 I/TestInvocation: Starting invocation for 'cts' on build '4.3_r2' on device 0123456789ABCDEF
11-30 10:30:28 I/0123456789ABCDEF: Created result dir 2014.11.30_10.30.28
11-30 10:31:44 I/0123456789ABCDEF: Collecting device info
11-30 10:31:45 I/0123456789ABCDEF: -----------------------------------------
11-30 10:31:45 I/0123456789ABCDEF: Test package android.aadb started
11-30 10:31:45 I/0123456789ABCDEF: -----------------------------------------
11-30 10:32:15 I/0123456789ABCDEF: com.android.cts.aadb.TestDeviceFuncTest#testBugreport PASS 
...

```

测试需要执行很多小时，所以请耐心等待；但你可以检查测试的状态：

```kt
cts-tf > l i
Command Id  Exec Time Device State 
1 8m:22 0123456789ABCDEF running cts on build 4.3_r2 

```

插上扬声器，享受来自媒体测试和铃声的声音！同时，CTS 会重启设备。如果你的 ADB 会话在重启后没有恢复，ADB 可能不会执行任何测试。在运行 `cts-tf > run cts --plan CTS --disable-reboot` 计划时，使用 `--disable-reboot` 选项。

# 收集结果

首先，我们将考虑 CTS 的结果。虽然我们预计会有一些失败，但我们也预计在进入强制模式时问题不会变得更糟。其次，我们将查看审计日志。让我们从设备中提取这两个文件。

## CTS 测试结果

每次运行 CTS 时，它都会创建一个测试结果目录。CTS 指出了目录名称，但未指出位置：

```kt
11-30 10:30:28 I/0123456789ABCDEF: Created result dir 2014.11.30_10.30.28

```

CTS 手册中提到了这个位置，可以在提取的 CTS 目录下的`repository/results`中找到，通常在`android-cts/repository/results`。测试目录包含一个 XML 测试报告，`testResult.xml`。这可以在大多数网络浏览器中打开，并且有一个很好的测试概览以及所有执行测试的详细信息。`通过:失败`的比例是我们的基线。作者有 18,736 个通过，只有 53 个失败，考虑到其中一半是功能问题，比如没有蓝牙或对摄像头支持返回真，这个结果相当不错。

## 审计日志

我们将使用审计日志来解决我们策略中的不足。使用本书中一直使用的标准`adb pull`命令从设备上提取这些日志。由于这是一个`userdebug`版本，默认的`adb`终端是 shell `uid`（不是 root），因此以 root 身份启动`adb`，使用`adb root`命令。在`userdebug`版本上也可以使用`su`。

### 提示

你可能会遇到错误提示`/data/misc/audit/audit.log`不存在。解决方案是使用`adb root`命令以 root 身份运行`adb`。此外，在运行此命令时，它可能会挂起。只需进入设置，禁用，然后在**开发者选项**下重新启用**USB 调试**。然后杀死`adb-root`命令，并通过运行`adb shell`验证你是否具有 root 权限。现在你应该再次成为 root 用户了。

# 设备策略编写

通过`audit2allow`运行`audit.log`和`audit.old`，看看发生了什么。`audit2allow`的输出按源域名分组。我们不会全部查看，而是从`audit2allow`的解释结果开始，突出不寻常的情况。假设你目前在审计日志目录中，执行`cat audit.* | audit2allow | less`。所有策略工作将在设备特定的 UDOOPolicy 目录中进行。

## adbd

以下是通过`audit2allow`过滤的我们的`adbd`拒绝：

```kt
#============= adbd ==============
allow adbd ashmem_device:chr_file execute;
allow adbd dumpstate:unix_stream_socket connectto;
allow adbd dumpstate_socket:sock_file write;
allow adbd input_device:chr_file { write getattr open };
allow adbd log_device:chr_file { write read ioctl open };
allow adbd logcat_exec:file { read getattr open execute execute_no_trans };
allow adbd mediaserver:binder { transfer call };
allow adbd mediaserver:fd use;
allow adbd self:capability { net_raw dac_override };
allow adbd self:process execmem;
allow adbd shell_data_file:file { execute execute_no_trans };
allow adbd system_server:binder { transfer call };
allow adbd tmpfs:file execute;
allow adbd unlabeled:dir getattr;
```

`adbd`域中的拒绝相当奇怪。首先引起我们注意的是对字符驱动器`/dev/ashmem`的`execute`操作。通常，这只有在 Dalvik JIT 时才需要。查看原始审计（`cat audit.* | grep adbd | grep execute`），我们看到以下内容：

```kt
type=1400 msg=audit(1417416666.182:788): avc: denied { execute } for pid=3680 comm="Compiler" path=2F6465762F6173686D656D2F64616C76696B2D6A69742D636F64652D6361636865202864656C6574656429 dev=tmpfs ino=412027 scontext=u:r:adbd:s0 tcontext=u:object_r:tmpfs:s0 tclass=file
type=1400 msg=audit(1417416670.352:831): avc: denied { execute } for pid=3753 comm="Compiler" path="/dev/ashmem" dev=tmpfs ino=1127 scontext=u:r:adbd:s0 tcontext=u:object_r:ashmem_device:s0 tclass=chr_file

```

我们发现编译器中的`comm`字段进程正在`ashmem`上执行。我们猜测这与 Dalvik 有关，但为什么它会出现在`adbd`域中？还有，为什么`adbd`要写入输入设备？这些都是奇怪的行为。通常，当你看到这类事情时，是因为子进程没有进入正确的域。运行以下命令检查域以确认我们的怀疑：

```kt
$ adb shell ps -Z | grep adbd
u:r:adbd:s0 root 20046 1 /sbin/adbd
u:r:adbd:s0 root 20101 20046 ps

```

然后，我们运行`adb shell ps -Z | grep adbd`来查看哪些进程在`adb`域中运行，进一步证实了我们的怀疑：

```kt
u:r:adbd:s0 root 20046 1 /sbin/adbd
u:r:adbd:s0 root 20101 20046 ps

```

`ps`命令不应该在`adbd`上下文中运行；它应该在`shell`中运行。这证实了`shell`不在正确的域中：

```kt
$ adb shell
root@udoo:/ # id
uid=0(root) gid=0(root) context=u:r:adbd:s0

```

首先要检查的是文件上下文：

```kt
root@udoo:/ # ls -Z /system/bin/sh
lrwxr-xr-x root shell u:object_r:system_file:s0 sh -> mksh
root@udoo:/ # ls -Z /system/bin/mksh
-rwxr-xr-x root shell u:object_r:system_file:s0 mksh

```

基本策略定义了当`adbd`使用`exec`加载 shell 以进入 shell 域时的域转换。这在外部 sepolicy 的`adbd.te`中定义为`domain_auto_trans(adbd, shell_exec, shell)`。

显然，shell 被错误地标记了，因此我们来看看外部 sepolicy 中的`file_contexts`以找出原因。

```kt
$ cat file_contexts | grep shell_exec
/system/bin/sh -- u:object_r:shell_exec:s0

```

两个破折号意味着只有常规文件将被标记，而符号链接将被跳过。我们可能不想标记符号链接，而是`mksh`的目标。通过向设备 UDOO sepolicy 添加自定义`file_contexts`条目，并将文件添加到`BOARD_SEPOLICY_UNION`配置中来实现。在`file_contexts`中添加`/system/bin/mksh -- u:object_r:shell_exec:s0`，在`sepolicy.mk`中添加`BOARD_SEPOLICY_UNION += file_contexts`。

### 提示

在本章的剩余部分，每当你创建或修改策略文件（例如，上下文文件或`*.te`文件）时，别忘了将它们添加到`sepolicy.mk`中的`BOARD_SEPOLICY_UNION`。

由于这是策略和`adbd`的一个相当严重的问题，我们现在不担心拒绝，除了未标记的。每当遇到未标记的文件时，都应该处理。导致此问题的`avc`拒绝如下： 

```kt
type=1400 msg=audit(1417405835.872:435): avc: denied { getattr } for pid=4078 comm="ls" path="/device" dev=mmcblk0p7 ino=2 scontext=u:r:adbd:s0 tcontext=u:object_r:unlabeled:s0 tclass=dir

```

因为这被挂载在`/device`，而 Android 的挂载通常在`/`，我们应该查看挂载表：

```kt
root@udoo:/ # mount | grep device
/dev/block/mmcblk0p7 /device ext4 ro,seclabel,nosuid,nodev,relatime,user_xattr,barrier=1,data=ordered 0 0

```

通常，挂载命令位于`mkdir`之后的 init 脚本中，或者在带有内置 init 的`fstab`文件中，通过`mount_all`。在`init.rc`中快速搜索`device`和`mkdir`没有发现任何内容，但我们确实在`fstab.freescale`中找到了它。设备是只读的，因此我们应该能够为其指定类型，用文件上下文进行标记，并将其目录类别应用到`getattr`域。既然它是只读且为空的，没有人应该需要更多权限。查看`make_sd.sh`脚本，我们注意到块设备分区 7 是`vender`目录。这是 OEM 放置专有 blob 的常见 vendor 目录的拼写错误。我们在`file.te`中放置文件类型，并在`domain.te`中放置域允许规则。

在`file.te`中，添加以下内容：

```kt
type udoo_device_file, file_type;
```

在`domain.te`中，添加以下内容：

```kt
allow domain udoo_device_file:dir getattr;
```

在`file_contexts`中，添加以下内容：

```kt
/device u:object_r:udoo_device_file:s0
```

如果这个目录不为空，你必须手动运行`restorecon -R`来标记现有文件。

如果你多次从 UDOO 提取审计日志，你也可能看到拒绝显示你这样做，因为`adbd`将无法访问它们。你可能会看到如下内容：

```kt
#============= adbd ==============
allow adbd audit_log:file { read getattr open };
```

这条规则来自于当你通过`adb pull`提取审计日志的测试结束时。我们可以安全地`dontaudit`这个，并添加一个`neverallow`以确保它不会意外被允许。审计日志包含恶意软件作者可能用来导航策略的信息，这些信息应该受到保护。在设备的`sepolicy`文件夹中，添加一个`adbd.te`文件，并在`sepolicy.mk`文件中进行合并：

在`adbd.te`文件中，添加以下内容：

```kt
# dont audit adb pull and adb shell cat of audit logs
dontaudit adbd audit_log:file r_file_perms;
dontaudit shell audit_log:file r_file_perms;
```

在`auditd.te`中，添加以下内容：

```kt
# Make sure no one adds an allow to the audit logs
# from anything but system server (read only) and
# auditd, rw access.
neverallow { domain -system_server -auditd -init -kernel } audit_log:file ~getattr;
neverallow system_server audit_log:file ~r_file_perms;
```

如果`auditd.te`仍然在`external/sepolicy`中，将其移到`device/fsl/udoo/sepolicy`，并带上所有依赖的类型。

`neverallow`条目展示了如何使用补集`~`和集合差集`-`操作符进行强断言或简洁表达。第一个`neverallow`从域开始，所有进程类型（域）都是 domain 属性的成员。我们通过集合差集阻止访问，留下了永远不应该访问的集合。然后我们对访问向量集取补集，只允许在日志上执行`getattr`或`stat`。第二个`neverallow`使用补集确保`system_server`仅限于读取操作。

## bootanim

`bootanim`域被分配给启动动画服务，该服务在启动时显示启动画面，通常是运营商的品牌：

```kt
#============= bootanim ==============
allow bootanim init:unix_stream_socket connectto;
allow bootanim log_device:chr_file { write open };
allow bootanim property_socket:sock_file write;
```

任何接触`init`域的都是一个红旗。这里，`bootanim`连接到一个 init Unix 域套接字。这是属性系统的一部分，我们可以看到连接后，它会写入属性套接字。套接字对象和它的 URI 是分开的。在这种情况下，它是文件系统，但它可能是一个匿名套接字：

```kt
type=1400 msg=audit(1417405616.640:255): avc: denied { connectto } for pid=2534 comm="BootAnimation" path="/dev/socket/property_service" scontext=u:r:bootanim:s0 tcontext=u:r:init:s0 tclass=unix_stream_socket

```

在新版本的 Android 中，`log_device`已被弃用，并用`logd`替换。然而，我们将新的主 sepolicy 回移植到 4.3，因此我们必须支持这个。移除支持的补丁在[`android-review.googlesource.com/#/c/108147/`](https://android-review.googlesource.com/#/c/108147/)。

我们不需要对 external sepolicy 应用反向补丁，只需将规则添加到我们的设备策略中的`domain.te`文件中。我们可以安全地使用设备 UDO `sepolicy`文件夹中的正确宏和样式允许这些操作。在`bootanim.te`中，添加`unix_socket_connect(bootanim, property, init)`，在`domain.te`中，添加以下内容：

```kt
allow domain udoo_device_file:dir getattr;
allow domain log_device:dir search;
allow domain log_device:chr_file rw_file_perms;
```

## debuggerd

```kt
#============= debuggerd ==============
allow debuggerd log_device:chr_file { write read open };
allow debuggerd system_data_file:sock_file write;
```

通过为所有域添加允许规则来使用`log_device`，在`bootanim`下解决了日志设备的拒绝。`system_data_file:sock_file write`很奇怪。在大多数情况下，你几乎永远不会希望允许跨域写入，但这是一种特殊情况。看看原始拒绝：

```kt
type=1400 msg=audit(1417415122.602:502): avc: denied { write } for pid=2284 comm="debuggerd" name="ndebugsocket" dev=mmcblk0p4 ino=129525 scontext=u:r:debuggerd:s0 tcontext=u:object_r:system_data_file:s0 tclass=sock_file

```

拒绝是关于`ndebugsocket`的。通过搜索这个发现了一个命名类型转换，而策略版本 23 不支持：

```kt
system_server.te:297:type_transition system_server system_data_file:sock_file system_ndebug_socket "ndebugsocket";

```

我们必须更改代码以设置正确的上下文，或者直接允许它，我们将会这样做。我们不会授予额外的权限，因为它从未请求过 open，而且我们正在跨域操作。阻止跨域文件打开是理想的，因为获取这个文件描述符的唯一方式是通过 IPC 调用到拥有该域的进程中。在`debuggerd.te`中，添加`allow debuggerd system_data_file:sock_file write;`。

## drmserver

```kt
#============= drmserver ==============
allow drmserver log_device:chr_file { write open };
```

这由`domain.te`规则处理，所以我们这里不需要做任何事情。

## dumpstate

```kt
#============= dumpstate ==============
allow dumpstate init:binder call;
allow dumpstate init:process signal;
allow dumpstate log_device:chr_file { write read open };
allow dumpstate node:rawip_socket node_bind;
allow dumpstate self:capability sys_resource;
allow dumpstate system_data_file:file { write rename create setattr };
```

在`dumpstate`上对`init:binder call`的拒绝很奇怪，因为`init`并不使用 binder。某些进程必须停留在 init 域中。让我们检查一下我们的进程列表中关于 init 的部分：

```kt
$ adb shell ps -Z | grep init
u:r:init:s0 root 1 0 /init
u:r:init:s0 root 2286 1 zygote
u:r:init:s0 radio 2759 2286 com.android.phone

```

在这里，`zygote` 和 `com.android.phone` 不应该以 `init` 身份运行。这一定是 `app_process` 文件的标签错误，它是 zygote。执行 `ls -laZ /system/bin/app_process` 命令会显示 `u:object_r:system_file:s0 app_process`，因此需要在 `file_contexts` 中添加一个条目来纠正此错误。我们可以在基础 sepolicy 中的 `zygote.te` 文件找到要使用的标签，定义为 `zygote_exec` 类型：

```kt
# zygote
type zygote, domain;
type zygote_exec, exec_type, file_type;
```

在 `file_contexts` 中，添加 `/system/bin/app_process u:object_r:zygote_exec:s0`。

## 安装守护进程

添加的 `domain.te` 规则处理 `installd`。

## 密钥存储

```kt
#============= keystore ==============
allow keystore app_data_file:file write;
allow keystore log_device:chr_file { write open };
```

日志设备由 `domain.te` 规则处理。让我们看看原始的 `app_data_file` 拒绝：

```kt
type=1400 msg=audit(1417417454.442:845): avc: denied { write } for pid=15339 comm="onCtsTestRunner" path="/data/data/com.android.cts.stub/cache/CTS_DUMP" dev=mmcblk0p4 ino=131242 scontext=u:r:keystore:s0 tcontext=u:object_r:app_data_file:s0:c512,c768 tclass=file

```

类别在上下文中定义。这意味着激活了 `app domains` 的 MLS 支持。在 `seapp_contexts` 基础 sepolicy 中，我们看到以下内容：

```kt
user=_app domain=untrusted_app type=app_data_file levelFrom=user
user=_app seinfo=platform domain=platform_app type=app_data_file levelFrom=user
```

MLS 应用数据分离仍在开发中，在 4.3 上不起作用，因此我们可以禁用这个功能。我们只需在特定于设备的 `seapp_contexts` 文件中声明它们。在 `seapp_contexts` 中，添加 `user=_app domain=untrusted_app type=app_data_file` 和 `user=_app seinfo=platform domain=platform_app type=app_data_file`。在 4.3 版本中，对数据上下文的任何更改都需要进行工厂重置。4.4 版本增加了智能重新标签的功能。

## 媒体服务器

```kt
#============= mediaserver ==============
allow mediaserver adbd:binder { transfer call };
allow mediaserver init:binder { transfer call };
allow mediaserver log_device:chr_file { write open };
```

日志设备在 `domain.te` 规则中处理。我们将跳过 `init` 和 `adbd`，因为它们的问题是由不正确的进程域引起的。不要盲目添加允许规则，因为现有域的大部分工作可以通过小的标签更改或少数规则处理。

## 网络守护进程

```kt
#============= netd ==============
allow netd kernel:system module_request;
allow netd log_device:chr_file { write open };
```

`netd` 对日志设备的拒绝在 `domain.te` 中处理。然而，我们应该仔细审查任何请求能力的请求。在授予能力时，策略作者需要非常小心。如果一个域被授予加载系统模块的能力，而该域或模块二进制本身受到威胁，它可能导致通过可加载模块将恶意软件注入内核。然而，`netd` 需要可加载内核模块支持以支持某些卡片。在设备 UDOO sepolicy 中的名为 `netd.te` 的文件中添加允许规则。在 `netd.te` 中，添加 `allow netd self:capability sys_module;`。

## 无线设备守护进程

```kt
#============= rild ==============
allow rild log_device:chr_file { write open };
```

这由 `domain.te` 规则处理，所以我们在这里不需要做任何事情。

## 服务管理器

```kt
#============= servicemanager ==============
allow servicemanager init:binder transfer;
allow servicemanager log_device:chr_file { write open };
```

同样，日志设备在 `domain.te` 中处理。我们将跳过 `init`，因为其问题是由不正确的进程域引起的。

## 表面抛掷器

```kt
#============= surfaceflinger ==============
allow surfaceflinger init:binder transfer;
allow surfaceflinger log_device:chr_file { write open };
```

同样，日志设备在 `domain.te` 中处理。我们将跳过 `init`，因为其问题是由不正确的进程域引起的。

## 系统服务器

```kt
#============= system_server ==============
allow system_server adbd:binder { transfer call };
allow system_server dalvikcache_data_file:file { write setattr };
allow system_server init:binder { transfer call };
allow system_server init:file write;
allow system_server init:process { setsched sigkill getsched };
allow system_server init_tmpfs:file read;
allow system_server log_device:chr_file write;
```

由于 `log_device` 由 `domain.te` 处理，而 `init` 和 `adbd` 被污染，我们只处理 Dalvik 缓存拒绝：

```kt
type=1400 msg=audit(1417405611.550:159): avc: denied { write } for pid=2571 comm="er.ServerThread" name="system@app@SettingsProvider.apk@classes.dex" dev=mmcblk0p4 ino=129458 scontext=u:r:system_server:s0 tcontext=u:object_r:dalvikcache_data_file:s0 tclass=file
type=1400 msg=audit(1417405611.550:160): avc: denied { setattr } for pid=2571 comm="er.ServerThread" name="system@app@SettingsProvider.apk@classes.dex" dev=mmcblk0p4 ino=129458 scontext=u:r:system_server:s0 tcontext=u:object_r:dalvikcache_data_file:s0 tclass=file

```

外部 sepolicy seandroid-4.3 分支允许`domain.te:allow domain dalvikcache_data_file:file r_file_perms;`。`system_app`通过`system_app.te:allow system_app dalvikcache_data_file:file { write setattr };`允许写入。我们应该能够授予这种写入权限，因为可能需要更新其 Dalvik 缓存文件。在`domain.te`中，添加`allow domain dalvikcache_data_file:file r_file_perms;`，在`system_server.te`中，添加`allow system_server dalvikcache_data_file:file { write setattr };`。

## toolbox

```kt
#============= toolbox ==============
allow toolbox sysfs:file write;
```

通常，人们不应该写入`sysfs`。现在看看对违规`sysfs`文件的原始否认：

```kt
type=1400 msg=audit(1417405599.660:43): avc: denied { write } for pid=2309 comm="cat" path="/sys/module/usbtouchscreen/parameters/calibration" dev=sysfs ino=2318 scontext=u:r:toolbox:s0 tcontext=u:object_r:sysfs:s0 tclass=file

```

从这里，我们正确标记了`/sys/module/usbtouchscreen/parameters/calibration`。在`file_contexts`中添加一个条目来标记`sysfs`，在`file.te`中声明一个类型，并允许`toolbox`访问它。在`file.te`中，添加`type sysfs_touchscreen_calibration, fs_type, sysfs_type, mlstrustedobject;`，在`file_contexts`中，添加`/sys/module/usbtouchscreen/parameters/calibration -- u:object_r:sysfs_touchscreen_calibration:s0`，在`toolbox.te`中，添加`allow toolbox sysfs_touchscreen_calibration:file w_file_perms;`。

## 非可信应用

```kt
#============= untrusted_app ==============
allow untrusted_app adb_device:chr_file getattr;
allow untrusted_app adbd:binder { transfer call };
allow untrusted_app adbd:dir { read getattr open search };
allow untrusted_app adbd:file { read getattr open };
allow untrusted_app adbd:lnk_file read;
...
```

`untrusted_app`有许多否认。考虑到域标记问题，我们现在不会解决大多数问题。但是，你应该注意错误标记和未标记的目标文件。在使用`audit2allow`解释的否认日志中搜索时，发现了以下内容：

```kt
allow untrusted_app device:chr_file { read getattr };
allow untrusted_app unlabeled:dir { read getattr open };
```

对于`chr_file`设备，我们得到这个：

```kt
type=1400 msg=audit(1417416653.742:620): avc: denied { read } for pid=3696 comm="onCtsTestRunner" name="rfkill" dev=tmpfs ino=1126 scontext=u:r:untrusted_app:s0:c512,c768 tcontext=u:object_r:device:s0 tclass=chr_file
type=1400 msg=audit(1417416666.152:784): avc: denied { getattr } for pid=3696 comm="onCtsTestRunner" path="/dev/mxs_viim" dev=tmpfs ino=1131 scontext=u:r:untrusted_app:s0:c512,c768 tcontext=u:object_r:device:s0 tclass=chr_file
type=1400 msg=audit(1417416653.592:561): avc: denied { getattr } for pid=3696 comm="onCtsTestRunner" path="/dev/.coldboot_done" dev=tmpfs ino=578 scontext=u:r:untrusted_app:s0:c512,c768 tcontext=u:object_r:device:s0 tclass=file

```

因此，我们需要正确标记`/dev/.coldboot_done`，`/dev/rfkill`和`/dev/mxs_viim`。`/dev/rfkill`应该按照 4.3 政策的做法进行标记：

```kt
file_contexts:/sys/class/rfkill/rfkill[0-9]*/state -- u:object_r:sysfs_bluetooth_writable:s0
file_contexts:/sys/class/rfkill/rfkill[0-9]*/type -- u:object_r:sysfs_bluetooth_writable:s0
```

`/dev/mxs_viim`设备似乎是一个全局可访问的 GPU。我们建议彻底审查源代码，但现在，我们将它标记为`gpu_device`。`/dev/.coldboot_done`是在`coldboot`进程完成后由`ueventd`创建的。如果`ueventd`重新启动，它会跳过冷启动。我们不需要标记这个。这个否认是由源域 MLS 对目标文件造成的，该文件不是源类别子集，并且没有`mlstrustedsubject`属性；当我们从应用中删除 MLS 支持时，它应该消失。

在`file_contexts`中：

```kt
# touch screen calibration
/sys/module/usbtouchscreen/parameters/calibration -- u:object_r:sysfs_touchscreen_calibration:s0
#BT RFKill node
/sys/class/rfkill/rfkill[0-9]*/state -- u:object_r:sysfs_bluetooth_writable:s0
/sys/class/rfkill/rfkill[0-9]*/type -- u:object_r:sysfs_bluetooth_writable:s0
```

## vold

```kt
#============= vold ==============
allow vold log_device:chr_file { write open };
```

同样，日志设备在`domain.te`中被处理。

## watchdogd

```kt
#============= watchdogd ==============
allow watchdogd device:chr_file { read write create unlink open };
```

监管机构的原始否认描绘了一幅有趣的画像：

```kt
type=1400 msg=audit(1417405598.000:8): avc: denied { create } for pid=2267 comm="watchdogd" name="__null__" scontext=u:r:watchdogd:s0 tcontext=u:object_r:device:s0 tclass=chr_file
type=1400 msg=audit(1417405598.000:9): avc: denied { read write } for pid=2267 comm="watchdogd" name="__null__" dev=tmpfs ino=2580 scontext=u:r:watchdogd:s0 tcontext=u:object_r:device:s0 tclass=chr_file
type=1400 msg=audit(1417405598.000:10): avc: denied { open } for pid=2267 comm="watchdogd" name="__null__" dev=tmpfs ino=2580 scontext=u:r:watchdogd:s0 tcontext=u:object_r:device:s0 tclass=chr_file
type=1400 msg=audit(1417405598.000:11): avc: denied { unlink } for pid=2267 comm="watchdogd" name="__null__" dev=tmpfs ino=2580 scontext=u:r:watchdogd:s0 tcontext=u:object_r:device:s0 tclass=chr_file
type=1400 msg=audit(1417416653.602:575): avc: denied { getattr } for pid=3696 comm="onCtsTestRunner" path="/dev/watchdog" dev=tmpfs ino=1095 scontext=u:r:untrusted_app:s0:c512,c768 tcontext=u:object_r:watchdog_device:s0 tclass=chr_file

```

`watchdog`创建并解除了对一个匿名文件的引用。在 unlink 之后，不存在文件系统引用，但文件描述符有效，只有`watchdog`可以使用它。在这种情况下，我们可以允许 watchdog 这个规则。在`watchdogd.te`中，添加`allow watchdogd device:chr_file create_file_perms;`。然而，这个规则在基本策略中引起了`neverallow`的违反：

```kt
out/host/linux-x86/bin/checkpolicy: loading policy configuration from out/target/product/udoo/obj/ETC/sepolicy_intermediates/policy.conf
libsepol.check_assertion_helper: neverallow on line 5375 violated by allow watchdogd device:chr_file { read write open };
Error while expanding policy

```

`neverallow` 规则位于 `domain.te` 基本策略中，为 `neverallow { domain -init -ueventd -recovery } device:chr_file { open read write };`。对于这种简单的更改，我们只需将基本 sepolicy 更改为 `neverallow { domain -init -ueventd -recovery -watchdogd } device:chr_file { open read write };`。

## wpa

```kt
#============= wpa ==============
allow wpa device:chr_file { read open };
allow wpa log_device:chr_file { write open };
allow wpa system_data_file:dir { write remove_name add_name setattr };
allow wpa system_data_file:sock_file { write create unlink setattr };
```

同样，日志设备在 `domain.te` 中处理。系统数据访问需要进一步调查，从原始拒绝开始：

```kt
type=1400 msg=audit(1417405614.060:193): avc: denied { setattr } for pid=2639 comm="wpa_supplicant" name="wpa_supplicant" dev=mmcblk0p4 ino=129295 scontext=u:r:wpa:s0 tcontext=u:object_r:system_data_file:s0 tclass=dir
type=1400 msg=audit(1417405614.060:194): avc: denied { write } for pid=2639 comm="wpa_supplicant" name="wlan0" dev=mmcblk0p4 ino=129318 scontext=u:r:wpa:s0 tcontext=u:object_r:system_data_file:s0 tclass=sock_file
type=1400 msg=audit(1417405614.060:195): avc: denied { write } for pid=2639 comm="wpa_supplicant" name="wpa_supplicant" dev=mmcblk0p4 ino=129295 scontext=u:r:wpa:s0 tcontext=u:object_r:system_data_file:s0 tclass=dir
type=1400 msg=audit(1417405614.060:196): avc: denied { remove_name } for pid=2639 co

```

使用 `ls -laR` 定位到问题文件：

```kt
/data/system/wpa_supplicant:
srwxrwx--- wifi wifi 2014-12-01 06:43 wlan0

```

这个套接字是由 `wpa_supplicant` 自身创建的。在没有类型转换的情况下重新标记它是不可能的，因此我们必须允许它。在 `wpa.te` 中添加 `allow wpa system_data_file:dir rw_dir_perms;` 和 `allow wpa system_data_file:sock_file create_file_perms;`。未标记的设备已经处理过了；它在 `rfkill` 上：

```kt
type=1400 msg=audit(1417405613.640:175): avc: denied { read } for pid=2639 comm="wpa_supplicant" name="rfkill" dev=tmpfs ino=1126 scontext=u:r:wpa:s0 tcontext=u:object_r:device:s0 tclass=chr_file

```

# 第二次策略传递

在加载草拟的策略后，设备在启动时仍然有拒绝：

```kt
#============= init ==============
allow init rootfs:file { write create };
allow init system_file:file execute_no_trans;
#============= shell ==============
allow shell device:chr_file { read write getattr };
allow shell system_file:file entrypoint;
```

所有这些拒绝都应该被调查，因为它们针对的是敏感类型，特别是 `tcontext`。

## init

`init` 的原始拒绝如下：

```kt
<5>type=1400 audit(4.380:3): avc: denied { create } for pid=2268 comm="init" name="tasks" scontext=u:r:init:s0 tcontext=u:object_r:rootfs:s0 tclass=file
<5>type=1400 audit(4.380:4): avc: denied { write } for pid=2268 comm="init" name="tasks" dev=rootfs ino=3080 scontext=u:r:init:s0 tcontext=u:object_r:rootfs:s0 tclass=file

```

这些情况发生在 `init` 将 `/` 重新挂载为只读之前。我们可以安全地允许这些，由于 `init` 正在非限制状态下运行，我们只需将其添加到 `init.te` 中。我们可以将 `allow` 规则添加到非限制集合中，但由于它即将消失，让我们将权限最小化到仅 `init`：

```kt
allow int rootfs:file create_file_perms;
```

### 注意

未限制的域并不是完全不受限制的。随着 AOSP 趋向于零未限制域，这个域的规则会被剥离。

然而，这样做会导致另一个 `neverallow` 失败。我们可以修改 `external/sepolicy domain.te` 来绕过这个问题。将 `neverallow` 从这个更改为：

```kt
# Nothing should be writing to files in the rootfs.
neverallow { domain -recovery} rootfs:file { create write setattr relabelto append unlink link rename };
```

更改为以下内容：

```kt
# Nothing should be writing to files in the rootfs.
neverallow { domain -recovery -init } rootfs:file { create write setattr relabelto append unlink link rename };
```

### 注意

如果你需要修改 `neverallow` 条目以构建，你将无法通过 CTS。正确的方法是从 `init` 中移除这种行为。

此外，我们需要查看哪些内容在没有域转换的情况下通过 `exec` 加载，导致 `execute_no_trans` 拒绝：

```kt
<5>type=1400 audit(4.460:6): avc: denied { execute_no_trans } for pid=2292 comm="init" path="/system/bin/magd" dev=mmcblk0p5 ino=146 scontext=u:r:init:s0 tcontext=u:object_r:system_file:s0 tclass=file
<5>type=1400 audit(4.460:6): avc: denied { execute_no_trans } for pid=2292 comm="init" path="/system/bin/rfkill" dev=mmcblk0p5 ino=148 scontext=u:r:init:s0 tcontext=u:object_r:system_file:s0 tclass=file

```

为了解决这个问题，我们可以用其自己的类型重新标记 `magd`，并将其放在自己的非限制域中。基本策略中的 `neverallow` 强迫我们将每个可执行文件移动到其自己的域中。

创建一个名为 `magd.te` 的文件，将其添加到 `BOARD_SEPOLICY_UNION` 中，并向其中添加以下内容：

```kt
type magd, domain;
type magd_exec, exec_type, file_type;
permissive_or_unconfined(magd);
```

同时更新 `file_contexts` 以包含以下内容：

```kt
/system/bin/magd  u:object_r:magd_exec:s0
```

对 `rfkill` 重复为 `magd` 所做的步骤。只需在前面示例中将 `magd` 替换为 `rfkill`。后来的测试发现了一个入口点拒绝，其中源上下文是 `init_shell`，目标是 `rfkill_exec`。在添加 shell 规则后，发现 `rfkill` 是从 `init_shell` 域使用 `exec` 加载的，因此我们也应该在 `rfkill.te` 文件中添加 `domain_auto_trans(init_shell, rfkill_exec, rfkill)`。此外，与这一发现相关的是 `rfkill` 尝试打开、读取和写入 `/dev/rfkill`。因此，我们必须用 `rfkill_device` 标记 `/dev/rfkill`，允许 `rfkill` 访问它，并在 `rfkill.te` 文件中追加 `allow rfkill rfkill_device:chr_file rw_file_perms;`。创建一个名为 `device.te` 的新文件来声明此设备类型，并添加 `type rfkill_device, dev_type;`。之后，通过在 `file_contexts` 中添加 `/dev/rfkill u:object_r:rfkill_device:s0` 来标记它。

## shell

我们将评估的第一个 shell 拒绝是在 `entrypoint` 上：

```kt
<5>type=1400 audit(4.460:5): avc: denied { entrypoint } for pid=2279 comm="init" path="/system/bin/mksh" dev=mmcblk0p5 ino=154 scontext=u:r:shell:s0 tcontext=u:object_r:system_file:s0 tclass=file
```

由于我们没有给 `mksh` 打标签，现在我们需要给它打标签。我们可以创建一个不受限制的域，让 `init` 启动的 shell 最终进入 `init_shell` 域。控制台仍然通过显式 `seclabel` 进入 `shell` 域，而其他调用则作为 `init_shell`。创建一个新文件 `init_shell.te`，并将其添加到 `BOARD_SEPOLICY_UNION` 中。

## init_shell.te

```kt
type init_shell, domain;
domain_auto_trans(init, shell_exec, init_shell);
permissive_or_unconfined(init_shell);
```

更新 `file_contexts` 以包含以下内容：

```kt
/system/bin/mksh  u:object_r:shell_exec:s0;
```

现在，我们将处理对原始设备的 shell 访问：

```kt
<5>type=1400 audit(6.510:7): avc: denied { read write } for pid=2279 comm="sh" name="ttymxc1" dev=tmpfs ino=122 scontext=u:r:shell:s0 tcontext=u:object_r:device:s0 tclass=chr_file
<5>type=1400 audit(7.339:8): avc: denied { getattr } for pid=2279 comm="sh" path="/dev/ttymxc1" dev=tmpfs ino=122 scontext=u:r:shell:s0 tcontext=u:object_r:device:s0 tclass=chr_file

```

这只是一个标签错误的 `tty`，我们可以将其标记为 `tty_device`。在文件上下文中添加以下条目：

```kt
/dev/ttymxc[0-9]*  u:object_r:tty_device:s0
```

# 现场试验

在此阶段，重新构建源代码树，清除数据文件系统，刷新，并重新运行 CTS。重复此操作直到所有拒绝都被处理。

当你完成 CTS 和内部 QA 测试后，我们建议在宽容模式下进行现场试验。在此期间，你应该收集日志并完善策略。如果域不稳定，你可以在策略文件中将它们声明为宽容模式，但仍然将设备设置为执行模式；执行部分域总比一个都不执行要好。

# 转为执行模式

你可以使用 `bootloader` 传递执行模式（这部分内容这里不涉及），或者在启动早期通过 `init.rc` 脚本进行设置。你可以在 `setcon` 之后立即进行如下操作：

```kt
setcon u:r:init:s0
setenforce 1
```

一旦这条语句被编译到 `init.rc` 脚本中，就只能通过后续构建和重新刷新 `boot.img` 来撤销。你可以通过运行 `getenforce` 命令来检查这一点。另外，作为一个有趣的测试，你可以尝试从根串行控制台运行 `reboot` 命令，并观察它失败：

```kt
root@udoo:/ # getenforce
Enforcing
root@udoo:/ # reboot
reboot: Operation not permitted

```

# 摘要

在本章中，您之前对系统的所有理解都被用来为全新设备开发实际的 Android 安全增强（SE）策略。现在，您已经掌握了如何编写 Android 的 SELinux 策略、系统的各个组件在哪里以及如何工作，以及如何将这些特性移植并启用在各种 Android 平台上。由于这是一个相当新的特性，它影响许多系统交互，因此将出现需要代码变更以及策略变更的问题。理解这两者是至关重要的。

作为策略作者和一般的安全人员，确保系统的安全责任落在了我们的肩上。在大多数组织中，您可能需要在缺乏信息的情况下工作。然而，如果您可以，尽可能多地工作并在邮件列表中提出问题，绝不要接受现状。Android 安全增强（SE）和 AOSP 项目欢迎所有人贡献，通过贡献，您将帮助改进项目并增强所有功能的特性集。
