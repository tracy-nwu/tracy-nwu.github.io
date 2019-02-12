---
title: 闭包
date: 2018-11-5
---

闭包是函数和声明该函数的词法环境的组合。

### 闭包的本质

JavaScript 闭包的本质源自两点，词法作用域和函数当作值传递。

词法作用域，就是，按照代码书写时的样子，内部函数可以访问函数外面的变量。引擎通过数据结构和算法表示一个函数，使得在代码解释执行时按照词法作用域的规则，可以访问外围的变量，这些变量就登记在相应的数据结构中。

函数当作值传递，即所谓的 first class 对象。就是可以把函数当作一个值来赋值，当作参数传给别的函数，也可以把函数当作一个值 return。一个函数被当作值返回时，也就相当于返回了一个通道，这个通道可以访问这个函数词法作用域中的变量，即函数所需要的数据结构保存了下来，数据结构中的值在外层函数执行时创建，外层函数执行完毕时理因销毁，但由于内部函数作为值返回出去，这些值得以保存下来。而且无法直接访问，必须通过返回的函数。这也就是私有性。

```JavaScript
function makeFunc() {
    var name = "Mozilla";
    function displayName() {
        alert(name);
    }
    return displayName;
}

var myFunc = makeFunc();  //myFunc 返回displayName函数
myFunc();
```

在我们的例子中，myFunc  是执行  makeFunc  时创建的  displayName  函数实例的引用，而  displayName  实例仍可访问其词法作用域中的变量，即可以访问到  name 。由此，当  myFunc  被调用时，name  仍可被访问，其值  Mozilla  就被传递到 alert 中。

### 函数工厂

```JavaScript
function makeAdder(x) {
  return function(y) {
    return x + y;
  };
}

var add5 = makeAdder(5);
var add10 = makeAdder(10);

console.log(add5(2));  // 7
console.log(add10(2)); // 12
```

makeAdder  是一个函数工厂——他创建了将指定的值和它的参数相加求和的函数。在上面的示例中，我们使用函数工厂创建了两个新函数 —— 一个将其参数和 5 求和，另一个和 10 求和。
函数工厂：返回一个函数

### 实用的闭包

闭包很有用，因为它允许将函数与其所操作的某些数据（环境）关联起来。这显然类似于面向对象编程。在面向对象编程中，对象允许我们将某些数据（对象的属性）与一个或者多个方法相关联。
因此，通常你使用只有一个方法的对象的地方，都可以使用闭包。

### 用闭包模拟私有方法

私有方法不仅仅有利于限制对代码的访问：还提供了管理全局命名空间的强大能力，避免非核心的方法弄乱了代码的公共接口部分。

```JavaScript
var Counter = (function() {
  var privateCounter = 0;
  function changeBy(val) {
    privateCounter += val;
  }
  return {
    increment: function() {
      changeBy(1);
    },
    decrement: function() {
      changeBy(-1);
    },
    value: function() {
      return privateCounter;
    }
  }
})();

console.log(Counter.value()); /* logs 0 */
Counter.increment();
Counter.increment();
console.log(Counter.value()); /* logs 2 */
Counter.decrement();
console.log(Counter.value()); /* logs 1 */
```

示例展现了如何使用闭包来定义公共函数，并令其可以访问私有函数和变量。这个方式也称为模块模式。

这次我们只创建了一个词法环境，为三个函数所共享：Counter.increment，Counter.decrement  和  Counter.value。

该共享环境创建于一个立即执行的匿名函数体内。这个环境中包含两个私有项：名为  privateCounter  的变量和名为  changeBy  的函数。这两项都无法在这个匿名函数外部直接访问。必须通过匿名函数返回的三个公共函数访问。

这里我们定义了一个立即执行的匿名函数，并将他的值赋给了变量 counter，用于创建一个计数器。但是，我们可以把这个函数储存在另外一个变量 makeCounter 中，并用他来创建多个计数器。

```JavaScript
var makeCounter = function() {
  var privateCounter = 0;
  function changeBy(val) {
    privateCounter += val;
  }
  return {
    increment: function() {
      changeBy(1);
    },
    decrement: function() {
      changeBy(-1);
    },
    value: function() {
      return privateCounter;
    }
  }
};

var Counter1 = makeCounter();
var Counter2 = makeCounter();
console.log(Counter1.value()); /* logs 0 */
Counter1.increment();
Counter1.increment();
console.log(Counter1.value()); /* logs 2 */
Counter1.decrement();
console.log(Counter1.value()); /* logs 1 */
console.log(Counter2.value()); /* logs 0 */
```

请注意两个计数器  counter1  和  counter2  是如何维护它们各自的独立性的。每个闭包都是引用自己词法作用域内的变量  privateCounter 。
每次调用其中一个计数器时，通过改变这个变量的值，会改变这个闭包的词法环境。然而在一个闭包内对变量的修改，不会影响到另外一个闭包中的变量。

### 在循环中创建闭包——一个常见的错误

作用域链这种配置机制引出了一个值得注意的副作用，闭包只能取得／包含函数／变量对象中／任何变量的最后一个值。

```HTML
<p id="help">Helpful notes will appear here</p>
<p>E-mail: <input type="text" id="email" name="email"></p>
<p>Name: <input type="text" id="name" name="name"></p>
<p>Age: <input type="text" id="age" name="age"></p>
```

```JavaScript
function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}

function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];

  for (var i = 0; i < helpText.length; i++) {
    var item = helpText[i];
    document.getElementById(item.id).onfocus = function() {
      showHelp(item.help);
    }
  }
}

setupHelp();
```

运行这段代码后，您会发现它没有达到想要的效果。无论焦点在哪个 input 上，显示的都是关于年龄的信息。

原因是赋值给  onfocus  的是闭包。这些闭包是由他们的函数定义和在  setupHelp  作用域中捕获的环境所组成的。这三个闭包在循环中被创建，但他们共享了同一个词法作用域，在这个作用域中存在一个变量 item。当 onfocus 的回调执行时，item.help 的值被决定。由于循环在事件触发之前早已执行完毕，变量对象 item（被三个闭包所共享）已经指向了 helpText 的最后一项。

### 解决办法

一、函数工厂

```JavaScript
function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}

function makeHelpCallback(help) {
  return function() {
    return showHelp(help);
  };
}

function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];

  for (var i = 0; i < helpText.length; i++) {
    var item = helpText[i];
    document.getElementById(item.id).onfocus = makeHelpCallback(item.help);
  }
}

setupHelp();
```

二、匿名闭包

```JavaScript
function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}

function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];

  for (var i = 0; i < helpText.length; i++) {
    (function() {
       var item = helpText[i];
       document.getElementById(item.id).onfocus = function() {
         showHelp(item.help);
       }
    })(); // 马上把当前循环项的item与事件回调相关联起来，不至于取到外部的item值
  }
}

setupHelp();
```

三、使用 let

```JavaScript
function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}

function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];

  for (var i = 0; i < helpText.length; i++) {
    let item = helpText[i];
    document.getElementById(item.id).onfocus = function() {
      showHelp(item.help);
    }
  }
}

setupHelp();
```

let 使每个闭包都绑定了块作用域的变量，这意味着不再需要额外的闭包。
