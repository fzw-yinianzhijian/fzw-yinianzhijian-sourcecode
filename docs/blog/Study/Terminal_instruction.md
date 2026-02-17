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
- `lscpu` 查看 CPU 信息
- `free -h` 查看内存使用情况
- `df -h` 查看磁盘使用情况

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

如何开放端口？

对于 `linux` 可以用 `ufw` 或者 `firewalld` 来开放端口。

对于 `windows` 可以配置防火墙来打开端口。

ufw 和 firewalld 是进行了一些封装的工具，其实本质上都是在 iptable 上改东西。

（需要 root 权限）

`ufw list` 查看当前开放的端口
`ufw allow 80/tcp` 开放 80 端口
`ufw delete allow 80/tcp` 删除 80 端口
`ufw delete 3` 删除第三条规则

注意，云服务器可能存在上层额外的拦截，需要手动配置。

可以用 `ping` 测试延迟，但是 `ping` 不是 TCP 也不是 UDP 协议，而是 ICMP 协议，理解为不一样就可以了。


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


### tmux

这是一个很好用的终端后台工具，会持续保持运行，远程服务器可以一直挂着，不用一直用 `ssh` 保持链接。

tmux 会分为三层，session，window，panel。

第一层是外层可以有多个 tmux session，每个都是独立的，第二层是对于一个确定的 session 看下面会有若干切换的 window，第三层是对于一个确切的 window 会分屏成多个 panel，每个 panel 是一个终端。


进入 `tmux` 之后，可以用 `ctrl-b` + `d` 把当前窗口隐藏掉

创建进程：`tmux new -s $new_name` 创建一个新 session 带名字的
- 如果没有指定名字，就会默认一个名字，每次新建一个就会自动加一个数字（建立 `n+1` 个默认名字就是 `n+1`），此时 `tmux` 和 `tmux new` 等价。
销毁进程：`tmux kill-session -t $name` 销毁一个 session 带名字的（或者进去 `exit`）

`tmux ls` 查看所有 tmux session，同 `tmux list-sessions`
`tmux attach -t $name` 进入特定的 session，不加 -t 就默认最后一个链接的 session。

控制键是 `ctrl-b`，之后加一个键就是对应的操作。
- + `%` 左右分屏（建立 panel）
- +`"` 上下分屏（建立 panel）
- +`c` 新建窗口（建立 window）
- +`n` 下一个窗口（切换 window）
- +`p` 前一个窗口（切换 window）
- +`0-9` 具体窗口（切换 window）
- +`z` 最大化/恢复 panel
- +`x` 关闭当前panel（关闭 panel）
- +`s` 查看缩略界面 session 层面的列表
- +`w` 查看缩略界面 windows 层面的列表（会精细到每个 session 下的 window）
- +`[` 进入复制模式按 `q` 退出（查找）可以用 `ctrl-s` 查找（大概类似于 vim 的操作，enter 了可以 n和N 前后查找）这个模式可以上下滚动
- +`:` 进入命令模式，一个是 rename-session $new_name 重命名当前 session。


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

`docker run -d -p 80:80 --name test_website myimage`
（把容器内的端口右侧 映射到外部的左侧）这里 myimage 是镜像名字，构建 container 运行当前一个镜像。

`docker start mycontainer` 启动一个容器（名字或者id）

`docker stop mycontainer` 停止一个容器

`docker rm mycontainer` 删除一个容器

`docker rmi myimage` 删除一个镜像

`docker ps (-a)` 查看在跑容器（-a 会显示所有容器，包括停止的）

`docker images` 查看所有镜像

`docker exec -it mycontainer /bin/bash` 进入一个容器

`docker build -t myimage .` 从当前目录下 Dockerfile 构建一个镜像。

剩下用到再说。

这里简单讲一下 `docker-compose`，多个容器一起启动的时候可以方便管理，只需要一个 `docker-compose.yml` 文件。

为什么需要多个容器？需要协作，例如前后端的一个前端一个后端需要通信的。

你需要配置一个 `docker-compose.yml` 文件，里面配置了多个容器，以及它们之间的依赖关系。

`docker-compose up (-d) (--build)` 启动所有容器 (-d 是后台运行的意思，不然就会卡在这里，--build 是构建镜像的意思，不然就会使用缓存的镜像)

`docker-compose down` 停止所有容器（这个会删除历史数据的，因为容器会直接被删掉）

`docker-compose restart` 重启所有容器

`docker-compose logs` 查看所有容器日志

对于数据可持久化，需要在 yml 里面写一个 volume 挂载到服务器本地。
【回头研究】

formyself，去看看 `deploy` 文件夹下面的例子就大概知道了。

可以多问 `AI`，这里只是一个 `AI` 询问的总结。

### 调试技巧

对于网页可以 f12 看当前的控制台和报错信息，看网络连接，返回的请求和参数。

对于当前 ssh，如果是 vscode/cursor 可以看这个输出日志，又可能是 connect 或者什么问题，调一下梯子网络之类的。

注意，本地浏览器是有缓存的，就如果服务器这边断了你可能还是能访问上网页

### ssh 内容

首先是 config 配置，一般是 ssh $user@$ip 这样连接，可以配置验证文件，默认是 `id_rsa`
```
Host $name
    HostName $ip
    User $user
    Port $port
    IdentityFile ~/.ssh/id_rsa
```

这里 $name 是别名，$ip 是 ip，$user 是用户名，$port 是端口，IdentityFile 是私钥文件。

下一次可以直接 `ssh $name` 连接。

ssh 端口转发（感谢 bear 解答疑惑）：

`ssh -L 8080:localhost:7777 user@server` 将本地的 8080 端口转发到服务器的 localhost:7777 端口，意思是本地访问 8080 端口就会转发到服务器的 localhost:7777 端口。
`ssh -L [本地监听端口]:[目标主机]:[目标端口] [SSH服务器用户名]@[SSH服务器地址]`
这里意思是走本地一个端口 实际上 让server访问目标主机的目标端口。

`ssh -R [远程监听端口]:[目标主机]:[目标端口] [SSH服务器用户名]@[SSH服务器地址]`
这里意思是走服务器一个端口 实际上让本机访问目标主机的目标端口。

`ssh -D [本地监听端口] [SSH服务器用户名]@[SSH服务器地址]`
这里意思建立 socket 代理，本地访问 socket 代理端口就会转发到服务器的 ssh 连接（把浏览器设置为这个端口代理，所有流量都从对面主机过）。
（注意，不能直接做 vpn 会被检测到）

宝塔面板：
`wget -O install_panel.sh https://download.bt.cn/install/install_panel.sh && sudo bash install_panel.sh ed8484bec`

去官网上看，安装了会给你一个账号用户，帮你看系统信息，可以方便的管理，如果上不去可以转发端口。


