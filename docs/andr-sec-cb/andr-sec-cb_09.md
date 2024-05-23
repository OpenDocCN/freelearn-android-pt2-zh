# 第九章：加密和开发设备管理策略

在本章中，我们将涵盖以下内容：

+   使用加密库

+   生成对称加密密钥

+   保护 SharedPreferences 数据

+   基于密码的加密

+   使用 SQLCipher 加密数据库

+   安卓 KeyStore 提供者

+   设置设备管理策略

# 引言

本章节的主要焦点将是如何正确使用加密技术，以在设备上安全地存储数据。我们从创建一个一致的加密基础开始，包括我们自己的加密实现库，以在旧设备上支持更强的加密算法。

我们要解决的一个直接问题就是生成对称加密密钥；然而，默认设置并不总是更安全。我们将查看具体参数以确保最强加密，并回顾一个常见的反模式和一个限制生成密钥安全的操作系统漏洞。

然后，我们探讨了多种使用第三方库或称为**Android KeyStore**的系统服务来安全存储加密密钥的方法，该服务在安卓 4.3 中引入。更进一步，我们学习如何完全避免在设备上存储密钥，使用密钥派生函数从用户的密码或 PIN 码生成密钥。  

我们将介绍如何有效地集成 SQLCipher，以确保你的应用程序的 SQLite 数据库得到加密，从而显著提高你的应用数据的安全性。

我们将以设备管理 API 结束本章，该 API 旨在让企业实施设备策略和保护措施，进一步保护设备。我们实施了两项虚构（但合理）的企业政策，以确保设备启用了加密存储并满足锁屏超时要求。

# 使用加密库

安卓使用 Java 作为核心编程语言的好处之一是它包含了**Java 加密扩展**（**JCE**）。JCE 是一套成熟、经过测试的安全 API。安卓使用 Bouncy Castle 作为这些 API 的开源实现。然而，Bouncy Castle 的版本在安卓版本之间有所不同；只有较新的安卓版本才能获得最新的修复。为了减少 Bouncy Castle 的大小，安卓定制了 Bouncy Castle 库并移除了一些服务和 API。例如，如果你打算使用**椭圆曲线密码学**（**ECC**），在低于 4.0 的安卓版本上运行时，你会看到提供者错误。另外，尽管 Bouncy Castle 支持 AES-GCM 方案（我们将在下一个食谱中介绍），但在安卓上使用它必须单独包含。

为了解决这个问题，我们可以包含特定于应用程序的加密库实现。本指南将展示如何包含 Spongy Castle 库，它相对于 Android 的 Bouncy Castle 实现来说更加更新，提供了更高层次的安全，并支持更多的加密选项。

你可能会想“为什么使用 Spongy Castle 而不是直接包含 Bouncy Castle 库”。原因是 Android 已经包含了一个较旧的 Bouncy Castle 库版本，因此我们需要重命名这个库的包以避免“类加载器”冲突。所以，Spongy Castle 实际上是 Bouncy Castle 的重新打包。实际上，只要包名与`org.bouncycastle`不同，它可以是任何你想要的名称。

## 如何操作...

让我们在 Android 应用程序中添加 Spongy Castle。

1.  从[`github.com/rtyley/spongycastle/#downloads`](https://github.com/rtyley/spongycastle/#downloads)下载最新的 Spongy Castle 二进制文件。

    查阅 MIT X11 许可证（与 Bouncy Castle 相同），以确保这与你打算使用的方式兼容。

1.  在你应用程序的`/libs`目录中提取并复制 Spongy Castle 的`.jar`文件：

    +   `sc-light-jdk15on`：核心轻量级 API

    +   `scprov-jdk15on`：JCE 提供者（需要`sc-light-jdk15on`）

1.  在你的 Android 应用程序对象中包含以下`static`代码块：

    ```kt
    static {
      Security.insertProviderAt(new org.spongycastle.jce.provider.BouncyCastleProvider(), 1);
    }
    ```

## 工作原理...

我们使用静态代码块来调用`Security.insertProviderAt()`。它确保我们捆绑在应用程序`/libs`文件夹中的 Spongy Castle 提供者优先使用。通过设置为`1`的位置，我们确保它优先于现有的安全提供者。

使用 Spongy Castle 与 JCE 的妙处在于，无需修改现有的加密代码。在本章中，我们展示了可以与 Bouncy Castle 或 Spongy Castle 同样良好工作的加密代码示例。

## 还有更多...

如前所述，代码可以从 GitHub 下载；但是，你也可以构建自己的版本。Spongy Castle 仓库的所有者*Roberto Tyley*包含了一个`become-spongy.sh` bash 脚本，该脚本将`com.bouncycastle`重命名为`com.spongycastle`。因此，你可以将其用于自己刚刚下载并更新版本的 Bouncy Castle 库，并将其转换为`org.spongycastle`或其他同样可爱且吸引人的名称。

### 注意

`become-spongy.sh` bash 脚本可以在[`gist.github.com/scottyab/8003892`](https://gist.github.com/scottyab/8003892)找到

## 另请参阅

+   《生成对称加密密钥》和《基于密码的加密》食谱演示了使用 JCE API 的方法

+   Spongy Castle 的 GitHub 仓库在[`rtyley.github.io/spongycastle/#downloads`](http://rtyley.github.io/spongycastle/#downloads)

+   Bouncy Castle 的主页在[`www.bouncycastle.org/java.html`](http://www.bouncycastle.org/java.html)

+   访问[`www.owasp.org/index.php/Using_the_Java_Cryptographic_Extensions`](https://www.owasp.org/index.php/Using_the_Java_Cryptographic_Extensions)的*使用 Java 加密扩展* OWASP 社区页面

# 生成对称加密密钥

对称密钥是指用于加密和解密的同一个密钥。为了在一般情况下创建加密安全的加密密钥，我们使用安全生成的伪随机数。这个方法演示了如何正确初始化`SecureRandom`类，以及如何用它来初始化**高级加密标准**（**AES**）的加密密钥。AES 是比 DES 更受欢迎的加密标准，通常与 128 位和 256 位的密钥大小一起使用。

### 注意事项

如前一个菜谱所述，无论您是使用 Bouncy Castle 还是 Spongy Castle，代码上没有差异。

## 如何操作...

让我们创建一个安全的加密密钥。

1.  编写以下函数以生成对称 AES 加密密钥：

    ```kt
    public static SecretKey generateAESKey(int keysize)
          throws NoSuchAlgorithmException {
        final SecureRandom random = new SecureRandom();

        final KeyGenerator generator = KeyGenerator.getInstance("AES");
        generator.init(keysize, random);
        return generator.generateKey();
      }
    ```

1.  创建一个匹配 256 位 AES 密钥大小的 32 字节随机初始化向量（IV）：

    ```kt
    private static IvParameterSpec iv;

    public static IvParameterSpec getIV() {
        if (iv == null) {
          byte[] ivByteArray = new byte[32];
          // populate the array with random bytes
          new SecureRandom().nextBytes(ivByteArray);
          iv = new IvParameterSpec(ivByteArray);
        }
        return iv;
      }
    ```

1.  编写以下函数以加密任意字符串：

    ```kt
    public static byte[] encrpyt(String plainText)
        throws GeneralSecurityException, IOException {
        final Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
        cipher.init(Cipher.ENCRYPT_MODE, getKey(), getIV());
        return cipher.doFinal(plainText.getBytes("UTF-8"));
      }

      public static SecretKey getKey() throws NoSuchAlgorithmException {
        if (key == null) {
          key = generateAESKey(256);
        }
        return key;
      }
    ```

1.  为了完整性，前面的代码片段展示了如何解密。唯一的不同是，我们使用`Cipher.DECRYPT_MODE`常量调用`Cipher.init()`方法：

    ```kt
    public static String decrpyt(byte[] cipherText)
          throws GeneralSecurityException, IOException {
          final Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
          cipher.init(Cipher.DECRYPT_MODE, getKey(),getIV());
          return cipher.doFinal(cipherText).toString();
        }
    ```

在此示例中，我们只是将密钥和 IV 作为静态变量存储；在实际使用中并不建议这样做。一个简单的方法是将密钥以`SharedPerferences`方式持久化，并使用`Context.MODE_PRIVATE`标志，以便在应用程序会话之间提供一致的密钥。下一个菜谱进一步开发这个想法，使用`SharedPerferences`的加密版本。

## 它是如何工作的…

创建`SecureRandom`对象只需实例化默认构造函数即可。还有其他构造函数可用；然而，默认构造函数使用的是可用的最强提供者。我们将`SecureRandom`的实例传递给`KeyGenerator`类，并带上`keysize`参数，`KeyGenerator`类负责创建对称加密密钥。256 位通常被认为是“军用级别”，对于大多数系统来说，它被认为是加密安全的。

在这里，我们引入一个初始化向量（IV），简而言之，它增加了加密的强度，并且在加密多个消息/项目时至关重要。这是因为使用相同密钥加密的消息可以一起分析，以帮助提取消息。弱 IV 是**有线等效隐私**（**WEP**）被破解的部分原因。因此，建议为每条消息生成一个新的 IV，并将其与密文一起存储；例如，你可以在密文前预先追加或连接 IV。

在实际的加密过程中，我们使用`Cipher`对象的 AES 实例，以新生成的`SecretKey`在`ENCRYPT_MODE`模式下初始化。然后我们调用`cipher.doFinal`方法，传入明文字节，以返回包含加密字节的字节数组。

当使用`Cipher`对象请求 AES 加密模式时，一个常见的疏忽（在安卓文档中也存在）是简单地使用`AES`。然而，这默认为最简单且安全性较低的 ECB 模式，具体为`AES/ECB/PKCS7Padding`。因此，我们应该明确请求更强大的 CBC 模式`AES/CBC/PKCS5Padding`，如示例代码所示。

## 还有更多...

在这里，我们探讨如何使用一种称为**AES-GCM**的强加密模式，以及一个常见的反模式，该反模式降低了生成的密钥的安全性。

### 使用 AES-GCM 进行强对称加密

我们注意到，简单地定义`AES`并不会默认为最强模式。如果我们包含 Spongy Castle 库，我们可以使用更强大的 AES-GCM，它包括验证，并且可以检测密文是否被篡改。要在定义算法/转换字符串时使用 AES-GCM，请使用如下代码所示的`AES/GCM/NoPadding`：

```kt
  final Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding", "SC");
```

### 反模式——设置种子

自从安卓 4.2 版本以来，`SecureRandom`的默认**伪随机数生成器**（**PRNG**）提供者被改为 OpenSSL。这使得 Bouncy Castle 提供者之前存在的手动设置`SecureRandom`对象种子的能力被禁用。这是一个受欢迎的变化，因为开发者设置种子的反模式已经出现。

```kt
byte[] myCustomSeed = new byte[] { (byte) 42 };
secureRandom.setSeed(myCustomSeed);
int notRandom = secureRandom.nextInt();
```

在这个代码示例中，我们可以看到种子被手动设置为`42`，结果是`notRandom`变量总是等于同一个数字。尽管这对于单元测试很有用，但这破坏了使用`SecureRandom`生成加密密钥的任何增强安全性。

### 安卓的伪随机数生成器（PRNG）漏洞

如前所述，自安卓 4.2 以来，伪随机数生成器（PRNG）的默认提供者是 OpenSSL。然而，在 2013 年 8 月，发现了一个生成随机数的严重错误。这通过几个安卓比特币钱包应用的妥协得到了突出。这个问题涉及到安全随机数生成器的种子设置；它没有使用复杂且独特的系统指纹，而是被初始化为 null。其结果与之前从可预测数字生成的安全密钥的反模式类似。受影响的安卓版本包括 Jelly Bean 4.1、4.2 和 4.3。

在《*关于 SecureRandom 的一些思考*》的安卓博客文章中记录了一个修复方法，并提交给了 Open Handset Alliance 公司。然而，建议您从应用程序的`onCreate()`方法中调用此修复，以防该修复尚未应用到运行您应用程序的设备上。

### 注意事项

为了方便起见，这里提供了一个来自 GitHub 的 gist，其中包含了谷歌的代码，可以在[`gist.github.com/scottyab/6498556`](https://gist.github.com/scottyab/6498556)找到。

## 另请参阅

+   *保护 SharedPreferences 数据*的配方，我们使用了生成的 AES 密钥来加密应用程序的 SharedPreferences

+   [《安卓应用中密码学误用实证研究》](http://cs.ucsb.edu/~yanick/publications/2013_ccs_cryptolint.pdf)指南

+   [Android 开发者参考指南中的`SecureRandom`类](https://developer.android.com/reference/java/security/SecureRandom.html)

+   [Android 开发者参考指南中的`KeyGenerator`类](https://developer.android.com/reference/javax/crypto/KeyGenerator.html)

+   [《关于 SecureRandom 的一些思考》Android 博客文章](http://android-developers.blogspot.co.uk/2013/08/some-securerandom-thoughts.html)

+   [开放手持设备联盟成员](http://www.openhandsetalliance.com/oha_members.html)

# 保护 SharedPreferences 数据

Android 为应用开发者提供了一个简单的框架，用于持久化存储基本数据类型的键值对。这个菜谱展示了伪随机生成的密钥的实际用途，并演示了**Secure-Preferences**的使用。它是一个开源库，包装了默认的 Android SharedPreferences 以加密键值对，从而保护它们免受攻击者的侵害。Secure-Preferences 兼容 Android 2.1+，并使用 Apache 2.0 许可，因此适合商业开发。

我应该补充一下，我是 Secure-Preferences 库的共同创建者和维护者。Secure-Preferences 的一个很好的替代品是名为**Cwac-prefs**的库，它由 SQLCipher 支持（在后面的菜谱中介绍）。

## 准备就绪

让我们添加 Secure-Preferences 库。

1.  从 GitHub 下载或克隆 Secure-Preferences，地址是[`github.com/scottyab/secure-preferences`](https://github.com/scottyab/secure-preferences)。

    Secure-Preferences 仓库包含一个 Android 库项目和示例项目。

1.  就像通常那样，将库链接到你的 Android 项目中。

## 如何操作...

让我们开始吧。

1.  使用 Android `context`简单初始化`SecurePreferences`对象：

    ```kt
    SharedPreferences prefs = SecurePreferences(context);

    Editor edit = prefs.edit();
    edit.putString("pref_fav_book", "androidsecuritycookbook");
    edit.apply();
    ```

1.  下面是一些你可以添加到你的应用程序中的辅助方法，以便在你的应用程序对象中获取（安全的）偏好设置实例：

    ```kt
    private SharedPreferences mPrefs;
    public final SharedPreferences getSharedPrefs() {
        if (null == mPrefs) {
          mPrefs = new SecurePreferences(YourApplication.this);
        }
        return mPrefs;
      }
    ```

    在这里，`YourApplication.this`是对你的应用程序对象的引用。

1.  然后，理想情况下，在一个基础的应用程序组件中，如`BaseActivity`、`BaseFragment`或`BaseService`，你可以包含以下内容以获取（安全的）偏好设置对象的实例：

    ```kt
    private SharedPreferences mPrefs;
    protected final SharedPreferences getSharedPrefs() {
        if (null == mPrefs) {
          mPrefs = YourApplication.getInstance().getSharedPrefs();
        }
        return mPrefs;
      }
    ```

## 它是如何工作的...

Secure-Preferences 库实现了`SharedPreferences`接口；因此，与默认的 SharedPreferences 相比，与它交互不需要进行代码更改。

标准的 SharedPreferences 键和值存储在一个简单的 XML 文件中，Secure-Preferences 使用相同的存储机制；不同之处在于，键和值会使用 AES 对称密钥进行透明加密。在写入文件之前，键和值的密文会使用 base64 编码。

如果你检查以下 SharedPreference XML 文件，你会看到使用和不使用 Secure-Preferences 库的区别。你会看到来自 Secure-Preferences 库的文件是一系列看似随机的条目，这些条目无法揭示其用途。

+   一个标准的 SharedPreferences XML 文件：

    ```kt
    <?xml version='1.0' encoding='utf-8' standalone='yes' ?>
    <map>
    <int name="timeout " value="500" />
    <boolean name="is_logged_in" value="true" />
    <string name="pref_fav_book">androidsecuritycookbook</string>
    </map>
    ```

+   使用 Secure-Preferences 库的 SharedPreferences XML 文件：

    ```kt
    <?xml version='1.0' encoding='utf-8' standalone='yes' ?>
    <map>
    <string name="MIIEpQIBAAKCAQEAyb6BkBms39I7imXMO0UW1EDJsbGNs">
    HhiXTk3JRgAMuK0wosHLLfaVvRUuT3ICK
    </string>
    <string name="TuwbBU0IrAyL9znGBJ87uEi7pW0FwYwX8SZiiKnD2VZ7"> va6l7hf5imdM+P3KA3Jk5OZwFj1/Ed2
    </string>
    <string name="8lqCQqn73Uo84Rj">k73tlfVNYsPshll19ztma7U">
    tEcsr41t5orGWT9/pqJrMC5x503cc=
    </string>
    </map>
    ```

第一次实例化`SecurePreferences`时，会生成一个 AES 加密密钥并存储。这个密钥用于加密/解密通过标准`SharedPreferences`接口保存的所有将来的键/值。

共享首选项文件是使用`Context.MODE_PRIVATE`创建的，这强制执行应用沙箱安全，确保只有你的应用可以访问。然而，在已 root 的设备上，不能依赖沙箱安全。更准确地说，Secure-Preferences 是在混淆首选项；因此，这不应当被视为坚不可摧的安全措施。相反，将其视为一种快速获胜的方法，逐步提高 Android 应用的安全性。例如，它将阻止已 root 设备上的用户轻松修改你应用的 SharedPreferences。

Secure-Preferences 可以进一步增强，通过使用一种称为**基于密码的加密**（**PBE**）的技术，根据用户输入的密码生成密钥，这将在下一章中介绍。

## 另请参阅

+   Android 开发者参考指南中的`SharedPreferences`接口位于[`developer.android.com/reference/android/content/SharedPreferences.html`](https://developer.android.com/reference/android/content/SharedPreferences.html)

+   *丹尼尔·亚伯拉罕*关于 Secure-Preferences 的文章，位于[`www.codeproject.com/Articles/549119/Encryption-Wrapper-for-Android-SharedPreferences`](http://www.codeproject.com/Articles/549119/Encryption-Wrapper-for-Android-SharedPreferences)

+   Secure-Preferences 库位于[`github.com/scottyab/secure-preferences`](https://github.com/scottyab/secure-preferences)

+   CWAC-prefs 库（Secure-Preferences 的替代品）位于[`github.com/commonsguy/cwac-prefs`](https://github.com/commonsguy/cwac-prefs)

# 基于密码的加密

加密面临的一个较大问题是密钥的管理和安全存储。到目前为止，在之前的食谱中，我们接受将密钥存储在 SharedPreferences 中，正如谷歌开发者的博客所推荐的；然而，这对于已获得 root 权限的设备来说并不理想。在已 root 的设备上，你不能依赖 Android 系统的安全沙箱，因为 root 用户可以访问所有区域。这意味着，与未 root 的设备不同，其他应用可以获得提升的 root 权限。

在不安全的 app 沙盒环境中，基于密码的加密（PBE）是一个理想的选择。它提供了在运行时使用由用户通常提供的密码/密码来创建（或更准确地说是派生）加密密钥的能力。

另一个密钥管理的解决方案是使用系统密钥链；Android 的版本称为 Android KeyStore，我们将在后面的食谱中进行审查。

## 准备就绪

PBE 是 Java 加密扩展的一部分，因此它已经包含在 Android SDK 中。

在这个食谱中，我们将使用初始化向量（IV）和**盐值**作为密钥派生的一部分。我们在上一个食谱中介绍了 IV，它有助于创建更多的随机性。因此，即使使用相同的密钥加密相同的消息，也会产生不同的密文。盐值与 IV 相似，它通常是一个随机数据，作为加密过程的一部分添加，以提高其加密强度。

## 如何操作...

让我们开始吧。

1.  首先，我们定义一些辅助方法来获取或创建 IV 和盐值。我们将它们作为密钥派生和加密的一部分来使用：

    ```kt
      private static IvParameterSpec iv;

      public static IvParameterSpec getIV() {
        if (iv == null) {
          iv = new IvParameterSpec(generateRandomByteArray(32));
        }
        return iv;
      }

      private static byte[] salt;

      public static byte[] getSalt() {
        if (salt == null) {
          salt = generateRandomByteArray(32);
        }
        return salt;
      }

      public static byte[] generateRandomByteArray(int sizeInBytes) {
        byte[] randomNumberByteArray = new byte[sizeInBytes];
        // populate the array with random bytes using non seeded secure random
        new SecureRandom().nextBytes(randomNumberByteArray);
        return randomNumberByteArray;
      }
    ```

1.  生成 PBE 密钥：

    ```kt
    public static SecretKey generatePBEKey(char[] password, byte[] salt)
          throws NoSuchAlgorithmException, InvalidKeySpecException {

        final int iterations = 10000;
        final int outputKeyLength = 256;

        SecretKeyFactory secretKeyFactory = SecretKeyFactory
            .getInstance("PBKDF2WithHmacSHA1");
        KeySpec keySpec = new PBEKeySpec(password, salt, iterations, outputKeyLength);
        SecretKey secretKey = secretKeyFactory.generateSecret(keySpec);
        return secretKey;
      }
    ```

1.  编写一个示例方法，展示如何使用新派生的 PBE 密钥进行加密：

    ```kt
    public static byte[] encrpytWithPBE(String painText, String userPassword)
          throws GeneralSecurityException, IOException {

        SecretKey secretKey = generatePBEKey(userPassword.toCharArray(),getSalt());

        final Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
        cipher.init(Cipher.ENCRYPT_MODE, secretKey, getIV());
        return cipher.doFinal(painText.getBytes("UTF-8"));
      }
    ```

1.  编写一个示例方法，展示如何使用新派生的 PBE 密钥解密密文：

    ```kt
    public static String decrpytWithPBE(byte[] cipherText, String userPassword)
          throws GeneralSecurityException, IOException {

        SecretKey secretKey = generatePBEKey(userPassword.toCharArray(),getSalt());

        final Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
        cipher.init(Cipher.DECRYPT_MODE, secretKey, getIV());
        return cipher.doFinal(cipherText).toString();
      }
    ```

## 它是如何工作的...

在步骤 1 中，我们定义了类似于以前食谱中使用的方法。再次强调，为了能够解密加密数据，盐值和 IV 必须保持一致。例如，你可以为每个 app 生成一个盐值并将其存储在`SharedPreferences`中。此外，盐值的大小通常与密钥大小相同，在这个例子中是 32 字节/256 位。通常，你会在解密时将 IV 和密文一起保存以便检索。

在步骤 2 中，我们使用用户的密码通过 PBE 派生一个 256 位的 AES `SecretKey`。`PBKDF2`是一种常用于从用户密码派生密钥的算法；Android 对该算法的实现被记为`PBKDF2WithHmacSHA1`。

作为`PBEKeySpec`的一部分，我们定义了在`SecretKeyFactory`内部使用的迭代次数，以生成密钥。迭代次数越多，密钥派生所需的时间就越长。为了防御暴力破解攻击，建议派生密钥的时间应超过 100 毫秒；Android 使用 10,000 次迭代来生成加密备份的加密密钥。

步骤 3 和 4 演示了如何使用`Cipher`对象和密钥进行加密和解密；你会注意到，这些方法与之前食谱中记录的方法非常相似。但当然，对于解密，IV 和盐值不是随机生成的，而是从加密步骤中重新使用。

## 还有更多…

在 Android 4.4 中，处理`PBKDF2WithHmacSHA1`和 Unicode 密码短语时，对`SecretKeyFactory`类进行了细微的更改。以前，`PBKDF2WithHmacSHA1`只查看密码短语中 Java 字符的低位 8 位；对`SecretKeyFactory`类的更改允许使用 Unicode 字符的所有可用位。为了保持向后兼容性，你可以使用这个新的密钥生成算法`PBKDF2WithHmacSHA1And8bit`。如果你使用 ASCII，这个更改不会影响你。

下面是一个如何保持向后兼容的代码示例：

```kt
SecretKeyFactory secretKeyFactory;
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
secretKeyFactory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1And8bit");
} else {
secretKeyFactory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1");
}
```

## 另请参阅

+   [Android 开发者参考指南中的`SecretKeyFactory`类](https://developer.android.com/reference/javax/crypto/SecretKeyFactory.html)

+   [Android 开发者参考指南中的`PBEKeySpec`类](https://developer.android.com/reference/javax/crypto/spec/PBEKeySpec.html)

+   [Java 加密体系结构（JCA）参考指南中的 Java 加密扩展](http://docs.oracle.com/javase/6/docs/technotes/guides/security/crypto/CryptoSpec.html)

+   [Android 开发者博客中的*使用加密安全存储凭据*](http://android-developers.blogspot.co.uk/2013/02/using-cryptography-to-store-credentials.html)

+   由*Nikolay Elenkov*提供的示例 PBE 项目，位于[`github.com/nelenkov/android-pbe`](https://github.com/nelenkov/android-pbe)

+   [Android 4.4 中对`SecretKeyFactory` API 的更改](http://android-developers.blogspot.co.uk/2013/12/changes-to-secretkeyfactory-api-in.html)

# 使用 SQLCipher 加密数据库

SQLCipher 是 Android 应用中实现安全存储的最简单方法之一，它兼容运行 Android 2.1+的设备。SQLCipher 使用 256 位 AES 加密每个数据库页面，采用 CBC 模式；此外，每个页面都有自己的随机初始化向量，以进一步提高安全性。

SQLCipher 是 SQLite 数据库的一个独立实现，它没有实现自己的加密，而是使用了广泛使用和测试的 OpenSSL `libcrypto`库。虽然这确保了更高的安全性和更广泛的兼容性，但它确实伴随着大约 7MB 的相对较大的`.apk`文件体积。这额外的重量可能是使用 SQLCipher 的唯一缺点。

根据 SQLCipher 网站的说法，在读写性能方面，大约有 5%的性能损失是微不足道的，除非你的应用正在执行复杂的 SQL 连接（但值得注意的是，这些在 SQLite 中也不太好）。对于商业开发来说，好消息是 SQLCipher for Android 不仅是开源的，而且还是基于 BSD 风格的许可发布的。

## 准备工作

首先，我们将下载并设置你的 Android 项目以使用 SQLCipher。

1.  通过 SQLCipher GitHub 页面上的链接下载最新的二进制包，或者直接使用这个链接[`s3.amazonaws.com/sqlcipher/SQLCipher+for+Android+v3.0.0.zip`](https://s3.amazonaws.com/sqlcipher/SQLCipher+for+Android+v3.0.0.zip)。

1.  解压 ZIP 文件。

1.  从`/asset`s 目录将`icudt46l.zip`文件复制到应用程序的`/assets`目录。

1.  `/libs`目录包含几个 JAR 文件和包含本地库的文件夹。

1.  将`*.jar`文件复制到你的应用程序的`/libs`目录。你可能已经在使用 Commons-codec 和/或 guava；如果是这样，请检查版本是否与 SQLCipher 兼容。

1.  本地代码的 ARM 和 x86 实现均已包含；然而，你可能只需要基于 ARM 的本地库。因此，请将`armeabi`文件夹复制到你的应用程序的`/libs`目录下。

## 如何操作...

让我们创建一个加密的 SQLite 数据库。

1.  处理 SQLite 数据库有几种方式，可以直接与`SQLiteDatabase`对象合作，也可以使用`SQLiteOpenHelper`。但是，通常如果你在应用中已经使用了 SQLite 数据库，只需将`import android.database.sqlite.*`声明替换为`import net.sqlcipher.database.*`即可。

1.  创建加密的 SQLCipher 数据库最简单的方式是使用密码调用`openOrCreateDatabase(…)`：

    ```kt
    private static final int DB_VERSION = 1;
      private static final String DB_NAME = "my_encrypted_data.db";

      public void initDB(Context context, String password) {
             SQLiteDatabase.loadLibs(context);
           SQLiteDatabase database = SQLiteDatabase.openOrCreateDatabase(DB_NAME, password, null);
              database.execSQL("create table MyTable(a, b)");

      }
    ```

1.  如果你正在使用`SQLiteOpenHelper`对象，你可能已经对其进行了扩展。在这个例子中，我们将假设你的扩展名为`SQLCipherHelper`。当你调用`getWritableDatabase`时，你会注意到需要传递一个字符串参数（数据库密码短语）与 SQLCipher 版本的`SQLiteOpenHelper`：

    ```kt
    import net.sqlcipher.database.SQLiteOpenHelper;

    public class SQLCipherHelper extends SQLiteOpenHelper {
    private static final int DB_VERSION = 1;

    private static final String DB_NAME = "my_encrypted_data.db";

    public SQLCipherHelper (Context context) {
        super(context, DB_NAME, null, DB_VERSION);
        SQLiteDatabase.loadLibs(context);

    }
    }
    ```

### 提示

在使用`SQLiteDatabase.loadLibs(context)`语句完成任何数据库操作之前，需要加载 SQLCipher 本地库。理想情况下，此调用应位于内容提供者或应用程序对象的`onCreate`生命周期方法中。

## 工作原理…

示例代码展示了与 SQLite 数据库合作的两种最常见方式：直接使用`SQLiteDatabase`对象或使用`SQLiteOpenHelper`。

需要注意的主要区别在于使用`net.sqlcipher.database` API 与默认 SQLite API 之间的区别在于，在创建或检索 SQLCipher 数据库对象时使用密码短语。SQLCipher 使用`PBKDF2`派生加密密钥，如前一个配方所介绍。在撰写本书时，默认配置生成了一个使用 4,000 次迭代的 256 位 AES 密钥。开发者需要决定如何生成密码短语。你可以基于每个应用使用 PRNG 生成，或者为了更大的随机性和安全性由用户输入。SQLCipher 使用派生的密钥透明地加密和解密。它还使用消息验证码（MAC）来确保数据的完整性和真实性，确保数据没有被意外或恶意篡改。

## 还有更多内容...

值得注意的是，由于 SQLCipher 的大部分代码是用本地 C/C++编写的，因此它与其他平台（如 Linux、Windows、iOS 和 Mac OS）兼容。

### IOCipher

将 IOCipher 视为来自 Guardian 项目的 SQLCipher 久未联系的表亲。它提供了挂载加密虚拟文件系统的能力，允许开发者在他们的应用目录中透明地加密所有文件。与 SQLCipher 一样，IOCipher 依赖开发者管理密码，支持 Android 2.1+。

IOCipher 的一个巨大优势是它是`java.io` API 的一个克隆。这意味着从集成的角度来看，对现有的文件管理代码进行修改很少。不同之处在于，首先需要使用密码挂载文件系统，并且不是使用`java.io.File`，而是使用`info.guardianproject.iocipher.File`。

尽管 IOCipher 使用了 SQLCipher 的部分内容，但它还不够成熟，但如果你希望保护的是文件而不是 SQLite 数据库中的数据，那么它值得研究。

## 另请参阅

+   SQLCipher 下载地址为[`sqlcipher.net/downloads/`](http://sqlcipher.net/downloads/)

+   SQLCipher for Android 的源代码位于[`github.com/sqlcipher/android-database-sqlcipher`](https://github.com/sqlcipher/android-database-sqlcipher)

+   *IOCipher: Virtual Encrypted Disks*项目位于[`guardianproject.info/code/iocipher/`](https://guardianproject.info/code/iocipher/)

# Android KeyStore 提供者

在 Android 4.3 中，新增了一个功能，允许应用将私有的加密密钥保存在系统**密钥库**中。这个被称为 Android KeyStore 的功能只允许创建它们的应用访问，并且使用设备 PIN 码进行保护。

特别地，Android KeyStore 是一个证书存储，因此只能存储公钥/私钥。目前，无法存储诸如 AES 密钥之类的任意对称密钥。在 Android 4.4 中，**椭圆曲线数字签名算法**（**ECDSA**）支持被添加到 Android KeyStore 中。本食谱讨论如何生成新密钥，以及如何将其保存和从 Android KeyStore 中获取。

## 准备开始

由于这个特性是在 Android 4.3 中添加的，请确保在 Android 清单文件中将最低 SDK 版本设置为`18`。

## 如何操作...

让我们开始吧。

1.  创建一个指向应用 KeyStore 的句柄：

    ```kt
    public static final String ANDROID_KEYSTORE = "AndroidKeyStore";

      public void loadKeyStore() {
        try {
          keyStore = KeyStore.getInstance(ANDROID_KEYSTORE);
          keyStore.load(null);
        } catch (Exception e) {
          // TODO: Handle this appropriately in your app
          e.printStackTrace();
        }
      }
    ```

1.  生成并保存应用的关键对：

    ```kt
      public void generateNewKeyPair(String alias, Context context)
          throws Exception {

        Calendar start = Calendar.getInstance();
        Calendar end = Calendar.getInstance();
        // expires 1 year from today
        end.add(1, Calendar.YEAR);

        KeyPairGeneratorSpec spec = new KeyPairGeneratorSpec.Builder(context)
    .setAlias(alias)
    .setSubject(new X500Principal("CN=" + alias))
    .setSerialNumber(BigInteger.TEN)
    .setStartDate(start.getTime())
    .setEndDate(end.getTime())
    .build();

        // use the Android keystore
        KeyPairGenerator gen = KeyPairGenerator.getInstance("RSA", ANDROID_KEYSTORE);
        gen.initialize(spec);

        // generates the keypair
        gen.generateKeyPair();
      }
    ```

1.  使用给定的别名检索密钥：

    ```kt
      public PrivateKey loadPrivteKey(String alias) throws Exception {

        if (keyStore.isKeyEntry(alias)) {
          Log.e(TAG, "Could not find key alias: " + alias);
          return null;
        }

        KeyStore.Entry entry = keyStore.getEntry(KEY_ALIAS, null);

        if (!(entry instanceof KeyStore.PrivateKeyEntry)) {
          Log.e(TAG, " alias: " + alias + " is not a PrivateKey");
          return null;
        }

        return ((KeyStore.PrivateKeyEntry) entry).getPrivateKey();
      }
    ```

## 它是如何工作的...

`KeyStore`类自 API 级别 1 以来就已经存在。要访问新的 Android KeyStore，你可以使用一个特殊的常量`"AndroidKeystore"`。

根据谷歌的文档，`KeyStore`类有一个奇怪的问题，即使你不是从输入流加载`KeyStore`，也需要调用`load(null)`方法；否则，你可能会遇到崩溃的情况。

在生成密钥对时，我们使用所需的详细信息填充`KeyPairGeneratorSpec.Builder`对象，包括我们稍后用于检索它的别名。在这个例子中，我们从当前日期开始设置了一个任意的验证期限为`1`年，并将序列号默认为`TEN`。

从别名加载密钥就像加载`keyStore.getEntry("alias", null)`一样简单；从这里，我们将其转换为`PrivateKey`接口，以便我们可以在加密/解密中使用它。

## 还有更多...

在 Android 4.3 中，`KeyChain`类的 API 也得到了更新，允许开发者确定设备是否支持硬件支持的证书存储。这基本上意味着设备支持证书存储的安全元素。这是一个令人兴奋的增强功能，因为它承诺即使在根设备上也能保持证书存储的安全。然而，并不是所有设备都支持这个硬件特性。流行的设备 LG Nexus 4 使用 ARM 的 TrustZone 进行硬件保护。

## 另请参阅

+   在 Android 开发者参考指南中的`KeyStore`类，可以在[`developer.android.com/reference/java/security/KeyStore.html`](https://developer.android.com/reference/java/security/KeyStore.html)找到

+   KeyStore API 示例可以在[`developer.android.com/samples/BasicAndroidKeyStore/index.html`](https://developer.android.com/samples/BasicAndroidKeyStore/index.html)找到

+   *Nikolay Elenkov*撰写的《Android 4.3 中的凭据存储增强》一文可以在[`nelenkov.blogspot.co.uk/2013/08/credential-storage-enhancements-android-43.html`](http://nelenkov.blogspot.co.uk/2013/08/credential-storage-enhancements-android-43.html)找到

+   ARM TrustZone 可以在[`www.arm.com/products/processors/technologies/trustzone/index.php`](http://www.arm.com/products/processors/technologies/trustzone/index.php)找到

# 设置设备管理策略

Device Admin 策略最早在 Android 2.2 中引入，它赋予应用程序更大的设备控制能力。这些功能主要针对企业应用开发者，因为它们具有控制性、限制性，可能具有破坏性，并提供了一种替代第三方**移动设备管理**（**MDM**）解决方案的方法。通常，这不是针对消费者应用，除非已经存在信任关系，例如银行和银行应用。

本食谱将定义两个旨在加强设备的设备策略，这可能是企业移动安全政策的一部分：

+   强制执行设备加密（这也确保设置了设备 PIN 码/密码）

+   强制执行最大屏幕锁定超时

尽管设备加密不能替代确保应用程序数据正确加密，但它确实增加了整体设备安全性。减少最大屏幕锁定超时有助于在设备无人看管时保护设备。

对执行设备策略的应用程序数量没有限制。如果策略上有冲突，系统默认采用最安全的策略。例如，如果密码强度要求策略上有冲突，将应用最严格的策略以满足所有策略。

## 准备就绪

设备管理员策略在 2.2 版本中添加，但是，此功能以及对设备加密的具体限制直到 Android 3.0 才添加。因此，对于此食谱，请确保你针对的是高于 API 11 的 SDK。

## 如何操作...

让我们开始吧。

1.  通过在`res/xml`文件夹中创建名为`admin_policy_encryption_and_lock_timeout.xml`的新`.xml`文件来定义设备管理策略，内容如下：

    ```kt
    <device-admin  >
        <uses-policies>
            <force-lock />
            <encrypted-storage />
        </uses-policies>
    </device-admin>
    ```

1.  创建一个扩展了`DeviceAdminReceiver`类的类。这是与设备管理相关的系统广播的应用程序入口点：

    ```kt
    public class AppPolicyReceiver extends DeviceAdminReceiver {

      // Called when the app is about to be deactivated as a device administrator.
      @Override
      public void onDisabled(Context context, Intent intent) {
        // depending on your requirements, you may want to disable the // app or wipe stored data e.g clear prefs
        context.getSharedPreferences(context.getPackageName(),
            Context.MODE_PRIVATE).edit().clear().apply();
        super.onDisabled(context, intent);
      }

      @Override
      public void onEnabled(Context context, Intent intent) {
        super.onEnabled(context, intent);

        // once enabled enforce
        AppPolicyController controller = new AppPolicyController();
        controller.enforceTimeToLock(context);

        controller.shouldPromptToEnableDeviceEncrpytion(context);
      }

      @Override
      public CharSequence onDisableRequested(Context context, Intent intent) {
        // issue warning to the user before disable e.g. app prefs // will be wiped
        return context.getText(R.string.device_admin_disable_policy);
      }
    }
    ```

1.  在你的 Android 清单文件中添加接收者定义：

    ```kt
    <receiver
           android:name="YOUR_APP_PGK.AppPolicyReceiver"
           android:permission="android.permission.BIND_DEVICE_ADMIN" >
           <meta-data
             android:name="android.app.device_admin"
             android:resource="@xml/admin_policy_encryption_and_lock_timeout" />

           <intent-filter>
             <action android:name="android.app.action.DEVICE_ADMIN_ENABLED" />
             <action android:name="android.app.action.DEVICE_ADMIN_DISABLED" />
             <action android:name="android.app.action.DEVICE_ADMIN_DISABLE_REQUESTED" />
           </intent-filter>
    </receiver>
    ```

    定义接收者使得`AppPolicyReceiver`能够接收系统广播意图，以禁用/请求禁用管理员设置。你应该注意到，这是我们通过文件名`admin_policy_encryption_and_lock_timeout`在元数据中引用策略 XML 文件的地方。

1.  设备策略控制器处理与`DevicePolicyManager`的通信以及任何特定于应用程序的逻辑。我们定义的第一个方法是供其他应用程序组件（如活动）验证设备管理员状态，并获得特定于设备管理员的意图：

    ```kt
    public class AppPolicyController {

      public boolean isDeviceAdminActive(Context context) {
        DevicePolicyManager devicePolicyManager = (DevicePolicyManager) context
            .getSystemService(Context.DEVICE_POLICY_SERVICE);

        ComponentName appPolicyReceiver = new ComponentName(context,
            AppPolicyReceiver.class);

        return devicePolicyManager.isAdminActive(appPolicyReceiver);
      }
      public Intent getEnableDeviceAdminIntent(Context context) {

        ComponentName appPolicyReceiver = new ComponentName(context,
            AppPolicyReceiver.class);

        Intent activateDeviceAdminIntent = new Intent(
            DevicePolicyManager.ACTION_ADD_DEVICE_ADMIN);

        activateDeviceAdminIntent.putExtra(
            DevicePolicyManager.EXTRA_DEVICE_ADMIN, appPolicyReceiver);

        // include optional explanation message
        activateDeviceAdminIntent.putExtra(
            DevicePolicyManager.EXTRA_ADD_EXPLANATION,
            context.getString(R.string.device_admin_activation_
    message));

        return activateDeviceAdminIntent;
      }

    public Intent getEnableDeviceEncryptionIntent() {
        return new Intent(DevicePolicyManager.ACTION_START_ENCRYPTION);
      }
    ```

1.  在`AppPolicyController`中，我们现在定义了实际执行锁定屏幕超时的方法。我们随意选择了`3`分钟的最大锁定时间，但这应该与企业安全政策保持一致：

    ```kt
      private static final long MAX_TIME_TILL_LOCK = 3 * 60 * 1000;

      public void enforceTimeToLock(Context context) {
        DevicePolicyManager devicePolicyManager = (DevicePolicyManager) context
            .getSystemService(Context.DEVICE_POLICY_SERVICE);

        ComponentName appPolicyReceiver = new ComponentName(context,
            AppPolicyReceiver.class);

        devicePolicyManager.setMaximumTimeToLock(appPolicyReceiver,
            MAX_TIME_TILL_LOCK);
      }
    ```

1.  根据设备的硬件和外部存储大小，加密设备可能需要一些时间。作为执行设备加密政策的一部分，我们需要一种方法来检查设备是否已加密或加密是否正在进行中：

    ```kt
    public boolean shouldPromptToEnableDeviceEncryption(Context context) {
        DevicePolicyManager devicePolicyManager = (DevicePolicyManager) context
            .getSystemService(Context.DEVICE_POLICY_SERVICE);
        int currentStatus = devicePolicyManager.getStorageEncryptionStatus();
        if (currentStatus == DevicePolicyManager.ENCRYPTION_STATUS_INACTIVE) {
          return true;
        }
        return false;
      }
    }
    ```

1.  我们定义了一个示例活动，以展示如何集成 `AppPolicyController` 以帮助指导用户启用系统设置并处理响应：

    ```kt
    public class AppPolicyDemoActivity extends Activity {

      private static final int ENABLE_DEVICE_ADMIN_REQUEST_CODE = 11;
      private static final int ENABLE_DEVICE_ENCRYPT_REQUEST_CODE = 12;
      private AppPolicyController controller;
      private TextView mStatusTextView;

      @Override
      public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_app_policy);
        mStatusTextView = (TextView) findViewById(R.id.deviceAdminStatus);

        controller = new AppPolicyController();

        if (!controller.isDeviceAdminActive(getApplicationContext())) {
          // Launch the activity to have the user enable our admin.
          startActivityForResult(
              controller
                  .getEnableDeviceAdminIntent(getApplicationContext()),
              ENABLE_DEVICE_ADMIN_REQUEST_CODE);
        } else {
          mStatusTextView.setText("Device admin enabled, yay!");
          // admin is already activated so ensure policies are set
          controller.enforceTimeToLock(getApplicationContext());
          if (controller.shouldPromptToEnableDeviceEncrpytion(this)) {
            startActivityForResult(
                controller.getEnableDeviceEncrpytionIntent(),
                ENABLE_DEVICE_ENCRYPT_REQUEST_CODE);
          }
        }

      }
    ```

1.  在这里，我们实现了 `onActivityResult(…)` 活动生命周期方法，以处理在启用设备管理和加密时，来自系统活动的结果：

    ```kt
      @Override
      protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == ENABLE_DEVICE_ADMIN_REQUEST_CODE) {
          if (resultCode != RESULT_OK) {
            handleDevicePolicyNotActive();
          } else {
            mStatusTextView.setText("Device admin enabled");
            if (controller.shouldPromptToEnableDeviceEncrpytion(this)) {
              startActivityForResult(
                  controller.getEnableDeviceEncryptionIntent(),
                  ENABLE_DEVICE_ENCRYPT_REQUEST_CODE);
            }
          }

        } else if (requestCode == ENABLE_DEVICE_ENCRYPT_REQUEST_CODE
            && resultCode != RESULT_OK) {
          handleDevicePolicyNotActive();
        }
      }
    ```

1.  最后，我们添加了一个方法来处理如果用户选择不将此应用作为设备管理员激活的情况。在这个示例中，我们只是简单地发布了一条消息；然而，你可能会阻止应用运行，因为设备不符合企业安全策略：

    ```kt
      private void handleDevicePolicyNotActive() {
        Toast.makeText(this, R.string.device_admin_policy_breach_message,
            Toast.LENGTH_SHORT).show();
      }
    }
    ```

## 它的工作原理...

`AppPolicyDemoActivity` 展示了一个处理用户交互和回调的例子，这些回调来自系统活动，用于启用设备管理和设备加密的 `onActivityResult(…)`。

`AppPolicyController` 封装了与 `DevicePolicyManager` 的交互，并包含了应用策略的逻辑。你可以在你的活动或片段中找到这段代码，但更好的做法是将它独立出来。

定义策略就像在设备管理员文件中的 `<uses-policies>` 元素中定义它们一样简单。这是在 Android 清单文件中 `AppPolicyReceiver` XML 声明的元数据元素中引用的：

```kt
<meta-data  android:name="android.app.device_admin"                android:resource="@xml/admin_policy_encryption_and_lock_timeout" />
```

由于设备管理员具有提升的权限，出于安全考虑，应用在安装时不会作为设备管理员启用。这是通过使用内置系统活动实现的，该活动通过使用具有特殊动作 `AppPolicyController.getEnableDeviceAdminIntent()` 的意图请求，如所示。这个活动通过 `startActivityForResult()` 启动，它将回调返回到 `onActivityResult(…)`，用户可以选择激活或取消。设备管理员的非激活可能被视为违反企业安全策略。因此，如果用户没有激活它，可能足以简单地阻止用户使用应用，直到它被激活。

我们使用 `DevicePolicyManager.isActive(…)` 方法来检查应用是否作为设备管理员激活。通常，这个检查应该在应用程序的入口点执行，比如第一个活动。

`AppPolicyReceiver` 的工作是监听设备管理系统的活动。为了接收这些事件，首先你必须扩展 `DeviceAdminReceiver` 并在 Android 清单文件中定义 `Receiver`。在 `OnEnabled()` 回调中，我们强制执行锁屏超时，因为它不需要额外的用户输入。启用设备加密需要用户确认；因此，我们从活动中启动这个过程。

如果用户将此应用程序作为设备管理员禁用，`AppPolicyReceiver`也将收到`onDisabled`事件。如前所述，当用户将应用作为设备管理员禁用时，不同应用的处理方式会有所不同，这取决于企业安全政策。还有一个`onDisableRequested`回调方法，允许我们向用户显示特定信息，详细说明禁用应用程序的后果。在这个例子中，我们会清除 SharedPreferences，以确保在设备不符合要求时数据不会处于风险之中。

## 还有更多...

除了此食谱中使用的策略外，设备管理员还可以强制执行以下操作：

+   启用密码

+   密码复杂性（从 3.0 版本开始增加了更多控制）

+   自 3.0 版本以来的密码历史

+   在恢复出厂设置之前允许的最大密码失败尝试次数

+   擦除设备（恢复出厂设置）

+   锁定设备

+   禁用锁屏小部件（自 4.2 版本起）

+   禁用摄像头（自 4.0 版本起）

用户无法卸载处于活动状态的设备管理员应用。要卸载，他们必须首先将应用作为设备管理员停用，然后再卸载。这允许你在`DeviceAdminReceiver.onDisabled()`中执行任何必要的功能，例如，向远程服务器报告事件。

Android 4.4 引入了一个可选的设备管理功能常量，可以在 app 的`manifest.xml`文件中的`<uses-feature>`标签中使用，这表明应用需要设备管理功能，并确保在 Google Play 商店正确筛选。

### 禁用设备摄像头

Android 4.0 增加的一个有趣功能是能够禁用摄像头使用。这对于希望限制数据泄露的组织可能很有用。以下代码段展示了启用应用禁用摄像头使用的策略：

```kt
<device-admin  >
    <uses-policies>
        <disable-camera />
    </uses-policies>
</device-admin>
```

## 另请参阅

+   Android 开发者参考资料中的设备管理 API，参考链接：[`developer.android.com/guide/topics/admin/device-admin.html`](https://developer.android.com/guide/topics/admin/device-admin.html)

+   设备管理员示例应用程序，参考链接：[`developer.android.com/guide/topics/admin/device-admin.html#sample`](https://developer.android.com/guide/topics/admin/device-admin.html#sample)

+   在 Android 开发者培训指南中的*通过设备管理策略增强安全性*网页，参考链接：[`developer.android.com/training/enterprise/device-management-policy.html`](https://developer.android.com/training/enterprise/device-management-policy.html)

+   在 Android 开发者参考资料中的`FEATURE_DEVICE_ADMIN`，参考链接：[`developer.android.com/reference/android/content/pm/PackageManager.html#FEATURE_DEVICE_ADMIN`](https://developer.android.com/reference/android/content/pm/PackageManager.html#FEATURE_DEVICE_ADMIN)
