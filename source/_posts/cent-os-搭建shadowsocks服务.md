---
title: cent os 搭建shadowsocks服务
date: 2018-05-18 10:20:29
tags: 
- 瞎折腾
categories: 
- 服务器
---

&emsp;&emsp;一直设想买一个国外的vps架个ss，碰巧这两天内网病毒泛滥网络几乎瘫痪，拿手里的腾讯云学生机搭个ss当个跳板。       

## 安装ss          

​	`wget --no-check-certificate https://down.upx8.com/shell/shadowsocks-go.sh `

​	`chmod +x shadowsocks-go.sh `

​	`./shadowsocks-go.sh 2>&1 | tee shadowsocks-go.log `

## 卸载方法

​	`./shadowsocks-go.sh uninstall `

​	