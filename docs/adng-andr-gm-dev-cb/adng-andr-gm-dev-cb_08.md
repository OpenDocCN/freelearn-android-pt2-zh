# 第八章：最大化的性能

在本章中，我们将介绍一些提高 AndEngine 应用程序性能的最佳实践。包括以下主题：

+   忽略实体更新

+   禁用背景窗口渲染

+   限制同时播放的音轨数量

+   创建精灵池

+   使用精灵组减少渲染时间

+   禁用实体剔除的渲染

# 引言

游戏优化在 Google Play 上游戏成功中起着关键作用。如果游戏在用户设备上运行不佳，用户很可能会给出负面评价。不幸的是，由于存在许多不同的设备，而且无法在 Google Play 上有效地大规模限制低端设备，因此最好尽可能优化 Android 游戏。忽略评分，可以公平地说，如果游戏在中端设备上的表现不佳，那么在下载和活跃用户方面将无法达到其全部潜力。本章将介绍一些与 AndEngine 性能问题相关的最有帮助的解决方案。这将帮助我们提高中低端设备的性能，无需牺牲质量。

### 注意

尽管本章中的方法可以大幅提高我们游戏的性能，但重要的是要记住，清晰高效的代码同样重要。游戏开发是一项非常注重性能的任务，与所有语言一样，有许多小事要做或避免。网上有许多资源涵盖了关于 Java 通用实践以及 Android 特定技巧的好坏话题。

# 忽略实体更新

在优化游戏方面，游戏开发最重要的规则之一是，*不要做不需要做的工作！*。在本节中，我们将讨论如何使用`setIgnoreUpdate()`方法在我们的实体上，以限制更新线程只更新应该更新的内容，而不是不断更新所有实体，不管我们是否使用它们。

## 如何操作…

以下`setIgnoreUpdate(boolean)`方法允许我们控制哪些实体将通过引擎的更新线程进行更新：

```kt
Entity entity = new Entity();

// Ignore updates for this entity
entity.setIgnoreUpdate(true);

// Allow this entity to continue updating
entity.setIgnoreUpdate(false);
```

## 工作原理…

如前几章所述，每个子对象的`onUpdate()`方法都是通过其父对象调用的。引擎首先更新，调用主`Scene`对象的更新方法。然后场景继续调用其所有子对象的更新方法。接下来，场景的子对象将分别调用其子对象的更新方法，依此类推。考虑到这一点，通过在主 Scene 对象上调用`setIgnoreUpdate()`，我们可以有效地忽略场景上所有实体的更新。

忽略未使用实体的更新，或者除非发生特定事件否则不做出反应的实体，可以节省大量的 CPU 时间。这对于包含大量实体的场景尤为如此。这可能看起来工作量不大，但请记住，对于每个带有实体修改器或更新处理器的实体，这些对象也必须更新。除此之外，每个实体的子实体也会因为父/子层次结构而继续更新。

最佳实践是为所有屏幕外的或不需要持续更新的实体设置`setIgnoreUpdate(true)`。对于可能根本不需要任何更新的精灵，比如场景的背景精灵，我们可以无限期地忽略更新，而不会造成任何问题。在实体需要更新，但不是非常频繁的情况下，例如从炮塔发射的子弹，我们可以在子弹从炮塔飞向目标的过程中启用更新，在不再需要时禁用它。

## 另请参阅

+   第二章中的*了解 AndEngine 实体*部分，*使用实体*

# 禁用后台窗口渲染

在大多数游戏中，开发者通常更倾向于使用全屏模式。虽然从视觉上看我们并没有发现明显的差异，但安卓操作系统并不会识别哪些应用程序是在全屏模式下运行的。这意味着除非在`AndroidManifest.xml`中另外指定，否则后台窗口将继续在我们的应用程序下方绘制。在本主题中，我们将介绍如何禁用后台渲染以提高应用程序的帧率，这主要有利于低端设备。

## 准备工作...

为了停止后台窗口的渲染，我们首先需要为应用程序创建一个主题。我们将在项目的`res/values/`文件夹中添加一个名为`theme.xml`的新 xml 文件来实现这一点。

用以下代码覆盖默认 xml 文件中的所有内容，并保存文件：

```kt
<?xml version="1.0" encoding="UTF-8"?>
<resources>
    <style name="Theme.NoBackground" parent="android:Theme">
        <item name="android:windowBackground">@null</item>
    </style>
</resources>
```

## 如何操作...

创建并填写完`theme.xml`文件后，我们可以在项目的`AndroidManifest.xml`文件中，将主题应用于我们的应用程序标签，从而禁用后台窗口渲染。应用程序标签的属性可能看起来类似于这样：

```kt
<application
        android:theme="@style/Theme.NoBackground"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name" 
        >
```

请注意，我们也可以将主题应用于特定的活动，而不是在整个应用程序范围内应用，只需在各个活动标签中添加`android:theme="@style/Theme.NoBackground"`代码即可。这对于需要同时使用 AndEngine 视图和原生安卓视图的混合游戏来说最为相关，这些视图跨越了多个活动。

## 工作原理...

禁用背景窗口渲染是一个简单的任务，主要在旧设备上可以提供一些百分比的性能提升。负责背景窗口的主要代码在`theme.xml`文件中找到。通过将`android:windowBackground`项设置为 null，我们通知设备，我们希望完全移除背景窗口的渲染，而不是绘制它。

# 限制同时播放的音轨数量

在 AndEngine 中，声音播放通常不会成为游戏性能的问题。然而，在某些情况下，大量声音可能在非常短的时间内播放，这可能会在旧设备上有时甚至在新设备上造成明显的延迟，这取决于同时播放的声音数量。默认情况下，AndEngine 允许同一`Sound`对象在任何给定时间同时播放五个音轨。在本主题中，我们将通过操作`EngineOptions`来更改同时播放的音轨数量，以更好地满足我们应用程序的需求。

## 如何操作...

为了增加或减少每个`Sound`对象的同时播放音轨数量，我们必须在活动的`onCreateEngineOptions()`方法中对`EngineOptions`进行简单的调整：

```kt
@Override
public EngineOptions onCreateEngineOptions() {
  mCamera = new Camera(0, 0, 800, 480);

  EngineOptions engineOptions = new EngineOptions(true,
                ScreenOrientation.LANDSCAPE_FIXED, new 
                FillResolutionPolicy(),mCamera);

  engineOptions.getAudioOptions().setNeedsSound(true);
  engineOptions.getAudioOptions().getSoundOptions().setMaxSimultaneousStreams(2);

  return engineOptions;
}
```

## 工作原理…

默认情况下，`Engine`对象的`AudioOptions`设置为每个`Sound`对象允许同时播放五个音轨。在大多数情况下，这对于不重度依赖声音播放的应用程序来说，不会造成明显的性能损失。另一方面，倾向于在碰撞或施加力时产生声音的游戏可能会同时播放大量音轨，特别是在任何给定时间场景中有超过 100 个精灵的游戏中。

限制同时播放的音轨数量是一个容易完成的任务。只需在我们的`EngineOptions`上调用`getAudioOptions().getSoundOptions().setMaxSimultaneousStreams(n)`，其中`n`是每个`Sound`对象的最大音轨数量，我们就可以减少在游戏过程中不适宜的时候播放的不必要声音。

## 另请参阅

+   第一章中的*引入声音和音乐*部分，*AndEngine 游戏结构*

# 创建精灵池

`GenericPool`类在考虑到移动平台在硬件资源上的限制时，是 AndEngine 游戏设计中极其重要的部分。在 Android 游戏开发中，要实现长时间游戏体验的流畅，关键在于尽可能少地创建对象。这并不意味着我们应该将屏幕上的对象限制在四个或五个，而是应该考虑回收已经创建的对象。这时对象池就派上用场了。

## 开始操作…

请参考代码包中名为`SpritePool`的类。

## 如何操作…

`GenericPool`类使用了一些有用的方法，使得回收对象以供后续使用变得非常简单。我们将在这里介绍主要使用的方法。

构造`SpritePool`类：

```kt
public SpritePool(ITextureRegion pTextureRegion, VertexBufferObjectManager pVertexBufferObjectManager){
  this.mTextureRegion = pTextureRegion;
  this.mVertexBufferObjectManager = pVertexBufferObjectManager;
}
```

1.  分配池项目：

    ```kt
    @Override
    protected Sprite onAllocatePoolItem() {
      return new Sprite(0, 0, this.mTextureRegion, this.mVertexBufferObjectManager);
    }
    ```

1.  获取池项目：

    ```kt
    public synchronized Sprite obtainPoolItem(final float pX, final float pY) {
      Sprite sprite = super.obtainPoolItem();

      sprite.setPosition(pX, pY);
      sprite.setVisible(true);
      sprite.setIgnoreUpdate(false);  
      sprite.setColor(1,1,1);

      return sprite;
    }
    ```

1.  回收池项目：

    ```kt
    @Override
    protected void onHandleRecycleItem(Sprite pItem) {
      super.onHandleRecycleItem(pItem);

      pItem.setVisible(false);
      pItem.setIgnoreUpdate(true);
      pItem.clearEntityModifiers();
      pItem.clearUpdateHandlers();
    }
    ```

## 工作原理...

`GenericPool`类的理念非常简单。当我们需要对象时，不是创建新对象并在用完后丢弃它们，而是可以告诉池分配有限数量的对象并存储起来以供后续使用。我们现在可以从池中调用`obtainPoolItem()`方法，以获取存储分配的对象之一，在我们的关卡中使用，例如作为敌人。一旦这个敌人被玩家摧毁，我们现在可以调用`recyclePoolItem(pItem)`将这个敌人对象送回池中。这使我们能够避免垃圾收集的调用，并有可能大大减少新对象所需的内存。

在*如何操作...*部分中的四种方法，对于使用普通池来说已经足够。显然，我们必须在使用之前创建池。然后，以下三种方法定义了对象分配、获取对象使用以及对象回收时会发生什么，或者在我们用完后将其送回池中存储，直到我们需要新对象。尽管对象池不仅仅用于精灵对象的回收，但我们会更深入地了解每个方法的用途、工作原理以及原因，从构造函数开始。

在第一步中，我们必须传递给池对象构造函数所需的任何对象。在这种情况下，我们需要获取`TextureRegion`和`VertexBufferObjectManager`以创建 Sprite 对象。这并不是什么新知识，但请记住，`GenericPool`类不仅限于创建精灵的池。我们可以为任何类型的对象或数据类型创建池。关键是要使用池的构造函数作为获取传递给池对象分配所需参数的方法。

在第二步中，我们覆盖了`onAllocatePoolItem()`方法。当池需要分配新对象时，它将调用此方法。两种情况是：池最初没有对象，或者所有回收的对象都已获取并在使用中。我们在这个方法中需要处理的是返回对象的新实例。

第三步涉及到使用`obtain`方法从对象池中获取一个对象，以便在我们的游戏中使用。我们可以看到，在这种情况下，`obtainPoolItem()`方法要求我们传入`pX`和`pY`参数，这些参数将被精灵的`setPosition(pX, pY)`方法使用，以重新定位精灵。然后我们将精灵的`visibility`设置为`true`，允许更新精灵，以及将颜色设置回初始值白色。在任何情况下，此方法应用于将对象的值重置为默认状态，或者定义对象必要的*新*属性。在代码中，我们可能会像以下代码片段所示从对象池中获取一个新的精灵：

```kt
// obtain a sprite and attach it to the scene at position (10, 10)
Sprite sprite = pool.obtainPoolItem(10, 10);
mScene.attachChild(sprite);
```

在最后的方法中，我们将从`GenericPool`类中使用`recyclePoolItem(pItem)`方法，其中`pItem`是要回收回对象池中的对象。此方法应处理与禁用游戏内使用的对象相关的所有方面。对于精灵来说，为了在精灵存储在池中时提高性能，我们将可见性设置为 false，忽略对精灵的更新，清除任何实体修饰符和更新处理器，这样在我们获取新精灵时它们就不会仍在运行。

### 注意

即使不使用对象池，也可以考虑在不再需要的`Entity`上使用`setVisible(false)`和`setIgnoreUpdate(true)`。不断附加和分离`Entity`对象可能会给垃圾收集器运行提供机会，并可能在游戏过程中引起帧率的明显卡顿。

## 还有更多…

创建对象池以处理对象回收对于减少性能卡顿非常重要，但是当游戏首次初始化时，池中不会有任何可用的对象。这意味着，根据池需要在整个关卡中分配以满足最大对象数的对象数量，玩家可能会在游戏的前几分钟内注意到帧率的突然中断。为了避免此类问题，最好在关卡加载时预分配池对象，以避免在游戏过程中创建对象。

为了在加载期间分配大量池对象，我们可以对任何扩展`GenericPool`的类调用`batchAllocatePoolItems(pCount)`，其中`pCount`是我们希望分配的项数。请记住，加载比我们需要的更多的物品是资源的浪费，但如果分配的物品不足，也可能会引起帧率卡顿。例如，为了确定我们的游戏中应分配多少敌方对象，我们可以制定一个公式，比如默认敌方数量乘以关卡难度。然而，所有游戏都是不同的，因此所需的对象创建公式也将不同。

## 另请参阅

+   第二章中关于*使用精灵为场景注入生命*的部分

# 使用精灵组减少渲染时间

精灵组是任何需要在任何时刻处理场景上数百个可见精灵的 AndEngine 游戏的一个很好的补充。`SpriteGroup`类允许我们将许多精灵渲染调用分组到有限的 OpenGL 调用中，从而消除大量开销。如果一个校车只接一个孩子，把他们送到学校，然后再接下一个孩子，直到所有的孩子都到学校，这个过程完成所需的时间会更长。使用 OpenGL 绘制精灵也是同样的道理。

## 开始操作…

请参考代码包中名为`ApplyingSpriteGroups`的类。这个示例需要一个名为`marble.png`的图像，该图像的宽度为 32 像素，高度为 32 像素。

## 如何操作…

当在我们的游戏中创建一个`SpriteGroup`时，我们可以将它们视为专门用于`Sprite`对象的`Entity`层。以下步骤说明如何创建并将`Sprite`对象附加到`SpriteGroup`。

1.  创建一个精灵组可以使用以下代码实现：

    ```kt
      // Create a new sprite group with a maximum sprite capacity of 500
      mSpriteGroup = new SpriteGroup(0, 0, mBitmapTextureAtlas, 500,     mEngine.getVertexBufferObjectManager());

      // Attach the sprite group to the scene
      mScene.attachChild(mSpriteGroup);
    ```

1.  将精灵附加到精灵组同样是一个简单的任务：

    ```kt
      // Create new sprite
      Sprite sprite = new Sprite(tempX, tempY, spriteWidth, spriteHeight, mTextureRegion, mEngine.getVertexBufferObjectManager());

      // Attach our sprite to the sprite group
      mSpriteGroup.attachChild(sprite);
    ```

## 工作原理…

在这个示例中，我们设置了一个场景，将大约 375 个精灵应用到我们的场景中，所有这些都是通过使用`mSpriteGroup`对象绘制的。一旦创建了精灵组，我们基本上可以将其视为一个普通实体层，根据需要附加精灵。

+   在我们活动的`onCreateResources(`方法中为我们的精灵创建一个`BuildableBitmapTextureAtlas`：

    ```kt
    // Create texture atlas
    mBitmapTextureAtlas = new BuildableBitmapTextureAtlas(mEngine.getTextureManager(), 32, 32, TextureOptions.BILINEAR);

    // Create texture region
    mTextureRegion = BitmapTextureAtlasTextureRegionFactory.createFromAsset(mBitmapTextureAtlas, getAssets(), "marble.png");

    // Build/load texture atlas
    mBitmapTextureAtlas.build(new BlackPawnTextureAtlasBuilder<IBitmapTextureAtlasSource, BitmapTextureAtlas>(0, 0, 0));
    mBitmapTextureAtlas.load();
    ```

    创建用于`SpriteGroup`中的纹理可以像处理普通 Sprite 一样处理。

+   构造我们的`mSpriteGroup`对象并将其应用到场景中：

    ```kt
    // Create a new sprite group with a maximum sprite capacity of 500
    mSpriteGroup = new SpriteGroup(0, 0, mBitmapTextureAtlas, 500, mEngine.getVertexBufferObjectManager());

    // Attach the sprite group to the scene
    mScene.attachChild(mSpriteGroup);
    ```

    `SpriteGroup`需要两个我们尚未处理的新参数。`SpriteGroup`是`Entity`的一个子类型，因此我们知道前两个参数是用于定位`SpriteGroup`的 x 和 y 坐标。第三个参数，我们传递了一个`BitmapTextureAtlas`。*精灵组只能包含与精灵组共享相同纹理图的精灵！*第四个参数是`SpriteGroup`能够绘制的最大容量。如果容量是 400，那么我们可以将最多 400 个精灵应用到`SpriteGroup`。将容量限制为我们希望绘制的最大精灵数非常重要。*超出限制将导致应用程序强制关闭*。

+   最后一步是将精灵应用到精灵组。

在这个示例中，我们设置了一个循环，以便将精灵应用到屏幕上的各个位置。然而，在这里我们真正关心的是以下用于创建`Sprite`并将其附加到`SpriteGroup`的代码：

```kt
Sprite sprite = new Sprite(tempX, tempY, spriteWidth, spriteHeight, mTextureRegion, mEngine.getVertexBufferObjectManager());

// Attach our sprite to the sprite group
mSpriteGroup.attachChild(sprite);
```

我们可以像创建任何其他精灵一样创建我们的精灵。我们可以像平常一样设置位置、缩放和纹理区域。现在要做好准备迎接棘手的部分！我们必须调用`mSpriteGroup.attachChild(sprite)`，以允许`mSpriteGroup`对象处理精灵对象的绘制。这就完成了！

按照这些步骤，我们可以在性能下降之前成功让我们的精灵组在屏幕上绘制许多精灵。与使用单独缓冲区单独绘制精灵相比，差异是巨大的。在许多情况下，用户声称在使用包含大量实体同时出现在场景中的游戏时，可以实现高达 50%的性能提升。

## 还有更多…

现在还不是将所有项目转换为使用精灵组的时候！使用精灵组的好处不言而喻，但这并不意味着没有负面影响。`SpriteGroup`类并不直接得到 OpenGL 的支持。这个类或多或少是一个'hack'，它让我们在额外的渲染调用中节省一些时间。在更复杂的项目中设置精灵组可能会因为'副作用'而变得麻烦。

在多次附着和分离利用了 alpha 修饰符和修改了可见性的许多精灵后，有时会出现一些情况，导致精灵组中的某些精灵出现'闪烁'。在越来越多的精灵被附着和分离，或者多次设置为不可见/可见之后，这种结果最为明显。有一种方法可以绕过这个问题，而且不会过多影响性能，即移动精灵使其离开屏幕，而不是从图层中分离它们或设置为不可见。然而，对于只利用一个活动并且根据当前关卡切换场景的大型游戏来说，将精灵移出屏幕可能会带来未来的问题。

在决定使用精灵组之前，要考虑这一点并明智地计划。在将精灵组整合到游戏中之前，测试你打算如何使用精灵的精灵组可能也会有所帮助。精灵组不总会引起问题，但这是需要记住的一点。此外，AndEngine 是一个开源项目，它正在不断更新和改进。关注最新修订版以获取修复或改进。

## 另请参阅

+   第二章中的*了解 AndEngine 实体*部分，*使用实体*

+   第二章中的*用精灵为场景注入生命*部分，*使用实体*

## 使用实体剔除来禁用渲染

剔除实体是一种防止不必要的实体被渲染的方法。在精灵在 AndEngine `Camera`的视图中不可见的情况下，这可以提高性能。

## 如何操作…

对任何预先存在的`Entity`或`Entity`子类型进行以下方法调用：

```kt
entity.setCullingEnabled(true);
```

## 它是如何工作的…

剔除实体会根据它们在场景中的位置相对于摄像机可见场景部分来禁止某些实体被渲染。当我们场景上有许多精灵可能会偶尔移出摄像机视野时，这非常有用。启用剔除后，那些在摄像机视图之外的实体将不会被渲染，以避免我们进行不必要的 OpenGL 调用。

请注意，剔除只发生在那些完全在摄像机视野之外的实体上。这考虑了实体的整个区域，从左下角到右上角。如果实体的部分在摄像机视野之外，不会应用剔除。

## 还有更多内容...

**剔除**只会停止渲染那些移出摄像机视野的实体。因此，对所有那些经常移出`Camera`区域的 游戏对象（如物品、敌人等）启用剔除并不是一个坏主意。对于由较小纹理组成的大型背景实例，剔除也可以显著提高性能，尤其是考虑到背景图像的大小。

剔除确实可以帮助我们节省渲染时间，但这并不意味着我们应该对所有实体启用剔除。毕竟，默认不启用它是有一个原因的。在 HUD 实体上启用剔除是一个糟糕的主意。对于暂停菜单或其他可能进出摄像机视野的大型实体来说，包含它似乎是一个可行的选择，但这可能会导致在移动摄像机时出现问题。AndEngine 的工作方式是 HUD 实际上永远不会随着摄像机移动，所以如果我们对 HUD 实体启用剔除，然后将摄像机向右移动 800 像素（假设我们的摄像机宽度是 800 像素），我们的 HUD 实体仍然会在物理上响应它们在屏幕上的正确位置，但它们不会渲染。它们仍然会响应触摸事件和其他各种场景，但我们就是看不到它们在屏幕上。

此外，在实体被绘制在场景上之前，剔除还需要进行一层额外的可见性检查。因此，较旧的设备在启用实体剔除时，如果这些实体没有被剔除，可能会有性能损失。这可能听起来不多，但当我们在仅能勉强运行 30 帧每秒的设备上有玩家运行时，对例如 200 个精灵进行额外的可见性检查可能会足以使游戏体验变得不便。

## 参见：

+   第二章中关于*理解 AndEngine 实体*的部分，*使用实体*。
