---
title: 从零开始搭建ReactApp
date: 2018-8-1
---

这篇文章可以：
帮助初学者更好的理解 React 应用程序，当使用 create-creat-app 命令时，react 内部发生了什么
帮助了解 React 应用中的 webpack、babel 等脚手架是如何发挥作用的，为什么要使用它们

### Step 1：建立文件结构及初始化

创建一个文件夹用以保存我们的项目，并用编辑器打开它

```bash
mkdir createReactAppFromScracth
code createReactAppFromScracth
```

执行 npm init 用来初始化生成一个新的 package.json 文件。它会向用户提问一系列问题，如果你觉得不用修改默认配置，一路回车就可以了。或者直接执行 npm init -y 选择默认配置。
安装 react 和 react－dom，因为我们将要在代码中使用它。在命令行运行

```bash
 npm install --save react react-dom
```

创建以下文件结构：

```bash
+-- public
+-- src
```

public 文件夹用来存放静态文件，最重要的是存放我们的 index.html 文件，该文件将用于呈现 React 应用程序。你可以将以下 HTML 代码复制到你的文件 index.html 中。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>React Starter</title>
  </head>
  <body>
    <div id="root"></div>
    <noscript> You need to enable JavaScript to run this app. </noscript>
    <script src="../src/index.js"></script>
  </body>
</html>
```

src 文件夹用来存放源文件，这里我们新建一个 App.js 和 index.js 文件。如果你之前接触过 react，应该对这部分非常熟悉。首先，我们在 App.j 简单写一个组件。

```js
import React, { Component } from "react";
class App extends Component {
  render() {
    return (
      <div>
        <h1>HELLO WORLD</h1>
      </div>
    );
  }
}
export default App;
```

接着在 index.js 中，引入它并挂载。

```js
import React from "react";
import ReactDOM from "react-dom";
import App from "./App.js";

ReactDOM.render(<App />, document.getElementById("root"));
```

当最终构建好 ReactAPP 之后，会有部分我们不希望提交的代码（如 node_modules），所以我们需要继续在根目录添加.gitignore 文件。

```text
node_module/
.vscode
dist/
```

做好上面的准备工作后，现在我们进入本文的重点部分。

### Step 2：打开 index.html 文件

这时，我们在浏览器中打开 index.html 文件，浏览器 console 口会报错
![](https://cdn.nlark.com/yuque/0/2018/png/160131/1541659827297-6e432b11-c7a7-40f0-b36a-fa1991cf985a.png)
报错信息：在 index.js 文件的第一行，出现了浏览器不能识别的代码。说明浏览器不能解析 react，事实上它只能解析普通的 javascript 代码（ES5）。显然我们需要一个帮助我们把 react 里的 JSX 语言编译为普通 ES5 的工具，这就是 babel。
由于 babel 是集成在 webpack 中与 loader 配合使用的，所以现在我们先把目光转向 webpack。

### Step 3：安装 webpack

webpack 本质上是一个静态模块打包器(module bundler)，它可以将 react 所有这些模块打包成一个或多个  bundle。详见https://www.webpackjs.com/
现在我们来安装 webpack，在命令行运行 npm install --save-dev webpack webpack-cli
--save-dev 表示保存为开发依赖，dev 即 devlepment，安装完成后，你会在 packge.json 文件中的 devDependencies 找到他们
在项目的根目录下创建一个名为 webpack.config.js 的新文件。在此文件中，我们会对 webpack 进行的一些配置，从而导出打包后的文件。

```js
const path = require("path");
module.exports = {
entry: "./src/index.js",
mode: "development",
output: {
path: path.resolve(\_\_dirname, "dist/"),
publicPath: "/dist/",
filename: "bundle.js"
}
};
```

entry 属性指明入口文件是./src/index.js，webpack 从这里开始构建依赖图，并捆绑我们的文件。
mode 属性指明我们正在开发模式下工作——这使我们不必在运行开发服务器时添加模式标志。
output 属性告诉 Webpack 将捆绑代码放在何处。
path 指明 bundle 所在目录的绝对路径，这里我们把它放在 dist 目录下
publicPath 属性指定 bundle 应该进入的目录，并告诉 webpack-dev-server 从哪里提供文件，从而帮助我们使用 dev-server。
同时，它指定在浏览器中所引用的「此输出目录对应的公开 URL」。 如果指定了一个错误的值，则在加载这些资源时会收到 404 错误，因为服务器将无法从正确的位置提供你的文件！
最后捆绑文件命名为 bundle.js
在 packge.json 文件的“scripts”中添加以下代码

```json
"scripts": {
"start": "webpack",
   //...
}
```

在命令行运行 npm start，生成 dist 目录，我们会发现虽然 webpackge 虽然打包生成了 bundle.js 文件，但其中代码并没有被编译，浏览器依旧不能识别。这是因为我们还没有使用 babel。
将 index.html 中引入 javascript 的地址改为捆绑文件的地址

```html
<script src="../dis/bundle.js"></script>
```

### Step 4：安装 babel

安装 babel。在命令行运行

```bash
npm install --save-dev @babel/core @babel/cli  @babel/preset-env @babel/preset-react
```

babel-core 是主要的 babel 包，它让 babel 可以对我们的代码进行任意转换。
babel-cli 允许我们从命令行编译文件（cli 即 command line interface 的缩写）。
preset-react 和 preset-env 都是转换特定代码风格的预设——在这种情况下，env 预设允许我们将 ES6 +转换为更传统的 ES5，react 预设则是允许将 JSX 转换为 ES5。
在项目根目录中，创建一个名为.babelrc 的文件。在这里，我们告诉 babel 我们将使用 env 和 react 预设

```js
{
"presets": ["@babel/env", "@babel/preset-react"]
}

```

### Step 5：webpack 配置-loder

webpack 的 loader 用于对模块的源代码进行编译。 可以使你在  import  或"加载"模块时预处理文件。
在 webpack.config.js 文件的 module.exports 中加入以下代码

```js
module.exports = {
  //...
  module: {
    rules: [
      {
        test: /\.(js|jsx)\$/,
        exclude: /(node_modules|bower_components)/,
        loader: "babel-loader",
        options: { presets: ["@babel/env"] }
      }
    ]
  },
  resolve: { extensions: ["*", ".js", ".jsx"] }
};
```

module 对象中的 rules 数组，定义了导出的 javascript 模块的编译方式以及编译哪些模块。
这里 rules 数组仅有一条规则，用来改变 ES6 和 JSX 语法。
test 和 exclude 属性是匹配文件类型的条件。这里它将匹配 node_modules 和 bower_components 目录之外的任何内容。
loader 属性指定 babel-loader 来处理匹配到的文件。
最后，在 options 属性中指定使用 env 预设。（options 的值可以传递到 loader 中，我们可以将其理解为 loader 的一个选项。）
resolve 属性指定 Webpack 将解析哪些扩展——这允许我们引入模块而无需添加其扩展名

```js
import File from "../path/to/file";
```

此时在浏览器中打开 index.html 文件，就可以看到我们写的 HELLO WORLD 了。

### Step 6：webpack 配置-devServer

我们希望通过 npm start 可以直接生成一个端口来查看项目，而不是之前在浏览器中打开 HTML 文件的方法。所以我们需要新建一个服务器。
安装 webpack-dev-server。在命令行执行 npm i webpack-dev-server --save-dev
在 webpack.config.js 文件中加入以下代码

```js
module.exports = {
 //...
devServer: {
contentBase: path.join(\_\_dirname, "public/"),
port: 3000,
publicPath: "http://localhost:3000/dist/"
}
};
```

我们在 devServer 属性中设置了 webpack-dev-server。
contentBase 属性中我们提供静态文件的位置（例如我们的 index.html）
我们想要运行服务器的端口。 请注意，devServer 还具有 publicPath 属性。
此 publicPath 告诉服务器我们的捆绑代码实际上在哪里。
最后一点可能有点令人困惑 - 请注意这里：output.publicPath 和 devServer.publicPath 是不同的。
在 packge.json 文件的 scripts 中加入以下代码

```json
"dev": "webpack-dev-server"
```

在命令行执行 npm run dev，点击生成的链接http://localhost:3000/，就可以看到我们写的HELLO WORLD 了。

### Step 7：webpack 配置-热模块加载

现在我们改动 App.js 文件中“HELLO WORLD”为“HELLO REACT”，浏览器中的网页内容并没有自动更新，需要我们手动刷新网页才可以。
Hot Module Replacement（热模块替换）可以自动刷新我们在文件中的更改。这就涉及到 webpack 插件的用法。
首先安装 react-hot-loader。
其次，我们对配置文件所做的就是在 plugins 属性中实例化插件的新实例，并确保在 devServer 中将 hotOnly 设置为 true。

```js
const webpack = require("webpack");

module.exports = {
devServer: {
contentBase: path.join(\_\_dirname, "public/"),
port: 3000,
publicPath: "http://localhost:3000/dist/",
hotOnly: true
},
plugins: [new webpack.HotModuleReplacementPlugin()]
};
```

在 App.js 中导入 react-hot-loader，并通过修改代码将导出的对象标记为 hot-reloaded，如下所示。

```JS
import React, { Component} from "react";
import {hot} from "react-hot-loader";

class App extends Component{
render(){
return(

<div>
<h1>HELLO WORLD</h1>
</div>
);
}
}

export default hot(module)(App);
```

### Finally

最终的配置文件长这样：

```jsx
const path = require("path");
const webpack = require("webpack");

module.exports = {
    entry: "./src/index.js",
    mode: "development",
    module: {
    rules: [
    {
        test: /\.(js|jsx)\$/,
        exclude: /(node_modules|bower_components)/,
        loader: "babel-loader",
        options: { presets: ["@babel/env"] }
    }
    },
    resolve: { extensions: ["*", ".js", ".jsx"] },
    output: {
    path: path.resolve(**dirname, "dist/"),
    publicPath: "/dist/",
    filename: "bundle.js"
    },
    devServer: {
    contentBase: path.join(**dirname, "public/"),
    port: 3000,
    publicPath: "http://localhost:3000/dist/",
    hotOnly: true
    },
    plugins: [new webpack.HotModuleReplacementPlugin()]
};
```

最终的文件结构长这样：

```bash
.
+-- public
| +-- index.html
+-- src
| +-- App.js
| +-- index.js
+-- .babelrc
+-- .gitignore
+-- package-lock.json
+-- package.json
+-- webpack.config.js
```

### 总结

你会发现，我们创建一个 react 项目，其核心是 webpack。它以 index.js 为入口，开始构建其内部依赖图，对它们进行编译，打包成为一个文件输出，与 devServer 合作提供给 react 一个运行端口，然后加入一个插件，使它实现自动更新。
当你了解这些以后，再去看其他框架就会有更深入的理解。
