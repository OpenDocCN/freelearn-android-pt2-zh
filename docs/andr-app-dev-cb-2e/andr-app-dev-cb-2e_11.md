# 第十一章：多媒体

在本章中，我们将涵盖以下主题：

+   使用 SoundPool 播放声音效果

+   使用 MediaPlayer 播放音频

+   在您的应用程序中响应用户的硬件媒体控制

+   使用默认相机应用程序拍照

+   使用（旧的）Camera API 拍照

+   使用 Camera2（新的）API 拍照

# 引言

在前几章中我们已经探讨了图形和动画，现在是我们看看 Android 中可用的声音选项的时候了。播放声音的两个最受欢迎的选项包括：

+   **SoundPool**：这适用于短声音片段

+   **MediaPlayer**：这适用于较大的声音文件（如音乐）和视频文件

我们将首先探讨使用这些库的前两个食谱。我们还会看看如何使用与声音相关的硬件，比如音量控制和媒体播放控制（耳机上常有的播放、暂停等）。

本章的其余部分将重点介绍如何使用相机，既通过 Intents 间接使用（将相机请求传递给默认相机应用程序），也直接使用相机 API。我们将探讨随 Android 5.0 Lollipop（API 21）发布的新 Camera2 API，但也会看看原始的 Camera API，因为大约 75%的市场还没有 Lollipop。（为了帮助您利用 Camera2 API 提供的新功能，我们将展示一种使用旧 Camera API 的新方法，以简化在您自己的应用程序中使用这两个 Camera API。）

# 使用 SoundPool 播放声音效果

当您的应用程序需要声音效果时，SoundPool 通常是一个很好的起点。

SoundPool 很有趣，因为它允许我们通过改变播放速率和允许同时播放多个声音来为我们的声音创建特殊效果。

支持的热门音频文件类型包括：

+   3GPP（`.3gp`）

+   3GPP（`.3gp`）

+   FLAC（`.flac`）

+   MP3（`.mp3`）

+   MIDI 类型 0 和 1（`.mid`、`.xmf`和`.mxmf`）

+   Ogg（`.ogg`）

+   WAVE（`.wav`）

请查看*支持的媒体格式*链接以获取完整列表，包括网络协议。

与 Android 中的常见做法一样，操作系统的更新带来了 API 的变化。`SoundPool`也不例外，原始的`SoundPool`构造函数在 Lollipop（API 21）中被弃用。我们不会将最小 API 设置为 21，也不会依赖可能随时停止工作的弃用代码，而是实现旧方法和新方法，并在运行时检查操作系统版本以使用适当的方法。

本食谱将演示如何使用 Android 的`SoundPool`库播放声音效果。为了演示同时播放声音，我们将创建两个按钮，每个按钮按下时都会播放声音。

## 准备就绪

在 Android Studio 中创建一个新项目，并将其命名为：`SoundPool`。使用默认的**Phone & Tablet**选项，并在提示**Activity Type**时选择**Empty Activity**。

为了演示同时播放声音，我们至少需要在项目中包含两个音频文件。我们访问了 SoundBible.com([免费版权声音](http://soundbible.com/royalty-free-sounds-5.html))，并找到了两个免费版权的公共领域声音，包含在下载项目文件中：

第一个声音是较长的播放声音：

[水声效果](http://soundbible.com/2032-Water.html)

第二个声音较短：

[金属掉落声](http://soundbible.com/1615-Metal-Drop.html)

## 如何操作...

如前所述，我们需要在项目中包含两个音频文件。准备好您的声音文件后，请按照以下步骤操作：

1.  创建一个新的 raw 文件夹（**文件** | **新建** | **Android 资源目录**），并在 **资源类型** 下拉菜单中选择 `raw`。

1.  将你的声音文件复制到 `res/raw` 作为 `sound_1` 和 `sound_2`。（保留它们的原始扩展名。）

1.  打开 `activity_main.xml` 并用以下按钮替换现有的 `TextView`：

    ```kt
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Play Sound 1"
        android:id="@+id/button1"
        android:layout_centerInParent="true"
        android:onClick="playSound1"/>
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Play Sound 2"
        android:id="@+id/button2"
        android:layout_below="@+id/button1"
        android:layout_centerHorizontal="true"
        android:onClick="playSound2"/>
    ```

1.  现在，打开 `ActivityMain.java` 并添加以下全局变量：

    ```kt
    HashMap<Integer, Integer> mHashMap= null;
    SoundPool mSoundPool;
    ```

1.  修改现有的 `onCreate()` 方法，如下所示：

    ```kt
    final Button button1=(Button)findViewById(R.id.button1);
    button1.setEnabled(false);
    final Button button2=(Button)findViewById(R.id.button2);
    button2.setEnabled(false);

    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
        createSoundPoolNew();
    }else{
        createSoundPooolOld();
    }
    mSoundPool.setOnLoadCompleteListener(new SoundPool.OnLoadCompleteListener() {
        @Override
        public void onLoadComplete(SoundPool soundPool, int sampleId, int status) {
            button1.setEnabled(true);
            button2.setEnabled(true);
        }
    });
    mHashMap = new HashMap<>();
    mHashMap.put(1, mSoundPool.load(this, R.raw.sound_1, 1));
    mHashMap.put(2, mSoundPool.load(this, R.raw.sound_2, 1));
    ```

1.  添加 `createSoundPoolNew()` 方法：

    ```kt
    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    private void createSoundPoolNew() {
        AudioAttributes audioAttributes = new AudioAttributes.Builder()
        .setUsage(AudioAttributes.USAGE_MEDIA)
        .setContentType(AudioAttributes.CONTENT_TYPE_SONIFICATION)
        .build();
        mSoundPool = new SoundPool.Builder()
                .setAudioAttributes(audioAttributes)
                .setMaxStreams(2)
                .build();
    }
    ```

1.  添加 `createSoundPooolOld()` 方法：

    ```kt
    @SuppressWarnings("deprecation")
    private void createSoundPooolOld(){
        mSoundPool = new SoundPool(2, AudioManager.STREAM_MUSIC, 0);
    }
    ```

1.  添加按钮 `onClick()` 方法：

    ```kt
    public void playSound1(View view){
        mSoundPool.play(mHashMap.get(1), 0.1f, 0.1f, 1, 0, 1.0f);
    }

    public void playSound2(View view){
        mSoundPool.play(mHashMap.get(2), 0.9f, 0.9f, 1, 1, 1.0f);
    }
    ```

1.  按如下方式重写 `onStop()` 回调：

    ```kt
    protected void onStop() {
        super.onStop();
        mSoundPool.release();
    }
    ```

1.  在设备或模拟器上运行应用程序。

## 工作原理...

首先要注意的是我们如何构建这个对象本身。正如我们在引言中提到的，SoundPool 构造函数在 Lollipop（API 21）中有所改变。旧的构造函数已弃用，推荐使用 `SoundPool.Builder()`。在像 Android 这样不断变化的环境中，API 的变化是很常见的，因此学习如何应对这些变化是个好主意。如您所见，在这个案例中，这并不困难。我们只需检查当前的操作系统版本，并调用相应的方法。值得注意的是方法注解：

```kt
@TargetApi(Build.VERSION_CODES.LOLLIPOP)
```

以及：

```kt
@SuppressWarnings("deprecation")
```

创建 SoundPool 后，我们设置了一个 `setOnLoadCompleteListener()` 监听器。启用按钮主要是为了演示 SoundPool 需要在声音资源可用之前加载它们。

使用 SoundPool 的最后一点是调用 `play()`。我们需要传入 `soundID`，这是我们在使用 `load()` 加载声音时返回的。`play()` 为我们提供了一些选项，包括声音音量（左右）、循环次数和播放速率。为了演示其灵活性，我们以较低的音量播放第一个声音（较长），以产生类似流水背景的效果。第二个声音以较高的音量播放，并且我们播放两次。

## 还有更多...

如果你只需要一个基本的声音效果，比如点击声，你可以使用 AudioManager 的 `playSoundEffect()` 方法。以下是一个示例：

```kt
AudioManager audioManager =(AudioManager) 
this.getSystemService(Context.AUDIO_SERVICE);
audioManager.playSoundEffect(SoundEffectConstants.CLICK);
```

你只能从 `SoundEffectConstants` 指定一个声音；你不能使用自己的声音文件。

## 另请参阅

+   **开发者文档：SoundPool**

    [Android 开发者网站关于 SoundPool 的参考页面](https://developer.android.com/reference/android/media/SoundPool.html)

+   **开发者文档：**

    [Android 开发者网站关于 AudioManager 的参考页面](https://developer.android.com/reference/android/media/AudioManager.html)

# 使用 `MediaPlayer` 播放音频

`MediaPlayer` 是为您的应用程序添加多媒体功能最重要的类之一。它支持以下媒体源：

+   项目资源

+   本地文件

+   外部资源（例如 URL，包括流媒体）

`MediaPlayer` 支持以下流行的音频文件：

+   3GPP (`.3gp`)

+   3GPP (`.3gp`)

+   FLAC (`.flac`)

+   MP3 (`.mp3`)

+   MIDI 类型 0 和 1 (`.mid`, `.xmf`, 和 `.mxmf`)

+   Ogg (`.ogg`)

+   WAVE (`.wav`)

以及以下流行的文件类型：

+   3GPP (`.3gp`)

+   Matroska (`.mkv`)

+   WebM (`.webm`)

+   MPEG-4 (`.mp4`, `.m4a`)

查看支持的媒体格式链接以获取完整列表，包括网络协议。

本示例将演示如何在您的应用中设置 `MediaPlayer` 以播放项目中的声音。（要全面了解 `MediaPlayer` 提供的全部功能，请查看本示例末尾的开发者文档链接。）

## 准备工作

在 Android Studio 中创建一个新项目，命名为 `MediaPlayer`。使用默认的 **Phone & Tablet** 选项，并在提示 **Activity Type** 时选择 **Empty Activity**。

我们这个示例还需要一个声音文件，将使用上一个示例中的相同长音效“水声”。

第一个声音是一个较长的音效：[水声](http://soundbible.com/2032-Water.html)

## 如何操作...

如前所述，我们需要在项目中包含一个声音文件。准备好声音文件后，请按照以下步骤操作：

1.  创建一个新的原始资源文件夹（**文件** | **新建** | **Android 资源目录**），并在 **资源类型** 下拉菜单中选择 `raw`

1.  将您的声音文件复制到 `res/raw` 目录下，命名为 `sound_1`。（保留原始扩展名。）

1.  打开 `activity_main.xml` 文件，将现有的 `TextView` 替换为以下按钮：

    ```kt
    <Button
        android:layout_width="100dp"
        android:layout_height="wrap_content"
        android:text="Play"
        android:id="@+id/buttonPlay"
        android:layout_above="@+id/buttonPause"
        android:layout_centerHorizontal="true"
        android:onClick="buttonPlay" />
    <Button
        android:layout_width="100dp"
        android:layout_height="wrap_content"
        android:text="Pause"
        android:id="@+id/buttonPause"
        android:layout_centerInParent="true"
        android:onClick="buttonPause"/>
    <Button
        android:layout_width="100dp"
        android:layout_height="wrap_content"
        android:text="Stop"
        android:id="@+id/buttonStop"
        android:layout_below="@+id/buttonPause"
        android:layout_centerHorizontal="true"
        android:onClick="buttonStop"/>
    ```

1.  现在，打开 `ActivityMain.java` 文件，并添加以下全局变量：

    ```kt
    MediaPlayer mMediaPlayer;
    ```

1.  添加 `buttonPlay()` 方法：

    ```kt
    public void buttonPlay(View view){
        if (mMediaPlayer==null) {
            mMediaPlayer = MediaPlayer.create(this, R.raw.sound_1);
            mMediaPlayer.setLooping(true);
            mMediaPlayer.start();
        } else  {
            mMediaPlayer.start();
        }
    }
    ```

1.  添加 `buttonPause()` 方法：

    ```kt
    public void buttonPause(View view){
        if (mMediaPlayer!=null && mMediaPlayer.isPlaying()) {
            mMediaPlayer.pause();
        }
    }
    ```

1.  添加 `buttonStop()` 方法：

    ```kt
    public void buttonStop(View view){
        if (mMediaPlayer!=null) {
            mMediaPlayer.stop();
            mMediaPlayer.release();
            mMediaPlayer = null;
        }
    }
    ```

1.  最后，用以下代码重写 `onStop()` 回调方法：

    ```kt
    protected void onStop() {
        super.onStop();
        if (mMediaPlayer!=null) {
            mMediaPlayer.release();
            mMediaPlayer = null;
        }
    }
    ```

1.  现在，您可以在设备或模拟器上运行应用程序了。

## 工作原理...

这里的代码非常直观。我们创建一个带有声音的 `MediaPlayer` 并开始播放声音。按钮将相应地重新播放、暂停和停止。

即使这个基本示例也说明了关于 `MediaPlayer` 的一个非常重要的概念，那就是 *状态*。如果您要严肃使用 `MediaPlayer`，请查看下面提供的链接以获取详细信息。

## 还有更多...

为了让我们的演示更容易理解，我们使用 UI 线程进行所有操作。对于这个例子，我们使用项目中包含的短音频文件，不太可能导致 UI 延迟。通常，在准备 MediaPlayer 时使用后台线程是一个好主意。为了使这个常见任务更容易，MediaPlayer 已经包含了一个名为`prepareAsync()`的异步准备方法。以下代码将创建一个`OnPreparedListener()`监听器，并使用`prepareAsync()`方法：

```kt
mMediaPlayer = new MediaPlayer();
mMediaPlayer.setOnPreparedListener(new MediaPlayer.OnPreparedListener() {
    @Override
    public void onPrepared(MediaPlayer mp) {
        mMediaPlayer.start();
    }
});
try {
    mMediaPlayer.setDataSource(*//*URI, URL or path here*//*));
} catch (IOException e) {
    e.printStackTrace();
}
mMediaPlayer.prepareAsync();
```

### 在后台播放音乐

我们的示例旨在应用程序在前台时播放音频，并在`onStop()`回调中释放 MediaPlayer 资源。如果你正在创建一个音乐播放器，并希望在其他应用程序使用时也能在后台播放音乐，该怎么办呢？在这种情况下，你需要在服务中使用 MediaPlayer，而不是 Activity。你仍然会以同样的方式使用 MediaPlayer 库；你只需要将从 UI 传递信息（如声音选择）到你的服务。

### 注意

请注意，由于服务与活动在同一个 UI 线程中运行，你仍然不希望在服务中执行可能阻塞的操作。MediaPlayer 确实处理后台线程以防止阻塞你的 UI 线程，否则，你需要自己执行线程操作。（有关线程和选项的更多信息，请参见第十四章，*让你的应用准备好上架 Play 商店*。）

### 使用硬件音量键控制你的应用的音频音量

如果你希望音量控制能控制你应用中的音量，请使用`setVolumeControlStream()`方法来指定应用程序的音频流，如下所示：

```kt
setVolumeControlStream(AudioManager.STREAM_MUSIC);
```

有关其他流选项，请参见以下`AudioManager`链接。

## 另请参阅

+   支持的媒体格式[`developer.android.com/guide/appendix/media-formats.html`](https://developer.android.com/guide/appendix/media-formats.html)

+   **开发者文档：MediaPlayer** [`developer.android.com/reference/android/media/MediaPlayer.html`](http://developer.android.com/reference/android/media/MediaPlayer.html)

+   **开发者文档：音频管理器**[`developer.android.com/reference/android/media/AudioManager.html`](https://developer.android.com/reference/android/media/AudioManager.html)

# 在你的应用中响应硬件媒体控制

让你的应用响应用户的媒体控制，如播放、暂停、跳过等，是一个用户会非常欣赏的贴心功能。

安卓通过媒体库使这成为可能。与之前*使用 SoundPool 播放声音效果*的食谱一样，Lollipop 版本改变了解决这个问题的方式。与`SoundPool`示例不同，这个食谱能够利用另一种方法——兼容性库。

本示例将展示如何设置 `MediaSession` 以响应硬件按钮，这将适用于 Lollipop 及以上版本，以及使用 `MediaSessionCompat` 库的早期 `Lollilop` 版本。（兼容性库将自动处理检查操作系统版本并使用正确的 API 调用。）

## 准备工作。

在 Android Studio 中创建一个新项目，并将其命名为 `HardwareMediaControls`。使用默认的 **Phone & Tablet** 选项，并在提示选择 **Activity Type** 时选择 **Empty Activity**。

## 如何操作...

我们将仅使用 Toast 消息来响应硬件事件，因此无需对活动布局进行任何更改。要开始，请打开 `ActivityMain.java` 并按照以下步骤操作：

1.  创建以下 `mMediaSessionCallback` 以响应媒体按钮：

    ```kt
    MediaSessionCompat.Callback mMediaSessionCallback = new MediaSessionCompat.Callback() {
        @Override
        public void onPlay() {
            super.onPlay();
            Toast.makeText(MainActivity.this, "onPlay()", Toast.LENGTH_SHORT).show();
        }
        @Override
        public void onPause() {
            super.onPause();
            Toast.makeText(MainActivity.this, "onPause()", Toast.LENGTH_SHORT).show();
        }
        @Override
        public void onSkipToNext() {
            super.onSkipToNext();
            Toast.makeText(MainActivity.this, "onSkipToNext()", Toast.LENGTH_SHORT).show();
        }
        @Override
        public void onSkipToPrevious() {
            super.onSkipToPrevious();
            Toast.makeText(MainActivity.this, "onSkipToPrevious()", Toast.LENGTH_SHORT).show();
        }
    };
    ```

1.  在现有的 `onCreate()` 回调中添加以下代码：

    ```kt
    MediaSessionCompat mediaSession = new MediaSessionCompat(this, getApplication().getPackageName());
    mediaSession.setCallback(mMediaSessionCallback);
    mediaSession.setFlags(MediaSessionCompat.FLAG_HANDLES_MEDIA_BUTTONS);
    mediaSession.setActive(true);
    PlaybackStateCompat state = new PlaybackStateCompat.Builder()
      .setActions(
        PlaybackStateCompat.ACTION_PLAY | PlaybackStateCompat.ACTION_PLAY_PAUSE | PlaybackStateCompat.ACTION_PAUSE | PlaybackStateCompat.ACTION_SKIP_TO_NEXT | PlaybackStateCompat.ACTION_SKIP_TO_PREVIOUS).build();
    mediaSession.setPlaybackState(state);
    ```

1.  在带有媒体控制功能（如耳机）的设备或模拟器上运行应用程序，以查看 Toast 消息。

## 工作原理...

设置此功能共有四个步骤：

1.  创建一个 `MediaSession.Callback` 并将其附加到 MediaSession。

1.  设置 MediaSession 标志，以表示我们希望使用媒体按钮。

1.  将 `SessionState` 设置为 `active`。

1.  使用我们将要处理的操作来设置 `PlayBackState`。

步骤 4 和步骤 1 一起工作，因为回调只会接收到在 `PlayBackState` 中设置的事件。

由于在本示例中我们实际上并未控制任何播放，只是演示如何响应硬件事件。你需要在 `PlayBackState` 中实现实际功能，并在 `setActions()` 调用后包含一个 `setState()` 的调用。

这是一个很好的示例，展示了 API 的变化如何使事情变得更容易。由于新的 `MediaSession` 和 `PlaybackState` 被整合到兼容性库中，我们可以在旧版本的操作系统上利用这些新的 API。

## 还有更多内容...

### 检查正在使用的硬件。

如果你想根据当前的输出硬件让应用有不同的响应，可以使用 `AudioManager` 来检查。以下是一个示例：

```kt
AudioManager audioManager =(AudioManager) this.getSystemService(Context.AUDIO_SERVICE);
if (audioManager.isBluetoothA2dpOn()) {
    // Adjust output for Bluetooth.
} else if (audioManager.isSpeakerphoneOn()) {
    // Adjust output for Speakerphone.
} else if (audioManager.isWiredHeadsetOn()) {
    //Only checks if a wired headset is plugged in
    //May not be the audio output
} else {
    // Regular speakers?
}
```

## 另请参阅

+   **开发者文档：MediaSession**

    [`developer.android.com/reference/android/media/session/MediaSession.html`](https://developer.android.com/reference/android/media/session/MediaSession.html)

+   **开发者文档：MediaSessionCompat**

    [`developer.android.com/reference/android/support/v4/media/session/MediaSessionCompat.html`](https://developer.android.com/reference/android/support/v4/media/session/MediaSessionCompat.html)

+   **开发者文档：PlaybackState**

    [`developer.android.com/reference/android/support/v4/media/session/PlaybackStateCompat.html`](https://developer.android.com/reference/android/support/v4/media/session/PlaybackStateCompat.html)

+   **开发者文档：PlaybackStateCompat**

    [`developer.android.com/reference/android/support/v4/media/session/PlaybackStateCompat.html`](https://developer.android.com/reference/android/support/v4/media/session/PlaybackStateCompat.html)

# 使用默认相机应用程序拍照

如果你的应用程序需要来自相机的图像，但不是相机的替代应用，那么允许“默认”相机应用拍照可能更好。这也尊重用户选择的首选相机应用程序。

当你拍照时，除非它仅适用于你的应用程序，否则最好将照片公开。 （这允许它包含在用户的照片库中。）这个方法将演示如何使用默认的照片应用程序拍照，将其保存到公共文件夹，并显示图像。

## 准备就绪

在 Android Studio 中创建一个新项目，并将其命名为：`UsingTheDefaultCameraApp`。使用默认的 **Phone & Tablet** 选项，并在提示 **Activity Type** 时选择 **Empty Activity**。

## 如何操作...

我们将创建一个带有 ImageView 和按钮的布局。按钮将创建一个 Intent 来启动默认的相机应用。当相机应用完成时，我们的应用将得到一个回调。首先打开 Android Manifest 并按照以下步骤操作：

1.  添加以下权限：

    ```kt
    <uses-permission
    android:name="android.permission.READ_EXTERNAL_STORAGE" />
    ```

1.  打开 `activity_main.xml` 文件，将现有的 `TextView` 替换为以下视图：

    ```kt
    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/imageView"
        android:src="img/ic_launcher"
        android:layout_centerInParent="true"/>

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Take Picture"
        android:id="@+id/button"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:onClick="takePicture"/>
    ```

1.  打开 `MainActivity.java` 并将以下全局变量添加到 `MainActivity` 类中：

    ```kt
    final int PHOTO_RESULT=1;
    private Uri mLastPhotoURI=null;
    ```

1.  添加以下方法来创建照片的 URI：

    ```kt
    private Uri createFileURI() {
        String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(System.currentTimeMillis());
        String fileName = "PHOTO_" + timeStamp + ".jpg";
        return Uri.fromFile(new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES),fileName));
    }
    ```

1.  添加以下方法来处理按钮点击：

    ```kt
    public void takePicture(View view) {
        Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_ CAPTURE);
        if (takePictureIntent.resolveActivity(getPackageManager()) != 
            null) {
            mLastPhotoURI = createFileURI();
            takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, mLastPhotoURI);
             startActivityForResult(takePictureIntent, PHOTO_RESULT);
        }
    }
    ```

1.  添加一个新的方法来重写 `onActivityResult()`，如下所示：

    ```kt
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == PHOTO_RESULT && resultCode == RESULT_OK ) {
            mImageView.setImageBitmap(BitmapFactory.decodeFile(mLastPhotoURI.getPath()));
        }
    }
    ```

1.  你可以准备在设备或模拟器上运行应用程序了。

## 它是如何工作的...

使用默认相机应用程序有两个部分。第一部分是设置意图来启动应用程序。我们使用 `MediaStore.ACTION_IMAGE_CAPTURE` 创建 Intent，表示我们想要一个拍照应用。我们通过检查 `resolveActivity()` 的结果来验证默认应用是否存在。只要它不是 null，我们就知道有一个应用程序可以处理这个意图。（否则，我们的应用会崩溃。）我们创建一个文件名，并将其添加到意图中：`putExtra(MediaStore.EXTRA_OUTPUT, mLastPhotoURI)`。

当我们在 `onActivityResult()` 中得到回调时，我们首先确保它是 `PHOTO_RESULT` 和 `RESULT_OK`（用户可能已取消），然后在 `ImageView` 中加载照片。

## 还有更多...

如果你不在意图片存储在哪里，可以在不使用 `MediaStore.EXTRA_OUTPUT` 额外参数的情况下调用意图。如果你没有指定输出文件，`onActivityResult()` 将在 data Intent 中包含图像的缩略图。以下是如何显示缩略图的方法：

```kt
if (data != null) {
    imageView
.setImageBitmap((Bitmap) data.getExtras().get("data"));
}
```

以下是使用 `data Intent` 返回的 URI 加载全分辨率图像的代码：

```kt
if (data != null) {
    try {
        imageView.setImageBitmap(
            MediaStore.Images.Media. getBitmap(getContentResolver(),
            Uri.parse(data.toUri(Intent.URI_ALLOW_UNSAFE))));
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

### 调用默认视频应用

如果你想要调用默认的视频捕捉应用程序，过程是相同的。只需在步骤 5 中更改意图，如下所示：

```kt
Intent takeVideoIntent = new Intent(MediaStore.ACTION_VIDEO_CAPTURE);
```

你可以在 `onActivityResult()` 中获取到视频的 URI，如下所示：

```kt
Uri videoUri = intent.getData();
```

## 另请参阅

+   第九章《图形和动画》中的*缩小大图像以避免内存溢出异常*食谱。

# 使用（旧的）Camera API 拍照

之前的食谱演示了如何使用意图调用默认照片应用程序。如果你只需要快速拍照，意图可能是理想的解决方案。如果不是，并且你需要更多控制相机，这个食谱将向你展示如何直接使用 Camera API。

实际上有两个使用 Camera API 的食谱——一个是针对在 Android 1.0（API 1）中发布的原始 Camera API，另一个是 Camera2 API，在 Android 5.0（API 21）中发布。我们将介绍新旧 API。理想情况下，你会希望根据可用的最新和最伟大的 API 编写应用程序，但在撰写本文时，Android 5.0（API 21）的市场份额只有大约 23%。如果你只使用 Camera2 API，你会排除超过 75%的市场。

编写你的应用程序以使用 Camera2 API 利用新功能，但对于其余用户仍然可以使用原来的 Camera API 实现功能性的应用。为了帮助同时使用两者，本食谱将利用 Android 的新功能，特别是从 Android 4.0（API 14）引入的`TextureView`。我们将使用`TextureView`代替更传统的`SurfaceView`来显示相机预览。这将允许你使用与新 Camera2 API 相同的布局，因为它也使用`TextureView`。（将最低 API 设置为 Android 4.0（API 14）及以上，其市场份额超过 96%，对你的用户群限制不大。）

## 准备就绪

在 Android Studio 中创建一个新项目，并将其命名为`CameraAPI`。在**目标 Android 设备**对话框中，选择**手机 & 平板电脑**选项，并为**最低 SDK**选择 API 14（或更高）。当提示选择**活动类型**时，选择**空活动**。

## 如何操作...

首先，打开 Android 清单文件并按照以下步骤操作：

1.  添加以下两个权限：

    ```kt
    <uses-permission android:name="android.permission.CAMERA"/>
    <uses-permission
    android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ```

1.  现在打开`activity_main.xml`文件，并用以下视图替换现有的 TextView：

    ```kt
    <TextureView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/textureView"
        android:layout_alignParentTop="true"
        android:layout_centerHorizontal="true" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Take Picture"
        android:id="@+id/button"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:onClick="takePicture"/>
    ```

1.  打开`MainActivity.java`文件，并将`MainActivity`类声明修改为实现`SurfaceTextureListener`，如下所示：

    ```kt
    public class MainActivity extends AppCompatActivity
            implements TextureView.SurfaceTextureListener {
    ```

1.  向`MainActivity`添加以下全局声明：

    ```kt
    @Deprecated
    private Camera mCamera;
    private TextureView mTextureView;
    ```

1.  创建以下`PictureCallback`以处理保存照片：

    ```kt
    Camera.PictureCallback pictureCallback = new Camera.PictureCallback() {
        @Override
        public void onPictureTaken(byte[] data, Camera camera) {
            try {
                String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(System.currentTimeMillis());
                String fileName = "PHOTO_" + timeStamp + ".jpg";
                File pictureFile = new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES),fileName);

                FileOutputStream fileOutputStream =new FileOutputStream(pictureFile.getPath());
                fileOutputStream.write(data);
                fileOutputStream.close();
                Toast.makeText(MainActivity.this, "Picture Taken", Toast.LENGTH_SHORT).show();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    };
    ```

1.  向现有的`onCreate()`回调添加以下代码：

    ```kt
    mTextureView = (TextureView)findViewById(R.id.textureView);
    mTextureView.setSurfaceTextureListener(this);
    ```

1.  添加以下方法来实现`SurfaceTextureListener`接口：

    ```kt
    public void onSurfaceTextureAvailable(SurfaceTexture surface, int width, int height) {
        mCamera = Camera.open();
        if (mCamera!=null) {
            try {
                mCamera.setPreviewTexture(surface);
                mCamera.startPreview();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    public boolean onSurfaceTextureDestroyed(SurfaceTexture surface) {
        if (mCamera!=null) {
            mCamera.stopPreview();
            mCamera.release();
        }
        return true;
    }
    public void onSurfaceTextureSizeChanged(SurfaceTexture surface, int width, int height) {
        // Unused
    }
    public void onSurfaceTextureUpdated(SurfaceTexture surface) {
        // Unused
    }
    ```

1.  添加以下方法来处理按钮点击：

    ```kt
    public void takePicture(View view) {
        if (mCamera!=null) {
            mCamera.takePicture(null, null, pictureCallback);
        }
    }
    ```

1.  在带有相机的设备或模拟器上运行应用程序。

## 工作原理...

首先要注意的是，在 Android Studio 中查看这段代码时，你会看到很多带有以下警告的删除线代码：

```kt
'android.hardware.Camera' is deprecated
```

如引言所述，`android.hardware.camera2` API 在 Android 5.0（API 19）中引入，并替代了`android.hardware.camera` API。

### 提示

你可以添加以下注解来抑制弃用警告：

```kt
@SuppressWarnings("deprecation")
```

使用 Camera API 时有两个主要步骤：

+   设置预览

+   捕获图像

我们从布局中获取`TextureView`，然后使用以下代码将我们的活动（实现了`SurfaceTextureListener`）作为监听器：

```kt
mTextureView.setSurfaceTextureListener(this);
```

当`TextureView`的表面准备就绪时，我们会收到`onSurfaceTextureAvailable`回调，在那里我们使用以下代码设置预览表面：

```kt
mCamera.setPreviewTexture(surface);
mCamera.startPreview();
```

下一步是在按下按钮时拍照。我们使用以下代码实现：

```kt
mCamera.takePicture(null, null, pictureCallback);
```

当图片准备好时，我们在创建的`Camera.PictureCallback`类中收到`onPictureTaken()`回调。

## 还有更多...

请记住，这段代码是为了展示其工作原理，并非用于创建完整的商业应用程序。正如大多数开发者所知，编码中的真正挑战在于处理所有的问题场景。改进的一些方面包括增加切换摄像头的功能，因为当前应用使用的是默认摄像头。同时，也要查看设备在预览和保存图片时的方向。更复杂的应用程序会在后台线程处理一些工作，以避免 UI 线程的延迟。（查看下一个食谱，了解我们如何在后台线程上处理一些摄像头处理工作。）

### 设置摄像头参数

Camera API 包括参数，使我们能够调整摄像头设置。通过这个例子，我们可以更改预览的大小：

```kt
Camera.Parameters parameters = mCamera.getParameters();
parameters.setPreviewSize(mPreviewSize.width, 
mPreviewSize.height);
mCamera.setParameters(parameters);
```

请记住，硬件也必须支持我们想要的设置。在这个例子中，我们首先需要查询硬件以获取所有可用的预览模式，然后设置符合我们要求的模式。（在下一个食谱中设置图片分辨率时，可以看到一个这样的例子。）请参阅 Camera 文档链接中的`getParameters()`。

## 另请参阅

+   下一个食谱：*使用 Camera2（新）API 拍照*

+   在第八章，*使用触摸屏和传感器*中的*读取设备方向*食谱，了解检测当前设备方向的示例

+   **开发者文档：构建摄像头应用程序**位于：[`developer.android.com/guide/topics/media/camera.html#custom-camera`](https://developer.android.com/guide/topics/media/camera.html#custom-camera)

+   [`developer.android.com/reference/android/hardware/Camera.html`](https://developer.android.com/reference/android/hardware/Camera.html)

# 使用 Camera2（新）API 拍照

现在我们已经了解了旧的 Camera API，是时候学习新的 Camera2 API 了。不幸的是，由于 API 的异步性质，它有点复杂。幸运的是，总体概念与之前的 Camera API 相同。

## 准备就绪

在 Android Studio 中创建一个新项目，命名为`Camera2API`。在**Target Android Devices**对话框中，选择**Phone & Tablet**选项，并将**Minimum SDK**设置为 API 21（或更高）。当提示选择**Activity Type**时，选择**Empty Activity**。

## 如何操作...

你会看到，这个配方有很多代码。首先打开 Android Manifest 文件，并按照以下步骤操作：

1.  添加以下两个权限：

    ```kt
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ```

1.  现在，打开`activity_main.xml`文件，用以下视图替换现有的 TextView：

    ```kt
    <TextureView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/textureView"
        android:layout_alignParentTop="true"
        android:layout_centerHorizontal="true" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Take Picture"
        android:id="@+id/button"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:onClick="takePicture"/>
    ```

1.  现在，打开`MainActivity.java`文件，并在`MainActivity`类中添加以下全局变量：

    ```kt
    private CameraDevice mCameraDevice = null;
    private CaptureRequest.Builder mCaptureRequestBuilder = null;
    private CameraCaptureSession mCameraCaptureSession  = null;
    private TextureView mTextureView = null;
    private Size mPreviewSize = null;
    ```

1.  添加以下`Comparator`类：

    ```kt
    static class CompareSizesByArea implements Comparator<Size> {
        @Override
        public int compare(Size lhs, Size rhs) {
            return Long.signum((long) lhs.getWidth() * lhs.getHeight() - (long) rhs.getWidth() * rhs.getHeight());
        }
    }
    ```

1.  添加以下`CameraDevice.StateCallback`：

    ```kt
    private CameraDevice.StateCallback mStateCallback = new CameraDevice.StateCallback() {
        @Override
        public void onOpened(CameraDevice camera) {
            mCameraDevice = camera;
            SurfaceTexture texture = mTextureView.getSurfaceTexture();
            if (texture == null) {
                return;
            }
            texture.setDefaultBufferSize(mPreviewSize.getWidth(), mPreviewSize.getHeight());
            Surface surface = new Surface(texture);
            try {
                mCaptureRequestBuilder = mCameraDevice.createCaptureRequest(CameraDevice.TEMPLATE_PREVIEW);
            } catch (CameraAccessException e){
                e.printStackTrace();
            }
            mCaptureRequestBuilder.addTarget(surface);
            try {
                mCameraDevice.createCaptureSession(Arrays.asList(surface), mPreviewStateCallback, null);
            } catch (CameraAccessException e) {
                e.printStackTrace();
            }
        }
        @Override
        public void onError(CameraDevice camera, int error) {}
        @Override
        public void onDisconnected(CameraDevice camera) {}
    };
    ```

1.  添加以下`SurfaceTextureListener`：

    ```kt
    private TextureView.SurfaceTextureListener mSurfaceTextureListener =     new TextureView.SurfaceTextureListener() {
        @Override
        public void onSurfaceTextureUpdated(SurfaceTexture surface) {}
        @Override
        public void onSurfaceTextureSizeChanged(SurfaceTexture surface, int width, int height) {}
        @Override
        public boolean onSurfaceTextureDestroyed(SurfaceTexture surface) {
                return false;
        }
        @Override
        public void onSurfaceTextureAvailable(SurfaceTexture surface, int width, int height) {
                openCamera();
        }
    };
    ```

1.  添加以下`CameraCaptureSession.StateCallback`：

    ```kt
    private CameraCaptureSession.StateCallback mPreviewStateCallback = new CameraCaptureSession.StateCallback() {
        @Override
        public void onConfigured(CameraCaptureSession session) {
            startPreview(session);
        }

        @Override
        public void onConfigureFailed(CameraCaptureSession session) {}
    };
    ```

1.  在现有的`onCreate()`回调中添加以下代码：

    ```kt
    mTextureView = (TextureView) findViewById(R.id.textureView);
    mTextureView.setSurfaceTextureListener(mSurfaceTextureListener);
    ```

1.  添加以下方法以覆盖`onPause()`和`onResume()`：

    ```kt
    @Override
    protected void onPause() {
        super.onPause();
        if (mCameraDevice != null) {
            mCameraDevice.close();
            mCameraDevice = null;
        }
    }
    @Override
    public void onResume() {
        super.onResume();
        if (mTextureView.isAvailable()) {
            openCamera();
        } else {
            mTextureView.setSurfaceTextureListener(mSurfaceTextureListener);
        }
    }
    ```

1.  添加`openCamera()`方法：

    ```kt
    private void openCamera() {
        CameraManager manager = (CameraManager) getSystemService(CAMERA_SERVICE);
        try{
            String cameraId = manager.getCameraIdList()[0];
            CameraCharacteristics characteristics = manager.getCameraCharacteristics(cameraId);
            StreamConfigurationMap map = characteristics.get(CameraCharacteristics.SCALER_STREAM_CONFIGURATION_MAP); 
            mPreviewSize = map.getOutputSizes(SurfaceTexture.class) [0];
            manager.openCamera(cameraId, mStateCallback, null);
        } catch(CameraAccessException e) {
            e.printStackTrace();
        } catch (SecurityException e) {
            e.printStackTrace();
        }
    }
    ```

1.  添加`startPreview()`方法：

    ```kt
    private void startPreview(CameraCaptureSession session) { 
        mCameraCaptureSession = session; 
        mCaptureRequestBuilder.set(CaptureRequest.CONTROL_MODE,CameraMetadata.CONTROL_MODE_AUTO); 
        HandlerThread backgroundThread = new HandlerThread("CameraPreview"); 
        backgroundThread.start();
        Handler backgroundHandler = new Handler(backgroundThread. getLooper());
        try {
            mCameraCaptureSession.setRepeatingRequest(mCaptureRequestBuilder.build(), null, backgroundHandler);
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }
    }
    ```

1.  添加`getPictureFile()`方法：

    ```kt
    private File getPictureFile() {
        String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss"). format(System.currentTimeMillis());
        String fileName = "PHOTO_" + timeStamp + ".jpg";
        return new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES),fileName);
    }
    ```

1.  添加`takePicture()`方法，该方法保存图像文件：

    ```kt
    protected void takePicture(View view) {
        if (null == mCameraDevice) {
            return;
        }
        CameraManager manager = (CameraManager)
        getSystemService(Context.CAMERA_SERVICE);
        try {
            CameraCharacteristics characteristics = manager.getCameraCharacteristics(mCameraDevice.getId());
            StreamConfigurationMap configurationMap = characteristics.get(CameraCharacteristics.SCALER_STREAM_CONFIGURATION_MAP);
            if (configurationMap == null) return;
            Size largest = Collections.max(
                Arrays.asList(configurationMap.getOutputSizes(ImageFormat.JPEG)),
                new CompareSizesByArea());
            ImageReader reader = ImageReader.newInstance(largest.getWidth(), largest.getHeight(), ImageFormat.JPEG, 1);
            List < Surface > outputSurfaces = new ArrayList < Surface > (2);
            outputSurfaces.add(reader.getSurface());
            outputSurfaces.add(new Surface(mTextureView.getSurfaceTexture()));
            final CaptureRequest.Builder captureBuilder = mCameraDevice.createCaptureRequest(CameraDevice.TEMPLATE_STILL_ CAPTURE);
            captureBuilder.addTarget(reader.getSurface());
            captureBuilder.set(CaptureRequest.CONTROL_MODE,
                CameraMetadata.CONTROL_MODE_AUTO);
            ImageReader.OnImageAvailableListener readerListener = new ImageReader.OnImageAvailableListener() {
                @Override
                public void onImageAvailable(ImageReader reader) {
                    Image image = null;
                    try {
                        image = reader.acquireLatestImage();
                        ByteBuffer buffer = image.getPlanes()[0].getBuffer();
                        byte[] bytes = new byte[buffer.capacity()];
                        buffer.get(bytes);
                        OutputStream output = new FileOutputStream( get PictureFile());
                        output.write(bytes);
                        output.close();
                    } catch (FileNotFoundException e) {
                        e.printStackTrace();
                    } catch (IOException e) {
                        e.printStackTrace();
                    } finally {
                        if (image != null) {
                            image.close();
                        }
                    }
                }
            };
            HandlerThread thread = new HandlerThread("CameraPicture");
            thread.start();
            final Handler backgroudHandler = new Handler(thread.getLooper());
            reader.setOnImageAvailableListener(readerListener, backgroudHandler);
            final CameraCaptureSession.CaptureCallback captureCallback = new CameraCaptureSession.CaptureCallback() {
                @Override
                public void onCaptureCompleted(
                CameraCaptureSession session, CaptureRequest request, TotalCaptureResult result) {
                        super.onCaptureCompleted(session, request, result);
                        Toast.makeText(MainActivity.this, "Picture Saved", Toast.LENGTH_SHORT).show();
                        startPreview(session);
                }
            };
            mCameraDevice.createCaptureSession(outputSurfaces, new CameraCaptureSession.StateCallback() {
                @Override
                public vod onConfigured(CameraCaptureSession session) {
                    try {
                        session.capture(captureBuilder.build(), captureCallback, backgroudHandler);
                    } catch (CameraAccessException e) {
                        e.printStackTrace();
                    }
                }
                @Override
                public void onConfigureFailed(CameraCaptureSession session) { }
            }, backgroudHandler);
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }
    }
    ```

1.  在带有摄像头的设备或模拟器上运行应用程序。

## 工作原理...

由于我们在上一个配方中了解了 TextureView，我们可以跳转到新的 Camera2 API 信息。

尽管涉及更多类，但与旧的 Camera API 一样，有两个基本步骤：

+   设置预览

+   捕获图像

### 设置预览

下面是代码如何设置预览的概要：

1.  首先，我们在`onCreate()`中使用`setSurfaceTextureListener()`方法设置`TextureView.SurfaceTextureListener`。

1.  当我们收到`onSurfaceTextureAvailable()`回调时，我们打开相机。

1.  我们将我们的`CameraDevice.StateCallback`类传递给`openCamera()`方法，该方法最终调用`onOpened()`回调。

1.  `onOpened()`通过调用`getSurfaceTexture()`获取预览的表面，并通过调用`createCaptureSession()`将其传递给 CameraDevice。

1.  最后，当调用`CameraCaptureSession.StateCallback onConfigured()`时，我们使用`setRepeatingRequest()`方法开始预览。

### 捕获图像

尽管`takePicture()`方法看起来可能是程序性的，但捕获图像也涉及到几个类，并且依赖于回调。以下是代码如何拍照的分解说明：

1.  用户点击**拍照**按钮。

1.  然后查询相机以找到最大的可用图像尺寸。

1.  然后，创建一个`ImageReader`。

1.  然后，设置`OnImageAvailableListener`，并在`onImageAvailable()`回调中保存图像。

1.  然后，创建`CaptureRequest.Builder`并包含`ImageReader`表面。

1.  接下来，创建`CameraCaptureSession.CaptureCallback`，它定义了`onCaptureCompleted()`回调。当捕获完成时，它会重新启动预览。

1.  然后，调用`createCaptureSession()`方法，创建一个`CameraCaptureSession.StateCallback`。在这里，会调用`capture()`方法，并传入之前创建的`CameraCaptureSession.CaptureCallback`。

## 还有更多...

与之前的 Camera 示例一样，我们刚刚创建了基础代码以展示一个工作的摄像头应用程序。同样，还有改进的空间。首先，你应该处理设备方向，既要考虑预览，也要在保存图片时考虑。（请参阅上一个食谱中的链接。）另外，随着 Android 6.0（API 23）的推出，现在是一个很好的时机来开始使用新的权限模型。我们不应该只在`openCamera()`方法中检查异常，而应该检查所需的权限。

## 另请参阅

+   上一个食谱：*使用（旧的）Camera API 拍照*

+   *新的 Android 6.0 运行时权限模型*，在第十四章*，让你的应用准备好上架 Play 商店*中介绍。

+   开发者文档：Camera2 API

+   [`developer.android.com/reference/android/hardware/camera2/package-summary.html`](https://developer.android.com/reference/android/hardware/camera2/package-summary.html)
