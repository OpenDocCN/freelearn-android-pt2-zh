# 第五章：音频、视频和摄像头特性

Android Marshmallow 为我们提供了良好的音频、视频和摄像头功能，并且可以看到已经进行了改进，以启用和更好地支持新的或全新的协议，甚至改变一些 API 的行为，如摄像头服务。

在本章中，我们将尝试通过适当的讨论来解释这些变化及其使用方法和好处。在接下来的几页中，我们的旅程将涵盖以下主题：

+   音频特性

+   视频特性

+   摄像头特性

# 音频特性

Android Marshmallow 6.0 对音频特性增加了一些增强功能，我们将在接下来的章节中进行介绍。

## 对 MIDI 协议的支持

`android.media.midi`包是在 Android 6.0（API 23）中新增的。

使用新的 midi API，现在可以比以前更简单的方式发送和接收**MIDI**（即**Musical Instrument Digital Interface**，乐器数字接口）事件。

该软件包旨在为我们提供以下能力：

+   连接并使用 MIDI 键盘

+   连接到其他 MIDI 控制器

+   使用外部 MIDI 合成器、外部外围设备、灯光、展示控制等

+   允许游戏或音乐创作应用动态生成音乐

+   允许应用之间创建和传递 MIDI 消息

+   允许 Android 设备在连接到笔记本电脑时作为多点触控控制器

在处理 MIDI 时，您必须在清单中声明，如下所示：

```kt
<uses-feature android:name="android.software.midi" android:required="true"/>
```

注意`required`部分；与其他特性类似，将其设置为`true`将使得只有当设备支持 MIDI API 时，您的应用才会在播放商店中可见。

您还可以在运行时检查 MIDI 支持，然后将 required 部分更改为`false`：

```kt
PackageManager pkgMgr = context.getPackageManager();
if (pkgMgr.hasSystemFeature(PackageManager.FEATURE_MIDI)) {
  //we can use MIDI API here as we know the device supports the MIDI API.
}
```

### MidiManager

正确使用 MIDI API 的一种方式是通过`MidiManager`类；通过`context`获取它，并在需要时使用：

```kt
MidiManager midiMgr = (MidiManager)context.getSystemService(Context.MIDI_SERVICE);
```

如需了解更多信息，您可以参考以下内容：

[`developer.android.com/reference/android/media/midi/package-summary.html`](https://developer.android.com/reference/android/media/midi/package-summary.html)

## 数字音频捕获和播放

为数字音频捕获和播放新增了两个类：

+   `android.media.AudioRecord.Builder` - 数字音频捕获

+   `android.media.AudioTrack.Builder` - 数字音频播放

这些将帮助配置音频源和接收器属性。

## 音频和输入设备

新增了`hasMicrophone()`方法到`InputDevice`类中。这将报告设备是否有开发者可用的内置麦克风。例如，您想要从连接到 Android TV 的控制器启用语音搜索，并在用户搜索时收到`onSearchRequested()`回调。然后，您可以验证在回调中获得的`inputDevice`对象是否有麦克风。

## 音频设备信息

新的`AudioManager.getDevices(int flags)`方法允许轻松检索当前连接到系统的所有音频设备。如果你想要在音频设备连接/断开时得到通知，可以通过`AudioManager.registerAudioDeviceCallback(AudioDeviceCallback callback, Handler handler)`方法注册你的应用到`AudioDeviceCallback`回调中。

## `AudioManager`的变化

`AudioManager`类中引入了一些变化，如下所示：

+   直接使用`AudioManager`来设置音量是不被支持的。

+   使用`AudioManager`静音特定流是不被支持的。

+   `AudioManager.setStreamSolo(int streamType, boolean state)`方法已弃用。如果你需要独占音频播放，请使用`AudioManager.requestAudioFocus(AudioManager.OnAudioFocusChangeListener l, int streamType, int durationHint)`。

+   `AudioManager.setStreamMute(int streamType, boolean state)`方法已弃用。如果你需要使用`AudioManager.adjustStreamVolume(int streamType, int direction, int flags)`来调整音量方向，你可以使用其中新增的常量之一。

+   `ADJUST_MUTE`将会静音。注意，如果该流已经静音，此操作无效。

+   `ADJUST_UNMUTE`将会取消静音。注意，如果该流未被静音，此操作无效。

# 视频特性

在 Android 棉花糖版本中，视频处理 API 已经升级，具备新的功能。一些新的方法甚至一个全新的类`MediaSync`只为开发者添加。

## `android.media.MediaSync`

全新的`MediaSync`类被设计用来帮助我们同步渲染音频和视频流。你也可以使用它来仅播放音频或视频流。你可以使用动态播放速率，并通过回调返回非阻塞动作来输入缓冲区。关于正确使用方法，请阅读：

[关于`MediaSync`的官方文档](https://developer.android.com/reference/android/media/MediaSync.html)

## `MediaCodecInfo.CodecCapabilities.getMaxSupportedInstances` 方法

现在，我们有了`MediaCodecInfo.CodecCapabilities.getMaxSupportedInstances`辅助方法，以获取支持的同时编解码器实例的最大数量。然而，我们只能将此视为上限。实际的同时实例数量可能会根据设备和在用时可用资源的数量而减少。

## 为什么我们需要知道这些？

假设我们有一个媒体播放应用，我们想在播放的电影之间添加效果。这将需要使用多个视频编解码器，解码两个视频，并将一个视频流重新编码以显示在屏幕上。通过检查这个 API，你可以添加更多依赖于多个编解码器实例的功能。

## `MediaPlayer.setPlaybackParams` 方法

我们现在可以设置媒体播放速率，以实现快速或慢速动作播放。这将给我们一个机会，创建一个有趣的应用，其中我们可以放慢部分动作或快速播放，从而在播放时创建一个新视频。音频播放相应地同步，因此您可能会听到一个人慢慢地或快速地说话。

# 相机功能

在 Android Lollipop 中，有新的`Camera2` API，现在在 Android Marshmallow 中，对相机、手电筒和图像重新处理功能有一些更多的更新。

## 手电筒 API

现在几乎每个设备都有相机，几乎每个相机设备都有闪光单元。添加了`setTorchMode()`方法来控制闪光手电筒模式。

`setTorchMode()`方法的使用方式如下：

```kt
CameraManager.setTorchMode (String cameraId, boolean enabled)
```

`cameraId`元素是您想要更改手电筒模式的闪光单元相机的唯一 ID。您可以使用`getCameraIdList()`获取相机列表，然后使用`getCameraCharacteristics(String cameraId)`检查该相机是否支持闪光。`setTorchMode()`方法允许您在不打开相机设备且不需要从相机请求权限的情况下打开或关闭它。一旦相机设备不可用，或者具有开启手电筒的其他相机资源不可用时，手电筒模式将会关闭。其他应用也可以使用闪光单元，因此您需要在需要时检查模式，或者通过`registerTorchCallback()`方法注册一个回调。

参考示例应用**Torchi**，查看完整代码：

[`github.com/MaTriXy/Torchi`](https://github.com/MaTriXy/Torchi)

### 注意

如果相机或其他相机资源正在使用中，开启手电筒模式可能会失败。

## 重新处理 API

如前所述，`Camera2` API 得到了一些增强，以支持**YUV**和私有不透明格式图像的重新处理。在使用这个 API 之前，我们需要检查这些功能是否可用。这就是为什么我们要使用`getCameraCharacteristics(String cameraId)`方法，并检查`REPROCESS_MAX_CAPTURE_STALL`键的原因。

### android.media.ImageWriter

这是一个在 Android 6.0 中新增的类。

它允许我们创建一个图像并将其输入到表面，然后返回到`CameraDevice`。通常，`ImageWriter`与`ImageReader`一起使用。

### android.media.ImageReader

这是一个在 Android 6.0 中新增的类。

它允许我们直接访问在表面上渲染的图像数据。`ImageReader`加上`ImageWriter`允许我们的应用从相机创建图像流到表面，然后返回相机进行重新处理。

## 相机服务的改变

安卓棉花糖对*先到先得*的访问模型进行了更改；现在，服务访问模型有了偏好进程——那些被标记为高优先级的进程。这个变化导致我们开发人员需要处理更多逻辑相关的工作。我们需要确保考虑到这种情况：我们的优先级被提升（更高优先级）或被降级（由于应用程序的变化而降低优先级）。

让我们尝试用几个简单的要点来解释这个问题：

+   当你想访问摄像头资源或打开及配置摄像头设备时，你的访问权限将根据应用程序进程的*优先级*进行验证。通常，具有前台活动（对用户可见）的应用程序进程会被赋予更高的优先级，这也使得在需要时获得所需访问权限的可能性更大。

+   在*优先级*的另一个极端，你会发现低优先级的应用程序可能会被搁置（从访问中撤销），当一个高优先级的应用程序尝试使用摄像头时。例如，当使用`Camera` API 时，你会在被踢出时收到`onError()`调用；当使用`Camera2` API 时，你会在被踢出时收到`onDisconnected()`调用。

+   野外一些设备能够允许不同的应用程序同时打开和使用不同的摄像头设备。摄像头服务现在可以检测并禁止由于多进程使用造成的性能问题。当服务检测到这类问题时，即使只有一个应用程序在使用该摄像头设备，它也会踢出低优先级的应用程序。

+   在多用户环境中，切换用户时，为允许当前用户的应用程序正确使用和访问，将踢出之前用户配置文件中所有正在使用摄像头的应用程序。这意味着，切换用户将确保使用摄像头的应用程序停止使用摄像头。

# 总结

在本章中，我们介绍了安卓 API 中的一些变化和新增内容。安卓棉花糖主要是帮助我们这些开发人员实现更好的媒体支持，在使用音频、视频或摄像头 API 时展示我们的想法。

在下一章中，我们将介绍一些安卓特性，以了解在使用音频、视频或摄像头 API 时，安卓所做的特性增加和更改。
