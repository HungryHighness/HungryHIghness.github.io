---
title: CS面试基础知识
date: 2022-02-05 20:32:59
tags: c#
categories: 语言基础
---

# C#

## 静态构造函数与构造函数

```csharp
  class A
    {
        public A(string text)
        {
            Console.WriteLine(text);
        }
    }

    class B
    {
        private static A a1 = new A("a1");
        private A a2 = new A("a2");

        public B()
        {
            a2 = new A("a4");
        }

        static B()
        {
            a1 = new A("a3");
        }
    }

    public static void Main(string[] args)
    {
        B b = new B();
    }

```

在调用B类型代码之前先执行静态构造函数。静态构造函数先初始化静态变量之后运行函数体内的语句。之后调用B的普通构造函数，普通构造函数先初始化成员变量之后运行函数体内的语句。因此输出为：

[![HnlQYj.png](https://s4.ax1x.com/2022/02/05/HnlQYj.png)](https://imgtu.com/i/HnlQYj)

## 反射与应用程序域

```csharp
using System.Diagnostics;
using System.Reflection;
using System.Runtime.CompilerServices;

namespace ConsoleApp2;

[Serializable]
internal class A : MarshalByRefObject
{
    public static int Number;

    public void SetNumber(int value)
    {
        Number = value;
    }
}

[Serializable]
internal class B
{
    public static int Number;

    public void SetNumber(int value)
    {
        Number = value;
    }
}

public class Solution
{
    public static void Main(string[] args)
    {
        string assembly = Assembly.GetEntryAssembly().FullName;
        AppDomain domain = AppDomain.CreateDomain("NewDomain");

        A.Number = 10;
        string nameOfA = typeof(A).FullName;
        A a = domain.CreateInstanceAndUnwrap(assembly, nameOfA) as A;
        a.SetNumber(20);
        Console.WriteLine($"Number in class A is {A.Number}");

        B.Number = 10;
        string nameOfB = typeof(B).FullName;
        B b = domain.CreateInstanceAndUnwrap(assembly, nameOfA) as B;
        b.SetNumber(20);
        Console.WriteLine($"Number in class B is {B.Number}");
    }
}
```

> 此代码在.NET5+以上无法运行，因为现在已经不再支持创建其他应用域，可详见文档[.NET Framework 技术在 .NET Core 和 .NET 5+ 上不可用 | Microsoft Docs](https://docs.microsoft.com/zh-cn/dotnet/core/porting/net-framework-tech-unavailable)。

上述代码先创建一个叫做**NewDomain**的应用程序域，并在该域中利用反射创建类型A和类型B的实例。但是**类型A继承于MarshalByRefObject**，而B没有，所以两者在跨越应用域的边界时表现出的行为不同。

首先考虑A的情况，因A继承自MarshalByRefObject，那么a实际上只是在默认的域中的一个代理实例(Proxy)，它指向位于NewDomain域中的一个实例。当调用a的方法SetNumber时，是在NewDomain域中调用该方法，它将修改NewDomain域中静态变量A.Number的值并设置为20。由于静态变量在每个应用程序域中都有一份独立的拷贝，因此修改NewDomain域中的A.Number对于默认域中没有任何影响。由于Console.WriteLine()是在默认应用域中输出，因此输出仍然是10。

B的情况比较简单，因为B只是从Object继承来的类型，它在穿越应用程序域的边界时，将会完整地复制实例。因此在代码中，我们在NewDomain中生成B的实例，但是会把实例b复制到默认的应用程序域。此时调用b.SetNumber也是在默认的应用程序域中修改，所以输出是20.

## 不同的相等

```csharp
public static bool ReferenceEquals(object left, object right);
public static bool Equals(object left, object right);
public virtual bool Equals(object right);
public static bool operator ==(MyClass left, MyClass right);
```

因为C#即允许创建值类型，又允许创建引用类型，所以要从不同角度考虑两个对象是否相等。

- 两个引用对象是否相等，要看引用的是否是同一个实例。
- 两个值类型是否相同，要看是否属于同一类型并且具有相同的内容。

其中静态版本的`ReferenceEquals()`和`Equals()`不需要重写，因为无论待比较的两个对象在运行期是什么类型，这两个方法都能正确的比较。

需要考虑的是针对自己所创建的值类型来重写实例版本`Equals()`方法，并且重载`==`运算符，以求提升比较的效率。

如果你创建的某个引用类型需要按照内容而非身份来判断两个对象是否相等，那么应该针对该类型重写实例版本的`Equals()`方法。

## 逆变与协变

[泛型中的协变和逆变 | Microsoft Docs](https://docs.microsoft.com/zh-cn/dotnet/standard/generics/covariance-and-contravariance)

# 计算机网络

# 操作系统

# Unity

## 动态加载资源与方式

`AssetsBundle.Load`：即将资源打成 asset bundle 放在服务器或本地磁盘，然后使用WWW模块get 下来，然后从这个bundle中load某个object，unity官方推荐也是绝大多数商业化项目使用的一种方式。

`AssetDatabase.LoadAssetAtPath` ：这种方式只在editor范围内有效，游戏运行时没有这个函数，它通常是在开发中调试用的。

`Resource.Load`:可以直接load并返回某个类型的Object，前提是要把这个资源放在Resource命名的文件夹下，Unity不管有没有场景引用，都会将其全部打入到安装包中。

[在运行时加载资源 - Unity 手册 (unity3d.com)](https://docs.unity3d.com/cn/2020.3/Manual/LoadingResourcesatRuntime.html)

## 引擎中有哪些坐标系

- 屏幕坐标系：即游戏画面，为二维坐标系

- 世界坐标系：游戏物体绝对位置坐标所在坐标系，为三维坐标系

- 模型坐标系：游戏物体自身坐标系，为三维坐标系

- 观察坐标系（视口坐标系）：也就是摄像机的视锥体空间，为三维坐标系，转换到屏幕空间需要进行投影操作

> Unity使用的是**左手**坐标系

[Unity3D坐标系 - 简书 (jianshu.com)](https://www.jianshu.com/p/0c5c8fb78074)

## 相机中的Clipping Plane、Near、Far数值有什么意义

[实时渲染中的坐标系变换（3）：投影变换-1 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/113662566)

论games101的重要性。。要不还是寄了把

## 四元数的作用

万向节死锁经典问题了

[四元数和三维转动，可互动的探索式视频（请看链接）_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Lt411U7og?spm_id_from=333.999.0.0)

## OnEnbale、Awake、Start运行时的先后顺序

- **Awake** ：始终在任何 Start 函数之前并在实例化预制件之后调用此函数。（如果游戏对象在启动期间处于非活动状态，则在激活之后才会调用 Awake。）
- **OnEnable：**（仅在对象处于激活状态时调用）在启用对象后立即调用此函数。在创建 MonoBehaviour 实例时（例如加载关卡或实例化具有脚本组件的游戏对象时）会执行此调用。
- **Start** ：仅当启用脚本实例后，才会在第一次帧更新之前调用 Start。

[事件函数的执行顺序 - Unity 手册 (unity3d.com)](https://docs.unity3d.com/cn/2020.3/Manual/ExecutionOrder.html)

## 相机的移动在Update函数中好吗？物理更新一般是在Update中吗？

相机移动在`LateUpdate`，物理更新在`FixedUpdate`

这里设计到帧更新问题，可以看一下[游戏循环 · Sequencing Patterns · 游戏设计模式 (tkchu.me)](https://gpp.tkchu.me/game-loop.html)

[重要课程 - 时间 - 统一手册 (unity.cn)](https://docs.unity.cn/cn/2020.3/Manual/TimeFrameManagement.html)

[Unity中的FixedUpdate、Update、LateUpdate的区别及游戏帧更新_enternalstar的博客-CSDN博客_fixedupdate](https://blog.csdn.net/enternalstar/article/details/108507205)
