---
title: fetch
date: 2019-1-24
---

参考文章
https://www.w3ctech.com/topic/854
https://juejin.im/post/59165ec2128fe1005cb6c950

### 背景

JavaScript 通过 XMLHttpRequest(XHR)来执行异步请求，这个方式已经存在了很长一段时间。虽说它很有用，但它不是最佳 API。它在设计上不符合职责分离原则，将输入、输出和用事件来跟踪的状态混杂在一个对象里。而且，基于事件的模型与最近 JavaScript 流行的 Promise 以及基于生成器的异步编程模型不太搭（事件模型在处理异步上有点过时了——译者注）。
新的 Fetch API 打算修正上面提到的那些缺陷。 它向 JS 中引入和 HTTP 协议中同样的原语（即 Fetch——译者注）。具体而言，它引入一个实用的函数 fetch()用来简洁捕捉从网络上检索一个资源的意图。
Fetch 规范的 API 明确了用户代理获取资源的语义。它结合 ServiceWorkers，尝试达到以下优化：
改善离线体验
保持可扩展性
到写这篇文章的时候，Fetch API 被 Firefox 39（Nightly 版）以及 Chrome 42（开发版）支持。在 github 上，有基于低版本浏览器的兼容实现

### Fetch 获取数据

使用 Fetch 获取数据很容易。只需要 Fetch 你想获取资源。

假设我们想通过 GitHub 获取一个仓库，我们可以像下面这样使用：

```js
fetch("https://api.github.com/users/chriscoyier/repos");
复制代码;
```

Fetch 会返回 Promise，所以在获取资源后，可以使用.then 方法做你想做的。

```js
fetch('https://api.github.com/users/chriscoyier/repos')
  .then(response => {/* do something */})复制代码
```

如果这是你第一次遇见 Fetch，也许惊讶于 Fetch 返回的 response。如果 console.log 返回的 response，会得到下列信息：

```js
{
  body: ReadableStream;
  bodyUsed: false;
  headers: Headers;
  ok: true;
  redirected: false;
  status: 200;
  statusText: "OK";
  type: "cors";
  url: "http://some-website.com/some-url";
  __proto__: Response;
}
复制代码;
```

可以看出 Fetch 返回的响应能告知请求的状态。从上面例子看出请求是成功的（ok 是 true，status 是 200），但是我们想获取的仓库名却不在这里。

显然，我们从 GitHub 请求的资源都存储在 body 中，作为一种可读的流。所以需要调用一个恰当方法将可读流转换为我们可以使用的数据。

当然 response 只是一个 HTTP 响应，而不是真的 JSON。为了获取 JSON 的内容，我们需要使用 json()方法

还有其他方法来处理不同类型的响应。如果请求一个 XML 格式文件，则调用 response.text。如果请求图片，使用 response.blob 方法。

所有这些方法(response.json 等等）返回另一个 Promise，所以可以调用.then 方法处理我们转换后的数据。

```js
fetch("https://api.github.com/users/chriscoyier/repos")
  .then(response => response.json())
  .then(data => {
    // data就是我们请求的repos
    console.log(data);
  });
复制代码;
```

可以看出 Fetch 获取数据方法简短并且简单。

接下来，让我们看看如何使用 Fetch 发送数据。

### Fetch 发送数据

使用 Fetch 发送也很简单，只需要配置三个参数。

```js
fetch("some-url", options);
复制代码;
```

第一个参数是设置请求方法（如 post、put 或 del），Fetch 会自动设置方法为 get。

第二个参数是设置头部。因为一般使用 JSON 数据格式，所以设置 ContentType 为 application/json。

第三个参数是设置包含 JSON 内容的主体。因为 JSON 内容是必须的，所以当设置主体时会调用

JSON.stringify。

实践中，post 请求会像下面这样：

```js
let content = { some: "content" };
// The actual fetch request
fetch("some-url", {
  method: "post",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify(content)
});
// .then()...复制代码
```

fetch 规范与 jQuery.ajax()主要有两种方式的不同：
当接收到一个代表错误的 HTTP 状态码时，从 fetch()返回的 Promise 不会被标记为 reject， 即使该 HTTP 响应的状态码是 404 或 500。相反，它会将 Promise 状态标记为 resolve （但是会将 resolve 的返回值的 ok 属性设置为 false ），仅当网络故障时或请求被阻止时，才会标记为 reject。
默认情况下，fetch 不会从服务端发送或接收任何 cookies, 如果站点依赖于用户 session，则会导致未经认证的请求（要发送 cookies，必须设置 credentials 选项）。
