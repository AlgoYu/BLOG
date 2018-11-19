---
title: 'C#委托事件'
id: DelegateEvent
directory: Guide
tags:
  - C#
  - 委托事件
cover: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1541499279668&di=34b23d42cbae5a27b76ba1e36d467037&imgtype=0&src=http%3A%2F%2Fpic1.win4000.com%2Fwallpaper%2F2018-01-03%2F5a4c4e8c2ebce.png
date: 2018-11-04 16:25:25
---
> ## 什么是委托？##
C# 中的**委托（Delegate）**类似于 C 或 C++ 中函数的指针。**委托（Delegate）** 是存有对某个方法的引用的一种引用类型变量。引用可在运行时被改变。
**委托（Delegate）**特别用于实现事件和回调方法。所有的**委托（Delegate）**都派生自 System.Delegate 类。
>## 什么是事件？##
**事件（Event）** 基本上说是一个用户操作，如按键、点击、鼠标移动等等，或者是一些出现，如系统生成的通知。应用程序需要在事件发生时响应事件。例如，中断。事件是用于进程间通信。
> ## 什么是委托事件？##
事件在类中声明且生成，且通过使用同一个类或其他类中的委托与事件处理程序关联。包含事件的类用于发布事件。这被称为 **发布器（publisher）** 类。其他接受该事件的类被称为 **订阅器（subscriber）** 类。事件使用 **发布-订阅（publisher-subscriber）**模型。
**发布器（publisher）** 是一个包含事件和委托定义的对象。事件和委托之间的联系也定义在这个对象中。**发布器（publisher）**类的对象调用这个事件，并通知其他的对象。
**订阅器（subscriber）** 是一个接受事件并提供事件处理程序的对象。在发布器（publisher）类中的委托调用**订阅器（subscriber）**类中的方法（事件处理程序）。

先来一个委托的例子：
```csharp
class Program
{
    //声明委托
    public delegate void Talk(string s);

    public static void Main(string[] args)
    {
        //实例化委托
        Talk talk = new Talk(SayMessage);
        //调用委托函数
        talk("QQ794763733");
    }

    public static void SayMessage(string s)
    {
        Console.WriteLine(s);
    }
}
```
声明委托，然后直接实例化，传方法名给委托就可以调用了。
委托支持多播，可以用“+”运算符合并，只有相同类型的委托才可以被合并。"-" 运算符可用于从合并的委托中移除组件委托。
```csharp
class Program
{
    //声明委托
    public delegate void Talk();

    public static void Main(string[] args)
    {
        //实例化委托
        Talk talk1 = new Talk(SayHello);
        Talk talk2 = new Talk(SayMessage);
        //相加委托
        talk1 += talk2;
        //多播调用
        talk1();
    }

    public static void SayHello()
    {
        Console.WriteLine("欢迎大家来到我的博客！");
    }

    public static void SayMessage()
    {
        Console.WriteLine("QQ794763733");
    }
}
```
委托常用例子，常常用来初始化数据。
```csharp
public class Test
{
    //声明委托
    public delegate void Configration(MyMessage message);
    //声明MyMessage类属性
    public MyMessage Message { get; set; }
    public Test()
    {
        //构造函数中初始化
        Message = new MyMessage();
    }
    //参数是一个委托
    public void SetName(Configration config)
    {
        //调用委托
        config(Message);
    }
}
//用来保存数据的类
public class MyMessage
{
    public string Contact { get; set; }
    public int Num { get; set; }
}
```
```csharp
class Program
{
    public static void Main(string[] args)
    {
        Test test = new Test();
        //传递一个委托
        //test.SetName(new Test.Configration(InIt));
        test.SetName(InIt);
        //打印
        Console.WriteLine("Contact:"+test.Message.Contact+ "\tNum:" + test.Message.Num);
    }

    public static void InIt(MyMessage message)
    {
        message.Contact = "QQ";
        message.Num = 794763733;
    }
}
```
接下来咱们看一下委托事件：
```csharp
public class Test
{
    //声明委托
    public delegate void Configration();
    //声明事件
    public event Configration ConfigrationEvents;
    //事件调用
    public void StartEvent()
    {
        ConfigrationEvents();
    }
}
```
```csharp
class Program
{

    public static void Main(string[] args)
    {
        //实例化一个Test对象
        Test test = new Test();
        //事件添加一个委托
        test.ConfigrationEvents += new Test.Configration(SayHello);
        //可以直接简写
        test.ConfigrationEvents += SayContact;
        //触发事件
        test.StartEvent();
    }

    public static void SayHello()
    {
        Console.WriteLine("Hello！欢迎来到我的博客！");
    }

    public static void SayContact()
    {
        Console.WriteLine("我的QQ是：794763733");
    }
}
```
其实多播和事件使用起来并没有多大差别，在Java中目前没有java这样的Delegate和Event的写法，但是利用接口或者多态还是可以实现，以上方式还可以结合Lambda一起用，谢谢大家。

<b><font color="FF0000">文章全部信息均为本人手写，禁止任何转载，本人QQ：794763733。</font></b>