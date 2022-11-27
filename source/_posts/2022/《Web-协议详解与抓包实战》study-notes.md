---
title: 《Web 协议详解与抓包实战》study notes
author: ratears
date: 2022-06-14 02:00:24
updated: 2022-06-14 02:00:24
categories:
  - network
tags:
	- wireshark
	- network
	- study-notes
---



# HTTP/1.1协议

## 浏览器发起 HTTP 请求的典型场景

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.4fdm1gdg9660.webp" width="70%">

<br>



<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.5ln9wca3rqw0.webp" width="70%">

<br>

## Hypertext Transfer Protocol (HTTP) 协议

- a stateless application-level request/response protocol that uses
  extensible semantics and self-descriptive message payloads for flexible
  interaction with network-based hypertext information systems
  （RFC7230 2014.6）
- 一种无状态的、应用层的、以请求/应答方式运行的协议，它使用可扩展的
  语义和自描述消息格式，与基于网络的超文本信息系统灵活的互动

<br>

## HTTP 协议格式

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.37se393k1h20.webp" width="70%">

<br>

## ABNF （扩充巴科斯-瑙尔范式）

### 操作符

- 空白字符：用来分隔定义中的各个元素
  - method SP request-target SP HTTP-version CRLF
- 选择 /：表示多个规则都是可供选择的规则
  - start-line = request-line / status-line
- 值范围 %c##-## ：
  - OCTAL = “0” / “1” / “2” / “3” / “4” / “5” / “6” / “7” 与 OCTAL = %x30-37 等价
- 序列组合 ()：将规则组合起来，视为单个元素
- 不定量重复 m*n：
  - 元素表示零个或更多元素： *( header-field CRLF )
  - 1* 元素表示一个或更多元素，2*4 元素表示两个至四个元素
- 可选序列 []：
  - [ message-body ]

<br>

### 核心规则

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.4p8vbzjyq9a0.webp" width="80%">

<br>

## 基于 ABNF 描述的 HTTP 协议格式

HTTP-message = start-line *( header-field CRLF ) CRLF [ message-body ]

- start-line = request-line / status-line
  - request-line = method SP request-target SP HTTP-version CRLF
  - status-line = HTTP-version SP status-code SP reason-phrase CRLF
- header-field = field-name ":" OWS field-value OWS
  - OWS = *( SP / HTAB )
  - field-name = token
  - field-value = *( field-content / obs-fold )
- message-body = *OCTET

<br>

## OSI（Open System Interconnection Reference Model）概念模型

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.4zpowyuqy1o0.webp" width="70%">

<br>

## OSI 模型与 TCP/IP 模型对照

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.7bh90wjw4980.webp" width="70%">



## Wireshark 抓包及分析工具



## 报文头部

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.6w78b2tep5k0.webp" width="70%">

## Roy Thomas Fielding 与 HTTP/1.1

- 参与制订 HTTP/1.0 规范（1996.5）
- 参与制订 URI 规范（1998.8）
- 主导制订 HTTP/1.1 规范（1999.6）
- 2000 年发布指导 HTTP/1.1 规范制订的论文
  - 《Architectural Style and the Design of Network-based Software
    Architectures》，即我们常谈的Representational State Transfer (REST)架构

- Apache基金会（The Apache Software Foundation）共同创始人
  - 参与开发Apache httpd服务

## Form Follows Function：HTTP 协议为什么是现在这个样子

- HTTP 协议

  - Roy Thomas Fielding：HTTP 主要作者，REST 架构作者

- URI：统一资源标识符

  

  <img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.5k1vc39i85k0.webp" width="70%">

  

## HTTP 解决了什么问题？

#### 解决 WWW 信息交互必须面对的需求：

• 低门槛
• 可扩展性：巨大的用户群体，超长的寿命
• 分布式系统下的 Hypermedia：大粒度数据的网络传输
• Internet 规模
• 无法控制的 scalability
• 不可预测的负载、非法格式的数据、恶意消息
• 客户端不能保持所有服务器信息，服务器不能保持多个请求间的状态信息
• 独立的组件部署：新老组件并存
• 向前兼容：自 1993 年起 HTTP0.9\1.0（1996）已经被广泛使用

## 评估 Web 架构的关键属性

### HTTP 协议应当在以下属性中取得可接受的均衡：

1. 性能 Performance：影响高可用的关键因素
2. 可伸缩性 Scalability：支持部署可以互相交互的大量组件
3. 简单性 Simplicity：易理解、易实现、易验证
4. 可见性 Visiable：对两个组件间的交互进行监视或者仲裁的能力。如缓存、分层设计等
5. 可移植性 Portability：在不同的环境下运行的能力
6. 可靠性 Reliability：出现部分故障时，对整体影响的程度
7. 可修改性 Modifiability：对系统作出修改的难易程度，由可进化性、可定制性、可扩展
   性、可配置性、可重用性构成

<br>

## 架构属性：性能

- 网络性能 Network Performance
  - Throughput 吞吐量：小于等于带宽 bandwidth
  - Overhead 开销：首次开销，每次开销
- 用户感知到的性能 User-perceived Performance
  - Latency 延迟：发起请求到接收到响应的时间
  - Completion 完成时间：完成一个应用动作所花费的时间
- 网络效率 Network Efficiency
  - 重用缓存、减少交互次数、数据传输距离更近、COD

<br>

## 架构属性：可修改性

• 可进化性 Evolvability：一个组件独立升级而不影响其他组件
• 可扩展性 Extensibility ：向系统添加功能，而不会影响到系统的其他部分
• 可定制性 Customizability ：临时性、定制性地更改某一要素来提供服务，
不对常规客户产生影响
• 可配置性 Configurability ：应用部署后可通过修改配置提供新的功能
• 可重用性 Reusabilit ：组件可以不做修改在其他应用在使用

<br>

## 



## 5 种架构风格

- 数据流风格 Data-flow Styles
  - 优点：简单性、可进化性、可扩展性、可配置性、可重用性
- 复制风格 Replication Styles
  - 优点：用户可察觉的性能、可伸缩性，网络效率、可靠性也可以提到提升
- 分层风格 Hierarchical Styles
  - 优点：简单性、可进化性、可伸缩性
- 移动代码风格 Mobile Code Styles
  - 优点：可移植性、可扩展性、网络效率
- 点对点风格 Peer-to-Peer Styles
  - 优点：可进化性、可重用性、可扩展性、可配置性

<br>

## Chrome 抓包：快速定位 HTTP 协议问题

[chrome-devtools](https://developers.google.com/web/tools/chrome-devtools/network/)

### Chrome 抓包：Network 面板

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.ae6gkxyy8ws.webp" width="90%">

- 控制器：控制面板的外观与功能
- 过滤器：过滤请求列表中显示的资源
  - 按住 Command （Mac）或 Ctrl （Window / Linux），然后点击过滤器可以
    同时选择多个过滤器
- 概览：显示 HTTP 请求、响应的时间轴
- 请求列表：默认时间排序，可选择显示列
- 概要：请求总数、总数据量、总花费时间等

### 控制器

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.4sak4sx7bko0.webp" width="60%">

### 过滤器：按类型

- XHR、JS、CSS、Img、Media、Font、Doc、WS (WebSocket)、Manifest 或 Other
  （此处未列出的任何其他类型）
- 多类型，按住 Command (Mac) 或 Ctrl（Windows、Linux）
- 按时间过滤：概览面板，拖动滚动条
- 隐藏 Data URLs：CSS 图片等小文件以 BASE64 格式嵌入 HTML 中，以减少 HTTP
  请求数

### 过滤器：属性过滤

- domain：仅显示来自指定域的资源。 您可以使用通配符字符 (*) 纳入多个域
- has-response-header：显示包含指定 HTTP 响应标头的资源
- is：使用 is:running 可以查找 WebSocket 资源，is:from-cache 可查找缓存读出的资源
- larger-than： 显示大于指定大小的资源（以字节为单位）。 将值设为 1000 等同于设置为
  1k
- method：显示通过指定 HTTP 方法类型检索的资源
- mime-type：显示指定 MIME 类型的资源

- mixed-content：显示所有混合内容资源 (mixed-content:all)，或者仅显示当前显示的资源
  (mixed-content:displayed)。
- scheme：显示通过未保护 HTTP (scheme:http) 或受保护 HTTPS (scheme:https) 检索的资
  源。
- set-cookie-domain：显示具有 Set-Cookie 标头并且 Domain 属性与指定值匹配的资源。
- set-cookie-name：显示具有 Set-Cookie 标头并且名称与指定值匹配的资源。
- set-cookie-value：显示具有 Set-Cookie 标头并且值与指定值匹配的资源。
- status-code：仅显示 HTTP 状态代码与指定代码匹配的资源。

**多属性间通过空格实现 AND 操作**

### 请求列表的排序

- 时间排序，默认
- 按列排序
- 按活动时间排序
  - Start Time：发出的第一个请求位于顶部
  - Response Time：开始下载的第一个请求位于顶部
  - End Time：完成的第一个请求位于顶部
  - Total Duration：连接设置时间和请求/响应时间最短的请求位于顶部
  - Latency：等待最短响应时间的请求位于顶部

### 请求列表

- Name : 资源的名称
- Status : HTTP 状态代码
- Type : 请求的资源的 MIME 类型
- Initiator : 发起请求的对象或进程。它可能有以下几种值：
  - Parser （解析器） : Chrome的 HTML 解析器发起了请求
    - 鼠标悬停显示 JS 脚本
  - Redirect （重定向） : HTTP 重定向启动了请求
  - Script （脚本） : 脚本启动了请求
  - Other （其他） : 一些其他进程或动作发起请求，例如用户点击链接跳转到
    页面或在地址栏中输入网址
- Size : 服务器返回的响应大小（包括头部和包体），可显示解压后大小
- Time : 总持续时间，从请求的开始到接收响应中的最后一个字节
- Waterfall：各请求相关活动的直观分析图



<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.27b9wyvwjz9c.webp" width="70%">



### 预览请求内容

- 查看头部
- 查看 cookie
- 预览响应正文：查看图像用
- 查看响应正文
- 时间详细分布
- 导出数据为 HAR 格式
- 查看未压缩的资源大小：Use Large Request Rows



- 浏览器加载时间（概览、概要、请求列表）
  - DOMContentLoaded 事件的颜色设置为蓝色，而 load 事件设置为红色
- 将请求数据复制到剪贴版
  - Copy Link Address: 将请求的网址复制到剪贴板
  - Copy Response: 将响应包体复制到剪贴板
  - Copy as cURL: 以 cURL 命令形式复制请求
  - Copy All as cURL: 以一系列 cURL 命令形式复制所有请求
  - Copy All as HAR: 以 HAR 数据形式复制所有请求

- 查看请求上下游：按住 shift 键悬停请求上，绿色是上游，红色是下游

### 浏览器加载时间

- 触发流程：
  - 解析 HTML 结构
  - 加载外部脚本和样式表文件
  - 解析并执行脚本代码 // 部分脚本会阻塞页面的加载
  - DOM 树构建完成 // DOMContentLoaded 事件
  - 加载图片等外部文件
  - 页面加载完毕 // load 事件

### 请求时间详细分布

- Queueing: 浏览器在以下情况下对请求排队
  - 存在更高优先级的请求
  - 此源已打开六个 TCP 连接，达到限值，仅适用于 HTTP/1.0 和 HTTP/1.1
  - 浏览器正在短暂分配磁盘缓存中的空间
- Stalled: 请求可能会因 Queueing 中描述的任何原因而停止
- DNS Lookup: 浏览器正在解析请求的 IP 地址
- Proxy Negotiation: 浏览器正在与代理服务器协商请求
- Request sent: 正在发送请求
- ServiceWorker Preparation: 浏览器正在启动 Service Worker
- Request to ServiceWorker: 正在将请求发送到 Service Worker
- Waiting (TTFB): 浏览器正在等待响应的第一个字节。 TTFB 表示 Time To First Byte
  （至第一字节的时间）。 此时间包括 1 次往返延迟时间及服务器准备响应所用的时
  间
- Content Download: 浏览器正在接收响应
- Receiving Push: 浏览器正在通过 HTTP/2 服务器推送接收此响应的数据
- Reading Push: 浏览器正在读取之前收到的本地数据

## URI的基本格式以及与URL的区别









<br>

<br>

<br>

# 学习备注

> 1. nginx -s 发送信号，做日志切割这个需要理解，需要了解课程中的日志切割shell
> 2. 如何让配置文件语法高亮

```html
&emsp;&emsp;
```

<br>

<br>

<br>

<img src="" width="60%">

<br>

