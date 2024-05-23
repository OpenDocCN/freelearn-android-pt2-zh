# 第十章：位置服务

在本章中，我们将学习如何使用 Google 的地图片段向我们的应用程序添加地图视图。我们将在地图上添加标记，用于指出感兴趣的位置。

为了做到这一点，我们还将讨论如何在 Google 开发者控制台创建项目，并设置我们的应用程序以使用 Google Play Services SDK，这是在任何 Android 应用程序中使用 Google 服务的必要条件。

Parse 中的每个工作机会都有一个位置字段；基于此，我们将在地图上显示标记。

+   配置项目

    +   获取 Google Maps API 密钥

    +   配置`AndroidManifest.xml`

+   添加地图

    +   为 ViewPager 创建片段

    +   实现地图片段

+   添加标记

    +   从 Parse 检索数据

    +   为每个位置显示一个标记

+   添加标题

# 配置项目

为了使我们能够使用 Google Play Service API，我们需要使用 Google Play Services SDK 配置我们的项目。如果你还没有安装，请进入 Android SDK 管理器并获取 Google Play Service SDK。

既然我们的应用使用了 Google Play 服务，为了测试应用，你必须确保在以下设备之一上运行应用：

1.  带有 Google Play 商店的 Android 2.3 或更高版本的 Android 设备（推荐）。

1.  已设置 Google Play 服务的模拟器。如果你使用 Genymotion，**Google Play 服务**默认不会安装：![配置项目](img/4887_10_01.jpg)

我们需要让 Google Play 服务 API 对应用可用。

打开应用的`build.gradle`文件，并在依赖项下添加`play-services`库。添加到`build.gradle`文件的行应该类似于这样：

```kt
compile 'com.google.android.gms:play-services:7.8.0'
```

确保你将其更改为最新版本的`play-services`，并在发布新版本时更新。

保存文件并点击**与 Gradle 文件同步项目**。

## 获取 API 密钥

为了使用 Google Maps API，我们需要在 Google 开发者控制台注册我们的项目，并获得一个 API 密钥，然后将其添加到我们的应用中。

首先，我们需要获取我们唯一应用程序的 SHA-1 指纹。我们可以从**调试证书**或**发布证书**中获得。

+   **调试** **证书**在完成调试构建时会自动创建。此证书仅应用于目前正在测试的应用。不要使用调试证书发布应用程序。

+   **发布** **证书**是在完成发布构建时生成的。也可以使用**keytool**程序创建证书。当应用准备发布到 Play 商店时，必须使用此证书。

### 显示调试证书指纹

+   找到名为`debug.keystore`的调试密钥库文件。这个文件通常位于 Android 虚拟设备文件所在的目录中：

    +   **OS X 和 Linux**：`~/.android/`

    +   **Windows Vista 和 Windows 7**：`C:\Users\your_user_name\.android\`

+   要显示 SHA-1 指纹，请打开终端或命令提示符窗口，并输入以下内容：

    +   **OS X 和 Linux**：我们使用命令 `keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android`。

    +   **Windows Vista 和 Windows 7**：我们使用命令 `keytool -list -v -keystore "%USERPROFILE%\.android\debug.keystore" -alias androiddebugkey -storepass android -keypass android`。

输入命令并按下*回车*键后，您将看到类似这样的输出：

```kt
Alias name: androiddebugkey
Creation date: Dec 16, 2014
Entry type: PrivateKeyEntry
Certificate chain length: 1
Certificate[1]:
Owner: CN=Android Debug, O=Android, C=US
Issuer: CN=Android Debug, O=Android, C=US
Serial number: 32f30c87
Valid from: Tue Dec 16 11:35:40 CAT 2014 until: Thu Dec 08 11:35:40 CAT 2044
Certificate fingerprints:
         MD5:  7E:06:3D:45:D7:1D:48:FE:96:88:18:20:0F:09:B8:2A
         SHA1: BD:24:B2:7C:DA:67:E5:80:78:1D:75:8C:C6:66:B3:D0:63:3E:EE:84
         SHA256: E4:8C:BD:4A:24:CD:55:5C:0E:7A:1E:B7:FC:A3:9E:60:28:FB:F7:20:C6:C0:E9:1A:C8:13:29:A6:F2:10:42:DB
         Signature algorithm name: SHA256withRSA
         Version: 3
```

### 创建 Google 开发者控制台项目

访问[`console.developers.google.com/project`](https://console.developers.google.com/project)，如果您还没有账户，请创建一个账户。首先，使用您想要的名字创建一个新项目。项目创建后，执行以下步骤：

1.  在左侧边栏中，点击**APIs & auth**，然后选择**APIs**选项：![创建 Google 开发者控制台项目](img/4887_10_02.jpg)

1.  选择**Google Maps Android API**并启用它。

1.  打开**凭据**，然后点击**[创建新密钥]**。

1.  选择**Android 密钥**，输入您的**SHA-1**指纹，然后是您的项目包名称，两者之间用分号分隔，如下所示：

    ```kt
    BD:24:B2:7C:DA:67:E5:80:78:1D:75:8C:C6:66:B3:D0:63:3E:EE:84;com.packtpub.masteringandroidapp
    ```

1.  完成此操作后，您将能够像以下截图一样查看凭据：![创建 Google 开发者控制台项目](img/4887_10_03.jpg)

## 配置 AndroidManifest.xml

既然我们已经获得了 Android 应用的 API 密钥，我们需要将其添加到`AndroidManifest.xml`文件中。

打开您的`AndroidManifest.xml`文件，并在`<application>`元素下添加以下代码作为子元素：

```kt
<meta-data
  android:name="com.google.android.geo.API_KEY"
  android:value="API_KEY"/>
```

将`value`属性中的`API_KEY`替换为 Google 开发者控制台给出的 API 密钥。

我们还需要在`AndroidManifest`中添加其他几个设置。如下设置 Google Play 服务的版本：

```kt
<meta-data
  android:name="com.google.android.gms.version"
  android:value="@integer/google_play_services_version" />
```

如下设置必要的权限：

+   `INTERNET`：此权限用于从 Google Maps 服务器下载地图数据。

+   `ACCESS_NETWORK_STATE`：这将允许 API 检查连接状态，以确定是否能够下载数据。

+   `WRITE_EXTERNAL_STORAGE`：这将允许 API 缓存地图数据。

+   `ACCESS_COARSE_LOCATION`：这允许 API 使用 Wi-Fi 或移动数据获取设备的位置。

+   `ACCESS_FINE_LOCATION`：这将比`ACCESS_COARSE_LOCATION`提供更精确的位置，并且还将使用 GPS 以及 Wi-Fi 或移动数据。看看以下代码：

    ```kt
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    ```

您还需要设置 OpenGL ES。地图 API 使用 OpenGL ES 来渲染地图，因此需要安装它才能显示地图。为了通知其他服务这一需求，并防止不支持 OpenGL 的设备在 Google Play 商店显示您的应用，请在您的`AndroidManifest.xml`文件中的`<manifest>`标签下添加以下内容：

```kt
<uses-feature
  android:glEsVersion="0x00020000"
  android:required="true"/>
```

您当前的`AndroidManifest.xml`文件应类似于以下代码：

```kt
<?xml version="1.0" encoding="UTF-8"?>
<manifest  package="com.packtpub.masteringandroidapp">
  <uses-feature android:glEsVersion="0x00020000" android:required="true" />
  <uses-permission android:name="android.permission.INTERNET" />
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  <application android:name=".MAApplication" android:allowBackup="true" android:icon="@drawable/ic_launcher" android:label="@string/app_name" android:theme="@style/AppTheme">
    <activity android:name=".SplashActivity" android:label="@string/app_name">
      <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
      </intent-filter>
    </activity>
    <activity android:name=".MainActivity" android:label="@string/title_activity_main" />
    <activity android:name=".OfferDetailActivity" android:label="@string/title_activity_offer_detail" />
    <provider android:name=".MAAProvider" android:authorities="com.packtpub.masteringandroidapp.MAAProvider" />
    <meta-data android:name="com.google.android.geo.API_KEY" android:value="AIzaSyC9o7cLdk_MIX_aQhaOLvoqYywK61bN0PQ" />
    <meta-data android:name="com.google.android.gms.version" android:value="@integer/google_play_services_version" />
  </application>
</manifest>
```

# 添加地图

既然我们的应用程序已经配置好了可以使用地图服务，那么我们可以开始讨论如何在应用程序中添加一个可视化的地图。对于地图，我们将创建另一个 Fragment，它将被加载到`ViewPager`的第二个页面。

有两种方法可以显示谷歌地图；一个`MapFragment`或`MapView`对象可以表示它。

## 添加 Fragment

在我们的`fragments`目录中创建一个名为`MyMapFragment`的新 Java 类。这个类应该继承`Fragment`类型。然后，重写`OnCreateView`方法，并让它返回`fragment_my_map`的填充视图：

```kt
package com.packtpub.masteringandroidapp.fragments;

import …

/**
* Created by Unathi on 7/29/2015.
*/
public class MyMapFragment extends Fragment {

  @Nullable
  @Override
  public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fragment_my_map, container, false);

    return view;
  }
}
```

接下来，为 Fragment 创建布局文件，并将其命名为`fragment_my_map`。将布局的根元素设置为`FrameLayout`。我们将在布局中暂时添加`TextView`以验证其是否有效。`fragment_my_map.xml`文件的代码应类似于以下这样：

```kt
<?xml version="1.0" encoding="UTF-8"?>
<FrameLayout  android:layout_width="match_parent" android:layout_height="match_parent">
  <TextView android:layout_width="wrap_content" android:layout_height="wrap_content" android:text="This is a TextView" android:layout_gravity="center" android:textSize="25dp" />
</FrameLayout>
```

将我们的 Fragment 添加到应用程序的最后一步将是编辑`MyPagerAdapter.java`文件，将其作为第二个页面显示。为此，将`getItem`方法中的第二个 case 更改为返回`MyMapFragment`的实例，并将`getPageTitle`方法中第二个 case 的页面标题更改为返回`MAP`：

```kt
@Override
public Fragment getItem(int i) {
  switch (i) {
    case 0 :
    return new ListFragment();
    case 1 :
    return new MyMapFragment();
    case 2 :
    return new SettingsFragment();
    default:
    return null;
  }
}

@Override
public CharSequence getPageTitle(int position) {
  switch (position) {
    case 0 :
    return "LIST";
    case 1 :
    return "MAP";
    case 2 :
    return "SETTINGS";
    default:
    return null;
  }
}
```

现在，当你运行应用程序时，`ViewPager`的第二个页面应该被我们新的 Fragment 替换。

![添加 Fragment](img/4887_10_04.jpg)

## 实现 MapFragment

我们现在将使用`MapFragment`在我们的应用程序上显示地图。我们可以通过添加一个带有`android:name`为`com.google.android.gms.maps.MapFragment`的`<fragment>`布局来实现这一点。这样做将自动将`MapFragment`添加到`activity`中：

以下是`fragment_my_map.xml`的代码：

```kt
<?xml version="1.0" encoding="UTF-8"?>
<FrameLayout  android:layout_width="match_parent" android:layout_height="match_parent">
  <fragment android:name="com.google.android.gms.maps.MapFragment" android:id="@+id/map" android:layout_width="match_parent" android:layout_height="match_parent" />
</FrameLayout>
```

接下来，为了能够处理我们添加到布局中的`MapFragment`，我们需要使用`FragmentManager`，我们从`getChildFragmentManager`获取它，通过`findFragmentById`来找到。这将在`OnCreateView`方法中完成：

```kt
FragmentManager fm = getChildFragmentManager();
mapFragment = (SupportMapFragment) fm.findFragmentById(R.id.map);
if (mapFragment == null) {
  mapFragment = SupportMapFragment.newInstance();
  fm.beginTransaction().add(R.id.map, mapFragment).commit();
}
```

我们将把我们的 Fragment 分配给`SupportMapFragment`，而不是仅仅`MapFragment`，这样应用程序就可以支持低于**12**的 Android API 级别。使用以下代码：

以下是`MyMapFragment.java`的代码：

```kt
public class MyMapFragment extends Fragment{

  private SupportMapFragment mapFragment;

  @Nullable
  @Override
  public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fragment_my_map, container, false);

    FragmentManager fm = getChildFragmentManager();
    mapFragment = (SupportMapFragment) fm.findFragmentById(R.id.map);
    if (mapFragment == null) {
      mapFragment = SupportMapFragment.newInstance();
      fm.beginTransaction().add(R.id.map, mapFragment).commit();
    }

    return view;
  }

}
```

现在，当我们运行应用程序时，地图将在屏幕上显示。

![实现 MapFragment](img/4887_10_05.jpg)

# 添加标记

现在谷歌地图已经可见，但它还没有为用户显示任何有用的数据。为了实现这一点，我们将添加**地图标记**来指示用户感兴趣的点。这些将是不同职位提供的位置，我们将从 Parse 数据库下载这些位置。

我们还将学习如何将用于在地图上标记点的图标更改为自定义图像，并在标记上加上标题。这将使我们的应用程序看起来更有趣、更具有信息性。

## 从 Parse 检索数据

在我们能够显示所有标记之前，我们需要从 Parse 下载所有必要的数据。

在 `MyMapFragment.java` 中，我们将使用 `ParseQuery` 获取位置列表，并使用它来获取每个职位在显示之前的相关信息。执行以下步骤：

+   创建一个名为 `googleMap` 的私有成员变量，其类型为 `GoogleMap`，并重写 `onResume()` 方法。

+   在 `onResume()` 中，检查 `googleMap` 是否为空；如果是，这意味着我们还没有向当前地图实例添加标记。如果 `googleMap` 为空，从我们已经创建的 `MapFragment` 分配地图。这是使用 `getMap()` 实现的：

    ```kt
    if (googleMap == null) {

      googleMap = MapFragment.getMap();

    }
    ```

+   创建一个 `ParseQuery`，它将检索我们 Parse 数据库中 `JobOffer` 表的所有数据。使用 `findInBackground()` 函数和 `FindCallback`，这样我们就可以在数据下载完成后开始处理数据。使用以下代码：

    ```kt
    ParseQuery<JobOffer> query = ParseQuery.getQuery("JobOffer");
    query.findInBackground(new FindCallback<JobOffer>() {
      @Override
      public void done(List<JobOffer> list, ParseException e) {

      }
    });
    ```

## 为每个位置显示标记

现在，我们将遍历从 Parse 收到的职位列表，并使用 `addMarker()` 向 `googleMap` 添加标记。执行以下步骤：

1.  当 `findInBackground` 执行完毕后，创建一个 `ParseGeoPoint` 变量和一个循环，该循环将遍历列表中的每个项目。我们将使用 `ParseGeoPoint` 变量来存储来自 Parse 数据库的坐标：

    ```kt
    ParseGeoPoint geoPoint = new ParseGeoPoint();

    for(int i =0;i<list.size();i++){

    }
    ```

1.  在循环内，从列表中获取 `GeoPoint` 数据并将其保存到我们的 `ParseGeoPoint` 变量中：

    ```kt
     geoPoint = list.get(i).getParseGeoPoint("coordinates");
    ```

1.  最后，在每次迭代中使用 `addMarker()` 向 `googleMap` 添加标记：

    ```kt
    googleMap.addMarker(new MarkerOptions()
    .position(new LatLng(geoPoint.getLatitude(), geoPoint.getLongitude())));
    ```

你的 `MyMapFragment.java` 文件应类似于以下内容： 

```kt
public class MyMapFragment extends Fragment{

  private SupportMapFragment mapFragment;
  private GoogleMap googleMap;

  @Nullable
  @Override
  public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fragment_my_map, container, false);

    FragmentManager fm = getChildFragmentManager();
    mapFragment = (SupportMapFragment) fm.findFragmentById(R.id.map);
    if (mapFragment == null) {
      mapFragment = SupportMapFragment.newInstance();
      fm.beginTransaction().add(R.id.map, mapFragment).commit();
    }

    return view;
  }

  @Override
  public void onResume() {
    super.onResume();

    if (googleMap == null) {
      googleMap = mapFragment.getMap();

      ParseQuery<JobOffer> query = ParseQuery.getQuery("JobOffer");
      query.findInBackground(new FindCallback<JobOffer>() {
        @Override
        public void done(List<JobOffer> list, ParseException e) {

          ParseGeoPoint geoPoint;

          for(int i =0;i<list.size();i++){
            geoPoint = list.get(i).getParseGeoPoint("coordinates");

            googleMap.addMarker(new MarkerOptions()
            .position(new LatLng(geoPoint.getLatitude(), geoPoint.getLongitude())));
          }

        }
      });

    }
  }
}
```

这些标记现在应该在应用中可见：

![为每个位置显示标记](img/4887_10_06.jpg)

## 向标记添加标题

地图上标记的实用性不仅仅在于显示一个点，而且还在于为用户提供一种简单且易于访问的方式来获取此位置的信息。我们将通过在点击标记时显示标题来实现这一点。

这可以通过简单地在我们的 `addMarker()` 方法中添加 `.title(string)` 来实现：

```kt
googleMap.addMarker(new MarkerOptions()
.position(new LatLng(geoPoint.getLatitude(), geoPoint.getLongitude()))
.title(list.get(i).getTitle()));
```

现在我们有一个完全功能的地图显示，当用户点击标记时，它会在标记上方显示一个标题，如下面的图片所示：

![向标记添加标题](img/4887_10_07.jpg)

# 总结

在本章中，你学习了如何向我们的应用添加地图。这需要我们在 Google Developer Console 上创建一个项目，并配置我们的应用以访问必要的 API。一旦我们的应用完全配置好，我们就继续将地图添加到我们选择的视图中。我们讨论了在另一个片段（`MyMapFragment` 中的 `MapFragment`）中处理片段。尽管单个 `MapFragment` 可以通过代码单独添加，但将其放在带有布局的另一个片段中，如果我们需要，可以让我们有可能向页面添加其他 UI 控件，例如 `FloatingActionButton`。最后，我们通过显示从 Parse 下载的位置的标记和信息，使地图变得有用。

在下一章中，你将学习如何调试和测试我们的应用程序。
