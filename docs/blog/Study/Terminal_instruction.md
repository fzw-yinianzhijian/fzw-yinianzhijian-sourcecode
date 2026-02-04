---
title: 命令行入门
tags:
  - Study
createTime: 2026/02/03 12:18:00
permalink: /blog/Study/Terminal_instruction/
---

这里是命令行入门，讲一些简单的实用工具与内容。

### 基础命令行

对于 Linux 命令行在根目录有 `.bashrc` 文件，可以在这个文件中配置环境变量，别名，函数等，如果当前状况不对，比如刚 ssh 上去之类的，可以先 `source ~/.bashrc` 一下。

对于 Mac 命令行在 `.zshrc` 文件中配置 zsh 的配置，同样 `source ~/.zshrc` 一下。

Windows 命令行很难用，建议用 `wsl` or `mysys2` 但是第二个始终是个模拟。

基础命令合集：

- `ls` 列出当前目录下的文件和目录(`dir` in Windows)
- `cd` 切换目录
- `pwd` 显示当前目录
- `mkdir` 创建目录
- `rm` 删除文件
- `rm -r` 删除目录
- `rm -rf` 删除目录及其内容
- `cp` 复制文件
- `mv` 移动文件
- `cat` 查看文件内容(`type` in Windows)
- `ssh` 远程连接到服务器,可以配置 .ssh/config 来简化链接，
- `scp` 简单发文件到服务器上发送到 Windows 的例子
```bash
scp ~/.vimrc 冯政玮@100.101.21.35:E:\\msys2\\home\\冯政玮
```
- `top`/`htop` 查看系统进程，以及内存使用情况
- `du -sh ./*` 查看当前目录下所有文件和目录的大小

---

网络与端口扫描

- `curl ifconfig.me` 查看本机公网 IP
- `nmap -sT ip` 用 TCP 扫描端口，时间较久，返回示例
```
(base) ➜  fzw-yinianzhijian-sourcecode git:(main) ✗ nmap -sT 140.143.xxx.xx
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-03 16:56 +0800
Nmap scan report for 140.143.xxx.xx
Host is up (0.010s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT     STATE  SERVICE
22/tcp   open   ssh
80/tcp   open   http
443/tcp  closed https
3389/tcp closed ms-wbt-server

Nmap done: 1 IP address (1 host up) scanned in 42.81 seconds
```
- `curl -I ip:port` 查看当前 ip 的链接信息，默认是 80 端口如果不指定，-I 是不下载东西只获取信息


### Vim

模式分为 默认模式/编辑模式/选择模式，按 `i` 和 `v` 切换，`a` 是当前指针靠后一个，`esc` 退出。

:::note 默认模式
默认模式方便较快的移动指针，编辑模式

---
**跳转命令**：
- `gg` 跳转到开头
- `G` 跳转到结尾
- 数字+`enter` 向后跳转几行
- `:`+数字+`enter` 跳转到第几行
- `/`+内容+`enter` 查找下一个内容在哪，可以用 `n` 和 `N` 后，前 查找
- `$` 当前行最后
- `^` 当前行最前
- `:%s/old/new/g` 替换所有 old 为 new，最有用的命令。

---

**剪切命令**：
- `yy` copy当前行
- 数字+`yy` copy当前往下数多少行
- `dd` 删除当前行（自动复制）
- 数字+`dd` 删除当前往下数多少行（自动复制）

---

`ctrl + c` 只能在当前模式进行。
`ctrl + x` 可以在编辑模式进行。
选择模式没探索过

:::

### 环境管理

::: note python venv

目前在我 wsl 里面用过这个。

创建 `python -m venv myenv`（myenv名字可以改）

启用 `source myenv/bin/activate`

禁用 `deactivate`

:::

::: note conda

较为好用，比较推荐。

`conda env list` 查看所有环境

`conda env remove -n myenv` 删除环境

`conda create -n myenv python=3.10` 创建环境（myenv名字可以改，python版本可以改）

`conda activate myenv` 激活环境

`conda deactivate` 禁用环境

:::

### docker 容器

单个机子上跑的虚拟容器，用于隔离和运行程序，类似虚拟机，但比虚拟机轻量级。

硬件是公用的，但是软件系统等资源都是隔离的，不会相互影响，多个容器之间可以。

在往上一层的部署是 kubernetes 集群，之后用到再说，目前都只需要小规模的用 docker 即可。

内容分为容器和镜像，镜像是内容打包 包括环境包括程序内容，可以在 docker hub 上

`docker run -d -p 8080:8080 myimage` 运行一个容器

`docker stop mycontainer` 停止一个容器

`docker rm mycontainer` 删除一个容器

`docker rmi myimage` 删除一个镜像

`docker ps` 查看所有容器

`docker images` 查看所有镜像

`docker exec -it mycontainer /bin/bash` 进入一个容器

`docker cp mycontainer:/app/myfile.txt ./myfile.txt` 从容器中复制文件到本地

`docker cp ./myfile.txt mycontainer:/app/myfile.txt` 从本地复制文件到容器

`docker commit mycontainer myimage` 创建一个镜像

`docker push myimage` 推送一个镜像到仓库

`docker pull myimage` 从仓库拉取一个镜像

`docker build -t myimage .` 从 Dockerfile 构建一个镜像

`docker run -d -p 8080:8080 myimage` 运行一个容器

`docker stop mycontainer` 停止一个容器

`docker rm mycontainer` 删除一个容器

`docker rmi myimage` 删除一个镜像



### 调试技巧

对于网页可以 f12 看当前的控制台和报错信息，看网络连接，返回的请求和参数。

对于当前 ssh，如果是 vscode/cursor 可以看这个输出日志，又可能是 connect 或者什么问题，调一下梯子网络之类的。

