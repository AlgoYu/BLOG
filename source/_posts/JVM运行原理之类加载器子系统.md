---
title: JVM运行原理之类加载器子系统
id: JVM-ClassLoaderSubsystem
directory: Guide
tags:
  - JVM
  - Java虚拟机
cover: https://static.runoob.com/images/mix/code-wallpaper-22.jpg
date: 2018-11-02 11:40:06
---
> JVM是Java Virtual Machine（Java虚拟机）的缩写，JVM是一种用于计算设备的规范，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的。
Java语言的一个非常重要的特点就是与平台的无关性。而使用Java虚拟机是实现这一特点的关键。一般的高级语言如果要在不同的平台上运行，至少需要编译成不同的目标代码。而引入Java语言虚拟机后，Java语言在不同平台上运行时不需要重新编译。Java语言使用Java虚拟机屏蔽了与具体平台相关的信息，使得Java语言编译程序只需生成在Java虚拟机上运行的目标代码（字节码），就可以在多种平台上不加修改地运行。Java虚拟机在执行字节码时，把字节码解释成具体平台上的机器指令执行。这就是Java的能够“一次编译，到处运行”的原因。

今天来为大家一步一步解析JVM的运行原理，在我们编译源代码.java文件时，Java编译器会生成具有相同文件名的.class文件（包含字节码）。当我们运行.class文件时，这些.class文件会进入各个步骤，这些步骤描述了整个JVM的工作内容,网上已经有详细的图解，示例三张：
![](https://cdncontribute.geeksforgeeks.org/wp-content/uploads/jvm-3.jpg)
![](https://javatutorial.net/wp-content/uploads/2017/10/jvm-architecture.png)
![](https://www.javainterviewpoint.com/java-virtual-machine-architecture-in-java/jvm-architecture/)

## **类加载器子系统(Class Loader Subsystem)** ##
.class文件都由类加载子系统加载，它主要负责了3个工作：

>  1. 加载
 2. 连接
 3. 初始化


## 第一小节：加载 ##

加载意味着该组件会从硬盘系统中把.class文件读取到JVM内存中，并在方法区域中存储该相应的二进制数据。例如：

 >1. 完全限定的类名称
2. 直接父类的完全限定名称
3. 方法信息
4. 变量信息
5. 构造函数信息
6. 修饰符信息
7. 常量池信息
8. .class文件是类、接口还是枚举。

对于每个加载的.class文件，JVM会立即在堆内存中创建一个java.lang.Class类型的对象。
我们可以用Class类来获取类的如级别信息，方法信息，构造函数，变量等。

> 在JVM中将在堆内存中创建多少个对象？
对于每个加载的类型，即使我们在程序中多次使用类，也只会创建一个对象。

加载器的类型有三种：
 >1. **Bootstrap Class Loader(引导类加载器)** - 这个类加载器负责加载包中存在的内部核心Java类rt.jar和其他类java.lang.\*。默认情况下，每个JVM都可以使用它，并使用本机C/C++语言编写。这个类加载器没有父类，如果开发人员调用String.class.getClassLoader()它，它将返回null，任何基于它的代码都会抛出NullPointerExceptionJava。
2. **Extensions Class Loader（扩展类加载器）** - 这个类加载器是Primordial类加载器的子类，负责从扩展类路径（即jdk\jre\lib\ext）加载类。它是用Java语言编写的，相应的.class文件是sun.misc.Launcher\$ExtClassLoder.class。
3. **System Class Loader（系统类加载器）** - 此类加载器是Extension类加载器的子类，负责从系统类路径加载类。它在内部使用' CLASSPATH'环境变量，并用Java语言编写。JVM中的系统类加载器是由sun.misc.Launcher$AppClassLoader.class
实现的。
## 第二小节：链接 ##
此组件执行类或接口的链接。由于此组件涉及新数据结构的分配，因此可能会抛出OutOfMemoryError。
它主要执行以下三项重要活动：

 >1. 验证
2. 准备
3. 解析
 
![](https://codepumpkin.com/wp-content/uploads/2017/07/jvm_architecture2.png)
**1.验证：**
在验证的过程中会检查以下几点：

 - 它是一个确保类的二进制表示在结构上是否正确的过程。
 - JVM将检查.class文件是否由有效的编译器生成。
 - .class文件的格式是否正确。
 - 内部字节码验证器负责此活动。
 - 字节码验证器是类装入器子系统的一部分。
 - 如果类或接口的二进制表示不满足静态或结构约束，则抛出VerifyError。抛出的错误是LinkageError(或它的子类)的实例。

> 为什么Java是安全的语言？
就是因为字节码验证器的存在，字节码验证器是使java成为安全语言的特性之一。如果攻击者手动更改类文件以创建某种病毒，字节码验证器将检测到该类文件，因为它不是由有效编译器生成的。Verfication失败，我们会得到运行时异常说java.lang.VerifyError。

**2.准备：**
在此阶段，JVM将为类级或接口级静态变量分配内存并分配默认值。
在初始化阶段，将原始值分配给静态变量，在准备阶段，只分配默认值。
**3.解析：**
解析是通过运行时常量池中的符号引用动态确定具体值的过程。简单地说，就是用方法区域的原始内存引用替换程序中的符号名的过程。


----------


举个例子：
```java
public class Testing {
 
    public static void main(String[] arg) {
        String s = new String("QQ794763733");
        Student s1 = new Student();
    }
}
```
上面的类，由类加载器加载：

> 1. Testing.class
2. Object.class（Testing.class的父类）
3. String.class
4. Student.class

解析内容如下：

> - 这些类的名称存储在Testing类的常量池中。
- 在解析阶段，这些名称将替换为方法区域中的原始内存级别引用。
- 所有符号引用（现在以运行时常量池的形式加载到方法区域中）将解析为此JVM加载的实际类型。
- 如果可以解析符号引用，但导致定义冲突，会抛出一个IncompatibleClassChangeError。
- 如果方法查找失败，方法解析会抛出一个NoSuchMethodError。
- 如果方法查找成功并且方法是Abstract但Class不是Abstract则方法解析会抛出AbstractMethodError等等。
- 所有上述错误都是类的子java.lang.LinkageError类。

 
## 第三小节：初始化 ##
在初始化阶段，所有静态变量都分配有原始值，静态块将从父级到子级以及从上到下执行。此过程需要仔细同步，因为JVM是多线程的，并且某些线程可能会尝试同时初始化相同的类或接口。
在加载，链接和初始化时，如果发生任何错误，我们将获得运行时异常说明java.lang.LinkageError 或其子类  java.lang.VerifyError。


----------

##**ClassLoader如何在Java中工作**##
Java中的类加载器有三个原则，即委托，可见性和唯一性。
![](http://www.javacodegeeks.com/wp-content/uploads/2018/04/jvm_archi_clrda_guide_4.jpg)
委托：
> - 当虚拟机遇到某个类，JVM就会检查是否.class加载了指定的文件
- 如果.class文件已经加载到方法区域中，那么JVM将考虑该类。如果没有，JVM请求类加载器子系统加载该特定类
- 类加载器子系统将请求传递给Application Class Loader，后者又将此请求委托给Extensions Class Loader。Extensions Class Loader将再次将此请求委托给Primordial Class Loader
- Primordial Class Loader将在引导类路径（即jdk\jre\lib\rt.jar）中搜索此类。如果找到，则加载相应的.class文件
- 如果没有，Primordial Class Loader会将请求委托给Extensions Class Loader。这将在jdk\jre\lib\ext路径中搜索类。如果找到，则加载相应的.class文件
- 如果没有，Extensions Class Loader会将请求委托给Application Class Loader，并将在应用程序的类路径中搜索该类。如果找到，则加载它，否则开发人员将在运行时获得ClassNotFoundException。

可见性：

> Application Class Loader具有查看父类加载器加载的类的可见性，但反之亦然，即如果类由Application Class Loader加载，稍后再次尝试使用Extensions Class Loader显式加载相同的类，将ClassNotFoundException在运行时抛出。例如：

```java
// 打印这个类的加载器
System.out.println("Test.getClass().getClassLoader()?= " + Test.class.getClassLoader());
// 尝试再次显式地使用该类加载器的父类加载这个类。
Class.forName("com.jcg.classloading.test.Test", true,  Test.class.getClassLoader().getParent());
```

唯一性：

> 父类加载器加载的类不应再由子类加载器重新加载


----------
##**如何在Java代码中加载类？**##
类加载器是分层的。应用程序中的第一个类是在静态main()方法的帮助下专门加载的。所有后续类都由Static或Dynamic类加载技术加载。

 - **静态加载：**在此技术中，类通过new运算符静态加载
 - **动态加载：**在此技术中，使用Class.forName()或loadClass()方法以编程方式加载类。两者之间的区别在于前者在加载后初始化对象，而后者只加载类但不初始化对象。

<b><font color="FF0000">文章内容均为本人手写；<br>禁止转载，请尊重原著；<br>本人QQ：794763733。</font></b>