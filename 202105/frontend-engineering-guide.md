# 前端工程化指南

# Frontend Engineering Guide

## 2021 年 5 月 13 日

## May 12 2021



### Vite

Vite对自身的描述是“下一代前端构建工具”，目前看来基本的思路是替代webpack。相比于webpack，vite有更快的打包和构建思路，代价是只能用ES6模块而非CommonJS的方式来写代码。Vite使用esbuild作为构建工具，这个npm包用Go语言编写，因此Vite有相比Webpack快很多的打包速度。Vite的打包体系基于rollup，同时这也是vite的插件体系。

通过以下代码开启一个Vite项目

```shell
npm init @vitejs/app
```

有意思的，在`select a framework`这一选项中，开发者可以选择React、Vue等知名框架；但如果要使用原生JS，那么对应的选项是

```shell
Select a framework: · vanilla
```

Why？这是一个有趣的故事，详见[前端社区的恶趣味之Vanilla JS](https://cloud.tencent.com/developer/article/1480652)，以及[stackoverflow: what is VanillaJS](https://stackoverflow.com/questions/20435653/what-is-vanillajs)。

在创建项目之后，可以看到项目的主要架构如下

```
vite-project
 |- /node_modules
 |- index.html
 |- main.js
 |- style.css
```

`index.html`是Vite项目的入口文件，其代码内部通过解析如下标签

```html
<script type="module" src="/main.js"></script>
```

引入根目录下的`main.js`文件。这里注意一点，`index.html`中的URL将被自动转换，`"/main.js"`就代表开发目录下的静态路径，这方便了开发者引入文件。

而css文件是通过`main.js`文件的第一行代码引入的：

```js
import './style.css'

document.querySelector('#app').innerHTML = `
  <h1>Hello Vite!</h1>
  <a href="https://vitejs.dev/guide/features.html" target="_blank">Documentation</a>
`
```

模块和模板字符串都是ES6的语法，可见Vite默认开发者使用这些功能。

> 与静态 HTTP 服务器类似，Vite 也有 “根目录” 的概念，即文件被提供的位置。你会看到它在整个文档中用 `<root>` 表示。源码中的绝对 URL 路径将以项目的 “根” 作为基础来解析，因此你可以像在普通的静态文件服务器上一样编写代码。
>
> [——Vite中文文档](https://vitejs.bootcss.com/guide/#overview)

利用Vite创建的原生JS项目非常适合用来做测验，当你要测试某些js、css或html代码的功能时，通过启动Vite项目来快速获取这些文件，并带有dev模式和“热更新”等功能，简直不要太爽。

### Webpack

> Webpack没有很好的中文文档，甚至没有合格的英文文档...官方的解释是，webpack是一个静态打包工具。相比于`gulp`等前端流程自动化（基于任务处理）工具，webpack更强调模块化。

我们这一节的目标是用webpack模拟Vite的功能。

webpack依赖于node环境，要开始使用Webpack需要，先安装其核心包，

```shell
npm init -y
npm install webpack webpack-cli --save-dev
```

要生成打包后的html文件，需要以下<u>插件</u>，注意，插件是webpack的一个概念，用于用户动态添加新功能，

```shell
npm install -D webpack-dev-server html-webpack-plugin
```

Webpack并不直接提供文件，因此除了打包后的文件，其他文件都需要手动创建，

```
  webpack-demo
  |- package.json
  |- /script/webpack.config.js
  |- /src/index.js
  |- dist
  |- index.html
```

先来看config文件，

```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: "./src/index.js", // 入口文件
  output: {
    // 出口文件和路径
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist"), // 动态获取路径
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: "index.html",
    }),
  ],
  devServer: {
    contentBase: path.resolve(__dirname, "dist"),
    hot: true,
    compress: true,
    port: "3000",
  },
};
```

有几点：

- **entry**文件是webpack项目的入口，这里是`src/index.js`，这个文件必须自己创建；
- **output**是打包后的文件和路径，这里指定为`dist/bundle.js`，这个文件由webpack打包后生成，不必自己创建；
- plugins是webpack插件，这里用到的`html-webpack-plugin`使得html文件作为打包出口；
- devServer带来了热更新，文件压缩等webpack的重要功能，稍后会详细介绍。

接着写`index.js`，我们用到`lodash`这个包，

```shell
npm install --save lodash
```

然后是内容，

```javascript
import _ from "lodash";

function component() {
  const element = document.createElement("div");

  element.innerHTML = _.join(["Hello", "webpack"], " ");

  return element;
}

document.body.appendChild(component());
```

位于根目录下的html文件作为模板，

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Webpack App</title>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

修改`package.json`

```js
  "scripts": {
    "dev": "webpack serve --mode=development --config script/webpack.config.js",
    "build": "webpack --mode=production --config script/webpack.config.js"
  },
```

运行命令`npm run dev`就能在`localhost:3000`看到运行的结果了。

需要注意的是，webpack的热更新基于入口文件`index.js`及其关联文件，所以对作为模板的`index.html`文件的修改不会引起热更新，更谈不上更新css了。

要怎么做才能在热更新里同时调试js、css、html呢？最好的办法还是基于JSX的写法，那么就需要用到React，可以参考这篇[从零搭建React开发环境](https://mp.weixin.qq.com/s/XLe5kSOLUbY3gSzLR-u12A)。

笔者参照上篇文章写了一个脚手架，可通过以下方式使用

```shell
git clone https://github.com/daifengqi/webpack-react.git
cd webpack-react
npm install
npm run dev
```

另外，运行`npm run build`构建项目。

**从前端模块化开始讲起吧**

一开始前端只需要写`script`标签，自从AJAX技术出现，就有了前后端分离，前端代码开始变得复杂。为了应对代码量的与日俱增，模块化应运而生。

模块化的发展相见我的另一篇[博客](https://cescdf.com/blog/202105/javascript-node-v8-basic#JavaScript%E6%A8%A1%E5%9D%97%E5%8C%96%E8%BF%9B%E5%8C%96%E8%BF%87%E7%A8%8B)。

> 总而言之，模块化是为了正确处理分块开发时的命名冲突问题。

这里重点讲一下CommonJS和ES6怎么实现的模块化，

**CommonJS**

CommonJS的关键字有`module`, `exports`, `require`。

原理暂略。

**ES6**

ES6直接定义了`import`和`export`这两个关键字，另外，大多数浏览器已经通过`type`直接支持了ES6模块，

```html
<script src="abc.js" type="module"></script>
```

`export default`只能用一次，它允许导入的时候用户自定义变量或函数的名字。

`import * as m from "module"`允许通过`m.data`取出模块的内容，也避免了命名污染。

原理也先暂略，之后要有时间再补上。

**bundle**

Webpack打包的命令如下，

```shell
webpack ./src/main.js ./dist/bundle.js
```

这样webpack会自动处理`main.js`文件里的依赖，然后最终打包到`bundle.js`中。

如果想把webpack的打包命令直接绑定到`npm run build`上，则需要在package.json文件中这样写，

```json
"scirpt": {
  "build": webpack,
}
```

这样命令行运行`npm run build`之后，webpack就会去找`webpack.config.js`文件，通过文件里入口和出口的声明（以及其他配置），就能输出打包文件。

**loader**

加载html文件，css以及各种变体（纯css也可以通过`require`引入，加上合适的loader就能实现，所以打包样式也是webpack的功能），还有从`.vue`，`.jsx`文件转译成普通js文件，都需要loader来完成，不添加loader的webpack是不具备这些功能的。

下面我们来配置一个css-loader，在`webpack.config.js`中，

```json
module.exports = {
  entry: '...',
  output: {
    ...,
  },
  module: {
    rules: [
      {
        test: "/\.css$/",
        use: ['style-loader', 'css-loader'],
      }
    ]
  }
}
```

其实就是在`module-rules`这个列表里配置一个对象，这个对象有两个属性，

- `test`：对应文件的正则表达式；
- `use`：对应文件用到的loader，是一个列表；

简单讲一下这两个loader，`css-loader`负责加载css文件，`style-loader`负责把样式挂载到DOM中，webpack这个版本**（3）**是从右向左读，所以顺序也不能乱（我觉得新版本**（5）**应该解决了这种小问题吧）。

**less**

这里不讲less，只讲一下webpack的<u>缝合</u>功能。其实像Babel，less这些东西都有自己的npm包和命令行功能的，

webpack只是一个打包工具，它可以把这些东西都配置成一个所谓“<u>环境</u>”，这样你就不用去单独整这些工具，而是直接使用封装在webpack里的对应的loader，所以：

**webpack-loader只是整合了现有工具，掌握webpack的根本在于理解并使用这些工具，webpack只是一个工具封装集合。**

**图片**

在纯css里通过url引用图片路径，也需要一个`url-loader`，代码示例如下，

> url-loader: A loader for webpack which transforms files into **base64** URIs.

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif|jpeg)$/i,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192, // 单位
            },
          },
        ],
      },
    ],
  },
};
```

> `url-loader`也可以[加载svg](https://v4.webpack.js.org/loaders/url-loader/#svg)。

注意，url的加载涉及开发时和发布时的路径问题，这个问题可以通过`output-publicPath`配置解决。

**babel**

Babel也可以在webpack里通过loader配置，即`babel-loader`。

官方推荐的配置是，

```js
module: {
  rules: [
    {
      test: /\.m?js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env']
        }
      }
    }
  ]
}
```

注意`exclude`属性的使用。

**plugins**

loader我们可以看作转换器，而plugins是一个扩展器，对webpack的功能进行扩展。

下面我们来随机介绍一个插件，比如给自己的代码加上版权、作者和协议声明。

- `webpack.BannerPlugin()`

这是webpack自带的插件，直接`webpack = require('webpack')`就可以使用。

```js
const webpack = require('webpack');

module.exports = {
  // ...
  plugins: [
    new webpack.BannerPlugin('hhh'),
  ]
}
```

> 所有的webpack自带插件链接：点[这里](https://webpack.js.org/plugins/)。

- `HtmlWebpackPlugin()`

**基本用法：**该插件将为您生成一个HTML5文件，其中包含使用`<script>`标记在正文中的所有webpack包。将插件添加到您的网页包配置中，如下所示：

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');
const path = require('path');

module.exports = {
  entry: 'index.js',
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'index_bundle.js',
  },
  plugins: [new HtmlWebpackPlugin()],
};
```

这将会自动生成 `dist/index.html` 文件，并包含如下内容，

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>webpack App</title>
  </head>
  <body>
    <script src="index_bundle.js"></script>
  </body>
</html>
```

你也可以在初始化插件时加入`template`属性，以创建不同模板的html。

>If you have multiple webpack entry points, they will all be included with `<script>` tags in the generated HTML.
>
>If you have any CSS assets in webpack's output (for example, CSS extracted with the [MiniCssExtractPlugin](https://webpack.js.org/plugins/mini-css-extract-plugin/)) then these will be included with `<link>` tags in the `<head>` element of generated HTML.

- `webpack-dev-server`

webpack官方的例子是，

```js
var path = require('path');

module.exports = {
  //...
  devServer: {
    contentBase: path.join(__dirname, 'dist'),
    compress: true,
    port: 9000,
  },
};
```

注意，要通过`npm run dev`启动，还需要配置`package.json`，

```json
"script": {
  "dev": "webpack-dev-server --open"
}
```

这个插件最好用的是**热更新**，配置如下，

```js
module.exports = {
  //...
  devServer: {
    hot: true,
  },
};

```

> 点击[这里](https://webpack.js.org/concepts/hot-module-replacement/)查看官方写的热更新原理。

**mode**

webpack还可以分离开发时和生产环境的配置，通过同时调整webpack.config.js的文件名，以及属性`mode`和package.json里的`scripts`标签来分离环境，这里略。

### Babel

> 这里的原理部分摘抄自字节前端的[推送](https://mp.weixin.qq.com/s/rioaemy9iRBxPnqFu-zOGQ)。

Babel 是一个 JavaScript 编译器。作为`JS`编译器，`Babel`接收输入的`JS`代码，经过内部处理流程，最终输出修改后的`JS`代码。

对于Babel的使用，我们需要安装如下包，

```shell
npm init -y
npm install --save-dev @babel/core @babel/cli @babel/preset-env @babel/polyfill
npm install -D @babel/preset-react
```

一一说明：

- `@babel/core`：提供编译AST及转换能力；
- `@babel/cli`：用于命令行转换；
- `@babel/preset-env`：实现高级语法的降级；
- `@babel/polyfill`：实现API的polyfill；
- `@babel/preset-react`：JSX语法转换器；

下面说一下babel的配置，babel添加配置文件有两个选择：

- `babel.config.json`：用于`monorepo`或需要编译`node_modules`；
- `.babelrc.json`：用于文件的单个部分；

配置内容如下：

```json
{
  "presets": [...],
  "plugins": [...],
}
```

也可以在`package.json`中这样配置：

```json
{
  "name": "my-package",
  "version": "1.0.0",
  "babel": {
    "presets": [ ... ],
    "plugins": [ ... ],
  }
}
```

使用Babel的一个好处是，可以通过将新语法转移成比较早期的JS语法，来理解这些新语法是怎么实现的，比如转译ES6的新特性，或者JSX语法。

如果只是想做polyfill，取消ES6及之后的特性，

```bash
npx babel file.jsx --presets=@babel/preset-env -o compiled.js
```

另外，如果是转移一个jsx文件，则需要添加react配置：

```shell
npx babel file.jsx --presets=@babel/preset-env, @babel/preset-react -o compiled.js
```

其中`--presets`声明presets，如果当前目录已经有了babel配置文件，则可以不指定；`-o`指定输出文件名称，也可以用`--out-file`。

Babel官方提供[在线编译器](https://babeljs.io/repl/)进行在线转换。

注意，当前Babel的版本是Babel7，它与Babel6的presets以及plugins都是不兼容的，通过包的版本号开头的数字可以判断包属于6还是7。

> 参考[阮一峰老师的博客](http://www.ruanyifeng.com/blog/2016/01/babel.html)。

在`Babel`内部，会执行如下步骤：

1. 将`Input Code`解析为`AST`（抽象语法树）,这一步称为`parsing`；
2. 编辑`AST`，这一步称为`transforming`；
3. 将编辑后的`AST`输出为`Output Code`，这一步称为`printing`；

从[Babel仓库](https://github.com/babel/babel/tree/main/packages)的源代码，可以发现：`Babel`是一个由几十个项目组成的`Monorepo`。其中`babel-core`提供了以上提到的三个步骤的能力。在`babel-core`内部，更细致的讲：

- `babel-parser`实现第一步
- `babel-generator`实现第三步

### TypeScript

> 参考[教程](https://ts.xcatliu.com/introduction/index.html)。

废话少说，直接上手：

```shell
npm init -y
npm install typescript
```

新建一个`hello.ts`文件，

```typescript
function sayHello(person: string) {
    return 'Hello, ' + person;
}

let user = 'Tom';
console.log(sayHello(user));
```

然后执行

```shell
npx tsc hello.ts
```

就可以看到生成的对应`hello.js`文件。

TypeScript的配置文件可以这样生成，是一个默认的模板

```
npx tsc --init
```

我们来看看TypeScript支持的原始数据类型：

```typescript
// 布尔值
let isDone: boolean = false;
// 数值
let decLiteral: number = 6;
// 字符串
let myName: string = 'Tom';
// 返回空值的函数
function alertName(): void {
    alert('My name is Tom');
}
// 空值变量只能是null或undefined
let unusable: void = undefined;
// 任意值
let myFavoriteNumber: any = 'seven';
```

如果在声明有赋值变量时没有指定类型，就默认为`any`，但是ts会做类型推断，从而受到类型约束。**如果定义的时候没有赋值，不管之后有没有赋值，都会被推断成 `any` 类型而完全不被类型检查**：

```typescript
// 联合类型
let myFavoriteNumber: string | number;
myFavoriteNumber = 'seven';
myFavoriteNumber = 7;
// 联合类型的方法调用只能是共有的
function getString(something: string | number): string {
 // return something.length; 报错
    return something.toString();
}
```

对象的类型由接口实现，

```typescript
interface Person {
    name: string;
    age?: number; // 可选属性
}

let tom: Person = {
    name: 'Tom',
    age: 25
};
```

此外，还有数组类型，函数类型，内置对象（`Boolean`、`Error`、`Date`、`RegExp`，或`HTMLElement`、`Event`、`NodeList`）、等等类型约束，这里不展开讲了。

