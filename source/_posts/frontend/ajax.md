---
title: AJAX
date: 2019-01-03
---

### 为什么是 AJAX？

背景问题：
web 自诞生以来，就沿用“单击，等待”的交互模式,用户单击后，必须刷新才能取得新数据。

AJAX 技术的核心——XMLHttpRequest:
XHR 向服务器发送请求和解析服务器响应提供了流畅的接口。XHR 对象能够以异步的方式从服务器获取更多的信息，然后再通过 DOM 将新数据插入到页面中。意味着用户单击页面后不用刷新页面也可以获取新数据。

### 什么是 AJAX

AJAX 是异步的 JavaScript 和 XML（Asynchronous JavaScript And XML）。实质就是，使用 XMLHttpRequest 对象与服务器通信。AJAX 最吸引人的就是它的“异步”特性，也就是说他可以在不重新刷新页面的情况下与服务器通信，交换数据，或更新页面。

虽然名字中包含 XML 成分，但 AJAX 通信与数据格式无关；这种技术就是不用刷新页面就可以从服务器取得数据，但不一定是 XML 数据。它可以使用 JSON，XML，HTML 和 text 文本等格式发送和接收数据。

两大功能: 1)在不重新加载页面的情况下发送请求给服务器。 2)接受并使用从服务器发来的数据。

### 请求步骤

Step1:发送 HTTP 请求

为了使用 JavaScript 向服务器发送一个 http 请求，需要一个包含必要函数功能的对象实例 XMLHttpRequest
设置对象的 onreadystatechange 属性。告诉 XMLHttp 请求对象,收到响应后是由哪一个 JavaScript 函数处理响应，当请求状态改变时调用函数。

```js
httpRequest.onreadystatechange = nameOfTheFunction;
```

通过调用 HTTP 请求对象的 open() 和 send() 方法，发送一个实际的请求

```js
httpRequest.open("GET", "http://www.example.org/some.file", true);
httpRequest.send();
```

open() 的第一个参数是 HTTP 请求方法 - 有 GET，POST，HEAD 以及服务器支持的其他方法。第二个参数是你要发送的 URL。由于安全原因，默认不能调用第三方 URL 域名。第三个参数是可选的，用于设置请求是否是异步的。
send() 方法的参数可以是任何你想发送给服务器的内容.

Step2：处理服务器响应

在发送请求时，你之前提供的 JavaScript 函数名负责处理响应：

```js
httpRequest.onreadystatechange = nameOfTheFunction;
```

首先，函数要检查请求的状态。如果状态的值是 XMLHttpRequest.DONE （对应的值是 4），意味着服务器响应收到了并且是没问题的，然后就可以继续执行。

```js
if (httpRequest.readyState === XMLHttpRequest.DONE) {
  // Everything is good, the response was received.
} else {
  // Not ready yet.
}
```

全部 readyState 状态值都在 XMLHTTPRequest.readyState，如下也是：
0 (未初始化) or (请求还未初始化)
1 (正在加载) or (已建立服务器链接)
2 (加载成功) or (请求已接受)
3 (交互) or (正在处理请求)
4 (完成) or (请求已完成并且响应已准备好)
然后，检查响应码，区别对待成功和不成功的 AJAX 调用。

```js
if (httpRequest.status === 200) {
  // Perfect!
} else {
  // There was a problem with the request.
  // For example, the response may have a 404 (Not Found)
  // or 500 (Internal Server Error) response code.
}
```

在检查完请求状态和 HTTP 响应码后， 你就可以用服务器返回的数据做任何你想做的了。你有两个方法去访问这些数据：
httpRequest.responseText – 服务器以文本字符的形式返回
httpRequest.responseXML – 以 XMLDocument 对象方式返回，之后就可以使用 JavaScript 来处理

注意上面这一步只在你发起异步请求时有效（既 open() 的第三个参数未特别指定或设为 true）。如果你发起的是同步请求则不必使用函数，但是非常不推荐使用同步请求，用户体验会很烂。

Step3：举例

让我们把所有的知识都集中起来做一个简单的 HTTP 请求。这个 JavaScript 会请求一个 HTML 文档 test.html，包含 "I'm a test" 内容。然后我们 alert() 响应的内容。

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

用户点击 “Make a request” 按钮；
事件处理调用 makeRequest() 函数；
请求已通过然后（onreadystatechange）传给 alertContents() 执行。
alertContents() 检查返回的响应是否 OK，然后 alert() test.html 文件内容。
在通信错误的事件中（例如服务器宕机），在访问响应状态 onreadystatechange 方法中会抛出一个例外。为了缓和这种情况，则可以使用 try...catch 把 if...then 语句包裹起来。

```js
function alertContents() {
  try {
    if (httpRequest.readyState === XMLHttpRequest.DONE) {
      if (httpRequest.status === 200) {
        alert(httpRequest.responseText);
      } else {
        alert("There was a problem with the request.");
      }
    }
  } catch (e) {
    alert("Caught Exception: " + e.description);
  }
}
```

Step 4 – 处理 XML 响应节

XML 被设计用来传输和存储数据。
HTML 被设计用来显示数据。
没有任何行为的 XML
也许这有点难以理解，但是 XML 不会做任何事情。XML 被设计用来结构化、存储以及传输信息。

下面是 John 写给 George 的便签，存储为 XML：

```xml
<note>
  <to>George</to>
  <from>John</from>
  <heading>Reminder</heading>
  <body>Don't forget the meeting!</body>
</note>
```

上面的这条便签具有自我描述性。它拥有标题以及留言，同时包含了发送者和接受者的信息。

但是，这个 XML 文档仍然没有做任何事情。它仅仅是包装在 XML 标签中的纯粹的信息。我们需要编写软件或者程序，才能传送、接收和显示出这个文档。

在上一个例子中，在收到 HTTP 请求的响应后我们会请求对象的 responseText 属性，包含 test.html 文件的内容。现在我们试试 responseXML 属性。
首先，我们创建一个稍后将要请求的有效的 XML 文档。文档（test.html）包含以下内容：

```xml
<?xml version="1.0" ?>
<root>
  I'm a test.
</root>
```

在脚本里我们只需要把请求行改为：

```js
...
onclick="makeRequest('test.xml')">
...
```

然后在 alertContents() 里，我们把 alert(httpRequest.responseText) 改为：

```js
var xmldoc = httpRequest.responseXML;
var root_node = xmldoc.getElementsByTagName("root").item(0);
alert(root_node.firstChild.data);
```

这部分代码采用 responseXML 提供的 XMLDocument 对象，并使用 DOM 方法访问 XML 文档中包含的一些数据。

Step 5 – 处理数据

最后，我们发送一个数据给服务器并收到响应。这次我们用 JavaScript 请求动态页面，test.php 并返回一个计算后的字符串 - “Hello, [user date]”，并用 alert() 出来。
首先要添加一个文本到 HTML 中以方便用户输入名字：

```html
<label
  >Your name:
  <input type="text" id="ajaxTextbox" />
</label>
<span id="ajaxButton" style="cursor: pointer; text-decoration: underline">
  Make a request
</span>
```

还要添加事件处理程序，从表单中获取用户数据连同服务器端的 UTL 一并发送给 makeRequest() 函数：

```js
document.getElementById("ajaxButton").onclick = function() {
  var userName = document.getElementById("ajaxTextbox").value;
  makeRequest("test.php", userName);
};
```

我们还要修改 makeRequest() 让它接受用户数据并将其发给服务器。把请求方法从 GET 改为 POST，把数据作为参数让 httpRequest.send() 调用。

```js
function makeRequest(url, userName) {
    ...
    httpRequest.onreadystatechange = alertContents;
    httpRequest.open('POST', url);
    httpRequest.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    httpRequest.send('userName=' + encodeURIComponent(userName));
  }
```

如果这就是服务器返回的全部内容，alertContents() 函数可以使用 step 3 中的相同函数写。可是，服务器会返回计算后的内容和原内容。所以，如果用户输入 “Jane” 在输入框中，那服务器就会返回如下内容：

```js
{"userData":"Jane","computedString":"Hi, Jane!"}
```

为了在 alertContents() 中使用这个数据，我们可不能只是 alert responseText ，我们要解析它并 alert conputedString，我们想要的属性：

```js
function alertContents() {
  if (httpRequest.readyState === XMLHttpRequest.DONE) {
    if (httpRequest.status === 200) {
      var response = JSON.parse(httpRequest.responseText);
      alert(response.computedString);
    } else {
      alert("There was a problem with the request.");
    }
  }
}
```

test.php 文件应该包含以下内容：

```php
$name = (isset($_POST['userName'])) ? $_POST['userName'] : 'no name';
$computedString = "Hi, " . $name;
$array = ['userName' => $name, 'computedString' => $computedString];
echo json_encode($array);
```

> PHP 是一种创建动态交互性站点的强有力的服务器端脚本语言。
> json 是一种轻量级的数据格式，而不是一种编程语言。可以表示简单值、对象、数组三种数据类型，与 JS 不同，它表示对象和数组时，没有变量和分号，而且对象中的属性也需要加上引号，且必须是双引号。
> 序列化选项 JSON.stringfy()，将 JS 数据转化为 JSON 格式
> 解析选项 JSON.parse()，将 JSON 数据转化为 JS
