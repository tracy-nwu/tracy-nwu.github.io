---
title: 创建一个ajax应用程序
date: 2018-9-5
---

关于这篇文章：
MDN 上介绍 ajax 的文章https://developer.mozilla.org/zh-CN/docs/Web/Guide/AJAX/Getting_Start
该篇中实现了一个 Make a request 按钮，当点击这个按钮，浏览器就会弹出 ajax 请求的文件的内容，本文将就我在写这个例子时遇到的问题进行一些分析。

### 概念：

AJAX 是异步的 JavaScript 和 XML（Asynchronous JavaScript And XML）。AJAX 最大的优点是在不重新加载整个页面的情况下，可以与服务器交换数据并更新部分网页内容。
XMLHttpRequest（即 XHR）对象是 ajax 的核心，简而言之，其实 ajax 就是使用 XMLHttpRequest 对象的方法和属性实现与服务器通信的。

### 准备工作：

建立 ajax－demo 文件夹，并在其中建立 index.html 文件，在<body>写入以下内容

```html
<button id="ajaxButton" type="button">Make a request</button>

<script>
  (function() {
    var httpRequest;
    document
      .getElementById("ajaxButton")
      .addEventListener("click", makeRequest);

    function makeRequest() {
      httpRequest = new XMLHttpRequest();

      if (!httpRequest) {
        alert("Giving up :( Cannot create an XMLHTTP instance");
        return false;
      }
      httpRequest.onreadystatechange = alertContents;
      httpRequest.open("GET", "test.html");
      httpRequest.send();
    }

    function alertContents() {
      if (httpRequest.readyState === XMLHttpRequest.DONE) {
        if (httpRequest.status === 200) {
          alert(httpRequest.responseText);
        } else {
          alert("There was a problem with the request.");
        }
      }
    }
  })();
</script>
```

<br/>
### 问题一：

在浏览器中打开 index.html 时，总是弹出“There was a problem with the request.”
![](https://cdn.nlark.com/yuque/0/2018/png/160131/1542366348472-a2c0ceda-db61-4544-9c7b-c485c144a006.png)
原因是访问 test.html 的路径被阻断了，让我们定位到相关代码

```js
httpRequest.open("GET", "test.html");
```

httpRequest 是 XMLHttpRequest 对象的实例，继承了它的方法和属性，这里的 open()便是其中之一。
open()  接受三个参数，第一个是 HTTP 请求方法 - 通常有 GET，POST，HEAD。  第二个参数你要取得的内容的地址，改地址是相对于执行代码的当前页面的（当然也可以是绝对路径），通常是一段 URL，如 http://服务器地址/test。第三个参数是表示是否异步加载的布尔值，默认为true，可省略。
这里第二个参数直接写为 test.html，表明是这是一段本地路径。所以我们需要起一个本地的 server 服务，解决方法就是利用 vscode 中的 Live Server 插件。安装 Live Server。
![](https://cdn-pri.nlark.com/yuque/0/2018/png/160131/1542367100098-9fd952de-a310-4a18-80fa-cce48b43f504.png)
<br/>
![](https://cdn-pri.nlark.com/yuque/0/2018/png/160131/1542367113922-fc2a2122-b487-4946-bdad-ad3ae5fdcbaf.png)
<br/>
使用 Live Server 打开 index.html，依然弹出“There was a problem with the request.”查看 console 口
![](https://cdn-pri.nlark.com/yuque/0/2018/png/160131/1542367364564-a1183f08-86fd-46ba-9f04-53f314932748.png)
浏览器提示，找不到相关文件，那么我们就建立 test.html 文件，随意写上你喜欢的一个句子，这里我们写“This is a test”

```bash
This is a test
```

再次打开，就会弹出我们想要的内容了。

### 问题二：

为什么整段 ajax 代码要包含在一个立即执行的匿名函数中？
因为变量 httpRequest 是要在 ajax 的全局范围内使用的，它会在 makeRequest() 函数中被相互覆盖，从而导致资源竞争。为了避免这个情况，所以在包含 AJAX 函数的闭包中声明 httpRequest 变量。

### ajax 运行过程小结：

1.发送请求，收到响应。
1.1 告诉请求对象是由哪一个 JavaScript 函数处理响应

```js
httpRequest.onreadystatechange = nameOfTheFunction;
```

1.2 发送一个实际的请求，通过调用 HTTP 请求对象的 open() 和 send() 方法
2 处理服务器响应
2.1 函数要检查请求的状态。状态值是否是 XMLHttpRequest.DONE
2.2 检查响应码 200 OK
2.3 具体设置处理响应的函数
