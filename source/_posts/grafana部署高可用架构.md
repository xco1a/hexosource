---
title: grafana部署高可用架构
date: 2019-01-11 19:10:46
tags:
- 云服务
categories: 
- 云服务
---


为了获得高可用性的服务，需要对单台部署grafana的服务器进行扩充。

![grafana-high-availability](\uploads\grafana-high-availability.png)

这个过程中需要做两件事：
* 配置一个公共数据库，使所有的grafana服务器可以将配置写入公共数据库中而不是存在本地；
* 选用一个会话存储方案，这一步可以通过让负载均衡启用会话保持功能解决

## 数据库配置
grafana为数据库提供多种可选方案，包括：mysql，postgresql，sqlite3(default)。<br />配置文件为：

```shell
vi /etc/grafana/grafana.ini
```
配置项为database部分：

```shell
[database]
# You can configure the database connection by specifying type, host, name, user and password
# as separate properties or as on string using the url properties.

# Either "mysql", "postgres" or "sqlite3", it's your choice
type = mysql                 ;数据库类型
host = www.url.com           ;数据库域名
name = grafana_configure     ;数据库名称
user = admin                 ;数据库用户名
# If the password contains # or ; you have to wrap it with triple quotes. Ex """#password;"""
password = admin_password    ;数据库用户密码

# Use either URL or the previous fields to configure the database
# Example: mysql://user:secret@host:port/database
;url =

# For "postgres" only, either "disable", "require" or "verify-full"
;ssl_mode = disable

"/etc/grafana/grafana.ini" 487L, 15112C written
```

仅需要为grafana开辟一个数据库即可，数据表和字段将在用户新建一个dashboard后自动写入。
## 会话策略
grafana支持将回话保存在磁盘/数据库中，这两种保存方法对应着两种策略：会话保持和无状态会话。
### 会话保持
配合负载均衡，将同一用户的请求转发到相同的服务器上。这样工作量最小，但是也会出现某台服务器负载较其他服务器更高的情况。
### 无状态回话
grafana将用户的会话保存在数据库中，这样同一用户的会话也将会被分发到各个服务器中处理，这种方式需要在数据库中提前预设一张数据表，在grafana.ini中的session部分有关于此项的配置信息。

## 方案步骤
* 新建一个数据库，为grafana新建可操作用户
* 部署N台grafana服务器，修改各服务器的grafana.ini文件，写入数据库相关配置信息，重启grafana-server。
* 将配置好的grafana服务器挂载到负载均衡中，开启会话保持
* 访问负载均衡的IP，获得高可用的grafana服务