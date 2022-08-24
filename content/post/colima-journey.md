+++
title = "Colima 初体验"
date = "2022-07-08"
author = "kopever"
+++

### 背景

1. 很长一段时间以来 Docker Desktop 都无法在线更新，点击更新后图标一直转动，但始终无法更新成功；之前在 Docker 官方社区论坛看到有人讨论过这个问题，至今也没有好的解决方案，基本都是手动下载最新镜像重新安装；
    * Docker 官方社区论坛讨论：<a href="https://forums.docker.com/t/docker-desktop-wont-update-on-macos/114648" target="_blank">Docker Desktop won’t update on MacOS</a>
2. Docker Desktop 资源消耗严重，单 com.docker.hyperkit 进程内存占用超 10G，实际容器占用内存很少，CPU 有时也因占用过多而导致风扇狂转、机器发烫，也比较耗电；
3. 可视化界面有点鸡肋，没有太大作用，而且是用内存大户 Electron 写的，天下苦 Electron 久矣。

### 介绍

<a href="https://github.com/abiosoft/colima" target="_blank">Colima</a> 是基于 <a href="https://github.com/lima-vm/lima" target="_blank">Lima</a> 开发的，Lima 基于 <a href="https://github.com/qemu/qemu" target="_blank">QEMU</a>，支持 ARM，M1，Docker，containerd，Kubernetes，端口转发，卷挂载等；  
按官方说法：是在 macOS 或 Linux 上所需最少设置的容器运行时。

### 安装问题

具体安装方法在其 <a href="https://github.com/abiosoft/colima" target="_blank">GitHub</a> 主页，这里只列举安装过程可能会遇到的问题：

1. brew 安装过程中可能会遇到某些依赖包下载不下来，可以多尝试几次，brew 不会全量重新下载，会缓存已下载的包，或者修改 brew 镜像源地址；
    * <a href="https://mirrors.ustc.edu.cn/help/brew.git.html" target="_blank">中科大镜像源</a>
    * <a href="https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/" target="_blank">清华大学开源软件镜像站</a>
    * <a href="https://developer.aliyun.com/mirror/homebrew" target="_blank">阿里云镜像源</a>

### 启动问题

1. 执行 colima start 时会从 GitHub 拉取 <a href="https://github.com/abiosoft/alpine-lima/releases/download/colima-v0.4.2-1/alpine-lima-clm-3.14.6-x86_64.iso" target="_blank">alpine-lima-clm-3.14.6-x86_64.iso</a> 镜像（277.9M），由于国内网络原因始终无法完整的拉取，想着应该可以手动下载并放在指定位置，正准备研究源码时发现了这篇文章 <a href="https://stanzhai.site/blog/post/stanzhai/%E8%A7%A3%E5%86%B3mac%E4%B8%8B%E4%BD%BF%E7%94%A8brew%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85colima%E6%8A%A5%E9%94%99%E7%9A%84%E9%97%AE%E9%A2%98" target="_blank">记 mac 下尝鲜 colima 的坎坷经历</a>，作者跟我遇到了一样的问题，于是参考他的方案解决了此问题，在此表示感谢；  
<a href="https://github.com/lima-vm/lima/blob/44454dd1285a0baba13ec0538f1f1b37b31160d5/pkg/qemu/qemu.go#L77" target="_blank">参考源码地址</a>  
**解决方案步骤：**
    * 手动下载 <a href="https://github.com/abiosoft/alpine-lima/releases/download/colima-v0.4.2-1/alpine-lima-clm-3.14.6-x86_64.iso" target="_blank">alpine-lima-clm-3.14.6-x86_64.iso</a> 镜像；
    * 重命名镜像名字为 basedisk；
    * 将 basedisk 放到 ~/.lima/colima/ 目录下。

### 挂载问题

启动 mysql 容器：

``` shell
docker run -p 3306:3306 --name mysql \
    -v ~/docker/data/mysql:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD='TNZSV4dA**b@ZK58Eez$3hYsfhM@6Kx#' \
    -d mysql:latest
```

出现以下错误：

``` shell
chown: changing ownership of '/var/lib/mysql': Permission denied
```

1. Colima / Lima 支持 sshfs 和 9p 两种挂载方式，默认 sshfs 不支持容器目录挂载，9p 支持虚拟机目录可被操作系统作为文件目录直接访问，具体参见 <a href="https://wiki.qemu.org/Documentation/9psetup" target="_blank">Documentation/9psetup</a>，可在 Colima 启动时指定挂载方式，建议启动前先删除已创建的 Colima 实例；

``` shell
colima stop
colima delete
colima start --mount-type 9p
```

**注意事项：**

* <a href="https://github.com/lima-vm/lima/releases/tag/v0.10.0" target="_blank">Lima v0.10.0</a> 开始支持 9p，并计划在 v1.0 中将 9p 作为默认挂载方式；  
* <a href="https://github.com/abiosoft/colima/releases/tag/v0.4.0" target="_blank">Colima v0.4.0</a> 开始支持 9p 及 --mount-type 参数，并可通过 colima status 查看当前挂载方式；

``` shell
colima status
INFO[0000] colima is running
INFO[0000] arch: x86_64
INFO[0000] runtime: docker
INFO[0000] mountType: 9p
INFO[0000] socket: unix:///Users/username/.colima/default/docker.sock
```

* Lima 配置文件目录：~/.lima/colima/lima.yaml
* Colima 配置文件目录：~/.colima/default/colima.yaml

### 使用问题

1. 使用 docker-compose（需手动安装 brew install docker-compose）时遇到错误：  
error getting credentials - err: exec: "docker-credential-desktop": executable file not found in $PATH  
参考此 GitHub Issue <a href="https://github.com/abiosoft/colima/issues/265" target="_blank">Unable to use docker-compose / docker compose with colima</a>；  
**解决方案：**
    * 将 ~/.docker/config.json 文件中的 credsStore 改为 credStore。
