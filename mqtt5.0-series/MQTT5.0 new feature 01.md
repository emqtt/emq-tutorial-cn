# MQTT 5.0 新特性（一）— 属性与载荷

MQTT 5.0 协议相比 MQTT 3.1.1 增加了很多属性，这些属性分布于报文的可变头部(Variable Header)和有效载荷(Payload)中。

MQTT 5.0 协议中携带有效载荷的报文有 CONNECT 报文，PUBLISH 报文，SUBSCRIBE 报文，SUBACK 报文，UNSUBSCRIBE 报文和 UNSUBACK 报文。

PUBLISH 报文的有效载荷负责存储消息内容，与 MQTT 3.1.1 协议相同。

### CONNECT 报文

CONNECT 报文的可变头部新增的属性有:

| ID       | CONNECT 属性                 | 描述           |
|----------|------------------------------|----------------|
| 17(0x11) | Session Expiry Interval      | 会话过期间隔   |
| 33(0x21) | Receive Maximum              | 接收最大值     |
| 39(0x27) | Maximum Packet Size          | 最大报文大小   |
| 34(0x22) | Topic Alias Maximum          | 主题别名最大值 |
| 25(0x19) | Request Response Information | 请求响应信息   |
| 23(0x17) | Request Problem Information  | 请求问题信息   |
| 38(0x26) | User Property                | 用户属性       |
| 21(0x15) | Authentication Method        | 认证方法       |
| 22(0x16) | Authentication Data          | 认证数据       |

在 CONNECT 报文的 Payload 中，部分字段发生了变化，遗嘱消息（Will Message）变成了遗嘱载荷（Will Payload）。Payload 中新增了遗嘱属性（Will Properties），用于定义遗嘱消息的行为。

新增的遗嘱属性有:

| ID       | 遗嘱属性                 | 描述           |
|----------|--------------------------|----------------|
| 24(0x18) | Will Delay Interval      | 遗嘱延迟间隔   |
| 1(0x01)  | Payload Format Indicator | 属性格式指示器 |
| 2(0x02)  | Message Expiry Interval  | 消息过期间隔   |
| 3(0x03)  | Content Type             | 内容类型       |
| 8(0x08)  | Response Topic           | 响应主题       |
| 9(0x09)  | Correlation Data         | 关联数据       |
| 38(0x26) | User Property            | 用户属性       |


### CONNACK 报文

CONNACK 报文没有 Payload，在可变头部中包含的属性有：

| ID       | CONNACK 属性                       | 描述                 |
|----------|------------------------------------|----------------------|
| 17(0x11) | Session Expiry Interval            | 会话过期间隔         |
| 33(0x21) | Receive Maximum                    | 接收最大值           |
| 36(0x24) | Maximum QoS                        | 最大QoS              |
| 37(0x25) | Retain Available                   | 保留可用             |
| 39(0x27) | Maximum Packet Size                | 最大报文大小         |
| 18(0x12) | Request Problem Information        | 已分配的客户端标识符 |
| 34(0x22) | Topic Alias Maximum                | 主题别名最大值       |
| 31(0x1F) | Reason String                      | 原因字符串           |
| 38(0x26) | User Property                      | 用户属性             |
| 40(0x28) | Wildcard Subscription Available    | 通配符订阅可用       |
| 41(0x29) | Subscription Identifiers Available | 订阅标识符可用       |
| 42(0x2A) | Shared Subscription Available      | 共享订阅可用         |
| 19(0x13) | Server Keep Alive                  | 服务器保活           |
| 26(0x1A) | Response Information               | 响应信息             |
| 28(0x1C) | Server Reference                   | 服务引用             |
| 21(0x15) | Authentication Method              | 认证方法             |
| 22(0x16) | Authentication Data                | 认证数据             |


### PUBLISH 报文

PUBLISH 报文可变头部的属性有

| ID       | PUBLISH 属性             | 描述           |
|----------|--------------------------|----------------|
| 1(0x01)  | Payload Format Indicator | 载荷格式指示器 |
| 2(0x02)  | Session Expiry Interval  | 消息过期间隔   |
| 35(0x23) | Topic Alias              | 主题别名       |
| 8(0x08)  | Response Topic           | 响应主题       |
| 9(0x09)  | Correlation Data         | 关联数据       |
| 38(0x26) | User Property            | 用户属性       |
| 11(0x0B) | Subscription Identifiers | 订阅标识符     |
| 3(0x03)  | Content Type             | 内容类型       |

### PUBACK, PUBREC, PUBREL, PUBCOMP, SUBACK, UNSUBACK 报文

PUBACK, PUBREC, PUBREL, PUBCOMP, SUBACK, UNSUBACK 都具备以下三个属性：

| ID       | PUBACK,PUBREC,PUBREL, PUBCOMP, SUBACK, UNSUBACK 属性 | 描述       |
|----------|------------------------------------------------------|------------|
| 31(0x1F) | Reason String                                        | 原因字符串 |
| 38(0x26) | User Property                                        | 用户属性   |


### SUBSCRIBE 报文

SUBSCRIBE 报文的属性同样存在可变头部中。

| ID       | SUBSCRIBE 属性           | 描述       |
|----------|--------------------------|------------|
| 11(0x0B) | Subscription Identifiers | 订阅标识符 |
| 38(0x26) | User Property            | 用户属性   |


MQTT 5.0 中 SUBSCRIBE 报文中的 Payload 包含了订阅选项(Subscription Options)。

| 　占位   　 | 订阅选项            | 描述         |
|-------------|:-------------------:|:------------:|
| 0~1位       | Maximum QoS         | QoS 最大值   |
| 2位         | No Local            | 非本地       |
| 3位         | Retain As Published | 保留为已发布 |
| 4~5 位      | Retain Handling     | 保留消息处理 |

订阅选项(Subscription Options)的第 0 位和第 1 位表示 QoS 最大值。该字段给出了服务器可以发送给客户端应用消息的最大 QoS 等级。如果 QoS 值为 3，就会触发协议错误。

订阅选项第 2 位表示非本地选项(No Local)。如果值为 1，应用消息就不会发布给订阅发布主题的发布者本身，如果在共享订阅中将该选项设置为 1 的话，就会触发协议错误。

订阅选项的第 3 位表示保留为已发布(Retain As Published)。若该值为 1，服务器须将转发消息的 RETAIN flag 设为与接收到的 PUBLISH 报文的 RETAIN flag 一致。若该值为 0，不管接收到的 PUBLISH 报文中的 RETAIN flag 是何值，服务器都需将转发消息的 RETAIN flag 置为 0。

订阅选项的第 4 第 5 位表示保留处理 (Retain Handling)。该选项是用来控制保留消息 (retained message) 的发送。当保留处理的值为 0 时，服务器须将保留消息转发到与订阅匹配的主题上去。当该值为 1 时，如果订阅已经不存在了，那么服务器需要将保留消息转发给与订阅匹配的主题上，但是如果订阅存在，服务器就无法再转发保留消息。当该值为 2 时，服务器不转发保留消息。

订阅选项的第 6 第 7 位是预留给未来使用的。如果有效载荷的任何一个预留位非零，那么服务器就会将该报文视为格式错误的报文。

### UNSUBSCRIBE 报文

UNSUBSCRIBE 报文仅有两个属性：属性长度和用户属性。

UNSUBSCRIBE 报文的载荷相比 SUBSCRIBE 的载荷要简单很多，它仅仅只是包含主题过滤器的列表，并不包含各种各样的订阅选项。

### DISCONNECT 报文(新增)

DISCONNECT 报文是 MQTT 5.0 新增的报文，它的引入意味着 mqtt broker 拥有了主动断开连接的能力。DISCONNECT 报文所具备的属性有：

| ID       | DISCONNECT 属性         | 描述         |
|----------|-------------------------|--------------|
| 17(0x11) | Session Expiry Interval | 会话过期间隔 |
| 31(0x1F) | Reason String           | 原因字符串   |
| 28(0x1C) | Server Reference        | 服务引用     |
