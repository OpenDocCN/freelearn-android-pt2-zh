# 第八章：探索外部数据库

在上一章中，我们介绍了从完全局限于 Android 客户端的数据库，转向使用外部数据库的概念，这在我们整个开发过程中可以以多种方式提供帮助。

我们已经看到，通过使用外部数据库，我们能够在 Android 应用程序中改善内存使用（特别是，不必存储非常大的数据库文件），同时通过使用缓存而没有牺牲太多性能。此外，我们还了解到，使用外部数据库允许我们备份用户数据（以防用户更换手机或卸载你的应用程序），防止用户看到过时的数据（因为所有数据都存在于一个中央位置），以及可能查看其他用户的数据（记得全球高分示例）。

使用可以通过网络与应用程序通信的外部数据库，将使你成为更加多才多艺的应用程序开发者，并为你提供创建完全可扩展的数据中心应用程序的工具。

# 不同的外部数据库

那么到底有哪些类型的外部数据库呢？正如 Android、iOS、Palm 等操作系统都允许你开发移动应用一样，目前有几个容易访问的平台允许你托管和开发外部数据库。

其中一个“平台”就是设置一个具有数据库功能的传统专用服务器。例如，这种方式的流行例子是使用专用计算机托管连接到**MySQL**数据库的**Apache Tomcat**服务器。我不会详细介绍如何设置这种服务器-数据库连接（主要是因为你可以用无数种方式来做），但让我们先考虑一下高级概念，然后再来看一个简单的优缺点列表。

在高层次上，Apache Tomcat 服务器充当了一个中介，处理所有的进出的 HTTP 请求（即网络请求）。服务器本身监听所有这些传入的请求，并在接收到请求时，有代码告诉它如何处理请求以及随后返回什么作为响应。处理请求并返回响应的代码通常被称为**HTTP 服务端小程序**，在接下来的章节中，我们将实际实现一些这样的小程序，以便让你更好地了解它们是如何工作的。

然而，Apache Tomcat 服务器还通过**Java 数据库连接**驱动（**JDBC**）连接到 MySQL 数据库。一旦配置好，这将允许我们处理传入的 HTTP 请求，这些请求然后告诉服务器向 MySQL 数据库发出查询。一旦 MySQL 数据库检索到查询，它将执行它并返回所需的数据，最终发送回原始请求者。

使用这种模式，优点是它完全可定制，你可以完全控制每个部分的实现方式。然而，这可以是一把双刃剑，是好是坏取决于谁在处理服务器和数据库。关注数据库部分，由于它完全可定制，我们可以完全控制我们想要使用的**数据库管理系统（DBMS）**，甚至进一步控制我们的数据库架构应该是什么样的，以满足给定的数据库管理系统。在整个应用程序开发过程中，如果我们认为有必要，我们甚至可以选择更换我们的 DBMS 或更改我们的架构——例如，如果我们需要一个更具可扩展性的 DBMS。

问题就出在这里。虽然 MySQL 到目前为止是全球使用最广泛的数据库管理系统，在大多数情况下都能出色完成任务，但它并非设计为极致可扩展的。因此，对于大型、数据密集型的应用程序来说，使用 MySQL 可能是一个次优的设计决策。回到我最初的观点，即使用完全可定制的服务器和数据库可能是一把双刃剑，可以很容易看出在这种情况下灵活性和责任感是如何并存的。当我们获得系统设计和实施的更多灵活性时，同时我们也承担着更多关于做出明智设计决策的责任——否则，我们的应用程序性能可能会迅速恶化（即想象一下，如果所有谷歌的数据都托管在一台计算机上——那将是一场噩梦）。

其他缺点是，这些系统通常需要更高的初始成本，因为我们需要实际购买计算机/服务器。此外，由于这些计算机/服务器容易发生故障，我们将不得不定期管理它们，以确保它们不会崩溃。由于它们的灵活性，许多公司和初创企业选择这种模式，尽管许多公司最终会雇佣专门负责维护这些服务器以及后端开发人员专门构建这些服务器和数据库的专家。

然而，近年来云计算的概念越来越受欢迎，这里我将介绍两个这样的平台。第一个是 **亚马逊网络服务（AWS）**，它提供了一系列云计算服务，特别是 **亚马逊弹性计算云（EC2）** 和 **亚马逊关系数据库服务（RDS）**。这两者之间的主要区别在于，亚马逊的 EC2 被设计成一个功能齐全、完全虚拟的计算环境，允许你控制尽可能多的服务器/数据库实例（从而使其具有固有的可扩展性）。而亚马逊的 RDS，则被设计成仅作为一个云数据库，尽管该服务包含了一些使你能够选择扩展计算和存储能力的功能。因此，根据你的应用程序，你可以选择最合适的服务。亚马逊的计算服务现在被许多公司使用，包括 Yelp、Reddit、Quora、FourSquare、Hootsuite 等知名初创公司，在设计任何未来后端时，这绝对是一个值得考虑的因素。

另一个云计算服务是 **谷歌的应用引擎（GAE）**，我们会更深入地了解它。AWS 和 GAE 都容易设置（相对于传统服务器方法），而 GAE 以更用户友好而闻名。然而，我们之所以要关注 GAE 而非 AWS（除了这是一本现在以 Google 为主题的书之外）的主要原因是，GAE 允许你免费运行小规模的应用程序（直至某些预定义的限制），而 AWS 只允许你在一年内访问其免费定价层。这样，每个人都能在后续章节中跟随我们查看更多代码。

最后，传统服务器/数据库模型与新的云计算模型之间的区别在于，我们实际上不需要拥有和管理专用的服务器！这些云计算服务允许我们基本上在亚马逊/谷歌的各个数据中心内“租用”服务器空间，并允许我们快速/低成本地创建可靠、可扩展的应用程序。然而，我们放弃的是在实施过程中的一些控制和灵活性，在下一节我们讨论 **Google App Engine 的 Java 数据对象 (JDO)** 数据库时，我会进一步阐述这一点。

# Google App Engine 和 JDO 数据库

那么，Google App Engine 究竟是什么，我们为什么需要它呢？其实，GAE 是一个平台，它允许你在支撑 Google 应用程序的同一天然系统上构建和托管网络应用。GAE 使我们能够快速开发部署应用程序，而无需担心可靠性、可扩展性、硬件、补丁或备份等问题。然而，这种可靠性和可扩展性是有代价的，那就是我们在选择 DBMS 和设计数据库架构时的灵活性。实际上，当你选择使用 GAE 作为后端时，这两者基本上都是为你预先选好的！

GAE 附带了一个 JDO 数据库——这意味着它带有一个特殊的数据库，允许你将 Java 对象直接转换为称为**实体**（因此得名）的数据行。这个 JDO 数据库构建在一个名为 BigTable 的特殊网络数据库之上，它旨在实现极快和可扩展，实际上并不是像 MySQL 那样的关系型数据库管理系统（见[`en.wikipedia.org/wiki/BigTable)`](http://en.wikipedia.org/wiki/BigTable)）。这主要意味着我们在第三章中学习的关于 SQL（即 JOINS）的所有功能在这里并不都适用，因此关于你的数据库架构应该如何设计，你的决策会有一定的限制。

鉴于此，谷歌提供了一种名为**GQL**的 SQL 变体，这是一种查询语言，专为从 App Engine 可扩展数据存储中检索实体而设计。同样，这里有一些差异，但 GQL 的一般感觉与 SQL 非常相似：你有带有`WHERE`筛选器的`SELECT`语句，以及其他熟悉的子句，如`ORDER BY`和`LIMIT`。这样，对于那些只熟悉像 MySQL 这样的关系系统的用户来说，学习起来应该不会太困难。

为了完整性起见，其他差异包括在不建立索引的情况下无法基于多个条件进行筛选，无法在同一个查询中对多个列使用不等式筛选器，以及无法筛选缺少字段的行等。所有这些看似任意的差异的原因都涉及到 BigTable 数据库的架构。由于它的设计方式以及它为每插入的行建立索引的方式，像 MySQL 这样的关系数据库中可用的某些查询将不再适用于 BigTable。然而，正是由于这种架构，BigTable 本质上是可扩展的，因此在选择两者之间时，请记住这些权衡。

无论如何，语言只能带你走到这么远，一旦我们开始看到一些实际的代码，所有这些差异和相似性将会变得更加清晰。因此，除了安装 Android SDK 之外，我建议你花些时间通过以下 URL 指南来搭建 Google App Engine：

[`code.google.com/appengine/downloads.html#Download_the_Google_App_Engine_SDK`](http://code.google.com/appengine/downloads.html#Download_the_Google_App_Engine_SDK)

在这一点上，我们已经准备好直接深入一些代码，尝试为我们的 Android 应用程序拼凑一个完全功能的 Google App Engine 后端！

# GAE：以视频游戏为例

在接下来的几章中，我们将通过一个例子来学习如何创建一个应用程序，以查看通过 Blockbuster 可以获取哪些视频游戏。这将涉及到从编写爬虫来从 Blockbuster 的网站获取和检索这些视频游戏，将游戏对象存储到我们的 GAE 数据库中，编写 servlet 通过 HTTP 请求从我们的 GAE 数据库中获取/删除游戏对象，最后但同样重要的是，完成一些适用于 Android 客户端的代码。

在本章中，我们将重点介绍如何设置数据库并编写包装方法，以帮助我们存储、检索、更新和删除数据，为后续步骤做准备。首先，每个 GAE 应用程序都需要定义一个基本实体类，这个类本质上定义了数据库中的行。请注意，每个实体都需要有一个与之关联的 ID 或键，所以我们真正需要的只是一个 ID 字段。下面是`ModelBase`类，我们将它用作我们的基本实体类，并对我们创建的所有对象进行重写：

```kt
@PersistenceCapable(detachable = "true")
@Inheritance(strategy = InheritanceStrategy.SUBCLASS_TABLE)
public class ModelBase {
@PrimaryKey
@Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
private Long id;
public Long getId() {
return id;
}
}

```

我们会注意到这个类的总体结构类似于相对简单的 Java 对象，但有一些奇怪的`@`标签。让我们先看看前两个：

```kt
@PersistenceCapable(detachable = "true")
@Inheritance(strategy = InheritanceStrategy.SUBCLASS_TABLE)

```

第一点告诉我们，这个类需要是`PersistenceCapable`。当你定义一个对象为可持久化的，你其实是在告诉 JDO 数据库，这个对象能够从数据存储中存储和检索。声明实体类为`PersistenceCapable`并声明所需的字段为`Persistent`是非常重要的。你会看到还有一个名为`detachable`的参数，我们将其设置为`true`。这使得我们有权在关闭数据库后编辑和修改从数据库中检索的实体。现在，这并不意味着这些修改会在数据库中持久化，因为数据库已经关闭，但至少我们有权限这样做。

接下来是`Inheritance`标签，这意味着我们允许创建覆盖这个基础实体的实体，从而继承基础实体。另外两个标签相当容易理解。第一个声明我们的 ID（我快速说明一下，在我的例子中我选择使用 long 类型的 ID，但也可以使用 Key 类型对象）作为我们实体的`PrimaryKey`。对于有 SQL 背景的人来说，这应该立即就能明白，但这基本上就是告诉 JDO 数据库，这个实体的对象将有一个唯一的 long ID 字段，用于查找等操作。最后一个标签是我们之前简要提到的一个——即`Persistent`标签，它只是告诉我们这个 long ID 字段应该作为我们表中自己的列存储。

现在，对于实际的`VideoGame`对象，首先注意我们是如何扩展（继承）之前的`ModelBase`类，然后继续定义所有期望的持久化字段，并实现构造函数等，如下所示：

```kt
// NOTE HOW WE DECLARE OUR OBJECT AS PERSISTENCE CAPABLE
@PersistenceCapable
public class VideoGame extends ModelBase {
// NOTE THE PERSISTENT TAGS
@Persistent
private String name;
// USE A SPECIAL GOOGLE APP ENGINE LINK CLASS FOR URLS
@Persistent
private Link imgUrl;
@Persistent
private int consoleType;
public VideoGame(String name, String url, String consoleType) {
this.name = name;
this.imgUrl = new Link(url);
// CONVERT ALL CONSOLES TO INTEGER TYPES
this.consoleType = VideoGameConsole.convertStringToInt(consoleType);
}
public String getName() {
return name;
}
public void setName(String name) {
this.name = name;
}
public Link getImgUrl() {
return imgUrl;
}
public void setImgUrl(Link imgUrl) {
this.imgUrl = imgUrl;
}
public int getConsoleType() {
return consoleType;
}
public void setConsoleType(int consoleType) {
this.consoleType = consoleType;
}
public static class VideoGameConsole {
public static final String XBOX = "Xbox";
public static final String PS3 = "Ps3";
public static final String WII = "Wii";
public static final String PSP = "Psp";
public static final String DS = "NintendoDS";
public static final String PS2 = "Ps2";
public static final String[] CATEGORIES = { "Xbox", "Ps3", "Wii", "Psp", "NintendoDS", "Ps2" };
public static int convertStringToInt(String type) {
if (type == null) { return -1; }
if (type.equalsIgnoreCase(XBOX)) {
return 0;
} else if (type.equalsIgnoreCase(PS3)) {
return 1;
} else if (type.equalsIgnoreCase(PS2)) {
return 2;
} else if (type.equalsIgnoreCase(PSP)) {
return 3;
} else if (type.equals(WII)) {
return 4;
} else if (type.equals(DS)) {
return 5;
} else {
return -1;
}
}
}
}

```

一旦你理解了`@`标签的作用，其余部分就相当容易理解了。这里我只是声明了几个字段为持久化字段，然后实现了一个构造函数以及一个方便的内部类。我喜欢有一个便捷类（在这个例子中是`VideoGameConsole`）的原因是，在表中，查询整数通常比查询字符串更有效且更可靠（一方面：你不需要担心大小写匹配，另一方面：整数比较通常比字符串比较更有效）。因此，理想情况下，我希望能有一种方法将字符串转换为整数，甚至可能将一组字符串映射到一个整数（例如，“PS3”可以映射到 1，同样“Playstation 3”或“PS 3”也可以）。

既然我们已经定义了`VideoGame`实体，我们就可以开始实现数据库，并告诉它如何与这些`VideoGame`实体进行交互。

# 持久化管理器和查询

第一步是定义一种方法，以建立服务器与数据库之间的连接。回想一下在本书开始时，我们在进行任何查询之前必须调用如`getWritableDatabase()`这样的方法？在这里也是一样的，但我们不是使用`SQLiteOpenHelper`类，而是定义一个`PersistenceManager`类，如下所示：

```kt
public final class PMF {
private static final PersistenceManagerFactory pmfInstance = JDOHelper.getPersistenceManagerFactory("transactions-optional");
private PMF() {
}
public static PersistenceManagerFactory get() {
return pmfInstance;
}
}

```

注意它被定义为单例以提高效率，我们所做的一切就是打开一个可以处理事务（查询）的持久化（数据库）管理器。然后在我们的未来查询中，我们不再需要通过重复请求`PersistenceManager`来牺牲性能，而是可以直接获取现有实例。

一旦我们定义了`PersistenceManager`，我们就可以开始实现一系列包装器，并且我们将从如何插入新的游戏对象开始看起：

```kt
public class VideoGameJDOWrapper {
/**
* INSERT A SINGLE VIDEOGAME OBJECT
*
* @param g
* - a video game object
*/
public static void insertGame(VideoGame g) {
PersistenceManager pm = PMF.get().getPersistenceManager();
try {
pm.makePersistent(g);
} finally {
pm.close();
}
}
/**
* INSERT MULTIPLE VIDEOGAME OBJECTS - MORE EFFICIENT METHOD
*
* @param games
* - a list of video game objects
*/
public static void batchInsertGames(List<VideoGame> games) {
PersistenceManager pm = PMF.get().getPersistenceManager();
try {
// ONLY NEED TO RETRIEVE AND USE PERSISTENCEMANAGER ONCE
pm.makePersistentAll(games);
} finally {
pm.close();
}
}
}

```

不错吧？这个概念很简单，我们之前已经见过，只需获取`PersistenceManager`的实例（即与数据库的连接）并使传入的`VideoGame`对象持久化。再次提醒，在使用 GAE 时，持久化的概念与插入相同，因此通过使对象持久化，我们实际上是在告诉数据库将我们的实体转换成`VideoGame`表的一行。我们还可以看到，当一次添加多个实体时，GAE 通过使用批量插入为我们提供了一个高效的实现方式。现在让我们看看如何从数据库中获取视频游戏对象。查询实体比简单插入实体要复杂得多，但与其像第三章那样专门用一整章介绍所有不同的查询方式，我只想展示一种方便直观的方法，如果你好奇，我邀请你查看：

[`code.google.com/appengine/docs/java/datastore/queries.html`](http://code.google.com/appengine/docs/java/datastore/queries.html)（链接内容不需翻译，保留英文）

但是，以下是实现方式之一，它应该会让你想起我们之前遇到的`SQLiteQueryBuilder`类：

```kt
public class VideoGameJDOWrapper {
public static void insertGame(VideoGame g) {
. . .
}
public static void batchInsertGames(List<VideoGame> games) {
. . .
}
/**
* GET ALL VIDEO GAMES OF A CERTAIN PLATFORM
*
* @param platform
* - desired platform of games
* @return
*/
public static List<VideoGame> getGamesByType(String platform) {
PersistenceManager pm = PMF.get().getPersistenceManager();
// CONVERT STRING OF PLATFORM TO INT TYPE
int type = VideoGameConsole.convertStringToInt(platform);
// INIT A NEW QUERY AND SPECIFY THE OBJECT TYPE
Query query = pm.newQuery(VideoGame.class);
// SET THE FILTER - EQUIVALENT TO SQL WHERE FILTER
query.setFilter("consoleType == inputType");
// TELL THE QUERY WHAT PARAMETERS YOU WILL SEND
query.declareParameters("int inputType");
List<VideoGame> ret = null;
try {
// EXECUTE QUERY WITH PARAMETERS
ret = (List<VideoGame>) query.execute(type);
} finally {
// CLOSE THE QUERY AT THE END
query.closeAll();
}
return ret;
}
/**
* GET ALL VIDEO GAMES OF A GIVEN PLATFORM WITH A LIMIT ON THE NUMBER OF
* RESULTS
*
* @param platform
* - desired platform of games
* @param limit
* - max number of results to return
* @return
*/
public static List<VideoGame> getGamesByTypeWithLimit (String platform, int limit) {
int type = VideoGameConsole.convertStringToInt(platform);
PersistenceManager pm = PMF.get().getPersistenceManager();
Query query = pm.newQuery(VideoGame.class);
query.setFilter("consoleType == inputType");
query.declareParameters("int inputType");
// SAME QUERY AS ABOVE BUT THIS TIME SET A MAX RETURN LIMIT
query.setRange(0, limit);
List<VideoGame> ret = null;
try {
ret = (List<VideoGame>) query.execute(type);
} finally {
query.closeAll();
}
return ret;
}
/**
* QUICKEST WAY TO RETRIEVE OBJECT IF YOU HAVE THE ID
*
* @param id
* - row id of the object
* @return
*/
public static VideoGame getVideoGamesById(long id) {
PersistenceManager pm = PMF.get().getPersistenceManager();
return (VideoGame) pm.getObjectById(VideoGame.class, id);
}
}

```

让我们逐块分析第一种方法：

```kt
PersistenceManager pm = PMF.get().getPersistenceManager();
// CONVERT STRING OF PLATFORM TO INT TYPE
int type = VideoGameConsole.convertStringToInt(platform);
// INIT A NEW QUERY AND SPECIFY THE OBJECT TYPE
Query query = pm.newQuery(VideoGame.class);

```

我们首先获取`PersistenceManager`实例，然后将传入的平台转换为整型，因为我们将按平台进行过滤。接下来，我们告诉`PersistenceManager`我们要打开一个新的查询（即开始一个新的`SELECT`语句），因此我们调用了`newQuery()`方法。然后，我们使用以下方法设置查询的详细信息：

```kt
// SET THE FILTER - EQUIVALENT TO SQL WHERE FILTER
query.setFilter("consoleType == inputType");
// TELL THE QUERY WHAT PARAMETERS YOU WILL SEND
query.declareParameters("int inputType");

```

这里我们首先设置过滤器，并指定想要执行过滤的列（即设置查询的`WHERE`部分）。接下来，我们为将要传递的参数设置一个占位符（回想一下之前的?占位符），最后，我们执行查询并传递平台类型参数。在下面这个方法中，除了增加了一个`LIMIT`过滤器外，其他都保持不变，该过滤器通过以下方法设置：

```kt
query.setRange(0, limit);

```

我们实现的第三种方法相对直接——JDO 数据库允许你通过调用`PersistenceManager`的`getObjectById()`方法，快速检索具有唯一键或 ID 的实体。同样，在 GAE 中执行查询的方法有很多，以及许多其他子句和细微差别，本书将不展开讨论，但现在你应该掌握了基本概念，并应该能够执行绝大多数需要的查询。最后，让我们看看如何从数据库中更新和删除对象：

```kt
public class VideoGameJDOWrapper {
public static void insertGame(VideoGame g) {
}
public static void batchInsertGames(List<VideoGame> games) {
}
public static List<VideoGame> getGamesByType(String platform) {
}
public static List<VideoGame> getGamesByTypeWithLimit (String platform, int limit) {
. . .
}
public static VideoGame getVideoGamesById(long id) {
. . .
}
/**
* METHOD FOR UPDATING THE NAME OF A VIDEO GAME
*
* @param newName
* - new name of the game
* @param id
* - the row id of the object
* @return
*/
public static boolean updateVideoGameName(String newName, long id) {
PersistenceManager pm = PMF.get().getPersistenceManager();
boolean success = false;
try {
// AS LONG AS PERSISTENCE MANAGER IS OPEN THEN ANY CHANGES TO OBJECT
// WILL AUTOMATICALLY GET UPDATED AND STORED
VideoGame v = (VideoGame) pm.getObjectById(VideoGame. class, id);
if (v != null) {
// KEEP PERSISTENCEMANAGER OPEN
v.setName(newName);
success = true;
}
} catch (JDOObjectNotFoundException e) {
e.printStackTrace();
success = false;
} finally {
// ONCE CHANGES ARE MADE - CLOSE MANAGER
pm.close();
}
return success;
}
/**
* DELETE ALL GAMES OF A CERTAIN PLATFORM
*
* @param platform
* - specify the platform of the games you want to delete
*/
public static void deleteGamesByType(String platform) {
PersistenceManager pm = PMF.get().getPersistenceManager();
int type = VideoGameConsole.convertStringToInt(platform);
// INIT QUERY AGAIN
Query query = pm.newQuery(VideoGame.class);
// SAME WHERE FILTERS
query.setFilter("consoleType == inputType");
query.declareParameters("int inputType");
// NOW CALL THE DELETE METHOD
query.deletePersistentAll(type);
}
}

```

再次，我们以第一种方法——更新方法为例，逐块分析：

```kt
PersistenceManager pm = PMF.get().getPersistenceManager();
boolean success = false;
try {
VideoGame v = (VideoGame) pm.getObjectById(VideoGame.class, id);
if (v != null) {
// KEEP PERSISTENCEMANAGER OPEN
v.setName(newName);
success = true;
}
}

```

与之前的示例一样，我们首先与 JDO 数据库建立连接。然后尝试通过调用`getObjectById()`方法并传入我们想要更新的实体的唯一 ID 来检索我们的`VideoGame`对象。以下是关于`PersistenceManager`的一件你应该记住的奇怪事情。

与我们现在习惯看到的显式更新方法不同，只要连接打开，对对象所做的任何更改都会自动在数据库中更新。因此请注意，在这个方法中，第一步是检索实体，在连接仍然打开时更新它，然后在实体更新后关闭连接。

当然，在这个例子中，我们一次只更新一个特定的 ID，但可以想象，如果我们牢记这个细节，就可以轻松编写一个同时更新一组实体的方法——只需查询它们的一列表，并在`PersistenceManager`仍然打开的情况下逐一更新。

最后但并非最不重要的是，对于我们的删除方法，我们看到除了最后一行使用方法之外，所有步骤都与之前的 get 方法相同：

```kt
// NOW CALL THE DELETE METHOD
query.deletePersistentAll(type);

```

否则，所有之前的逻辑保持不变。就是这样！现在我们有一个 JDO 数据库包装类，它让我们可以抽象出所有混乱的`PersistenceManager`语法，并为我们提供了一种快速插入、检索、更新和删除 GAE 后端数据的方法！下一步是实际找出获取这个视频游戏数据的方法，到那时我们可以简单地将它包装在我们的`VideoGame`实体类中，并将其推送到我们的数据库。

# 总结

在本章中，我们离开了 Android 平台，开始扩展对外部数据库的理解。我们首先简要地了解了我们的选择：传统的专用服务器与数据库连接（例如，将 Apache Tomcat 服务器连接到 MySQL 服务器）或云计算服务器/数据库组合，如**亚马逊网络服务（AWS）**或**谷歌应用引擎（GAE）**。

谷歌应用引擎的好处在于，它更容易设置，并允许我们构建简单、相对小规模的应用程序，不受成本和时间限制。这两种云计算解决方案都配备了可靠的服务器以及高效、可扩展的数据库，但限制了你对后端的控制程度——特别是与购买自己的专用服务器时拥有的无限自由相比。

我们继续使用 GAE，开始构建一个简单的视频游戏应用程序，显示我们通过 Blockbuster 可以获得的所有游戏。我们引入了 GAE 中持久化的概念，并编写了我们的第一个实体类。然后，我们编写了自己的`PersistenceManager`单例类，并实现了一个方便的类，用于从我们的数据库获取、插入、更新和删除数据。

我们在本章中涉及了很多内容，但要在拥有一个完整、完全可用的应用程序之前，我们还有很长的路要走。在下一章中，我们将探讨如何使用本章编写的包装方法来检索数据并将其存储起来。
