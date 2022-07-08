+++
title = "Shadowsocks 服务端教程"
date = "2019-07-12"
author = "kopever"
+++

### 基于 shadowsocks-libev+v2ray 的服务端搭建教程  

> shadowsocks 选择的是基于 C 语言的 <a href="https://github.com/shadowsocks/shadowsocks-libev" target="_blank">shadowsocks-libev</a>  
> <a href="https://github.com/shadowsocks/v2ray-plugin" target="_blank">v2ray-plugin</a> 作为其插件  
> <a href="https://github.com/shadowsocks/shadowsocks-libev/blob/master/README.md" target="_blank">Github 安装文档</a>

<ol>
    <li>参考 GitHub 文档安装 shadowsocks-libev；</li>
    <li>安装 v2ray-plugin 插件；
        <ol>
            <li>根据操作系统下载最新版<a href="https://github.com/shadowsocks/v2ray-plugin/releases" target="_blank">v2ray-plugin</a>插件程序包；</li>
            <li>解压程序包并将解压出的可执行文件 v2ray-plugin_linux_*移动到/usr/local/bin 目录；</li>
        </ol>
    </li>
    <li>配置文件中 plugin 字段填入插件程序名 v2ray-plugin_linux_*；</li>
</ol>

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