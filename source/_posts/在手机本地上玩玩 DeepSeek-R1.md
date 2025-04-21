---
title: 在手机本地上玩玩 DeepSeek-R1
date: 2025-01-28 15:52:00
tags:
  - ollama
  - DeepSeek
categories: 无用教程
permalink: archives/185/index.html
---

[DeepSeek-R1](https://huggingface.co/deepseek-ai/DeepSeek-R1) 的发布可以说引爆了整个科技圈的话题。今天没事干的时候手动部署了一个玩玩，发现还不错，至少数学推理比隔壁的 [Microsoft Copilot](https://copilot.microsoft.com/) 强。

![2025-01-28T07:12:37.png][1]

那么一个问题就来了：**我能不能试试本地部署一个 DeepSeek-R1 玩玩？**

## 准备工作

我使用的是安卓手机，正好也在使用 [Termux](https://termux.dev/)，于是我就去网上搜索了一下有没有在 Termux 上部署成功的先例。

搜索了一下，还真让我发现了一个辅助项目：[Dev-ing-ing/ollama-termux](https://github.com/Dev-ing-ing/ollama-termux)，它直接在 Termux 中编译 [Ollama](https://ollama.com/) 并安装二进制产物，以达到正常使用的目的。

于是整个流程就清晰很多了：更新 Termux 后安装 Ollama，然后导入 DeepSeek-R1 即可直接上手。

### 1. 下载模型

由于我的手机性能还不错，所以我直接下载了 [8B 模型](https://huggingface.co/bartowski/DeepSeek-R1-Distill-Llama-8B-GGUF)。当然，对于一般的手机来说，[7B](https://huggingface.co/bartowski/DeepSeek-R1-Distill-Qwen-7B-GGUF) 甚至 [1.5B](https://huggingface.co/bartowski/DeepSeek-R1-Distill-Qwen-1.5B-GGUF) 的模型可能会更适合一些。

下载完模型后，需要在模型同目录下创建一个 `Modelfile` 文件，文件的大致内容看起来像这样：

```
FROM ./DeepSeek-R1-Distill-Llama-8B-Q4_K_L.gguf
```

其中 `DeepSeek-R1-Distill-Llama-8B-Q4_K_L.gguf` 是你下载的模型的文件名（带文件后缀）。如果你下载的不是 8B 的模型，那么你的就需要用你所下载的模型名替换掉它。

然后，将这两个文件保存在你手机里方便找到的地方（最好先新建一个文件夹），稍后我们会用到。

### 2. 准备 Termux

下载安装好 Termux 后，我非常建议先使用如下命令更新 Termux 自带的各种包：

```shell
$ apt update && apt upgrade -y
```

如果在更新时遇到了网络问题，可以先使用 `termux-change-repo` 命令更换包镜像源（建议在 `Settings`->`Terminal I/O` 中打开 `Soft Keyboard` 以方便使用方向键）。

更新完毕后，使用下面的命令安装 `wget`：

```shell
$ apt install wget -y
```

安装完成后，使用如下指令安装 Ollama（可能需要更好的网络环境）：

```shell
$ wget https://github.com/Dev-ing-ing/ollama-termux/releases/latest/download/ollama-installer.sh && bash ollama-installer.sh
```

结束后，如果使用 `ollama` 命令会出现帮助信息则代表安装成功。

最后，使用如下指令，并在弹出的授权窗口中授予 Termux 使用所有文件的权限，即完成了对 Termux 的配置：

```shell
$ termux-setup-storage
```

### 3. 导入模型

Termux 会将存储空间映射到 `~/storage` 文件夹中，Android 规范目录会直接映射到这个文件夹内（全小写），而存储根目录会映射到 `shared` 文件夹内。

首先运行以下命令启动 Ollama：

```shell
$ ollama serve
```

启动后，点击小键盘上的菜单键，然后点击 `New Session` 启动新会话。

接下来，在新会话中运行以下命令以导入模型：

```shell
$ ollama create DeepSeek -f /path/to/model/Modelfile
```
其中 `DeepSeek` 是模型在 Ollama 中的名字，由用户命名，你也可以修改成其他的名称方便记忆。

导入模型后，运行如下命令使用模型：

```shell
$ ollama start DeepSeek
```

其中 `DeepSeek` 就是你刚才导入的模型名字了。

等待处理完成后，你就可以开始把玩了！

![QQ图片20250128154936.jpg][2]

如果需要打断模型说话，按下小键盘上的 `Ctrl` 键，然后再按下键盘上的 `C` 键即可。如果需要退出 Ollama，先在模型内输入 `/bye` 命令，然后输入 `exit` 退出当前会话，最后在最开始的会话中使用刚才的 `Ctlr+C` 组合键就可以了。

  [1]: /static/images/2025/01/3362990995.png
  [2]: /static/images/2025/01/4072372110.jpg
