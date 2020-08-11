---
title: hostus搭建多用户shadowsocks
date: 2018-05-24 10:10:49
tags: 
- 瞎折腾
categories: 
- 服务器
---
<!-- more -->
&emsp;&emsp;emmm。。花了60多买了一年多的ss到期了。再续费一年要200，免费的翻墙软件用起来都不太称心如意。遂决心自己搭建一个。。没抢到搬瓦工的VPS于是一顿操作上了hostus的船，价格可以配置一般主要便宜而且不用paypal但是刷一次IP要5刀。

- CPU核心：1核心
- 内存：768MB /768MB vSwap
- 硬盘：20GB
- 端口：1Gbps
- 月流量：2000GB
- 架构/系统：OpenVZ
- 独立IP：1IP
- 价格：[$16.00/年]

&emsp;&emsp;hostus的初始密码比较复杂，putty连上以后先改密码。。

`passwd root`

&emsp;&emsp;安装shadowsocks，这里先安装个go语言的shadowsocks，后来才发现只能找到python语言的流量管理，流量管理的事emmm以后再说吧。。

&emsp;&emsp;wget获取shadowsocks-go脚本。

`wget --no-check-certificate https://down.upx8.com/shell/shadowsocks-go.sh `

&emsp;&emsp;赋予脚本执行权限。

`chmod +x shadowsocks-go.sh `

&emsp;&emsp;执行脚本。

`./shadowsocks-go.sh 2>&1 | tee shadowsocks-go.log `

&emsp;&emsp;这里记录一下卸载脚本的方法。。

`./shadowsocks-go.sh uninstall `

&emsp;&emsp;配置多用户登录。

`vi /etc/shadowsocks/config.json `

&emsp;&emsp;配置文件里添加端口和对应的登录口令。

```json
{
"server":"0.0.0.0",
"local_address":"127.0.0.1",
"local_port":1080,
"port_password":{
"1212":"password1",
"1213":"password2",
"1214":"password3"
},
"timeout":300,
"method":"aes-256-cfb",
"fast_open": false
}
```

&emsp;&emsp;至此，配置就算ok了。记录下shadowsocks服务的各种操作。

```shell
启动：/etc/init.d/shadowsocks start 
停止：/etc/init.d/shadowsocks stop 
重启：/etc/init.d/shadowsocks restart 
状态：/etc/init.d/shadowsocks status
```

&emsp;&emsp;配置完后需要重启shadowsocks。不同用户通过不同端口以及对应的密码即可连接。

&emsp;&emsp;看了一眼hostus的默认服务器系统是centos6。。防火墙还是iptables。添加防火墙过滤规则，否则可能会出现端口被屏蔽服务器拒绝连接的情况。

`iptables -I INPUT -p tcp --dport 1314 -j ACCEPT `

&emsp;&emsp;测试了一下三个端口，都可以正常访问google。但是不完善的一点是go脚本好像没有ss流量管理，有空重装python试试。