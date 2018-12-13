---
title: WebSocket的使用
id: UsingTheWebSocket
directory: Guide
tags:
  - WebSocket
  - 网络协议
cover: https://static.runoob.com/images/mix/code_by_rasmusir-d4a4dj2.jpg
date: 2018-09-26 17:12:40
---
> **什么是WebSocket？** 
它是一种通讯协议，由HTML5提供的在单个TCP连接中进行全双工通讯的协议。
**它与HTTP有什么不同？**
1、HTTP 协议是一种无状态的、无连接的、单向的应用层协议。它采用了请求/响应模型。
通信请求只能由客户端发起，服务端对请求做出应答处理。WebSocket 连接允许客户端和服务器之间进行全双工通信。
2、在HTTP中一个请求（Request）对应一个响应（Response）。请求完毕即断开连接。WebSocket 只需要建立一次连接，就可以一直保持连接状态。

创建一个Web项目，引入WebSocket的包。
```xml
<dependency>
    <groupId>javax.websocket</groupId>
    <artifactId>javax.websocket-api</artifactId>
    <version>1.1</version>
    <scope>provided</scope>
</dependency>
```
创建类MyWebSocket，加上ServerEndpoint注解，和相应的生命周期注解。
```java
@ServerEndpoint("/websocket")
public class MyWebSocket {
    @OnOpen
    public void open(Session session, EndpointConfig conf) {
        System.out.println("连接回调");
    }

    @OnMessage
    public void message(Session session, String msg) {
        System.out.println("传输回调");
    }


    @OnError
    public void error(Session session, Throwable error) {
        System.out.println("错误回调");
    }


    @OnClose
    public void close(Session session, CloseReason reason) {
        System.out.println("关闭回调");
    }
}

```
这样就是一个WebSocket类了；
@ServerEndpoint注解中路径就是访问这个WebSocket的URL地址；
你可以在URL中加入参数，在客户端连接的时候可以接受到这个参数，如：
```java
@ServerEndpoint("/websocket/{id}")

@OnOpen
public void open(Session session, EndpointConfig conf,@PathParam("id")String id) {
    System.out.println(id);
}
```
onOpen()回调中的Session代表了当前的WebSocket连接，你可以自己添加用户的属性；
```java
session.getUserProperties().put("key","value");
```
WebSocket端点可以发送和接收文本和二进制消息。此外，他们还可以发送ping帧并接收pong帧。
@OnMessage在端点中最多可以有三个注释方法，支持重载，所以可以通过以下方法来接收不同的数据类型。
```java
@OnMessage 
public void textMessage（Session session，String msg）{ 
  System.out.println（“Text message：”+ msg）; 
} 
@OnMessage 
public void binaryMessage（Session session，ByteBuffer msg）{ 
  System.out.println（“Binary message：”+ msg.toString（））; 
} 
@OnMessage 
public void pongMessage（Session session，PongMessage msg）{ 
  System.out.println（“Pong message：”+ 
                      msg.getApplicationData（）。toString（））; 
} 
```
发送消息，在回调实参中，通过session.getOpenSessions()来获取连接的实例；
官方对这个方法的解释如下：
Return a copy of the Set of all the open web socket sessions that represent connections to the same endpoint to which this session represents a connection.
返回所有开放web套接字会话集的副本，这些会话表示连接到此会话表示连接的同一端点。
```java
for (Session sess : session.getOpenSessions()){
    if(sess.isOpen()){
        try {
            sess.getBasicRemote().sendText("Text message");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
实际上在使用过程中，session.getOpenSessions()并没有返回所有的WebSocket连接，因此，你可以使用这个并发集合，在onOpen()方法中来保存这些Session。（注：一定要是并发的集合，在多线程和中不会出现错误）
```java
CopyOnWriteArrayList<Session> clients = new CopyOnWriteArrayList<Session>();
clients.add(session);
```
发送消息的区别，Basic是I/O阻塞的，Async是异步的，一般用Async就可以了。
```java
session.getBasicRemote().sendText("Text message：");
session.getAsyncRemote().sendText("Text message：");
```
关闭回调，关闭连接的时候调用，会有一个CloseReason关闭原因的实参，如果对方不填为null。
```java
@OnClose
public void close(Session session, CloseReason reason){
    clients.remove(session);
}
```
错误回调，当连接出现意外中断的情况下调用，正常的关闭连接会发送关闭状态码：1000，如果出现连接意外中断，可以通过状态码来判断错误原因。
```java
@OnError
public void error(Session session, Throwable error){
    System.out.println(reason.getCloseCode());
}
```

<b><font color="FF0000">文章内容均为本人手写；<br>禁止转载，请尊重原著；<br>本人QQ：794763733。</font></b>