---
title: "八股文 -- 计算机网络"
date: 2025-01-19T21:38:54+08:00
draft: false
tags: ["javascript", "http", "notes"]
summary: "记录 整理遇到的八股文知识点之 计算机网络篇"
series: ["八股文 查缺补漏"]
# series_order: 2
---

## url uri urn

URI 语法：scheme:[//[user:password@]host[:port]][/]path[?query][#fragment]
Scheme：URI 的起始点，并与 URI 协议相关。URI schema 对大小写不敏感，后面带有一个“:”。几个流行的 URI schema 的例子：HTTP、HTTPS、FTP 和 mailto。
Authority（权限）：Authority 字段是位于 schema 字段之后的第二个条目，以两个斜杠(//)开头。这个字段由多个子字段组成：
• authentication（认证信息）-可选的字段和用户名，密码，由冒号隔开，后面跟着“@”符号
• host（主机名）—注册的名称或 IP 地址，出现在“@”符号之后
• port（端口号）-可选字段，后面跟着一个冒号
Path（路径）：Path 是第三个字段，由斜杠分隔的段序列来表示，用来提供资源的位置。注意，不管 authority 部分存在或不存在，path 都应该以一个斜杠开始，而不是双斜杠(//)。
Query（查询）：Query 是第四个字段，是 URI 的可选字段，包含一串非结构数据并以“?”和 Path 隔开。
Fragment（片段）：Fragment 是第五个组成部分，也是一个可选字段，提供指向辅助资源的方向，并以“#”开始。简单来说，Fragment 字段可以用于指向 HTML 页面的特定元素(主资源)。

url（统一资源定位符），用于标示网络资源的位置。URL 是一个给定唯一 Web 资源的地址，表明了这个唯一的 Web 资源的位置，用户可以通过 URL 浏览互联网。如果我们在任何应用程序中点击任何超链接，它会将我们重定向到相关的 URL，这些 URL 也可以很容易的输入到浏览器地址栏中，并可以加载特定的资源。

URN 是一种具有静态名称的互联网资源，即使它的数据被移动到另一个位置也仍然有效。URL 在内容被移动后就失效了，与之不同的是，URN 可以始终跟踪 Web 上某些数据的资源，从而解决了频繁移动数据的问题。
URN 语法：scheme:NID: NSS

## http https

### http 1.0

无状态、无连接的应用层协议，一次只能发出一个未完成的请求

### http1.1

-- 基于 connection：keep-alive 建立持久链接，
-- 基于管道可以一次发出多个请求，但服务端依旧需要按顺序响应。
-- 新增 身份认证、缓存等机制相关的请求头。
-- HTTP1.1 的请求消息和响应消息都应支持 Host 头域

### http2.0

-- 采用二进制消息帧，建立请求优先级
帧(frame)包含部分：类型 Type, 长度 Length, 标记 Flags, 流标识 Stream 和 frame payload 有效载荷
消息(message)：一个完整的请求或者响应，比如请求、响应等，由一个或多个 Frame 组成。

-- 多路复用
请求响应都采用流式数据在同一个 tcp 通道内传输，通过 流标识 进行同一个 请求响应 的匹配。
-- 头部压缩 http2.0 头部也采用二进制数据格式，利用 HPACK 压缩算法（维护一张 header 字段表，利用索引代替实际 header）
-- 服务器推送
服务器可以对一个客户端请求发送多个响应，服务器向客户端推送资源无需客户端明确地请求

### http3.0

-- 采用 QUIC 协议（基于 UDP） 解决了 TCP 协议 头包丢失导致的阻塞问题

### http 缓存

强缓存：浏览器会检测缓存时间是否过期，若未过期则不会向服务器发请求转而直接从缓存中读取。
在 http1.1 后改由请求头 cache-control 中的 max-age 控制。
命中强缓存 状态码返回 200 size 为 form disk cache / memory cache。

协商缓存：浏览器优先检测是否命中强缓存，若未命中强缓存则浏览器携带协商缓存标识向服务器发起请求，由服务器根据缓存标识确定是否使用缓存，若生效则返回状态码 304，否则正常返回 200 和请求资源。
缓存是否失效 由 Last-Modified / If-Modified-Since 和 ETag 决定。
浏览器首次请求资源时服务器会在响应头中给到 Last-Modified ，浏览器再次请求该资源时 会在请求头中添加 If-Modified-Since 由服务器判断缓存资源是否失效。
由于 Last-Modified 只能精确到秒级，可能会导致已失效缓存被误判，且如果本地打开缓存文件也会更新 Last-Modified 。因此 Etag 是服务器响应请求时，返回当前资源文件的一个唯一标识(由服务器生成)，只要资源有变化，Etag 就会重新生成。

## ajax fetch

### ajax

```ts
const xhr = new XMLHttpRequest(); // 创建 xhr 操作对象 readyState = 0
xhr.open("get", "http://jsonplaceholder.typicode.com/posts/2"); // 建立请求 readyState = 1
xhr.setRequestHeader("name", "123");
xhr.send(/\*_ 请求体可以放这里 _/); // 发送请求 readyState = 2
xhr.onreadystatechange = () => {
  // readyState = 3 说明正在接收服务器传来的 body 部分的数据
  if (xhr.readyState === 4 && xhr.status === 200) {
    // readyState = 4 说明数据完全接收
    console.log(xhr.response);
  }
};
xhr.onprogress = (p) => {
  if (p.lengthComputable) {
    // 表示底层流程将需要完成的总工作量和已经完成的工作量是否可以计算
    console.log(p.total); // 表示正在执行的底层流程的工作总量
    console.log(p.loaded); // 表示底层流程已经执行的工作总量
  }
};
```

### fetch

-- 请求实例

```ts
fetch("http://jsonplaceholder.typicode.com/posts", {
  //请求方法
  method: "POST",
  //请求头
  headers: {
    name: "zhang"
  },
  //请求体
  body: JSON.stringify({
    id: 1
  })
})
  .then((response) => {
    return response.json();
  })
  .then((response) => {
    console.log(response);
  });
```

-- 中断请求：

```ts
const controller = new AbortController();
const { signal } = controller;
fetch("http://", { signal });
controller.abort();
```

### 区别

-- fetch 使用 promise、 XMLHttpRequest 则使用回调的方式(需要开发手动包装 promise)
-- fetch 只有在中断请求或网络故障或配置错误时才会被标记 reject，其他时候都是 resolve；XHR 则需要在 onReadyStateChange 回调里自行判断
-- XHR 拥有 onProgress 回调监控传输进度
-- fetch 将请求分成了多个模型（Request、Responce、Headers）; XMLHttpRequest 只有一个自己的实例。
-- fetch 通过数据流处理数据，可以分块读取；XMLHTTPRequest 对象不支持数据流，所有的数据必须放在缓存里，不支持分块读取，必须等待全部拿到后，再一次性吐出来

## 跨域

什么是跨域： 请求 url 的协议 域名 端口 中任意一个与当前页面 url 不同即为跨域
原因： 浏览器的同源策略规定非同源请求不能被浏览器所接受以防止 xss csrf 攻击

解决方案：

### 跨域资源共享（CORS）

-- 简单请求 浏览器会自动带上 Origin 请求头 服务器端验证 Origin 在白名单内则返回时添加 Access-Control-Allow-Origin 响应头告诉浏览器 允许该 Origin 访问
简单请求判断标准：
a. 请求方法为 GET、POST、HEAD
b. 请求头只允许出现 CORS 安全规范定义的标头集合
c. 请求中没有使用 ReadableStream 对象

-- 非简单请求则需要在正式发起请求前进行 OPTIONS 预请求，该请求携带 Origin、Access-Control-Request-Method、Access-Control-Request-Headers 等请求头以便服务器验证是否允许实际请求。

### 代理服务器

### JSONP & window.name

## Web 安全

1. XSS（跨站脚本攻击）
   原理： 向目标网站注入脚本以获取用户信息等。
   分类： 存储型（向服务器发送攻击脚本，存储在服务器中，用户正常请求时受到攻击），反射型（恶意链接），基于 DOM
   防御： 浏览器/服务器端做好特殊字符转换校验，重要 cookie 可设置 httpOnly 属性防止被窃取。利用 csp 禁止除安全脚本以外的脚本运行。

2. CSRF(跨站请求伪造)
   原理：诱导用户打开三方网站，利用用户的登录态发起跨站请求。
   分类：
   自动发起 Get 请求(利用 img 等 src 属性)

   ```html
   <img src="https://time.geekbang.org/sendcoin?user=hacker&number=100" />
   ```

   自动发起 Post 请求（利用隐藏表单自动提交 ）

   ```html
   <script> document.getElementById ('hacker-form').submit();</script>
   ```

   不良链接

   ```html
   <img width=150 src=http://images.xuejuzi.cn/1612/1-161230185104_1.jpg></img>
   <a href="https://time.geekbang.org/sendcoin?user=hacker&number=100" taget="\_bla 点击下载美女照片"
   ```

防御：
通过设置 Access-Control-Allow-Origin 不允许或只允许安全域名下发起请求
利用 cookie 的 SameSite 属性 禁止三方请求发送重要 cookie。
利用 token 验证是否为合法三方请求。

## CSP

定义：CSP 是一种用于增强网站安全的策略，他通过限制网页内容的来源和执行的方式来减少恶意攻击。

原理：
· 服务器设置策略：服务器端通过相应头 content-security-policy / content-security-policy-report-only 来设置 CSP 策略。
· 浏览器执行策略：浏览器加载网页时，会根据服务器端返回的 csp 策略限制网页中各种资源的加载和执行，并阻止不符合策略的操作。
· 检查资源来源：浏览器会检查网页中所有资源的来源是否符合 CSP 策略中定义的规则，如果资源的来源于策略不匹配，浏览器组织加载该资源。
· 限制脚本执行：CSP 还可以通过禁止或限制内连脚本的执行来阻止 XSS 攻击。
