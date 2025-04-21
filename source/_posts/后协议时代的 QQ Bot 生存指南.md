---
title: 后协议时代的 QQ Bot 生存指南
date: 2024-01-13 22:16:00
tags:
  - 酷Q
  - OneBot
  - Mirai
  - 机器人
  - QQ
categories: 无用教程
permalink: archives/142/index.html
---

**向所有维护过签名服务器的前辈们致敬，是你们让协议库又延续了至少几个月的寿命，感恩。**

**如果你追求稳定，请尽量迁移到基于官方客户端开发的项目，例如 [OpenShamrock](https://github.com/whitechi73/OpenShamrock)。**

为了继续苟且偷生，且部分资源或仓库应版权方要求已经删库，故本文章不会直接贴出部分资源或仓库的链接，请自行寻找。

## 前言

从 2023 年开始，腾讯官方就一直在对 Bot 社区进行打压围堵，导致许多 Bot 用户无法继续使用 Bot。

而在公开签名服务后，其中很多像我一样的用户依靠这些服务为 Bot 续命。

在连续折腾了几天后，我决定写下这一篇文章，为还想继续使用协议库方式登陆的用户们指一条还算明确的路。

## 开始

本文章将假设你：

* 已经在使用 [Mirai](https://github.com/mamoe/mirai) 或 [go-cqhttp](https://github.com/Mrs4s/go-cqhttp)
* 已经配置好了 Java 环境（推荐 Java 11）
* 知道你所使用的系统该如何编写和运行脚本

## 开始使用

目前 Mirai 和 go-cqhttp 都已经支持了使用外置签名服务进行签名操作。

如果您使用的是 Mirai，可以参考如何使用插件式签名服务器。

### 使用外置签名服务

本章将假设：

* 你已获取到 `fuqiuluo/unidbg-fetch-qsign` 相关的可执行文件
* 你要将 Bot 和签名服务器配置在同一台设备上
* 你将要使用的协议版本是 `8.9.78`，并且你已经拥有相关的文件

由于版权方要求，该仓库已经被删除且不再更新，故将不在此处贴出对应链接。如果您没有相关文件，请自行寻找。

#### 配置签名服务器

解压 `unidbg-fetch-qsign` 至你可以访问的目录，然后在根目录下新建 `start.sh`（Linux） 或 `start.bat`（Windows）。在脚本中输入以下内容：

`start.sh`：   
```bash
##!/bin/bash
java -cp "$CLASSPATH:./lib/*" MainKt --basePath="txlib/8.9.78"
```
`start.bat`：   
```shell
java -cp .\lib\* MainKt --basePath="txlib/8.9.78"
```

其中 `basePath` 参数中的 `8.9.78` 就是你将要使用的协议版本。

在使用该协议版本前，请先前往 `txlib` 文件夹检查是否存在该文件夹且该文件夹内是否包含 `config.json`、`dtconfig.json`、`libfekit.so` 以及 `libQSec.so`（非必须）文件，否则该协议版本无效，不能被用于启动签名服务。

之后我们打开 `txlib/8.9.78/config.json` 这个文件，修改签名服务器的配置：

```json
{
  "server": {
    // `server.host`：签名服务器监听地址，如果不想暴露给公网可以改成 127.0.0.1
    "host": "0.0.0.0",
    // `server.port`：签名服务器监听端口
    "port": 8080
  },
  // 以下两项设置 `share_token` 和 `count` 是用于一个签名服务器对应多个客户端时的配置。
  // `share_token`：用于控制当多个客户端连接至签名服务器时，客户端之间是否共用一个 token。
  "share_token": false,
  // `count`：用于控制最大客户端数量（假设一个客户端只有一个实例）。如果只有一个客户端，则控制一个客户端的最大实例数量。
  "count": 1,
  // 以上两者不会对账号安全性做出影响，在老版本的签名服务器中没有以上两个设置项。
  // `key`：连接至签名服务器时的鉴权用 key
  "key": "114514", 
  // `auto_register`：实例连接到服务器后是否自动注册
  "auto_register": true,
  // `protocol` 内的参数是自动生成的，请勿随意改动
  "protocol": {
    "package_name": "com.tencent.mobileqq", // 老版本的签名服务器无此参数
    "qua": "V1_AND_SQ_8.9.78_4548_YYB_D",
    "version": "8.9.78",
    "code": "4548"
  },
  "unidbg": {
    // `unidbg.dynarmic`：高并发请启用，但请注意控制实例数量，否则会导致内存爆炸
    "dynarmic": true,
    // `unidbg.unicorn`：未明写其作用，个人推测开启后会使用原生方法调用相关方法从而减少内存占用
    "unicorn": false,
    // `unidbg.kvm`：未明写其作用，开启后会降低内存占用
    "kvm": false,
    // `unidbg.debug`：开启后会显示详细日志，不是需要排错的话建议关闭
    "debug": false
  },
  // `black_list`：黑名单账号，在该列表中的账号将无法使用此签名服务
  // 此设置项在旧版签名服务器中无效
  "black_list": [
    1008611
  ]
}
```

调整完配置之后保存文件，记下 `server.port` 和 `token` 两个参数，稍后我们将会使用它。

然后，运行 `start.sh` 或 `start.bat` 启动签名服务器。此时打开浏览器访问签名服务器（在本例中，签名服务器的地址是 `http://127.0.0.1:28080`），你应该会看到类似下面的内容：

```json
{
    "code": 0,
    "msg": "IAA 云天明 章北海",
    "data": {
        "version": "1.1.9",
        "protocol": {
            "qua": "V1_AND_SQ_8.9.78_4548_YYB_D",
            "version": "8.9.78",
            "code": "4548"
        }
    }
}
```

如果你没有看到类似上面的内容或浏览器出现错误提示，请检查你的服务器配置和连接地址。

关于协议版本：目前对于绝大多数账号来说 `8.9.78` 及以上的版本才可以正常登陆，但这些版本文件并没有经过验证，可能存在不稳定的现象。如果你需要更新的协议版本，请自行逆向官方客户端获取。

#### 配置 Bot 客户端

本章将分成两个部分：Mirai 和 go-cqhttp。

##### 配置 Mirai

下载 [fix-protocol-version](https://github.com/cssxsh/fix-protocol-version) 插件，将其放在 Mirai 根目录的 `plugins` 文件夹内（如果可以，请搭配 [mirai-login-solver-sakura](https://github.com/KasukuSakura/mirai-login-solver-sakura) 和 [mirai-device-generator](https://github.com/cssxsh/mirai-device-generator) 使用），然后启动一次 Mirai，让插件生成对应的配置文件。

接下来，退出 Mirai，打开 Mirai 根目录下的 `KFCFactory.json` 文件，将其修改成类似下面的内容：

```json
{
    // 此处的 `8.9.78` 应当是你准备使用的协议版本。
    // fix-protocol-version 支持连接多个签名服务器，因此你可以复制本节内容并
    // 稍加修改后即可使用另一个支持不同版本的签名服务器。
    "8.9.78": {
        // 请注意修改 `base_url` 中的端口号
        "base_url": "http://127.0.0.1:8080",
        // 请不要修改这一项
        "type": "fuqiuluo/unidbg-fetch-qsign",
        // 填入你刚才在配置文件中定义的 key
        "key": "114514"
    }
}

```

然后将协议版本对应的 `android_phone.json` 和 `android_pad.json` 两个文件复制到 Mirai 根目录（可按需复制），启动 Mirai，如果没有问题此时你的 Bot 应当已经成功登陆并且可以正常收发消息了。

##### 配置 go-cqhttp

对于 go-cqhttp，我建议使用 [Actions 构建版本](https://github.com/Mrs4s/go-cqhttp/actions)，正式版本并不支持加载自定义的客户端信息，并且 go-cqhttp 的客户端信息来源是写死的，使用并不方便。

更新完 go-cqhttp 后，启动一次 go-cqhttp 以更新配置文件。

然后，退出 go-cqhttp，打开 `config.yml`，找到 `account.sign-servers` 一节，修改其内容：

```yaml
## go-cqhttp 自带的注释已经很详细了，我想我不需要再写什么了吧？
## ...
  ## 数据包的签名服务器列表，第一个作为主签名服务器，后续作为备用
  ## 兼容 https://github.com/fuqiuluo/unidbg-fetch-qsign
  ## 如果遇到 登录 45 错误, 或者发送信息风控的话需要填入一个或多个服务器
  ## 不建议设置过多，设置主备各一个即可，超过 5 个只会取前五个
  ## 示例:
  ## sign-servers: 
  ##   - url: 'http://127.0.0.1:8080' ## 本地签名服务器
  ##     key: "114514"  ## 相应 key
  ##     authorization: "-"   ## authorization 内容, 依服务端设置
  ##   - url: 'https://signserver.example.com' ## 线上签名服务器
  ##     key: "114514"  
  ##     authorization: "-"   
  ##   ...
  ## 
  ## 服务器可使用docker在本地搭建或者使用他人开放的服务
  sign-servers: 
    - url: 'http://127.0.0.1:8080'  ## 主签名服务器地址， 必填
      key: '114514'  ## 签名服务器所需要的apikey, 如果签名服务器的版本在1.1.0及以下则此项无效
      authorization: '-'   ## authorization 内容, 依服务端设置，如 'Bearer xxxx'
## ...
```
然后进入 `data/versions` 文件夹，将 `android_phone.json` 和 `android_pad.json` 分别改名为 `1.json` 和 `6.json` 复制进该文件夹中（可按需复制）。如果你在 go-cqhttp 的启动参数中添加了 `-update-protocol` 项目，请删除它。

然后启动 go-cqhttp，如果没有问题此时你的 Bot 应当已经成功登陆并且可以正常收发消息了。

如果你想的话，可以在 go-cqhttp 的配置文件的 `account` 一节中找到更多关于签名服务器的配置项，这些项目提供了更为精细的客户端行为调整，调整得当的话可能可以让你的 Bot 运行得更稳定。

### 使用 Mirai 插件

本章节参考的是 Mirai 论坛中的 [这篇帖子](https://mirai.mamoe.net/post/24989)，向楼主致敬。

到 [原帖](https://mirai.mamoe.net/post/24989) 下载好插件后按照其使用说明安装，安装完毕后直接启动 Mirai 即可。

如果需要更新协议版本，可自行获取对应文件后放置在 `txlib` 文件夹下，然后使用 `/qsign <version>` 指令切换协议版本。

## 善后

### 在 OneBot 平台上使用 Mirai 插件

对于不得已已经迁移至官方实现的用户可以尝试使用 [MrXiaoM/Overflow](https://github.com/MrXiaoM/Overflow) 转译框架。

由于该项目还在缓慢开发中，存在 Bug 和不稳定等问题，建议测试后再放入生产环境中使用。如果遇到了问题也欢迎到 [issue](https://github.com/MrXiaoM/Overflow/issues) 与开发者反馈。

### 使用 `酷Q` 插件

酷Q 早在 2020 年 7 月就已经终止了服务，由于 酷Q 本身闭源，且 Coxxs 并未公布 `*.cpk` 的加载逻辑，因此目前所有的社区第三方实现都不支持直接加载 `*.cpk` 文件。

如果你手上正好有 酷Q 插件的原始 `*.dll` 和原始 `*.json` 文件，不妨试试下面几个项目：

#### super1207/MiraiCQ

此项目同时支持 `Mirai HTTP API` 和 `OneBot v11` 实现，并可通过 `wine` 适配 Linux。

仓库地址：https://github.com/super1207/MiraiCQ

对于某些插件，可能会依赖 酷Q 自己提供的库，此时可寻找对应的库文件并将其放入 `bin` 文件夹内。

#### debumori-osc/Another-Mirai-Native

此项目仅支持 `Mirai HTTP API`。

项目地址：https://github.com/debumori-osc/Another-Mirai-Native

本项目拥有和原 酷Q 近似的用户界面，并在其基础上添加了插件调试功能（类似于 Koishi 的沙盒功能），可以让插件开发者方便地测试自己的插件。

本项目同时提供开发者已打包好的运行环境，可在 [debumori-osc/another-mirai-native-release](https://github.com/debumori-osc/another-mirai-native-release) 下载。

#### iTXTech/mirai-native

此项目仅支持 Mirai 环境，并且只支持 32 位的 Java。

项目地址：https://github.com/iTXTech/mirai-native

本项目是最早的在 Mirai 上支持 酷Q 插件的项目，但已经很久没有新的更新了。


## 结语

其实腾讯官方在最近向开发者开放了 Bot 开发（[文档](https://bot.q.qq.com/wiki/)），但是就 [Mirai 论坛用户的使用反馈](https://mirai.mamoe.net/topic/2526/qq%E7%BE%A4bot%E5%AE%98%E6%96%B9%E6%8E%A5%E5%8F%A3-%E4%B8%8E%E4%BD%BF%E7%94%A8%E4%BD%93%E9%AA%8C) 来看，目前这个 Bot 不仅审核严格、功能稀少，连 SDK 更新也不频繁。

所以我个人认为，除非万不得已，还是继续使用第三方实现吧。

愿我们能在更开放的平行世界里相遇。
