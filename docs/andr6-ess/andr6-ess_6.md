# 第六章：安卓工作

众所周知，安卓设备在全球市场占有巨大的市场份额，越来越多的企业采用**BYOD**（即**自带设备**）政策。这是借助**Android for Work**实现的，这是一个针对公司的特殊程序，其中安卓平台增加了多项功能，以便更好地管理移动设备，在公司内部进行管理和整合。

在处理企业甚至小型和中小型企业时，你需要遵循特定的指导方针，利用安卓 API 为你服务。你可以通过以下链接了解更多关于 Android for Work 的信息：

[`developer.android.com/training/enterprise/index.html`](http://developer.android.com/training/enterprise/index.html)

安卓棉花糖对 Android for Work 程序进行了一些更改，其中许多更改旨在为开发者和工作用户带来更好、更简单的使用体验。

在本章中，我们将介绍与 Android for Work 直接相关的安卓棉花糖的更改：

+   行为变更

+   单次使用设备改进

+   静默安装/卸载应用

+   改进的证书访问

+   自动系统更新

+   第三方证书安装

+   数据使用统计

+   管理运行时权限

+   VPN 访问和显示

+   工作资料状态

# 行为变更

安卓棉花糖引入了一些与 Android for Work 相关的行为变更。

## 工作资料联系人显示选项

使用以下设置，你现在可以在拨号器通话记录中显示你的工作资料联系人：

```kt
DevicePolicyManager.setCrossProfileCallerIdDisabled(ComponentName admin, boolean disabled)
```

你还可以使用新的选项通过蓝牙显示工作联系人。将此设置为`false`将允许显示；默认值为`true`（禁用联系人共享选项）：

```kt
DevicePolicyManager.setBluetoothContactSharingDisabled(ComponentNa me admin, boolean disabled)
```

## Wi-Fi 配置选项

通过工作资料添加 Wi-Fi 网络时，通常添加的配置在资料被删除后仍然保持持久。现在，如果工作资料被删除，所有由资料所有者添加的配置都会被移除。

## Wi-Fi 配置锁定

新增了一个`Settings.Global`设置：

```kt
WIFI_DEVICE_OWNER_CONFIGS_LOCKDOWN
```

此设置是一个整数值设置，这意味着零值或不存在将导致用户修改或删除所有 Wi-Fi 配置。将整数值设置为非零值将启动锁定，这意味着用户无法修改或删除设备所有者创建的 Wi-Fi 配置——用户创建的配置仍然可以修改。注意，活跃的设备所有者拥有任何 Wi-Fi 配置的完全权限，即使这些配置不是由他们创建的。

## 工作策略控制器添加

你可以继续向设备添加 Google 账户，但现在，在添加由**工作策略控制器**管理的账户时，流程已更改，以包括工作策略控制器的添加。添加的账户会提示用户安装适当的工作策略控制器。通过设置或通过启动设备设置向导添加账户时也是如此。有关如何构建工作策略控制器的更多信息，请阅读：

[`developer.android.com/training/enterprise/work-policy-ctrl.html`](http://developer.android.com/training/enterprise/work-policy-ctrl.html)

## `DevicePolicyManager`的变化

在`DevicePolicyManager`中，你可能会遇到一些行为上的变化；以下是与简短解释一起列出的一些变化：

+   `setCameraDisabled()`仅影响调用用户摄像头；如果配置文件是受管理的配置文件，那么该调用不会影响主用户上运行的摄像头应用。

+   `setKeyguardDisabledFeatures()`现在对配置文件所有者和设备所有者可用。

+   配置文件所有者可以通过以下方式设置键盘锁限制：

    +   `KEYGUARD_DISABLE_TRUST_AGENTS`：这将忽略在安全屏幕（PIN 码、图案或密码屏幕）上的键盘锁上的信任代理状态。

    +   `KEYGUARD_DISABLE_FINGERPRINT`：这将禁用在安全屏幕（PIN 码、图案或密码屏幕）上的键盘锁上的指纹传感器。

    +   `KEYGUARD_DISABLE_UNREDACTED_NOTIFICATIONS`：这将允许在安全键盘锁屏幕上只显示编辑过的通知，并且只有由管理配置文件中的应用程序生成的通知。

+   `createAndInitializeUser()`现在已被弃用。

+   `createUser()`现在已被弃用。

+   使用`setScreenCaptureDisabled()`方法，可以阻止`Assist`功能，但这仅在给定用户的应用程序在前台时发生。

+   `EXTRA_PROVISIONING_DEVICE_ADMIN_PACKAGE_CHECKSUM`现在使用 SHA-256。对 SHA-1 的传统支持仍然存在，但根据文档，它将在未来的版本中被移除。

+   `EXTRA_PROVISIONING_DEVICE_ADMIN_SIGNATURE_CHECKSUM`现在仅使用 SHA-256。

+   移除了`EXTRA_PROVISIONING_RESET_PROTECTION_PARAMETERS`，以防止 NFC 碰撞配置解锁一个受工厂重置保护的设备。

+   在 NFC 配置过程中，可以通过`EXTRA_PROVISIONING_ADMIN_EXTRAS_BUNDLE`向设备所有者传递数据。

+   新的`DevicePolicyManager` API 支持在 Android Marshmallow 新的权限模型下进行权限管理。你可以通过访问[`developer.android.com/reference/android/app/admin/DevicePolicyManager.html`](https://developer.android.com/reference/android/app/admin/DevicePolicyManager.html)了解更多关于`DevicePolicyManager`的信息。

+   如果用户取消了通过`ACTION_PROVISION_MANAGED_PROFILE`或`ACTION_PROVISION_MANAGED_DEVICE`意图启动的设置流程，现在将返回`RESULT_CANCELED`。

+   `Settings.Global`发生了变化。

+   通过`setGlobalSettings()`禁用以下设置：

    +   `BLUETOOTH_ON`

    +   `DEVELOPMENT_SETTINGS_ENABLED`

    +   `MODE_RINGER`

    +   `NETWORK_PREFERENCE`

    +   `WIFI_ON`

+   启用了通过`setGlobalSettings()`设置的`WIFI_DEVICE_OWNER_CONFIGS_LOCKDOWN`设置。

# 单次使用设备改进

作为设备所有者，你现在可以控制添加的设置，从而通过以下方式改善设备管理：

+   可以使用`setKeyguardDisabled()`来禁用或重新启用键盘锁

+   可以使用`setStatusBarDisabled()`来禁用或重新启用状态栏

+   `UserManager.DISALLOW_SAFE_BOOT`是一个新的常量，用于指示用户是否可以将设备启动到安全模式

+   `Settings.Global.STAY_ON_WHILE_PLUGGED_IN`将防止在连接电源时屏幕关闭

# 静默安装/卸载应用

现在，你可以使用`PackageInstaller` API 静默安装和卸载应用程序。这意味着可以在没有用户交互的情况下安装应用程序，甚至可以按照公司政策删除应用程序。此功能使你可以在不实际激活 Google 账户的情况下使用设备。**Google Play for Work**不是必需的，允许你将设备用作**展示柜**，展示尚未发布的特定应用等。

# 改进了证书访问

在 Android Marshmallow 之前，允许用户在无用户交互的情况下授予管理应用程序访问证书是不可能的，现在，已添加了一个新的回调：

```kt
DeviceAdminReceiver.onChoosePrivateKeyAlias (Context context, Intent intent, int uid, Uri uri, String alias)
```

此回调将允许设备所有者向请求的应用程序静默提供别名。

# 自动系统更新

在 Android 6.0 中添加了以下选项，其主要目的是允许设备所有者自动接受系统更新：

```kt
DevicePolicyManager.setSystemUpdatePolicy (ComponentName admin, SystemUpdatePolicy policy)
```

`SystemUpdatePolicy`也已添加，你可以从三个选项中选择：

+   `TYPE_INSTALL_AUTOMATIC`：一收到更新就更新

+   `TYPE_INSTALL_WINDOWED`：更新应该在定时系统维护期间完成，并且仅在那之后，持续 30 天，然后恢复正常行为

+   `TYPE_POSTPONE`：最多推迟 30 天的更新，之后恢复正常行为

如果你拥有如展示平板或展示柜模式的设备，此功能将非常有用，因为更新不应该干扰设备的工作。

# 第三方证书安装

第三方应用程序现在可以调用`DevicePolicyManager` API：

+   `getInstalledCaCerts()`

+   `hasCaCertInstalled()`

+   `installCaCert()`

+   `uninstallCaCert()`

+   `uninstallAllUserCaCerts()`

+   `installKeyPair()`

只有在设备所有者或配置文件所有者授权后，才能执行这些 API 调用。

# 数据使用统计

在 Android 6.0 中添加了一个新类：`NetworkStatsManager`。这可以帮助你查询可以在**设置** | **数据使用**中看到的数据使用统计信息。

配置文件所有者可以自动获得访问权限，以便查询其配置文件上的数据。设备所有者可以访问管理的主用户的 数据使用情况。

### 注意

`android.app.usage.NetworkUsageStats`类已重命名为`NetworkStats`。

# 管理运行时权限

Android Marshmallow 引入了运行时权限模型，Android for Work 必须处理设备上的策略管理。作为设备所有者，你现在可以使用`setPermissionPolicy()`为所有应用程序的所有运行时请求设置策略。

你可以选择提示用户授予权限，或者自动静默地授予权限或拒绝权限。自动策略意味着用户不能在**设置**中修改应用的权限屏幕。

# VPN 访问和显示

当你进入**设置** | **更多** | **VPN**时，现在可以查看 VPN 应用。使用 VPN 时，显示的通知现在会具体到该 VPN 的配置方式：

+   **配置文件所有者**：根据 VPN 配置和配置文件（个人、工作或两者都有）显示通知

+   **设备所有者**：当 VPN 配置为整个设备时，会显示通知

# 工作配置文件状态

为了让用户知道他们处于不同的配置文件中，引入了两项新的指示：

+   使用工作配置文件中的应用程序时，状态栏将显示一个公文包图标

+   当直接从工作配置文件应用解锁设备时，会弹出一个提示，告知用户此应用运行在工作配置文件上。

# 概述

正如本章所看到的，Android Marshmallow 为 Android for Work 领域带来了许多变化。作为开发者，我们需要始终关注组织的实际需求。我们需要确保深入了解 Android for Work 的世界；Marshmallow 中的变化帮助我们构建和针对企业工作流程，同时得益于更简单的 API。

在下一章中，我们将学习 Chrome 自定义标签页 API 的使用和流程。
