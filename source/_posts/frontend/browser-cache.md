---
title: 浏览器缓存相关知识整理
date: 2018-10-3
---

浏览器缓存是提升页面性能同时减少服务器压力的利器，但同时在开发中经常会由于浏览器缓存而展示了「错误」的东西。那么浏览器缓存的机制究竟是怎样的？今天在网上看了一些资料之后，在这里做一个简单的整理，希望对你也会有所帮助。

web 缓存分为很多种，比如数据库缓存、代理服务器缓存、CDN 缓存，以及浏览器缓存。

浏览器缓存是将文件保存在客户端，在同一个会话过程中会检查缓存的副本是否足够新，在后退网页时，访问过的资源可以从浏览器缓存中拿出使用。通过减少服务器处理请求的数量，用户将获得更快的体验

浏览器缓存分为强缓存和协商缓存

### 强制缓存／文档过期验证

命中强制缓存

当浏览器对某个资源的请求命中了强制缓存时，返回的 http 状态为 200，在 chrome 的开发者工具的 network 里面 size 会显示为 from memory cache
![](https://cdn.nlark.com/yuque/0/2018/png/160131/1545310126003-5e7dbd94-ff09-4cd6-ae47-f1060c7c3639.png)

强制缓存原理

强制缓存是利用 Expires 或者 Cache-Control 这两个 http response header 实现的，它们都用来表示资源在客户端缓存的有效期。

Expires 和 Cache-Control

Expires 是 http1.0 提出的一个表示资源过期时间的 header，它是第一次发送 HTTP 请求时，由服务器返回的绝对时间，用 GMT 格式的字符串表示，如：Expires:Thu, 31 Dec 2037 23:55:55 GMT。当再次发送请求时，将请求时间与 Expires 指定的时间进行比较，确认缓存是否过期，若时间在 Expires 之前则未过期，命中缓存。

Expires 是较老的强制缓存管理 header，由于它是服务器返回的一个绝对时间，在服务器时间与客户端时间相差较大时，缓存管理容易出现问题，比如随意修改下客户端时间，就能影响缓存命中的结果。所以在 http1.1 的时候，提出了一个新的 header，就是 Cache-Control。

Cache-Control 是一个相对时间，在配置缓存的时候，以秒为单位，用数值表示，如：Cache-Control:max-age=315360000，在进行缓存命中的时候，都是利用客户端时间进行判断，所以相比较 Expires，Cache-Control 的缓存管理更有效，安全一些。

当 response header 中，Expires 和 Cache-Control 同时存在时，Cache-Control 优先级高于 Expires：

实现过程：

1)浏览器第一次跟服务器请求一个资源，服务器在返回这个资源的同时，在 respone 的 header 加上 Expires 或 Cache-Control 的 header，如：
![](https://cdn.nlark.com/yuque/0/2018/png/160131/1545310477052-bf862467-bf49-4fea-88b9-8d8e04442880.png)

2)浏览器在接收到这个资源后，会把这个资源连同所有 response header 一起缓存下来（所以缓存命中的请求返回的 header 并不是来自服务器，而是来自之前缓存的 header）；

3）浏览器再请求这个资源时，先从缓存中寻找，找到这个资源后，拿出它的 Expires 和 Cache-Control 跟当前的请求时间比较，如果请求时间在指定的过期时间之前，就能命中缓存，否则就不行。

4）如果缓存没有命中，浏览器直接从服务器加载资源时，Expires 和 Cache-Control 的 header 会被更新。

Cache-Control 的取值：

在 HTTP/1.1 中，Cache-Control 几乎取代了 Expires，成为最重要的规则，主要用于控制网页缓存，那么 Cache-Control 又是如何控制的呢？它的主要取值有：

max-age（单位为 s）：指定设置缓存最大的有效时间，定义的是时间长短。当浏览器向服务器发送请求后，在 max-age 这段时间里浏览器就不会再向服务器发送请求了，即使服务器上的资源发生了变化，浏览器也不会得到通知。max-age 会覆盖掉 Expires，后面会有讨论。

s-maxage（单位为 s）：同 max-age，只用于共享缓存（比如 CDN 缓存）。 比如，当 s-maxage=60 时，在这 60 秒中，即使更新了 CDN 的内容，浏览器也不会进行请求。也就是说 max-age 用于普通缓存，而 s-maxage 用于代理缓存。如果存在 s-maxage，则会覆盖掉 max-age 和 Expires header。

public：指定响应会被缓存，并且在多用户间共享。即客户端缓存和代理器缓存如果没有指定 public 还是 private，则默认为 public。
![](https://cdn.nlark.com/yuque/0/2018/png/160131/1545308709115-bf32eabe-0769-454c-bb48-783cd7828fad.png)

private ：响应只作为私有的缓存，不能在用户间共享。如果要求 HTTP 认证，响应会自动设置为 private。

no-cache ：指跳过文档过期的验证而直接进行服务器再验证，而 no-store 是指资源禁止被缓存。因此有的时候只设置 no-cache 防止缓存还是不够保险，还可以加上 private 指令，将过期时间设为过去的时间。

no-store : 绝对禁止缓存，每次请求资源都要从服务器重新获取。

must-revalidate：指定如果页面是过期的，则去服务器进行获取。这个指令并不常用，就不做过多的讨论了。

强制缓存的应用

强制缓存是前端性能优化最有力的工具，没有之一，对于有大量静态资源的网页，一定要利用强制缓存，提高响应速度。通常的做法是，为这些静态资源全部配置一个超时时间超长的 Expires 或 Cache-Control，这样用户在访问网页时，只会在第一次加载时从服务器请求静态资源，其它时候只要缓存没有失效并且用户没有强制刷新的条件下都会从自己的缓存中加载。

然而这种缓存配置方式会带来一个新的问题，就是发布时资源更新的问题，比如某一张图片，在用户访问第一个版本的时候已经缓存到了用户的电脑上，当网站发布新版本，替换了这个图片时，已经访问过第一个版本的用户由于缓存的设置，导致在默认的情况下不会请求服务器最新的图片资源，除非他清掉或禁用缓存或者强制刷新，否则就看不到最新的图片效果。

禁止浏览器缓存静态资源的方法

可以设置请求头: Cache-Control: no-cache, no-store, must-revalidate .

当然, 还有一种常用做法: 即给请求的资源增加一个版本号,这样做的好处就是你可以自由控制什么时候加载最新的资源. 如下:

```js
<link rel="stylesheet" type="text/css" href="css/style.css?version=1.8.9" />
```

### 协商缓存／服务器再验证

协商缓存的原理

协商缓存是利用的是【Last-Modified，If-Modified-Since】和【ETag、If-None-Match】这两对 Header 来管理的。

【Last-Modified，If-Modified-Since】

1）浏览器第一次跟服务器请求一个资源，服务器在返回这个资源的同时，在 respone 的 header 加上 Last-Modified 的 header，这个 header 表示这个资源在服务器上的最后修改时间：
![](https://cdn.nlark.com/yuque/0/2018/png/160131/1545314628466-5abf69ee-717b-4add-9359-7a4adae3de6d.png)

2）浏览器再次跟服务器请求这个资源时，在 request 的 header 上加上 If-Modified-Since 的 header，这个 header 的值就是上一次请求时返回的 Last-Modified 的值：
![](https://cdn.nlark.com/yuque/0/2018/png/160131/1545314615595-90406da4-53da-4c6a-aabf-5b7fd10e230d.png)

3）服务器再次收到资源请求时，根据浏览器传过来 If-Modified-Since 和资源在服务器上的最后修改时间判断资源是否有变化，如果没有变化则返回 304 Not Modified，但是不会返回资源内容；如果有变化，就正常返回资源内容。当服务器返回 304 Not Modified 的响应时，response header 中不会再添加 Last-Modified 的 header，因为既然资源没有变化，那么 Last-Modified 也就不会改变.

4）浏览器收到 304 的响应后，就会从缓存中加载资源。

5）如果协商缓存没有命中，浏览器直接从服务器加载资源时，Last-Modified Header 在重新加载的时候会被更新，下次请求时，If-Modified-Since 会启用上次返回的 Last-Modified 值。

【Last-Modified，If-Modified-Since】都是根据服务器时间返回的 header，一般来说，在没有调整服务器时间和篡改客户端缓存的情况下，这两个 header 配合起来管理协商缓存是非常可靠的，但是有时候也会服务器上资源其实有变化，但是最后修改时间却没有变化的情况，而这种问题又很不容易被定位出来，而当这种情况出现的时候，就会影响协商缓存的可靠性。所以就有了另外一对 header 来管理协商缓存，这对 header 就是【ETag、If-None-Match】。

【ETag、If-None-Match】

1）浏览器第一次跟服务器请求一个资源，服务器在返回这个资源的同时，在 respone 的 header 加上 ETag 的 header，这个 header 是服务器根据当前请求的资源生成的一个唯一标识，这个唯一标识是一个字符串，只要资源有变化这个串就不同，跟最后修改时间没有关系，所以能很好的补充 Last-Modified 的问题：
![](https://cdn.nlark.com/yuque/0/2018/png/160131/1545314822927-5c5508ac-e03e-4bf0-9818-dc13fa41704b.png)

2）浏览器再次跟服务器请求这个资源时，在 request 的 header 上加上 If-None-Match 的 header，这个 header 的值就是上一次请求时返回的 ETag 的值：
![](https://cdn.nlark.com/yuque/0/2018/png/160131/1545314876121-afb2fa05-de70-4a1c-a407-2e373e582aa4.png)

3）服务器再次收到资源请求时，根据浏览器传过来 If-None-Match 和然后再根据资源生成一个新的 ETag，如果这两个值相同就说明资源没有变化，否则就是有变化；如果没有变化则返回 304 Not Modified，但是不会返回资源内容；如果有变化，就正常返回资源内容。与 Last-Modified 不一样的是，当服务器返回 304 Not Modified 的响应时，由于 ETag 重新生成过，response header 中还会把这个 ETag 返回，即使这个 ETag 跟之前的没有变化。

### 总结：

浏览器与服务器通信的方式为应答模式，即：浏览器发起 HTTP 请求 – 服务器响应该请求。

浏览器第一次向服务器发起该请求后拿到请求的结果中，除了请求的资源，还有包含了 Expires 或 Cache-Control、Last-Modified、ETag 等信息的响应，浏览器会根据响应报文中 HTTP 头的缓存标识（Expires 或 Cache-Control），决定是否缓存结果，是就将请求结果和缓存标识存入浏览器缓存中，简单的过程如下图：
![](https://cdn.nlark.com/yuque/0/2018/png/160131/1545316857666-ef44d404-b0b4-4a0d-8b79-9ffaa8453c7f.png)

下面一张流程图完整说明当浏览器发起 HTTP 请求时缓存机制的过程
![](https://cdn.nlark.com/yuque/0/2018/png/160131/1545316154267-d8197006-7142-447d-aea0-9395d6bd869c.png)

相关文章推荐
https://juejin.im/entry/5ad86c16f265da505a77dca4
https://juejin.im/post/59c602276fb9a00a3d135f2e
https://segmentfault.com/a/1190000008377508?utm_source=weekly&utm_medium=email&utm_campaign=email_weekly
