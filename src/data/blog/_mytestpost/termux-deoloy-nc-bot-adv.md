---
title: 如何在 termux 部署 Astrbot 聊天机器人
author: 行简
pubDatetime: 2025-11-3T22:13:00+08
slug: termux-deoloy-bot-adv
featured: false
draft: false
tags:
  - Termux
  - napcat
  - bot
  - astrbot
description: (省流版)如何在 termux 部署 NapCat QQ 和 Astrbot 聊天机器人。
---

## 前言

### 注意：本文为省流版，小白请看 -> [基础教程](/posts/termux-deoloy-bot)

本文主要是指导如何**快速从零**在 termux 部署 NapCat QQ 和 Astrbot 聊天机器人。便于有基础的读者快速部署，相较于基础教程，本文要求会更高，但是也更简洁

请确保你应该有以下基本知识:
- 有 MacOS/Linux 任一系统的命令行使用经验
- 掌握 nano/vim、curl、wget、git、screen 等命令行工具基本用法
- 有完整 Python 项目部署经验，了解 uv 基础命令和使用方法
- 了解计算机网络基础
- 能使用至少一种代理软件，如 Clash，确保能正常访问 Github


## 1、下载 ZeroTermux

`Termux` 是安卓设备的一个终端模拟器，一个仿真的 Linux 环境。允许你在安卓设备使用 shell 命令、使用 git、配置编程环境、安装 proot Linux 容器等等。本文使用 Termux 的社区版本 ZeroTermux，并统一使用 `Debian12 CLI` 数据包作为系统镜像，确保环境一致。

首先，打开 ZeroTermux 官网，点击下载即可：

 [ZeroTermux 官方网站 - 更简单强大的安卓终端模拟器](https://zerotermux.dev/)

## 2、下载镜像

### 方式一、直接使用 ZeroTermux 下载站下载

1. 右滑打开侧边栏(或`音量+`键)
2. 找到 `线上功能`
3. 点击 `下载站`
4. 下载 `Debian12_单命令行 CLI` 数据包
5. 下载后，回到 `侧边栏` - `常用功能` - `备份/恢复` - `恢复`，找到刚刚下载的 `Debian12_cli.tar.gz`，点击创建恢复的系统容器，随便起名，确定，等待全部恢复完成
6. 右滑打开 `侧边栏` - `常用功能` - `容器切换` - 点击你刚刚命名的容器 - `切换` - `需要`
7. 现在，你应该就进入一个已经安装好 `Debian12` 容器的环境中了

### 方式二、通过远程服务器下载一键包

1. 打开 `ZeroTermux`
2. 输入命令并回车:
	```bash
	curl -sSL https://gist.githubusercontent.com/timetetng/789c163172db5cb814296ee3ef00c99e/raw/main | bash
	```
3. 下载后，回到 `侧边栏` - `常用功能` - `备份/恢复` - `恢复`，找到刚刚下载的 `Debian12-uv-python一键包.tar.gz`，点击创建恢复的系统容器，随便起名，确定，等待全部恢复完成
4. 右滑打开 `侧边栏` - `常用功能` - `容器切换` - 点击你刚刚命名的容器 - `切换` - `需要`
5. 现在，你应该就进入一个已经安装好 `Debian12` 容器的环境中了，同时这个容器还预装了 `uv` 和 `python` 环境
6. 通过命令 `start` 进入 `Debian12` 容器，接下来请跳转到章节: [`下载 Astrbot`](#下载-astrbot) 即可

## 3、进入容器

在 Termux 中输入 `ls` 查看当前目录的文件

```bash
~ $ ls                                
Debian  start.sh  storage
```

可以看到 `Debian12` 容器的目录和一个启动脚本 `start.sh`，启动脚本：

```bash
bash ./start.sh
```

显示

```bash
Hi Debian12
root@localhost:~#
```

说明已经处于容器中了，此时输入命令

**在正式开始下一步前，可以花一分钟快速阅读下面信息**

### 常见问题

> Q: 如何退出 `Debian12` 容器到 Termux 环境中？
> 
> A: 输入命令 `exit` 并回车即可

> Q: 平时如何进入 `Debian12` 容器？
> 
> A: 在 Termux 根目录中，输入命令 `bash ./start.sh` 即可

### ZeroTermux 使用技巧

1. 双指同时点按屏幕呼出`剪贴板`，下面安装中需要频繁使用
2. 单指快速双击屏幕呼出`快捷命令`
3. 长按选中命令 - 添加命令，可将此命令保存到`快捷命令`中
4. 左滑呼出菜单栏，右滑呼出文件管理器，音量键亦可
5. (**可选**)如何在外部访问 zerotermux 文件目录: 
	 - 打开[MT管理器](https://mt2.cn/) - 左侧菜单栏 - 左上角三个点 - 添加本地存储 - 点击左上角呼出侧边栏 - 点击`ZeroTermux`- 使用此文件夹

### 使用 SSH 连接到 Termux（可选）

对于熟练使用 ssh 的读者，可以将手机和电脑连接到同一网络后，在电脑终端或 ssh 客户端中直接 ssh 连接到 Termux 终端操作，会更加方便

1. 首先，Termux 安装 `OpenSSH`，请确保退出 Debian 容器回到 Termux 外部
	```bash
	# 如果你还 Debian 容器中，请先退出
	# exit
	# (可选) pkg 换源，在左侧边栏中点击切换源-清华源-一路y即可。不过你只安装一个包不换源也无所谓
	
	# 安装 OpenSSH
	pkg install -y openssh
	```
2. 启用 ssh 服务
	```bash
	sshd
	```
3. 设置 ssh 密码
	```bash
	passwd
	```
4. 在你电脑主机终端使用命令连接，或者用你熟悉的 SSH 工具亦可
	```bash
	ssh u0_xxx@192.168.xx.xx -p 8022
	```
	此处用户名`u0_xxx`可以在 termux 中输入 `whoami` 查看，IP 可通过命令 `ifconfig` 查看，或者直接在左侧边栏查看，注意默认端口是 `8022` 而不是 Linux 的 22。
## 4、`apt` 换源

`apt` 是 Debian 系 Linux 发行版的软件包管理器，几乎所有常用软件都可以通过 `apt` 来安装、卸载、更新等等。但是由于国内网络环境问题，默认 `apt` 下载源访问较慢，甚至版本过旧，因此新的系统应该先进行换源。这里推荐[中国清华大学源](https://mirrors.tuna.tsinghua.edu.cn/help/debian/)：

### 步骤：

1. 确保在 Debian 容器中(即显示 `root@localhost` )
2. 点开上面的网站，下滑找到 `DEB822 格式` 一栏
3. Debian 版本切换到 `Debian12 (bookworm)`，其他默认即可
4. 复制那一大段代码，类似（下面仅为笔者编辑时的版本，实际可能不一样，请以清华源最新版本为准）:
	```
	Types: deb
	URIs: https://mirrors.tuna.tsinghua.edu.cn/debian
	Suites: bookworm bookworm-updates bookworm-backports
	Components: main contrib non-free non-free-firmware
	Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
	
	# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
	# Types: deb-src
	# URIs: https://mirrors.tuna.tsinghua.edu.cn/debian
	# Suites: bookworm bookworm-updates bookworm-backports
	# Components: main contrib non-free non-free-firmware
	# Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
	
	# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
	Types: deb
	URIs: https://security.debian.org/debian-security
	Suites: bookworm-security
	Components: main contrib non-free non-free-firmware
	Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
	
	# Types: deb-src
	# URIs: https://security.debian.org/debian-security
	# Suites: bookworm-security
	# Components: main contrib non-free non-free-firmware
	# Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
	```
1. 使用 nano 或 vim 打开 `/etc/apt/sources.list.d/debian.sources`
	```bash
	nano /etc/apt/sources.list.d/debian.sources
	```
2. 粘贴所有内容，Ctrl + X 保存，再按 y 确认修改并退出
3. 更新源
	```bash
	apt update
	```
4. 安装常用软件<sup>[1]</sup>
    ```bash
    apt install -y curl wget git screen
    ```

## 5、安装 uv

`uv` 是一个由 `rust` 编写的现代化、高性能的 python 包管理器，不仅能方便的创建虚拟环境、管理依赖, uv 还能自动根据项目依赖下载对应 python 版本。本文就没有下载 python 和 pip，而是完全交给 uv。

### 官方脚本一键安装 

```bash
# 下载脚本并安装 uv,耐心等待安装完成
curl -LsSf https://astral.sh/uv/install.sh | sh
# 加载到环境变量或手动重启终端
source $HOME/.local/bin/env
# 验证安装
uv --version
uvx --version
```

输出版本号说明安装 uv 成功！

## 6、安装 Astrbot


### 下载/克隆仓库

1. 进入你需要安装的目录，根据需要自己修改，这里直接安装到 /root 目录下：

	```bash
	cd ~
	```

2. 从 Github 克隆 Astrbot

	```bash
	# 直接从 Github 克隆 Astrbot 仓库
	git clone https://github.com/AstrBotDevs/AstrBot.git
	
	# 进入 AstrBot 目录
	cd AstrBot
	```

### 使用 uv 安装依赖

在 AstrBot 目录下，理想情况下能安装成功，但是 termux 环境不是真正的 Linux 环境，所以会因为权限问题导致 uv 的硬链接失败。为了稳妥，修改环境变量为复制模式再安装依赖：

```bash
# 修改环境变量为复制模式
export UV_LINK_MODE=copy
# 同步依赖
uv sync
```

uv 会自动安装 Python 并创建虚拟环境管理，等待所有依赖安装完毕即可。

>[!TIP]
>为了后续使用 uv 安装插件时，不出现 uv 硬链接安装失败报错，可以将其加入系统环境变量中：
>```bash
>echo "export UV_LINK_MODE=copy" >> ~/.bashrc
>source ~/.bashrc
>```


### <a id="astrbot"></a>启动 Astrbot
<a id="astrbot"></a>
安装好依赖以后就可以启动 Astrbot 了！

Termux 没有完整安卓系统的守护进程权限。因此无法使用守护进程后台运行 Astrbot。这里使用原始办法，通过终端复用工具挂载在后台进程。推荐使用 `screen` 或者 `tmux` 创建会话，以便让应用分离到后台运行。

[1]:之前有安装过 `screen`，下面以此为例示范如何启动:

#### 后台启动（使用 `screen`）

```bash
# 创建 astrbot 会话
screen -S astrbot
```

在会话中启动 Astrbot

```bash
# 进入安装目录
cd ~/AstrBot
# 启动
uv run main.py
```

确认启动完毕后，使用快捷键同时按住 `Ctrl + a` 再按 `d` 键分离会话即可。重进会话只需要 `screen -r 会话名`

不要关闭 ZeroTermux ,确保其后台运行，打开手机浏览器，地址栏输入: `http://localhost:6185` 访问，能访问说明 Astrbot 启动成功！

其他设备可通过`http://手机内网IP:6185`访问，Napcat 侧同理，不再重复。

> 默认用户名和密码均为: astrbot

首次登录需要修改用户名、密码并重新登录。

### 配置 Astrbot 
这里只讲如何连接 NapCat QQ，其他配置自行查看 [AstrBot 官方文档](https://docs.astrbot.app/)
<img src="/assets/astrbot-config.png" style="float: right; margin-left: 15px; width: 250px;">



#### 步骤


1. 点击左侧菜单-平台适配器/机器人
2. 创建机器人
3. 平台选择 `QQ个人号`
4. id 随意填，勾选启用，反向 websocket 主机和端口保持默认。反向 websocket Token 自行填写，相当于连接密码，**待会 NapCat 侧要填一样的**
5. 配置好请保存并启用。

接下来安装并配置 NapCat 。
<div style="clear: both;"></div>



## 7、安装 NapCat 

### 使用官方一键脚本

```bash
# 回到家目录
cd ~

# 下载脚本并安装
curl -o \
napcat.sh \
https://nclatest.znin.net/NapNeko/NapCat-Installer/main/script/install.sh \
&& sudo bash napcat.sh \
--docker n \
--cli y \
--force
```

一般情况能一次成功，如果因为网络或者其他原因意外中断，**重新运行命令**即可。

### 启动 NapCat 

建议使用 CLI 命令启动，如需 TUI 界面，直接输入 `napcat` 回车即可。下面以 CLI 为例:

先大致查看所有 CLI 命令
```bash
# 列出 CLI 帮助命令
napcat help
```

启动 Napcat

```bash
napcat start QQ号
```

若无报错，说明启动成功，开始下一步扫码登录！
### 配置 NapCat 

#### 1、扫码登录

启动后，使用 TUI 或者 CLI **打开日志**，通常能看到二维码，通常直接截图日志然后本机扫码，也可以使用另一个设备扫码登录。若二维码过期或扫码登录失败，重启 NapCat 再获取即可。

**注意**: 请使用 Bot 账号扫码，如果重启扫码仍然失败，请用**前置摄像头**重试！

#### 1.1 如何获取二维码（已扫跳过）
考虑到部分终端二维码很难扫，还有几种获取二维码的方式:

1.  使用 TUI 或者 CLI **打开日志** 扫码
2. （懒狗）复制下面命令到终端，并按要求操作即可：
	```bash
	echo -n "请输入你的QQ号后回车: "; read qq; grep "解码URL" ~/Napcat/log/napcat_${qq}.log | tail -n 1 | awk '{print "请用浏览器打开: https://qrcode.lsgbin.com/api/generate?url=" $2}'
	```
3. 先完成下一步`获取 WebUI 初始 token`，然后在 WebUI 点击扫码登录
4. 将二维码图片复制到手机其他位置，然后用文件管理器(推荐 mt 管理器)打开。这里复制到标准下载目录 `~/手机内部存储/Download/`:
	```bash
	cp ~/Napcat/opt/QQ/resources/app/app_launcher/napcat/cache/qrcode.png ~/手机内部存储/Download/
	```

#### 2、获取 WebUI 初始 token

启动后，首先在终端输入命令

```bash
cat ~/Napcat/opt/QQ/resources/app/app_launcher/napcat/config/webui.json | grep  "token"
```

或者

```bash
echo -n "请输入你的QQ号后回车: "; read qq; grep "WebUi Token" ~/Napcat/log/napcat_${qq}.log
```

可以看到 WebUI 的初始 token，浏览器访问 `http://localhost:6099/webui`, 填写上面获取的 token 即可登录，首次登录需要修改密码(token)并重新登录。

#### 3、配置 WebUI 

浏览器访问 `http://localhost:6099/webui` 并登录，按以下步骤配置：

1. 打开侧边栏 - 其他配置 - 登录配置 - 填写 bot QQ - 保存，以便将来无需扫码登录；
2. 打开侧边栏 - 其他配置 - 登录配置 - 修改密码，旧密码输入默认 token，设置新密码，保存并重新登录；
3. 打开侧边栏 - 网络配置 - 新建 - Websocket Client，名称自定义，Token 填写**之前 Astrbot 那边设置的**，其他完全按照下面配置填写：
	- URL: `ws://localhost:6199/ws`
	- 是否上报自身消息: 默认关闭即可
	- 消息格式: `Array`
	- 心跳间隔: `5000`
	- 重连间隔: `5000`
4. 最后保存并启用。
5. (可选)发送几条消息，分别看看 Napcat 和 Astrbot 日志有没有显示，均显示证明连接成功。也可以直接发送命令 `/help` 测试是否回复来验证。连接失败请自行检查上述步骤是否全部正确，**仍然失败请携带 NapCat 日志、Astrbot 日志、部署方式和你的具体做了哪些步骤后，前往群聊提问。**

## 结语

完成以上步骤，你应该能成功安装并连接 NapCat 和 Astrbot，接下来请前往官方文档完成其他配置，尤其是 Astrbot 需要配置 LLM 相关以及添加管理员等等，Napcat 通常无需额外配置。下面是你可能需要的文档:

- [AstrBot 文档](https://docs.astrbot.app/)
- [NapCatQQ | 现代化的基于 NTQQ 的 Bot 协议端实现](https://napneko.icu/)
- [uv 中文文档](https://uv.doczh.com/)
- [Termux Wiki](https://wiki.termux.com/wiki/Main_Page)

此外，Termux 在一些机型容易被杀后台，请将省电策略调整成**无限制**，并尽可能提高其权限，开启允许自启动等等。具体参考你的机型软件保活教程。
### 日常使用启动流程

1. 打开 Zerotermux
2. 进入容器，用户名变为 root 说明处于容器中
	```bash
	# 进入 Debian12 容器
	bash start.sh
	```
3. 启动 NapCat
	```bash
	napcat start qq号
	```
	如果需要扫码或者查看日志
	```bash
	napcat log qq号
	```
4. 启动 Astrbot
	```bash
	# 创建并进入一个screen会话
	screen -S astrbot
	```
	在会话中启动 Astrbot
	```bash
	# 进入 Astrbot 目录
	cd AstrBot
	uv run main.py
	```
	Ctrl + a + d 分离会话到后台
5. 一切就绪，使用即可！
	- Astrbot WebUI: `http://localhost:6185`
	- NapCat WebUI: `http://localhost:6099/webui`

### 安装插件

```bash
# 进入插件目录
cd ~/AstrBot/data/plugins
```

使用 git 安装插件

```bash
git clone [插件仓库URL]
```

安装依赖(如果需要)

```bash
uv add 包名1 包名2 包名3
```


### 退出流程

```bash
exit
```

> 补充: 如果你感兴趣，还可以研究如何将上述流程写一个一键启动脚本，结合 ZeroTermux 的快捷命令，可以很方便一键启动上述两个应用。