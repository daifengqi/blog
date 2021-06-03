# CSS入门笔记

# CSS Tutorials

## 2021 年 5 月 27 日

## May 27 2021



### 引言

这篇博客的内容来自于[W3School上的CSS教程](https://www.w3school.com.cn/css/index.asp)，仅作学习记录。

### CSS选择器

CSS 选择器用于“查找”（或选取）要设置样式的 HTML 元素。

我们可以将 CSS 选择器分为五类：

- 简单选择器（根据名称、id、类来选取元素）
  - 元素选择器
  - id选择器
  - class选择器
- [组合器选择器](https://www.w3school.com.cn/css/css_combinators.asp)（根据它们之间的特定关系来选取元素）
- [伪类选择器](https://www.w3school.com.cn/css/css_pseudo_classes.asp)（根据特定状态选取元素）
- [伪元素选择器](https://www.w3school.com.cn/css/css_pseudo_elements.asp)（选取元素的一部分并设置其样式）
- [属性选择器](https://www.w3school.com.cn/css/css_attribute_selectors.asp)（根据属性或属性值来选取元素）

element[attribute] 选择器用于选取带有指定属性的元素。

例如，

```css
a[target] {
  background-color: yellow;
}
```

还可以通过类似正则表达式的机制实现特殊的属性选择器，详情见链接。

**选择器的组合器**

CSS 中有四种不同的组合器：

- 后代选择器 (空格)
- 子选择器 (>)
- 相邻兄弟选择器 (+)
- 通用兄弟选择器 (~)

### CSS使用

有三种插入样式表的方法：

- 外部 CSS
- 内部 CSS
- 行内 CSS

注意，`JSX`的行内CSS语法，属性键由下划线命名法转变为驼峰命名法。

**层叠顺序**

1. 行内样式（在 HTML 元素中）
2. 外部和内部样式表（在 head 部分，其中内部更高）
3. 浏览器默认样式

### CSS注释

以 `/*` 开始，以 `*/` 结束。

### CSS颜色

指定颜色是通过使用预定义的颜色名称，或 RGB、HEX、HSL、RGBA、HSLA 值。

- RGB值：rgb(*red*, *green*, *blue*)，每个参数定义了 0 到 255 之间的颜色强度。
- RGBA值：rgba(*red*, *green*, *blue*, *alpha*)，具有 alpha 通道的 RGB 颜色值的扩展 - 它指定了颜色的不透明度，*alpha* 参数是介于 0.0（完全透明）和 1.0（完全不透明）之间的数字。
- HEX值：#*rrggbb*，其中 rr（红色）、gg（绿色）和 bb（蓝色）是介于 00 和 ff 之间的十六进制值（与十进制 0-255 相同）。
- HSL值：hsla(*hue*, *saturation*, *lightness*)，色相（*hue*）是色轮上从 0 到 360 的度数。0 是红色，120 是绿色，240 是蓝色；饱和度（*saturation*）是一个百分比值，0％ 表示灰色阴影，而 100％ 是全色；亮度（*lightness*）也是百分比，0％ 是黑色，50％ 是既不明也不暗，100％是白色。
- HSLA值：hsla(*hue*, *saturation*, *lightness*, *alpha*)，是带有 Alpha 通道的 HSL 颜色值的扩展。

### CSS背景

CSS背景有以下4个<u>主要</u>属性，

- background-color
- background-image
- background-repeat
- background-attachment

一个例子，

```css
body {
  background-color: #ffffff;
  background-image: url("tree.png");
  background-repeat: no-repeat;
  background-position: right top;
}

/*简写为*/
body {
  background: #ffffff url("tree.png") no-repeat right top;
}
```

### CSS边框

边框有以下主要属性，

- border-width
- border-style
- border-color

可以这样简写，

```css
p {
  border: 5px solid red;
}
```

**对于单个属性，CSS遵循“上-右-下-左”（顺时针）的顺序。**

还可以通过border-radius添加圆角属性。

### CSS外边距

所有外边距属性都可以设置以下值：

- auto - 浏览器来计算外边距
- *length* - 以 px、pt、cm 等单位指定外边距
- % - 指定以包含元素宽度的百分比计的外边距
- inherit - 指定应从父元素继承外边距

**提示：**允许负值。

**外边距合并：外边距合并指的是，当两个垂直外边距相遇时，它们将形成一个外边距。合并后的外边距的高度等于两个发生合并的外边距的高度中的较大者。**

### CSS内边距

所有内边距属性都可以设置以下值：

- *length* - 以 px、pt、cm 等单位指定内边距
- % - 指定以包含元素宽度的百分比计的内边距
- inherit - 指定应从父元素继承内边距

**提示：**不允许负值。

### CSS轮廓

CSS 拥有如下轮廓属性：

- outline-style
- outline-color
- outline-width
- outline-offset （向外偏移，中空透明）
- outline

outline有类似border的简写。

**提示：**不允许负值。

### CSS框模型

下面给一个例子，先看html，

```html
<div>
  <h1>Hello Vite!</h1>
  <a href="..." target="_blank">Documentation</a>
</div>
<div id="t">Hello 2</div>
```

然后是CSS，

```css
#t {
  margin: 20px;
  border: 30px solid black;
  padding: 20px;
  height: 200px;
  width: 200px;
  background-color: powderblue;
  outline: solid 20px red;
}
```

我们来看Chrome给出的盒模型示意图，

![hemodel](https://cescdf.com/image/hemoxingchrome.png)

从内到外看，有以下注意点：

- width和height是不包括padding的，也就是说padding扩大会扩大整个block；
- background包括了padding + width * height，也就说border之内都会影响；
- border，margin都有自己的规模，向外扩张；
- outline属性是从border开始向外算，和margin相互独立，应该是位于不同层级；

### CSS文本

文本有以下常用属性，

- color
- background-color
- text-align（对齐）
- text-decoration（装饰）
- text-transform（转换大小写）
- text-indent（首行缩进）
- letter-spacing（字母间距，可为负）
- line-height（行高，小于字号会导致上下行文字重叠）
- word-spacing（单词或字间距）
- white-space: nowrap;（禁止换行）
- text-shadow（文本阴影效果）

```css
h1 {
  text-shadow: 2px 2px 5px red;
}
/*水平 垂直 模糊 颜色*/
```

### CSS字体

如下，

- font-family
- font-style
- font-weight
- font-size

### CSS链接

可以根据链接处于什么状态来设置链接的不同样式。

四种链接状态分别是：

- a:link - 正常的，未访问的链接
- a:visited - 用户访问过的链接
- a:hover - 用户将鼠标悬停在链接上时
- a:active - 链接被点击时

### CSS列表

- list-style-type 属性指定列表项标记的类型；
- list-style-image 属性将图像指定为列表项标记；
- list-style-position 属性指定列表项标记（项目符号）的位置（inside或outside）；
- list-style简写属性

### CSS表格

`<th>`是header，`<td>`是data cell。

```css
th, td {
  text-align: center /*水平居中*/
	vertical-align: center /*垂直居中*/
}
```

### CSS布局

**display**

默认情况下，大多数元素的默认 display 值为 **block（块级元素）** 或 **inline（行内元素）**。

`display: none`和`visibility: hidden`的区别：前者是完全不进入HTML流，后者仍会影响布局，只是变得不可见。

**position**

position 属性规定应用于元素的定位方法的类型。

有五个不同的位置值：

- static：HTML 元素默认情况下的定位方式为 static（静态）。静态定位的元素不受 top、bottom、left 和 right 属性的影响。`position: static`的元素不会以任何特殊方式定位；它始终根据页面的正常流进行定位：
- relative：`position: relative`的元素**相对于其正常位置**进行定位。**设置元素的 top、right、bottom 和 left 属性将导致其偏离正常位置进行调整**。不会对其余内容进行调整来适应元素留下的任何空间。
- fixed：`position: fixed` 的元素是相对于视口定位的，这意味着即使滚动页面，它也始终位于同一位置。 top、right、bottom 和 left 属性用于定位此元素。
- absolute：`position: absolute`的元素**相对于最近的定位祖先元素**进行定位（而不是相对于视口定位，如 fixed）。如果绝对定位的元素没有祖先，它将使用文档主体（body），并随页面滚动一起移动。注意，定位祖先元素是除 **static** 以外的任何元素。
- sticky：粘性元素根据用户的滚动位置进行定位，*有点类似Excel中的冻结窗口，比如冻结首行*。

**重叠元素**

z-index 属性指定元素的堆叠顺序（哪个元素应放置在其他元素的前面或后面。具有较高堆叠顺序的元素始终位于具有较低堆叠顺序的元素之前。

**注意：**如果两个定位的元素重叠而未指定 **z-index**，则位于 HTML 代码中最后的元素将显示在顶部。

**溢出的处理方式**

overflow 属性指定在元素的内容太大而无法放入指定区域时是剪裁内容还是添加滚动条。

overflow 属性可设置以下值：

- visible - 默认。溢出没有被剪裁。内容在元素框外渲染
- hidden - 溢出被剪裁，其余内容将不可见
- scroll - 溢出被剪裁，同时添加滚动条以查看其余内容
- auto - 与 scroll 类似，但仅在必要时添加滚动条

**浮动布局**

见[这里](https://www.w3school.com.cn/css/css_float_examples.asp)。

**display: inline-block**

- 与 display: inline 相比，主要区别在于 display: inline-block 允许在元素上设置宽度和高度。同样，如果设置了 display: inline-block，将保留上下外边距/内边距，而 display: inline 则不会。

- 与 display: block 相比，主要区别在于 display：inline-block 在元素之后不添加换行符，因此该元素可以位于其他元素旁边。

**布局：水平和垂直对齐**

见[这里](https://www.w3school.com.cn/css/css_align.asp)。

做一个总结：

- margin: auto；

- text-align: center；

- padding: 外层元素不指定高度，同时指定相等的内边距上下值；

- height值等于 line-height 属性（单行文本），多行文本要配合display: inline-block和vertical-align: middle；

- 使用 position 和 transform 属性；

  ```css
  .center { 
    position: relative;
  }
  
  .center p {
    margin: 0;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }
  ```

- 使用弹性盒模型：flexbox；

  ```css
  .center {
    display: flex;
    justify-content: center; /*水平居中*/
    align-items: center; /*垂直居中*/
  }
  ```

### CSS伪类

**伪类用于定义元素的特殊状态。**

例如，它可以用于：

- 设置鼠标悬停在元素上时的样式
- 为已访问和未访问链接设置不同的样式
- 设置元素获得焦点时的样式

常见的伪类：

- div:hover
- a:active
- p:first-child
- input:focus
- input:disabled
-  p:last-child

**伪元素**

CSS 伪元素用于设置元素指定部分的样式。

例如，它可用于：

- 设置元素的首字母、首行的样式
- 在元素的内容之前或之后插入内容

所有CSS伪元素如下，

| 选择器                                                       | 例子            | 例子描述                      |
| :----------------------------------------------------------- | :-------------- | :---------------------------- |
| [::after](https://www.w3school.com.cn/cssref/selector_after.asp) | p::after        | 在每个 <p> 元素之后插入内容。 |
| [::before](https://www.w3school.com.cn/cssref/selector_before.asp) | p::before       | 在每个 <p> 元素之前插入内容。 |
| [::first-letter](https://www.w3school.com.cn/cssref/selector_first-letter.asp) | p::first-letter | 选择每个 <p> 元素的首字母。   |
| [::first-line](https://www.w3school.com.cn/cssref/selector_first-line.asp) | p::first-line   | 选择每个 <p> 元素的首行。     |
| [::selection](https://www.w3school.com.cn/cssref/selector_selection.asp) | p::selection    | 选择用户选择的元素部分。      |

### CSS用列表设置导航栏

- list-style-type: none; - 删除项目符号。导航条不需要列表项标记。
- 设置 margin: 0; 和 padding: 0; 删除浏览器的默认设置。

垂直导航栏：

```css
li {
  display: block;
}
```

水平导航栏：

```css
li {
  display: inline;
}
```

### CSS下拉菜单

通过`display`属性和伪类`:hover`实现鼠标悬停时显示`dropdown-content`的内容，

```html
<!-- html -->
<div class="dropdown">
  <span>把鼠标移到我上面</span>
  <div class="dropdown-content">
  <p>Hello World!</p>
  </div>
</div>xxxxxxxxxx html
```

```css
/* css */
.dropdown-content {
  display: none;
  position: absolute;
  background-color: #f9f9f9;
  min-width: 160px;
  box-shadow: 0px 8px 16px 0px rgba(0,0,0,0.2);
  padding: 12px 16px;
  z-index: 1;
}

.dropdown:hover .dropdown-content {
  display: block;
}
```

### CSS单位

区别**绝对单位**和**相对单位**。

### CSS选择器顺序

从 0 开始，

- 为 style 属性添加 1000，
- 为每个 ID 添加 100，
- 为每个属性、类或伪类添加 10，
- 为每个元素名称或伪元素添加 1。

 如果将同一规则两次写入外部样式表，那么采用后写入的（视为最新的）规则。

### CSS渐变

CSS 渐变使您可以显示两种或多种指定颜色之间的平滑过渡。

CSS 定义了两种渐变类型：

- *线性渐变*（向下/向上/向左/向右/对角线）
- *径向渐变*（由其中心定义）

线性渐变的语法如下，

```css
div {
  background-image: linear-gradient(direction, color-stop1, color-stop2, ...);
}
```

从左上到右下的`direction`写法：`to right bottom`；也可以用角度指定`direction`，比如`90deg`。

径向渐变，

```css
div {
  background-image: radial-gradient(shape size at position, start-color, ..., last-color);
}
```

默认地，*shape* 为椭圆形，*size* 为最远角，*position* 为中心。

### CSS阴影效果

- text-shadow：为文本添加阴影；
- box-shadow：为元素添加阴影；

通过box-shadow可以创建纸质卡片的效果，

```css
div.card {
  width: 250px;
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
  text-align: center;
}
```

### CSS转换（2D）

CSS 转换（transforms）允许您移动、旋转、缩放和倾斜元素。

通过使用 CSS transform 属性，您可以利用以下 2D 转换方法：

- translate()

  ```css
  /* 相对于自身移动 */
  div {
    transform: translate(50px, 100px);
  }
  ```

- rotate()

  ```css
  div {
    transform: rotate(20deg);
  }
  ```

- scale()：增大、缩小元素；

- scaleX()

- scaleY()

- skew()：倾斜角度；

- skewX()：

- skewY()

- matrix()

matrix() 方法把所有 2D 变换方法组合为一个。

matrix() 方法可接受六个参数，其中包括数学函数，这些参数使您可以旋转、缩放、移动（平移）和倾斜元素。

参数如下：`matrix(scaleX(),skewY(),skewX(),scaleY(),translateX(),translateY())`

### CSS转换（3D）

CSS transform 属性还提供以下 3D 转换方法：

- rotateX()
- rotateY()
- rotateZ()

### CSS过渡

CSS 过渡（transition）允许您在给定的时间内平滑地改变属性值。

- transition：简写属性，用于将四个过渡属性设置为单一属性。
- transition-delay：延迟触发；
- transition-duration：过渡持续时间；
- transition-property：添加过渡的属性；
- transition-timing-function：规定速度曲线函数；

如需创建过渡效果，必须明确三件事：

- 要添加效果的 CSS 属性
- 效果的持续时间
- 触发效果的场景

一个最简单的例子，

```css
div {
  width: 100px;
  transition: width 2s linear 1s; /* 属性 持续时间 函数 延迟时间 */
}

div:hover {
  width: 300px;
}
```

当然也可以把四个属性分开写，此处略。

### CSS动画

动画使元素逐渐从一种样式变为另一种样式。

您可以随意更改任意数量的 CSS 属性。

如需使用 CSS 动画，您必须首先为动画指定一些关键帧。

关键帧包含元素在特定时间所拥有的样式。

- @keyframes
- animation-name
- animation-duration
- animation-delay
- animation-iteration-count
- animation-direction
- animation-timing-function
- animation-fill-mode
- animation

