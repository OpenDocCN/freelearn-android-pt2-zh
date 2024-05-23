# 第十二章：掌握工具链

迄今为止，我们已经深入探讨了推动 SE for Android 技术的代码和政策，但构建系统和工具常常被忽视。掌握工具链将帮助你提高开发实践。在本章中，我们将了解 SE for Android 构建的所有组件及其工作原理。我们将涵盖以下主题：

+   构建特定目标

+   sepolicy `Android.mk` 文件

+   自定义构建策略配置

+   构建工具：

    +   `check_seapp`

    +   `insertkeys.py`

    +   `checkpolicy`

    +   `checkfc`

    +   `sepolicy-check`

    +   `sepolicy-analyze`

# 构建子组件——目标和项目

迄今为止，我们已经运行了一些神奇的命令，如 `mm`、`mmm` 和 `make bootimage`，实际上构建了 SE for Android 代码的各个部分。谷歌在文档 [`source.android.com/source/building-running.html`](https://source.android.com/source/building-running.html) 中正式描述了其中一些工具，但大多数命令并未列出。尽管如此，[`elinux.org/Android_Build_System`](http://elinux.org/Android_Build_System) 有一个更全面的相关编写。

在谷歌的“构建和运行”文档中，他们将目标描述为设备，这最终是你启动的目标。在构建 Android 时，`lunch` 命令为稍后执行的 `make` 命令设置环境变量。它设置构建系统以输出目标设备的正确配置。本章将不会讨论这种目标概念。相反，当提到 `target` 时，它指的是一个特定的 `make` 目标。然而，在需要提及目标设备的情况下，将使用完整的短语“`target device`”。虽然有些令人困惑，但这种术语是标准的，现场的工程师将会理解。

我们已经多次运行了 `make` 命令，可选地提供一个目标作为参数和选项，例如 `-j16` 选项。像 `make` 或 `make -j16` 这样的命令本质上构建了整个 Android。可选地，你可以指定一个或多个目标作为命令参数。例如，当构建 `boot.img` 时，可以通过指定 `bootimage` 目标来构建和重新构建 `boot.img` 文件。我们为此目的使用的命令是 `make bootimage`。它通过仅重建系统中需要的部分来加快构建速度。但如果你只需要重新构建一个特定文件呢？或许，你只想重新构建 `sepolicy`。你可以将其指定为构建目标，如 `make sepolicy`。这引出了一个问题：“其他文件如 `mac_permissions.xml`、`seapp_contexts` 等怎么办？”它们也可以以同样的方式构建。更有趣的问题是：“一个人如何知道目标名称是什么？它总是输出文件的名称吗？”

Android 的构建系统是建立在 GNU `make`之上的（[`www.gnu.org/software/make/`](http://www.gnu.org/software/make/)）。Android 构建系统的核心 makefile 系统可以在`build/core`中找到，而文档可以在 NDK 中找到（[`developer.android.com/tools/sdk/ndk/index.html`](https://developer.android.com/tools/sdk/ndk/index.html)）。从阅读中可以得出的主要结论是，一个典型的`Android.mk`文件定义了称为`LOCAL_MODULE := mymodulename`的东西，以及构建名为`mymodulename`的东西。目标名称由这些`LOCAL_MODULE`语句定义。让我们查看外部 sepolicy 的`Android.mk`，关注其中 sepolicy 部分，因为在该`Makefile`中还定义了其他本地模块或目标。以下是从 Android 4.3 的一个示例：

```kt
include $(CLEAR_VARS)
LOCAL_MODULE := sepolicy
LOCAL_MODULE_CLASS := ETC
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_PATH := $(TARGET_ROOT_OUT)
...
```

只需要查找以`LOCAL_MODULE`声明开始的行，并且是全词匹配，就可以在`Android.mk`文件中找到所有模块：

```kt
$ grep -w '^LOCAL_MODULE' Android.mk
LOCAL_MODULE := sepolicy
LOCAL_MODULE := file_contexts
LOCAL_MODULE := seapp_contexts
LOCAL_MODULE := property_contexts
LOCAL_MODULE := selinux-network.sh
LOCAL_MODULE := mac_permissions.xml
LOCAL_MODULE := eops.xml

```

正则表达式规定，`^`表示行的开始，而`grep`的手册页指出`-w`提供全词搜索。

前面的列表对于我们目前在 UDOO 上使用的 Android 版本来说是很全面的。但是，你应该在你的`Makefile`的确切版本上运行命令，以了解可以构建哪些内容。

Android 还有一些额外的工具，这些工具与构建目标分开，并在你使用`source build/envsetup.sh`时添加到你的环境中。这些工具是`mm`和`mmm`。它们执行相同的任务，即构建`Android.mk`文件中指定的所有目标，但不同之处在于它们不构建任何依赖项。这两个命令的区别仅在于它们查找构建目标所在的`Android.mk`的位置。`mm`命令使用当前工作目录，而`mmm`使用提供的路径。此外，这两个命令的一个很好的选项是`-B`，它强制重新构建。工程师使用`mm(m)`命令而不是`make <target>`可以节省大量时间。完整的`make`命令在确定依赖关系树上浪费了很多时间，所以如果在之前构建的源代码树（如果你知道你的所有更改都在一个项目中）上执行`mmm path/to/project`可以节省几分钟。但是，由于它不构建依赖项，你需要确保它们已经构建并且没有依赖性更改。

# 探索 sepolicy 的 Android.mk

位于`external/sepolicy`的项目与其他 Android 项目一样，使用`Android.mk`文件来构建它们的输出。让我们剖析这个文件，看看它都做了些什么。

## 构建 sepolicy

我们将从中间开始，查看针对`sepolicy`的目标。它以相当标准化的`Android.mk`内容开始：

```kt
...
include $(CLEAR_VARS)
LOCAL_MODULE := sepolicy
LOCAL_MODULE_CLASS := ETC
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_PATH := $(TARGET_ROOT_OUT)
include $(BUILD_SYSTEM)/base_rules.mk
...
```

接下来的部分有点类似于标准的`make`。它首先声明一个目标文件，该文件会被构建到`intermediates`位置。`intermediates`位置是由 Android 构建系统定义的。然后它将`MLS_SENS`和`MLS_CATS`的值赋给一些局部变量以供后续使用。最后一条语句是最有趣的。它使用了一个名为`build_policy`的`make`函数，并接受文件名作为参数：

```kt
...
sepolicy_policy.conf := $(intermediates)/policy.conf
$(sepolicy_policy.conf): PRIVATE_MLS_SENS := $(MLS_SENS)
$(sepolicy_policy.conf): PRIVATE_MLS_CATS := $(MLS_CATS)
$(sepolicy_policy.conf) : $(call build_policy, security_classes initial_sids access_vectors global_macros mls_macros mls policy_capabilities te_macros attributes bools *.te roles users initial_sid_contexts fs_use genfs_contexts port_contexts)
...
```

接下来，我们定义了构建此中间目标`policy.conf`的 recipe。recipe 中有趣的部分是`m4`命令和`sed`命令。

### 注意事项

有关`m4`的更多信息，请参见[`www.gnu.org/software/m4/manual/m4.html`](http://www.gnu.org/software/m4/manual/m4.html)，有关`sed`的更多信息，请参考[`www.gnu.org/software/sed/manual/sed.html`](https://www.gnu.org/software/sed/manual/sed.html)。

SELinux 策略文件是通过`m4`处理的。`m4`是一种通常被用作编译器前端的宏处理器语言。`m4`命令取一些值，如`PRIVATE_MLS_SENS`和`PRIVATE_MLS_CATS`，并将它们作为宏定义传递。这类似于`gcc -D`选项。然后它通过`make`扩展`$^`获取目标的依赖项，并使用`make`扩展`$@`将它们输出到目标名称。它还取该输出并生成一个`.dontaudit`版本。该版本使用`sed`从策略文件中删除所有`dontaudit`行。MLS 值告诉 SELinux 生成多少类别和敏感性。这些必须在加载到内核的策略块中静态定义，如下所示：

```kt
...
@mkdir -p $(dir $@)
$(hide) m4 -D mls_num_sens=$(PRIVATE_MLS_SENS) -D mls_num_cats=$(PRIVATE_MLS_CATS) -s $^ > $@
$(hide) sed '/dontaudit/d' $@ > $@.dontaudit
...
```

下一个部分定义了构建实际目标 recipe，该目标名为`LOCAL_MODULE_POLICY`，即使这一点并不明显。`LOCAL_BUILT_MODULE`扩展为要构建的中间文件，在这种情况下是`sepolicy`。最后，它由 Android 构建系统在幕后作为`LOCAL_INSTALLED_MODULE`复制。此目标依赖于中间`policy.conf`文件和`checkpolicy`。它使用`checkpolicy`将`m4`扩展的`policy.conf`和`policy.conf.dontaudit`转换为两个 sepolicy 文件，`sepolicy`和`sepolicy.dontaudit`。实际用于将 SELinux 语句编译成二进制形式以加载到内核的工具是`checkpolicy`，如下所示：

```kt
...
$(LOCAL_BUILT_MODULE) : $(sepolicy_policy.conf) $(HOST_OUT_EXECUTABLES)/checkpolicy
@mkdir -p $(dir $@)
$(hide) $(HOST_OUT_EXECUTABLES)/checkpolicy -M -c $(POLICYVERS) -o $@ $<
$(hide) $(HOST_OUT_EXECUTABLES)/checkpolicy -M -c $(POLICYVERS) -o $(dir $<)/$(notdir $@).dontaudit $<.dontaudit
...
```

最后，它通过设置局部变量`built_policy`供`Android.mk`文件中的其他地方使用，并清除`policy.conf`以避免污染`make`的全局命名空间，如下所示：

```kt
...
built_sepolicy := $(LOCAL_BUILT_MODULE)
sepolicy_policy.conf :=
...
```

此外，构建`sepolicy`还依赖于`POLICYVERS`变量，如果未设置，则条件赋值为`26`。这是`checkpolicy`使用的策略版本号，正如我们在本书前面看到的，我们不得不为我们的 UDOO 覆盖这一点。

## 控制策略构建

我们看到`sepolicy`语句调用了`build_policy`函数。我们还在`Android.mk`文件中看到它的用途，用于构建`sepolicy`、`file_contexts`、`seapp_contexts`、`property_contexts`和`mac_permissions.xml`，因此可以推断这个函数相当重要。该函数输出用于策略文件的完全解析路径列表。该函数以变量参数列表的文件名为输入，并支持正则表达式（注意`build_policy`中的`*.te`用于目标 sepolicy）。在内部，该函数使用一些技巧，允许你覆盖或追加当前策略构建，而无需直接修改`external/sepolicy`目录。这是为了使 OEM 和设备构建者能够增加策略，以覆盖其特定设备。

在构建策略时，你可以在设备的`Makefile`中设置以下`make`变量，以控制生成的构建结果。这些变量如下：

+   `BOARD_SEPOLICY_DIRS`：这是潜在策略文件的搜索路径。

+   `BOARD_SEPOLICY_UNION`：这是一个要附加到所有同名文件的策略文件。

+   `BOARD_SEPOLICY_REPLACE`：这是一个用于覆盖基础`external/sepolicy`策略文件的策略文件。

+   `BOARD_SEPOLICY_IGNORE`：这是用于从构建中移除特定策略文件，给定仓库的相对路径。

以 UDOO 为例，编写策略的正确方式是永远不要修改`external/sepolicy`，而是在`device/fsl/udoo/sepolicy`中创建一个目录：

```kt
$ mkdir <PATH>

```

然后我们修改`BoardConfig.mk`： 

```kt
$ vim BoardConfig.mk

```

接下来，我们添加以下几行：

```kt
BOARD_SEPOLICY_DIRS += device/fsl/udoo/sepolicy
```

### 提示

在使用`+=`与`:=`时要非常小心。在大型项目树中，这些变量可能会在构建树的更高位置被常见的`BoardConfigs`设置，而你可能会覆盖它们的设置。通常，最安全的选择是`+=`。有关详细信息，请参阅 GNU make 手册中的*变量赋值*部分，在[`www.gnu.org/software/make/manual/make.html`](http://www.gnu.org/software/make/manual/make.html)。

这将告诉`Android.mk`中的`build_policy()`函数不仅搜索`external/sepolicy`目录，还要搜索`device/fsl/udoo/sepolicy`目录下的策略文件。

接下来，我们可以在该目录中创建一个`file_contexts`文件，并通过在该目录中创建一个新的`file_contexts`文件，将我们对标签的更改移动到`device/fsl/udoo/sepolicy`目录。

之后，我们需要指导构建系统将我们的`file_contexts`文件与`external/sepolicy`中的文件进行合并或联合。我们通过在`BoardConfig.mk`文件中添加以下声明来实现这一点：

```kt
BOARD_SEPOLICY_UNION += file_contexts
```

你可以对任何策略文件执行此操作，甚至是自定义文件。它仅通过文件名（不包括目录）进行匹配。例如，如果你有一个名为`watchdog.te`的规则文件，你想将其添加到基础的`watchdog.te`规则文件中，你可以像下面这样只添加`watchdog.te`：

```kt
BOARD_SEPOLICY_UNION += file_contexts watchdog.te
```

这将在构建过程中生成一个新的`watchdog.te`文件，将你的新规则与在`external/sepolicy/watchdog.te`中找到的规则进行联合。

还请注意，你可以使用`BOARD_SEPOLICY_UNION`将新文件添加到构建中，因此要为自定义域（如`custom.te`）添加一个`.te`文件，你可以：

```kt
BOARD_SEPOLICY_UNION += file_contexts watchdog.te custom.te
```

假设你想用你自己的文件覆盖`external/sepolicy watchdog.te`。你可以将其添加到`BOARD_SEPOLICY_REPLACE`中，如下所示：

```kt
BOARD_SEPOLICY_REPLACE := watchdog.te
```

请注意，你不能替换在基础策略中不存在的文件。同时，你不能让同一个文件在`UNION`和`REPLACE`中出现，因为这是模棱两可的。在同一个策略文件上不能有超过一个的`BOARD_SEPOLICY_REPLACE`规范。

假设我们正在为两个虚构的设备（设备 X 和设备 Y）进行分层构建。这两个设备，设备 X 和设备 Y，都从设备 A 继承了`BoardConfigCommon.mk`。设备 A 不是一个真实的设备，但由于 X 和 Y 有共同点，因此将共同的部分保存在设备 A 中。

假设设备 A 的`BoardConfigCommon.mk`包含以下语句：

```kt
BOARD_SEPOLICY_DIRS += device/OEM/A
BOARD_SEPOLICY_UNION += file_contexts custom.te
```

假设设备 X 的`BoardConfig.mk`包含以下内容：

```kt
BOARD_SEPOLICY_DIRS += device/OEM/X
BOARD_SEPOLICY_UNION += file_contexts custom.te
```

最后，假设设备 Y 的`BoardConfig.mk`包含以下内容：

```kt
BOARD_SEPOLICY_DIRS += device/OEM/Y
BOARD_SEPOLICY_UNION += file_contexts custom.te
```

用于构建设备 X 和设备 Y 的结果策略集如下：

设备 X 策略集：

```kt
device/OEM/A/file_contexts
device/OEM/A/custom.te
device/OEM/X/file_contexts
device/OEM/X/custome.te
external/sepolicy/* (base policy files)
```

设备 Y 也包含以下内容：

```kt
device/OEM/A/file_contexts
device/OEM/A/custom.te
device/OEM/Y/file_contexts
device/OEM/Y/custom.te
external/sepolicy/* (base policy files)
```

在一个常见场景中，你可能不希望设备 Y 的结果策略集包含`device/OEM/A/custom.te`。这是`BOARD_SEPOLICY_IGNORE`的一个用例。你可以用它来过滤特定的策略文件。但是，你必须具体指明并使用仓库的相对路径。例如，在设备 Y 的`BoardConfig.mk`中：

```kt
BOARD_SEPOLICY_IGNORE += device/OEM/A/custom.te
```

现在，当你为设备 Y 构建策略时，策略集将不会包括那个文件。`BOARD_SEPOLICY_IGNORE`也可以与`BOARD_SEPOLICY_REPLACE`一起使用，允许在设备层次结构中多次使用，但只有一个`BOARD_SEPOLICY_REPLACE`语句生效。

## 深入挖掘 build_policy

现在我们已经了解了如何使用一些新的机制来控制策略构建，让我们实际剖析构建过程发生在哪里。如前所述，策略构建由`Android.mk`文件控制。我们之前遇到了对`build_policy()`函数的调用，这正是与我们所设置的所有的`BOARD_SEPOLICY_*`变量相关的魔法发生的地方。检查`build_policy`函数，我们看到它引用了`sepolicy_replace_paths`变量，所以让我们从查看这个变量开始。

`sepolicy_replace_paths`变量在`Makefile`执行时进行评估，换句话说，它会无条件执行。代码首先遍历所有的`BOARD_SEPOLICY_REPLACE`文件，检查是否存在于`BOARD_SEPOLICY_UNION`中。如果发现一个，就会打印错误信息，构建失败，显示`Ambiguous request for sepolicy $(pf). Appears in both BOARD_SEPOLICY_REPLACE and BOARD_SEPOLICY_UNION`，其中`$(pf)`会被扩展为有问题的策略文件。之后，它用`BOARD_SEPOLICY_DIRS`设置的搜索路径中找到的条目来扩展`BOARD_SEPOLICY_REPLACE`，从而得到从 Android 树的根目录开始的完整相对路径。然后它将这些条目与`BOARD_SEPOLICY_IGNORE`进行过滤，删除任何应该被忽略的内容。接着确保只找到一个替换的文件候选。否则，它会发出适当的错误信息。最后，它会确保文件存在于`LOCAL_PATH`或基础策略中，如果两者都找不到，它会发出错误信息：

```kt
...
# Quick edge case error detection for BOARD_SEPOLICY_REPLACE.
# Builds the singular path for each replace file.
sepolicy_replace_paths :=
$(foreach pf, $(BOARD_SEPOLICY_REPLACE), \
  $(if $(filter $(pf), $(BOARD_SEPOLICY_UNION)), \
    $(error Ambiguous request for sepolicy $(pf). Appears in both \
      BOARD_SEPOLICY_REPLACE and BOARD_SEPOLICY_UNION), \
  ) \
  $(eval _paths := $(filter-out $(BOARD_SEPOLICY_IGNORE), \
  $(wildcard $(addsuffix /$(pf), $(BOARD_SEPOLICY_DIRS))))) \
  $(eval _occurrences := $(words $(_paths))) \
  $(if $(filter 0,$(_occurrences)), \
    $(error No sepolicy file found for $(pf) in $(BOARD_SEPOLICY_DIRS)), \
  ) \
  $(if $(filter 1, $(_occurrences)), \
    $(eval sepolicy_replace_paths += $(_paths)), \
    $(error Multiple occurrences of replace file $(pf) in $(_paths)) \
  ) \
  $(if $(filter 0, $(words $(wildcard $(addsuffix /$(pf), $(LOCAL_PATH))))), \
    $(error Specified the sepolicy file $(pf) in BOARD_SEPOLICY_REPLACE, \
      but none found in $(LOCAL_PATH)), \
  ) \
)
```

在此之后，构建策略的调用可以使用`replace_paths`作为在构建期间将被替换的文件的扩展列表。

`build_policy`函数的参数是您希望使用`BOARD_SEPOLICY_*`系列变量的提供的功能扩展到它们的 Android 根相对路径名称的文件名。例如，在我们的设备 A、X 和 Y 的上下文中调用`$(build_policy, file_contexts)`将导致如下结果：

```kt
device/OEM/A/file_contexts
device/OEM/Y/file_contexts
```

`build_policy`函数的阅读有点棘手。许多嵌套的函数调用导致最深的缩进首先运行。然而，像所有代码一样，我们从上到下，从左到右阅读，因此解释将从这里开始。该函数首先遍历作为参数传递的所有文件。然后针对`BOARD_SEPOLICY_DIRS`进行一次替换和一次联合的扩展。检查`sepolicy_replace_paths`变量以确保文件没有同时出现在替换和联合的位置。对于替换路径的扩展，它会检查扩展后的路径是否在`sepolicy_replace_dirs`中，如果是，则替换它。对于联合部分，它只是进行扩展。这些扩展的结果随后通过`BOARD_SEPOLICY_IGNORE`的过滤器，从而删除任何明确忽略的路径：

```kt
# Builds paths for all requested policy files w.r.t
# both BOARD_SEPOLICY_REPLACE and BOARD_SEPOLICY_UNION
# product variables.
# $(1): the set of policy name paths to build
build_policy = $(foreach type, $(1), \
  $(filter-out $(BOARD_SEPOLICY_IGNORE), \
    $(foreach expanded_type, $(notdir $(wildcard $(addsuffix /$(type), $(LOCAL_PATH)))), \
      $(if $(filter $(expanded_type), $(BOARD_SEPOLICY_REPLACE)), \
        $(wildcard $(addsuffix $(expanded_type), $(sort $(dir $(sepolicy_replace_paths))))), \
        $(LOCAL_PATH)/$(expanded_type) \
      ) \
    ) \
    $(foreach union_policy, $(wildcard $(addsuffix /$(type), $(BOARD_SEPOLICY_DIRS))), \
      $(if $(filter $(notdir $(union_policy)), $(BOARD_SEPOLICY_UNION)), \
        $(union_policy), \
      ) \
    ) \
  ) \
)
...
```

## 构建 mac_permissions.xml

如我们在第十章《将应用置于域中》所见，`mac_permissions.xml`的构建有点棘手。首先，`mac_permissions.xml`可以与迄今为止引入的所有`BOARD_SEPOLICY_*`变量一起使用。最终结果是生成一个符合这些变量规则的 XML 文件。此外，原始 XML 文件由位于`sepolicy/tools`目录下的一个名为`insertkeys.py`的工具处理。`insertkeys.py`工具使用`keys.conf`将 XML 文件签名区域的标签与包含证书的`.pem`文件进行映射。`keys.conf`文件同样适用于`BOARD_SEPOLICY_*`变量。构建配方首先对`keys.conf`调用`build_policy`，并使用`m4`连接结果。因此，`keys.conf`中的`m4`声明将被尊重。然而，这尚未被使用。最初的意图是使用`m4 -s`同步行，以便在`m4`处理连接`keys.conf`文件时，您可以跟随包含链。另一方面，当连接多个文件时，`m4`会提供同步行，并提供符合`#line NUM "FILE"'`格式的注释行。这些很有用，因为`m4`将多个输入文件合并成一个扩展的输出文件。将会有指示每个文件开头的同步行，它们可以帮助您追踪问题。回到`mac_permissions.xml`的构建，经过`m4`对`keys.conf`的扩展后，该文件以及通过调用`build_policy()`获取的所有`mac_permissions.xml`文件最终被传递给`insertkeys.py`。`insertkeys.py`工具然后使用`keys.conf`文件将所有匹配的`signature=<TAG>`行替换为来自 PEM 文件的十六进制编码的实际 X509，即`signature=308E3600`。此外，`insertkeys.py`工具将 XML 文件合并为一个文件，并去除空格和注释以减少其在磁盘上的大小。这不会对其他主要文件如`sepolicy`、`seapp_contexts`、`property_contexts`和`mac_permissions.xml`产生构建依赖。

## 构建 seapp_contexts

`seapp_contexts`文件也受所有`BOARD_SEPOLICY_*`变量的影响。来自`build_policy()`调用结果的所有`seapp_contexts`文件也通过`m4 -s`处理，以获得包含同步行的单个`seapp_contexts`文件。同样，类似于`mac_permissions.xml`文件的`keys.conf`构建，除了用于 synclines 之外，没有使用`m4`。这个结果，连接的`seapp_contexts`文件随后被输入到`check_seapp`中。这个工具是用 C 编程语言编写的，并在构建过程中编译成可执行文件。源代码可以在`tools/check_seapp`中找到。此工具读取`seapp_contexts`文件并检查其语法。它验证没有无效的键值对，`levelFrom`是一个有效的标识符，并且对于给定的`sepolicy`，类型和域字段是有效的。此构建依赖于`sepolicy`，以对策略文件中的域和类型字段进行严格类型检查。

## 构建 file_contexts

`file_contexts`文件也受所有`BOARD_SEPOLICY_*`变量的影响。生成的集合通过`m4 -s`处理，单一输出通过`checkfc`工具运行。`checkfc`工具检查文件的语法和句法，并验证在构建的`sepolicy`中存在这些类型。因此，它依赖于`sepolicy`构建。

## 构建 property_contexts

`property_contexts`的行为与`file_contexts`构建完全一样，只不过它检查一个`property_contexts`文件。它也使用`checkfc`。

## 当前国家安全局的研究文件

此外，国家安全局已经在企业运营（`eops`）方面开始了工作。由于此功能尚未合并到主流 Android 中，并且可能会发生巨大变化，因此这里不会介绍。然而，对于追求前沿技术的最佳地点始终是源代码和国家安全局的 Bitbucket 仓库。`selinux-network.sh`也属于这一类；它尚未被主流采用，并且可能会从 AOSP 中删除（[`android-review.googlesource.com/#/c/114380/`](https://android-review.googlesource.com/#/c/114380/)）。

# 独立工具

还有一些为 Android 策略评估构建的独立工具，你可能会觉得有用。我们将探讨其中一些及其用途。大多数在其他参考资料中找到的标准桌面工具仍然适用于 SE for Android SELinux 策略。请注意，如果你运行以下任何工具并遇到段错误，你可能需要应用来自[thread at http://marc.info/?l=seandroid-list&m=141684060409894&w=2](http://marc.info/?l=seandroid-list&m=141684060409894&w=2)的补丁。

## sepolicy-check

这个工具允许你查看策略文件中是否存在给定的允许规则。其命令的基本语法如下：

```kt
sepolicy-check -s <domain> -t <type> -c <class> -p <permission> -P <policy_file>

```

例如，如果你想查看`system_app`是否可以写入`system_data_file`的 class 文件，你可以执行：

```kt
$ sepolicy-check -s system_app -t system_data_file -c file -p write -P $OUT/root/sepolicy

```

## sepolicy-analyze

这是一个检查 SELinux 开发中常见问题的好工具，它捕捉到了一些新的 SELinux 策略编写者容易犯的常见陷阱。它可以检查等价域、重复的允许规则。它还可以执行策略类型差异检查。

域等价性检查功能非常有帮助。它能显示你可能（从理论上讲）希望不同的域，尽管在实际实现中它们可能已经收敛。这些类型的域将是合并的理想候选者。然而，它也可能揭示了政策设计中应该修正的问题。换句话说，你原本不期望这些域是等价的。调用该命令如下所示：

```kt
$ sepolicy-analyze -e -P $OUT/root/sepolicy

```

重复的允许规则检查是否存在这样的规则：类型上存在允许规则，而这些规则也存在于该类型继承的属性上。由于在属性上已经有一个`allow`规则，因此特定类型上的允许规则是删除的候选者。要执行此检查，请运行以下命令：

```kt
$sepolicy-analyze -D -P $OUT/root/sepolicy

```

在文件内查看域类型差异的功能也很有用。如果你想了解两个域之间的差异，可以使用这个功能。这对于识别可能的合并域很有帮助。要执行此检查，请执行以下命令：

```kt
$sepolicy-analyze -d -P $OUT/root/sepolicy

```

# 总结

在本章中，我们介绍了控制设备上策略的各种组件是如何实际构建和创建的，例如`sepolicy`和`mac_permissions.xml`。本章还介绍了用于跨设备和配置管理构建策略的`BOARD_SEPOLICY_*`变量。然后我们回顾了`Android.mk`组件，详细说明了构建和配置管理核心的工作原理。
