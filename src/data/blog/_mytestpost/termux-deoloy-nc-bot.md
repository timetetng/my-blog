---
title: 如何在 termux 部署 Astrbot 聊天机器人
author: 行简
pubDatetime: 2025-11-3T02:07:00+08
slug: termux-deoloy-bot
featured: true
draft: false
tags:
  - Termux
  - napcat
  - bot
description: 手把手教你如何在 termux 部署 NapCat QQ 和 Astrbot 聊天机器人。
---

## 前言

本文主要是指导如何**从零**在 termux 部署 NapCat QQ 和 Astrbot 聊天机器人。你首先应该有以下基本知识:
- 会用至少一款代理软件，如 Clash、Clash meta 等，**确保你的网络能访问 Github** (重要!)，没有的[看这](https://wangpan.yfjc.my/)
- 初中以上语文水平，能看懂本文基本逻辑
- 会打开浏览器访问某个具体网址
- 会复制粘贴，能灵活利用剪切板

## 1、下载 ZeroTermux

打开官网，点击下载即可：

 [ZeroTermux 官方网站 - 更简单强大的安卓终端模拟器](https://zerotermux.dev/)

## 2、下载镜像

### 直接使用 ZeroTermux 下载站下载

1. 右滑打开侧边栏
2. 找到 `线上功能`
3. 点击 `下载站`
4. 下载 `Debian12_单命令行 CLI` 数据包
5. 下载后，回到侧边栏 - 常用功能 - 备份/恢复 - 恢复，找到刚刚下载的 `Debian12_cli.tar.gz`，点击创建恢复的系统容器，随便起名，确定，等待全部恢复完成
6. 右滑打开侧边栏 - 常用功能 - 容器切换 - 点击你刚刚命名的容器 - 切换 - 需要
7. 现在，你应该就进入一个已经安装好 `Debian12` 容器的环境中了

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
root@localhost:~#
```

说明已经处于容器中了，此时输入命令

```bash
whoami
```

可以看到输出的是 `root` 而不是 Termux 的 `u0_axxx`

```bash
root@localhost:~# whoami
root
```

### 常见问题

> Q: 如何退出 `Debian12` 容器到 Termux 环境中？
> 
> A: 输入命令 `exit` 并回车即可

> Q: 平时如何进入 `Debian12` 容器？
> 
> A: 在 Termux 根目录中，输入命令 `bash ./start.sh` 即可


## 4、`apt` 换源

`apt` 是 Debian 系 Linux 发行版的软件包管理器，几乎所有常用软件都可以通过 `apt` 来安装、卸载、更新等等。但是由于国内网络环境问题，默认 `apt` 下载源访问较慢，甚至版本过旧，因此新的系统应该先进行换源。这里推荐[中国清华大学源](https://mirrors.tuna.tsinghua.edu.cn/help/debian/)：

### 步骤：

1. 点开上面的网站，下滑找到 `DEB822 格式` 一栏
2. Debian 版本切换到 `Debian12 (bookworm)`，其他默认即可
3. 复制那一大段代码，类似（下面仅为笔者编辑时的版本，实际可能不一样，请以清华源最新版本为准）:
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
4. 使用 nano 打开 `/etc/apt/sources.list.d/debian.sources`，如果显示未找到 nano，则先通过命令 `apt install -y nano` 安装
	```bash
	nano /etc/apt/sources.list.d/debian.sources
	```
5. 粘贴所有内容，Ctrl + X 保存，再按 y 确认修改并退出
6. 更新源
	```bash
	apt update
	```
7. 安装常用软件<sup>[1]</sup>
    ```bash
    apt install -y curl wget git screen
    ```

## 5、安装 uv

`uv` 是一个由 `rust` 编写的现代化、高性能的 python 包管理器，不仅能方便的创建虚拟环境、管理依赖, uv 还能自动根据项目依赖下载对应 python 版本。本文就没有下载 python 和 pip，而是完全交给 uv。

安装之前最好**开启代理**，否则可能会因为网络问题安装较慢甚至失败:

### 方式一 (有代理推荐): 官方脚本一键安装 

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

安装完成请根据提示手动加载并验证安装：

```bash
# 加载到环境变量
source $HOME/.local/bin/env
# 验证安装
uv --version
uvx --version
```

输出版本号说明安装 uv 成功！

### 方式二（无代理时）：通过 github 镜像站手动安装

```bash
# 通过镜像站点下载
curl -LO https://kkgithub.com/astral-sh/uv/releases/download/0.9.7/uv-aarch64-unknown-linux-gnu.tar.gz

# 验证文件类型
file uv-aarch64-unknown-linux-gnu.tar.gz

# 解压文件
tar -xzf uv-aarch64-unknown-linux-gnu.tar.gz

# 移动可执行文件到系统PATH
mv uv-aarch64-unknown-linux-gnu/uv /usr/local/bin/
mv uv-aarch64-unknown-linux-gnu/uvx /usr/local/bin/
chmod +x /usr/local/bin/uv
chmod +x /usr/local/bin/uvx

# 清理临时文件
rm -rf uv-aarch64-unknown-linux-gnu/
rm uv-aarch64-unknown-linux-gnu.tar.gz

# 验证安装
uv --version
uvx --version
```

输出版本号说明安装 uv 成功！

## 6、安装 Astrbot

安装前最好也开启代理，国内访问 Github 较慢

### 下载/克隆仓库

```bash
# 回到家目录，不熟悉 Linux 的所有命令复制粘贴即可，确保项目结构一致）
cd ~

# 直接从 Github 克隆 Astrbot 仓库
git clone https://github.com/AstrBotDevs/AstrBot.git

# 进入 AstrBot 目录
cd AstrBot
```

如果 git clone 失败，说明你的网络无法直连 Github ,使用下面镜像站重试

```bash
# 清理可能的克隆残留
cd ~ && rm -rf AstrBot

# 使用镜像站
git clone https://kkgithub.com/AstrBotDevs/AstrBot.git

# 进入 AstrBot 目录
cd AstrBot
```

### 使用 uv 安装依赖

在 AstrBot 目录下，理想情况下能安装成功，但是 termux 环境不是真正的 Linux 环境，所以会因为权限问题导致 uv 的硬链接失败。为了稳妥，修改环境变量为复制模式再安装依赖：

```bash
# 确保进入 Astrbot 目录
cd ~/AstrBot
# 修改环境变量为复制模式
export UV_LINK_MODE=copy
# 同步依赖
uv sync
```

uv 会自动安装 Python 并创建虚拟环境管理，等待所有依赖安装完毕即可。

>[!TIP]
>如果使用 `uv` 下载软件包时速度慢，可以更换源后重试 (以 `清华源` 为例)
>```bash
>export UV_DEFAULT_INDEX="https://pypi.tuna.tsinghua.edu.cn/simple"
>export UV_LINK_MODE=copy
>uv sync
>```

(可选但推荐）为了后续不出现 uv 硬链接安装失败报错，可以将其加入系统环境变量中：

```bash
echo "export UV_LINK_MODE=copy" >> ~/.bashrc
source ~/.bashrc
```

>[!TIP]
>如果上面步骤出现安装 `python3.10.x 失败`报错，**优先考虑开启代理重试**。仍然失败请根据下面流程手动安装 `python3.10`。无报错不需要进行，直接跳到`启动 Astrbot` 部分。


####  使用`apt`安装`software-properties-common` (添加PPA前置)


```bash
apt update && apt install software-properties-common
```

#### 添加`deadsnakes`PPA(Python官方维护)

```bash
add-apt-repository ppa:deadsnakes/ppa && apt update
```
添加时你可能会看到:`Press [ENTER] to continue or Ctrl-c to cancel.` ，此时按下回车(换行)即可

#### 安装 `Python`

在进行完以上步骤后，即可安装`Python 3.10`

```bash
apt install python3.10
```

#### 重新安装依赖

```bash
# 确保进入 Astrbot 目录
cd ~/AstrBot
# 修改环境变量为复制模式
export UV_LINK_MODE=copy
# 同步依赖
uv sync
```

### 启动 Astrbot

安装好依赖以后就可以启动 Astrbot 了！

#### 1、前台启动

```bash
# 进入安装目录
cd ~/AstrBot

# 直接启动
uv run main.py
```

不推荐前台启动，仅用于可行性测试，因为前台启动意味着你无法操控终端输入命令，只能看着 Astbot 日志。

推荐使用 `screen` 或者 `tmux` 创建会话，以便让应用分离到后台运行。

[1]:之前有安装过 `screen`，下面以此为例示范如何使用:

#### 2、后台启动（使用 `screen`）

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

确认启动完毕后，使用快捷键同时按住 `Ctrl + a` 再按 `d` 键分离会话即可后台运行！

不要关闭 ZeroTermux ,确保其后台运行，打开手机浏览器，地址栏输入: `http://localhost:6185` 访问，能访问说明 Astrbot 启动成功！

> 默认用户名和密码均为: astrbot

首次登录需要修改用户名、密码并重新登录。

#### screen 常用命令/快捷键

| 命令                      | 用途                            |
| ----------------------- | ----------------------------- |
| `screen -S astrbot`     | 创建并进入一个名为 astrbot 的 screen 会话 |
| `Ctrl+a d`              | 退出当前screen会话(程序继续在后台运行)       |
| `screen -r astrbot`     | 重新连接到 astrbot 会话              |
| `screen -ls`            | 查看所有 screen 会话列表              |
| `exit`                  | 从会话内退出并删除                     |
| `screen -S 会话名 -X quit` | 从外部强制结束会话                     |

#### Linux 常用命令

| 命令           | 用途                    |
| ------------ | --------------------- |
| `cd 路径`      | 进入指定路径的目录，支持相对路径和绝对路径 |
| `cd ~`       | 进入当前用户的家目录            |
| `cd ..`      | 进入上级目录                |
| `ls`         | 列出当前目录文件和子目录          |
| `ls -a`      | 在 ls 基础上列出隐藏文件        |
| `mkdir 目录名`  | 在当前目录下创建目录            |
| `rm 文件路径`    | 删除指定文件, -f 强制删除       |
| `rm -r 目录路径` | 删除指定目录, -f 强制删除       |
| `mv a b`     | 将路径 a 的文件/目录移动到路径 b   |
| `cp -r a b`  | 将路径 a 的文件/目录复制到路径 b   |
| `cat 文件路径`   | 打印文件内容                |
| `clear`      | 清除终端                  |

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

以下两种启动方式任选:

#### 方式一：CLI 命令行

CLI 是指命令行脚本，对于喜欢用命令的用户，推荐 CLI 启动和管理，便于自动化

```bash
# 列出 CLI 帮助命令
napcat help
```

常用命令：

- 启动 Napcat
```
napcat start QQ号
```
- 停止 Napcat
```
napcat stop QQ号
```
- 重启 Napcat
```
napcat restart QQ号
```
- 查看日志
```
napcat log QQ号
```

> Ctrl + c 退出日志、中断脚本

#### 方式二：TUI 图形化界面

TUI 是指终端图形化界面，对于不擅长命令的小白比较友好，使用方向键和回车操作即可

只输入 `napcat` 回车就能启动 TUI 

```bash
napcat
```

使用方向键和回车操作即可。

### 配置 NapCat 

#### 1、扫码登录

启动后，使用 TUI 或者 CLI **打开日志**，通常能看到二维码，通常直接截图日志然后本机扫码，也可以使用另一个设备扫码登录。若二维码过期，重启 NapCat 再获取即可。

**注意**: 请使用 Bot 账号扫码

#### 1.1 如何获取二维码（已扫跳过）
考虑到部分终端二维码很难扫，还有几种获取二维码的方式:

1. （纯小白或懒狗）使用我的二维码解析服务一键生成，复制下面命令到终端，并按要求操作即可：
	```bash
	echo -n "请输入你的QQ号后回车: "; read qq; grep "解码URL" ~/Napcat/log/napcat_${qq}.log | tail -n 1 | awk '{print "请用浏览器打开: https://qrcode.lsgbin.com/api/generate?url=" $2}'
	```
2. 先完成下一步`获取 WebUI 初始 token`，然后在 WebUI 点击扫码登录

3. 将二维码图片复制到手机其他位置，然后用文件管理器(推荐 mt 管理器)打开。这里复制到标准下载目录 `~/手机内部存储/Download/`:
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

完成以上步骤，你应该能成功安装并连接 NapCat 和 Astrbot，接下来请前往官方文档完成其他配置，尤其是 Astrbot 需要配置 LLM 相关以及添加管理员等等，Napcat 通常无需额外配置:

- [AstrBot 文档](https://docs.astrbot.app/)
- [NapCatQQ | 现代化的基于 NTQQ 的 Bot 协议端实现](https://napneko.icu/)

此外，Termux 在一些机型容易被杀后台，请将省电策略调整成**无限制**，并尽可能提高其权限，开启允许自启动等等。具体参考你的机型软件保活教程。

补充: 如果你感兴趣，还可以研究如何写一个一键启动脚本，结合 ZeroTermux 的快捷命令，可以很方便一键启动上述两个应用。




