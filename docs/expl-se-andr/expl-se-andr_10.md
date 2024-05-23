# 第十章：将应用程序置于域中

在第三章，*安卓古怪*，我们介绍了 zygote，所有应用程序（在安卓中称为 APK）都源自 zygote，就像服务源自`init`进程一样。因此，它们需要被标记，正如我们在前一章所做的那样。回想一下，标记等同于将进程放置在相应标签的域中。应用程序也需要被标记。

### 注意

APK 是安卓上可安装应用程序包的文件扩展名和格式。它类似于桌面包格式，如 RPM（基于 Redhat）或 DEB（基于 Debian）。

在本章中，我们将学习：

+   正确标记应用程序的私有数据目录及其运行时上下文

+   进一步检查 zygote 及其安全方法

+   了解一个完成的`mac_permssions.xml`文件是如何分配`seinfo`值的

+   创建一个新的自定义域

# 保护 zygote 的情况

安卓上具有提升权限和能力的应用程序是从 zygote 中产生的。一个例子就是系统服务器，这是一个由本地和非本地代码组成的大型进程，提供各种服务。系统服务器包含了活动管理器、包管理器、GPS 信息等。系统服务器也以高度敏感的`system` UID（`1000`）运行。此外，许多 OEM 将所谓的**系统应用**打包，这些是使用`system` UID 独立运行的应用程序。

zygote 还产生不需要提升权限的应用程序。所有第三方应用程序都属于这一类。第三方应用程序以自己的 UID 运行，与敏感的 UID（如`system`）分开。此外，应用程序会被放入各种 UID 中，如`media`、`nfc`等。OEM 倾向于定义额外的 UID。

需要注意的是，要进入像`system`这样的特殊 UID，你必须使用适当的密钥签名。安卓有四个主要密钥用于签名应用程序：`media`、`platform`、`shared`和`testkey`。它们位于`build/target/product/security`目录中，以及一个`README`文件。

根据`README`，密钥使用如下：

+   `testkey`：对于那些没有指定密钥的包的通用密钥。

+   `platform`：为核心平台部分包的测试密钥。

+   `shared`：用于在 home/contacts 进程中共享事物的测试密钥。

+   `media`：用于媒体/下载系统中部分的包的测试密钥。

为了为你的应用程序请求`system` UID，你必须使用`platform`密钥进行签名。在这些更加特权的环境中执行，需要拥有私钥。

如您所见，我们的应用程序在不同的权限级别和信任级别下执行。我们不能信任第三方应用程序，因为它们是由未知实体创建的，而我们可以信任使用我们的私钥签名的实体。然而，在 SELinux 之前，应用程序权限仍然受到与第一章中提到的*Linux 访问控制*相同的 DAC 权限限制。由于这些特性，zygote 成为了攻击的主要目标，同时也需要用 SELinux 来加固。

# 加固 zygote

既然我们已经确定了 zygote 的问题，下一步就是了解如何将应用程序放入适当的域中。我们需要 SELinux 策略或代码更改来将新进程放入一个域中。在第九章中，我们讨论了基于 init 服务的动态域转换，并在章节末尾提到了`exec()`系统调用在“应用程序标签限制”部分的重要性。这是动态域转换发生的触发器。如果路径中没有`exec`，我们将不得不依赖代码更改。但是，在这个安全模型中，我们还必须考虑签名密钥，而纯粹的 SELinux 策略语言无法表达进程签名的密钥。

我们不必探索整个 zygote，可以剖析以下引入应用程序标签到 Android 的补丁。此外，我们可以发现引入的设计如何满足尊重签名密钥、在 SELinux 和 zygote 的设计内工作的要求。

## 管理 zygote 套接字

在第三章中，我们了解到 zygote 通过监听套接字来等待请求启动新的应用程序。要检查的第一个补丁是[`android-review.googlesource.com/#/c/31066/`](https://android-review.googlesource.com/#/c/31066/)。这个补丁修改了 Android 基础框架中的三个文件。第一个文件是`Process.java`中的`startViaZygote()`方法。这个方法是相对于构建字符串参数并将它们通过`zygoteSendArgsAndGetResult()`传递给 zygote 的其他方法的主要入口点。补丁引入了一个名为`seinfo`的新参数。稍后，我们将看到如何使用它。看起来这个补丁正在通过套接字传输这个新的`seinfo`参数。请注意，这段代码是在 zygote 进程外部调用的。

在这个补丁中要查看的下一个文件是`ZygoteConnection.java`。这段代码从上下文中执行。补丁首先在`ZygoteConnection`类中声明了一个字符串成员变量`peerContext`。在构造函数中，这个`peerContext`成员被设置为调用`SELinux.getPeerContext(mSocket.getFileDescriptor())`得到的值。

由于底层的`LocalSocket` `mSocket`是 Unix 域套接字，你可以获取连接客户端的凭据。在这种情况下，调用`getPeerContext()`获取客户端的安全上下文，或者更正式地说，是进程标签。初始化后，在方法`runOnce()`中进一步向下，我们看到它在调用`applyUidSecurityPolicy`和其他`apply*SecurityPolicy`例程时被使用。受保护的`runOnce()`方法被调用以从套接字读取一个启动命令和参数。最终，在`apply*SecurityPolicy`检查之后，它调用`forkandSpecialize()`。每个安全策略检查都已修改为在现有的 DAC 安全控制之上使用 SELinux。如果我们审查`applyUidSecurityPolicy`，我们会看到他们进行如下调用：

```kt
boolean allowed = SELinux.checkSELinuxAccess(peerSecurityContext, peerSecurityContext, "zygote", "specifyids");
```

这是一个用户空间利用强制访问控制的示例，这在对象管理器中是众所周知的。此外，在`applyseInfoSecurityPolicy()`方法中为神秘的`seinfo`字符串添加了一个安全检查。这里所有的 SELinux 安全检查都指定了目标类`zygote`。所以如果我们查看`sepolicy access_vectors`，我们会看到添加的类`zygote`。这是 Android 的一个自定义类，定义了所有在安全检查中检查的向量。

我们将从这个补丁中考虑的最后一个文件是`ActivityManagerService.java`。`ActivityManager`负责启动应用程序并管理它们的生命周期。它是`Process.start` API 的使用者，需要指定`seinfo`。这个补丁很简单，目前只是发送了`null`。稍后，我们将看到启用其使用的补丁。

下一个补丁，[`android-review.googlesource.com/#/c/31063/`](https://android-review.googlesource.com/#/c/31063/)，在 Android Dalvik VM 的上下文中执行，并在 VM zygote 进程空间中编码。我们在`ZygoteConnection`中看到的`forkAndSpecialize()`最终进入了这个本地例程。它通过`static pid_t forkAndSpecializeCommon(const u4* args, bool isSystemServer)`进入。这个例程负责创建成为应用程序的新进程。

它从 Java 开始，将清理代码移动到 C，并设置 C 风格字符串的`niceName`和`seinfo`值。最终，代码调用`fork()`，子进程开始执行操作，如执行`setgid`和`setuid`。`uid`和`gid`值通过`Process.start`方法指定给 zygote 连接。我们还看到一个对`setSELinuxContext()`的新调用。顺便说一下，这些事件的顺序在这里很重要。如果你太早设置新进程的 SELinux 上下文，那么进程在新上下文中需要额外的能力才能执行像`setuid`和`setgid`这样的操作。然而，这些权限最好留给`zygote`域，这样我们进入的应用程序域可以尽可能最小化。

接着，`setSELinuxContext`最终调用了`selinux_android_setcontext()`。注意，在这个提交之后，移除了`HAVE_SELINUX`条件编译宏，但在 4.3 版本发布之前。还要注意，`selinux_android_setcontext()`在`libselinux`中定义，所以我们的旅程将带我们到那里。在这里我们看到神秘的`seinfo`仍然在传递。

下一个要评估的补丁是[`android-review.googlesource.com/#/c/39601/`](https://android-review.googlesource.com/#/c/39601/)。这个补丁实际上从 Java 层传递了一个更有意义的`seinfo`值。这个补丁没有设置为`null`，而是引入了从 XML 文件中解析的逻辑，并将其传递给`Process.start`方法。

这个补丁修改了两个主要组件：`PackageManager`和`installd`。`PackageManager`在`system_server`内部运行，执行应用程序安装。它维护系统中所有已安装包的状态。第二个组件，称为`installd`的服务，是一个非常特权级的 root 服务，在磁盘上创建所有应用程序的私有目录。这种方法不是给系统服务器，因此`PackageManager`提供创建这些目录的能力，只有`installd`拥有这些权限。即使系统服务器也无法读取您的私有数据目录中的数据，除非您将其设置为全局可读。

这个补丁比其他的要大，因此我们只检查与讨论直接相关的部分。我们将从查看`PackageManagerService.java`开始。这个类是 Android 的包管理器。在`PackageManagerService()`的构造函数中，我们看到了添加了`mFoundPolicyFile = SELinuxMMAC.readInstallPolicy();`这一行。

根据命名，我们可以推测这个方法是在寻找某种策略配置文件，如果找到，返回 true，并设置`mFoundPolicyFile`成员变量。我们还看到一些对`createDataDirs`和`mInstaller.*`的调用。我们可以忽略这些，因为那些调用是发送给`installd`的。

下一个主要部分添加了以下内容：

```kt
if (mFoundPolicyFile) {
  SELinuxMMAC.assignSeinfoValue(pkg);
}
```

重要的是要注意这段代码被添加到了`scanPackageLI()`方法中。每次需要扫描包以进行安装时，都会调用这个方法。因此，在高级别上，如果在服务启动期间找到某些策略文件，那么就会为包分配一个`seinfo`值。

下一个要查看的文件是`ApplicationInfo.java`，这是一个用于维护关于包的元信息的容器类。正如我们所见，`seinfo`值在这里指定以供存储。此外，还有一些通过 Android 特定的`Parcel`实现序列化和反序列化类的代码。

在这一点上，我们应该仔细查看`SELinuxMMAC.java`代码，以确认我们对正在发生的事情的理解。这个类开始时声明了两个策略文件的位置。

```kt
// Locations of potential install policy files.
private static final File[] INSTALL_POLICY_FILE = {
  new File(Environment.getDataDirectory(), "system/mac_permissions.xml"),
  new File(Environment.getRootDirectory(), "etc/security/mac_permissions.xml"),
  null };
```

根据这个，策略文件可以存在于两个位置：`/data/system/mac_permissions.xml`和`/system/etc/security/mac_permissions.xml`。最终，我们看到`PackageManagerService`初始化时对类中定义的方法`readInstallPolicy()`的调用，最终简化为以下调用：

```kt
private static boolean readInstallPolicy(File[] policyFiles) {
  FileReader policyFile = null;
  int i = 0;
  while (policyFile == null && policyFiles != null && policyFiles[i] != null) {
    try {
      policyFile = new FileReader(policyFiles[i]);
      break;
    } catch (FileNotFoundException e) {
      Slog.d(TAG,"Couldn't find install policy " + policyFiles[i].getPath());
    }
  i++;
  }
...
```

当`policyFiles`设置为`INSTALL_POLICY_FILE`时，这段代码使用数组在指定位置查找文件。它是基于优先级的，`/data`位置优先于`/system`。这个方法中的其余代码看起来像解析逻辑，并填充了在类声明中定义的两个哈希表：

```kt
// Signature seinfo values read from policy.
private static final HashMap<Signature, String> sSigSeinfo =
new HashMap<Signature, String>();
// Package name seinfo values read from policy.
private static final HashMap<String, String> sPackageSeinfo =
new HashMap<String, String>();
```

`sSigSeinfo`将`Signatures`（或签名密钥）映射到`seinfo`字符串。另一个映射`sPackageSeinfo`将包名映射到字符串。

在这一点上，我们可以从`mac_permissions.xml`文件中读取一些格式化的 XML，并从签名密钥到`seinfo`以及包名到`seinfo`创建内部映射。

`PackageManagerService` 类调用这个类的另一个方法来自于 `void assignSeinfoValue(PackageParser.Package pkg)`。

让我们调查一下这个方法能做什么。它首先检查应用程序是否为系统 UID 或系统安装的应用程序。换句话说，它检查应用程序是否为第三方应用程序：

```kt
if (((pkg.applicationInfo.flags & ApplicationInfo.FLAG_SYSTEM) != 0) ||
((pkg.applicationInfo.flags & ApplicationInfo.FLAG_UPDATED_SYSTEM_APP) != 0)) {
```

这段代码后来被谷歌删除，最初是合并的要求。然而，我们可以继续进行评估。代码遍历包中的所有签名，并与哈希表进行对比。如果它使用该映射中的某个内容签名，它就会使用关联的`seinfo`值。另一种情况是它通过包名匹配。在任一情况下，包的`ApplictionInfo`类的`seinfo`值都会更新以反映这一点，并供`installd`和 zygote 应用程序生成在其他地方使用：

```kt
// We just want one of the signatures to match.
for (Signature s : pkg.mSignatures) {
  if (s == null)
    continue;
  if (sSigSeinfo.containsKey(s)) {
    String seinfo = pkg.applicationInfo.seinfo = sSigSeinfo.get(s);
    if (DEBUG_POLICY_INSTALL)
      Slog.i(TAG, "package (" + pkg.packageName +
        ") labeled with seinfo=" + seinfo);
    return;
    }
  }
  // Check for seinfo labeled by package.
  if (sPackageSeinfo.containsKey(pkg.packageName)) {
    String seinfo = pkg.applicationInfo.seinfo = sPackageSeinfo.get(pkg.packageName);
    if (DEBUG_POLICY_INSTALL)
      Slog.i(TAG, "package (" + pkg.packageName +
        ") labeled with seinfo=" + seinfo);
      return;
    }
  }
}
```

顺便一提，主线 AOSP（Android Open Source Project）中合并的内容与 NSA 在 Bitbucket 仓库中维护的内容略有不同。NSA 在这些策略文件中有额外的控制，可能导致应用程序安装被终止。可以说，谷歌和 NSA 在这个问题上“分道扬镳”。在 NSA 版本的`SELinuxMMAC.java`中，你可以指定匹配特定签名或包名的应用程序被允许拥有某些 Android 级别的权限集。例如，你可以阻止所有请求`CAMERA`权限的应用程序安装，或者阻止使用某些密钥签名的应用程序。这也突显了在大型代码库中找到补丁并快速了解项目如何发展的重要性，这往往可能显得有些困难。

在这个补丁中，我们需要考虑的最后一个文件是`ActivityManagerService.java`。这个补丁用`app.info.seinfo`替换了 null。经过所有这些工作和管道铺设，我们最终有了完全解析的神秘的`seinfo`值，与每个应用程序包关联，并传递给 zygote，在`selinux_android_setcontext()`中使用。

现在让我们回顾一下，我们希望在标记应用程序时实现的一些属性。其中之一是以某种方式将安全上下文与应用程序签名密钥耦合，这正是 `seinfo` 的主要好处。这是一个高度敏感且受信任的与签名密钥相关联的字符串值。字符串的实际内容是任意的，在 `mac_permissions.xml` 中指定，这是我们冒险旅程的下一站。

## `mac_permissions.xml` 文件

`mac_permissions.xml` 文件的名字非常容易混淆。展开来看，名字是 MAC 权限。然而，其主要主流功能是将签名密钥映射到一个 `seinfo` 字符串。其次，它还可以用于配置非主流的安装时权限检查功能，称为安装时 MMAC。MMAC 控制是国家安全局（NSA）在中层实现强制访问控制工作的一部分。MMAC 代表“中间件强制访问控制”。谷歌没有合并任何 MMAC 功能。但是，由于我们使用了 NSA 的 Bitbucket 仓库，我们的代码库包含了这些功能。

`mac_permissions.xml` 是一个 XML 文件，应遵循以下规则，其中斜体部分仅在 NSA 分支上支持：

+   签名是一个十六进制编码的 X.509 证书，每个签名者标签都需要。

+   `<signer signature="" >` 元素可能有多个子元素：

    +   `allow-permission`：它生成一组最大允许的权限集合（白名单）。

    +   `deny-permission`：它生成一个要拒绝的权限黑名单。

    +   `allow-all`：这是一个通配符标签，将允许所有请求的权限。

    +   `package`：这是一个复杂的标签，定义了一个特定包名的签名保护的允许、拒绝和通配符子元素。

+   零个或多个全局 `<package name="">` 标签是被允许的。这些标签允许在特定包名的外部设置策略，不受任何签名限制。

+   允许使用 `<default>` 标签，其中可以包含未使用先前列出的证书签名的所有应用的安装策略，且没有每个包的全局策略。

+   任何级别的未知标签将被跳过。

+   零个或多个签名者标签是被允许的。

+   每个签名者标签允许零个或多个包标签。

+   `<package name="">` 标签可能不包含另一个 `<package name="">` 标签。如果发现，则跳过。

+   当一个标签出现多个子元素时，以下逻辑用于最终确定执行类型：

    +   如果至少找到一个 deny-permission 标签，则使用黑名单。

    +   如果没有黑名单，则使用白名单，并且至少找到一个 allow-permission 标签。

    +   如果没有黑名单和白名单，且至少存在一个 allow-all 标签，则使用通配符（接受所有权限）策略。

    +   如果找到 `<package name="">` 子元素，则根据之前的逻辑使用该子元素的策略，并覆盖任何签名全局策略类型。

    +   为了使策略段落得到执行，至少需要满足前述情况之一。这意味着，不接受空签名人、默认或软件包标签。

+   每个`signer/default/package`（全局或附加到签名人）标签允许包含一个`<seinfo value=""/>`标签。这个标签表示每个应用程序可以在设置 SELinux 安全上下文时使用的附加信息，在最终的处理过程中。

+   在大多数情况下，并不严格执行任何 XML 段落的规则。这主要适用于允许的重复标签。如果已经存在一个标签，则原始标签将被替换。

+   同时也没有检查权限名称的有效性。尽管预期是有效的安卓权限，但并未阻止未知权限。

+   以下是执行决策：

    +   用于签署应用程序的所有签名都将根据签名人标签检查策略。然而，只有一个签名策略需要通过。

    +   如果所有的签名策略都未通过，或者没有任何匹配项，那么将寻求全局软件包策略。如果找到，此策略将调解安装。

    +   如果需要，最后将咨询默认标签。

    +   本地软件包策略总是覆盖任何父策略。

    +   如果没有任何情况适用，那么应用程序将被拒绝。

以下示例忽略了安装 MMAC 支持，并专注于`seinfo`映射的主要用途。以下是将所有使用平台密钥签名的项映射到`seinfo`值平台的段落映射示例：

```kt
<!-- Platform dev key in AOSP -->
<signer signature="@PLATFORM" >
  <seinfo value="platform" />
</signer>
```

下面是一个将使用发布密钥签名的所有内容映射到发布域的示例，但浏览器除外。浏览器被分配了一个`seinfo`值为`browser`，如下所示：

```kt
<!-- release dev key in AOSP -->
<signer signature="@RELEASE" >
  <seinfo value="release" />
  <package name="com.android.browser" >
  <seinfo value="browser" />
  </package>
</signer>
...
```

任何具有未知密钥的内容，都会被映射到默认标签：

```kt
...
<!-- All other keys -->
<default>
  <seinfo value="default" />
</default>
```

签名标签值得关注，`@PLATFORM`和`@RELEASE`是在构建期间使用的特殊处理字符串。另一个映射文件将这些映射到实际的关键值。处理过的文件被放置在设备上，所有密钥引用都被替换为十六进制编码的公钥，而不是这些占位符。它还删除了所有的空白和注释，以减少大小。让我们通过从设备中提取构建的文件并格式化它来查看。

```kt
$ adb pull /system/etc/security/mac_permissions.xml
$ xmllint --format mac_permissions.xml

```

现在，滚动到格式化输出的顶部，你应该看到以下内容：

```kt
<?xml version="1.0" encoding="iso-8859-1"?>
<!-- AUTOGENERATED FILE DO NOT MODIFY -->
<policy>
  <signer signature="308204ae30820396a003020102020900d2cba57296ebebe2300d06092a864886f70d0101050500308196310b300906035504061302555331133...
dec513c8443956b7b0182bcf1f1d">
    <allow-all/>
    <seinfo value="platform"/>
  </signer>
```

请注意，`signature=@PLATFORM`现在是一个十六进制字符串。这个十六进制字符串是一个有效的 X509 证书。

## keys.conf

实际上，从`mac_permissions.xml`中的`signature=@PLATFORM`到`keys.conf`的映射才是魔法所在。这个配置文件允许你将一个 pem 编码的 x509 映射到一个任意的字符串。约定是使用`@`开始，但这不是强制性的。该文件的格式基于 Python 配置解析器，并包含部分。部分名称是你在`mac_permissions.xml`文件中希望用密钥值替换的标签。平台示例是：

```kt
[@PLATFORM]
ALL : $DEFAULT_SYSTEM_DEV_CERTIFICATE/platform.x509.pem
```

在 Android 中，构建时你可以有三个级别的构建：`engineering`，`userdebug` 或 `user`。在 `keys.conf` 文件中，你可以将一个密钥与 `ALL` 区段属性关联以用于所有级别，或者你可以为每个级别分配不同的密钥。这对于使用非常特殊的发布密钥构建发布或用户版本很有帮助。我们在 `@RELEASE` 区段看到了一个这样的例子：

```kt
[@RELEASE]
ENG       : $DEFAULT_SYSTEM_DEV_CERTIFICATE/testkey.x509.pem
USER      : $DEFAULT_SYSTEM_DEV_CERTIFICATE/testkey.x509.pem
USERDEBUG : $DEFAULT_SYSTEM_DEV_CERTIFICATE/testkey.x509.pem
```

该文件还允许通过传统的 `$` 特殊字符使用环境变量。pem 文件的默认位置是 `build/target/product/security`。然而，你*绝不能*将这些密钥用于用户发布版本。这些密钥是 AOSP 测试密钥，是公开的！这样做的话，任何人都可以使用系统密钥来签署他们的应用并获得系统权限。`keys.conf` 文件只在构建过程中使用，并且不在系统上。

## seapp_contexts

到目前为止，我们已经了解了完成的 `mac_permssions.xml` 文件如何分配 `seinfo` 值。现在我们应该探讨标记实际上是如何配置并使用这个值的。应用程序的标记是在另一个配置文件 `seapp_contexts` 中管理的。与 `mac_permissions.xml` 一样，它被加载到设备上。然而，默认位置是 `/seapp_contexts`。`seapp_contexts` 的格式是每行遵循 `key=value` 对映射，以下规则：

+   输入选择器：

    +   `isSystemServer`（布尔值）

    +   `user`（字符串）

    +   `seinfo`（字符串）

    +   `name`（字符串）

    +   `sebool`（字符串）

+   输入选择器规则：

    +   `isSystemServer=true` 只能使用一次。

    +   未指定的 `isSystemServer` 默认为 false。

    +   未指定的字符串选择器将匹配任何值。

    +   以 `*` 结尾的用户字符串选择器将执行前缀匹配。

    +   `user=_app` 将匹配任何常规的应用 UID。

    +   `user=_isolated` 将匹配任何隔离服务 UID。

    +   一个条目中所有指定的输入选择器必须匹配（逻辑与）。

    +   匹配不区分大小写。

    +   优先级规则如下：

        +   `isSystemServer=true` 优先于 `isSystemServer=false`

        +   指定的 `user=` 字符串优先于未指定的 `user=` 字符串。

        +   修复了 `user=` 字符串，使其优先于以 `*` 结尾的 `user=` 前缀。

        +   较长的 `user=` 前缀优先于较短的前缀。

        +   指定的 `seinfo=` 字符串优先于未指定的 `seinfo=` 字符串。

        +   指定的 `name=` 字符串优先于未指定的 `name=` 字符串。

        +   指定的 `sebool=` 字符串优先于未指定的 `sebool=` 字符串。

+   输出：

    +   `domain`（字符串）：它指定了应用程序的进程域。

    +   `type`（字符串）：它指定了应用程序私有数据目录的磁盘标签。

    +   `levelFrom`（字符串；值为 `none`，`all`，`app` 或 `user`）：它给出了 MLS 指示符。

    +   `level`（字符串）：它显示硬编码的 MLS 值。

+   输出规则：

    +   只有指定了 `domain=` 的条目会被用于应用进程标记。

    +   只有指定了 `type=` 的条目才会用于应用目录标记。

    +   `levelFrom=user` 只支持 `_app` 或 `_isolated` UIDs。

    +   `levelFrom=app` 或 `levelFrom=all` 只支持 `_app` UIDs。

    +   `level` 可用于为任何 UID 指定固定的级别。

在应用程序生成期间，`selinux_android_setcontext()` 和 `selinux_android_setfilecon2()` 函数会使用此文件来查找适当的应用程序域或文件系统上下文。这些函数的源代码可以在 `external/libselinux/src/android.c` 中找到，推荐阅读。例如，以下条目将所有具有 UID `bluetooth` 的应用程序放在 `bluetooth` 域中，数据目录标签为 `bluetooth_data_file`：

```kt
user=bluetooth domain=bluetooth type=bluetooth_data_file
```

此示例将所有第三方或“默认”应用程序放入 `untrusted_app` 的进程域和 `app_data_file` 的数据目录中。它还使用基于 MLS 的 `levelFrom=app` 类别以帮助提供额外的分离。

```kt
user=_app domain=untrusted_app type=app_data_file levelFrom=app
```

目前，此功能是实验性的，因为它破坏了一些已知的应用程序兼容性问题。在撰写本文时，这成为了谷歌和美国国家安全局工程师的热门关注点。由于它是实验性的，让我们验证其功能，然后禁用它。

我们还没有安装任何第三方应用程序，因此我们需要安装一个以便进行实验。FDroid 是一个寻找第三方应用程序的好地方，因此我们可以从那里下载并安装一些内容。我们可以使用位于 [`f-droid.org/repository/browse/?fdid=org.zeroxlab.zeroxbenchmark`](https://f-droid.org/repository/browse/?fdid=org.zeroxlab.zeroxbenchmark) 的 `0xbenchmark` 应用程序，APK 下载地址为 [`f-droid.org/repo/org.zeroxlab.zeroxbenchmark_9.apk`](https://f-droid.org/repo/org.zeroxlab.zeroxbenchmark_9.apk)，如下所示：

```kt
$ wget https://f-droid.org/repo/org.zeroxlab.zeroxbenchmark_9.apk
$ adb install org.zeroxlab.zeroxbenchmark_9.apk 
567 KB/s (1193455 bytes in 2.052s)
pkg: /data/local/tmp/org.zeroxlab.zeroxbenchmark_9.apk
Success

```

### 提示

检查 `logcat` 中的安装时 `seinfo` 值：

```kt
$ adb logcat | grep SELinux
I/SELinuxMMAC( 2557): package (org.zeroxlab.zeroxbenchmark) installed with seinfo=default

```

从 UDOO 中启动 `0xbenchmark` APK。我们应在 `ps` 中看到它正在运行，并带有其标签：

```kt
$ adb shell ps -Z | grep untrusted
u:r:untrusted_app:s0:c40,c256 u0_a40 17890 2285 org.zeroxlab.zeroxbenchmark

```

注意上下文字符串中的级别部分 `s0:c40,c256`。这些类别是在 `seapp_contexts` 中使用 `level=app` 设置创建的。

要禁用它，我们可以简单地从 `seapp_contexts` 中的条目中删除 level 的键值对，或者我们可以利用 `sebool` 条件赋值。让我们使用布尔值方法。修改 sepolicy `seapp_contexts` 文件，以便修改现有的 `untrusted_app` 条目，并添加一个新条目。将 `user=_app domain=untrusted_app type=app_data_file` 更改为 `user=_app sebool=app_level domain=untrusted_app type=app_data_file levelFrom=app`。

使用 `mmm external/sepolicy` 进行构建，如下所示：

```kt
Error:
out/host/linux-x86/bin/checkseapp -p out/target/product/udoo/obj/ETC/sepolicy_intermediates/sepolicy -o out/target/product/udoo/obj/ETC/seapp_contexts_intermediates/seapp_contexts out/target/product/udoo/obj/ETC/seapp_contexts_intermediates/seapp_contexts.tmp
Error: Could not find selinux boolean "app_level" on line: 42 in file: out/target/product/udoo/obj/ETC/seapp_contexts_intermediates/seapp_contexts
Error: Could not validate

```

好吧，在 `seapp_contexts` 的第 42 行有一个构建错误，抱怨找不到 `selinux` 布尔值。让我们尝试通过声明布尔值来纠正问题。在 `app.te` 中添加：`bool app_level false;`。现在将新构建的 `seapp_contexts` 和 sepolicy 文件推送到设备上，并触发动态重载：

```kt
$ adb push $OUT/root/sepolicy /data/security/current/
$ adb push $OUT/root/seapp_contexts /data/security/current/
$ adb shell setprop selinux.reload_policy 1

```

我们可以通过以下方式验证布尔值是否存在：

```kt
$ adb shell getsebool -a | grep app_level
app_level --> off

```

由于设计限制，我们需要卸载并重新安装应用程序：

```kt
$ adb uninstall org.zeroxlab.zeroxbenchmark

```

在启动进程后，重新安装并检查进程的上下文内容：

```kt
$ adb shell ps -Z | grep untrusted
u:r:untrusted_app:s0:c40,c256 u0_a40 17890 2285 org.zeroxlab.zeroxbenchmark

```

很好！它失败了。在经过一些调试后，我们发现问题的根源是 `/data/security` 路径不是全局可搜索的，导致 DAC 权限失败。

### 注意

我们通过在 `android.c` 中打印结果和错误代码找到这个，我们看到在检查 `fp = fopen(seapp_contexts_file[i++], "r")` 的结果时，`selinux_android_seapp_context_reload()` 中的 `seapp_contexts_file[]` 数组（按优先级排序的文件）上的 `fopen`，并使用 `selinux_log()` 将数据转储到 `logcat`。

```kt
$ adb shell ls -la /data | grep security
drwx------ system system 1970-01-04 00:22 security

```

请记住，`set selinux` 上下文发生在 UID 切换之后，因此我们需要使其对其他人可搜索。我们可以通过更改 `device/fsl/imx6/etc/init.rc` 中的 UDOO `init.rc` 脚本的权限来修复权限。具体来说，将行 `mkdir /data/security 0700 system system` 更改为 `mkdir /data/security 0711 system system`。构建并刷新 `bootimage`，然后再次尝试上下文测试。

```kt
$ adb uninstall org.zeroxlab.zeroxbenchmark
$ adb install ~/org.zeroxlab.zeroxbenchmark_9.apk
<launch apk>
$ adb shell ps -Z | grep org.zeroxlab.zeroxbenchmark
u:r:untrusted_app:s0 u0_a40 3324 2285 org.zeroxlab.zeroxbenchmark

```

迄今为止，我们已经演示了如何使用 `seapp_contexts` 上的 `sebool` 选项来禁用 MLS 类别。需要注意的是，在更改 APK 的类别或类型时，需要卸载并重新安装 APK，否则在大多数情况下，由于没有访问权限，该进程会与其数据目录脱离。

接下来，让我们拿这个 APK，卸载它，并通过更改其 `seinfo` 字符串为其分配一个唯一的域。通常，你使用这个特性将一组用共同密钥签名的应用程序放入自定义域以执行自定义操作。例如，如果你是 OEM，你可能需要允许未用 OEM 控制的密钥签名的第三方应用程序拥有自定义权限。首先卸载 APK：

```kt
$ adb uninstall org.zeroxlab.zeroxbenchmark

```

通过添加以下内容在 `mac_permissions.xml` 中创建一个新条目：

```kt
<signer signature="@BENCHMARK" >
<allow-all />
<seinfo value="benchmark" />
</signer>

```

现在，我们需要为 `keys.conf` 获取一个 pem 文件。因此，解压 APK 并提取公共证书：

```kt
$ mkdir tmp
$ cd tmp
$ unzip ~/org.zeroxlab.zeroxbenchmark_9.apk
$ cd META-INF/
$ $ openssl pkcs7 -inform DER -in *.RSA -out CERT.pem -outform PEM  -print_certs

```

我们需要从生成的 `CERT.pem` 文件中删除任何多余的内容。如果你打开它，你应该会在顶部看到这些行：

```kt
subject=/C=UK/ST=ORG/L=ORG/O=fdroid.org/OU=FDroid/CN=FDroid
issuer=/C=UK/ST=ORG/L=ORG/O=fdroid.org/OU=FDroid/CN=FDroid
-----BEGIN CERTIFICATE-----
MIIDPDCCAiSgAwIBAgIEUVJuojANBgkqhkiG9w0BAQUFADBgMQswCQYDVQQGEwJV
SzEMMAoGA1UECBMDT1JHMQwwCgYDVQQHEwNPUkcxEzARBgNVBAoTCmZkcm9pZC5v
...
```

它们需要被删除，因此只删除主题和发行者行。文件应以 `BEGIN CERTIFICATE` 开头，以 `END CERTIFICATE` 剪切线结尾。

让我们将这个移动到工作区中名为 `certs` 的新文件夹，并将证书移动到这个文件夹，并赋予其一个更好的名字：

```kt
$ mkdir UDOO_SOURCE_ROOT/certs
$ mv CERT.pem UDOO_SOURCE_ROOT/certs/benchmark.x509.pem

```

我们可以通过添加以下内容来设置 `keys.conf`：

```kt
[@BENCHMARK]
ALL : certs/benchmark.x509.pem

```

别忘了更新 `seapp_contexts` 以使用新的映射：

```kt
user=_app seinfo=benchmark domain=benchmark_app type=benchmark_app_data_file

```

现在声明要使用的新类型。域类型应在 `sepolicy` 中名为 `benchmark_app.te` 的文件中声明：

```kt
# Declare the new type
type benchmark_app, domain;
# This macro adds it to the untrusted app domain set and gives it some allow rules
# for basic functionality as well as object access to the type in argument 2.
untrustedapp_domain(benchmark_app, benchmark_app_data_file)

```

还在 `file.te` 中添加 `benchmark_app_data_file`：

```kt
type benchmark_app_data_file, file_type, data_file_type, app_public_data_type;

```

### 提示

你可能并不总是想要这些*所有*属性，尤其是如果你在做一些安全关键的事情。确保你查看每个属性和宏以及其用法。你不想因为过于宽松的域而打开一个未预期的大门。

重新构建策略，推送所需的部分，并触发重新加载。

```kt
$ mmm external/sepolicy/
$ adb push $OUT/system/etc/security/mac_permissions.xml /data/security/current/
$ adb push $OUT/root/sepolicy /data/security/current/
$ adb push $OUT/root/seapp_contexts /data/security/current/
$ adb shell setprop selinux.reload_policy 1

```

启动一个 shell 并使用 grep logcat 查看基准测试 APK 安装时的 `seinfo` 值。然后安装该 APK：

```kt
$ adb install ~/org.zeroxlab.zeroxbenchmark_9.apk
$ adb logcat | grep -i SELinux

```

在`logcat`输出中，你应该看到：

```kt
I/SELinuxMMAC( 2564): package (org.zeroxlab.zeroxbenchmark) installed with seinfo=default

```

它应该是`seinfo=benchmark`！可能发生了什么？

问题出在`frameworks/base/services/java/com/android/server/pm/SELinuxMMAC.java`中。它查看`/data/security/mac_permissions.xml`；所以我们可以直接推送`mac_permissions.xml`。这是动态策略重载中的另一个错误，与加载过程中历史更改有关。罪魁祸首在`frameworks/base/services/java/com/android/server/pm/SELinuxMMAC.java`文件中：

```kt
private static final File[] INSTALL_POLICY_FILE = {
new File(Environment.getDataDirectory(), "security/mac_permissions.xml"),
new File(Environment.getRootDirectory(), "etc/security/mac_permissions.xml"),
null};
```

为了解决这个问题，重新挂载`system`并将其推送到默认位置。

```kt
$ adb remount
$ adb push $OUT/system/etc/security/mac_permissions.xml /system/etc/security/

```

这不需要`setprop selinux.reload_policy 1`。卸载并重新安装基准测试 APK，并检查日志：

```kt
I/SELinuxMMAC( 2564): package (org.zeroxlab.zeroxbenchmark) installed with seinfo=default

```

好的，它仍然没有工作。当我们检查代码时，发现`mac_permissions.xml`文件在包管理器服务启动时被加载。没有重启的情况下，这个文件不会被重新加载，所以让我们卸载基准测试 APK，并重启 UDOО。启动后，启用`adb`，触发动态重载，安装 APK，并检查`logcat`。它应该包含：

```kt
I/SELinuxMMAC( 2559): package (org.zeroxlab.zeroxbenchmark) installed with seinfo=benchmark

```

现在让我们通过启动 APK，检查`ps`，并验证其应用程序私有目录来验证进程域：

```kt
<launch apk>
$ adb shell ps -Z | grep org.zeroxlab.zeroxbenchmark
u:r:benchmark_app:s0 u0_a45 3493 2285 org.zeroxlab.zeroxbenchmark
$ adb shell ls -Z /data/data | grep org.zeroxlab.zeroxbenchmark
drwxr-x--x u0_a45 u0_a45 u:object_r:benchmark_app_data_file:s0 org.zeroxlab.zeroxbenchmark

```

这一次，所有类型都检查通过了。我们成功创建了一个新的自定义域。

# 总结

在本章中，我们研究了如何通过配置文件和 SELinux 策略正确标记应用程序的私有数据目录及其运行时上下文。我们还探讨了使这一切正常工作的子系统及代码，以及在此过程中可能出错的一些基本问题。在下一章中，我们将通过查看 SE for Android 构建系统，详细介绍策略和配置文件是如何构建的。
