# 附录 A. 小测验答案

# 第一章：– 开始使用 Corona SDK

## 小测验 – 了解 Corona

| Q1 使用 Corona 模拟器有哪些正确之处？ | 1 |
| --- | --- |
| Q2 在 iPhone 开发者计划中，你可以使用多少个 iOS 设备进行开发？ | 4 |
| Q3 使用 Corona SDK 为 Android 构建时，版本代码需要是什么？ | 2 |

# 第二章：– Lua 速成课程和 Corona 框架

## 小测验 – Lua 基础

| Q1 以下哪些是值？ | 4 |
| --- | --- |
| Q2 哪个关系运算符是错误的？ | 3 |
| Q3 正确缩放对象在*x*方向的方法是什么？ | 4 |

# 第三章：– 建立我们的第一个游戏 – Breakout

## 小测验 – 构建游戏

| Q1 在代码中添加物理引擎时，哪些函数可以添加到你的应用程序中？ | 4 |
| --- | --- |
| Q2 添加事件监听器时以下哪个是正确的？ | 4 |
| Q3 以下显示对象正确过渡到`x = 300`, `y = 150`并将 alpha 改为 0.5，需要 2 秒的方法是什么？ | 1 |

# 第四章：– 游戏控制

## 小测验 – 使用游戏控制

| Q1 正确从舞台移除显示对象的方法是什么？ | 3 |
| --- | --- |

| Q2 将以下显示对象正确转换为物理对象的方法是什么？ |

```kt
local ball = display.newImage("ball.png")
```

| 3 |
| --- |

| Q3 在以下函数中，`"began"`最好表示什么意思？ |

```kt
local function onCollision( event )
  if event.phase == "began" and event.object1.myName == "Box 1" then

    print( "Collision made." )

  end
end
```

| 4 |
| --- |

# 第五章：– 让我们的游戏动起来

## 小测验 – 动画图形

| Q1 正确暂停图像表动画的方法是什么？ | 1 |
| --- | --- |
| Q2 如何使动画序列无限循环？ | 3 |
| Q3 如何创建一个新的图像表？ | 4 |

# 第六章：– 播放声音和音乐

## 小测验 – 关于音频的一切

| Q1 正确清除内存中音频文件的方法是什么？ | 3 |
| --- | --- |
| Q2 应用程序中可以同时播放多少个音频通道？ | 4 |
| Q3 如何使音频文件无限循环？ | 1 |

# 第七章：– 物理 – 下落物体

## 小测验 – 动画图形

| Q1 有哪个功能可以获取或设置文本对象的文本字符串？ | 1 |
| --- | --- |
| Q2 有哪个函数能将任何参数转换成字符串？ | 3 |
| Q3 哪种体型受到重力和其他体型碰撞的影响？ | 1 |

# 第八章：– 操作 Composer

## 小测验 – 游戏过渡和场景

| Q1 使用 Composer 改变场景时需要调用哪个函数？ | 2 |
| --- | --- |
| Q2 有哪个函数能将任何参数转换成数字或 nil？ | 1 |
| Q3 如何暂停一个计时器？ | 3 |
| Q4. 如何恢复一个计时器？ | 2 |

# 第九章：– 处理多设备和网络应用

## 小测验 – 处理社交网络

| Q1 缩放高分辨率精灵表的特定 API 是什么？ | 2 |
| --- | --- |
| Q2 在 Facebook 上允许在用户墙发布内容的发布权限叫什么？ | 2 |
| Q3 `facebook.login()`需要哪些参数？ | 4 |

# 第十章：– 优化、测试和发布你的游戏

## 小测验 – 发布应用

| Q1 创建 iOS Distribution Provisioning 文件时，需要使用哪种分发方法？ | 2 |
| --- | --- |
| Q2 提交的 iOS 应用程序的状态应在哪里查询？ | 1 |
| Q3 在 Google Play 商店中构建应用需要什么？ | 4 |

# 第十一章：– 实现应用内购买

## 突击测验 – 关于应用内购买的一切

| Q1 非消耗性购买是什么？ | 1 |
| --- | --- |
| Q2 关于测试应用内购买，以下哪项是正确的？ | 3 |
| Q3 测试应用内购买必须使用哪种类型的 Provisioning Profile？ | 2 |
