# 阅读：JavaScript20 年（上）

# Reading: JavaScript 20 years (First Half)

## 2021 年 2 月 19 日

## Feb 19 2021

---

### JavaScript 1.0

> JavaScript 1.0 [[Netscape 1996d](https://cn.history.js.org/references.html#netscape:js1.0:handbook)] 是一种简单的*动态类型*[g](https://cn.history.js.org/appendices.html#dynamically-typed)语言，它支持数字、字符串与布尔值、一等公民函数，以及对象数据类型。从语法上看，JavaScript 与 Java 一样属于 C 家族，其控制流语句借鉴了 C，其表达式语法也包括了大多数 C 的数字运算符。

约 2010 年时的 JavaScript 甚至没有独立的 Array 对象或字面量，没有正则表达式，没有对象字面量，整个 JavaScript 1.0 是一门非常精简的语言。

> JavaScript 1.0 有两个特殊值，用于表示「缺少有用的数据值」。未初始化的变量会被设置为特殊值 _undefined_[17](https://cn.history.js.org/notes.html#17)。这也是程序在尝试访问对象中尚不存在的属性时所返回的值。在 JavaScript 1.0 中，可以通过声明和访问未初始化变量的方式，获取到 _undefined_ 这个值。而值 `null` 则旨在表示某个预期存在对象值的上下文里「没有对象」。它是根据 Java 的 `null` 值建模的，有助于将 JavaScript 与 Java 实现的对象进行集成。在整个历史上，同时存在这样两个相似但又有显著不同的值导致了 JavaScript 程序员的困惑，很多人不确定应在何时使用哪个。

`undefined`和`null`的历史，以及两者的区别。

> JavaScript 1.1 不再需要直接在每个新实例上创建方法属性。它通过函数对象名为 `prototype` 的属性，将*原型*[g](https://cn.history.js.org/appendices.html#prototype)对象与构造函数关联起来。《JavaScript 1.1 指南》[[Netscape 1996e](https://cn.history.js.org/references.html#netscape:js1.1:handbook)] 将 `prototype` 描述为「由所有该类型对象共享的属性」。这是个模糊的描述，更好的表述可能是这样的：原型是一种特殊的对象，其自身属性与所有「由构造函数创建的对象」所共享。

对象原型是 JavaScript1.1 出现的概念，是一种绑定共享机制。

> 函数需要用形式参数列表（formal parameter list）来声明。但参数列表的大小，并不会限制调用函数时可传递的参数数量。如果调用函数时传递的实参（实际参数，argument）数量少于其声明的形参（形式参数，parameter）数量，那么多余的形参将被设置为 _undefined_。而如果传递的实参数量超过形参数量，则会对额外的实参求值，但无法通过形参名称获得这些值。

函数参数一开始就被设定为不定长，并且可以通过`arguments`属性灵活应用。

> JavaScript 有一些令人感到奇特或意外的特性。它们之中有些是故意为之，有些则是在最初的 Mocha 10 天冲刺期间做出的快速设计决策的产物。JavaScript 1.0 也有 bug 和未完成的半成品特性。

一些迷惑特性如下：var 冗余声明、隐式类型转换与==运算符、this 关键字问题等等。

> 在 JavaScript 1.2 中，大多数新加入的库都来自于其他流行语言现有特性的启发。数组 `concat` 和 `slice` 方法的灵感来自 Python 的序列操作，而 `push` / `pop` / `shift` / `unshift` / `splice` 方法都直接根据同名的 Perl 数组函数建模。Python 还启发了字符串的 `concat` / `slice` / `search` 方法。字符串的 `match` / `replace` / `substr` 来自 Perl。Java 启发了 `charCodeAt` 方法。至于正则表达式的字符串匹配语法和语义，借鉴的则还是 Perl。

JavaScript1.2 一些特性的来源。至此，正式引入数组字面量和对象字面量。

### ECMAScript

> 规范制定过程中的某些问题，对语言的使用产生了长期的影响。比如有个受到持续讨论的问题是这样的：短路布尔运算符 `&&`和 `||` 在遇到可转换为布尔值的操作数时，是应该求值为其中一个操作数的实际值（所谓「Perl 风格」），还是 `true` 或 `false` 的布尔值（所谓「Java 风格」）。Brendan Eich 最初的实现主要使用了「Perl 风格」的语义，但少数情况下也有「Java 风格」的行为。微软和 Borland 则已经实现了完整的「Java 风格」语义。最终决定是一致采用「Perl 风格」。

这个决定直接促成了几年后广泛使用的 JavaScript 惯用法：

```js
function f(options) {
  options = options || getDefaultOptionsObject();
  // 如果传递了 options 对象，那么就使用它
  // 否则使用默认的一组配置
  //  ...
}
```

> 早期的交互式 Web 应用主要是基于表单的。用户会将数据输入 HTML 表单，然后由浏览器传输回 Web 服务器，在服务端处理数据并更新数据库，最后将更新的 HTML 文稿传输回浏览器显示。JavaScript 在浏览器端用于基本的输入数据验证，以及对服务端生成的 HTML 做简单的动态更改。这种 Web 应用的形式后来被表述为 Web 1.0[45](https://cn.history.js.org/notes.html#45)。

来看看 Web1.0 的定义。

> 在 2000 年代上半叶，许多组织都使用过类似的技术来构建 Web 应用。但直到 Google 用它来实现 GMail、Google Maps 和其他应用后，这种 Web 应用风格才广为人知。Jesse James Garrett [[2005](https://cn.history.js.org/references.html#ajax)] 创造了「AJAX」一词来形容它。AJAX 和使用它构建的社交媒体应用，成为了 _Web 2.0_[g](https://cn.history.js.org/appendices.html#Web-20) 时代的标志。

Web2.0 的定义。

在【改革失败】这一章，阐述了 ECMAScript 从版本 3 到 4 之间遇到的一些问题，比如微软决定维护自己的技术，比如.Net，而不是 Web 浏览器技术；Flash 以及基于它的 ActionScript 的出现近乎取代了 JavaScript 的作用。

> 第四版的 ECMAScript 语言（ES4）代表了 ECMA 在 1999 年批准的 ECMA-262 标准语言第三版（ES3）的重要演变。ES4 与 ES3 兼容，并增加了重要的设施，用于大型程序设计（类、接口、命名空间、包、程序单元、可选的类型注解，以及可选的静态类型检查和验证）、演化式编程和脚本编写（结构化类型、鸭子类型、类型定义和方法多重派发）、数据结构构建（参数化类型、getter / setter 和元级方法）、控制抽象（适当的尾调用、迭代器和生成器）以及类型自省（类型元对象和堆栈标记）。

初稿 [[Hansen 2007b](https://cn.history.js.org/references.html#lars:overview0)] 名为《ECMAScript 第四版语言概述》的第一段。

> 在 2000 年代的前半段，AJAX 式大型 Web 应用的出现，开始严重突破了第一代引擎的性能限制。到 2006 与 2007 年，Web 开发者对性能问题的呼声越来越高，浏览器厂商也开始派出团队来解决其 JavaScript 引擎的性能限制。

性能革命：促使了 Chrome 的 _V8_[g](https://cn.history.js.org/appendices.html#V8) JavaScript 引擎的开发。

> Node.js 最早被设想为一种用于构建服务端应用的技术。但它已经成为了一个平台，使 JavaScript 能作为通用编程语言，应用在包括小型嵌入式设备在内的各种平台上。Node.js 的 I/O 模块与高性能的 V8 引擎相结合，在能力上足以与 Python 和 Ruby 等其他动态应用语言相媲美，在性能上也往往更胜一筹，成为了编写命令行 JavaScript 应用时的事实标准。Node.js 使掌握了 JavaScript 的 Web 程序员能将其技能转移到其他类型的应用和非浏览器环境中。最初许多客户端 Web 应用的开发者们之所以使用 JavaScript 编程，是因为他们别无选择。而许多 Node.js 开发者选择使用它，反而是因为他们更喜欢用 JavaScript 编程。

CommonJS 和 Node.js 的出现：应运而生。

以上是一些摘抄和想法记录，接下来的章节是《继往开来》，有时间会继续看。
