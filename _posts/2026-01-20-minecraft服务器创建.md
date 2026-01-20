---
layout: post
title: Minecraft服务器创建
categories: Games-server
description: null
keywards: games,server,java
mermaid: true
mathjax: true
---

Minecraft服务器的创建及部分答疑

# 一、原版纯净服务器的创建
1. 从 https://mcversions.net/ 下载所需版本的服务器核心
2. &emsp;
    1. 直接开服
        <font color=blue>此方法无法调整内存大小限制，且未对低版本的可行性进行测试，建议使用脚本开服</font>
       1.  直接双击server.jar
       2. 确认ELUA协议
       3. 再次启动server.jar
    2. 编写开服脚本
        ```
        @echo off
        java -Xmx最大内存 -Xms最小内存 -jar server.jar nogui
        pause
        ```
## 可能遇到的问题
1. java版本不匹配导致的闪退：
      手动设置对应java版本的路径或将对应版本设为系统全局变量
      ```
      "java.exe所在路径"
      ```
      1.20.5 及以上:  Java 21
      1.18 - 1.20.4:  Java 17
      1.16.5 及以下:  Java 8


<br />

# 二、forge/fabric端服务器的创建
1. 从 https://files.minecraftforge.net/net/minecraftforge/forge/ 下载所需版本对应的服务器核心，若网络问题无法加载广告 可复制下载链接将前半部分移除直接下载
2. <font color=blue><b>(forge1.17以前或fabric)</b></font>:
   使用开服脚本进行开服，大体如原版，只需将server.jar替换为你所安装的forge/fabric核心名
3. <font color=blue><b>(forge1.17以后)</b></font>
   开服脚本需要略加修改，添加参数引导
   
   <pre>
   <code>
   java @user_jvm_args.txt @libraries/net/minecraftforge/forge/<font color=blue>forge详细版本(可在libraries内找到)</font>/win_args.txt %*
   //最后一项为系统参数文件，若为linux/Mac系统则换为unix_args.txt
   </code>
   </pre>

### 可能遇到的问题
1. java版本不匹配导致的闪退:
   解决方法同上，但java引导需与参数引导位于同一行，如:

   <pre>
   <code>
   "<font color=blue>java所在路径</font>" @user_jvm_args.txt @libraries/net/minecraftforge/forge/<font color=blue>forge详细版本(可在libraries内找到)/win_args.txt %*
   </code>
   </pre>
