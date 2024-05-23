# 第一章：Spring for Android 快速入门

欢迎阅读《Spring for Android 快速入门》。本书特别为您提供了使用 Spring for Android 进行 Android 开发所需的所有信息。您将学习 Spring for Android 提供的不同功能，并开始使用这些功能构建您的第一个应用。

本书包含以下部分：

*那么，Spring for Android 是什么？* Spring for Android 被描述为 Spring 框架的扩展，它有助于简化本地 Android 应用程序的开发；目前（版本 1.0.0.RELEASE）它包括一个 REST 客户端（RestTemplate）和 OAuth 支持（Spring Social）。

*将其集成到 Android 应用中* 教授如何将 Spring for Android 与 Eclipse IDE 和 ADT（Android 开发工具）以及 Maven 集成。

*快速入门 - Rest Template* 将向您展示如何使用 Spring for Android Rest Template 模块的一些核心功能；例如，创建一个 REST 客户端。在本节结束时，您将能够使用消息转换器和/或 HTTP Basic 身份验证来检索、更新、创建和删除 REST 资源。

*快速入门 - OAuth* 将向您展示如何使用 Spring for Android Auth 模块的一些核心功能；例如，创建到 Google+ 的 OAuth 连接以读取用户资料。在本节结束时，您将能够建立 OAuth 连接来验证您的用户和应用。

*您应该了解的人和地点* —— 每个开源项目都围绕一个社区。本节为您提供了许多有用的链接到项目页面和论坛，以及一些有帮助的文章、教程和博客，此外还有 Spring for Android 优秀贡献者的 Twitter 动态。

# 那么，Spring for Android 是什么？

在这个第一部分，我们将介绍 Spring for Android 的主要方面：`RestTemplate`，`Auth`，以及 Spring for Android 不是什么。

## RestTemplate

`RestTemplate` 模块是基于 Java 的 REST 客户端 `RestTemplate` 的移植，最初在 2009 年的 Spring for MVC 中出现。与其他 Spring 模板对应物（`JdbcTemplate`, `JmsTemplate` 等）一样，它的目标是向 Java 开发者（进而也是 Android 开发者）提供一个低级别 Java API 的高级抽象；在这种情况下，它简化了 HTTP 客户端的发展。

在其 Android 版本中，`RestTemplate` 依赖于核心 Java HTTP 功能（`HttpURLConnection`）或 Apache HTTP 客户端。根据您运行应用使用的 Android 设备版本，`RestTemplate` for Android 可以为您选择最合适的一个。这是根据 Android 开发者的建议。

### 注意

请查看 [`android-developers.blogspot.ca/2011/09/androids-http-clients.html`](http://android-developers.blogspot.ca/2011/09/androids-http-clients.html)。这篇博客文章解释了在某些情况下为什么首选 Apache HTTP 客户端而不是 `HttpURLConnection`。

`RestTemplate`在 Android 平台同样支持 gzip 压缩和不同的消息转换器，用于将你的 Java 对象转换成 JSON、XML 等格式。

## Auth/Spring Social

Spring Android `Auth`模块的目标是让 Android 应用使用**OAuth**（版本 1 或 2）获得对网络服务提供商的授权。

OAuth 可能是最受欢迎的授权协议（值得一提的是，它是一个开放标准），目前被 Facebook、Twitter、Google 应用（以及其他许多应用）用来允许第三方应用程序访问用户账户。

Spring for Android 的`Auth`模块基于多个 Spring 库，因为它需要通过加密安全地（通过 JDBC）持久化通过 HTTP 获得的令牌；以下是 OAuth 所需的库列表：

+   **Spring Security Crypto**：用于加密令牌

+   **Spring Android OAuth**：这扩展了`Spring Security Crypto`，为 Android 添加了一个专用的加密器，以及基于 SQLite 的持久化提供者

+   **Spring Android Rest Template**：用于与 HTTP 服务交互

+   **Spring Social Core**：OAuth 工作流抽象

在执行 OAuth 工作流时，我们还需要浏览器将用户带到服务提供商的身份验证页面，例如，以下是 Twitter OAuth 认证对话框：

![Auth/Spring Social](img/1905_01_02.jpg)

## Spring for Android 不是什么

SpringSource（Spring for Android 背后的公司）在 Java 开发者中非常有名。他们最受欢迎的产品是包括依赖注入框架（也称为控制反转框架）的 Spring Framework for Java。Spring for Android 没有为 Android 平台带来控制反转。

在其第一个版本（1.0.0.M1）中，Spring for Android 为 Android 带来了一个通用的日志外观；在下一个版本中，作者删除了它。

# 在 Android 应用中集成

Spring for Android 包含多个 JAR 库，应该与项目关联。这些 JAR 库不是标准 Android 发行版的一部分。例如，为了使用 Spring For Android 的`RestTemplate`消费 JSON REST API，我们需要以下 JAR 库：

![在 Android 应用中集成](img/1905_02_01.jpg)

## 添加或使用 RestTemplate 的最小依赖

你可以使用 IDE 手动满足依赖关系（及其传递依赖）。手动依赖管理的替代方法是使用**Maven**进行自动项目构建。我们将在后面的部分解释 Maven 构建。

## 开始使用 Eclipse 和 Spring 进行 Android 开发

Eclipse 无疑是开发 Android 应用最受欢迎的 IDE 之一；其受欢迎的原因之一是因为由谷歌维护的**Android 开发工具**（**ADT**）提供了 Eclipse 插件，简化了 Android 开发流程（如调试器、自定义 XML 编辑器等）。

### 注意

IntelliJ IDEA Community Edition 默认支持 Android；Netbeans 也允许你安装`nbandroid`插件，这有助于 Android 应用开发。

我们需要执行以下步骤才能开始使用 Eclipse：

1.  从[`www.eclipse.org/downloads`](http://www.eclipse.org/downloads)下载最新版本（ADT 与从版本 3.6.2 开始的 Eclipse 兼容；在撰写本文时，4.2 Juno 是最新的）。优先选择*Eclipse IDE for Java developers*而不是其他可用的版本。

1.  将其下载并解压到您的机器上后，启动它。选择一个工作区位置（你的项目将位于此处），并安装 ADT 插件：点击**帮助** | **Eclipse Marketplace...**，在文本框中输入**adt**（如下截图所示），然后按*回车*；现在点击**安装**按钮，选择**Android Development Tools for Eclipse**。

### 注意

由于 Spring For Android 库仅包含 Java 库，因此你无需选择支持本地 Android 开发（使用 C 或 C++语言）的 NDK 支持功能。

![开始使用 Eclipse 和 Spring for Android](img/1905_02_02.jpg)

### 安装 ADT 插件

Eclipse 会多次提示你关于许可协议，并最终要求你重启它。

1.  当你回到工作区时，请确保你的机器上安装了（最新的）Android SDK：点击一个 Android 机器人从盒子中出来的图标，安装或更新 Android SDK（你不需要所有的 Android 版本，只需安装最受欢迎的版本，例如 2.2 又名 Froyo，2.3.3 又名 Gingerbread，4.0.3 又名 Ice Cream Sandwich 和 4.1 又名 Jelly Bean）；完成后重启 Eclipse。![安装 ADT 插件](img/1905_02_03.jpg)

    Android SDK 管理器用于管理 Android 工具和平台

1.  如果你还没有这样做，你还需要创建一个**Android Virtual Device**（**AVD**），这样你就可以将你的 Android 应用部署到它上面（你也可以通过 USB 连接手机或平板电脑来部署应用）；为此，点击表示设备屏幕中的 Android 机器人的图标，并创建一个新的 AVD，如下截图所示：![安装 ADT 插件](img/1905_02_04.jpg)

    创建新的 AVD

    ### 注意

    你可以从官方网站找到关于 ADT 插件的更多详细信息：[`developer.android.com/tools/sdk/eclipse-adt.html`](http://developer.android.com/tools/sdk/eclipse-adt.html)。

1.  最后，我们需要下载 Spring for Android 的 JAR 文件，访问 Spring for Android 网站：[`www.springsource.org/spring-android`](http://www.springsource.org/spring-android)，然后点击**下载**按钮（在接下来的屏幕中你可以跳过注册）。现在选择最新的 Spring for Android 版本（在撰写本文时：1.0.1.RELEASE），在你的机器上解压；下一步我们需要这些库：![安装 ADT 插件](img/1905_02_04_bis.jpg)

    Spring for Android 官方下载页面

### 第一个示例应用

现在，我们准备使用 Spring for Android 创建我们的第一个应用程序：

1.  点击**File** | **New...** | **Android Application**，输入你的项目名称，并在接下来的对话框中接受默认设置：![第一个示例应用](img/1905_02_05.jpg)

1.  将会弹出一个名为**New Android Application**的新窗口，如下面的截图所示：![第一个示例应用](img/1905_02_05.jpg)

    从 Eclipse 创建 Android 应用

1.  当被要求创建新的活动时，选择默认的**BlankActivity**，如下面的截图所示：![第一个示例应用](img/1905_02_05-bis.jpg)

1.  现在，将你之前下载的 Spring for Android ZIP 文件中的 JARs `spring-android-core-{version}.jar`和`spring-android-rest-template-{version}.jar`复制到新项目的`$Project_home/libs`文件夹中；你的项目结构应该如下所示：![第一个示例应用](img/1905_02_06.jpg)

    第一个示例应用

1.  为了使第一个示例应用程序能够使用 Spring for Android 请求 Web 服务（我们将使用[`ifconfig.me/all`](http://ifconfig.me/all)），我们只需对这个 URL 执行`GET`方法，并将收到客户端信息：IP、用户代理等。然后，我们需要声明我们访问网络的意图。这通过在 Android 清单文件的`application`标签之前添加以下`permission`标签来实现：

    ```kt
    (...)<uses-sdk
            android:minSdkVersion="8"
            android:targetSdkVersion="15" />
    <uses-permission android:name="android.permission.INTERNET"/>
    <application (...)
    ```

    ### 提示

    Android 的清单文件位于项目的根目录中：`AndroidManifest.xml`。

    忘记这样做会导致日志猫视图中出现模糊的消息（所有日志都收集在这里），如`Fatal Exception`；有关权限的更多信息，请参见[`developer.android.com/guide/topics/security/permissions.html`](http://developer.android.com/guide/topics/security/permissions.html)。

    HTTP 协议定义了方法或动词，以指示对远程资源执行的操作：使用`GET`获取资源，使用`POST`存储资源，使用`PUT`更新资源，使用`DELETE`删除资源，这些都是 HTTP 动词的例子，你可以通过阅读这篇文章了解更多信息：[`en.wikipedia.org/wiki/Hypertext_Transfer_Protocol`](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)。

1.  然后，我们通过删除默认的"hello world"（`android:text="@string/hello_world"`）并替换为一个锚点来调整活动布局，我们将使用该锚点来打印 Web 服务的响应（`android:id="@+id/hello_text"`）：

    ```kt
    <RelativeLayout 

        android:layout_width="match_parent"
        android:layout_height="match_parent" >
        <TextView
            android:id="@+id/result_text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerHorizontal="true"
            android:layout_centerVertical="true"
            tools:context=".MainActivity" />
    </RelativeLayout>
    ```

    活动布局位于`res/layout/activity_main.xml`。

1.  最后，我们可以重新编写`MainActivity`本身（只需更新`onCreate()`方法）：

    ```kt
      @Override
      public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        final TextView resultTextView = 
                (TextView) findViewById(R.id.result_text);
        AsyncTask<String, Void, String> simpleGetTask =  
                    new AsyncTask<String, Void, String>() {
          @Override
          protected String doInBackground(String... params) {
            //executed by a background thread

           //create a new RestTemplate instance
            RestTemplate restTemplate = new RestTemplate();

            //add the String message converter, since the result of
                 // the call will be a String
            restTemplate.getMessageConverters().add(
                         new StringHttpMessageConverter());

            // Make the HTTP GET request on the url (params[0]),
                 // marshaling the response to a String
            return
                    restTemplate.getForObject(params[0],String.class);
            }

          @Override
          protected void onPostExecute(String result) {
            // executed by the UI thread once the background 
                 // thread is done getting the result
            resultTextView.setText(result);
          }
        };
        String url = "http://ifconfig.me/all";
        // triggers the task; it will update the resultTextView once
          // it is done
        simpleGetTask.execute(url);
      }
    ```

    ### 注意

    如果 Eclipse 提示缺少导入，请同时按下*Shift* + *Ctrl* + *O*以自动添加所需的导入。

    在这段代码中，我们首先获取到`result_text`文本视图的引用，并将其设置为名为`resultTextView`的`final`变量（`final`是因为我们需要通过内部类访问它）。

    接下来，我们创建了一个匿名内部类，扩展了 `AsyncTask` 以实现访问网络服务的所有逻辑（创建 `RestTemplate` 实例，添加 `String` 转换器并调用 `getForObject`），并将结果设置到文本视图中（使用 `setText` 方法）。当我们调用 `simpleGetTask.execute(url)` 时，URL 被添加到 `doInBackground(String... params)` 中的参数数组中（在这种情况下，`params[0]` 就是 `url`）。

    ### 注意

    如果我们直接在 `onCreate()` 方法中编写访问网络服务的代码，应用程序甚至将无法启动，因为 Android 平台阻止开发者在 UI（主）线程中执行 HTTP 请求（并且有很好的理由：这些与网络相关的操作通常需要时间来完成，并且在操作进行时会冻结 UI）。

    ```kt
    @Override
    public void onCreate(Bundle savedInstanceState) {
     TextView resultTextView = (TextView) findViewById(R.id.result_text);
     String url = "http://ifconfig.me/all";
     RestTemplate restTemplate = new RestTemplate();
     RestTemplate.getMessageConverters().add(new  StringHttpMessageConverter());
     String result = restTemplate.getForObject(url, String.class);
     resultTextView.setText(result);
    }
    ```

    ### 提示

    不要这样做！你不能从 UI（主）线程执行与 HTTP 相关的调用。

    ### 注意

    关于 `AsyncTasks` 的更多信息可以在官方文档中找到：[`developer.android.com/reference/android/os/AsyncTask.html`](http://developer.android.com/reference/android/os/AsyncTask.html)，以及关于 Android 中的一般线程和进程信息：[`developer.android.com/guide/components/processes-and-threads.html`](http://developer.android.com/guide/components/processes-and-threads.html)。

1.  我们现在准备启动这个基于 Spring for Android 的第一个应用！

    右键点击项目名称并选择 **运行方式...** | **Android 应用程序**：

    ![第一个示例应用](img/1905_02_06-bis.jpg)

1.  Eclipse 将启动你之前创建的 AVD 并自动运行应用，你应该会看到类似于以下截图的内容：![第一个示例应用](img/1905_02_07.jpg)

    来自 AVD 的 MainActivity 截图

恭喜！你已经成功运行了你的第一个基于 Spring for Android 的应用！

## 使用 Maven 构建 Spring for Android 应用

在上一个示例中，我们只向 `libs` 文件夹中添加了两个 JAR 文件；但是如果你需要其他依赖，比如 `MessageConverters`（用于解组 JSON、XML 响应），`Auth` 模块，或项目中的任何其他现有库，管理你的依赖关系很快就会成为负担！

Maven 可以帮助你管理依赖关系（简而言之，你在一个名为 `pom.xml` 的文件中指定它们，它会自动从互联网上获取它们，包括它们的传递依赖关系）以及更多：发布操作，测试运行，头部生成等等。Maven 是一个模块化的开源构建工具，拥有大量的插件！

### 注意

使用 Android Maven 插件时，你应在环境中添加以下变量 `ANDROID_HOME`（指向你的 Android 主目录）并将 `ANDROID_HOME/platform-tools` 添加到你的路径中。这详细解释在[`code.google.com/p/maven-android-plugin/wiki/GettingStarted`](https://code.google.com/p/maven-android-plugin/wiki/GettingStarted)。

首先，让我们向 Eclipse 添加 **m2e**（Maven 到 Eclipse）插件：就像我们对 ADT 插件所做的那样，打开 **帮助** | **Eclipse** 市场并搜索 **maven**；选择 **Maven Integration for Eclipse**，点击 **安装** 并确认对话框；在过程结束时重启你的 IDE。

![使用 Maven 构建 Spring for Android 应用](img/1905_02_08.jpg)

安装 Eclipse 的 Maven 集成

现在将名为 `spring-for-android-first-example-maven` 的项目导入你的工作空间：你可以从 GitHub 克隆它，或者解压 `examples.zip` 压缩包；然后，在 Eclipse 中，点击 **文件** | **导入…** 并选择 **现有 Maven 项目**，然后点击 **下一步**。

![使用 Maven 构建 Spring for Android 应用](img/1905_02_09.jpg)

导入现有 Maven 项目

浏览到克隆或解压项目的目录并点击 **下一步** 按钮：

![使用 Maven 构建 Spring for Android 应用](img/1905_02_10.jpg)

浏览到你的 Maven 项目

M2e 将会显示一个 m2e 连接器列表，这些连接器将 Maven 项目集成到你的 IDE 中（基本上这些连接器告诉 m2e 如何生成和编译源代码）；点击 **完成** 并接受后续对话框；重启 Eclipse。

### 注意

这里最重要的连接器是 m2e-android 连接器：它使 m2e 能够与 ADT 集成，更多信息请访问 [`rgladwell.github.com/m2e-android/`](http://rgladwell.github.com/m2e-android/)。

项目应该已经顺利导入，你可以像上一个项目一样在 AVD 上部署项目（右键点击项目，并选择 **运行方式** | **Android 应用程序**）。

让我们查看一下 `pom.xml` 文件（与上一个项目相比唯一的附加文件），右键点击它并选择 **打开方式…** | **Maven POM 编辑器**；点击视图底部的 **pom.xml** 标签。

```kt
(...)<dependencies>
        <dependency>
            <groupId>com.google.android</groupId>
            <artifactId>android</artifactId>
            <version>4.0.1.2</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
          <groupId>org.springframework.android</groupId>
          <artifactId>spring-android-rest-template</artifactId>
          <version>${spring-android-version}</version>
        </dependency>
    </dependencies>
    <build>
        <finalName>${project.artifactId}</finalName>
        <sourceDirectory>src</sourceDirectory>
        <plugins>
            <plugin>
        <groupId>com.jayway.maven.plugins.android.generation2</groupId>
                <artifactId>android-maven-plugin</artifactId>
                <version>3.3.0</version>
                <extensions>true</extensions>
                <configuration>
                    <sdk>
                        <!-- platform or api level (api level 4 = platform 1.6)-->
                        <platform>15</platform>
                    </sdk>
                </configuration>
            </plugin>
        </plugins>
    </build>(...)
```

在依赖关系部分，你可以看到 Android 平台和 Spring for Android `RestTemplate` 工件（无需指定 Spring for Android 核心！记住，Maven 会自动获取传递依赖），以及在插件部分配置为需要 Android 4.0.3（API 级别 15）的 Android-Maven 插件（它通过 `extensions=true` 属性扩展 Maven 以支持 Android 项目）。

Maven 的核心是一个命令行工具，如果你在机器上下载并解压 Maven，可以进入 Maven 项目的根目录（即 `pom.xml` 所在的位置）并输入带有不同目标的 Maven 命令：

+   `mvn clean install`：编译，运行测试并打包

+   `mvn clean install android:run`：在 AVD 中启动你的应用

### 注意

要查看所有可用的目标，你可以输入 `mvn help:describe -Dplugin=com.jayway.maven.plugins.android.generation2:maven-android-plugin`。

你还可以在 Eclipse 内部模拟命令行 Maven，右键点击项目并选择 **运行方式** | **Maven 构建...**，并指定你的目标。

Maven 安装说明可以在网上找到：[`www.sonatype.com/books/mvnref-book/reference/installation-sect-maven-install.html`](http://www.sonatype.com/books/mvnref-book/reference/installation-sect-maven-install.html)。

要了解更多关于 Maven 的信息，我强烈建议阅读在线免费提供的 Maven 书籍，它甚至有一个专门的 Android 章节，可以在[`www.sonatype.com/books/mvnref-book/reference/`](http://www.sonatype.com/books/mvnref-book/reference/)找到。

现在我们准备开始构建一些更丰富的基于 Spring for Android 的应用程序！

# 快速开始 - RestTemplate

既然我们的工作空间已经设置好，并且我们已经成功部署了第一个基于 Spring for Android `RestTemplate`的应用程序，那么让我们探索`RestTemplate`的更多功能。

本节依赖于几个示例应用程序。我们将逐一讲解每个代码样本的重要点。随时可以将项目导入你的 IDE 并浏览代码。

### 注意

从现在开始，我将使用 Maven 来管理依赖项，但你不必如此。每个项目在它的`README.md`文件中都有依赖项列表：下载它们（或者从你的 Spring for Android ZIP 存档中的`libs`文件夹获取），并将它们添加到`libs`文件夹中！

## 不同的消息转换器

在上一节中，我们已经使用了一个消息转换器：`StringHttpMessageConverter`。每个消息转换器支持从和向特定的媒体类型（通常称为 MIME 类型）读取和写入；`StringHttpMessageConverter`支持读取所有类型的文档（MIME */*）以及写入文本（MIME text/plain）。让我们来发现更复杂的信息转换器。

### 注意

**MIME**（**多用途互联网邮件扩展**）不仅用于描述电子邮件的内容，也广泛应用于通过 HTTP 交换的文档内容的描述（[`en.wikipedia.org/wiki/MIME`](http://en.wikipedia.org/wiki/MIME)）。

### MappingJacksonHttpMessageConverter

JSON 消息在 REST 网络服务中非常常见。最初为 JavaScript 语言（`JavaScriptObjectNotation`）设计；它们轻量级且易于阅读。

`Ifconfig.me`可以返回一个 JSON 响应，如果你使用这个 URL：[`ifconfig.me/all.json`](http://ifconfig.me/all.json)。

在我们第一个例子的代码中，让我们用`MappingJacksonHttpMessageConverter`替换`StringHttpMessageConverter`：

```kt
AsyncTask<String, Void, IfConfigMeJson> simpleGetTask =  
  new AsyncTask<String, Void, IfConfigMeJson>() {
  @Override
  protected IfConfigMeJson doInBackground(String... params) {
    String url = params[0];
    RestTemplate restTemplate = new RestTemplate();
    MappingJacksonHttpMessageConverter jacksonConverter = 
         new MappingJacksonHttpMessageConverter();
    restTemplate.getMessageConverters().add(jacksonConverter);
    return restTemplate.getForObject(url,
           IfConfigMeJson.class);
  }

  @Override
  protected void onPostExecute(IfConfigMeJson result) {
    String resultAsString =  new StringBuilder()
                      .append("Your current IP is : ")
                      .append(result.getIpAddr()).toString();
    resultTextView.setText(resultAsString );
  }

};

String url = "http://ifconfig.me/all.json";
//triggers the task
simpleGetTask.execute(url);
```

如你所见，我们还引入了一个名为`IfConfigMeJson`的模型类：这个类定义了 JSON 消息属性（`{"connection":"keep-alive","ip_addr":"70.30.43.43",[..]}`）与一个**POJO**（**普通旧 Java 对象**：一个带有成员变量及其访问器的简单类）之间的映射，这样我们就可以在我们的 Java 代码中轻松访问响应的每个属性（在以下示例中：`result.getIpAddr()`)。

```kt
public class IfConfigMeJson {
  private String connection;
  @JsonProperty("ip_addr")
  private String ipAddr;
  public String getConnection() {
    return connection;
  }
  public void setConnection(String connection) {
    this.connection = connection;
  }
  public String getIpAddr() {
    return ipAddr;
  }
  public void setIpAddr(String opAddr) {
    this.ipAddr = opAddr;
  }
```

### 注意

只要它们的名称相同（Java 中为驼峰命名，JSON 中为下划线），Jackson 就会自动将 JSON 属性与 Java 字段匹配。你肯定注意到了，我们使用 Jackson 的`@JsonProperty`注解在字段定义之前，将 JSON 属性`ip_addr`映射到字段`ipAddr`，以遵守 Java 命名约定。顺便一提，我们可以将类的字段声明为 public，并移除访问器。

当你运行应用程序时，你只会看到以下截图：

![MappingJacksonHttpMessageConverter](img/1905_03_01.jpg)

使用`MappingJacksonHttpMessageConverter`的应用程序

在本节中，我们在`pom.xml`文件中添加了对`jackson-mapper-asl`的依赖：

```kt
<dependency>
<groupId>org.codehaus.jackson</groupId>
  <artifactId>jackson-mapper-asl</artifactId>
  <version>${jackson-version}</version>
</dependency>
```

### 注意

另外还有一个可用的 JSON 消息转换器，`GsonHttpMessageConverter`，它使用 Google 的 Gson 库而不是 Jackson，可以作为替代使用。

### `SimpleXmlHttpMessageConverter`

另一个值得一提的消息转换器是`SimpleXmlHttpMessageConverter`：它将 XML 响应和请求映射到 POJO，就像`MappingJacksonHttpMessageConverter`处理 JSON 一样。

作为一个示例，让我们请求`Ifconfig.me`的 XML 服务：[`ifconfig.me/all.xml`](http://ifconfig.me/all.xml)。

在我们上一个示例中的`MainActivity`类，我们将`MappingJacksonHttpMessageConverter`替换为`SimpleXmlHttpMessageConverter`：

```kt
AsyncTask<String, Void, IfConfigMeXml> simpleGetTask =  
   new AsyncTask<String, Void, IfConfigMeXml>() {
   @Override
   protected IfConfigMeXml doInBackground(String... params) {
      String url = params[0];
RestTemplate restTemplate = new RestTemplate();
      SimpleXmlHttpMessageConverter xmlConverter = 
           new SimpleXmlHttpMessageConverter();
      restTemplate.getMessageConverters().add(xmlConverter);
      return restTemplate.getForObject(url, IfConfigMeXml.class);
   }

   @Override
   protected void onPostExecute(IfConfigMeXml result) {
       String resultAsString =  new StringBuilder()
                  .append("Your current IP is : ")	
                  .append(result.getIpAddr()).toString();
       resultTextView.setText(resultAsString );
   }

};

String url = "http://ifconfig.me/all.xml";

```

你会再次注意到，我们依赖一个名为`IfConfigMeXml.java`的类，允许`SimpleXml`在 XML 响应和 Java 代码之间进行映射。以下是 XML 响应内容：

```kt
<info>
<forwarded/>
<ip_addr>70.43.43.43</ip_addr>
[...]
</info>
```

以下是 Java 代码：

```kt
@Root (name="info")
public class IfConfigMeXml {

  @Element(required=false)
  private String forwarded;

  @Element(name="ip_addr")
  private String ipAddr;
  [...]
}
```

这些注解与我们对 JSON 消息转换器使用的类似。

`@Root`注解定义了根 XML 标签的名称：在这种情况下是`info`。

`@Element`注解在每个字段之前添加，让`SimpleXml`知道这些字段映射到 XML 标签：如果`required`设置为 false，意味着标签可能是空的；如果指定了`name`，意味着 POJO 字段与 XML 标签名不匹配。

如果你运行应用程序，你将得到与上一个示例完全相同的输出。

### 注意

在本节中，我们向`SimpleXml`框架添加了一个依赖项。不幸的是，这个依赖项具有对 stax 和 xpp3 的传递依赖，这对于 Android 应用程序是不需要的。

我们不得不显式过滤一些传递依赖，让 Maven 知道不要将它们添加到类路径中。

```kt
<dependency>
  <groupId>org.simpleframework</groupId>
  <artifactId>simple-xml</artifactId>
  <version>${simple-version}</version>
  <exclusions>
    <!-- StAX is not available on Android -->
    <exclusion>
      <artifactId>stax</artifactId>
      <groupId>stax</groupId>
    </exclusion>
    <exclusion>
      <artifactId>stax-api</artifactId>
      <groupId>stax</groupId>
    </exclusion>
    <!-- Provided by Android -->
    <exclusion>
      <artifactId>xpp3</artifactId>
      <groupId>xpp3</groupId>
    </exclusion>
  </exclusions>
</dependency>
```

### `RssChannelHttpMessageConverter`

这个消息转换器是关于解析 RSS 提要的；像往常一样，我们将它注入到我们的`RestTemplate`实例中，但这次我们将从博客提要中读取条目。

```kt
final WebView resultTextView = (WebView) findViewById(R.id.result_text);
AsyncTask<String, Void, Channel> simpleGetTask =  new AsyncTask<String, Void, Channel>() {
   @Override
   protected Channel doInBackground(String... params) {
      RestTemplate restTemplate = new RestTemplate();
      // Configure the RSS message converter.
       RssChannelHttpMessageConverter rssChannelConverter = new RssChannelHttpMessageConverter();
      rssChannelConverter.setSupportedMediaTypes(
         Collections.singletonList(MediaType.TEXT_XML));

        // Add the RSS message converter to the RestTemplate instance
        restTemplate.getMessageConverters().add(rssChannelConverter);

      // Make the HTTP GET request on the url (params[0]), marshaling //the response to a String
      return restTemplate.getForObject(params[0], Channel.class);
   }

   @Override
   protected void onPostExecute(Channel result) {
      //get the latest article from the blog
      Item item = (Item) result.getItems().get(0);

      // load the content of the article into the WebView
      resultTextView.loadData(item.getContent().getValue(), "text/html", "UTF-8");
   }
};
String url = "http://blog.dahanne.net/feed/";
```

在这里我们不需要创建一个 POJO 映射类，因为结果将始终是一个`Channel`——一个为我们提供访问提要不同属性的方法的类：项目、语言、发布日期等等。在这个例子中，我们只读取第一个项目的内容——我的博客的最新文章！

提要如下所示：

```kt
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0">
<channel>
   <title>Anthony Dahanne's blog</title>
   <description>A blog</description>
   [...]
   <item>
      <title>Article 1</title>
      <description>content of the article</description>
      [...]
   </item>
   [...]
</channel>
</rss>
```

设备上的结果如下所示：

![RssChannelHttpMessageConverter](img/1905_03_02.jpg)

RSS feed 中第一项的内容

### 注意

你可能已经注意到，我将之前的 TextView 替换成了一个能够解释超链接等 HTML 代码的 WebView。

`RssChannelHttpMessageConverter` 没有默认支持的类型，这就是为什么我们添加了媒体类型 `text/xml` 作为支持类型的原因。

### 注意

为了这个 RSS feed 示例，我们不得不添加了对 Android ROME Feed Reader 的依赖；这个库没有发布在 Maven Central 上，这意味着我们也必须声明它的构件部署到的仓库。

```kt
<dependency>
   <groupId>com.google.code.android-rome-feed-reader</groupId>
   <artifactId>android-rome-feed-reader</artifactId>
   <version>${android-rome-version}</version>
</dependency>

<repositories>
   <repository>
      <id>android-rome-feed-reader-repository</id>
      <name>Android ROME Feed Reader Repository</name>
      <url>https://android-rome-feed-reader.googlecode.com/svn/maven2/releases</url>
   </repository>
</repositories>
```

## Gzip 压缩

为了在传输过程中节省带宽，你可以作为请求的一部分告诉服务器你支持 gzip 压缩。如果服务器支持对响应进行 gzip 压缩，它将会这样做。

Spring for Android 的 `RestTemplate` 依赖于自 Gingerbread (2.3) 版本以来的 Android `java.net.HttpURLConnection`，默认情况下会发送 `Accept-Encoding : gzip` 头部；因此默认开启 gzip 压缩。

否则，你只需在请求的头部指定你支持 gzip 压缩即可。让我们来看一个具体的例子——Jackson 示例，这次我们只展示了客户端支持的编码机制：

```kt
@Override
protected IfConfigMeJson doInBackground(String... params) {

   HttpHeaders requestHeaders = new HttpHeaders();
   requestHeaders.setAcceptEncoding(ContentCodingType.IDENTITY);
// Add the gzip Accept-Encoding header   
//requestHeaders.setAcceptEncoding(ContentCodingType.GZIP);
   HttpEntity<?> requestEntity = 
        new HttpEntity<Object>(requestHeaders);
   RestTemplate restTemplate = new RestTemplate();
   MappingJacksonHttpMessageConverter jacksonConverter = 
        new MappingJacksonHttpMessageConverter();
   restTemplate.getMessageConverters().add(jacksonConverter);
   ResponseEntity<IfConfigMeJson> response = 
        restTemplate.exchange(params[0], HttpMethod.GET, 
                      requestEntity, IfConfigMeJson.class);
   return response.getBody();
}
```

使用 `Identity`，我们声明除了原始文本外不支持其他任何编码。

### 注意

当你显式或隐式地请求 gzip 压缩时，`java.util.zip.GzipInputStream` 将自动用于解码响应。

你注意到我们没有使用 `getForObject(url,Pojo.class)`，而是使用了 `exchange(url,GET,requestEntity,Pojo.class)` 吗？`exchange()` 方法是一种更灵活的方法，允许自定义请求。

让我们看看这段代码执行期间交换的请求/响应：

| 请求（IDENTITY 然后 GZIP） | 响应（IDENTITY 然后 GZIP） |
| --- | --- |
| `GET/all.json HTTP/1.1``Accept: application/json``Accept-Encoding: identity``Content-Length: 0``Host: ifconfig.me:80``Connection: Keep-Alive` | `HTTP/1.1 200 OK``Date: Fri, 02 Nov 2012 02:41:04 GMT``Server: Apache``Vary: Accept-Encoding``Connection: close``Transfer-Encoding: chunked``Content-Type: application/json``13c``{"connection":"Keep-Alive","ip_addr:[...],"encoding":"identity"}``0` |
| `GET /all.json HTTP/1.1``Accept: application/json``Accept-Encoding: gzip``Content-Length: 0``User-Agent: Dalvik/1.6.0 (Linux; U; Android 4.0.4; sdk Build/MR1)``Host: ifconfig.me:80``Connection: Keep-Alive``X-Forwarded-For: 192.168.1.111` | `HTTP/1.1 200 OK` `—îPq``Date: Fri, 02 Nov 2012 02:42:08 GMT``Server: Apache``Vary: Accept-Encoding``Content-Encoding: gzip``Content-Length: 258``Connection: close``Content-Type: application/json``­ã/%êAOÑ0 ÖÔ˛``"ì&À–"ß5ﬁ'ãâg2€él•¥§4�ˇªÖΩÕ&#124;3ÔeÊ˝2È¨%9kgŸ/{&ö"ì—+±/"SáJ˘àk` |

### 注意

分析设备间往返的请求和响应通常很有帮助；你可以使用开源软件 Membrane 作为 HTTP 代理（[`www.membrane-soa.org/soap-monitor/`](http://www.membrane-soa.org/soap-monitor/)），创建一个新的简单代理，监听 localhost:2000 并重定向到你目标的服务（例如，[`ifconfig.me`](http://ifconfig.me)）。

在 Android 代码中，只需将你的 Web 服务 URL 替换为`computer_ip:2000`（可能是`192.168.1.1:2000`）：然后你将看到你的 Android 应用和 Web 服务 URL 之间的所有流量。

## HTTP 基本认证

如其名称所示，这种认证方案非常基础且不安全。你只需提供一个用户名和密码即可访问一个域（通常是一个网站的一部分）。

正如我们对编码所做的那样，我们可以通过一个头文件提供凭据：用户名和密码将被 Base64 编码。

### 注意

Base64 编码绝对不提供安全性。你可以像编码一样轻松地解码它！有关 Base64 的更多信息，请参见[`en.wikipedia.org/wiki/Base64`](https://en.wikipedia.org/wiki/Base64)。

作为一个例子，我们将尝试访问一个显示**Hello World**的网页；只要你发送了正确的凭据。该页面是[`restfulapp.appspot.com/helloworld`](http://restfulapp.appspot.com/helloworld)，凭据是用户名 - `s4a` 和密码 - `s4a`（你可以在你的网页浏览器中尝试）。

让我们尝试使用 Spring for Android 的`RestTemplate`来访问这个页面！

```kt
AsyncTask<String, Void, String> simpleGetTask =  new AsyncTask<String, Void, String>() {
   @Override
   protected String doInBackground(String... params) {
      // Set the credentials for creating a Basic Auth request
      HttpAuthentication authHeader = 
          new HttpBasicAuthentication("s4a", "s4a");
      HttpHeaders requestHeaders = new HttpHeaders();
      requestHeaders.setAuthorization(authHeader);
      HttpEntity<?> requestEntity = 
          new HttpEntity<Object>(requestHeaders);

      RestTemplate restTemplate = new RestTemplate();

      restTemplate.getMessageConverters()
              .add(new StringHttpMessageConverter());

      try {
          // Make the GET request to the Basic Auth protected URL
          ResponseEntity<String> response = 
                restTemplate.exchange(params[0], HttpMethod.GET, 
                       requestEntity, String.class);
          return response.getBody();
      } catch (HttpClientErrorException e) {
         // Handle 401 Unauthorized response
         Log.e("MainActivity",e.getLocalizedMessage(),e);
         return "Wrong credentials";
      }
   }

   @Override
   protected void onPostExecute(String result) {
      // executed by the UI thread		 
      resultTextView.setText(result);
   }

};
```

就像在上一个示例中设置`Accepted-Encoding`一样，我们将一个**HttpBasicAuthentication**头设置到`RestTemplate`头中。

让我们看一下在执行此代码期间交换的请求/响应：

| 请求 | 响应 |
| --- | --- |
| `GET/helloworld HTTP/1.1``Accept: text/plain, */*``Authorization: Basic czRhOnM0YQ==``Content-Length: 0``User-Agent: Dalvik/1.6.0 (Linux; U; Android 4.0.4; sdk Build/MR1)``Host: restfulapp.appspot.com:80``Connection: Keep-Alive``Accept-Encoding: gzip` | `HTTP/1.1 200 OK``Content-Type: text/plain; charset=iso-8859-1``Vary: Accept-Encoding``Date: Fri, 02 Nov 2012 03:33:06``Server: Google Frontend``Cache-Control: private``Transfer-Encoding: chunked``d``Hello, world``0` |

### 注意

字符串`czRhOnM0YQ==`是`s4a:s4a`的 Base64 编码。

在下一节*快速入门 - OAuth*中，我们将介绍一种更好的用户身份验证和授权方法。

## RESTful 食谱应用

让我们研究 RESTful 食谱应用的情况：一个 Android 应用与 RESTful 服务交互，以创建、更新、读取和删除食谱。

一个食谱相对简单 - 它是一个具有以下字段的 POJO：

+   一个 ID（`Integer`）

+   一个标题（`String`）

+   食谱的描述（`String`）

+   一个类型：通常是主菜、主菜或甜点（`String`）

+   一个作者（`String`）

我们将使用的在线网络应用作为后端服务，是一个基于 JAX-RS（JEE 规范 REST 服务）的应用程序，使用 Jersey（参考 JAX-RS 实现库）并在 Google App Engine 上部署——它本可以用 Ruby、Python 或任何其他语言编写。

### 注意

如果你已经研究过 Android 示例中的记事本应用，你会对 RESTful 食谱应用很熟悉：它是基于记事本应用代码编写的，将持久化从嵌入式 SQLite 数据库替换为 REST 在线服务。

### RecipeList 活动：列出服务器上存储的所有食谱

当用户启动应用时，他首先想看到的是可用的食谱列表。

此活动使用`ListView`，并配有`ListAdapter`向用户展示内容。

但首先，它需要从服务器获取它们：

+   我们需要一个模型（`Recipe.java`）：

    ```kt
    public class Recipe {

       private Long id;
       private String title;
       private String description;
       private String type;
       private String author;
    }
    ```

+   以及一个嵌入在 AsyncTask 中的`RestTemplate`实例：

    ```kt
    private class GetRecipesTask extends RecipeAbstractAsyncTask <Void, Void, List<Recipe>> {

       @Override
       protected void onPreExecute() {
          showProgressDialog("Loading recipes. Please wait...");
       }

       @Override
       protected List<Recipe> doInBackground(Void... params) {
          HttpHeaders requestHeaders =  
                   prepareHeadersWithMediaTypeAndBasicAuthentication();

          // Populate the headers in an HttpEntity object		
          HttpEntity<?> requestEntity = 
                   new HttpEntity<Object>(requestHeaders);

          // Create a new RestTemplate instance
          RestTemplate restTemplate = new RestTemplate();
          restTemplate.getMessageConverters()
    .add(new MappingJacksonHttpMessageConverter());

          try {
             // Perform the HTTP GET request
             ResponseEntity<Recipe[]> responseEntity = 
                restTemplate.exchange(
                   "http://www.restfulapp.appspot.com/rest/recipes/",
                   HttpMethod.GET, requestEntity,
       Recipe[].class);
             return Arrays.asList(responseEntity.getBody());
          }
          catch (RestClientException e) {
             Log.e(TAG, e.getMessage(), e);
             exception = e;
             return null;
          }
       }

       @Override
       protected void onPostExecute(List<Recipe> result) {
          dismissProgressDialog();
          if(result != null) {
             recipes = result;
          } else {
             String message = exception != null ? 
                exception.getMessage() : "unknown reason";
             Toast.makeText(RecipesList.this, 
                "A problem occurred during the reception of all recipes
                : " +message , Toast.LENGTH_LONG).show();
              recipes = new ArrayList<Recipe>();
          }
          ListAdapter adapter = new RecipeAdapter(RecipesList.this, 
             R.layout.recipeslist_item, recipes ) ;
          setListAdapter(adapter );
       }
    }
    ```

每当我们需要获取更新的食谱列表时，都会执行此任务：

+   当活动创建（或恢复）时：`onResume()`

+   当我们成功更新操作返回后：`onActivityResult()`

您可能已经注意到，这个 AsyncTask 扩展了`RecipeAbstractAsyncTask`，这是一个定义了异常和实用方法`prepareHeadersWithMediaTypeAndBasicAuthentication()`的类：

```kt
protected HttpHeaders prepareHeadersWithMediaTypeAndBasicAuthentication() {
      HttpHeaders requestHeaders = new HttpHeaders();
      List<MediaType> acceptableMediaTypes = new ArrayList<MediaType>();
      acceptableMediaTypes.add(MediaType.APPLICATION_JSON);
      requestHeaders.setAccept(acceptableMediaTypes);
      HttpAuthentication authHeader = new HttpBasicAuthentication("s4a", "s4a");
      requestHeaders.setAuthorization(authHeader);
      return requestHeaders;
   }
```

如其名所示，它会准备请求头，包括所需的 Basic 身份验证和服务器上的期望`MediaType`对象。

回到`GetRecipesTask`：我们准备请求头，创建一个新的`RestTemplate`实例——配置为使用 Jackson 来(反)序列化消息，并执行 GET 请求：

```kt
// Perform the HTTP GET request
               ResponseEntity<Recipe[]> responseEntity = restTemplate.exchange(getString(R.string.recipe_resource_url), HttpMethod.GET, requestEntity, Recipe[].class);
```

### 注意

`R.string.resource_url`值在`strings.xml`中定义，定义为等于：[`www.restfulapp.appspot.com/rest/recipes/`](http://www.restfulapp.appspot.com/rest/recipes/)。

由于我们想要获取一个食谱列表，所以给`ResponseEntity`对象的类型是一个数组。

另一个需要注意的重要事项是异常处理程序：**RestClientException**与它的名字不符，它是一个包装服务器和客户端异常的异常。如果你想告诉用户哪里出了问题，你最好捕获这个异常，并在执行`onPostExecute()`方法时保留它直到回到 UI 线程。

### 注意

注意`RestClientException`是`RuntimeException`的子类，这就是为什么你不需要显式捕获它的原因。不过，如果你不这样做，这个异常可能会在抛出时杀死你的活动。

谈到用户界面，这个任务使用其预执行和后执行方法，通过使用`ProgresDialogs`和`Toasts`来保持用户对加载状态的了解。

### 注意

从应用程序外部手动调试 REST 资源可能很有用；**cURL**应用程序（在[`curl.haxx.se/download.html`](http://curl.haxx.se/download.html)可下载）可能是最流行的工具。

对于本节中提到的每个 HTTP 请求，我现在将包括与之匹配的 cURL 命令行。

```kt
curl --user s4a:s4a  -H "Accept: application/json" -i http://www.restfulapp.appspot.com/rest/recipes 
[{"id":0,"title":"Montreal's Poutine","description":"French fries are covered with fresh cheese curds, and topped with brown gravy.","type":"MAIN_DISH","author":"Anthony"},{"id":1,"title":"a title","description":"a description","type":"a type","author":"an author"}]
```

![食谱列表活动：列出服务器上存储的所有食谱](img/1905_03_03.jpg)

食谱列表活动概览

### 食谱编辑器活动

此活动负责获取、更新、创建和删除食谱。

![食谱编辑器活动](img/1905_03_04.jpg)

更新模式下的食谱编辑器活动

+   获取食谱

    当用户在食谱列表活动中点击创建图标，或者点击其中一个食谱标题时，他将直接进入食谱编辑器活动。

    如果用户想要创建一个新食谱，他将看到空白字段，但如果他想要查看食谱，他们首先会从服务器获取，然后再填写字段；这要感谢内部的`GetRecipeTask`：

    ```kt
    // Perform the HTTP GET request
    Log.i(TAG,"Getting the recipe with id : "+params[0] + " : " +url + params[0]);
    ResponseEntity<Recipe> responseEntity = restTemplate.exchange(url + params[0], HttpMethod.GET, requestEntity,	Recipe.class);
    return responseEntity.getBody();
    ```

    这儿没有真正的新内容，我们想要获取一个通过其 ID（`params[0]`）标识的单一食谱。

    ```kt
    curl --user s4a:s4a  -H "Accept: application/json" -i http://www.restfulapp.appspot.com/rest/recipes/0
    {"id":0,"title":"Montreal's Poutine","description":"French fries are covered with fresh cheese curds, and topped with brown gravy.","type":"MAIN_DISH","author":"Anthony"}
    ```

+   更新食谱

    要更新一个食谱，我们必须使用 PUT HTTP 动词请求食谱资源。

    为此，我们再次有一个专用的 AsyncTask：`UpdateRecipeTask`，它将实例化一个`RestTemplate`实例，携带更新后的食谱（一旦用户按下`保存`按钮，我们就会调用`updateNote()`方法，将视图的内容与当前食谱的模型同步）。

    ```kt
    // Create a new RestTemplate instance
    RestTemplate restTemplate = new RestTemplate();
    restTemplate.getMessageConverters()
       .add(new MappingJacksonHttpMessageConverter());

    // Populate the headers in an HttpEntity object HttpEntity<Recipe> requestEntity = new HttpEntity<Recipe>(recipe,requestHeaders);
    try {
       // Perform the HTTP PUT request
       Log.i(TAG,"Updating the recipe with id : "
          +recipe.getId() + " : " + url);
       ResponseEntity<Void> responseEntity = 
          restTemplate.exchange(url, HttpMethod.PUT, requestEntity,
          Void.class);
       if(responseEntity.getStatusCode() != HttpStatus.OK) {
          throw new HttpServerErrorException(
             responseEntity.getStatusCode());
       }
    }
    catch (RestClientException e) {
       Log.d(TAG, e.getMessage(), e);
       exception = e;
    }
    ```

    在这里需要注意的是`RequestEntity`，它包装了我们的食谱：`RestTemplate`会将它传递给 Jackson，后者将其转换为 JSON 字符串。

    如果服务器没有用 HTTP 状态 200（OK）响应，我们可以认为出现了错误，并为用户界面保留异常。

    ```kt
    curl --user s4a:s4a  -H "Accept: application/json" -H "Content-type: application/json"  -X PUT -d '{"id":"0", "title":"a title","description":"a description","type":"a type","author":"an author"}'  -i http://www.restfulapp.appspot.com/rest/recipes/
    ```

+   发布食谱

    要创建一个食谱，我们必须使用 POST HTTP 动词请求食谱资源。

    在 REST 文献中，你还会读到关于使用 PUT HTTP 动词创建实体的用法；但有一个条件——你必须提供 ID（这样连续的 PUT 不会改变资源状态。PUT 是幂等的，就像 GET 和 DELETE，而且与 DELETE 和 POST 一样不安全）。

    ### 注意

    在我们的案例中，我们创建了一个新资源，所以 POST 很好地反映了我们的意图。

    让我们看看`CreateRecipeTask`，它与`UpdateRecipeTask`非常相似：

    ```kt
    // Populate the headers in an HttpEntity object 
    HttpEntity<Recipe> requestEntity = 
       new HttpEntity<Recipe>(recipe,requestHeaders);
    try {
       // Perform the HTTP POST request
       Log.i(TAG,"Posting the recipe with id : "
          +recipe.getId() + " : to " +url);
       ResponseEntity<Void> responseEntity = 
          restTemplate.exchange(url, HttpMethod.POST, requestEntity,
          Void.class);
       if(responseEntity.getStatusCode() != HttpStatus.CREATED) {
          throw new HttpServerErrorException(
             responseEntity.getStatusCode());
       }
    }
    catch (RestClientException e) {
       Log.d(TAG, e.getMessage(), e);
       exception = e;
    }
    return null;
    ```

    有几个有趣的地方需要注意：由于用户是从零开始创建食谱，所以食谱没有设置任何 ID。Web 服务将为这个食谱分配一个 ID（并且应该返回创建资源的 URI）；预期的状态不是 200，而是 201（`HttpStatus.CREATED`）。

    ```kt
    curl --user s4a:s4a  -H "Accept: application/json" -H "Content-type: application/json"  -X POST -d '{"title":"a title","description":"a description","type":"a type","author":"an author"}'  -i http://www.restfulapp.appspot.com/rest/recipes/
    ```

+   删除食谱

    要删除一个食谱，我们必须使用 DELETE HTTP 动词请求食谱资源。

    名为`DeleteRecipeTask`的 AsyncTask 与`GetRecipeTask`相似，因为它们只需要将食谱的 ID 附加到资源 URI（无需像 PUT 和 POST 那样携带整个食谱）。

    ```kt
    try {
       // Perform the HTTP DELETE request
       Log.i(TAG,"Deleting the recipe with id : "
          +recipe.getId() + " : from " +url +recipe.getId() );
       ResponseEntity<Void> responseEntity = 
          restTemplate.exchange(url+recipe.getId(), 
          HttpMethod.DELETE, requestEntity,	Void.class);
       if(responseEntity.getStatusCode() != HttpStatus.NO_CONTENT) {
          throw new HttpServerErrorException(
          responseEntity.getStatusCode());
       }
    }
    catch (RestClientException e) {
       Log.d(TAG, e.getMessage(), e);
       exception = e;
    }
    ```

    该请求预期会有一个状态为 204 的 HTTP 响应码，意味着请求成功，但没有内容返回（`HttpStatus.NO_CONTENT`）。

    ### 注意

    HTTP 状态码 200 对于一个成功的 DELETE 请求来说是可以接受的。

    查阅你的网络服务文档（或者更好的，用 cURL 进行实验），以最佳假设了解网络服务应该返回什么。

    ```kt
    curl --user s4a:s4a  -H "Accept: application/json" -X DELETE   -i http://www.restfulapp.appspot.com/rest/recipes/1
    ```

## Android 注解

你可能听说过这个项目，它位于[`github.com/excilys/androidannotations/`](https://github.com/excilys/androidannotations/)，旨在减少在 Android 应用中注入视图、活动、提供者、服务等所需的代码量。

项目领导者已经决定使用 Spring for Android 的`RestTemplate`作为支持 REST 注解的 REST 客户端库。

### 注意

Android 注解是基于**注解处理器工具**（**apt**），现在是 Java 6 编译器的一部分；每个带注解的类都会触发生成一个子类（与原文件同名，在文件名后加上`_`，例如`OriginalActivity_.java`），其中包含所有注解为你省去编写的模板代码！

因此，除了`@Eactivity`（允许活动使用 Android 注解），`@ViewById`（注入视图）之外，还有`@Get`，`@Post`，`@Put`等等。

让我们看看如何通过 JSON 示例（从`Ifconfig.me`获取 IP 地址）从 Android 注解中受益。

```kt
@EActivity(R.layout.activity_main)
public class MainActivity extends Activity {

   private IfConfigMeJson all;

   //inject the view to the activity layout
    @ViewById(R.id.result_text)
    TextView resultTextView;

    //inject the Rest service that wraps RestTemplate
    @RestService
   IfConfigMeRestClient restClient;
         //Use the Rest Service in a background thread
    @Background
    @AfterInject
    void getAllInfo() {
       all = restClient.getAll();
    }

    //wait a few seconds for the service to finish
   @UiThread(delay = 5000)
   @AfterViews
   void afterViews() {
      resultTextView.setText("Your IP is : "+all.getIpAddr());
   }

}
```

代码量肯定减少了，对吧？

不过你可能注意到了，我们正在使用一个名为`IfConfigMeRestClient`的新类：

```kt
@Rest(rootUrl = "http://ifconfig.me", converters = 
      { MappingJacksonHttpMessageConverter.class })
public interface IfConfigMeRestClient {

   @Get("/all.json")
   @Accept(MediaType.APPLICATION_JSON)
   IfConfigMeJson getAll();

}
```

这个类配置了`RestTemplate`，提供转换器类、使用的 HTTP 动词、映射类（`IfConfigMeJson`）和接受的媒体类型。

Android 注解对 Spring for Android 的`RestTemplate`提供了很好的支持（有关基于`RestTemplate`的注解列表，请参见[`github.com/excilys/androidannotations/wiki/Rest-API`](https://github.com/excilys/androidannotations/wiki/Rest-API)），我建议你尝试一下，看看你是否喜欢在你的 Spring for Android 项目中使用它。

### 注意

如果你从零开始启动一个使用 Android 注解的项目，我建议你仔细遵循这些环境设置说明：[`github.com/excilys/androidannotations/wiki/Building-Project-Maven-Eclipse`](https://github.com/excilys/androidannotations/wiki/Building-Project-Maven-Eclipse)。

# 快速开始 - OAuth

**OAuth**是一个开放的身份验证标准，正如维基百科所说。

OAuth 允许用户将托管在服务提供商上的数据或资源与网站或桌面或移动应用共享。

当然，如果你将你的凭证（用户名和密码）提供给这些网站或应用，它们可以代表你访问你的数据；但你会信任第三方应用或网站来保护你的凭证（你数字生活的钥匙）吗？如果这个应用或网站是恶意的呢？或者仅仅是安全性不足？你有多少次听说某个网站的用户凭证被盗，或者在数据库中以明文形式保存密码？

OAuth 就是关于这一点；它允许第三方应用或网站通过有限的访问（通过一系列授权或范围：`访问用户电子邮件`、`访问用户资料`、`代表用户发布消息`等）来访问你在服务提供商（知名的 OAuth 服务提供商包括 Google、Facebook、Twitter、Yahoo!、GitHub、LinkedIn 等）上托管的数据，而无需向它们提供你的凭据。

## 依赖服务提供商的网站或应用开发

你是否注意过一些网站上的**使用 Facebook 登录**或**使用 Google 登录**按钮（例如 [Deezer.com](http://Deezer.com), [StackOverFlow.com](http://StackOverFlow.com) 等），或者是否曾在 Android 设备上的第三方 Twitter 应用（如 Twicca）请求过你的 Twitter 凭据？

所有这些网站和应用都使用 OAuth 来获取对你的数据的访问权限：

+   首先，它们为你省去了创建和记住其他凭据的麻烦。它们甚至通过访问你的数据（如个人资料图片、活动信息等）来丰富你的用户体验。

+   此外，它们吸引了更广泛的受众（因为用户无需经历繁琐的账户创建过程），并且它们无需管理认证凭据（及其风险）或授权。

通常，如果一个网站或应用创建者想要从一个提供 OAuth 授权的服务中受益，他/她必须向 OAuth 服务提供商（Google、Facebook 等）注册，后者将向他/她提供一个**客户端 ID**和**客户端密钥**值：

![依赖服务提供商的网站或应用开发](img/1905_04_01.jpg)

使用 Google API 控制台注册新应用，并将其与客户端 ID 和客户端密钥关联

让我们探究一下，客户端 ID 和客户端密钥是如何让应用访问用户数据的。

## OAuth 跳舞

一个 Android 应用（或其他任何已安装的应用）依赖于像 Google 这样的 OAuth 2.0 服务提供商。让我们来看看以下五个简单步骤中的 OAuth 授权过程的工作流程：

1.  用户第一次启动应用时，它将生成令牌请求——一个指向服务提供商的 URL，包括应用的客户端 ID 和客户端密钥，以及应用所需的多个授权（用户信息、用户电子邮件等）。

    以 Google OAuth 2.0 服务提供商为例：

    ```kt
    https://accounts.google.com/o/oauth2/auth?
    client_id=508046100884-o6jgcn8e7c1g5gklhc8gibr80ouio8df.apps.googleusercontent.com&
    response_type=code&
    redirect_uri=http://localhost&
    scope=https://www.googleapis.com/auth/userinfo.profile https://www.googleapis.com/auth/userinfo.email https://www.googleapis.com/auth/plus.me
    ```

1.  这个 URL 由 Android Chrome 浏览器加载，很可能是通过 WebView 嵌入在应用中的。用户被要求接受或拒绝应用要求的授权。

1.  作为回报，如果用户授权该应用，应用将会拦截来自嵌入式浏览器的响应，其中包含一个授权码。

    使用 Google OAuth 2.0 服务提供商的响应示例：

    ```kt
    http://localhost/?code=4/urIB_wqrOqGpX-2w1UPXD8dHQAYO.ArEX_6EbNP0ZuJJVnL49Cc98lfsNdgI
    ```

1.  应用最终将用这个授权码交换令牌（如果令牌的有效期有限，应用还将收到一个刷新令牌和过期时间）。这个访问令牌将被应用安全保存。

    使用 Google OAuth 2.0 服务提供商的令牌请求/响应示例。

    **请求**：

    ```kt
    POST /o/oauth2/token HTTP/1.1
    Host: accounts.google.com
    Content-Type: application/x-www-form-urlencoded

    code=4/v6xr77ewYqhvHSyW6UJ1w7jKwAzu&
    client_id=8819981768.apps.googleusercontent.com&
    client_secret={client_secret}&
    redirect_uri=https://oauth2-login-demo.appspot.com/code&
    grant_type=authorization_code
    ```

    **响应**：

    ```kt
    {
      "access_token":"1/fFAGRNJru1FTz70BzhT3Zg",
      "expires_in":3920,
      "token_type":"Bearer",
      "refresh_token":"1/xEoDL4iW3cxlI7yDbSRFYNG01kVKM2C-259HOF2aQbI"
    }
    ```

1.  应用程序最终能够与资源（如 REST 服务）交互，前提是它随每个请求发送有效的令牌。

    使用 Google OAuth 2.0 服务提供商的带令牌的请求示例：

    ```kt
    GET /oauth2/v1/userinfo HTTP/1.1
    Authorization: Bearer 1/fFBGRNJru1FQd44AzqT3Zg
    Host: googleapis.com
    ```

### 注意

如果不是所有的 OAuth 2.0 服务提供商，大多数服务提供商只接受通过 HTTPS 的授权服务调用，以确保令牌不会被截获。

![OAuth 跳舞](img/1905_04_02.jpg)

访问 Google UserInfo 服务的 OAuth 工作流程，来自 [`developers.google.com/accounts/docs/OAuth2`](https://developers.google.com/accounts/docs/OAuth2)

### 注意

你可以阅读 [`developers.google.com/accounts/docs/OAuth2InstalledApp`](https://developers.google.com/accounts/docs/OAuth2InstalledApp) 了解 Google OAuth 2.0 服务提供商的具体信息，或者阅读 [`developers.facebook.com/docs/concepts/login/login-architecture/`](http://developers.facebook.com/docs/concepts/login/login-architecture/) 了解 Facebook OAuth 2.0 服务提供商的具体信息。

强烈建议在开始开发客户端应用程序之前阅读 OAuth 服务提供商的文档（它们都有一些细微的差异，比如请求令牌的参数、回调 URL 等）。

## Spring for Android Auth 模块

Spring for Android 的 `Auth` 模块支持 OAuth 1.0a 和 OAuth 2.0。

简而言之：

+   OAuth 2.0 更好地支持非 Web 应用（如 Android 应用）

+   OAuth 2.0 不再要求开发者使用加密技术

+   OAuth 2.0 访问令牌是*短期的*（得益于刷新令牌）

### 注意

要了解更多关于 OAuth 2.0 的信息，你可以阅读 *Eran Hammer* 在其博客上的几篇文章：[`hueniverse.com/2010/05/introducing-oauth-2-0/`](http://hueniverse.com/2010/05/introducing-oauth-2-0/)

Spring for Android 的 `Auth` 模块依赖于以下内容：

+   Spring for Android 核心模块（公共类：`Base64`、`StringUtils`、资源抽象等）

+   Spring for Android `RestTemplate`（抽象了底层 HttpClient 的 REST API）

+   Spring Social Core（封装 OAuth 1.0 和 2.0 API 的 OAuth 流程：请求授权码、请求令牌等）

+   Spring Security Crypto（提供对称加密、密钥生成和密码编码的支持）

`Auth` 模块本身定义了 Android 的加密和 SQL 类（使你能够将授权令牌安全地持久化到 SQLite Android 数据库中）。

### 注意

目前，Spring for Android 的 `Auth` 模块只包含七个类，这些类主要是关于在 Android 设备上持久化 OAuth 令牌的支持；所有的 OAuth 代码都在 Spring Social Core 中。

为了让 Twitter、Facebook 和 Google 应用程序开发者的生活更轻松，Spring Social 有扩展，分别名为 Spring Social Twitter、Spring Social Facebook 和 Spring Social Google。这些扩展定义了访问每个社交网络中用户信息、帖子、联系人和日程的 API。

## 使用 Google 的 OAuth 示例

我们将要构建一个显示 Google 用户信息的 Android 应用：当然会使用 Spring for Android 的`Auth`，但更重要的是 Spring Social 和 Spring Social Google。

### 注意

官方文档示例是关于 Facebook 和 Twitter 的。如果您想创建一个与这些社交网络互动的应用，请查看 Spring for Android 的示例，地址在[`github.com/SpringSource/spring-android-samples`](https://github.com/SpringSource/spring-android-samples)。

让我们查看一下这个项目的`pom.xml`文件，特别是 Spring Social Google 的包含部分：

```kt
<dependency>
  <groupId>org.springframework.social</groupId>
  <artifactId>spring-social-google</artifactId>
  <version>1.0.0.M1</version>
  <exclusions>
    <!-- Exclude in favor of Spring Android RestTemplate -->
    <exclusion>
      <artifactId>spring-web</artifactId>
      <groupId>org.springframework</groupId>
      </exclusion>
      <exclusion>
      <artifactId>spring-core</artifactId>
      <groupId>org.springframework</groupId>
    </exclusion>
  </exclusions>
</dependency>
[...]
<repository>
  <id>spring.social.google</id>
  <name>Spring Social Google</name>
  <url>https://github.com/GabiAxel/maven/raw/master/</url>
</repository>
```

您会注意到（在`repository`部分），这个模块并没有由 Spring Source 托管，因为它实际上是一个未得到 Spring Source 认可的社区项目。

### 注意

这个`pom.xml`文件包含许多排除项；这是因为大多数使用的库都是以 Java SE 为目标的，这就是为什么它们依赖于 Spring Core、Spring MVC 等。Spring for Android Core 和`RestTemplate`为这些模块提供了必要的依赖。

现在让我们看看`AndroidManifest.xml`文件：

```kt
[...]
<application
    android:name=".MainApplication"
    android:icon="@drawable/app_notes"
    android:label="@string/app_name" >
    <activity
        android:name=".GoogleActivity"
        android:label="@string/title_main" >
        [...]
    </activity>
    <activity
        android:name=".GoogleWebOAuthActivity"
        android:excludeFromRecents="true"
        android:noHistory="true" />
    <activity android:name=".GoogleProfileActivity" />
</application>
```

在我们的示例中，首次将使用一个名为`MainApplication`的`Application`类。

### 注意

`GoogleWebOAuthActivity`将嵌入一个浏览器，并且仅在需要认证时启动。我们不希望这个活动成为应用历史或用户能够返回的部分；这就是为什么我们添加了`android:noHistory="true"`和`android:excludeFromRecents="true"`。关于这方面的更多信息可以在[`developer.android.com/guide/topics/manifest/activity-element.html`](http://developer.android.com/guide/topics/manifest/activity-element.html)找到。

这个类将用于准备我们应用程序中两个最重要的工厂（它们将在所有活动中被访问）：`ConnectionFactoryRegistry`和`ConnectionRepository`：

```kt
public class MainApplication extends Application {
  private ConnectionFactoryRegistry connectionFactoryRegistry;
  private SQLiteOpenHelper repositoryHelper;
  private ConnectionRepository connectionRepository;

  // ***************************************
  // Application Methods
  // ***************************************
  @Override
  public void onCreate() {
    // create a new ConnectionFactoryLocator and populate it with Google ConnectionFactory
    this.connectionFactoryRegistry = new ConnectionFactoryRegistry();
    this.connectionFactoryRegistry.addConnectionFactory(new GoogleConnectionFactory(getClientId(),
        getClientSecret()));

    // set up the database and encryption
    this.repositoryHelper = new SQLiteConnectionRepositoryHelper(this);
    this.connectionRepository = new SQLiteConnectionRepository(this.repositoryHelper,
        this.connectionFactoryRegistry, AndroidEncryptors.text("password", "5c0744940b5c369b"));
  }
```

如您所见，在`onCreate()`方法中我们初始化了：

+   `ConnectionFactoryRegistry`：通过从`ConnectionFactoryRegistry`获取应用程序的客户端 ID 和客户端密钥，我们将能够访问`GoogleConnectionFactory`，它是`OAuth2ConnectionFactory`的 Google 服务扩展，提供了所有 OAuth 操作访问权限

+   `ConnectionRepository`：这将负责持久化`ConnectionFactoryRegistry`，以便可以在不需要每次都执行整个 OAuth 工作流的情况下检索 OAuth 令牌

### 注意

您可能注意到了在初始化数据库时使用了盐值和密码（加密）。

这将防止恶意应用访问设备数据库以获取用户的 OAuth 令牌。简单提醒一下：应用将永远无法访问用户的 Google 密码。对服务提供商（本例中为 Google）的认证始终是从设备浏览器执行的。

让我们看看该项目的 MainActivity，即启动时将启动的`GoogleActivity`：

```kt
@Override
public void onStart() {
  super.onStart();
  if (isConnected()) {
    showGoogleOptions();
  } else {
    showConnectOption();
  }
}

private boolean isConnected() {
  return connectionRepository.findPrimaryConnection(Google.class) != null;
}
```

如果用户已连接，此活动将显示与用户资料相关的条目列表；如果用户尚未连接，则只显示一个**连接**按钮（由于`GoogleConnectionFactoryRegistry`保存在数据库中，只需在`ConnectionRepository`中查找类型为`Google`的连接就足以知道访问令牌是否已获取）。

![使用 Google 的 OAuth 示例](img/1905_04_03.jpg)

如果用户尚未登录的 GoogleActivity

因此，如果我们尚未连接，点击**连接**将调用`displayGoogleAuthorization()`，这将启动`GoogleWebOAuthActivity`。

`GoogleWebOAuthActivity`无疑是这个应用最重要的活动。它负责 OAuth 2.0 的身份验证和授权。

```kt
@Override
public void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);

  //javascript is mandatory
  getWebView().getSettings().setJavaScriptEnabled(true);

  // Using a custom web view client to capture the access token
  getWebView().setWebViewClient(new GoogleOAuthWebViewClient());

  this.connectionRepository = getApplicationContext().getConnectionRepository();
  this.connectionFactory = getApplicationContext().getGoogleConnectionFactory();
}

@Override
public void onStart() {
  super.onStart();

  // display the Google authorization page
  getWebView().loadUrl(getAuthorizeUrl());
}

// ***************************************
// Private methods
// ***************************************
private String getAuthorizeUrl() {
  String redirectUri = getString(R.string.google_oauth_callback_url);
  String scope = getString(R.string.google_scope);

  // Generate the Google authorization url to be used in the browser 
  OAuth2Parameters params = new OAuth2Parameters();
  params.setRedirectUri(redirectUri);
  params.setScope(scope);
  return this.connectionFactory.getOAuthOperations().buildAuthorizeUrl(GrantType.AUTHORIZATION_CODE, params);
}
```

当创建此活动时，它会配置相关的 WebView（你会注意到这个活动扩展了`AbstractWebViewActivity`，它将 Chrome 浏览器实例注入到 WebView 中），以接受 JavaScript（服务提供商和 Google OAuth 2.0 需要使用 JavaScript 来验证用户身份），并注入一个自定义的`WebViewClient`对象，我们将使用它来拦截 OAuth 流程（稍后会详细介绍）。

然后，当活动开始时，我们请求 WebView（嵌入的 Chrome 浏览器）为此应用请求授权码（请参阅*OAuth 舞蹈*部分的第 1 步）。

此请求是使用回调 URL 构建的，应用所需的授权范围，以及客户端 ID 和密钥（这两个在我们创建`ConnectionFactoryRegistry`时已经提供给 Spring OAuth）。

```kt
<resources>
    <string name="google_app_id">508046100884-o6jgcn8e7c1g5gklhc8gibr80ouio8df.apps.googleusercontent.com</string>
    <string name="google_app_secret">RuUyrF5qoGYWTFm1r_o8Gs4F</string>
    <string name="google_oauth_callback_url">http://localhost</string>
    <string name="google_scope">https://www.googleapis.com/auth/userinfo.profile https://www.googleapis.com/auth/userinfo.email https://www.googleapis.com/auth/plus.me</string>
</resources>
```

### 注意

这里的回调 URL 是[`localhost`](http://localhost)，因为 Google OAuth 2.0 服务提供商提供了[`localhost`](http://localhost)和`urn:ietf:wg:oauth:2.0:oob`之间的选择。

这将影响持有授权码的响应；它将是查询字符串参数的一部分，还是浏览器标题栏中的一部分。你可以阅读[`developers.google.com/accounts/docs/OAuth2InstalledApp#choosingredirecturi`](https://developers.google.com/accounts/docs/OAuth2InstalledApp#choosingredirecturi)了解更多详情。

如果用户尚未从设备上对任何 Google 网络服务进行身份验证，他/她应该会看到一个邀请进行身份验证的对话框：

![使用 Google 的 OAuth 示例](img/1905_04_04.jpg)

如果用户尚未在设备上对任何 Google 服务进行身份验证，此对话框才会出现

但在任何情况下，用户都将看到此授权对话框，其中列出了应用请求的所有*作用域*：

![使用 Google 的 OAuth 示例](img/1905_04_05.jpg)

Google OAuth 2.0 服务提供商授权对话框

### 注意

如果用户拒绝授权，那么，如预期的那样，授权过程将被终止。

根据服务提供商的说法，此对话框可能会有所不同。

一旦用户接受请求的授权，`GoogleWebOAuthActivity`将检测到网络客户端被重定向到 localhost（**回调 URI**），并带有授权码：

```kt
private class GoogleOAuthWebViewClient extends WebViewClient {

  private static final String LOCALHOST = "localhost";
  private static final String CODE = "code";

  /*
   * The WebViewClient has another method called shouldOverridUrlLoading which does not capture the javascript 
   * redirect to the success page. So we're using onPageStarted to capture the url.
   */
  @Override
  public void onPageStarted(WebView view, String url, Bitmap favicon) {
    // parse the captured url
    Uri uri = Uri.parse(url);
     // log the url : very interesting for debugging the OAuth workflow
    Log.d(TAG, url);

    /*
     * The WebViewClient is launched to load an URL from the provider that will ask the user whether or not he accepts our app to access his data.
     * Once the provider successfully gets the approval from the user, it will redirect this WebViewClient to the callback_uri, with a query parameter named "code" : this is the authorization code
     */
    String host = uri.getHost();
    String code = uri.getQueryParameter(CODE);

    // The WebViewClient is redirected to the callback_uri, let's trade the authorization code for the access token
    if (LOCALHOST.equals(host)) {
      if(!exchangeAuthorizationCodeForAccessToken.getStatus().equals(AsyncTask.Status.RUNNING)) {
        exchangeAuthorizationCodeForAccessToken.execute(code);
        Toast.makeText(getApplicationContext(), "Redirecting you to the app main activity", Toast.LENGTH_LONG).show();
        //preparing to quit this activity for the main activity
        getWebView().setVisibility(View.INVISIBLE);
      }
    }
  }
}
```

`exchangeAuthorizationCodeForAccessToken.execute(code)`将执行以下 AsyncTask（我们将使用`RestTemplate`，依赖 Java `UrlConnection`，从后台线程发送此调用）：

```kt
private AsyncTask<String, Void, Void> exchangeAuthorizationCodeForAccessToken =  new AsyncTask<String, Void, Void>() {

  private Exception exception;

  @Override
  protected Void doInBackground(String... params) {
    // executed by a background thread
    //params[0] should contain the authorization code
    try {
      AccessGrant exchangeForAccess = connectionFactory.getOAuthOperations().exchangeForAccess(params[0], getString(R.string.google_oauth_callback_url), null);
      Connection<Google> connection = connectionFactory.createConnection(exchangeForAccess);
      connectionRepository.addConnection(connection);
    } catch (DuplicateConnectionException e) {
      Log.e(TAG,"something went wrong when adding the accessToken to the connection repository",e);
      exception = e;
    } catch (Exception e) {
      Log.e(TAG,"something went wrong when adding the accessToken to the connection repository",e);
      exception = e;
    }
    return null;
  }

  @Override
  protected void onPostExecute(Void result) {
    // executed by the UI thread once the background thread is done getting the result
    if(exception != null) {
      Toast.makeText(getApplicationContext(), exception.getMessage(), Toast.LENGTH_LONG).show();
    }
      // we go back to the main activity to display the options
      displayGoogleOptions();
  }

};
```

一旦调用了`exchangeForAccess`方法，我们将获取用户令牌并将其保存在`ConnectionRepository`类中。

我们的应用程序最终被授权访问用户的 Google 个人资料！

![使用 Google 的 OAuth 示例](img/1905_04_06.jpg)

如果用户尚未登录，则为`GoogleActivity`

如果用户点击**个人资料**，他将启动`GoogleProfileActivity`，从中，如你所料，我们可以获取用户资料。

为此，我们使用了一个名为`FetchProfileTask`的 AsyncTask，它将访问两个 Google 网络服务：`UserOperations`（读取用户的主资料和他的个人资料图片）和`PersonOperations`（读取他的 Google+资料，这里我们只访问**关于我**的描述）：

```kt
@Override
protected LegacyGoogleProfile doInBackground(Void... params) {

    LegacyGoogleProfile userProfile = google.userOperations().getUserProfile();
    aboutMe = google.personOperations().getGoogleProfile().getAboutMe();
    profileBitmap = BitmapFactory.decodeStream(new URL(userProfile.getProfilePictureUrl()).openConnection().getInputStream());
    return userProfile;

}
```

然后这些信息被注入到视图中：

![使用 Google 的 OAuth 示例](img/1905_04_07.jpg)

`GoogleProfileActivity`显示用户资料图片，"关于我"的描述和一些个人资料信息

# 你应该了解的人和地方

如果你需要关于 Spring for Android（或一般的 REST 或 OAuth）的帮助，以下是一些人和地方，他们将提供极大的帮助。

## 官方网站

+   主页：[`www.springsource.org/spring-android`](http://www.springsource.org/spring-android)

+   手册和文档：[`static.springsource.org/spring-android/docs/1.0.x/reference/htmlsingle/`](http://static.springsource.org/spring-android/docs/1.0.x/reference/htmlsingle/)

+   博客：[`blog.springsource.org/category/android/`](http://blog.springsource.org/category/android/)

+   源代码：[`github.com/SpringSource/spring-android`](https://github.com/SpringSource/spring-android)

+   官方示例源代码：[`github.com/SpringSource/spring-android-samples`](https://github.com/SpringSource/spring-android-samples)

+   Android Maven 插件：[`code.google.com/p/maven-android-plugin/`](http://code.google.com/p/maven-android-plugin/)

## 文章和教程

+   OAuth 解释：[`hueniverse.com/oauth/guide/`](http://hueniverse.com/oauth/guide/)

+   Google 服务的 OAuth：[`support.google.com/a/bin/answer.py?hl=en&answer=61017`](http://support.google.com/a/bin/answer.py?hl=en&answer=61017)

+   官方 Android 文档关于远程连接：[`developer.android.com/training/basics/network-ops/connecting.html`](http://developer.android.com/training/basics/network-ops/connecting.html)

## 社区

+   官方论坛：[`forum.springsource.org/forumdisplay.php?88-Android`](http://forum.springsource.org/forumdisplay.php?88-Android)

+   官方错误跟踪器：[`jira.springsource.org/browse/ANDROID`](https://jira.springsource.org/browse/ANDROID)

## 博客

+   安卓团队博客：[`android-developers.blogspot.ca/`](http://android-developers.blogspot.ca/)

## 推特

+   在推特上关注 Spring for Android：[`twitter.com/springandroid`](https://twitter.com/springandroid)

+   在推特上关注 Roy Clarkson（Spring for Android 首席开发者）：[`twitter.com/royclarkson`](https://twitter.com/royclarkson)

+   想了解更多开源信息，请关注 Packt：[`twitter.com/#!/packtopensource`](http://twitter.com/#!/packtopensource)
