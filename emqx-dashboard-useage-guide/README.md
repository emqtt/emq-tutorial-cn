EMQ 管理控制台功能简介
---
[TOC]
### 简介

EMQ 管理控制台 (EMQ Dashboard，以下简称 Dashboard) 是 EMQ 提供的一个后端 Web 控制台，用户可通过 Web 控制台查看服务器与集群的运行状态、统计指标，进行插件配置与停启、简单的连接测试等操作。

> 关于 EMQ 的搭建与基本使用详见文章 [常见MQTT服务器搭建与试用](https://www.jianshu.com/p/e5cf0c1fd55c) ，EMQ 君不在此累述。



### 基本使用

如果 EMQ 安装在本机，则使用浏览器打开地址 [http://127.0.0.1:18083](http://127.0.0.1:18083) ，输入默认用户名 `admin` 与默认密码 `public` ，登录进入 Dashboard。如果忘记了管理控制台密码，使用 [管理命令](http://emqtt.com/docs/v2/commands.html#admins) 重置或新建管理账号。




### 主题与语言切换

Dashboard 界面与展示上提供**暗色** (默认)、**明亮**两种主题风格，**中文**、**英文**(默认)两种语言支持。用户可在 **ADMIN (系统)** -> **Settings (设置)** 中进行切换设置。

![settings](_images/setting.png)

### 运行数据监控

Dashboard 提供 EMQ 单机与集群的运行状态监控功能，监控指标涵盖服务器基本信息，设备连接信息，会话信息，EMQ 当前主体与订阅信息。

#### 控制台

控制台可查看 EMQ 当前节点及服务器集群的基本信息如服务器版本、运行时间、CPU、内存、进程、运行统计等数据。

系统信息、度量指标展示的是当前节点数据，用户可以通过界面右上角下拉切换至集群内其他节点；

节点信息、运行统计展示集群内的所有节点列表的信息，标题括号内的数字即代表当前集群内节点的数量。

> 控制台展示的数据每隔 10 秒刷新一次。

![状态概览](./_images/overview.png)


#### 连接

连接界面可查看当前客户端的连接情况，通过右上角下拉切换按钮可以切换查看某节点内、集群内的连接信息；搜索框可按照客户端 ID (clientid) 进行搜索。

![连接状态](./_images/connect.png)


#### 会话

会话界面可查看客户端会话信息如会话数、订阅数等，其右上角切换、搜索功能同上。


#### 主题

主题界面可查看集群内所有主题信息，右上角可进行主题搜索。


#### 订阅

订阅界面可查看单节点/集群内主题订阅信息，右上角切换、搜索功能同连接与会话界面。



### 插件与监听

#### 插件

插件界面可查看当前节点插件运行状况，点击 **启动/停止** 按钮可以进行插件的停启，点击 **配置** 按钮可以查看并配置插件参数。 关于插件更详细的介绍请看 [扩展插件](http://emqtt.com/docs/v2/plugins.html)。

![插件列表](./_images/plugin.png)

出于安全性考虑，通过 Dashboard 配置的插件参数不会持久化到配置文件，即每次重启 EMQ 后配置信息会丢失。用户通过界面上配置的插件参数，在确认正确可用后应当将配置写到 `etc/plugins/` 目录下响应的配置文件中。 

![插件配置](./_images/plugin_details.png)



#### 监听器

监听器界面可查看节点下网络监听状况，包含有每个服务的监听协议、地址与端口及其最大连接数与当前连接数。

![监听器](_images/listeners.png)



### 工具与应用

#### WebSocket 

该工具通过 WebSocket 与 EMQ 连接，提供客户端连接、发布/订阅、消息查看功能。WebSocket 支持非加密连接 (默认 8083 端口) 与 SSL 加密连接 (默认 8084 端口)，但请注意使用加密连接时必须配置了 WebSocket 证书且主机地址填写的是与证书对应的域名。

> 同一设备(clientid) 只能有一个在线，请勿使用已连接的客户端 clientid 进行连接测试。

![websocket](_images/websocket.png)


#### HTTP 接口

HTTP 接口列举了 Dashboard 所有 API 接口，点击**路径**中的 URL 可以以当前登录用户调用该接口并显示数据，部分 POST/PUT/DELETE 方法接口不支持该操作。


![http](_images/http.png)


#### 应用

通过应用可以创建一个 API 接口凭证，用于调用 [管理监控 API](http://emqtt.com/docs/v2/rest.html) 监控服务器、管理客户端、发布订阅消息等。

应用可以分配到期日期实现过期失效，如需暂时禁用应用，可以将其状态置为 **拒绝访问**。

![application](_images/application.png)

#### 用户

管理 Dashboard 的登录用户，支持新建、编辑、修改密码等。


### 进阶配置

#### Dashboard 绑定域名

使用单独的域名或将 Dashboard 绑定到现有域名的某个路径如 `http://example.com/dashboard` 下，参见文章： [使用 nginx 部署 EMQ Dashboard](https://github.com/emqx/emqx-tutorial-cn/blob/master/nginx-emq-dashboard.md)。

