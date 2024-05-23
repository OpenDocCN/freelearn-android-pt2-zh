# 第四章：并发与软件设计模式

作为开发者，你不仅要编写能工作的代码，而且要尽可能使用现有的解决方案，以便将来能更好地维护你的代码。如果其他开发者需要在你项目中工作，他们会很快理解你的做法。我们能够实现这一点，要归功于软件设计模式。

为了正确理解这些模式，我们需要了解 Android 中并发是如何工作的基本概述。我们将阐明 UI 线程是什么，并讨论在线程中延迟事件的不同机制。

我们将介绍 Android 中最常用的模式，这将帮助我们进一步理解 Android 功能和开发技术，并成为更好的开发者。

+   并发

    +   处理器和线程

    +   AsyncTask

    +   Service

    +   IntentService

    +   Loader

+   Android 中的模式

    +   单例模式

    +   适配器和持有者模式

    +   观察者模式

# Android 中的并发

如果你是一个 Android 用户，你可能对 ANR 消息有所了解。这可能不会让你立刻明白，所以请看以下图片：

![Android 中的并发](img/4887_04_01.jpg)

当 UI 或主线程中的代码运行时间超过 5 秒，阻塞用户交互时，就会发生**应用无响应**（**ANR**）。

在 Android 中，一个应用程序运行一个单一的线程，称为用户界面线程。我们将解释线程是什么，即使是没有编程背景的读者也能理解。我们可以将线程视为由 CPU 执行的指令或消息列。这些指令来自不同的地方；它们来自我们的应用程序以及操作系统。这个线程用于处理用户的响应、生命周期方法和系统回调。

CPU 逐个顺序处理消息；如果它很忙，消息将在队列中等待执行。因此，如果我们在应用程序中执行长时间的操作并向 CPU 发送许多消息，我们将不会让 UI 消息得到执行，这将导致用户感觉到手机无响应。

![Android 中的并发](img/4887_04_02.jpg)

这个问题的解决方案似乎很明显：如果一个线程不够用，我们可以使用多个。例如，如果我们发起一个网络请求，这个请求将在另一个线程中完成，并且当它结束时，会与主线程通信以显示所请求的数据。

只有主线程或 UI 线程可以访问 UI；因此，如果我们再另一个线程中进行任何后台计算，我们必须告诉主线程显示这些计算的结果，因为我们不能直接从那里做。

## 处理器和线程

我们之前描述的消息在一个称为`MessageQueue`的队列中运行，这个队列对于每个线程是唯一的。处理器可以发送消息到这个队列。当我们创建一个处理器时，它与创建它的线程的`MessageQueue`相关联。

处理器用于两种情况：

+   向同一线程发送延迟消息

+   向另一个线程发送消息

这就是为什么在我们的`SplashActivity`中，我们将使用以下代码：

```kt
new Handler().postDelayed(new Runnable() {
  @Override
  public void run() {

    Intent intent = new Intent(SplashActivity.this, MainActivity.class)

    startActivity(intent);
  }
},3000);
```

### 提示

当你创建一个新的`Handler()`方法时，确保导入`Android.OS`处理器。

在这里，我们使用了`postDelayed(Runnable, time)`方法来发送一个延迟的消息。在这种情况下，消息是一个可运行对象，代表可以执行的命令。

当`runOnUIThread()`活动内部有一个方法允许我们向 UI 线程发送可运行对象时，你就不需要创建一个处理器来与它通信。当我们有活动的上下文并且想在 UI 上运行某些内容时，这非常有用，例如从在后台执行的任务中向 UI 发布更新。

如果我们查看 Android 中该方法的源代码，可以看到它只是简单地使用处理器在 UI 线程中发布可运行对象：

```kt
public final void runOnUiThread(Runnable action) {
  if (Thread.currentThread() != mUiThread) {
    mHandler.post(action);
  } else {
    action.run();
  }
}
```

通常，当我们想要在后台执行一个长时间的任务并希望管理并行线程的执行时，会手动创建线程。线程有一个`run()`方法，其中包含要执行的指令，并且必须在创建后启动以执行`run()`：

```kt
Thread thread = new Thread(){

  @Override
  public void run() {
    super.run();
  }
};

thread.start();
```

创建线程和处理器以执行后台任务的缺点是它的手动处理，如果我们有多个任务，很容易导致应用程序的可读性变得极差。Android 还有其他机制来执行任务，如`AsyncTask`。

## 介绍 AsyncTasks

这可能是你在初学阶段就已经了解的内容，但我们将从并发性的角度来审视它。`Asynctask`是基于线程和处理器的一个类，旨在提供一种在后台执行任务并更新 UI 的简便方法。

要使用`AsyncTask`，需要对其进行子类化，它有四个可以被重写的方法：`onPreExecute`、`doInBackground`、`onProgressUpdate`和`onPostExecute`。

`OnPreExecute`方法是在后台执行任何工作之前被调用的；这意味着它仍然在 UI 线程中，用于在开始任务前初始化变量和进度。

`doInBackground`方法在后台线程中执行。在这里，你可以调用`onProgressUpdate`，它向 UI 线程发布一个更新，例如，通过增加`ProgressBar`的值来显示任务的进度。

最后一个方法`onPostExecute`是在后台任务完成且在 UI 线程中运行时被调用的。

以一个例子来说明：一个`AsyncTask`在后台需要*x*秒完成，并且每秒更新一次进度。进度条对象作为参数在构造函数中传递，秒数作为参数在`execute`方法中传递，并在`doInBackground`中获取。请注意，在以下代码中，`<Integer,Integer,Void>`类型分别指的是输入参数、进度更新和后执行的类型：

```kt
public class MyAsyncTask extends AsyncTask<Integer,Integer,Void> {

  ProgressBar pB;

  MyAsyncTask(ProgressBar pB) {
    this.pB = pB;
  }

  @Override
  protected void onPreExecute() {
    super.onPreExecute();
    pB.setProgress(0);
  }

  @Override
  protected void onProgressUpdate(Integer... values) {
    super.onProgressUpdate(values);
    pB.setProgress(values[0]);
  }

  @Override
  protected Void doInBackground(Integer... integers) {
    for (int i = 0; i < 10; i++){
      try {
        Thread.sleep(1000);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      onProgressUpdate(new Integer[]{i});
    }
    return null;
  }

  @Override
  protected void onPostExecute(Void o) {
    super.onPostExecute(o);
    Log.d("AsyncTask","Completed");
  }

}
```

创建了`AsyncTask`类后，以下是执行它的方法：

```kt
new MyAsyncTask( progressBar ).execute(new Integer[]{10});
```

如果我们同时执行多个 `AsyncTask`，从 Android 3.0 版本开始，默认情况下它们将会按顺序依次运行。如果我们希望它们并行运行，我们需要创建一个执行器，并通过 `THREAD_POOL_EXECUTOR` 参数调用 `executeOnExecutor()`。

至于限制，我们应该提到 `AsyncTask` 必须始终从主线程执行，并且不能在同一个对象中两次调用 `execute()`；因此，它们不能循环执行。

## 理解服务

当下载文件或执行任何需要任务完成时通知 UI 的短操作时，`AsyncTask` 是理想的。然而，在 Android 中，有时需要执行非常长时间的任务，这些任务可能不需要 UI 交互。例如，你可以有一个应用，它与服务器打开一个套接字（直接通道），用于收听广播应用中的音频流。

即使应用不在屏幕上显示，服务也会运行；它默认在后台运行，但使用主线程。因此，如果我们想要执行长时间的任务，我们需要在服务内创建一个线程。它必须在清单中声明，如果我们将其声明为公开，也可以从另一个应用中使用。

与 `AsyncTask` 相比，服务可以从任何线程触发；它们通过 `onStartService()` 方法触发，并通过 `onStopService()` 停止。

可选地，服务可以绑定到一个组件；一旦绑定组件，就会调用 `onBind()`。当绑定发生时，我们有一个接口供组件与服务交互。

### 一种服务类型 —— IntentService

`IntentService` 是 `services` 的一个子类，可以从一个意图触发。它创建一个线程并包含回调，以了解任务何时完成。

`IntentService` 的背后的想法是，如果你不需要并行运行任务，实现一个按顺序接收意图并完成任务后通知的服务更容易。

当我们调用 `onStart` 时，服务会持续运行；然而，`IntentService` 被创建后，只会在接收到意图并完成任务时，在短时间内运行。

以一个实际例子来说，我们可以考虑一个应用在屏幕不在显示时，在后台执行短任务的需求。比如一个新闻阅读应用，将新闻存储在您的设备上，以便您在没有网络的情况下也能离线阅读。这可能是来自一个每天发布文章的报纸的应用，允许用户在没有网络连接的情况下，如在飞机上或火车通勤时阅读文章。

这个想法是，当文章发布时，应用在后台运行时用户会收到推送通知。这个通知将触发一个下载文章的意图，这样用户下次打开应用时，无需任何额外操作，文章就已经在那里了。

下载文章是一个小而重复的任务，需要在应用在后台、在线程中运行时完成，无需并行处理，这正是`IntentService`的理想场景。

## 介绍加载器

为了结束并发部分，我们将快速概览`Loader`类。加载器的目的是更容易在活动中异步加载数据，因此在片段中也是如此。从 Android 3.0 开始，每个活动都有`LoaderManager`来管理在其中使用的加载器。在基于片段导航的应用程序中，即使在你切换片段时，也可以在活动级别执行后台操作。

加载器从数据源加载数据；当这个数据源发生变化时，它会自动刷新信息，这就是为什么加载器与数据库一起使用是完美的。例如，一旦我们将加载器连接到数据库，这个数据库就可以被修改，而加载器将捕获这些更改。这将允许我们立即刷新 UI，让用户看到这些变化。

在第八章《*数据库* *和加载器》中，我们将实现`CursorLoader`来查询我们在`MasteringAndroidApp`中创建的数据库。

# 模式的重要性

当软件开发者需要开发具有特定功能特性的功能或组件时，通常可以用不同的方法完成；可以用不同的代码或不同的结构来完成。很可能同样的问题已经被其他开发者解决了无数次，以至于解决方案从具体实现中抽象出来，变成了一个模式。与其重新发明轮子，不如了解并实现这些模式。

在 Android 开发中，我们每天都在使用模式，即使我们没有意识到。大多数时候，我们使用的是 Android 内置的模式实现。例如，当我们想要执行按钮点击并设置`OnClickListener`时——换句话说，等待`onClick()`方法被调用——我们使用的是观察者模式的实现。如果我们创建一个弹出窗口`AlertDialog`，我们使用的是`AlertDialog.Builder`，它使用了构建器模式。这样的例子有很多，但我们希望的是能够将这些解决方案应用到我们自己的问题中。

有不同类型的模式分为四类，以下是在开发 Android 应用时我们可能会遇到的一些例子：

+   创建型

    +   单例模式

    +   构建器模式

    +   工厂方法模式

+   行为型

    +   观察者模式

    +   策略模式

    +   迭代器模式

+   结构型

    +   适配器模式

    +   门面模式

    +   装饰器模式

+   并发

    +   锁

    +   调度器

    +   读写锁

为了完成`MasteringAndroidApp`，我们需要实现前三组中的模式。关于第四组（并发），我们需要了解 Android 中的并发概念，但我们不会自己实现一个并发模式。

### 小贴士

模式通常由 UML 图表示。

根据维基百科（[`en.wikipedia.org/wiki/Class_diagram`](http://en.wikipedia.org/wiki/Class_diagram)），“*在软件工程中，统一建模语言（UML）中的类图是一种静态结构图，通过显示系统的类，它们的属性，操作（或方法）以及对象之间的关系来描述系统的结构*”。

## 单例模式

软件设计模式中的单例模式（Singleton）限制了对象的创建，只允许有一个实例。其思想是全局访问这个单一对象。

该模式的实现是，如果之前没有创建对象，则创建对象，如果已创建，则返回现有实例。以下是 UML 图：

![单例模式](img/B04887_04_03.jpg)

在某些情况下，我们希望一个对象能够全局访问，并且希望它在我们的应用中是唯一的。例如，在使用 Volley 时，我们希望保持一个唯一的请求队列，以使所有请求都在同一个队列中，并且我们希望它能够全局访问，因为我们需要从任何片段或活动中添加请求。

这是一个单例实现的基本示例：

```kt
public class MySingleton {

    private static MySingleton sInstance;

    public static MySingleton getInstance(){
        if (sInstance == null) {
            sInstance = new MySingleton();
        }
        return sInstance;
    }
}
```

为了理解实现，请记住在 Java 中，静态变量与类相关联，而不是与对象相关。同样，静态方法可以在不创建类实例的情况下调用。

拥有一个静态方法意味着它可以在我们应用的任何地方被调用。我们可以调用`MySingleton.getInstance()`，它总是返回同一个实例。第一次调用时，它会创建并返回它；之后的调用，它会返回已创建的实例。

使用单例和测试框架有一个缺点；我们将在第十一章，*安卓上的调试与测试*中讨论这个问题。

### 应用类中的单例

我们可以将单例实现适配到 Android。考虑到在`Application`类中的`onCreate`方法在我们打开应用时只被调用一次，且`Application`对象不会被销毁，我们可以在应用中实现`getInstance()`方法。

应用这些更改后，我们的应用类将类似于以下结构：

```kt
public class MAApplication extends Application {

  private static MAApplication sInstance;

  @Override
  public void onCreate() {
    super.onCreate();

    sInstance = this;

    // Enable Local Datastore.
    Parse.enableLocalDatastore(this);

    ParseObject.registerSubclass(JobOffer.class);

    Parse.initialize(this, "KEy", "KEY");
  }

  private static MAApplication getInstance(){
    return sInstance;
  }
}
```

现在，我可以在应用的任何地方调用`MAAplication.getInstance()`，并在应用类中创建可以通过单例`MAAplication`对象全局访问的成员变量。例如，在 Volley 的情况下，我可以在`OnCreate()`中创建`RequestQueue`，然后随时从`MAAplication`对象中获取它。执行以下代码：

```kt
private RequestQueue mRequestQueue;

@Override
public void onCreate() {
  super.onCreate();

  sIntasnce = this;

  mRequestQueue = Volley.newRequestQueue(this);
  .
  .
  .
}

public RequestQueue getRequestQueue(){
  return mRequestQueue;
}
```

采用这种方法，我们有一个单例，即我们的`Application`类；其余的全局可访问对象都是成员变量。另一个选项是创建一个新的单例类来存储 Volley 请求队列，并为每个需要全局访问的对象创建一个新的请求单例。

### 提示

不要在`Application`类中使用此方法来持久化数据。例如，如果我们通过点击主页按钮进入后台，经过一段时间后，Android 可能需要内存，并会杀死该应用。因此，下次打开应用时，即使看起来像是返回到上一个实例，实际上会创建一个新的实例。如果你在`onCreate`中重新初始化所有变量，并且后来不改变它们的状态，这样做是没问题的。为了避免这个问题，请避免使用 setter 方法。

## 观察者模式

这种模式在 Android 中被广泛使用。我们讨论的大多数网络库都实现了这种模式，如果你是 Android 开发者，你肯定已经多次使用过它——我们需要实现它，甚至是为了检测按钮的点击。

观察者模式基于一个对象，即观察者，它注册其他对象来通知它们状态变化；在这里，监听状态变化的对象是*观察者*。这种模式可以用来创建一个发布/订阅系统：

![观察者模式](img/B04887_04_04.jpg)

以下是一个注册多个观察者的模式的实现：

```kt
public class MyObserved {

  public interface ObserverInterface{
    public void notifyListener();
  }

  List<ObserverInterface> observersList;

  public MyObserved(){
    observersList = new ArrayList<ObserverInterface>();
  }

  public void addObserver(ObserverInterface observer){
    observersList.add(observer);
  }

  public void removeObserver(ObserverInterface observer){
    observersList.remove(observer);
  }

  public void notifyAllObservers(){
    for (ObserverInterface observer : observersList){
      observer.notify();
    }
  }
}

public class MyObserver
implements MyObserved.ObserverInterface {

  @Override
  public void notify(){
    //Do something
  }
}
```

你会注意到，观察者可以是任何实现接口——`ObserverInterface`的对象。这个接口在观察对象中定义。

如果我们将这个与在 Android 中处理按钮点击的方式进行比较，我们会执行`myButton.setOnClickListener(observer)`。在这里，我们添加了一个等待点击的观察者；这个观察者实现了`OnClick()`方法，这是我们案例中通知的方法。

在 Volley 中，当我们创建一个网络请求时，我们必须指定两个作为参数的监听器：`Response.Listener`和`Response.ErrorListener`。它们分别调用`onResponse()`和`onErrorResponse()`。这是观察者模式的一个清晰实现。

我们将在第六章 *CardView 和材料设计*中实现观察者模式的一个变体，即发布/订阅模式的一个示例。

## 介绍适配器模式

**适配器**是在创建`ListView`或`ViewPager`时在 Android 中使用的元素，但它也是一个著名的设计模式。我们将看看两者的定义及其关系。

一方面，适配器作为设计模式，是介于两个不兼容接口之间的桥梁。它允许两个不同的接口一起工作。这就像现实世界中的适配器，比如 SD 卡到 micro SD 卡的适配器，它允许两个不兼容的系统协同工作。如图所示，适配器被调用新的所需方法，但在内部，它调用来自*被适配者*的旧方法。

![介绍适配器模式](img/4887_04_05.jpg)

另一方面，来自`android.widget.Adapter`的适配器是一个对象，我们用它为列表的每一行或视图翻页的每一页创建视图。因此，它适配了一组数据和一组视图。

要实现一个适配器，我们必须扩展`BaseAdapter`并重写`getView()`和`getCount()`方法。通过这两个方法，适配器将知道它必须创建多少个视图以及如何创建这些视图。

我们将在下一章更深入地探讨这个主题，同时会涉及到`ListViews`的使用，并讨论`ViewHolder`模式，这是在 Android 中处理适配器和列表时经常使用的一种特定模式。

# 总结

在本章结束时，你应该能够理解 Android 中的并发性以及与之相关的所有不同机制。你应该知道有一个主线程用于更新 UI，我们可以创建后台线程来执行其他任务。你还必须了解应用程序在后台（换句话说，不在屏幕上）执行任务与在后台线程中执行任务的区别。你也应该知道软件设计模式的重要性，并能够实现其中的一些。

在下一章中，我们将看看如何使用列表视图，我们将实现一个适配器，并发现一种新模式`ViewHolder`，它将是理解在 Android Lollipop 中引入的`ListView`和`RecyclerView`之间区别的关键。
