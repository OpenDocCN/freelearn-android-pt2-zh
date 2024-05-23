# 第九章：数据收集与存储

我们继续前进！在上一章中，我们介绍了一些你可以使用的外部数据库，并决定使用谷歌的应用引擎（GAE）开发一个功能齐全的后端。我们在 GAE 上成功创建了一个新项目，并使用`PersistenceManager`构建了一个非常实用的包装类，该包装类展示了我们 JDO 数据库中的一些核心概念。当我们开始插入实际数据，并随后使用我们的 Android 应用程序查询这些数据时，这个包装类将非常方便。

接下来就是这里——下一步！对于大多数试图构建以数据为中心的应用程序的人来说，实际获取这些数据将极其困难，通常需要大量的时间和金钱。然而，我们现在有很多工具和方法可以帮助我们利用现有数据来填充我们的数据库。在接下来的章节中，我们将研究其中一些方法，并最终将我们新获取的数据插入到 JDO 数据库中。

# 数据收集方法

首先，让我们简要回顾一下你可以收集数据的两种不同方式：

+   通过应用程序编程接口（API）

+   通过网络抓取

第一种也是最简单的方式是使用 API。对于那些以前从未使用过 API 的人，可以将这看作是由第三方公司创建的*网络图书馆*，通常允许你调用一些函数（几乎总是以 HTTP 请求的形式执行），从而访问他们数据的一个子集。

例如，一个常见的 API 是 Facebook Graph API，当通过验证后，它允许你查询用户的个人资料信息或事件的详情等。本质上，通过 API，我可以访问到在 Facebook 网站上能看到的人或事件的相同数据，只是通过不同的渠道。这就是我所说的公司*公开*其数据的一个子集。另一个例子可能是 Yelp，它的 API 允许你通过传递一组参数（即位置）来查询餐馆和场所。在这里，即使我实际上没有在 Yelp 的网页上，我仍然可以通过他们的 API 访问到他们的数据。

拥有一个 API 来收集你的数据非常有用，因为数据已经存在并准备好供你使用；根据公司的信誉，通常数据已经被清理并格式化得很好。这让你不必自己寻找数据，然后自己清理数据。然而，问题是，通常公司出于版权原因不允许你存储他们的数据，所以根据你的应用程序的用途，你可能需要考虑这个法律问题。

那么，如果没有可用的 API 供你使用，会发生什么呢？这时，你可能需要自己获取数据，而进行**网络爬虫**是完成这项任务的一种好方法。在下一节中，我会花大量时间解释网络爬虫的艺术以及如何进行爬虫操作。现在，让我们以讨论 API 经常返回数据的两种流行格式来结束这个简短的部分。

第一种是可扩展标记语言（XML），它是一种可读性强、以树形结构呈现的数据格式，实际上与 HTML 非常相似。举个例子，如果你调用 Facebook 图形 API，它会返回你的好友列表。这棵树的根可能有一个标签`<friends>`，下面可能有一系列的叶子标签`<friend>`。然后，每个`<friend>`节点可能会分支出几个描述符标签，如`<name>, <age>`等等。实际上，在后面的例子中，我会使用 XML 作为首选的数据格式，因为它易于阅读，这样你可以看到这种格式的真实例子。

下一种是**JavaScript 对象表示法（JSON）**，它比 XML 更轻量级。JSON 仍然可以被机器读取，但不如 XML 适合人类阅读。然而，其优点是解析 JSON 通常更高效，因此选择使用哪种格式实际上取决于人类可读性相对于性能的重要性。JSON 的一般结构类似于映射而非树形结构。使用前面的例子，而不是返回以`<friends>`为根节点的树形结构，我们可能会得到一个以`friends`为键，值为 JSON 数组的结构。该 JSON 数组将包含一系列的`friend`键，每个键的值都是一个 JSON 对象。最后，JSON 对象将包含等于`name, age`等键。换句话说，你可以将 JSON 结构看作是一系列嵌套的映射，很多时候键将指向一个子映射，该映射又有自己的键，依此类推。

当你使用第三方 API 时，通常需要知道它们选择以哪种数据格式返回数据，并相应地解析结果。此外，即使在你实现网络爬虫并需要构建自己的 API 时，通常也最好选择两种数据格式之一并坚持使用。这样，在从外部应用程序调用你的 API 并解析返回结果时，你的生活会简单得多。现在，让我们来谈谈网络爬虫。

# 网络爬虫入门

网页抓取是将网页 HTML 结构化，并系统地从中解析数据的艺术。这个想法是 HTML 应该在一定程度上固有地具有良好的结构，因为每个开放标签（即`<font>`）都应该有一个关闭标签（即`</font>`）相对应。这样，如果 HTML 结构正确，它可以被视为一个树状结构，非常类似于 XML。抓取一个网站可以通过多种方式实现，这通常与底层 HTML 源代码的复杂性有关，但在高层次上，它涉及三个步骤：

1.  获取所需的 URL，建立与 URL 的连接，并获取其源代码。

1.  组织和清理底层源代码，使其成为一个有效的 XML 文档。

1.  运行像 XPath（或 XQuery 和 XSLT）这样的树遍历语言，以及/或使用正则表达式（REGEX）来解析所需的节点。

第一步相对容易理解，但我需要指出一点。通常你会发现需要抓取某种动态网页，这意味着 URL 不是静态的，可能会根据日期、一组标准等而变化。让我们通过两个例子来解释我的意思。

第一个涉及股票。假设你正在尝试编写一个可以抓取给定股票当前价格的网页抓取器，比如从 Yahoo! Finance 获取。那么，首先，URL 是什么样子的？快速检查谷歌（股票代码为 GOOG）的当前价格，我们看到相应网页的 URL 是：

[`finance.yahoo.com/q?s=GOOG`](http://finance.yahoo.com/q?s=GOOG)

这是一个相当简单的 URL，我们会很快注意到股票代码作为参数传递给 URL。在这种情况下，参数的键为`s`，值等于股票代码。现在我们可以很容易地看出如何快速构建一个动态 URL 来解决问题——我们只需要编写以下简单的函数：

```kt
public void stockScraper(String ticker) {
String URL_BASE = "http://finance.yahoo.com/q?s=";
String STOCK_URL = URL_BASE + ticker;
// CONTINUE SCRAPING STOCK_URL
}

```

很整洁，对吧？现在假设我们不仅仅想要当前的股票价格，我们还想要获取两个日期之间的所有历史价格。首先，让我们看看一个示例 URL 是什么样的，再次以谷歌股票为例：

[`finance.yahoo.com/q/hp?s=GOOG&a=07&b=19&c=2004&d=02&e=14&f=2012`](http://finance.yahoo.com/q/hp?s=GOOG&a=07&b=19&c=2004&d=02&e=14&f=2012)

那么我们在这里注意到了什么？我们注意到股票代码仍然作为参数传递，键为`s`，除此之外我们还注意到似乎有两个不同的日期被传递，带有各种键。日期看起来像 07/19/2004，很可能是开始日期，以及 02/14/2012，看起来是结束日期，它们似乎有键值`a`到`f`。在这种情况下，键值并不是最直观的，而且很多时候你会看到键值为`day`或`d`以及`month`或`m`。然而，这个想法仅仅是通过这个 URL，你不仅可以动态调整股票代码，还可以根据用户需要查看的日期范围来调整这些日期。牢记这个想法，你会逐渐学会如何更好地解读各种 URL，并学会如何使它们极具动态性，适合你的抓取需求。

现在，一些网站通过 POST 请求发送请求。不同之处在于，在 POST 请求中，参数嵌入在请求中（而不是嵌入在 URL 中）。这样，潜在的私人数据就不会在 URL 中明显显示（尽管这只是使用 POST 请求的一个用例）。那么当这种情况发生时，我们应该怎么做呢？嗯，没有特别简单的方法。通常，你需要下载一个 HTTP 请求监听器（对于像 Chrome 和 Firefox 这样的浏览器，只需搜索 HTTP 请求监听器插件）。这将允许你看到正在进行的请求（包括 GET 和 POST 请求），以及传递的参数。一旦你知道了参数是什么，其余的工作就像 GET 请求一样进行。

现在，当我们有了 URL 之后，下一步就是获取底层的源代码并将其结构化。当然，自己来做这件事可能会很痛苦，但幸运的是，有一些库可以帮助我们清理和结构化源代码。我经常使用的一个库叫做**HtmlCleaner**，可以在以下 URL 找到：

[`htmlcleaner.sourceforge.net/`](http://htmlcleaner.sourceforge.net/)

这是一个很棒的库，它提供了清理和结构化源代码的方法，导航生成的 XML 文档，最终解析 XML 节点的值和属性。一旦我们的数据被清理，最后一步就是简单地遍历树并挑选出我们想要的数据片段。现在，说起来容易做起来难，因为仅使用 Java 及其本地包并没有真正简单的方法来系统地和可靠地遍历树。我所说的系统性和可靠性是指即使底层源代码的结构稍微有所变化，也能够遍历树并解析正确的数据。

例如，假设你的解析方法简单到告诉代码获取第五个节点的值。那么当 Yahoo!（或你正在抓取的任何网站）决定在其网站上添加一个新标题，现在第五个节点变成了第六个节点时，会发生什么？即使在这种对底层网站的相对简单的更改下，你的爬虫也会崩溃，并开始从错误的节点返回值，因此，我们理想上希望找到一种方法，无论底层网站如何变化，都能获取正确的节点值。

幸运的是，前端工程师经常会构建这样的网站：重要字段会拥有带有唯一值的`class`或`id`属性的标签。我们可以利用这些有帮助的、描述性的命名约定，使用一种名为**XPath**的便捷语言。一旦你了解了它，这门语言本身是相当容易解释的；实际上，它的语法类似于任何路径（即目录路径、URL 路径等），所以如果你愿意，我可以直接让你访问以下 URL 来了解其细节：

[`www.w3schools.com/xpath/`](http://www.w3schools.com/xpath/)

无论如何，现在只需记住 XPath 是一种允许你通过路径返回节点集的简单语言。XPath 的特殊之处在于，在路径中，你可以通过包含各种过滤器来进一步细化搜索，例如，只返回特定`class`的`div`。这就是具有描述性的`class`和`id`属性发挥作用的地方，因为我们可以深入 HTML 中，高效地找到只对我们重要的节点。此外，如果你还需要额外的工具来解析结果 XML，你可以使用正则表达式（REGEX）来帮助你搜索。

最终，目标是要使解析尽可能健壮，因为你们最不想做的就是随着底层网站的微小、不重要更改而不断更新爬虫。同样，有时网站会进行重大更改，你确实需要更新爬虫，但目标是要尽可能健壮地编写它们。

在这一点上，我相信你们一定有很多问题。代码实际上长什么样？如何获取一个网站的 HTML？如何使用`HtmlCleaner`库？XPath 的一个例子是什么？之前，我的目标是引导你们理解什么是网络爬虫，在此过程中，我介绍了很多不同的技术和方法。现在，让我们通过一些代码实操，看看前面的每个步骤。以下是抓取我们 Blockbuster 视频游戏数据的步骤一和步骤二：

```kt
public class HTMLNavigator {
// STEP 1 - GET THE URL'S SOURCE CODE
public static CharSequence navigateAndGetContents(String url_str) throws IOException {
URL url = new URL(url_str);
// ESTABLISH CONNECTION TO URL
URLConnection conn = url.openConnection();
conn.setConnectTimeout(30000);
String encoding = conn.getContentEncoding();
if (encoding == null) {
encoding = "ISO-8859-1";
}
// WRAP BUFFERED READER AROUND INPUT STREAM
BufferedReader br = new BufferedReader (new InputStreamReader(conn.getInputStream(), encoding));
StringBuilder sb = new StringBuilder();
try {
String line;
while ((line = br.readLine()) != null) {
sb.append(line);
sb.append('\n');
}
} finally {
br.close();
}
return sb;
}
}

```

首先，我们有一个简单的便捷类，可以获取传入 URL 的源代码。它只是打开一个连接，设置一些标准的网页参数，然后读取输入流。我们使用`StringBuilder`高效地构建一个包含输入流每一行的大字符串，并最终关闭所有连接并返回该字符串。这个字符串将是传入 URL 的底层 HTML，也是下一步构建干净、有序的 XML 文档所需的。下面是相应的代码：

```kt
import org.htmlcleaner.CleanerProperties;
import org.htmlcleaner.HtmlCleaner;
import org.htmlcleaner.TagNode;
import org.htmlcleaner.XPatherException;
import app.helpers.HTMLNavigator;
import app.types.VideoGame;
public class VideoGameScraper {
private static String content;
private static final String BASE_URL = "http://www.blockbuster.com/
games/platforms/gamePlatform";
/**
* QUERY FOR GAMES OF CERTAIN PLATFORM
*
* @param type
* the platform type
* @return
* @throws IOException
* @throws XPatherException
*/
public static List<VideoGame> getVideoGamesByConsole(String type) throws IOException, XPatherException {
// CONSTRUCT FULL URL
String query = BASE_URL + type;
// STEPS 1 + 2 - GET AND CLEAN THE DYNAMIC URL
TagNode node = getAndCleanHTML(query);
// STEP 3 - PARSE AND ADD GAMES
List<VideoGame> games = new ArrayList<VideoGame>();
. . .
return games;
}
/**
* CLEAN AND STRUCTURE THE PASSED IN HTML
*
* @param result
* the underlying html
* @return
* @throws IOException
*/
private static TagNode getAndCleanHTML(String result) throws IOException {
String content = HTMLNavigator.navigateAndGetContents(result). toString();
VideoGameScraper.content = content;
// USE HTMLCLEANER TO STRUCTURE HTML
HtmlCleaner cleaner = new HtmlCleaner();
CleanerProperties props = cleaner.getProperties();
props.setOmitDoctypeDeclaration(true);
return cleaner.clean(content);
}
.
.
.
}

```

在这里，我们首先编写一个简单的方法，允许我们连接到结果 URL 并获取其底层的源代码。然后我们取那个结果并传递给一个清理方法，该方法实例化我们`HtmlCleaner`类的新实例并调用`clean()`方法。这个方法将结构化底层的 HTML 成为一个格式良好的 XML 文档，并返回 XML 的根作为一个`TagNode`对象。最后一步只是查看底层的源代码，确定正确的 XPath 是什么，然后在给定的根`TagNode`上运行这些 XPath。Blockbuster 视频游戏租赁页面的缩略源代码如下所示：

```kt
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html lang="en" xml:lang="en" >
<head>
<body class="full">
<script type="text/javascript">
<div class="body clearDiv">
<div id="pageMask">&nbsp;</div>
<div id="boxPopup">&nbsp;</div>
<div id="head" class="head">
<style type="text/css">
<div>
<div id="gamesNav" class="secondaryNav">
<script type="text/javascript" language="javascript">
<div class="page clearDiv">
<div class="main contentsMain clearDiv">
<div class="primary clearDiv">
<span class="contentsDM"></span>
<span class="contentsLB"></span>
<img align="right" src="img/gameXboxOrig.gif" alt="Xbox Games">
<h1>Action &amp; Adventure Video Games</h1>
<div class="pagination">
<div class="gb6 listViewHeader">
<div class="">
<div id="4453" class="addToQueueEligible game sizeb gb6 bvr-gamelistitem ">
<mkt marketingitemid="4453" catalystinfo="A" listname="gameActionAdventure"></mkt>
<a onmouseover="if(DndUtil.windowLoaded){ new GameRollover(this); }" href="http:///games/catalog/gameDetails/4136" title="Superman Returns: The Video Game">
<img class="box" height="143" width="100" src="img/g25653wauzo. jpg?wid=100&hei=143">
</a>
<div class="details">
<h4>
<a onmouseover="if(DndUtil.windowLoaded){new GameRollover(this);}" href="http:///games/catalog/gameDetails/4136" title="Superman Returns: The Video Game">Superman Returns: The Video Game</a>
</h4>
<dl class="release">
<dl class="rated">
<div class="platform">
<dl class="movieInfo">
<div class="summary ">
<p class="readMore">
<div class="rolloverDetailsDiv" contentsrc="img/false">&nbsp;</div>
</div>
<div class="movieOptions">
<div id="movieRating" class="ratingWidget">
</div>
</div>
...

```

但是请注意，这段源代码是截至我撰写本文时的，不能保证保持不变。然而，从上面的源代码中，我们可以看到每个游戏都列在一个带有类`addToQueueEligible game sizeb gb6 bvr-gamelistitem`的`div`标签中。这是一个相当长的类名，但我们可以确信，通过搜索带有这个类标签的`divs`，我们会找到视频游戏，而且只有视频游戏，因为类标签涉及到将符合条件的游戏添加到队列中。

现在，一旦我们找到了那些想要的`divs`，我们会发现我们需要的节点只是第一个`a`节点，以及该`a`节点的相应`img`标签。因此，为了分别获取标题和图片 URL，我们想要的 XPath 应该如下所示：

```kt
//div[@class='addToQueueEligible game sizeb gb6 brv-gamelistitem']/a[1]
//div[@class='addToQueueEligible game sizeb gb6 brv-gamelistitem']/a[1]/img

```

有了这个，现在让我们看一下我们抓取器的完整代码：

```kt
import org.htmlcleaner.CleanerProperties;
import org.htmlcleaner.HtmlCleaner;
import org.htmlcleaner.TagNode;
import org.htmlcleaner.XPatherException;
import app.helpers.HTMLNavigator;
import app.types.VideoGame;
public class VideoGameScraper {
private static String content;
// XPATH FOR GETTING TITLE NAMES
private static String TITLE_EXPR = "//div[@class='%s']/a[1]";
// XPATH FOR GETTING IMAGE URLS
private static String IMG_EXPR = "//div[@class='%s']/a[1]/img";
// BASE OF BLOCKBUSTER URL
public static final String BASE_URL = "http://www.blockbuster.com/ games/platforms/gamePlatform";
/**
* QUERY FOR GAMES OF CERTAIN PLATFORM
*
* @param type
* the platform type
* @return
* @throws IOException
* @throws XPatherException
*/
public static List<VideoGame> getVideoGamesByConsole(String type) throws IOException, XPatherException {
// CONSTRUCT FULL URL
String query = BASE_URL + type;
// USE HTMLCLEANER TO STRUCTURE HTML
TagNode node = getAndCleanHTML(query);
// ADD GAMES
List<VideoGame> games = new ArrayList<VideoGame>();
games.addAll(grabGamesWithTag(node, "addToQueueEligible game sizeb gb6 bvr-gamelistitem ", type));
return games;
}
/**
* GIVEN THE STRUCTURED HTML, PARSE OUT NODES OF THE PASSED IN TAG
*
* @param head
* the head of the structured html
* @param tag
* the tag we are looking for
* @param type
* the platform type
* @return
* @throws XPatherException
*/
private static List<VideoGame> grabGamesWithTag(TagNode head, String tag, String type) throws XPatherException {
// RUN VIDEO GAME TITLE AND IMAGE XPATHS
Object[] gameTitleNodes = head.evaluateXPath(String.format (TITLE_EXPR, tag));
Object[] imgUrlNodes = head.evaluateXPath(String.format (IMG_EXPR, tag));
// ITERATE THROUGH VIDEO GAMES
List<VideoGame> games = new ArrayList<VideoGame>();
for (int i = 0; i < gameTitleNodes.length; i++) {
TagNode gameTitleNode = (TagNode) gameTitleNodes[i];
TagNode imgUrlNode = (TagNode) imgUrlNodes[i];
// BY LOOKING AT THE HTML, WE CAN DETERMINE
// WHICH ATTRIBUTES OF THE NODE TO PULL
String title = gameTitleNode.getAttributeByName("title");
String imgUrl = imgUrlNode.getAttributeByName("src");
// BUILD OUR VIDEO GAME OBJECT AND ADD TO LIST
VideoGame v = new VideoGame(title, imgUrl, type);
games.add(v);
}
return games;
}
/**
* CLEAN AND STRUCTURE THE PASSED IN HTML
*
* @param result
* the underlying html
* @return
* @throws IOException
*/
private static TagNode getAndCleanHTML(String result) throws IOException {
. . .
}
}

```

就这样！我们之前已经看过了大部分代码，所以实际上我们只需要关注`grabGamesWithTag()`方法。该方法的第一部分是将我们之前看到的 HTML 模式（网站的源代码）与我们的 XPath 格式结合起来。此时，我们有一个有效的 XPath，它将引导我们找到视频游戏的标题以及视频游戏的图片 URL。我们需要使用`HtmlCleaner`中的方法来实际运行这个 XPath 命令，如下所示：

```kt
Object[] gameTitleNodes = head.evaluateXPath(String.format (TITLE_EXPR, tag));

```

这将返回一个`Objects`列表，然后可以将其转换为单独的`TagNode`对象。然后我们需要做的是遍历数组中的每个`Object`，将其转换为`TagNode`，并提取节点的值或属性以获取所需的数据。我们可以在方法以下部分看到这一点：

```kt
// ITERATE THROUGH VIDEO GAMES
List<VideoGame> games = new ArrayList<VideoGame>();
for (int i = 0; i < gameTitleNodes.length; i++) {
TagNode gameTitleNode = (TagNode) gameTitleNodes[i];
TagNode imgUrlNode = (TagNode) imgUrlNodes[i];
// BY LOOKING AT THE HTML, WE CAN DETERMINE
// WHICH ATTRIBUTES OF THE NODE TO PULL
String title = gameTitleNode.getAttributeByName("title");
String imgUrl = imgUrlNode.getAttributeByName("src");
// BUILD OUR VIDEO GAME OBJECT AND ADD TO LIST
VideoGame v = new VideoGame(title, imgUrl, type);
games.add(v);
}

```

在这两种情况下，我们需要的是节点的特定属性值，而不是节点的值。如果是一个值的话，我们的代码看起来会更像下面这样：

```kt
List<VideoGame> games = new ArrayList<VideoGame>();
for (int i = 0; i < gameTitleNodes.length; i++) {
TagNode gameTitleNode = (TagNode) gameTitleNodes[i];
TagNode imgUrlNode = (TagNode) imgUrlNodes[i];
String title = gameTitleNode.getText().toString();
String imgUrl = imgUrlNode.getAttributeByName("src");
// BUILD OUR VIDEO GAME OBJECT AND ADD TO LIST
VideoGame v = new VideoGame(title, imgUrl, type);
games.add(v);
}

```

在这一点上，我们已经快速了解了网络爬取的基本知识。再次强调，网络爬取是一种技术和艺术，需要时间去适应和掌握，但它是一项非常棒的技术，可以为你打开无数的网络数据挖掘机会。现在，关注本章介绍的概念，而不是实际的代码。之所以这样说，是因为你的代码看起来会很大程度上取决于你试图爬取的网页。不会改变的是爬取背后的概念，因此使用本章提到的三个步骤作为指导，你可以为任何网页编写爬虫。

# 扩展 HTTP servlet 以支持 GET/POST 方法

现在我们已经编写好了网络爬虫，我们需要一种方法来处理返回的`VideoGame`对象，并将它们实际存储到我们的数据库中。此外，我们还需要一种方式与服务器通信，一旦服务器启动并运行，告诉它去抓取网站内容并插入到我们的 JDO 数据库中。我们与服务器通信的网关是通过所谓的 HTTP servlet，这在本书前面简要提到过。

以这种方式设置后端将在我们稍后讨论 CRON 作业时特别有用，这些作业为了自动运行某种功能，需要一个 servlet 与之通信（关于这一点我们很快会详细介绍）。现在，让我们看看如何扩展`HttpServlet`类并实现其`doGet()`方法，该方法将监听并处理所有发送给它的 HTTP GET 请求。但首先，HTTP GET 请求到底是什么？实际上，HTTP 网络请求只是用户向某个服务器发起的请求，并通过网络（即互联网）发送。根据请求类型，服务器将处理并回送 HTTP 响应给用户。然后，有两种常见的 HTTP 请求类型：

+   GET 请求——仅用于检索数据的网络请求。这些请求通常会要求服务器查询并返回某种数据。

+   POST 请求——提交数据处理的网络请求。通常，这会要求服务器插入用户提交的某种数据。

在这种情况下，由于我们既不需要获取用户数据，也不需要提交任何用户数据（实际上我们根本不与任何用户交互），使用哪种类型的请求实际上并没有区别，因此我们将使用更简单的 GET 请求，如下所示：

```kt
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
// EXTEND THE HTTPSERVLET CLASS TO MAKE THIS METHOD AVAILABLE
// TO EXTERNAL WEB REQUESTS, NAMELY CLIENTS AND CRON JOBS
public class VideoGameScrapeServlet extends HttpServlet {
private ArrayList<VideoGame> games;
/**
* METHOD THAT IS HIT WHEN HTTP GET REQUEST IS MADE
*
* @param request
* a servlet request object (any params passed can be retrieved
* with this)
* @param response
* a servlet response that you can embed data back to user
* @throws IOException
* @throws ServletException
*/
public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException {
games = new ArrayList<VideoGame>();
String message = "Success";
try {
// GRAB GAMES FROM ALL PLATFORMS
games.addAll(
VideoGameScraper.getVideoGamesByConsole(VideoGameConsole.DS));
games.addAll(
VideoGameScraper.getVideoGamesByConsole(VideoGameConsole.PS2));
games.addAll(
VideoGameScraper.getVideoGamesByConsole(VideoGameConsole.PS3));
games.addAll(
VideoGameScraper.getVideoGamesByConsole(VideoGameConsole.PSP));
games.addAll(
VideoGameScraper.getVideoGamesByConsole(VideoGameConsole.WII));
games.addAll(
VideoGameScraper.getVideoGamesByConsole(VideoGameConsole.XBOX));
} catch (Exception e) {
e.printStackTrace();
message = "Failed";
}
// HERE WE ADD ALL GAMES TO OUR VIDEOGAME JDO WRAPPER
VideoGameJDOWrapper.batchInsertGames(games);
// WRITE A RESPONSE BACK TO ORIGINAL HTTP REQUESTER
response.setContentType("text/html");
response.setHeader("Cache-Control", "no-cache");
response.getWriter().write(message);
}
}

```

因此，这个方法本身非常简单。我们之前已经有了`getVideoGamesByConsole()`方法，它会执行所有抓取操作，并返回一个`VideoGame`对象列表作为结果。然后，我们只需针对想要的所有游戏机运行它，最后使用我们巧妙的 JDO 数据库包装类，并调用其`batchInsertGames()`方法以快速插入数据。完成这些后，我们获取传入的 HTTP 响应对象，并快速向用户编写一些信息，以告知他们抓取是否成功。在这种情况下，我们没有使用传入的`HttpServletRequest`对象，但如果请求者在 URL 中传递参数，这个对象将非常有用。例如，假设你想以只抓取一个特定的游戏平台而非所有平台的方式来编写 Servlet。在这种情况下，你需要某种方式将平台类型参数传递给 Servlet，并在 Servlet 中提取传入的参数值。正如我们之前看到的雅虎财经允许你使用键值对`s`传递股票代码一样，为了传递平台类型，我们可以简单地执行以下操作：

```kt
http://{your-GAE-base-url}.appspot.com/videoGameScrapeServlet?type =Xbox

```

然后，在 Servlet 端执行以下操作：

```kt
public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException {
String type = request.getParameter("type");
games = new ArrayList<VideoGame>();
String message = "Success";
try {
// GRAB GAMES FROM SPECIFIC PLATFORM
games.addAll(VideoGameScraper.getVideoGamesByConsole(type));
} catch (Exception e) {
e.printStackTrace();
message = "Failed";
}
// ADD GAMES TO JDO DATABASE
VideoGameJDOWrapper.batchInsertGames(games);
// WRITE A RESPONSE BACK TO ORIGINAL HTTP REQUESTER
response.setContentType("text/html");
response.setHeader("Cache-Control", "no-cache");
response.getWriter().write(message);
}

```

很简单，对吧？你只需要确保 URL 中使用的键与 Servlet 类中请求的参数相匹配。现在，为了将所有这些连接在一起，最后一步是在你的 GAE 项目中定义 URL 路径——即确保你的 GAE 项目知道 URL 模式实际上指向你刚刚编写的这个类。这可以在你的 GAE 项目的`/war/WEB-INF/`目录中找到，具体是在`web.xml`文件中。你需要在其中添加以下内容，以确保 Servlet 名称和类路径与给定的 URL 模式相匹配：

```kt
<?xml version="1.0" encoding="utf-8"?>
<web-app  version="2.5">
<servlet>
<servlet-name>videoGameScrapeServlet</servlet-name>
<servlet-class>app.httpservlets.VideoGameScrapeServlet</servlet-class>
</servlet>
<servlet-mapping>
<servlet-name>videoGameScrapeServlet</servlet-name>
<url-pattern>/videoGameScrapeServlet</url-pattern>
</servlet-mapping>
</web-app>

```

在这一点上，我们有了抓取器，我们有了 JDO 数据库，甚至我们的第一个 Servlet 也已经连接并准备就绪。最后一部分是定期安排你的抓取器运行；这样，你的数据库就有最新的、最及时的数据，而不需要你每天坐在电脑前手动调用你的抓取器。在下一节中，我们将看到如何使用 CRON 作业来实现这一点。

# 安排 CRON 作业

首先，让我们定义一下 CRON 作业是什么。**cron**这个术语最初指的是 Unix 中基于时间的工作调度程序，允许你安排作业/脚本在特定时间定期运行。同样的概念可以应用于网页请求，而在我们的情况下，目标是定期运行我们的网页抓取器并更新数据库中的数据，而无需我们的干预。GAE 平台之所以方便使用，另一个原因是它让安排 CRON 作业变得非常简单。为此，我们只需在 GAE 项目的`/war/WEB-INF/`目录中创建一个`cron.xml`文件。在这个 XML 文件中，我们添加以下代码：

```kt
<?xml version="1.0" encoding="UTF-8"?>
<cronentries>
<cron>
<url>/videoGameScrapeServlet</url>
<description>Scrape video games from Blockbuster</description>
<schedule>every day 00:50</schedule>
<timezone>America/Los_Angeles</timezone>
</cron>
</cronentries>

```

这相当于是自解释的。首先，我们定义了一个名为`<cronentries>`的根标签，在这些标签内，我们可以插入任意数量的`<cron>`标签——每个标签都表示一个计划中的进程。在这些`<cron>`标签中，我们需要告诉调度程序我们想要访问的 URL 是什么（这当然会相对于根 URL），以及计划本身（在我们的案例中，是每天凌晨 12:50）。其他可选标签有描述标签、时区标签和/或目标标签，允许你指定调用指定 URL 的 GAE 项目的哪个版本。

在我的案例中，我让调度程序每天太平洋标准时间凌晨 12:50 运行该任务，但其他调度格式的例子如下：

```kt
every 12 hours
every 5 minutes from 10:00 to 14:00
2nd,third mon,wed,thu of march 17:00
every monday 09:00
1st monday of sep,oct,nov 17:00
every day 00:00

```

我不会深入讲解调度标签的确切语法，但你可以看出它非常直观。然而，对于那些想要在 GAE 中了解有关 CRON 作业更多信息或者查看一些较少使用的功能的人，可以随时查看以下 URL 以全面了解 CRON 作业：

[`code.google.com/appengine/docs/java/config/cron.html`](http://code.google.com/appengine/docs/java/config/cron.html)

但就我们的示例而言，我们之前所做的工作已经足够，因此我们就此打住！

# 总结

在本章中，我们再次涉猎了很多内容。我们从探讨收集数据的各种方法开始本章。在有些情况下，其他公司发布的便捷 API 可供我们使用和查询（尽管在存储这些数据时必须注意法律问题）。然而，很多时候我们需要自己出去抓取数据，这可以通过网页抓取完成。

在下一节中，我们通过一个网页抓取入门教程进行了学习——从网页抓取是什么以及执行抓取需要采取哪些步骤的高层次概念开始，到具体实现结束。我们通过抓取 Blockbuster 网站获取可供租借的最新视频游戏为例进行了学习，在这个过程中，我们编写了我们的第一个 XPath 表达式并实现了第一个 HTTP servlet。

在实现我们的 HTTP servlet 时，我们简要讨论了两种常见的 HTTP 请求类型（GET 和 POST 请求），然后实现了一个 HTTP GET 请求，这将允许我们调用我们的视频游戏抓取器类，收集聚合的`VideoGame`对象，并使用前一章中的便捷包装类将它们插入到我们的 JDO 数据库中。

最后，我们通过探讨如何安排对 Blockbuster 网站的抓取来结束本章，以确保获取最新和最及时的数据，而无需我们每天手动调用抓取器。我们介绍了一种称为 CRON 作业的特殊技术，并使用 GAE 平台实现了一个。

在下一章也是最后一章中，我们将尝试把所学的一切融合在一起。更具体地说，既然我们系统的数据收集和插入部分已经运行起来了，我们将实现几个额外的 servlet，使我们能够发起 HTTP GET 请求并检索各种类型的数据。接下来，我们将研究代码的客户端部分，看看如何从 Android 应用程序发起这些 GET 请求并解析响应以获取需要的数据。
