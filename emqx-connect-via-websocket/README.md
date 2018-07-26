使用 WebSocket 客户端连接 MQTT 服务器
---
[TOC]

### 简介

近年来随着 Web 前端的快速发展，浏览器新特性层出不穷，越来越多的应用可以在浏览器端或通过浏览器渲染引擎实现，Web 应用的即时通信方式 WebSocket 得到了广泛的应用。

WebSocke 是一种在单个 TCP 连接上进行全双工通讯的协议。WebSocket 通信协议于2011年被 IETF 定为标准 RFC 6455，并由 RFC 7936 补充规范。WebSocket API 也被 W3C 定为标准。

WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在 WebSocket API 中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。 —— 摘自 维基百科 WebSocket

[MQTT 协议第 6 章](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718127)详细约定了 MQTT 在 WebSocket [RFC6455] 连接上传输需要满足的条件，协议内容笔者不在此累述。由于协议实现细节较为复杂，本文选取两个常用的 JavaScript MQTT 客户端进行连接测试。



### 两款客户端比较

#### Paho.mqtt.js

[Paho](https://www.eclipse.org/paho/) 是 Eclipse 的一个 MQTT 客户端项目，Paho JavaScript Client 是其中一个基于浏览器的库，它使用 WebSockets 连接到 MQTT 服务器。相较于另一个 JavaScript 连接库来说，其功能较少，不推荐使用。

#### MQTT.js

[MQTT.js](https://www.npmjs.com/package/mqtt) 一个 MQTT 协议的客户端库，用 JavaScript 编写，可用于 Node.js 和浏览器。在 Node.js 端可以通过全局安装使用命令行连接，同时还支持 MQTT ，MQTT TLS 证书连接；值得一提的是 MQTT.js 还对微信小程序有较好的支持。

EMQ 君将以 MQTT.js 库进行连接讲解。


### 安装 MQTT.js

如果读者机器上装有 Node.js 运行环境，可使用 npm 命令安装 MQTT.js

#### 在当前目录安装

```bash
npm i mqtt
```

#### 全局安装

将注册 `mqtt` `mqtt_pub` `mqtt_sub` 命令到当前用户，此处借助 `iot.eclipse.org` 讲解一下命令行的使用

```bash
# 全局安装
npm i mqtt -g

# 使用命令行订阅
$ mqtt sub -t 'hello' -h 'iot.eclipse.org' -v
> hello 09860

# 成功连接到服务器并订阅了主题 hello, 命令行将阻塞等待消息


# 在另一个终端上使用命令行发布
mqtt pub -t 'hello' -h 'iot.eclipse.org' -m 'from MQTT.js'

# 命令行将进行 连接 -> 发布 -> 断开连接 操作，此时读者会到订阅命令行，应当收到来自 hello 主题的消息

> hello from MQTT.js
```
> npm 在当前目录安装仍然可以使用 ./node_module/.bin/mqtt 命令来执行以上操作。



#### CDN 引用

MQTT.js 包可以通过 [http://unpkg.com](http://unpkg.com/) 获得

```html
<script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>

<script>
    // 将在全局初始化一个 mqtt 变量
    console.log(mqtt)
</script>
```



### 连接至 MQTT 服务器

几个公共的用于 WebSocket 测试连接服务器：

- test.mosquitto.org - 使用端口 8080 未加密，8081 用于 SSL 上的 WebSocket；
- iot.eclipse.org - 使用端口 80 未加密，443 用于 SSL 上的 WebSocket；
- broker.hivemq.com - 使用端口 8000 未加密，不支持 SSL 上的 WebSocket。

由于需要展示客户端认证部分内容，但上述服务器未提供客户端认证服务，笔者特通过 [ActorCloud](https://www.actorcloud.io/) 平台注册了一个设备进行接入连接。

> EMQ 使用 8083 端口用于普通连接，8084 用于 SSL 上的 WebSocket 连接。

```Js
// <script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>
// const mqtt = require('mqtt')
import mqtt from 'mqtt'

// 连接选项
const options = {
      connectTimeout: 4000, // 超时时间
      // 认证信息
      clientId: 'emqx-connect-via-websocket',
      username: 'emqx-connect-via-websocket',
      password: 'emqx-connect-via-websocket',
}

const client = mqtt.connect('wss://iot.actorcloud.io:8084/mqtt', options)

client.on('reconnect', (error) => {
    console.log('正在重连:', error)
})

client.on('error', (error) => {
    console.log('连接失败:', error)
})

```

#### 连接地址

上文示范的连接地址可以拆分为： `wss:` // `iot` . `actorcloud.io` : `8084` `/mqtt` 

即 `协议` // `主机名` . `域名` : `端口 ` / `路径`

初学者容易出现以下几个错误：

- 连接地址没有指明协议：WebSocket 作为一种通信协议，其使用 `ws`(非加密)、`wss`(SSL 加密) 作为协议标识。MQTT.js 客户端支持多种协议，连接地址需指明协议类型；

- 连接地址没有指明端口：MQTT 并未对 WebSocket 接入端口做出规定，EMQ 上默认使用 `8083` `8084` 分别作为非加密连接、加密连接端口。而 WebSocket 协议默认端口同 HTTP 保持一致 (80/443)，不填写端口则表明使用 WebSocket 的默认端口连接；而使用标准 MQTT 连接时则无需指定端口，如 MQTT.js 在 Node.js 端可以使用 `mqtt://localhost` 连接至标准 MQTT 8083 端口，当连接地址是 `mqtts://localhost` 则连接到 8884 端口；

- 连接地址无路径：MQTT-WebSoket 统一使用 `/path` 作为连接路径，连接时需指明；

- 协议与端口不符：使用了 `wss` 连接却连接到 `8083` 端口；

- 在 HTTPS 下使用非加密的 WebSocket 连接： Google 等机构在推进 HTTPS 的同时也通过浏览器约束进行了安全限定，即 HTTPS 连接下浏览器会自动禁止使用非加密的 `ws` 协议发起连接请求；

- 证书与连接地址不符： 篇幅较长，详见下文 **EMQ 启用 SSL/TLS 加密连接**。

#### 连接选项

上面代码中， `options` 是客户端连接选项，以下是主要参数说明，其余参数详见[https://www.npmjs.com/package/mqtt#connect](https://www.npmjs.com/package/mqtt#connect)。

- keepalive：心跳时间，默认 60秒，设置 0 为禁用；

- clientId： 客户端 ID ，默认通过 `'mqttjs_' + Math.random().toString(16).substr(2, 8)` 随机生成；

- username：连接用户名（如果有）；

- password：连接密码（如果有）；

- clean：true，设置为 false 以在离线时接收 QoS 1 和 2 消息；

- reconnectPeriod：默认 1000 毫秒，两次重新连接之间的间隔，客户端 ID 重复、认证失败等客户端会重新连接；

- connectTimeout：默认 30 * 1000毫秒，收到 CONNACK 之前等待的时间，即连接超时时间。


#### 订阅/取消订阅

连接成功之后才能订阅，且订阅的主题必须符合 MQTT 订阅主题规则；

注意 JavaScript 异步非阻塞特性，只有在 connect 事件后才能确保客户端已成功连接，或通过 `client.connected` 判断是否连接成功：

```js
// 错误示例
client.on('connect', handleConnect)
client.subscribe('hello')
client.publish('hello', 'Hello EMQ')

// 正确示例

client.on('connect', (e) => {
    console.log('成功连接服务器')
    
    // 订阅一个主题
    client.subscribe('hello', { qos: 1 }, (error) => {
        if (!error) {
            cosnole.log('订阅成功')
            client.publish('hello', 'Hello EMQ', { qos: 1, rein: false }, (error) => {
                cosnole.log(error || '发布成功')
            })
        }
    })
    
    // 订阅多个主题
    client.subscribe(['hello', 'one/two/three/#', '#'], { qos: 1 },  onSubscribeSuccess)
    
    // 订阅不同 qos 的不同主题
    client.subscribe(
        [
            { hello: 1 }, 
            { 'one/two/three': 2 }, 
            { '#': 0 }
        ], 
        onSubscribeSuccess,
    )
})

// 取消订阅
client.unubscribe(
    // topic, topic Array, topic Array-Onject
    'hello',
    onUnubscribeSuccess,
)
```


#### 发布/接收消息

发布消息到某主题，发布的主题必须符合 MQTT 发布主题规则，否则将断开连接。发布之前无需订阅该主题，但要确保客户端已成功连接：

```js
// 监听接收消息事件
client.on('message', (topic, message) => {
    console.log('收到来自', topic, '的消息', message.toString())
})

// 发布消息
if (!client.connected) {
    console.log('客户端未连接')
    return
}

client.publish('hello', 'hello EMQ', (error) => {
    console.log(error || '消息发布成功')
})
```



### 微信小程序

MQTT.js 库对微信小程序特殊处理，使用 `wxs` 协议标识符。注意小程序开发规范中要求必须使用加密连接，连接地址应类似为`wxs://iot.actorcloud.io:8084/mqtt`。



### EMQ 启用 SSL/TLS 加密连接 

EMQ 内置自签名证书，默认已经启动了加密的 WebSocket 连接，但大部分浏览器会报证书无效错误如`net::ERR_CERT_COMMON_NAME_INVALID` (Chrome、360 等 webkit 内核浏览器在开发者模式下， Console 选项卡 可以查看大部分连接错误)。

#### 准备工作

这篇文章 [https流程和原理](https://www.jianshu.com/p/b0b6b88fe9fe) 中对证书认证进行了详细的阐述，EMQ 君总结启用 SSL/TLS 证书需要具备的条件是：

- 将域名绑定到 EMQ 服务器公网地址：CA 机构签发的证书签名是针对域名的；

- 申请证书：向 CA 机构申请所用域名的证书，注意选择一个可靠的 CA 机构且证书要区分泛域名与主机名；

- 使用加密连接的时候选择 `wss` 协议，并**使用域名连接**：绑定域名-证书之后，必须使用域名而非 IP 地址进行连接，这样浏览器才会根据域名去校验证书以在通过校验后建立连接。



#### 在 EMQ 上配置

打开 `etc/emqx.conf` 配置文件，修改以下配置

```bash
# wss 监听地址
listener.wss.external = 8084

# 修改密钥文件地址
listener.wss.external.keyfile = etc/certs/cert.key

# 修改证书文件地址
listener.wss.external.certfile = etc/certs/cert.pem

```

重启 EMQ 即可。

> 可以使用你的证书与密钥文件直接替换到 etc/certs/ 下。

#### 在 nginx 上配置反向代理与证书

使用 nginx 来反向代理并加密 WebSocket 可以减轻 EMQ 服务器计算压力，同时实现域名复用，同时通过 nginx 的负载均衡可以分配多个后端服务实体。

```bash

# 建议 WebSocket 也绑定到 443 端口
listen 443, 8084;
server_name example.com;

ssl on;

ssl_certificate /etc/cert.crt;  # 证书路径
ssl_certificate_key /etc/cert.key; # 密钥路径


# upstream 服务器列表
upstream emq_server {
    server 10.10.1.1:8883 weight=1;
    server 10.10.1.2:8883 weight=1;
    server 10.10.1.3:8883 weight=1;
}

# 普通网站应用
location / {
    root www;
    index index.html;
}

# 反向代理到 EMQ 非加密 WebSocket
location / {
    proxy_redirect off;
    # upstream
    proxy_pass http://emq_server;
    
    proxy_set_header Host $host;
    # 反向代理保留客户端地址
    proxy_set_header X-Real_IP $remote_addr;
    proxy_set_header X-Forwarded-For $remote_addr:$remote_port;
    # WebSocket 额外请求头
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection “upgrade”;
}

```
### 其他资源

MQTT.js [官方例子](https://github.com/mqttjs/MQTT.js/tree/master/examples)给出了详细的连接与使用操作实例代码，读者可前往查看；

EMQ Dashboard 中的 WebSocket 工具、ActorCloud [测试工具 -> MQTT 客户端](https://console.actorcloud.io/mqtt_client) (需到 ActorCloud 商城开通)，均使用 MQTT.js 构建，读者可体验参考。

