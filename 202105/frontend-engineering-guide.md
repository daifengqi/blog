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

Webpack没有很好的中文文档，甚至没有合格的英文文档...

我们这一节的目标是用webpack模拟以上功能，首先

要开始使用Webpack需要，先安装其核心包，

```shell
npm init -y
npm install webpack webpack-cli --save-dev
```

要生成打包后的html文件，需要以下插件

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
    path: path.resolve(__dirname, "dist"),
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

- entry文件是webpack项目的入口，这里是`src/index.js`，这个文件必须自己创建；
- output是打包后的文件和路径，这里指定为`dist/bundle.js`，这个文件由webpack打包后生成，不必自己创建；
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
    "dev": "webpack serve --mode=development --config script/webpack.config.js"
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

### Babel

> 这里的原理部分摘抄自字节前端的[推送](https://mp.weixin.qq.com/s/rioaemy9iRBxPnqFu-zOGQ)。

Babel 是一个 JavaScript 编译器。作为`JS`编译器，`Babel`接收输入的`JS`代码，经过内部处理流程，最终输出修改后的`JS`代码。

在`Babel`内部，会执行如下步骤：

1. 将`Input Code`解析为`AST`（抽象语法树）,这一步称为`parsing`；
2. 编辑`AST`，这一步称为`transforming`；
3. 将编辑后的`AST`输出为`Output Code`，这一步称为`printing`；

从[Babel仓库](https://github.com/babel/babel/tree/main/packages)的源代码，可以发现：`Babel`是一个由几十个项目组成的`Monorepo`。其中`babel-core`提供了以上提到的三个步骤的能力。在`babel-core`内部，更细致的讲：

- `babel-parser`实现第一步
- `babel-generator`实现第三步

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

例如，通过如下语句转移一个jsx文件：

```shell
npx babel file.jsx --presets=@babel/preset-env, @babel/preset-react -o compiled.js
```

其中`--presets`声明presets，如果当前目录已经有了babel配置文件，则可以不指定；`-o`指定输出文件名称，也可以用`--out-file`。

Babel官方提供[在线编译器](https://babeljs.io/repl/)进行在线转换。

注意，当前Babel的版本是Babel7，它与Babel6的presets以及plugins都是不兼容的，通过包的版本号开头的数字可以判断包属于6还是7。

> 参考[阮一峰老师的博客](http://www.ruanyifeng.com/blog/2016/01/babel.html)。

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