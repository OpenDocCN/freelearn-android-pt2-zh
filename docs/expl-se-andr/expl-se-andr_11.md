# 第十一章：标签属性

在本章中，我们将介绍如何通过`property_contexts`文件标记属性。

属性是我们在第三章，*Android Is Weird*中学到的 Android 的独特特性。我们希望对这些属性进行标签化，以限制设置属性仅限于应设置它们的域，防止经典的 DAC 根攻击无意中更改其值。在本章中，我们将学习：

+   创建新属性

+   标签新属性和现有属性

+   解释和处理属性拒绝

+   列举特殊的 Android 属性及其行为

# 通过 property_contexts 标签化

所有属性都使用`property_contexts`文件进行标记，其语法类似于`file_contexts`。但是，它不是在文件路径上工作，而是在属性名称或属性键上工作（Android 中的属性是键值存储）。属性键本身通常用句点（`.`）分隔。这类似于`file_contexts`，只不过斜杠（`/`）变成了句点。一些示例属性及其在`property_contexts`中的条目可能如下所示：

```kt
ctl.ril-daemon  u:object_r:ctl_rildaemon_prop:s0
ctl.   u:object_r:ctl_default_prop:s0
```

注意到所有`ctl.`属性都被标记为`ctl_default_prop`类型，但`ctl.ril-daemon`具有不同的类型标签`ctl_rildaemon_prop`。这代表了你如何可以从通用开始，并根据需要移动到更具体的值/类型。

此外，任何未明确标记的属性默认通过`property_contexts`中的“匹配所有”表达式设置为`default_prop`：

```kt
# default property context
* u:object_r:default_prop:s0
```

# 属性上的权限

可以查看系统上的当前属性，并使用命令行工具`getprop`和`setprop`创建新属性，如下代码片段所示：

```kt
root@udoo:/ # getprop
...
[sys.usb.state]: [mtp,adb]
[wifi.interface]: [wlan0]
[wlan.driver.status]: [unloaded]

```

回顾第三章，*Android Is Weird*，我们知道属性被映射到每个人的地址空间，因此任何人都可以读取它们。然而，并不是每个人都可以设置（写入）它们。属性的 DAC 权限模型硬编码在`system/core/init/property_service.c`中：

```kt
/* White list of permissions for setting property services. */
struct {
  const char *prefix;
  unsigned int uid;
  unsigned int gid;
} property_perms[] = {
  { "net.rmnet0.", AID_RADIO, 0 },
  { "net.gprs.", AID_RADIO, 0 },
  { "net.ppp", AID_RADIO, 0 },
...
  { "persist.service.bdroid.", AID_BLUETOOTH, 0 },
  { "selinux." , AID_SYSTEM, 0 },
  { "persist.audio.device", AID_SYSTEM, 0 },
  { NULL, 0, 0 }
```

如果要在`property_perms`数组中设置与任何属性前缀匹配的属性，你必须具有 UID 或 GID。例如，为了设置`selinux.`属性，你必须具有 UID `AID_SYSTEM`（uid 1000）或 root 权限。是的，root 总是可以设置属性，这是将 SELinux 应用于 Android 属性的关键优势。不幸的是，目前没有方法使用`getprop -Z`列出属性及其标签，就像使用`ls -Z`和文件一样。

# 重新标记现有属性

为了更熟悉标签属性，让我们重新标记`wifi.interface`属性。首先，让我们通过引发拒绝并查看拒绝日志来验证其上下文，如下代码所示：

```kt
root@udoo:/ # setprop wifi.interface wlan0
avc: denied { set } for property=wifi.interface scontext=u:r:shell:s0 tcontext=u:object_r:default_prop:s0 tclass=property_service

```

当我们通过 UDOOUART 控制台执行`setprop`命令时，发生了一件有趣的事情。打印出了 AVC 拒绝记录。这是因为串行控制台包括了使用`printk()`从内核打印的任何内容。这里发生的情况是，如第三章 *安卓古怪* 中详细控制的`init`进程，向内核日志写入一条消息。当我们执行`setprop`命令时，这条日志消息会显示出来。如果你通过`adb shell`运行，你会在串行控制台上看到这个消息，但在`adb`控制台上看不到。然而，要做到这一点，你必须重新启动你的系统，因为 SELinux 在宽容模式下只打印一次拒绝记录。

使用`adb shell`的命令如下：

```kt
$ adb shell setprop wifi.interface wlan0

```

使用串行控制台的命令如下：

```kt
root@udoo:/ # avc: denied {set} for property=wifi.interface scontext=u:r:shell:s0 tcontext=u:object_r:default_prop
usb 2-1.3: device descriptor read/64, error -110

```

从拒绝输出中，我们可以看到属性类型标签是`default_prop`。让我们将其更改为`wifi_prop`。

我们首先在`sepolicy`目录中编辑`property.te`文件，通过添加以下行来声明新类型，以便对这些属性进行标签化：

```kt
type wifi_prop, property_type;
```

类型声明后，下一步是通过对`property_contexts`进行修改来应用标签，添加以下内容：

```kt
# wifi properties
wifi. u:object_r:wifi_prop:s0
```

按如下方式构建策略：

```kt
$ mmm external/sepolicy

```

推送新的`property_contexts`文件：

```kt
$ adb push out/target/product/udoo/root/property_contexts /data/security/current
51 KB/s (2261 bytes in 0.042s)

```

触发动态重载：

```kt
$ adb shell setprop selinux.reload_policy 1
# setprop wifi.interface wlan0
avc: denied { set } for property=wifi.interface scontext=u:r:shell:s0 tcontext=u:object_r:default_prop:s0 tclass=property_service

```

好吧，那不起作用！`property_contexts`文件必须在`/data/security`中，而不是`/data/security/current`。

要发现这一点，请搜索`libselinux/src/android.c`文件。这个文件中没有提到`property_contexts`；因此，它必须在其他地方提到。这引导我们搜索`system/core`，其中包含属性服务对该文件的引用。在`init.c`中的匹配代码从优先位置加载文件。

```kt
$ grep -rn property_contexts *
init/init.c:745: { SELABEL_OPT_PATH, "/data/security/property_contexts" },
init/init.c:746: { SELABEL_OPT_PATH, "/property_contexts" },
init/init.c:760: ERROR("SELinux: Could not load property_contexts: %s\n",

```

让我们将`property_contexts`文件推送到正确的位置，并再次尝试：

```kt
$ adb push out/target/product/udoo/root/property_contexts /data/security
51 KB/s (2261 bytes in 0.042s)
$ adb shell setprop selinux.reload_policy 1
root@udoo:/ # setprop wifi.interface wlan0
avc: received policyload notice (seqno=3)
init: sys_prop: permission denied uid:0 name:wifi.interface

```

哇！又失败了。这个练习是为了指出如果你忘记做一些事情，这会有多么棘手。没有显示任何有用的拒绝信息，只有一个被拒绝的指示。这是因为包含`wifi_prop`类型声明的`sepolicy`文件从未被推送。这导致`system/core/init/property_service.c`中的`check_mac_perms()`在`selinux_check_access()`函数中失败，因为它找不到要计算访问检查的类型，尽管在`property_contexts`中的查找成功了。没有来自此的详细错误日志。

我们可以通过确保也推送`sepolicy`来更正这个问题：

```kt
$ adb push out/target/product/udoo/root/sepolicy /data/security/current/
550 KB/s (87385 bytes in 0.154s)
$ adb shell setprop selinux.reload_policy 1
root@udoo:/ # setprop wifi.interface wlan0
avc: received policyload notice (seqno=4)
avc: denied { set } for property=wifi.interface scontext=u:r:shell:s0 tcontext=u:object_r:wifi_prop:s0 tclass=property_service

```

现在我们看到了预期的拒绝消息，但目标的标签（或属性）是`u:object_r:wifi_prop:s0`。

现在目标属性已标记，你可以允许访问它。请注意，这是一个虚构的例子，在现实世界中，你可能*不*希望允许从 shell 访问大多数属性。策略应与你的安全目标和最小权限属性保持一致。

我们可以在`shell.te`中以下面的方式添加一个`allow`规则：

```kt
# wifi prop
allow shelldomain wifi_prop:property_service set;
```

编译策略，将其推送到手机上，并触发动态重新加载：

```kt
$ mmm external/sepolicy/
$ adb push out/target/product/udoo/root/sepolicy /data/security/current/
547 KB/s (87397 bytes in 0.155s)
$ adb shell setprop selinux.reload_policy 1

```

现在尝试设置`wifi.interface`属性，并注意没有拒绝。

```kt
root@udoo:/ # setprop wifi.interface wlan0
avc: received policyload notice (seqno=5)

```

# 创建和标记新属性

所有属性都是通过使用`setprop`调用或在 C (`bionic/libc/include/sys/system_properties.h`) 和 Java (`android.os.SystemProperties`)中执行等效功能的函数调用在系统中动态创建的。请注意，`System.getProperty()`和`System.setProperty()` Java 调用是针对应用程序私有属性存储的，并且没有与全局属性存储绑定。

对于 DAC 控制，您需要按照之前提到的修改`property_perms[]`，以便非 root 用户可以创建或设置属性。请注意，除非受到 SELinux 策略的限制，否则 root 用户始终可以`set`和`create`。

假设我们想要创建`udoo.name`和`udoo.owner`属性；我们只希望 root 用户和 shell 域访问它们。我们可以这样创建它们：

```kt
root@udoo:/ # setprop udoo.name udoo
avc: denied { set } for property=udoo.name scontext=u:r:shell:s0 tcontext=u:object_r:default_prop:s0 tclass=property_service
root@udoo:/ # setprop udoo.owner William

```

注意否认显示这些为`default_prop`类型。要纠正这一点，我们会像前一部分*重新标记现有属性*中所做的那样重新标记这些属性。

# 特殊属性

在 Android 中，有些特殊属性具有不同的行为。我们在接下来的部分列举了属性名称及其含义。

## 控制属性

以`ctl`开头的属性被保留为控制属性，用于通过`init`控制服务：

+   `start`：启动服务（`setprop ctl.start <服务名>`）

+   `stop`：停止服务（`setprop ctl.stop <服务名>`）

+   `restart`：重启服务（`setprop ctl.restart <服务名>`）

## 持久属性

任何以`persist`为前缀的属性在重启后会保留并恢复。数据被保存到`/data/property`目录下，文件名与属性相同。

```kt
root@udoo:/ # ls /data/property/
persist.gps.oacmode
persist.service.bdroid.bdaddr
persist.sys.profiler_ms
persist.sys.usb.config

```

## SELinux 属性

`selinux.reload_policy`属性是特殊的。正如我们所见，它的用途是触发动态重新加载事件。

# 总结

在本章中，我们探讨了如何创建和标记新属性和现有属性，以及在这样做时出现的一些异常情况。我们还检查了`property_service.c`中属性的硬编码 DAC 权限表，以及像`ctl.`系列这样的硬编码特殊属性。在下一章中，我们将了解工具链如何构建和创建我们一直在使用的所有策略文件。
