---
title: 使用国内外双机器和宝塔面板搭建中转 Socks5 代理
date: 2021-09-16 15:25:00
tags:
  - socks5
  - 代理
  - Telegram
categories: 无用教程
permalink: archives/127/index.html
---

## 前言
一直想搭建一个 Telegram 的代理以实现无梯聊天。可惜在宝塔环境下的教程不多，也没有针对 Windows Server 的教程，于是便自行参考了一些现有的教程琢磨，便有了这个教程。

## 提前警告！
本教程仅供学习参考，严禁使用本教程进行违法犯罪活动。本人不承担使用不当造成的任何后果。

## 开始之前
感谢 [@ginuerzh](https://github.com/ginuerzh) 大佬编写的 [Gost](https://github.com/ginuerzh/gost)。[Gost](https://github.com/ginuerzh/gost) 是一个使用 Golang 语言编写的简单隧道搭建程序，你可以点击上述名称以进入该程序的相关仓库。

本教程参考了 [@KANIKIG](https://github.com/KANIKIG) 大佬的 [这篇文章](https://blog.kanikig.xyz/%E7%9B%AE%E5%89%8D%E6%9C%80%E7%A8%B3%EF%BC%81%E4%BD%BF%E7%94%A8Gost%E9%9A%A7%E9%81%93%E4%B8%AD%E8%BD%AC%E6%90%AD%E5%BB%BATelegram%E4%BB%A3%E7%90%86/)。如果你没有安装宝塔面板，或者你使用的服务器均是 Linux 服务器，亦或不想手动部署，那么建议你参考这篇文章。

## 搭建的环境
* 一台坐标海外的落地服务器，使用的系统是 CentOS 7。
* 一台坐标上海的转发服务器，使用的系统是 Windows Server 2016。

由于环境的差异，实际操作的步骤可能有不同，请自行斟酌。

## 开始部署
### 落地服务器
* 在 [Gost 的 Release 页](https://github.com/ginuerzh/gost/releases) 选择适合自己服务器的 Gost 软件版本下载。你也可以选择 [自行 Clone 源码编译](https://github.com/ginuerzh/gost##%E6%BA%90%E7%A0%81%E7%BC%96%E8%AF%91)。
* 登录宝塔面板，安装`Supervisor 管理器`。
* 转到`文件`选项卡，寻找/新建一个方便的目录，将 Gost 程序上传至此。
* 新建`config.json`文件，在该配置文件中填入下列设置项：

```json
{
    "Debug": true,
    "Retries": 0,
    "ServeNodes": [
        "socks5://username:password@:port1"
    ],
    "Routes": [
        {
            "Retries": 0,
            "ServeNodes": [
        		  "relay+ws://:port2/127.0.0.1:port1"
            ]
        }
    ]
}
```

其中：
* `username`：代理的登录用户名
* `password`：代理的登录密码
* `port1`：Socks5 代理的对外端口，可随意指定
* `port2`：用于接收转发服务器数据的端口，你需要记住该端口以在配置转发服务器时使用

填写完设置项后保存该文件。有关更多设置项，你可以访问 [Gost Wiki](https://docs.ginuerzh.xyz/gost/) 来了解更多。

注意：你可能需要在宝塔面板和/或服务器防火墙中放行相关端口后才可使用。

* 转到`Supervisor 管理器`，新建一个守护项目。

相关参数填写参考：

* `名称`：可以随意填写，如`Gost`，但请不要带有空格和特殊字符
* `启动用户`：保持默认即可，除非你喜欢折腾
* `运行目录`：存放 Gost 程序及其配置文件的目录
* `启动指令`：填写`gost -C config.json`。更多可用参数请参阅 [Wiki](https://docs.ginuerzh.xyz/gost)
* `进城数量`：保持默认值`1`

全部填写完后提交保存。如无意外，你应该会看见 Gost 的运行状态为`RUNNING`。如果为其他状态，请自行查阅程序日志报错并排错。

* *（这一步不是必须的，但推荐你先试一下）* 打开 Telegram，新建一个 Socks5 代理，分别填入你的落地服务器的 IP 地址、Port1、代理用户名、代理密码，填写完毕后保存并尝试连接。如果能成功链接，请 **立即断开链接** 并进行下一步，否则请检查哪里出了问题。

### 转发服务器
* *略过下载程序等步骤。由于 Windows 系统的特殊性，故我们直接编写`.bat`文件运行，而无需安装 Supervisor*
* 在`config.json`中填写如下内容：

```json
{
    "Debug": true,
    "Retries": 0,
    "Routes": [
        {
            "Retries": 0,
            "ServeNodes": [
                "tcp://:port3",
	            "udp://:port3"
	        ],
	        "ChainNodes": [
	            "relay+ws://server1:port2"
            ]
        }
    ]
}
```

其中：

* `server1`：你的落地服务器的 IP 地址
* `port2`：同你在配置落地服务器时的`port2`
* `port3`：你在 Telegram 客户端要输入的端口，可随意指定

填写完设置项后保存该文件。有关更多设置项，你可以访问 [Gost Wiki](https://docs.ginuerzh.xyz/gost/) 来了解更多。

注意：你可能需要在宝塔面板和/或服务器防火墙中放行相关端口后才可使用。

* 新建一个用于启动 Gost 的 bat 文件，名称自定。在文件中填入以下内容：

```bat
echo off
cls
gost -C config.json
pause
```

* 保存该文件，使用 RDP 进入服务器，运行该 bat 文件，如果 Gost 启动正常，那么可进行下一步，否则请根据报错排错。
* 再次打开 Telegram，编辑之前测试用的代理设置，将服务器地址和端口号分别替换为转发服务器的 IP 和 port3，保存后尝试链接。如果能成功链接，说明配置正常，否则请检查 Gost 配置与服务器。

## 其他说明
请注意管理好你的代理用户名和密码，尽量使用高强度密码。

代理速度取决于两台服务器中最慢的那一台。
