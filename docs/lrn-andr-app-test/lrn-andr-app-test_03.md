# 第三章：测试方法的应用

本章提供了多个常见情况的实用示例，这些示例应用了前几章描述的纪律和技术。示例以易于跟随的方式呈现，因此你可以调整并用于自己的项目。

本章将涵盖以下主题：

+   安卓单元测试

+   测试活动和应用程序

+   测试数据库和内容提供者

+   测试本地和远程服务

+   测试用户界面

+   测试异常

+   测试解析器

+   测试内存泄漏

+   使用 Espresso 进行测试

在本章之后，你将有一个参考，可以将不同的测试方法应用到你的项目中，以应对不同的情境。

# 安卓单元测试

有些情况下，你确实需要隔离测试应用程序的部分内容，与底层系统的联系很少。在安卓中，系统是活动框架（Activity framework）。在这种情况下，我们必须选择一个在测试层次结构中足够高的基类，以移除一些依赖，但又不能太高，以至于我们需要负责一些基本的基础设施，如实例化上下文（Context），例如。

在这些情况下，候选基类是`AndroidTestCase`，因为这样可以在不考虑活动（Activities）的情况下使用上下文（Context）和资源（Resources）：

```kt
public class AccessPrivateDataTest extends AndroidTestCase {

   public void testAccessAnotherAppsPrivateDataIsNotPossible()  {
        String filesDirectory = getContext().getFilesDir().getPath();
        String privateFilePath = filesDirectory + 
"/data/com.android.cts.appwithdata/private_file.txt";
        try {
            new FileInputStream(privateFilePath);
            fail("Was able to access another app's private data");
        } catch (FileNotFoundException e) {
            // expected
        }
   }
}
```

### 提示

本例基于[Android 兼容性测试套件（CTS）](http://source.android.com/compatibility/cts-intro.html)。CTS 旨在为应用开发者提供一个一致的 Android 硬件和软件环境，无论原始设备制造商如何。

`AccessPrivateDataTest`类扩展了`AndroidTestCase`，因为这是一个不需要系统基础设施的单元测试。在这种情况下，我们不能直接使用`TestCase`，因为我们稍后会用到`getContext()`。

这个测试方法`testAccessAnotherAppsPrivateDataIsNotPossible()`测试了对另一个包私有数据的访问，如果可以访问则测试失败。为此，捕获了预期的异常，如果异常没有发生，则会使用自定义消息调用`fail()`。测试看似简单，但你可以看到这对于防止无意中的安全错误非常有效。

# 测试活动和应用程序

在这里，我们涵盖了一些在日常测试中会遇到的常见情况，包括处理意图（Intents）、偏好设置（Preferences）和上下文（Context）。你可以根据具体需求调整这些模式。

## 模拟应用程序和偏好设置

在安卓术语中，应用程序是指需要维护全局应用状态时使用的基类。完整的包名是`android.app.Application`。在处理共享偏好设置时可以使用。

我们期望那些更改这些偏好设置值的测试不会影响实际应用程序的行为。如果没有正确的测试框架，这些测试可能会删除将偏好值存储为共享偏好的应用程序中的用户账户信息。这听起来可不是个好主意。因此，我们真正需要的是模拟一个`Context`，它同时也能模拟对`SharedPreferences`的访问。

我们最初的尝试可能是使用`RenamingDelegatingContext`，但不幸的是，它并不模拟`SharedPreferences`，尽管它已经很接近了，因为它模拟了数据库和文件系统的访问。所以首先，我们需要模拟对共享偏好的访问。

### 提示

每当遇到一个新类（如`RenamingDelegatingContext`）时，阅读相关的 Java 文档以了解框架开发者期望如何使用它是个不错的主意。更多信息，请参考[`developer.android.com/reference/android/test/RenamingDelegatingContext.html`](http://developer.android.com/reference/android/test/RenamingDelegatingContext.html)。

### RenamingMockContext 类

让我们创建一个专门的`Context`。`RenamingDelegatingContext`类是一个很好的起点，因为我们之前提到过，数据库和文件系统的访问将被模拟。问题是怎样模拟对`SharedPreferences`的访问。

请记住，`RenamingDelegatingContext`，顾名思义，将所有操作委托给一个`Context`。所以我们的问题的根源就在这个`Context`中。当从`Context`访问`SharedPreferences`时，你会使用`getSharedPreferences(String name, int mode)`。为了改变这个方法的工作方式，我们可以在`RenamingMockContext`内部重写它。现在我们有了控制权，我们可以用我们的测试前缀来添加名称参数，这意味着当我们的测试运行时，它们将写入与主应用程序不同的偏好设置文件：

```kt
public class RenamingMockContext extends RenamingDelegatingContext {

    private static final String PREFIX = "test.";

    public RenamingMockContext(Context context) {
        super(context, PREFIX);
    }

    @Override
    public SharedPreferences getSharedPreferences(String name, int mode) {
        return super.getSharedPreferences(PREFIX + name, mode);
    }
}
```

现在，我们可以完全控制偏好设置、数据库和文件的存储方式。

### 模拟上下文

我们有`RenamingMockContext`这个类。现在，我们需要一个使用它的测试。由于我们将要测试应用程序，测试的基类将是`ApplicationTestCase`。这个测试用例提供了一个框架，你可以在这个框架中在一个受控环境中测试应用程序类。它为应用程序的生命周期提供基本支持，以及钩子来注入各种依赖并控制应用程序测试的环境。使用`setContext()`方法，我们可以在创建应用程序之前注入`RenamingMockContext`。

我们将要测试一个名为`TemperatureConverter`的应用程序。这是一个简单的应用程序，用于将摄氏度转换为华氏度，反之亦然。我们将在第六章，*实践测试驱动开发*中讨论更多关于这个应用程序的开发。现在，这些细节不是必需的，因为我们专注于测试场景。`TemperatureConverter`应用程序将把任何转换的小数位数存储为共享偏好设置。因此，我们将创建一个测试来设置小数位数，然后检索它以验证其值：

```kt
public class TemperatureConverterApplicationTests extends ApplicationTestCase<TemperatureConverterApplication> {

    public TemperatureConverterApplicationTests() {
        this("TemperatureConverterApplicationTests");
    }

    public TemperatureConverterApplicationTests(String name) {
        super(TemperatureConverterApplication.class);
        setName(name);
    }

    public void testSetAndRetreiveDecimalPlaces() {
        RenamingMockContext mockContext = new RenamingMockContext(getContext());
        setContext(mockContext);
        createApplication();
        TemperatureConverterApplication application = getApplication();

        application.setDecimalPlaces(3);

        assertEquals(3, application.getDecimalPlaces());
    }
}
```

我们使用`TemperatureConverterApplication`模板参数扩展了`ApplicationTestCase`。

然后，我们使用了在第二章中讨论的给定名称构造函数模式，*了解使用 Android SDK 的测试*。

在这里，我们没有使用`setUp()`方法，因为类中只有一个测试——正如他们所说，*你不会需要它*。有一天，如果你要向这个类添加另一个测试，这时你可以重写`setUp()`并移动行为。这遵循了 DRY 原则，即不要重复自己，这会导致软件更易于维护。因此，在测试方法顶部，我们创建模拟上下文并使用`setContext()`方法为此测试设置上下文；我们使用`createApplication()`创建应用程序。你需要确保在`createApplication`之前调用`setContext`，因为这是你获得正确实例化顺序的方式。现在，实际测试所需行为的代码设置小数位数，检索它，并验证其值。就是这样，使用`RenamingMockContext`让我们控制`SharedPreferences`。每当请求`SharedPreference`时，该方法将调用委派上下文，为名称添加前缀。应用程序使用的原始`SharedPreferences`类保持不变：

```kt
public class TemperatureConverterApplication extends Application {
    private static final int DECIMAL_PLACES_DEFAULT = 2;
    private static final String KEY_DECIMAL_PLACES = ".KEY_DECIMAL_PLACES";

    private SharedPreferences sharedPreferences;

    @Override
    public void onCreate() {
        super.onCreate();
        sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);
    }

    public void setDecimalPlaces(int places) {
        Editor editor = sharedPreferences.edit();
        editor.putInt(KEY_DECIMAL_PLACES, places);
        editor.apply();
    }

    public int getDecimalPlaces() {
        return sharedPreferences.getInt(KEY_DECIMAL_PLACES, DECIMAL_PLACES_DEFAULT);
    }
}
```

我们可以通过为`TemperatureConverterApplication`类提供一些共享偏好设置中的值，运行应用程序，然后执行测试，并最终验证执行测试后该值未受影响，以确保我们的测试不会影响应用程序。

## 测试活动（Testing activities）

下一个示例展示了如何使用`ActivityUnitTestCase<Activity>`基类完全独立地测试活动。第二个选择是`ActivityInstrumentationTestCase2<Activity>`。然而，前者允许你创建一个活动但不将其附加到系统，这意味着你不能启动其他活动（你是一个活动的单一单元）。这种父类的选择不仅要求你在设置时更加小心注意，同时也为被测试的活动提供了更大的灵活性和控制。这种测试旨在测试一般的活动行为，而不是活动实例与系统其他组件的交互或任何与 UI 相关的测试。

首先要明确，下面是被测试的类。这是一个带有一个按钮的简单活动。当按下此按钮时，它会触发一个意图来启动拨号器并结束自己：

```kt
public class ForwardingActivity extends Activity {
    private static final int GHOSTBUSTERS = 999121212;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_forwarding);
        View button = findViewById(R.id.forwarding_go_button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent("tel:" + GHOSTBUSTERS);
                startActivity(intent);
                finish();
            }
        });
    }
}
```

对于我们的测试`case`，我们扩展了`ActivityUnitTestCase<ForwardingActivity>`，正如我们之前提到的，作为一个`Activity`类的单元测试。这个被测试的活动将脱离系统，因此它仅用于测试其内部方面，而不是与其他组件的交互。在`setUp()`方法中，我们创建了一个意图，用于启动我们被测试的活动，即`ForwardingActivity`。注意`getInstrumentation()`的使用。此时在`setUp()`方法中的活动上下文`getContext`类仍然是 null：

```kt
public class ForwardingActivityTest extends ActivityUnitTestCase<ForwardingActivity> {
    private Intent startIntent;

    public ForwardingActivityTest() {
        super(ForwardingActivity.class);
    }

    @Override
    protected void setUp() throws Exception {
        super.setUp();
        Context context = getInstrumentation().getContext();
        startIntent = new Intent(context, ForwardingActivity.class);
    }
```

现在设置完成了，我们可以继续进行我们的测试：

```kt
public void testLaunchingSubActivityFiresIntentAndFinishesSelf() {
Activity activity = startActivity(startIntent, null, null);
View button = activity.findViewById(R.id.forwarding_go_button);

button.performClick();

assertNotNull(getStartedActivityIntent());
assertTrue(isFinishCalled());
}
```

第一个测试对 Forwarding 活动的**Go**按钮进行点击。该按钮的`onClickListener`类调用`startActivity()`，并带有一个定义了将要启动的新`Activity`的意图。执行此操作后，我们验证用于启动新活动的`Intent`不为 null。`getStartedActivityIntent()`方法返回了如果被测试的活动调用了`startActivity(Intent)`或`startActivityForResult(Intent, int)`所使用的意图。接下来，我们断言`finish()`被调用，通过验证`FinishCalled()`的返回值来做到这一点，如果被测试活动中的`finish`方法之一（`finish()`、`finishFromChild(Activity)`或`finishActivity(int)`）被调用，它将返回`true`：

```kt
public void testExampleOfLifeCycleCreation() {
  Activity activity = startActivity(startIntent, null, null);

  // At this point, onCreate() has been called, but nothing else
  // so we complete the startup of the activity
  getInstrumentation().callActivityOnStart(activity);
  getInstrumentation().callActivityOnResume(activity);

  // At this point you could test for various configuration aspects
  // or you could use a Mock Context 
  // to confirm that your activity has made
  // certain calls to the system and set itself up properly.

  getInstrumentation().callActivityOnPause(activity);

  // At this point you could confirm that 
  // the activity has paused properly,
  // as if it is no longer the topmost activity on screen.

    getInstrumentation().callActivityOnStop(activity);

  // At this point, you could confirm that 
  // the activity has shut itself down appropriately,
  // or you could use a Mock Context to confirm that 
  // your activity has released any
  // system resources it should no longer be holding.

  // ActivityUnitTestCase.tearDown() is always automatically called
  // and will take care of calling onDestroy().
 }
```

第二个测试可能是这个测试案例中更有趣的测试方法。这个测试案例演示了如何执行活动生命周期。启动活动后，`onCreate()`会自动调用，然后我们可以通过手动调用其他生命周期方法来进行测试。为了能够调用这些方法，我们使用了这个测试的`Intrumentation`。同时，我们不手动调用`onDestroy()`，因为`tearDown()`会为我们调用它。

让我们逐步了解代码。这个方法以之前分析过的测试相同的方式启动 Activity。Activity 启动后，系统会自动调用其 `onCreate()` 方法。然后我们使用 `Instrumentation` 来调用其他生命周期方法，以完成被测试 Activity 的启动。这些对应于 Activity 生命周期中的 `onStart()` 和 `onResume()`。

现在 Activity 已经完全启动，是时候测试我们感兴趣的那些方面了。一旦完成，我们可以按照生命周期的其他步骤进行。请注意，这个示例测试在这里并没有断言任何内容，只是指出了如何逐步执行生命周期。为了完成生命周期，我们调用了 `onPause()` 和 `onStop()`。我们知道，`onDestroy()` 会被 `tearDown()` 自动调用，因此避免了它。

这个测试代表了一个测试框架。你可以用它来隔离测试你的 Activities，以及测试与生命周期相关的案例。注入模拟对象还可以方便地测试 Activity 的其他方面，比如访问系统资源。

# 测试文件、数据库和内容提供者

一些测试案例需要执行数据库或 `ContentProvider` 操作，很快就需要模拟这些操作。例如，如果我们正在实机上测试一个应用程序，我们不想干扰该设备上应用程序的正常运行，尤其是如果我们更改可能被多个应用程序共享的值。

这些案例可以利用另一个不属于 `android.test.mock` 包，而是属于 `android.test` 的模拟类，即 `RenamingDelegatingContext`。

请记住，这个类允许我们模拟文件和数据库操作。在构造函数中提供的缀会在修改这些操作的目标时使用。所有其他操作都委托给你指定的委托上下文。

假设我们正在测试的 Activity 使用了一些我们希望在某种方式下控制的文件或数据库，可能是为了引入特殊内容来驱动我们的测试，而我们不想或不能使用真实的文件或数据库。在这种情况下，我们创建一个指定前缀的 `RenamingDelegatingContext`。我们使用这个前缀提供模拟文件，并引入我们需要驱动测试的任何内容，被测试的 Activity 可以毫无修改地使用它们。

保持我们的 Activity 不变（即不修改它以从不同的来源读取数据）的优势在于，这样可以确保所有测试的有效性。如果我们引入了一个只为测试而设计的改变，我们将无法确保在实际条件下，Activity 的行为是相同的。

为了说明这个情况，我们将创建一个极其简单的 Activity。

`MockContextExampleActivity` 活动在 `TextView` 中显示文件的内容。我们想要演示的是，在 Activity 正常运行时与处于测试状态时，它如何显示不同的内容：

```kt
public class MockContextExampleActivity extends Activity {
    private static final String FILE_NAME = "my_file.txt";

    private TextView textView;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_mock_context_example);

        textView = (TextView) findViewById(R.id.mock_text_view);
        try {
            FileInputStream fis = openFileInput(FILE_NAME);
            textView.setText(convertStreamToString(fis));
        } catch (FileNotFoundException e) {
            textView.setText("File not found");
        }
    }

    private String convertStreamToString(java.io.InputStream is) {
   Scanner s = new Scanner(is, "UTF-8").useDelimiter("\\A");
       return s.hasNext() ? s.next() : "";
    }

    public String getText() {
        return textView.getText().toString();
    }
}
```

这是我们的简单活动。它读取 `my_file.txt` 文件的内容并将其显示在 `TextView` 上。它还会显示可能发生的任何错误。显然，在真实场景中，你会比这有更好的错误处理。

我们需要为此文件准备一些内容。创建文件最简单的方法可能如下面的代码所示：

```kt
$ adb shell 
$ echo "This is real data" > data/data/com.blundell.tut/files/my_file.txt

$ echo "This is *MOCK* data" > /data/data/com.blundell.tut/files/test.my_file.txt

```

我们创建了两个不同的文件，一个名为 `my_file.txt`，另一个名为 `test.my_file.txt`，内容不同。后者表示它是一个模拟内容。如果你现在运行前面的活动，你会看到**这是真实数据**，因为它是从预期的文件 `my_file.txt` 中读取的。

下面的代码演示了在我们的活动测试中使用这个模拟数据：

```kt
public class MockContextExampleTest 
extends ActivityUnitTestCase<MockContextExampleActivity> {

private static final String PREFIX = "test.";
private RenamingDelegatingContext mockContext;

public MockContextExampleTest() {
super(MockContextExampleActivity.class);
}

@Override
protected void setUp() throws Exception {
super.setUp();
mockContext = new RenamingDelegatingContext(getInstrumentation().getTargetContext(), PREFIX);
mockContext.makeExistingFilesAndDbsAccessible();
}

public void testSampleTextDisplayed() {
setActivityContext(mockContext);

   startActivity(new Intent(), null, null);

assertEquals("This is *MOCK* data\n", getActivity().getText());
}
}
```

`MockContextExampleTest` 类扩展了 `ActivityUnitTestCase`，因为我们要对 `MockContextExampleActivity` 进行隔离测试，并且我们将注入一个模拟上下文；在这种情况下，注入的上下文是作为依赖的 `RenamingDelegatingContext`。

我们的夹具包括模拟上下文 `mockContext` 和 `RenamingDelegatingContext`，使用通过 `getInstrumentation().getTargetContext()` 获取的目标上下文。请注意，运行仪器化的上下文与被测试活动的上下文是不同的。

这里有一个基本步骤——由于我们希望让现有的文件和数据库可供这个测试使用，因此我们必须调用 `makeExistingFilesAndDbsAccessible()`。

然后，我们的名为 `testSampleTextDisplayed()` 的测试通过使用 `setActivityContext()` 注入模拟上下文。

### 提示

在调用 `startActivity()` 启动被测活动之前，你必须调用 `setActivityContext()` 来注入一个模拟上下文。

然后，通过使用刚刚创建的空白意图调用 `startActivity()` 启动活动。

我们通过使用我们添加到活动中的 getter 来获取 `TextView` 中持有的文本值。我绝不建议在真实项目中仅仅为了测试而改变生产代码（即暴露 getter），因为这可能导致错误、其他开发者的错误使用模式和安全问题。然而，这里，我们是在展示使用 `RenamingDelegatingContext` 而不是测试正确性。

最后，获取的文本值与字符串 `This is MOCK* data` 进行了对比。这里需要注意的是，用于此测试的值是测试文件内容，而不是真实文件内容。

## 浏览器提供者测试

这些测试基于 Android 开源项目（AOSP）的浏览器模块。AOSP 有很多很好的测试示例，使用它们作为这里的例子可以让你不必编写大量用于设置测试场景的样板代码。它们旨在测试浏览器书签的一部分方面，内容提供者，这是 Android 平台（不是 Chrome 应用，而是默认的浏览器应用）包含的标准浏览器的一部分：

```kt
public class BrowserProviderTests extends AndroidTestCase {
    private List<Uri> deleteUris;

    @Override
    protected void setUp() throws Exception {
       super.setUp();
        deleteUris = new ArrayList<Uri>();
    }

    @Override
    protected void tearDown() throws Exception {
        for (Uri uri : deleteUris) {
            deleteUri(uri);
        }
        super.tearDown();
    }
}
```

### 注意

```kt
AndroidTestCase. The BrowserProviderTests class extends AndroidTestCase because a Context is needed to access the provider content.
```

在`setUp()`方法中创建的夹具创建了一个`Uris`列表，用于跟踪每个测试在`tearDown()`方法结束时需要删除的插入的`Uris`。开发者本可以使用一个模拟内容提供者来避免这个麻烦，以保持测试与系统的隔离。无论如何，`tearDown()`方法遍历这个列表并删除存储的`Uris`。这里不需要重写构造函数，因为`AndroidTestCase`不是一个参数化类，我们也不需要在其中进行特殊操作。

现在是测试时间：

```kt
public void testHasDefaultBookmarks() {
  Cursor c = getBookmarksSuggest("");
  try {
    assertTrue("No default bookmarks", c.getCount() > 0);
  } finally {
    c.close();
  }
}
```

`testHasDefaultBookmarks()`方法是一个测试，用于确保数据库中始终存在一些默认书签。启动时，游标遍历通过调用`getBookmarksSuggest("")`获得的默认书签，这返回一个未经过滤的书签游标；这就是内容提供者查询参数为`""`的原因：

```kt
public void testPartialFirstTitleWord() {
   assertInsertQuery(
"http://www.example.com/rasdfe", "nfgjra sdfywe", "nfgj");
}
```

`testPartialFirstTitleWord()`方法以及其他三个类似的方法（这里未显示的`testFullFirstTitleWord()`、`testFullFirstTitleWordPartialSecond()`和`testFullTitle()`）测试书签的插入。为此，它们使用书签的 URL、标题和查询调用`assertInsertQuery()`。`assertInsertQuery()`方法将书签添加到书签提供者中，插入作为参数给出的指定标题的 URL。返回的`Uri`被验证不为空且不完全是默认的。最后，`Uri`被插入到`tearDown()`中要删除的`Uri`实例列表中。以下代码可以在显示的实用方法中看到：

```kt
public void testFullTitleJapanese() {
String title = "\u30ae\u30e3\u30e9\u30ea\u30fc\u30fcGoogle\u691c\u7d22";
assertInsertQuery("http://www.example.com/sdaga", title, title);
}
```

### 注意

Unicode 是一个计算行业标准，旨在一致且唯一地编码全世界书面语言中使用的字符。Unicode 标准使用十六进制来表示一个字符。例如，值\u30ae 表示片假名字母 GI（ギ）。

我们有多个测试旨在验证此书签提供者对于除了英语之外的其他地区和语言的利用情况。这些特定案例涵盖了书签标题中日语的使用情况。测试`testFullTitleJapanese()`以及这里未显示的其他两个测试，即`testPartialTitleJapanese()`和`testSoundmarkTitleJapanese()`是之前使用 Unicode 字符引入的测试的日语版本。建议在不同的条件下测试应用程序的组件，就像在这种情况下，使用具有不同字符集的其他语言。

接下来有几个实用方法。这些是在测试中使用的工具。我们之前简要介绍了`assertInsertQuery()`，现在让我们看看其他方法：

```kt
private void assertInsertQuery(String url, String title, String query) {
        addBookmark(url, title);
        assertQueryReturns(url, title, query);
    }
    private void addBookmark(String url, String title) {
        Uri uri = insertBookmark(url, title);
        assertNotNull(uri);
        assertFalse(BOOKMARKS_URI.equals(uri));
        deleteUris.add(uri);
    }
    private Uri insertBookmark(String url, String title) {
        ContentValues values = new ContentValues();
        values.put("title", title);
        values.put("url", url);
        values.put("visits", 0);
        values.put("date", 0);
        values.put("created", 0);
        values.put("bookmark", 1);
        return getContext().getContentResolver().insert(BOOKMARKS_URI, values);
    }

private void assertQueryReturns(String url, String title, String query) {
  Cursor c = getBookmarksSuggest(query);
  try {
    assertTrue(title + " not matched by " + query, c.getCount() > 0);
    assertTrue("More than one result for " + query, c.getCount() == 1);
    while (c.moveToNext()) {
      String text1 = getCol(c, SearchManager.SUGGEST_COLUMN_TEXT_1);
      assertNotNull(text1);
      assertEquals("Bad title", title, text1);
      String text2 = getCol(c, SearchManager.SUGGEST_COLUMN_TEXT_2);
      assertNotNull(text2);
      String data = getCol(c, SearchManager.SUGGEST_COLUMN_INTENT_DATA);
      assertNotNull(data);
      assertEquals("Bad URL", url, data);
    }
  } finally {
    c.close();
  }
}

private String getCol(Cursor c, String name) {
  int col = c.getColumnIndex(name);
  String msg = "Column " + name + " not found, " 
               + "columns: " + Arrays.toString(c.getColumnNames());
  assertTrue(msg, col >= 0);
  return c.getString(col);
}

private Cursor getBookmarksSuggest(String query) {
  Uri suggestUri = Uri.parse("content://browser/bookmarks/search_suggest_query");
  String[] selectionArgs = {query};
  Cursor c = getContext().getContentResolver().query(suggestUri, null, "url LIKE ?", selectionArgs, null);
  assertNotNull(c);
  return c;
}

private void deleteUri(Uri uri) {
  int count = getContext().getContentResolver().delete(uri, null, null);
  assertEquals("Failed to delete " + uri, 1, count);
}
```

`assertInsertQuery()`方法在`addBookmark()`之后调用`assertQueryReturns(url`、`title`和`query)`，以验证`getBookmarksSuggest(query)`返回的游标是否包含预期的数据。这个期望可以概括为：

+   查询返回的行数大于 0

+   查询返回的行数等于 1

+   返回行中的标题不为空

+   查询返回的标题与方法的参数完全相同

+   对于建议的第二行不为空

+   查询返回的 URL 不为空

+   这个 URL 与作为方法参数发出的 URL 完全匹配

这种策略为我们的测试提供了一个有趣的模式。我们需要创建的一些实用方法来完成我们的测试，也可以自行验证多个条件，提高我们的测试质量。

在我们的类中创建断言方法，可以引入一种特定领域的测试语言，当测试系统的其他部分时可以重复使用。

# 测试异常

我们在第一章中提到过*开始测试*，我们指出你应该测试异常和错误值，而不仅仅是测试正面情况：

```kt
@Test(expected = InvalidTemperatureException.class)
public final void testExceptionForLessThanAbsoluteZeroF() {
 TemperatureConverter.
fahrenheitToCelsius(TemperatureConverter.ABSOLUTE_ZERO_F - 1);
}

@Test(expected = InvalidTemperatureException.class)
public final void testExceptionForLessThanAbsoluteZeroC() {
  TemperatureConverter.
celsiusToFahrenheit(TemperatureConverter.ABSOLUTE_ZERO_C - 1);
}
```

我们之前已经介绍过这些测试，但在这里，我们将更深入地探讨它。首先要注意的是，这些是 JUnit4 测试，意味着我们可以使用`expected`注解参数测试异常。当你下载本章的示例项目时，你会看到它被分为两个模块，其中一个是核心模块，它是一个纯 Java 模块，因此我们有使用 JUnit4 的机会。在撰写本文时，Android 已经宣布支持 JUnit4，但尚未发布，因此对于 Android 的仪器测试，我们仍然使用 JUnit3。

每当我们有一个应该生成异常的方法时，我们都应该测试这种异常情况。最佳的做法是使用 JUnit4 的`expected`参数。这声明测试应该抛出异常，如果没有抛出异常或抛出不同的异常，测试将失败。在 JUnit3 中也可以通过在 try-catch 块中调用测试方法，捕获预期的异常，否则失败：

```kt
    public void testExceptionForLessThanAbsoluteZeroC() {
        try {
          TemperatureConverter.celsiusToFahrenheit(ABSOLUTE_ZERO_C - 1);
          fail();
        } catch (InvalidTemperatureException ex) {
          // do nothing we expect this exception!
        }
    }
```

# 测试本地和远程服务

当你想测试一个`android.app.Service`时，想法是扩展`ServiceTestCase<Service>`类，在受控环境中进行测试：

```kt
public class DummyServiceTest extends ServiceTestCase<DummyService> {
    public DummyServiceTest() {
        super(DummyService.class);
    }

    public void testBasicStartup() {
        Intent startIntent = new Intent();
        startIntent.setClass(getContext(), DummyService.class);
        startService(startIntent);
    }

    public void testBindable() {
        Intent startIntent = new Intent();
        startIntent.setClass(getContext(), DummyService.class);
        bindService(startIntent);
    }
}
```

构造函数，像其他类似的情况一样，调用父构造函数，将 Android 服务类作为参数传递。

这之后是`testBasicStartup()`。我们使用一个 Intent 启动服务，在这里创建它，将其类设置为正在测试的服务类。我们还为这个 Intent 使用仪器化上下文。这个类允许一些依赖注入，因为每个服务都依赖于其运行的上下文以及与之关联的应用程序。这个框架允许你注入修改过的、模拟的或独立的依赖替代品，从而执行真正的单元测试。

### 注意

**依赖注入**（**DI**）是一种软件设计模式，涉及组件如何获取其依赖关系。你可以手动完成这一操作，或者使用众多依赖注入库中的一个。

由于我们只是按原样运行测试，服务将被注入一个功能完整的`Context`和一个通用的`MockApplication`对象。然后，我们使用`startService(startIntent)`方法启动服务，这与通过`Context.startService()`启动服务的方式相同，并提供它所提供的参数。如果你使用此方法启动服务，它将自动由`tearDown()`停止。

另一个测试`testBindable()`，将测试服务是否可以被绑定。这个测试使用`bindService(startIntent)`，它以与通过`Context.bindService()`启动服务相同的方式启动正在测试的服务，并提供它所提供的参数。它返回与服务通信的通道。如果客户端无法绑定到服务，它可能返回 null。这个测试应该用类似`assertNotNull(service)`的断言检查服务中的 null 返回值，以验证服务是否正确绑定，但实际没有这样做，因此我们可以专注于使用的框架类。在编写类似情况的代码时，请务必包含此测试。

返回的`IBinder`通常是一个使用 AIDL 描述的复杂接口。为了测试这个接口，你的服务必须实现一个`getService()`方法，如本章示例项目中的`DummService`所示；该方法有以下实现：

```kt
    public class LocalBinder extends Binder {
        DummyService getService() {
            return DummyService.this;
        }
    }
```

# 广泛使用模拟对象

在前面的章节中，我们描述并使用了 Android SDK 中存在的模拟类。尽管这些类可以覆盖很多情况，但也有其他 Android 类和你的领域类需要考虑。你可能需要其他模拟对象来丰富你的测试用例。

几个库提供了满足我们模拟需求的基础设施，但现在我们专注于 Mockito，这可能是 Android 中使用最广泛的库。

### 注意

这不是一个 Mockito 教程。我们只是分析它在 Android 中的使用，所以如果你不熟悉它，我建议你查看其网站上的文档，网址为[`code.google.com/p/mockito/`](https://code.google.com/p/mockito/)。

Mockito 是一个开源软件项目，可在 MIT 许可下使用，并提供测试替身（模拟对象）。由于其验证期望的方式和动态生成的模拟对象，它非常适合测试驱动开发，因为它们支持重构，且在重命名方法或更改其签名时，测试代码不会断裂。

概括其文档，Mockito 最相关的优势如下：

+   执行后询问交互问题

+   它不是期望-运行-验证——避免昂贵的设置

+   一种模拟简单 API 的方法

+   使用类型进行简单的重构

+   它可以模拟具体类以及接口

为了演示其用法，并确立一种稍后可以用于其他测试的风格，我们正在完成一些示例测试用例。

### 注意

在本文撰写时，Android 支持的最新版 Mockito 是 Dexmaker Mockito 1.1。你可能想尝试其他版本，但很可能会遇到问题。

我们首先应该做的是将`Mockito`作为依赖添加到你的 Android 仪器测试中。这只需简单地在你的依赖闭包中添加`androidTestCompile`引用。Gradle 会完成剩下的工作，即下载 JAR 文件并将其添加到你的类路径中：

```kt
dependencies {
    // other compile dependencies

    androidTestCompile('com.google.dexmaker:dexmaker-mockito:1.1')
}
```

为了在我们的测试中使用 Mockito，我们只需要从`org.mockito`静态导入其方法。通常，你的 IDE 会给你静态导入这些选项，但如果它没有，你可以尝试手动添加（如果手动添加时代码变红，那么你遇到的问题就是库不可用的问题）。

```kt
  import static org.mockito.Matchers.*;
import static org.mockito.Mockito.*;
```

最好使用特定的导入，而不是使用通配符。这里使用通配符只是为了简洁。很可能当你的 IDE 自动保存时，它会将它们扩展为所需的导入（或者如果你没有使用它们，就会移除它们！）。

## 导入库

我们已经将 Mockito 库添加到了项目的 Java 构建路径中。通常这不会有问题，但有时，重新构建项目会导致以下错误，阻止项目构建：**错误：在 APK 打包期间文件重复**。

这取决于项目中包含了多少库以及它们是什么。

大多数可用的开源库都包含类似 GNU 提议的内容，并包含如`LICENSE`、`NOTICE`、`CHANGES`、`COPYRIGHT`和`INSTALL`等文件。当我们尝试在同一个项目中包含多个库以最终构建一个单一的 APK 时，我们会立即遇到这个问题。你可以在你的`build.gradle`中解决这个问题：

```kt
    packagingOptions {
        exclude 'META-INF/LICENSE'
        exclude 'folder/duplicatedFileName'
  }
```

## Mockito 使用示例

让我们创建一个只接受有符号十进制数的`EditText`，我们将它称为`EditNumber`。`EditNumber`使用`InputFilter`提供此功能。在以下测试中，我们将执行此过滤器来验证是否实现了正确的行为。

为了创建测试，我们将使用`EditNumber`从`EditText`继承的一个属性，这样它就可以添加一个监听器，实际上是一个`TextWatcher`。这将提供当`EditNumber`的文本发生变化时调用的方法。这个`TextWatcher`是测试的协助者，我们可以将其实现为单独的类，并验证调用其方法的结果，但这样做既繁琐，可能会引入更多错误，所以我们采用的方法是使用 Mockito，以避免编写外部的`TextWatcher`。

这正是我们引入一个模拟的`TextWatcher`来检查文本变化时方法调用的方式。

## EditNumber 过滤器测试

这个测试套件将执行`EditNumber`的`InputFilter`行为，检查`TextWatcher`模拟上的方法调用并验证结果。

我们使用`AndroidTestCase`，因为我们希望独立于其他组件或活动来测试`EditNumber`。

我们有多个需要测试的输入（我们允许小数，但不允许多个小数点、字母等），因此我们可以有一个带有预期输入数组和预期输出数组的测试。然而，测试可能会变得非常复杂，难以维护。更好的方法是针对`InputFilter`的每个测试用例都有一个测试。这允许我们给测试赋予有意义的名称，并解释我们旨在测试的内容。我们将以如下列表结束：

```kt
testTextChangedFilter*
        * WorksForBlankInput
        * WorksForSingleDigitInput
        * WorksForMultipleDigitInput
        * WorksForZeroInput
        * WorksForDecimalInput
        * WorksForNegativeInput
        * WorksForDashedInput
        * WorksForPositiveInput
        * WorksForCharacterInput
        * WorksForDoubleDecimalInput
```

现在，我们将通过一个测试`testTextChangedFilterWorksForCharacterInput()`来介绍模拟对象的使用，如果你查看示例项目，你会发现所有其他测试都遵循相同的模式，实际上我们已经提取了一个帮助方法，该方法作为所有测试的自定义断言：

```kt
public void testTextChangedFilterWorksForCharacterInput() {
  assertEditNumberTextChangeFilter("A1A", "1");
}
/**
 * @param input  the text to be filtered 
 * @param output the result you expect once the input has been filtered
*/
private void assertEditNumberTextChangeFilter(String input, String output) {
 int lengthAfter = output.length();
 TextWatcher mockTextWatcher = mock(TextWatcher.class);
 editNumber.addTextChangedListener(mockTextWatcher);

 editNumber.setText(input);

 verify(mockTextWatcher)
.afterTextChanged(editableCharSequenceEq(output));
 verify(mockTextWatcher)
.onTextChanged(charSequenceEq(output), eq(0), eq(0), eq(lengthAfter));
 verify(mockTextWatcher)
.beforeTextChanged(charSequenceEq(""), eq(0), eq(0), eq(lengthAfter));
}
```

如你所见，测试用例非常直接；它断言当你将`A1A`输入到**EditNumber**视图的文本中时，文本实际上被更改为`1`。这意味着我们的 EditNumber 已经过滤掉了字符。当我们查看`assertEditNumberTextChangeFilter(input, output)`帮助方法时，会发生一件有趣的事情。在我们的帮助方法中，我们验证了`InputFilter`是否正在执行其工作，这里我们使用了 Mockito。使用 Mockito 模拟对象时有四个常见步骤：

1.  实例化准备好的模拟对象。

1.  确定预期的行为并将其存根以返回任何固定数据。

1.  调用方法，通常是通过调用测试类的各个方法。

1.  验证模拟对象的行为以通过测试。

根据第一步，我们使用`mock(TextWatcher.class)`创建一个模拟的`TextWatcher`，并将其设置为 EditNumber 上的`TextChangedListener`。

在这个实例中，我们跳过第二步，因为没有固定数据，即我们模拟的类没有任何预期返回值的方法。稍后我们在另一个测试中会回到这一点。

在第三步中，我们已经设置好了模拟对象，可以执行测试方法以执行其预期操作。在我们的案例中，方法是`editNumber.setText(input)`，预期操作是设置文本，从而触发我们的`InputFilter`运行。

第四步是验证文本是否确实被我们的过滤器更改。让我们稍微分解一下第四步。以下是我们再次的验证：

```kt
verify(mockTextWatcher)
.afterTextChanged(editableCharSequenceEq(output));
verify(mockTextWatcher)
.onTextChanged(charSequenceEq(output), eq(0), eq(0), eq(lengthAfter));
verify(mockTextWatcher)
.beforeTextChanged(charSequenceEq(""), eq(0), eq(0), eq(lengthAfter));
```

我们将使用两个自定义匹配器（`editableCharSequenceEq(String)` 和 `charSequenceEq(String)`），因为我们关心的是比较 Android 使用的不同类（如 `Editable` 和 `CharSequence`）的字符串内容。当你使用一个特殊的匹配器时，这意味着对该验证方法调用的所有比较都需要一个特殊的包装方法。

另一个匹配器 `eq()`，期望得到一个等于给定值的 `int`。后者由 Mockito 为所有原始类型和对象提供，但我们需要实现 `editableCharSequenceEq()` 和 `charSequenceEq()`，因为这是一个针对 Android 的特定匹配器。

Mockito 有一个预定义的 `ArgumentMatcher`，可以帮助我们创建匹配器。你扩展这个类，它会给你一个要覆盖的方法：

```kt
    abstract boolean matches(T t);
```

`matches` 参数匹配器方法期望得到一个你可以用来与预定义变量进行比较的参数。这个参数是你方法调用的“实际”结果，而预定义变量是“预期”的。然后你决定返回 true 或 false，看它们是否相同。

你可能已经意识到，自定义 `ArgumentMatcher` 类在测试中的频繁使用可能会变得非常复杂，并可能导致错误，为了简化这个过程，我们将使用一个辅助类，我们称之为 `CharSequenceMatcher`。我们还有 `EditableCharSequenceMatcher`，可以在本章的示例项目中找到：

```kt
class CharSequenceMatcher extends ArgumentMatcher<CharSequence> {

    private final CharSequence expected;

    static CharSequence charSequenceEq(CharSequence expected) {
        return argThat(new CharSequenceMatcher(expected));
    }

    CharSequenceMatcher(CharSequence expected) {
        this.expected = expected;
    }

    @Override
    public boolean matches(Object actual) {
        return expected.toString().equals(actual.toString());
    }

    @Override
    public void describeTo(Description description) {
        description.appendText(expected.toString());
    }
}
```

我们通过返回将作为参数传递的对象与转换为字符串后我们预定义的字段比较的结果来实现匹配。

我们还覆盖了 `describeTo` 方法，这允许我们在验证失败时更改错误消息。这是一个始终要记住的好技巧：在这样做之前和之后，查看错误消息。

```kt
Argument(s) are different! Wanted: 
textWatcher.afterTextChanged(<Editable char sequence matcher>);
Actual invocation has different arguments:
textWatcher.afterTextChanged(1);

Argument(s) are different! Wanted: 
textWatcher.afterTextChanged(1XX);
Actual invocation has different arguments: 
textWatcher.afterTextChanged(1);
```

当我们使用匹配器的静态实例化方法并将其作为静态方法导入测试中时，我们可以简单地编写：

```kt
verify(mockTextWatcher).onTextChanged(charSequenceEq(output), …
```

# 隔离测试视图

我们在这里分析的测试是基于 Android SDK ApiDemos 项目中的 Focus2AndroidTest。它演示了当行为本身无法被隔离时，如何测试符合布局的视图的一些属性。测试视图的可聚焦性就是这种情况之一。

我们只测试单个视图。为了避免创建完整的 Activity，这个测试扩展了 `AndroidTestCase`。你可能考虑过仅使用 `TestCase`，但不幸的是，这是不可能的，因为我们需要一个 Context 来通过 `LayoutInflater` 加载 XML 布局，而 `AndroidTestCase` 将为我们提供此组件：

```kt
public class FocusTest extends AndroidTestCase {
 private FocusFinder focusFinder;

 private ViewGroup layout;

 private Button leftButton;
 private Button centerButton;
 private Button rightButton;

@Override
protected void setUp() throws Exception {
 super.setUp();

 focusFinder = FocusFinder.getInstance();
 // inflate the layout
 Context context = getContext();
 LayoutInflater inflater = LayoutInflater.from(context);
 layout = (ViewGroup) inflater.inflate(R.layout.view_focus, null);

 // manually measure it, and lay it out
 layout.measure(500, 500);
 layout.layout(0, 0, 500, 500);

 leftButton = (Button) layout.findViewById(R.id.focus_left_button);
 centerButton = (Button) layout.findViewById(R.id.focus_center_button);
 rightButton = (Button) layout.findViewById(R.id.focus_right_button);
}
```

设置将按以下方式准备我们的测试：

1.  我们请求一个`FocusFinder`类。这是一个提供用于查找下一个可聚焦视图的算法的类。它实现了单例模式，因此我们使用`FocusFinder.getInstance()`来获取它的引用。这个类有几种方法可以帮助我们找到在不同条件下可聚焦和可触摸的项，例如在给定方向上最近的或者从特定矩形区域开始搜索。

1.  然后，我们获取`LayoutInflater`类并展开测试下的布局。由于我们的测试与其他系统部分隔离，我们需要考虑的一件事是，我们必须手动测量和布局组件。

1.  然后，我们使用查找视图模式并将找到的视图分配给字段。

在前面的章节中，我们列举了我们的工具库中所有可用的断言，您可能还记得，为了测试视图的位置，我们在`ViewAsserts`类中有一套完整的断言。然而，这取决于布局是如何定义的：

```kt
public void testGoingRightFromLeftButtonJumpsOverCenterToRight() {
 View actualNextButton = 
focusFinder.findNextFocus(layout, leftButton, View.FOCUS_RIGHT);
 String msg = "right should be next focus from left";
 assertEquals(msg, this.rightButton, actualNextButton);
}

public void testGoingLeftFromRightButtonGoesToCenter() {
 View actualNextButton = 
focusFinder.findNextFocus(layout, rightButton, View.FOCUS_LEFT);
 String msg = "center should be next focus from right";
 assertEquals(msg, this.centerButton, actualNextButton);
}
```

`testGoingRightFromLeftButtonJumpsOverCenterToRight()`方法，如其名称所示，测试了当焦点从左向右移动时，右侧按钮获得焦点的情况。为了实现这一搜索，我们在`setUp()`方法中获得的`FocusFinder`实例被使用。这个类有一个`findNextFocus()`方法，可以获取在给定方向上接收焦点的视图。获得的值与我们的预期进行对比检查。

类似地，`testGoingLeftFromRightButtonGoesToCenter()`测试检查了相反方向上的焦点移动。

# 测试解析器

在许多情况下，您的 Android 应用程序依赖于从 Web 服务获取的外部 XML、JSON 消息或文档。这些文档用于本地应用程序和服务器之间的数据交换。有许多用例需要从服务器获取 XML 或 JSON 文档，或者由本地应用程序生成并发送到服务器。理想情况下，由这些活动调用的方法必须独立测试以实现真正的单元测试，为此，我们需要在 APK 中包含一些模拟文件以运行测试。

但问题是，我们可以在哪里包含这些文件呢？

让我们找出答案。

## 安卓资产

首先，可以在 Android SDK 文档中找到关于资产定义的简要回顾。

> *“资源”和“资产”之间的区别表面上看不大，但通常您会更频繁地使用资源来存储外部内容，而不是使用资产。真正的区别在于，放在资源目录中的任何东西都可以通过 Android 编译的 R 类轻松地从应用程序中访问。而放在资产目录中的任何东西将保持其原始文件格式，为了读取它，您必须使用`AssetManager`将文件作为字节流读取。因此，将文件和数据放在资源（res/）目录中可以更容易地访问它们。*

显然，assets 是我们需要存储将被解析以测试解析器的文件。

因此，我们的 XML 或 JSON 文件应该放在 assets 文件夹中，以防止编译时被操纵，并能够在应用程序或测试运行时访问原始内容。

但要小心，我们需要将它们放在`androidTest`文件夹的 assets 中，因为这样，这些就不是应用程序的一部分，而且我们不想在发布实时应用程序时将它们与我们的代码打包在一起。

# 解析器测试

这个测试实现了一个`AndroidTestCase`，因为我们只需要一个上下文来引用我们的 assets 文件夹。同时，我们在测试中编写了解析，因为此测试的重点不是如何解析 xml，而是如何从你的测试中引用模拟资产：

```kt
public class ParserExampleActivityTest extends AndroidTestCase {

 public void testParseXml() throws IOException {
 InputStream assetsXml = getContext().getAssets()
.open("my_document.xml");

  String result = parseXml(assetsXml);
  assertNotNull(result);
 }
}
}
```

`InputStream`类是通过使用`getContext().getAssets()`从 assets 中打开`my_document.xml`文件获得的。请注意，这里获得的上下文和资产来自测试包，而不是被测 Activity。

接下来，使用最近获得的`InputStream`调用`parseXml()`方法。如果发生`IOException`，测试将失败并输出堆栈跟踪中的错误，如果一切顺利，我们将测试结果不为空。

然后，我们应该在名为`my_document.xml`的资产中提供我们想要用于测试的 XML，资产应该在测试项目文件夹下；默认情况下，这是`androidTest/assets`。

内容可能是：

```kt
<?xml version="1.0" encoding="UTF-8" ?>
<records>
  <record>
    <name>Paul</name>
  </record>
</records>
```

# 测试内存使用情况

有时，内存消耗是衡量测试目标（无论是 Activity、Service、Content Provider 还是其他组件）良好行为的一个重要因素。

为了测试这种情况，我们可以使用一个实用测试工具，你可以在运行测试循环后，主要从其他测试中调用它：

```kt
public void assertNotInLowMemoryCondition() {
//Verification: check if it is in low memory
ActivityManager.MemoryInfo mi = new ActivityManager.MemoryInfo();
 ((ActivityManager)getActivity()
.getSystemService(Context.ACTIVITY_SERVICE)).getMemoryInfo(mi);
assertFalse("Low memory condition", mi.lowMemory);
}
```

这个断言可以从其他测试中调用。首先，它通过使用`getSystemService()`获取实例后，使用`getMemoryInfo()`从`ActivityManager`获取`MemoryInfo`。如果系统认为自己当前处于低内存状态，则`lowMemory`字段被设置为`true`。

在某些情况下，我们想要更深入地了解资源使用情况，并可以从进程表中获得更详细的信息。

我们可以创建另一个辅助方法来获取进程信息，并在我们的测试中使用它：

```kt
    private String captureProcessInfo() {
        InputStream in = null;
        try {
           String cmd = "ps";
           Process p = Runtime.getRuntime().exec(cmd);
           in = p.getInputStream();
           Scanner scanner = new Scanner(in);
           scanner.useDelimiter("\\A");
           return scanner.hasNext() ? scanner.next() : "scanner error";
        } catch (IOException e) {
           fail(e.getLocalizedMessage());
        } finally {
           if (in != null) {
               try {
                   in.close();
               } catch (IOException ignore) {
               }
            }
        }
        return "captureProcessInfo error";
    }
```

为了获得这些信息，使用`Runtime.exec()`执行了一个命令（在本例中使用了`ps`，但你可以根据需要调整它）。这个命令的输出被连接在一个字符串中，稍后返回。我们可以使用返回值将输出发送到测试中的日志，或者进一步处理内容以获得摘要信息。

这是一个记录输出的例子：

```kt
        Log.d(TAG, captureProcessInfo());
```

当运行此测试时，我们可以获取有关运行进程的信息：

```kt
D/ActivityTest(1): USER     PID   PPID  VSIZE  RSS     WCHAN    PC   NAME
D/ActivityTest(1): root      1     0     312    220   c009b74c 0000ca4c S /init
D/ActivityTest(1): root      2     0     0      0     c004e72c 00000000 S kthreadd
D/ActivityTest(1): root      3     2     0      0     c003fdc8 00000000 S ksoftirqd/0
D/ActivityTest(1): root      4     2     0      0     c004b2c4 00000000 S events/0
D/ActivityTest(1): root      5     2     0      0     c004b2c4 00000000 S khelper
D/ActivityTest(1): root      6     2     0      0     c004b2c4 00000000 S suspend
D/ActivityTest(1): root      7     2     0      0     c004b2c4 00000000 S kblockd/0
D/ActivityTest(1): root      8     2     0      0     c004b2c4 00000000 S cqueue
D/ActivityTest(1): root      9     2     0      0     c018179c 00000000 S kseriod

```

输出为了简洁起见已被截断，但如果你运行它，你会得到系统上运行的完整进程列表。

获得的信息简要解释如下：

| 列 | 描述 |
| --- | --- |
| USER | 这是文本用户 ID。 |
| PID | 这是进程的进程 ID 号。 |
| PPID | 这是父进程 ID。 |
| VSIZE | 这是进程的虚拟内存大小，以 KB 为单位。这是进程保留的虚拟内存。 |
| RSS | 这是常驻集合大小，即任务已使用的非交换物理内存（以页为单位）。这是进程实际占用的真实内存页数。这不包括尚未按需加载的页面。 |
| WCHAN | 这是进程等待的“通道”。它是系统调用的地址，如果需要文本名称，可以在名称列表中查找。 |
| PC | 这是当前的 EIP（指令指针）。 |

| 状态（无标题） | 这表示以下的过程状态： |

+   S 用于表示在可中断状态下的睡眠

+   R 用于表示运行中

+   T 用于表示已停止的进程

+   Z 用于表示僵尸进程

|

| **列** | **描述** |
| --- | --- |
| NAME | 这表示命令名称。Android 中的应用程序进程会以其包名重命名。 |

# 使用 Espresso 进行测试

测试 UI 组件可能很困难。了解视图何时被加载或确保不在错误的线程上访问视图可能会导致奇怪的行为和不确定的测试结果。这就是谷歌发布了一个用于 UI 相关自动化测试的帮助库 Espresso 的原因。（[`code.google.com/p/android-test-kit/wiki/Espresso`](https://code.google.com/p/android-test-kit/wiki/Espresso)）。

将 Espresso 库 JAR 添加到`/libs`文件夹中可以实现，但为了方便 Gradle 用户，谷歌发布了他们的 Maven 仓库版本（因为幸运的是，在 2.0 版本之前这是不可用的）。使用 Espresso 时，还需要使用捆绑的 TestRunner。因此，设置变为：

```kt
dependencies {
// other dependencies
androidTestCompile('com.android.support.test.espresso:espresso-core:2.0')
}
android {
    defaultConfig {
    // other configuration
    testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
}
// Annoyingly there is a overlap with Espresso dependencies at the moment 
// add this closure to fix internal jar file name clashes
packagingOptions {
        exclude 'LICENSE.txt'
    }
}
```

一旦 Espresso 依赖项被添加到项目中，您就可以流畅地断言 UI 元素的行为。在我们的示例中，我们有一个允许您订购 Espresso 咖啡的 Activity。当您按下订单按钮时，会出现一个精美的 Espresso 图像。我们希望在一个自动化测试中验证这种行为。

首先要做的是设置我们的 Activity 进行测试。我们使用`ActivityInstrumentationTestCase2`，这样我们就可以拥有一个完整的生命周期 Activity 运行。在测试开始时或`setup()`方法中需要调用`getActivity()`，以允许 Activity 启动并且 Espresso 在恢复状态下找到 Activity：

```kt
public class ExampleEspressoTest extends ActivityInstrumentationTestCase2<EspressoActivity> {

    public ExampleEspressoTest() {
        super(EspressoActivity.class);
    }

    @Override
    public void setUp() throws Exception {
        getActivity();
    }
```

设置完成后，我们可以使用 Espresso 编写一个测试，点击按钮并检查图像是否在 Activity 中显示（变为可见）：

```kt
    public void testClickingButtonShowsImage() {
        Espresso.onView(
              ViewMatchers.withId(R.id.espresso_button_order))
              perform(ViewActions.click());

        Espresso.onView(
              ViewMatchers.withId(R.id.espresso_imageview_cup))
                .check(ViewAssertions.matches(ViewMatchers.isDisplayed()));
    }
```

这个示例展示了使用 Espresso 查找我们的订单按钮，点击按钮，并检查我们的订单 Espresso 是否对用户可见。Espresso 有一个流畅的接口，意味着它遵循构建器样式模式，而且大多数方法调用可以被链式调用。在上面的示例中，我为了清晰展示了完全限定类，但这些可以很容易地更改为静态导入，使测试更具可读性：

```kt
    public void testClickingButtonShowsImage() {
        onView(withId(R.id.espresso_button_order))
                .perform(click());

        onView(withId(R.id.espresso_imageview_cup))
                .check(matches(isDisplayed()));
    }
```

现在可以以更加*句子*的样式来阅读这个。这个示例展示了使用 Espresso 查找我们的订单按钮`onView(withId(R.id.espresso_button_order))`。点击`perform(click())`，然后我们找到咖啡杯图片`onView(withId(R.id.espresso_imageview_cup))`，并检查它是否对用户可见`check(matches(isDisplayed()))`。

这表明你需要考虑的类只有：

+   **Espresso**：这是入口点。始终从这一点开始与视图交互。

+   **ViewMatchers**：这用于在当前层次结构中定位视图。

+   **ViewActions**：这用于对定位的视图执行点击、长按等操作。

+   **ViewAssertions**：这用于在执行操作后检查视图的状态。

Espresso 有一个非常强大的 API，它允许你测试视图之间的位置，匹配 ListView 中的数据，直接从头部或脚部获取数据，并检查 ActionBar/ToolBar 中的视图以及许多其他断言。另一个特点是它能处理线程；Espresso 将等待异步任务完成，然后断言 UI 是否已更改。这些特性以及更多的解释都列在 wiki 页面上（[`code.google.com/p/android-test-kit/w/list`](https://code.google.com/p/android-test-kit/w/list)）。

# 概述

在本章中，我们提出了涵盖广泛情况的几个现实世界中的测试示例。在创建你自己的测试时，你可以将它们作为起点。

我们涵盖了一系列的测试方法，你可以为你的测试进行扩展。我们使用了模拟上下文，并展示了`RenamingDelegatingContext`如何在各种情况下被用来改变测试获取的数据。我们还分析了这些模拟上下文注入测试依赖的过程。

然后，我们使用`ActivityUnitTestCase`以完全隔离的方式测试活动。我们使用`AndroidTestCase`以隔离的方式测试视图。我们展示了结合使用 Mockito 和`ArgumentMatchers`来提供任何对象的定制匹配器的模拟对象。最后，我们探讨了潜在的内存泄漏分析，并窥视了使用 Espresso 测试 UI 的强大功能。

下一个章节将专注于管理你的测试环境，以便你能够以一致、快速且始终确定性的方式运行测试，这导致了自动化和那些淘气的猴子！
