---
title: 宝塔面板 PM2 管理器启动项目时出现 ecosystem.config.js Not Found 的解决办法
date: 2021-09-29 00:13:00
tags: 
  - pm2
  - nodejs
  - 宝塔面板
categories: 无用教程
permalink: archives/128/index.html
---

## 问题描述
本人 Node.js 版本 16.7.0，在使用宝塔面板的 PM2 管理器添加并启动项目时出现了很奇怪的问题：

第一次启动项目时，一切正常，项目被正常添加，也被正常启动了。但是如果你因为某些原因停止了这个项目，之后重新启动就会出现 `echosystem.config.js Not Found` 的错误，而且是只有在启动的时候会出现这个问题，重启或停止都不会有报错。

## 解决方法
一开始我以为是我没有去项目目录执行 `pm2 init` 的问题，于是 RDP 进服务器迅速初始化之后回来再启动，发现还是一样的报错。

于是我打算看看是不是面板的问题。打开面板的 `应用商店` 选项卡，找到 `PM2 管理器`，打开插件的安装目录。双击 `pm2_main.py`，跳转到第 490 行。你应该会看见如下代码：

```python
result = public.ExecShell(self.__SR + 'pm2 start  --name="' + get.pname + '"');
```

把这行代码中的 `--name=` 去掉就行了，处理后看起来应该是这个样子的：

```python
result = public.ExecShell(self.__SR + 'pm2 start "' + get.pname + '"');
```

之后保存，再尝试启动，应该就没问题了。
