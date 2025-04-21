---
title: 在 Linux 上运行 Windows 版网易云音乐
date: 2025-04-01 18:43:00
tags:
  - 网易云音乐
  - Wine
  - Linux
categories: 无用教程
permalink: archives/190/index.html
---

考虑到 Linux 版网易云音乐已经有许久没有更新，于是尝试花了点时间配置了一下在 Wine 上运行，特写下这篇博客以做记录。

## 0. 前期配置

考虑到网易云音乐是一个 32 位程序，你可能会在运行时出现一些问题（如程序黑屏无显示、没有任何声音等）。如果出现了这样的问题，你可能需要考虑安装部分 32 位库：

```shell
## 32 位 Pipewire 音频库，具体安装哪个库请根据你系统所实用的音频库决定
sudo pacman -S lib32-pipewire
```

## 1. Wine 及其配置

### 1.1. 安装 Wine

除了 Wine 本体，还建议安装 Winetricks，方便安装一些字体和依赖：

```shell
sudo pacman -S wine winetricks
```

### 1.2. 配置 Wine

启动 Winetricks，创建一个 32 位容器。

#### 1.3. 安装字体 

安装 `arial`、`courier` 和 `cjkfonts` 这三个基础字体即可。

#### 1.4. 安装系统组件

安装 `allcodecs`、`d3dx9`、`dxvk`、`winhttp` 和 `wininet` 即可。

## 2. 安装与配置网易云音乐

### 2.1. 获取安装包

最新版安装包可直接去 [官网](https://music.163.com/##/download) 下载，如果有历史版本的需求可以前往 [这个博客](https://blog.amarea.cn/archives/netease-cloudmusic-history-version.html) 寻找需要的版本。

### 2.2. 安装网易云音乐

在 Winetricks 中选择 `Run an arbitrary executable`，然后选择刚刚下载好的安装包，根据正常流程安装即可。

### 2.3. 配置网易云音乐

打开网易云后，先不要进行操作，进入`设置`->`关于网易云音乐`，先将自动更新关闭。

然后前往 `播放`，将输出设备更改为 `WaveOut` 或 `Wasapi` 即可正常输出音频。

如果部分显示出现了异常，也可以前往 `常规`，将 `禁用动画效果` 和 `禁用 GPU 加速` 打开。

### 2.4. 快速启动

前往 Wine 的 prefix 目录（一般在 `${HOME}/.local/share/wineprefix`），进入刚刚创建的容器，然后新建一个 sh 文件：

```sh
##!/bin/bash

export WINEARCH=win32
export WINEPREFIX="${HOME}/.local/share/wineprefixes/netease-cloudmusic"

export INSTALLEDPATH="${WINEPREFIX}/drive_c/Program Files/NetEase/CloudMusic"
export EXEPATH="${INSTALLEDPATH}/cloudmusic.exe"

wine "${EXEPATH}"

```

之后需要启动网易云音乐只需要运行这个 sh 即可。

## 3. 配置沙盒

TODO

## 4. 使用 BetterNCM

TODO
