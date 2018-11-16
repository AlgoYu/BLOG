---
title: 从回调、闭包到Lambda表达式只需要10分钟
id: Callback-Closure-Lambda
directory: Guide
tags:
  - 回调
  - 闭包
  - Lambda
cover: https://static.runoob.com/images/mix/rfwDB3L.png
date: 2018-10-29 17:00:40
---
> ## 什么是回调函数？ ##
回调函数就是一个通过函数指针调用的函数。如果你把函数的指针（地址）作为参数传递给另一个函数，当这个指针被用来调用其所指向的函数时，我们就说这是回调函数。回调函数不是由该函数的实现方直接调用，而是在特定的事件或条件发生时由另外的一方调用的，用于对该事件或条件进行响应。
> ## 什么是闭包函数？ ##
闭包是ECMAScript （JavaScript）最强大的特性之一，但用好闭包的前提是必须理解闭包。闭包的创建相对容易，人们甚至会在不经意间创建闭包，但这些无意创建的闭包却存在潜在的危害，尤其是在比较常见的浏览器环境下。如果想要扬长避短地使用闭包这一特性，则必须了解它们的工作机制。而闭包工作机制的实现很大程度上有赖于标识符（或者说对象属性）解析过程中作用域的角色。
> ## 什么是Lambda表达式？ ##
“Lambda 表达式”(lambda expression)是一个匿名函数，Lambda表达式基于数学中的λ演算得名，直接对应于其中的lambda抽象(lambda abstraction)，是一个匿名函数，即没有函数名的函数。Lambda表达式可以表示闭包（注意和数学传统意义上的不同）。
> ## 看不懂？ ##
以上都是官方解释，网上的教程多如牛毛，举例数不胜数，大家都说很简单，却举着非常不容易新手理解的例子，翻来覆去但是你就是看不懂？没关系，让我来以最通俗易懂的方式告诉你怎么去理解：

## **回调函数：** ##
咱们先写一个Official类：
```java
public class Official {
    //定义一个方法，返回一个字符串“QQ794763733”；
    public String getMessage(){
        return "QQ794763733";
    }
}
```
很简单，只有一个getMessage（）方法，返回“QQ794763733”这个字符串，在Main中调用：
```java
public class Main {
    public static void main(String arg[]){
        //直接new一个Official对象并调用getMessage()方法得到这个字符串并打印；
        System.out.println(new Official().getMessage());
    }
}
```
直接new一个调用就可以得到这个字符串了，那如此简单的返回字符串，你得用接口来写呢？那会是什么样子？我们写了一个接口：
```java
public interface Message {
    //参数是我想要传回的数据；
    void getMessage(String message);
}
```
接口方法的参数，就是你想要传的数据！然后在Official类中让构造函数传递这个接口的实现类，直接调用接口的getMessage（）方法，把“QQ794763733”这个字符串传进去：
```java
public class Official {
    private Message message;

    //构造函数，传递这个接口的实现类；
    public Official(Message message) {
        this.message = message;
    }

    public void getMessage(){
        //调用了接口的方法，我把我想要传回的数据传给了这个方法；
        message.getMessage("QQ794763733");
    }
}
```
我们再写一个这个接口的实现类：
```java
public class MessageImpl implements Message{
    @Override
    public void getMessage(String message) {
        //实现了这个接口，这个方法会接受一个字符串，并打印；
        System.out.println(message);
    }
}
```
实现getMessage（）方法，然后把参数message打印出来，现在看一下Main:
```java
public class Main {
    public static void main(String arg[]){
        MessageImpl messageimpl = new MessageImpl();
        //实例化Official时，通过构造函数，让Official里的属性接口等于MessageImpl这个实现类；
        Official official = new Official(messageimpl);
        official.getMessage();
    }
}
```
想必你应该理解了，大部分时候接口用来返回数据，对于接口的立场来说，我需要一个什么功能，我就抽象成一个接口，我可以提供给你的数据我就加在接口的参数里，我不管你使用这些数据做了什么，或者我不用提供数据，反正我会调用这个方法，你只需要实现这个方法，完成你自己的业务逻辑，或者按我的需求返回相应的数据即可，以上例子在实例化Official的构造函数中把MessageImpl这个实现类对象传进去，Official中的属性message就等于MessageImpl，在Official对象调用message.getMessage（）等于是在调用MessageImpl.getMessage（）,传递的字符串message自然就会被打印出来了，你甚至完全可以不用写实现类，可以简写为以下：
```java
public class Main {
    public static void main(String arg[]){
        //直接new一个匿名内部类实现接口
        Official official = new Official(new Message() {
            @Override
            public void getMessage(String message) {
                System.out.println(message);
            }
        });
        official.getMessage();
    }
}
```
有人会问Message不是接口吗？接口不是不能实例化吗？其实这里并没有直接new一个接口，在我们这样写new一个接口的时候，编译器会在编译的时候为我们自动生成一个类（匿名内部类）来帮我们实现这个接口，如Class A implements Message,只不过是匿名的；所以，无论是构造函数还是方法函数，只要参数是一个接口的情况下，我们可以直接new一个匿名内部类来实现这个接口，在@Override覆写的方法中写入我们的业务逻辑就可以了，这就是回调函数！


----------


## **闭包函数：** ##
闭包是JavaScript的强大特性，我们都知道JavaScript可以直接把函数声明称一个变量，如：
```javascript
var fuc = function(data){
	console.log(data);
}

fuc("QQ794763733");
```
它可以作为参数传递：
```javascript
var fuc = function(data){
	console.log(data);
}

function log(callback){
	callback("QQ794763733");
}

log(fuc);
```
或者，这样写：
```javascript
function log(callback){
	callback("QQ794763733");
}

log(function(data){
	console.log(data);
});
```
JavaScript可以更轻松简单的实现回调函数，那什么是闭包函数呢？我们都知道变量都会有一个作用域，闭包的通俗定义为，定义在函数内部的函数。
```javascript
function fuc() {
	var data = 0;
	return function() {
		data += 1;
		console.log(data);
	};
}
var fucObj = fuc();
fucObj(); //1，执行data += 1后，data还在；
fucObj(); //2；
fucObj = null; //data 被回收 即便注释掉这句话 也不影响funcObj1的data值；
var fucObj1 = fuc();
fucObj1(); //1；
fucObj1(); //2；
```
这就是闭包函数，如果你希望制造一个变量长期驻扎在内存中，并且拥有私有成员的存在，避免全局变量的污染，可以使用闭包来实现。


----------


## **Lambda表达式** ##
Lambda 表达式，也可称为闭包，它是Java8发布的最重要新特性；
Lambda允许把函数作为一个方法的参数（函数作为参数传递进方法中）；
使用 Lambda 表达式可以使代码变的更加简洁紧凑。
它的大概使用方法如下：
```java
// 1. 不需要参数,返回值为 5  
() -> 5  
  
// 2. 接收一个参数(数字类型),返回其2倍的值  
x -> 2 * x  
  
// 3. 接受2个参数(数字),并返回他们的差值  
(x, y) -> x – y  
  
// 4. 接收2个int型整数,返回他们的和  
(int x, int y) -> x + y  
  
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
(String s) -> System.out.print(s)
```
它们可以实例化对象，或者说代替匿名内部类，还可以用来筛选数据，遍历……各种千奇百怪的写法。
```java
// 类型声明
MathOperation addition = (int a, int b) -> a + b;

// 不用类型声明
MathOperation subtraction = (a, b) -> a - b;

// 大括号中的返回语句
MathOperation multiplication = (int a, int b) -> { return a * b; };

// 没有大括号及返回语句
MathOperation division = (int a, int b) -> a / b;

System.out.println("10 + 5 = " + tester.operate(10, 5, addition));
System.out.println("10 - 5 = " + tester.operate(10, 5, subtraction));
System.out.println("10 x 5 = " + tester.operate(10, 5, multiplication));
System.out.println("10 / 5 = " + tester.operate(10, 5, division));

// 不用括号
GreetingService greetService1 = message ->
System.out.println("Hello " + message);

// 用括号
GreetingService greetService2 = (message) ->
System.out.println("Hello " + message);

greetService1.sayMessage("Runoob");
greetService2.sayMessage("Google");
```
例如之前的匿名内部类实现接口，你可以像这样写：
```java
//原来的写法
public class Main {
    public static void main(String arg[]){
        Official official = new Official(new Message() {
            @Override
            public void getMessage(String message) {
                System.out.println(message);
            }
        });
        official.getMessage();
    }
}
```
```java
//Lambda表达式写法
public class Main {
    public static void main(String arg[]){
        //x直接等于回调接口的参数
        Official official = new Official(x->System.out.println(x));
        official.getMessage();
    }
}
```
又例如，遍历一个集合，你可以这样写：
```java
//原来的写法
public class Main {
    public static void main(String arg[]){
        //x=回调接口的参数
        List<String> strings = new ArrayList<String>();
        strings.add("794763733");
        strings.add("123456789");
        strings.add("987654321");
        for (String s:strings) {
            System.out.println(s);
        }
    }
}
```

```java
//Lambda表达式写法
public class Main {
    public static void main(String arg[]){
        //x=回调接口的参数
        List<String> strings = new ArrayList<String>();
        strings.add("794763733");
        strings.add("123456789");
        strings.add("987654321");
        strings.forEach(x->System.out.println(x));
    }
}
```
Lambda表达式的用法还有很多，我们只讨论这么多，下一次更新文章也不知道是什么时候了。

<b><font color="FF0000">文章全部信息均为本人手写，禁止任何转载，本人QQ：794763733。</font></b>