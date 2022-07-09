+++
title = "Shadowsocks 服务端教程"
date = "2019-07-12"
author = "kopever"
+++

### 基于 shadowsocks-libev + v2ray 的服务端搭建教程  

shadowsocks 选择的是基于 C 语言的 <a href="https://github.com/shadowsocks/shadowsocks-libev" target="_blank">shadowsocks-libev</a>  
<a href="https://github.com/shadowsocks/v2ray-plugin" target="_blank">v2ray-plugin</a> 作为其插件  

### 安装说明

1. 参考 <a href="https://github.com/shadowsocks/shadowsocks-libev/blob/master/README.md" target="_blank">Github 文档</a> 安装 shadowsocks-libev；
2. 安装 <a href="https://github.com/shadowsocks/v2ray-plugin/releases" target="_blank">v2ray-plugin</a> 插件；  
    + 根据操作系统类型下载最新版 v2ray-plugin 插件程序压缩包；  
    + 解压出可执行文件 v2ray-plugin 并移动到 /usr/local/bin 目录。
3. 配置文件中 plugin 字段填入插件程序名 v2ray-plugin，注意与上一步解压出的可执行文件名称保持一致。

### 安装 BBR 加速

参考秋水逸冰<a href="https://teddysun.com/489.html" target="_blank">一键安装最新内核并开启 BBR 脚本</a>  

### 配置文件示例

#### 单端口配置 config.json

``` json
{
    "server":"0.0.0.0",
    "server_port":8000,
    "local_port":1080,
    "mode":"tcp_and_udp",
    "password":"password",
    "timeout":300,
    "fast_open":true,
    "method":"chacha20-ietf-poly1305",
    "plugin":"v2ray-plugin",
    "plugin_opts":"server"
}
```

#### 多端口配置 manager.json

``` json
{
    "server":"0.0.0.0",
    "local_port":1080,
    "mode":"tcp_and_udp",
    "port_password":{
        "8000":"password0",
        "8001":"password1",
        "8002":"password2"
    },
    "timeout":300,
    "fast_open":true,
    "method":"chacha20-ietf-poly1305",
    "plugin":"v2ray-plugin",
    "plugin_opts":"server"
}
```

### 相关命令

<ol>
    <li><font size=4>nohup ss-server -c /etc/shadowsocks-libev/config.json > ~/shadowsocks.log 2>&1 &</font></li>
    <li><font size=4>nohup ss-manager -c /etc/shadowsocks-libev/manager.json > ~/shadowsocks.log 2>&1 &</font></li>
    <li><font size=4>pkill ss-server / pkill v2ray-plugin / killall ss-server / killall v2ray-plugin</font></li>
</ol>
