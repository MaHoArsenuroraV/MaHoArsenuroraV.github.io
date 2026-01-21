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
### 核心下载
从 <https://mcversions.net/> 下载所需版本的服务器核心
### 开服
1. 直接开服
   <font color=blue>(此方法无法调整内存大小限制，且未对低版本的可行性进行测试，建议使用脚本开服)</font>
      1.  直接双击server.jar
      2. 确认ELUA协议
      3. 再次启动server.jar
2. 编写开服脚本
   ```
   @echo off
   java -Xmx最大内存 -Xms最小内存 -jar server.jar nogui
   pause
   ```
   其余步骤同上

### 可能遇到的问题

1. 通用debug方式:
   删去脚本中的@echo off和pause以保持终端和返回便于查看报错并做针对性修改
2. java版本不匹配导致的闪退：
   手动设置对应java版本的路径或将对应版本设为系统全局变量
   ```
   "java.exe所在路径"
   ```

   | mc版本 | 对应java版本 |
   | :--- | :--- |
   | 1.20.5 及以上 | Java 21 |
   | 1.18 - 1.20.4 | Java 17 |
   | 1.16.5 及以下 | Java 8 |


<br />

# 二、forge/fabric端服务器的创建

### 核心下载
从 <https://files.minecraftforge.net/net/minecraftforge/forge/> 下载所需版本对应的服务器核心，若网络问题无法加载广告 可复制下载链接将前半部分移除直接下载

### 开服
1. <font color=blue><b>(forge1.17以前或fabric)</b></font>:
   使用开服脚本进行开服，大体如原版，只需将server.jar替换为你所安装的forge/fabric核心名
2. <font color=blue><b>(forge1.17以后)</b></font>
   开服脚本需要略加修改，添加参数引导

   <pre>
   <code>
   java @user_jvm_args.txt @libraries/net/minecraftforge/forge/<font color=blue>forge详细版本(可在libraries内找到)</font>/win_args.txt %*
   //最后一项为系统参数文件，若为linux/Mac系统则换为unix_args.txt
   //最大最小内存等参数需要通过自动生成的user_jvm_args.txt中进行修改
   </code>
   </pre>

### 可能遇到的问题

1. java版本不匹配导致的闪退:
   解决方法同上，但java引导需与参数引导位于同一行，如:

   <pre>
   <code>
   "<font color=blue>java所在路径</font>" @user_jvm_args.txt @libraries/net/minecraftforge/forge/<font color=blue>forge详细版本(可在libraries内找到)</font>/win_args.txt %*
   </code>
   </pre>

# 三、插件服务器的创建(原版)

### 插件服务器核心
[spigot核心下载](https://getbukkit.org/download/spigot)
[paper核心下载](https://papermc.io/downloads/)

### 插件的获取
- [SpigotMC](https://www.spigotmc.org/resources/categories/spigot.4)
- [Bukkit](https://dev.bukkit.org/bukkit-plugins)
不同插件所支持的服务端和版本不尽相同
位于(服务器根目录/plugins/对应插件名)的文件夹用于存放插件的配置文件、语言文件、数据库文件等，会自动生成

### 开服
与原版相同，核心名替换为对应即可

# 四、混合型(Mod+插件)服务器的创建

**<font color=red>警告:不建议新手尝试创建混合型服务器，可以使用模组代替插件的功能；Mod和插件的运行逻辑完全不同，混合端是靠魔改代码强行让它们兼容的，这可能会导致数据错乱，体现在：物品丢失、事件冲突等，且难以修复</font>**  
<br/>

以下是一些插件的替代建议:  

| 对应的插件功能 | 传统插件 | 可用Mod |
| :---: | :---: | :---: |
| 基础指令 (tpa, home, spawn) | EssentialsX | FTB Essentials / MaEssentials |
| 权限管理 (VIP, OP权限) | GroupManager / LuckPerms | LuckPerms |
| 领地/圈地 | Residence | FTB Chunks / FLAN |
| 经济系统 | Vault / Essentials Economy | Goodall / Lightman's Currency |
| 性能优化 | ClearLag | LaggGoggles / Spark |

### 核心下载
- [Arclight](https://github.com/IzzelAliz/Arclight)  
- [catserver (仅支持1.12)](https://github.com/Luohuayu/CatServer/tree/1.12.2)

### 开服
与原版相同，核心名替换为对应即可