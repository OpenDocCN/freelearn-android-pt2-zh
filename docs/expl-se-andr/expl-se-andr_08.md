# 第八章：应用上下文到文件

在上一章中，我们升级了系统，收集了审计日志，并开始分析审计记录。我们发现文件系统上的一些对象未标记。在本章中，我们将：

+   了解文件系统和文件系统对象如何获取它们的标签

+   展示更改标签的技术

+   引入扩展属性进行标记

+   调查文件上下文和动态类型转换

# 标记文件系统

Linux 上的文件系统源自 mount，Android 上的`ramdisk rootfs`除外。Linux 上的文件系统差异极大。通常，为了支持 SELinux 的所有功能，你需要一个支持`xattr`和`security`命名空间的文件系统。我们在设置内核配置时遇到了这个要求。

文件系统对象在创建时，都带有初始上下文，就像所有其他内核对象一样。文件上的上下文简单地继承自它们的父级，因此如果父级未标记，则子级未标记，除非有类型转换规则。通常，如果上下文未标记，它推断数据是在启用 SELinux 支持之前的文件系统上创建的，或者当前加载的策略中不存在`xattr`中的类型标签。

初始标签或初始**安全 ID**（**sid**）在`sepolicy`文件`initial_sid_contexts`中。每个对象类都有其相关的初始`sid`。例如，让我们看一下以下代码片段：

```kt
...
sid fs u:object_r:labeledfs:s0
sid file u:object_r:unlabeled:s0
...
```

## fs_use

文件系统可以通过多种方式进行标记。最佳的情况是文件系统支持`xattrs`。在这种情况下，策略中应该会出现`fs_use_xattr`声明。这些声明位于`sepolicy`目录中的`fs_use`文件中。`fs_use_xattr`的语法如下：

```kt
fs_use_xattr <fstype> <context>

```

要查看`sepolicy`中的`fs_use`，我们可以参考`ext4`文件系统的示例：

```kt
...
fs_use_xattr ext3 u:object_r:labeledfs:s0;
fs_use_xattr ext4 u:object_r:labeledfs:s0;
fs_use_xattr xfs u:object_r:labeledfs:s0;
...
```

这告诉 SELinux，当它遇到`ext4` `fs`对象时；在扩展属性中查找标签或文件上下文。

## fs_task_use

文件系统可以通过在创建对象时使用进程上下文来进行标记。这对于伪文件系统来说是有意义的，因为这些对象实际上是进程上下文，如`pipefs`和`sockfs`。这些伪文件系统管理管道和套接字系统调用，并不真正挂载到用户空间。它们存在于内核内部，供内核使用。然而，它们确实有对象，并且像任何其他对象一样，它们需要被标记。在这种情况下，`fs_task_use`策略声明是有意义的。这些内部文件系统只能被进程直接访问，并为这些进程提供服务。因此，用创建者进行标记是合理的。语法如下：

```kt
fs_task_use <fstype> <context>

```

`sepolicy`文件`fs_use`中的示例如下：

```kt
...
# Label inodes from task label.
fs_use_task pipefs u:object_r:pipefs:s0;
fs_use_task sockfs u:object_r:sockfs:s0;
...
```

## fs_use_trans

你可能希望设置的下一个在实际上挂载的伪文件系统上设置标签的方法是使用`fs_use_trans`。这为伪文件系统设置一个文件系统范围的标签。这个的语法如下：

```kt
fs_use_trans <fstype> <context>

```

`sepolicy`文件中`fs_use`的示例如下：

```kt
...
fs_use_trans devpts u:object_r:devpts:s0;
fs_use_trans tmpfs u:object_r:tmpfs:s0;
...
```

## genfscon

如果没有任何`fs_use_*`语句符合你的使用场景，比如`vfat`文件系统和`procfs`，那么你会使用`genfscon`语句。为`genfscon`指定的标签适用于*所有*该文件系统挂载的实例。例如，你可能希望对`vfat`文件系统使用`genfscon`。如果你有两个`vfat`挂载点，它们将针对每个挂载点使用相同的`genfscon`语句。然而，`genfscon`在处理`procfs`时行为不同，允许你为文件系统内的每个文件或目录设置标签。

`genfscon`的语法如下：

```kt
genfscon <fstype> <path> <context>

```

`sepolicy genfs_contexts`的示例如下：

```kt
...
# Label inodes with the fs label.
genfscon rootfs / u:object_r:rootfs:s0
# proc labeling can be further refined (longest matching prefix).
genfscon proc / u:object_r:proc:s0
genfscon proc /net/xt_qtaguid/ctrl u:object_r:qtaguid_proc:s0
...
```

请注意，`rootfs`的部分路径是`/`。它不是`procfs`，所以不支持对其标记的任何细粒度控制；因此，`/`是你唯一可以使用的。然而，你可以对`procfs`进行任意粒度的设置。

## 挂载选项

如果这些选项都不符合你的需求，另一个选项是可以通过`mount`命令行传递`context`选项。这设置一个文件系统范围的挂载上下文，如`genfscon`，但在需要分别设置标签的多个文件系统中很有用。例如，如果你挂载了两个`vfat`文件系统，你可能希望分开访问它们。使用`genfscon`语句，两个文件系统将使用由`genfscon`提供的相同标签。通过在挂载时指定标签，你可以让两个`vfat`文件系统使用不同的标签挂载。

以以下命令为例：

```kt
mount -ocontext=u:object_r:vfat1:s0 /dev/block1 /mnt/vfat1
mount -ocontext=u:object_r:vfat2:s0 /dev/block1 /mnt/vfat2

```

除了作为挂载选项的上下文之外，还有：`fscontext`和`defcontext`。这些选项与上下文是互斥的。`fscontext`选项设置用于某些操作（如挂载）的元文件系统类型，但不会改变每个文件的标签。`defcontext`设置未标记文件的默认上下文，覆盖`initial_sid`语句。最后，另一个选项`rootcontext`允许你设置文件系统中的根 inode 上下文，但仅适用于该对象。根据 mount 的手册页（`man 8 mount`），在无状态 Linux 中它被证明是有用的。

## 使用扩展属性进行标记

最后，最常用于标记的方法之一是使用扩展属性支持，也称为`xattr`或 EA 支持。即使有`xattr`支持，新对象也会继承其父目录的上下文；然而，这些标签具有基于每个文件系统对象或 inode 的细粒度。如果你记得，我们需要为 Android 上的文件系统启用或验证`XATTR(CONFIG_EXT4_FS_XATTR)`支持，并通过配置选项`CONFIG_EXT4_FS_SECURITY`配置 SELinux 使用它。

扩展属性是文件的键值元数据存储。SELinux 安全上下文使用`security.selinux`键，值是一个字符串，即安全上下文或标签。

## `file_contexts`文件

在`sepolicy`目录中，你会找到`file_contexts`文件。这个文件用于设置支持每个文件安全标签的文件系统的属性。请注意，一些伪文件系统也支持这一点，如`tmpfs`、`sysfs`以及最近的`rootfs`。`file_context`文件具有基于正则表达式的语法，如下所示，其中`regexp`是路径的正则表达式：

```kt
regexp <type> ( <file label> | <<none>> )

```

如果为文件定义了多个正则表达式，将使用最后一个匹配项，因此顺序很重要。

下面的列表显示了每种文件系统对象的类型字段值，它们的含义以及系统调用接口：

+   `--`：这表示一个常规文件。

+   `-d`：这表示一个目录。

+   `-b`：这表示一个块文件。

+   `-s`：这表示一个套接字文件。

+   `-c`：这表示一个字符文件。

+   `-l`：这表示一个链接文件。

+   `-p`：这表示一个命名管道文件。

如你所见，类型本质上是`ls -la`命令输出的模式。如果没有指定，它将匹配所有内容。

下一个字段是文件标签或特殊标识符`<<none>>`。两者都可以提供上下文或标识符`<<none>>`。如果你指定了上下文，那么咨询`file_contexts`的 SELinux 工具将使用与指定上下文最后的匹配项。如果指定的上下文是`<<none>>`，这意味着没有分配上下文。所以，保留我们找到的那个。关键字`<<none>>`没有在 AOSP 参考的`sepolicy`中使用。

需要注意的是，前一段明确指出 SELinux 工具使用了`file_contexts`策略。内核并不知道这个文件的存在。SELinux 通过明确地从用户空间设置工具来给所有对象贴上标签，这些工具会在`file_context`中查找上下文，或者通过`fs_use_*`和`genfs`策略声明。换句话说，`file_contexts`没有内置于核心策略文件中，也没有被内核直接加载或使用。在构建时，`file_contexts`文件被构建在 ramdisk 的 rootfs 中，可以在`/file_contexts`找到。此外，在构建时，系统镜像被贴上标签，从而减轻设备本身的负担。

在 Android 中，`init`、`ueventd`和`installd`都已经修改为在创建对象时查找它们的上下文；这样它们可以正确地给它们贴上标签。因此，所有创建文件系统对象的 init 内置命令，如`mkdir`，都已被修改以使用存在的`file_contexts`文件，`installd`和`ueventd`也是如此。

让我们来看一些来自`sepolicy`目录中的`file_context`文件的部分内容：

```kt
...
/dev(/.*)? u:object_r:device:s0
/dev/accelerometer u:object_r:sensors_device:s0
/dev/alarm u:object_r:alarm_device:s0
...
```

在这里，我们为`/dev`中的文件设置上下文。请注意，条目是从最通用到更具体的`dev`文件的顺序。因此，未被更具体条目覆盖的任何文件最终将具有上下文`u:object_r:device:s0`，而匹配到更下面文件的将具有更具体的标签。例如，在`/dev/accelerometer`的加速度计将获得上下文`u:object_r:sensors_device:s0`。注意类型字段被省略了，这意味着它匹配*所有*文件系统对象，如目录（`type -d`）。

你可能想知道`/dev`目录本身是如何获得文件上下文的。查看一些代码片段，我们看到根目录`/`通过`genfs_context`文件中的声明`genfscon rootfs / u:object_r:rootfs:s0`被标记。本章前面提到，“新对象继承其父目录的上下文。”因此，我们可以推断出`/dev`的上下文是`u:object_r:rootfs:s0`，因为这是`/`的标签。我们可以通过向`ls`传递`-Z`标志来显示`/dev`的标签来测试这一点。在 UDOО串行连接上，执行以下命令：

```kt
130|root@udoo:/ # ls -laZ / 
...
drwxr-xr-x root root u:object_r:device:s0 dev
...

```

看起来这个假设是错误的，但请注意，确实一切都有标签，如果没有明确指定，那么它将从父级继承。回顾`sepolicy`，我们可以看到`dev`文件系统最初设置了`fs_use_trans devtmpfs u:object_r:device:s0;`这样的策略声明。因此，当文件系统被挂载时，它是全局设置的。后来，当`init`或`ueventd`添加条目时，它们使用`file_contexts`条目将新创建的文件系统对象的上下文设置为`file_contexts`文件中指定的内容。在`/dev`的文件系统，它是一个`devtmps`伪文件系统，就是一个同时具有通过`fs_use_trans`声明设置的全局标签，同时也能通过`file_contexts;`支持细粒度标签的文件系统示例。在 Linux 上，文件系统的能力并不一致。

## 动态类型转换

由 SELinux 策略语句`type_transition`指示的动态类型转换是一种允许文件动态确定其类型的方法。因为这些是编译到策略中的，所以它们与`file_contexts`文件无关。这些策略语句允许策略作者基于文件创建的上下文动态地指示文件上下文。在你不控制源代码，或者不想以任何方式将 SELinux 耦合在一起的情况下，这些是非常有用的。例如，`wpa`请求者，这是一个为 Wi-Fi 支持运行的服务，在其数据目录中创建一个套接字文件。其数据目录被标记为类型`wifi_data_file`，如预期的那样，套接字最终也具有该标签。然而，此套接字由系统服务器共享。现在，我们可以允许系统服务器访问类型和对象类，但是`hostapd`和其他东西正在该目录中创建套接字和其他对象，因此这些对象也具有此类型。我们确实希望确保两个有问题的套接字，一个由`hostapd`使用，另一个由系统服务器使用，彼此保持独立。为此，我们需要能够以更细的粒度标记其中一个套接字，为此，我们可以修改代码或使用动态类型转换。与其修改代码，不如使用以下类型的转换：

```kt
type_transition wpa wifi_data_file:sock_file wpa_socket;

```

这是来自`sepolicy`文件`wpa_supplicant.te`中的实际语句。它表示，当类型为`wpa`的进程创建类型为`wifi_data_file`的文件且对象类为`sock_file`时，在创建时将其标记为`wpa_socket`。语句语法如下：

```kt
type_transition <creating type> <created type>:<class> <new type>;
```

从 SELinux 策略版本 25 开始，`type_transition`语句可以支持带名称的类型转换，其中第四个参数存在，且是文件的名称：

```kt
type_transition <creating type> <created type>:<class> <new type> <file name>;
```

我们将在`sepolicy`文件`system_server.te`中看到一个关于此文件名的示例使用：

```kt
type_transition system_server system_data_file:sock_file system_ndebug_socket "ndebugsocket";
```

请注意文件名或基名称，而不是路径，并且必须完全匹配。不支持正则表达式。有趣的是，动态转换不仅限于文件对象，还包括任何对象类事件进程。我们将在第九章《将服务添加到域》中看到如何使用动态进程转换。

# 示例和工具

理论知识我们已经有了，现在让我们看看系统中标记文件的工具和技术。首先，我们从挂载一个`ramfs`文件系统开始。由于`/`是只读的，我们将重新挂载它并为文件系统创建一个挂载点。通过 UDOOUDOO 串行控制台，执行以下命令：

```kt
root@udoo:/ # mount -oremount,rw /
root@udoo:/ # mkdir /ramdisk
root@udoo:/ # mount -t ramfs -o size=20m ramfs /ramdisk

```

现在，我们想要查看文件系统具有哪个标签：

```kt
# ls -laZ / | grep ramdisk 
drwxr-xr-x root root u:object_r:unlabeled:s0 ramdisk

```

如你所记得，`initial_sid_context`文件为此文件系统设置了此初始`sid`：

```kt
sid file u:object_r:unlabeled:s0
```

如果我们想要在新标签中获取这个 ramdisk，我们需要在策略中创建类型，并设置一个新的`genfscon`语句来使用它。我们将在 sepolicy 文件`file.te`中声明新类型：

```kt
type ramdisk, file_type, fs_type;
```

类型策略语句的语法如下：

```kt
type <new type>, <attribute0,attribute1…attributeN>;
```

SELinux 中的属性是允许你定义常见组的语句。它们是通过`attribute`语句定义的。在 Android SELinux 策略中，我们已经定义了`file_type`和`fs_type`。我们将在这里使用它们，因为我们要创建的这种新类型具有`file_type`和`fs_type`属性。`file_type`属性与文件的类型相关联，而`fs_type`属性意味着此类型也与文件系统相关联。目前，属性并不是非常重要；所以不要在细节上纠结。

下一个要修改的是`sepolicy`文件，`genfs_context`，通过添加以下内容：

```kt
genfscon ramfs / u:object_r:ramdisk:s0
```

现在，我们将编译引导映像并将其闪存到设备上，或者更好的是，让我们使用如下所示的动态策略重新加载支持。

从 UDOOb 项目的根目录仅构建`sepolicy`项目：

```kt
$ mmm external/sepolicy/

```

通过`adb`推送新策略，如下所示：

```kt
$ adb push $OUT/root/sepolicy /data/security/current/sepolicy
544 KB/s (86409 bytes in 0.154s)

```

使用`setprop`命令触发重新加载：

```kt
$ adb shell setprop selinux.reload_policy 1

```

如果你连接了串行控制台，你应该会看到：

```kt
SELinux: Loaded policy from /data/security/current/sepolicy

```

如果你没有，只有`adb`，检查`dmesg`：

```kt
$ adb shell dmesg | grep "SELinux: Loaded"
<4>SELinux: Loaded policy from /sepolicy
<6>init: SELinux: Loaded property contexts from /property_contexts
<4>SELinux: Loaded policy from /data/security/current/sepolicy

```

成功加载应该使用我们在路径`/data/security/current/sepolicy`上的策略。让我们卸载 ramdisk 并重新挂载它，以查看其类型：

```kt
root@udoo:/ # umount /ramdisk 
root@udoo:/ # mount -t ramfs -o size=20m ramfs /ramdisk
root@udoo:/ # ls -laZ / | grep ramdisk
drwxr-xr-x root root u:object_r:ramdisk:s0 ramdisk

```

我们能够修改策略并使用`genfscon`更改文件系统类型，现在为了显示继承，让我们继续在文件系统上使用`touch`创建一个文件：

```kt
root@udoo:/ # cd /ramdisk
root@udoo:/ramdisk # touch hello
root@udoo:/ramdisk # ls -Z
-rw------- root root u:object_r:ramdisk:s0 hello

```

正如我们所预期的，新文件被标记为 ramdisk 类型。现在，假设我们从 shell 执行 touch 操作，希望文件是另一种类型，比如`ramdisk_newfile`，我们该如何操作？我们可以通过修改 touch 本身来咨询`file_contexts`，或者我们可以定义一个动态类型转换；让我们尝试动态类型转换的方法。`type_transition`语句的第一个参数是创建类型；那么我们的 shell 是什么类型呢？你可以通过执行以下操作来获取：

```kt
root@udoo:/ramdisk # echo `cat /proc/self/attr/current`
u:r:init_shell:s0

```

更简单的方法是运行`id -Z`命令，该命令使用前述的`proc`文件。对于串行控制台，执行：

```kt
root@udoo:/ramdisk # id -Z
uid=0(root) gid=0(root) context=u:r:init_shell:s0

```

并为`adb` shell 运行相同的命令：

```kt
$ adb shell id -Z
uid=0(root) gid=0(root) context=u:r:shell:s0

```

注意我们在串行控制台 shell 和`adb` shell 之间的差异，在第九章*将服务添加到域*中，我们将修复这个问题。因此，我们现在编写的策略将解决这两种情况。

首先，打开`sepolicy`文件，`init_shell.te`，并在文件的末尾添加以下内容：

```kt
type_transition init_shell ramdisk:file ramdisk_newfile;
```

对`sepolicy`文件，`shell.te`执行以下操作：

```kt
type_transition shell ramdisk:file ramdisk_newfile;
```

现在，我们需要声明新类型；因此，打开`sepolicy`文件，`file.te`，并在末尾添加以下内容：

```kt
type ramdisk_newfile, file_type;
```

请注意，这里我们只使用了`file_type`属性。这是因为文件系统不应该有`ramdisk_newfile`类型，只有位于该文件系统内的文件才应该有。

现在，构建`adb`策略，将其推送到设备上，并触发重新加载。完成这些操作后，创建文件并检查结果：

```kt
$ adb shell 'touch /ramdisk/shell_newfile'
$ adb shell 'ls -laZ /ramdisk'
-rw-rw-rw- root root u:object_r:ramdisk:s0 shell_newfile

```

所以它没有起作用。让我们通过尝试一个`ext4`文件系统的例子来调查原因。我们将使用以下命令：

```kt
root@udoo:/ # cd /data/
root@udoo:/data # mkdir ramdisk

```

现在检查其上下文：

```kt
root@udoo:/data # ls -laZ | grep ramdisk
drwx------ root rootu:object_r:system_data_file:s0 ramdisk

```

标签是`system_data_file`。这并不有用，因为它不适用于我们的类型转换规则；为了修复这个问题，我们可以使用`chcon`命令显式地更改文件的上下文：

```kt
root@udoo:/data # chcon u:object_r:ramdisk:s0 ramdisk
root@udoo:/data # ls -laZ | grep ramdisk
drwx------ root root u:object_r:ramdisk:s0 ramdisk

```

现在，将上下文更改为与我们之前尝试的内存盘相匹配，让我们尝试在这个目录中创建一个文件：

```kt
root@udoo:/data/ramdisk # touch newfile
root@udoo:/data/ramdisk # ls -laZ
-rw------- root root u:object_r:ramdisk_newfile:s0 newfile

```

如你所见，类型转换已经发生。这是为了说明你在使用 SELinux 和 Android 时可能会遇到的问题。既然我们已经证明了我们的`type_transition`语句是有效的，那么失败只有两种可能：文件系统不支持它，或者我们在某个地方遗漏了“开启”它的内容。事实证明是后者；我们遗漏了`fs_use_trans`语句。那么打开`sepolicy`文件，`fs_use`并添加以下行：

```kt
fs_use_trans ramfs u:object_r:ramdisk:s0;
```

这个语句在这个文件系统上启用了 SELinux 动态转换。现在，重建`sepolicy`项目，使用`adb push`推送策略文件，并通过`setprop`启用动态重载：

```kt
$ mmm external/sepolicy
$ adb push $OUT/root/sepolicy /data/security/current/sepolicy546 KB/s (86748 bytes in 0.154s)
$ adb shell setprop selinux.reload_policy 1
root@udoo:/ # cd ramdisk
root@udoo:/ramdisk # touch foo
root@udoo:/ramdisk # ls -Z
-rw------- root root u:object_r:ramdisk_newfile:s0 foo

```

你看，对象具有由动态类型转换确定的正确值。我们遗漏了`fs_use_trans`，它启用了不支持`xattrs`的文件系统上的类型转换。

现在，假设我们想挂载另一个内存盘，会发生什么？由于它被`genfscon`语句标记，所有使用该类型挂载的文件系统都应该得到上下文`u:object_r:ramdisk:s0`。我们将在`/ramdisk2`挂载这个文件系统，并验证这种行为：

```kt
root@udoo:/ # mkdir ramdisk2
root@udoo:/ # mount -t ramfs -o size=20m ramfs /ramdisk2

```

同时，检查上下文：

```kt
root@udoo:/ # ls -laZ | grep ramdisk
drwxr-xr-x root root u:object_r:ramdisk:s0 ramdisk
drwxr-xr-x root root u:object_r:ramdisk:s0 ramdisk2

```

如果我们想要编写允许规则以分隔对这些文件系统的访问，我们需要将它们的目标文件放在不同的类型中。为此，我们可以使用上下文选项挂载新的内存盘。但首先，我们需要创建新的类型；让我们打开`sepolicy`文件，`file.te`并添加一个名为`ramdisk2`的新类型：

```kt
type ramdisk2, file_type, fs_type;
```

现在，使用命令`mmm`构建`sepolicy`，然后使用命令`adb push`推送策略，并通过`setprop`命令触发重载：

```kt
$ mmm external/sepolicy/
$ adb push out/target/product/udoo/root/sepolicy /data/security/current/sepolicy542 KB/s (86703 bytes in 0.155s)
$ adb shell setprop selinux.reload_policy 1

```

在这一点上，让我们卸载`/ramdisk2`并使用`context=`选项重新挂载它：

```kt
root@udoo:/ # umount /ramdisk2/ 
root@udoo:/ # mount -t ramfs -osize=20m,context=u:object_r:ramdisk2:s0 ramfs /ramdisk2

```

现在，验证上下文：

```kt
root@udoo:/ # ls -laZ | grep ramdisk 
drwxr-xr-x root root u:object_r:ramdisk:s0 ramdisk
drwxr-xr-x root root u:object_r:ramdisk2:s0 ramdisk2

```

我们可以使用`mount`选项`context=<context>`覆盖`genfscon`上下文。实际上，如果我们查看`dmesg`，我们可以看到一些很好的信息。当我们没有使用上下文选项挂载`ramfs`时，我们得到了：

```kt
<7>SELinux: initialized (dev ramfs, type ramfs), uses genfs_contexts

```

当我们使用`context=<context>`选项挂载它时，我们得到了：

```kt
<7>SELinux: initialized (dev ramfs, type ramfs), uses mountpoint labeling

```

我们可以看到，当 SELinux 试图找出其标签来源时，它给出了一些有用的信息。

现在，让我们开始给支持`xattr`的文件系统，如`ext4`打标签。我们将从工具箱命令`chcon`开始。`chcon`命令允许你显式地设置文件系统对象的上下文，它不会咨询`file_contexts`。

让我们看看`/system/bin`目录，以及其中的前 10 个文件：

```kt
$ adb shell ls -laZ /system/bin | head -n10
-rwxr-xr-x root shell u:object_r:system_file:s0 InputDispatcher_test
-rwxr-xr-x root shell u:object_r:system_file:s0 InputReader_test
-rwxr-xr-x root shell u:object_r:system_file:s0 abcc
-rwxr-xr-x root shell u:object_r:system_file:s0 adb
-rwxr-xr-x root shell u:object_r:system_file:s0 am
-rwxr-xr-x root shell u:object_r:zygote_exec:s0 app_process
-rwxr-xr-x root shell u:object_r:system_file:s0 applypatch
-rwxr-xr-x root shell u:object_r:system_file:s0 applypatch_static
drwxr-xr-x root shell u:object_r:system_file:s0 asan
-rwxr-xr-x root shell u:object_r:system_file:s0 asanwrappe

```

我们可以看到，其中许多文件都有`system_file`标签，这是该文件系统的默认标签；让我们将`am`类型更改为`am_exec`。同样，我们需要通过向`sepolicy`文件`file.te`中添加以下内容来创建一种新类型：

```kt
type am_exec, file_type;
```

现在，重新构建策略文件，将其推送到 UDO，并触发重新加载。之后，让我们开始重新挂载系统，因为它是只读的：

```kt
root@udoo:/ # mount -orw,remount /system

```

现在执行`chcon`：

```kt
root@udoo:/ # chcon u:object_r:am_exec:s0 /system/bin/am 

```

验证结果：

```kt
root@udoo:/ # la -laZ /system/bin/am 
-rwxr-xr-x root shell u:object_r:am_exec:s0 am

```

此外，`restorecon`命令将使用`file_contexts`，并将该文件恢复为`file_contexts`文件中设置的内容，这应该是`system_file`：

```kt
root@udoo:/ # restorecon /system/bin/am 
root@udoo:/ # la -laZ /system/bin/am 
-rwxr-xr-x root shell u:object_r:system_file:s0 am

```

如你所见，`restorecon`能够咨询`file_contexts`并恢复该对象的指定上下文。

安卓系统的文件系统在构建时进行构造，因此，其所有文件对象在此过程中都被标记。我们还可以在构建时通过更改`file_contexts`来更改这一点。更改后，重新构建系统分区，并在重新刷新系统后，我们应该会看到具有`am_exec`类型的`am`文件。我们可以通过在`system/bin`部分的末尾添加这一行来修改`sepolicy`文件`file_contexts`进行测试：

```kt
/system/bin/am u:object_r:am_exec:s0
```

使用以下命令重新构建整个系统：

```kt
$ make -j8 2>&1 | tee logz

```

现在刷新并重启，然后让我们按照以下方式查看`/system/bin/am`的上下文：

```kt
root@udoo:/ # ls -laZ /system/bin/am 
-rwxr-xr-x root shell u:object_r:am_exec:s0 am

```

这表明系统分区尊重构建时的文件上下文标记，以及我们如何控制这些标签。

## 修复`/data`

在审计日志中，我们还发现了一堆未标记的文件，例如下面的拒绝访问记录：

```kt
type=1400 msg=audit(86559.780:344): avc: denied { append } for pid=2668 comm="UsbDebuggingHan" name="adb_keys" dev=mmcblk0p4 ino=42 scontext=u:r:system_server:s0 tcontext=u:object_r:unlabeled:s0 tclass=file
```

我们可以看到设备是`mmcblk0p4`，挂载命令会告诉我们这个文件系统挂载到了哪里，其输出如下：

```kt
root@udoo:/ # mount | grep mmcblk0p4
/dev/block/mmcblk0p4 /data ext4 rw,seclabel,nosuid,nodev,noatime,nodiratime,errors=panic,user_x0

```

那么`/data`文件系统为什么有这么多未标记的文件呢？原因是 SELinux 应该从空设备开始启用，即从第一次启动时。Android 按需构建数据目录结构。因此，由于它是`ext4`，所有`/data`的标签都由`file_contexts`文件处理。同时，这些由创建`/data`文件和目录的系统处理。这些系统已经被修改为根据`file_contexts`规范对数据分区进行标记。因此，这里有两个选择：擦除`/data`并重启，或者执行`restorecon -R /data`。

第一个选项有点激烈，但如果你弹出 SD 卡并删除数据分区`partition 4`上的所有文件，Android 将重新构建，你就不会再看到任何未标记的问题。然而，对于升级时的已部署设备，这并不推荐；你将破坏所有用户的数据。

在部署场景中，第二个选项更受欢迎，但也有其局限性。特别是，执行`restorecon -R /data`将花费很长时间，并且必须在启动早期，在挂载之后立即进行。然而，目前这确实是唯一的选择。不过，谷歌在这一领域做了大量工作，并创建了一个系统，可以在策略更新时智能地重新标记`/data`。考虑到我们的使用情况，我们将选择第二个选项的一个变体，尤其是考虑到`/data`文件系统的稀疏性；我们实际上还没有安装或生成大量用户数据。基于这一点，执行：

```kt
root@udoo:/ # restorecon -R /data
root@udoo:/ # reboot

```

由于我们的系统处于宽容模式，且不在部署场景中，因此我们无需在启动早期执行`restorecon`。现在，让我们拉取`audit.log`文件，并将其与已拉的`audit.log`进行比较：

```kt
$ adb pull /data/misc/audit/audit.log audit_data_relabel.log
170 KB/s (14645 bytes in 0.084s)

```

让我们使用`grep`来计算每个文件中出现的次数：

```kt
$ grep -c unlabeled audit.log 
185
$ grep -c unlabeled audit_data_relabel.log 
0

```

太棒了，我们已经修复了`/data`上的所有未标记问题！

# 关于安全的补充说明

请注意，尽管我们运行了所有这些命令并更改了所有这些内容，但这并不是 SELinux 中的安全漏洞。更改类型标签、挂载文件系统以及将文件系统与类型关联，都需要允许规则。如果你查看审核日志，你会看到一系列的拒绝记录；以下是一个示例：

```kt
type=1400 msg=audit(90074.080:192): avc: denied { associate } for pid=3211 comm="touch" name="foo" scontext=u:object_r:ramdisk_newfile:s0 tcontext=u:object_r:ramdisk:s0 tclass=filesystem
type=1400 msg=audit(90069.120:187): avc: denied { mount } for pid=3205 comm="mount" name="/" dev=ramfs ino=1992 scontext=u:r:init_shell:s0 tcontext=u:object_r:ramdisk:s0 tclass=filesystem
```

如果我们处于强制模式，我们将无法执行这里展示的任何实验。

# 总结

在本章中，我们看到了如何通过重新标记文件将文件放入上下文中。我们使用了各种技术来完成这项任务，从工具箱命令如`chcon`和`restorecon`，到挂载选项和动态转换。有了这些工具，我们可以确保所有文件系统对象都被正确标记。这样，我们最终得到了正确的目标上下文，以便我们编写的策略能够有效。在下一章中，我们将关注进程，确保它们处于正确的域或上下文中。
