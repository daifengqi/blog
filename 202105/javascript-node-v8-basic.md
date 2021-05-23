# JavaScript基础知识点汇总

# JavaScript From Zero to One

## 2021 年 5 月 11 日

## May 11 2021



### 重要参考

- [自检清单](http://www.conardli.top/blog/article/%E7%BB%BC%E5%90%88/%E3%80%90%E8%87%AA%E6%A3%80%E3%80%91%E5%89%8D%E7%AB%AF%E7%9F%A5%E8%AF%86%E6%B8%85%E5%8D%95.html#%E4%B8%80%E3%80%81javascript%E5%9F%BA%E7%A1%80)
- [淘系消费者运营前端团队](https://github.com/mqyqingfeng/Blog)



### JavaScript变量类型

[ECMAScript标准](http://www.ecma-international.org/ecma-262/9.0/index.html)规定了`7`种数据类型，其把这`7`种数据类型又分为两种：原始类型和对象类型。

**原始类型**

- `Null`：只包含一个值：`null`
- `Undefined`：只包含一个值：`undefined`
- `Boolean`：包含两个值：`true`和`false`
- `Number`：整数或浮点数，还有一些特殊值（`-Infinity`、`+Infinity`、`NaN`）
- `String`：一串表示文本值的字符序列
- `Symbol`：一种实例是唯一且不可改变的数据类型

(在`es10`中加入了第七种原始类型`BigInt`，现已被最新`Chrome`支持)

**对象类型**

- `Object`：自己分一类丝毫不过分，除了常用的`Object`，`Array`、`Function`等也都属于特殊的对象

其中，原始类型具有**不可变性**。`ECMAScript`中所有的函数的参数都是按值传递的，对于引用对象，传递的是一个**同名拷贝**，指向相同的堆内存的地址。因此，在函数体内是不可以通过赋值修改作为参数的引用对象的。

比较：

- 对于原始类型，比较时会直接比较它们的值，如果值相等，即返回`true`。
- 对于引用类型，比较时会比较它们的引用地址，虽然两个变量在堆中存储的对象具有的属性值都是相等的，但是它们被存储在了不同的存储空间，因此比较值为`false`。

### 深入理解JavaScript数据类型

在`ECMAScript`关于类型的定义中，只给出了`Object`类型，实际上，我们平时使用的很多引用类型的变量，并不是由`Object`构造的，但是它们原型链的终点都是`Object`，这些类型都属于引用类型。

- `Array` 数组
- `Date` 日期
- `RegExp` 正则
- `Function` 函数

为了便于操作基本类型值，`ECMAScript`还提供了几个特殊的引用类型，他们是基本类型的包装类型：

- `Boolean`
- `Number`
- `String`

引用类型和包装类型的主要区别就是对象的生存期，使用new操作符创建的引用类型的实例，在执行流离开当前作用域之前都一直保存在内存中，而自基本类型则只存在于一行代码的执行瞬间，然后立即被销毁，这意味着我们不能在运行时为基本类型添加属性和方法。

### 基本类型和包装类型：装箱拆箱

- 装箱转换：把基本类型转换为对应的包装类型；
- 拆箱操作：把引用类型转换为基本类型；

既然原始类型不能扩展属性和方法，那么我们是如何使用原始类型调用方法的呢？

每当我们操作一个基础类型时，后台就会自动创建一个包装类型的对象，从而让我们能够调用一些方法和属性，例如下面的代码：

```js
var name = "ConardLi";
var name2 = name.substring(2);
```

实际上发生了以下几个过程：

- 创建一个`String`的包装类型实例
- 在实例上调用`substring`方法
- 销毁实例

### 隐式类型转换

因为`JavaScript`是弱类型的语言，所以类型转换发生非常频繁，上面我们说的装箱和拆箱其实就是一种类型转换。还有其他典型的隐式转换场景，如下

- 在`if`语句和逻辑语句中，如果只有单个变量，会先将变量转换为`Boolean`值；
- 我们在对各种非`Number`类型运用数学运算符(`- * /`)时，会先将非`Number`类型转换为`Number`类型;注意`+`是个例外，执行`+`操作符时：
  1. 当一侧为`String`类型，被识别为字符串拼接，并会优先将另一侧转换为字符串类型。
  2. 当一侧为`Number`类型，另一侧为原始类型，则将原始类型转换为`Number`类型。
  3. 当一侧为`Number`类型，另一侧为引用类型，将引用类型和`Number`类型转换成字符串后拼接。
- 使用`==`时，若两侧类型相同，则比较结果和`===`相同，否则会发生隐式转换；

一道经典的面试题，如何让：`a == 1 && a == 2 && a == 3`。

根据上面的拆箱转换，以及`==`的隐式转换，我们可以轻松写出答案：

```js
const a = {
   value:[3,2,1],
   valueOf: function () {return this.value.pop(); },
} 
```

### 判断数据类型的方式

**typeof**

使用`typeof`操作符判断原始类型，不能用其来判断引用类型，因为除函数外所有的引用类型都会被判定为`object`。

**instanceof**

`instanceof`操作符可以帮助我们判断引用类型具体是什么类型的对象：

```js
[] instanceof Array // true
new Date() instanceof Date // true
new RegExp() instanceof RegExp // true
```

然而，使用`instanceof`来检测数据类型，不会很准确，这不是它设计的初衷：

```js
[] instanceof Object // true
function(){}  instanceof Object // true
```

另外，使用`instanceof`也不能检测基本（原始）数据类型，而只能检测原始类型的包装函数，所以`instanceof`并不是一个很好的选择。

```javascript
new String('foo') instanceof String; // true
new String('foo') instanceof Object; // true

'foo' instanceof String; // false
'foo' instanceof Object; // false
```

**toString**

最好的方法是：

```javascript
Object.prototype.toString.call()
```

这个方法在一种情况下是不可靠的，即对象的`Symbol.toStringTag`属性被污染时，

```javascript
const myDate = new Date();
Object.prototype.toString.call(myDate);     // [object Date]

myDate[Symbol.toStringTag] = 'myDate';
Object.prototype.toString.call(myDate);     // [object myDate]
```

我们来看看`jquery`源码中如何进行类型判断：

```js
var class2type = {};
jQuery.each( "Boolean Number String Function Array Date RegExp Object Error Symbol".split( " " ),
function( i, name ) {
	class2type[ "[object " + name + "]" ] = name.toLowerCase();
} );

type: function( obj ) {
	if ( obj == null ) {
		return obj + "";
	}
	return typeof obj === "object" || typeof obj === "function" ?
		class2type[Object.prototype.toString.call(obj) ] || "object" :
		typeof obj;
}

isFunction: function( obj ) {
		return jQuery.type(obj) === "function";
}
```

原始类型直接使用`typeof`，引用类型使用`Object.prototype.toString.call`取得类型，借助一个`class2type`对象将字符串多余的代码过滤掉，例如`[object function]`将得到`array`，然后在后面的类型判断，如`isFunction`直接可以使用`jQuery.type(obj) === "function"`这样的判断。

### Number精度丢失和大数问题

在[ECMAScript®语言规范](http://www.ecma-international.org/ecma-262/5.1/#sec-4.3.19)中可以看到，`ECMAScript`中的`Number`类型遵循`IEEE 754`标准。使用64位固定长度来表示。事实上有很多语言的数字类型都遵循这个标准，例如`JAVA`,所以很多语言同样有着上面同样的问题。

由于尾数位只能存储`52`个数字，这就能解释`toString(2)`的执行结果了。`0.1、0.2`有着同样的问题，其实正是由于这样的存储，在这里有了精度丢失，导致了`0.1+0.2!=0.3`。

JavaScript能表示的最大数字为`1.111...`X 2^1023 这个结果转换成十进制是`1.7976931348623157e+308`,这个结果即为`Number.MAX_VALUE`。JavaScript中`Number.MAX_SAFE_INTEGER`表示最大安全数字,计算结果是`9007199254740991`，即在这个数范围内不会出现精度丢失（小数除外）,这个数实际上是`1.111...`X 252。

`bigInt`类型在`es10`中被提出，现在`Chrome`中已经可以使用，使用`bigInt`可以操作超过最大安全数字的数字。

### JavaScript内存空间

在`JavaScript`中，每一个变量在内存中都需要一个空间来存储。

内存空间又被分为两种，栈内存与堆内存。

栈内存：

- 存储的值大小固定
- 空间较小
- 可以直接操作其保存的变量，运行效率高
- 由系统自动分配存储空间

`JavaScript`中的原始类型的值被直接存储在栈中，在变量定义时，栈就为其分配好了内存空间。

堆内存：

- 存储的值大小不定，可动态调整
- 空间较大，运行效率低
- 无法直接操作其内部存储，使用引用地址读取
- 通过代码进行分配空间

引用类型的值存储在堆内存中，它在栈中只存储了一个固定长度的地址，这个地址指向堆内存中的值。

### JavaScript底层数据结构

V8里面所有的数据类型的根父类都是Object，Object派生HeapObject，提供存储基本功能，往下的JSReceiver用于原型查找，再往下的JSObject就是JS里面的Object，Array/Function/Date等继承于JSObject。左边的FixedArray是实际存储数据的地方。

在创建一个JSObject之前，会先把读到的Object的文本属性序列化成constant_properties，如下的data：

```js
var data = {
  name: "yin",
  age: 18,
  "-school-": "high school"
};
```

会被序列成：

> ```
> ../../v8/src/runtime/[http://runtime-literals.cc](https://link.zhihu.com/?target=http%3A//runtime-literals.cc) 72 constant_properties:
> 0xdf9ed2aed19: [FixedArray]
> – length: 6
> [0]: 0x1b5ec69833d1 <String[4]: name>
> [1]: 0xdf9ed2aec51 <String[3]: yin>
> [2]: 0xdf9ed2aec71 <String[3]: age>
> 
> \[3]: 18
> 
> [4]: 0xdf9ed2aec91 <String[8]: -school->
> [5]: 0xdf9ed2aecb1 <String[11]: high school>
> ```

它是一个FixedArray，一共有6个元素，由于data总共是有3个属性，每个属性有一个key和一个value，所以Array就有6个。第一个元素是第一个key，第二个元素是第一个value，第三个元素是第二个key，第四个元素是第二个key，依次类推。Object提供了一个Print()的函数，把它用来打印对象的信息非常有帮助。上面的输出有两种类型的数据，一种是String类型，第二种是整型类型的。

FixedArray是V8实现的一个类似于数组的类，它表示一段连续的内存，上面的FixedArray的length = 6，那么它占的内存大小将是：

```text
length * kPointerSize
```

因为它存的都是对象的指针（或者直接是整型数据类型，如上面的18），在64位的操作系统上，一个指针为8个字节，它的大小将是48个字节。它记录了一个初始的内存开始地址，使用元素index乘以指针大小作为偏移，加上开始地址，就可以取到相应index的元素，这和数组是一样的道理。只是V8自己封装了一个，方便添加一些自定义的函数。

全文见[知乎](https://zhuanlan.zhihu.com/p/26169639)。

### ES6之Symbol

每个从Symbol()返回的symbol值都是唯一的。一个symbol值能作为对象属性的标识符；这是该数据类型仅有的目的。

Symbol除了**唯一性**以外，还具有**不可枚举性**，当使用`Symbol`作为对象属性时，可以保证对象不会出现重名属性，调用`for...in`不能将其枚举出来，另外调用`Object.getOwnPropertyNames、Object.keys()`也不能获取`Symbol`属性。由此可见，借助`Symbol`类型的不可枚举，我们可以在类中模拟私有属性，控制变量读写。Symbol还可以用于防止属性污染，在某些情况下，我们可能要为对象添加一个属性，此时就有可能造成属性覆盖，用`Symbol`作为对象属性可以保证永远不会出现同名属性。

**例子：React对Symbol的应用（防止XSS攻击）**

在`React`的`ReactElement`对象中，有一个`$$typeof`属性，它是一个`Symbol`类型的变量：

```js
var REACT_ELEMENT_TYPE =
  (typeof Symbol === 'function' && Symbol.for && Symbol.for('react.element')) ||
  0xeac7;
```

`ReactElement.isValidElement`函数用来判断一个React组件是否是有效的，下面是它的具体实现。

```js
ReactElement.isValidElement = function (object) {
  return typeof object === 'object' && object !== null && object.$$typeof === REACT_ELEMENT_TYPE;
};
```

可见`React`渲染时会把没有`$$typeof`标识，以及规则校验不通过的组件过滤掉。

如果你的服务器有一个漏洞，允许用户存储任意`JSON`对象， 发起攻击的客户端代码需要一个字符串，这可能会成为一个问题；而`JSON`中不能存储`Symbol`类型的变量，这就是防止`XSS`的一种手段。

> [ECMAScript](https://262.ecma-international.org/6.0/#sec-symbol-description)对Symbol的描述。
>
> 简要版如下：
>
> 1. Symbol 值通过 Symbol 函数生成，使用 typeof，结果为 "symbol"
> 2. Symbol 函数前不能使用 new 命令，否则会报错。这是因为生成的 Symbol 是一个原始类型的值，不是对象。
> 3. instanceof 的结果为 false
> 4. Symbol 函数可以接受一个字符串作为参数，表示对 Symbol 实例的描述，主要是为了在控制台显示，或者转为字符串时，比较容易区分。
> 5. 如果 Symbol 的参数是一个对象，就会调用该对象的 toString 方法，将其转为字符串，然后才生成一个 Symbol 值。
> 6. Symbol 函数的参数只是表示对当前 Symbol 值的描述，相同参数的 Symbol 函数的返回值是不相等的。
> 7.  Symbol 值不能与其他类型的值进行运算，会报错。
> 8.  Symbol 值可以显式转为字符串。
> 9. Symbol 值可以作为标识符，用于对象的属性名，可以保证不会出现同名的属性。
> 10. Symbol 作为属性名，该属性不会出现在 for...in、for...of 循环中，也不会被 Object.keys()、Object.getOwnPropertyNames()、JSON.stringify() 返回。但是，它也不是私有属性，有一个 Object.getOwnPropertySymbols 方法，可以获取指定对象的所有 Symbol 属性名。
> 11. 如果我们希望使用同一个 Symbol 值，可以使用 Symbol.for。它接受一个字符串作为参数，然后搜索有没有以该参数作为名称的 Symbol 值。如果有，就返回这个 Symbol 值，否则就新建并返回一个以该字符串为名称的 Symbol 值。
> 12. Symbol.keyFor 方法返回一个已登记的 Symbol 类型值的 key。

### JavaScript原型和原型链

先看一张网图：

![proto](https://cescdf.com/image/jsproto.jpeg)

注意，以下讨论的范畴不包括null和undefined。

- 每一个**JavaScript对象**都有一个原型，通过`__proto__`指向；
- 每一个**JavaScript函数**都有一个`prototype`属性，且仅有函数拥有，该属性是一个对象；

通过调用构造函数（`new`）创建的实例，其原型（`__proto__`）指向构造函数的`prototype`属性。

```javascript
function Person() {
	// ...
}
var person = new Person();
person.__proto__ === Person.prototype; // true
```

这么一来，实例的`__proto__`和构造函数的`prototype`是一个东西，这个东西我们就称为**JavaScript对象的原型**。

**第三个概念：constructor**

原型一定有一个属性：`constructor`，原型通过`constructor`属性指向构造函数，即

```javascript
Object.prototype.constructor === Object // true
```

该例中，`Object`是一个（构造）函数。

```javascript
Object.prototype.toString.call(Object) // "[object Function]"
typeof Object // function
```

**原型链**

当读取一个JavaScript对象（实例）的属性或调用方法时，是先在实例对象本身上查找，如果找不到就去原型上找；如果在这个对象的原型上（`o.__proto__`）还找不到该属性或方法，就沿着原型链，在原型的原型（`o.__proto__.__proto__`）上继续查找，直到空。

```javascript
let a = [];
a.__proto__ === Array.prototype // true
a.__proto__.__proto__ === Object.prototype // true
```

所以，原型链其实就是由`__proto__`连接成的一条链表。

**原型链的终点就是Object的原型，它指向空对象**，

```javascript
Object.prototype.__proto__ === null // true
```

**构造函数原型链的次终点是Function的原型，之后的最终点才是Object的原型**，

```javascript
Array.__proto__ === Function.prototype // true
Function.__proto__ === Function.prototype // true
Function.__proto__.__proto__ === Object.prototype // true
```

注意，

- 构造函数的原型链是从Function继承下来的；
- 构造函数的原型的原型链直接从Object继承下来；

```javascript
Array.__proto__ === Function.prototype // true
Array.prototype.__proto__ === Object.prototype // true
```

这说明构造函数自身所在的原型链多绕了Function的原型一次，

```javascript
Function.prototype.__proto__ === Object.prototype // true
```

### JavaScript实现继承的几种方式

关于JavaScript的继承，《红宝书》提出了**6**种实现继承的方式，分别是

- 原型链
- 盗用构造函数
- 组合继承
- 原型式继承
- 寄生式继承
- 寄生式组合继承

下面将一一简述：。

**原型链**

简单回顾一下构造函数、原型和实例的关系：**每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针**，即

```javascript
Array.prototype.constructor === Array // true

let arr = new Array();
arr.__proto__ === Array.prototype // true
```

原型链的实现关键是，**子类不使用默认的原型，而是将其替换成父类的实例**；核心代码如下，

```javascript
// 继承SuperType
SubType.prototype = new SuperType(); 
```

这样以来，子类的实例就可以通过原型链（`__proto__`）找到父类的实例原型。在读取子类实例上的属性时，首先在实例上搜索，如果没有则会搜索继承的实例原型。在通过原型链实现继承以后，搜索就可以自下而上，搜索原型的原型。

- 优点：自定义类型最终会指向`Object.prototype`，能够继承包括`toString()、valueOf()`在内的所有默认方法；
- 缺点：1. 原型中包含引用值时，会在所有实例间共享，这是不能接受的。这也是为什么属性通常定义在构造函数而不是定义在原型上的原因。2. 子类型在实例化时不能给父类型的构造函数传参。

**盗用构造函数**

盗用构造函数又称对象伪装或经典继承，基本思路很简单：**在子类构造函数中调用父类构造函数**。如下，

```javascript
function SubType() {
  SuperType.call(this);
}
```

通过使用`call()`方法，在`Subtype`的实例创建的新对象的上下文中执行了`SuperType`构造函数，这相当于在子类对象上运行了父类函数所有的初始化代码。

- 优点：可以向父类构造函数传参，

  ```javascript
  function SubType(args) {
    SuperType.call(this, args[0]);
    
    this.para2 = args[1];
  }
  ```

- 缺点：1. 必须在构造函数中定义方法，因此函数不能重用。2. 子类也不能访问父类原型上定义的方法。

**组合继承**

组合继承也称伪经典继承，**它综合了原型链和盗用构造函数：使用原型链继承原型上的属性和方法，使用盗用构造函数来继承实例属性。**

```javascript
function SubType(name, age) {
  SuperType.call(this, name);
  
  this.age = age;
}

SubType.prototype = new SuperType();
```

- 优点：弥补了原型链和盗用构造函数的不足；
- 缺点：效率问题，父类构造函数被调用了两次，一次是在子类的构造函数中调用，另一次是覆盖子类原型时。这导致父类的属性始终存在两组，一组在子类的实例上，另一组在子类的原型上。

**原型式继承**

原型式继承适用于一种特殊情况：**在一个对象的基础上创建另一个对象，并且不需要定义新的子类型**。原型式继承非常适合不需要单独创建构造函数，但仍需要在对象间共享信息的场合。

大致代码如下，

```javascript
function object(o) {
  function F() {};
  F.prototype = o;
  return new F();
}
```

`object`函数会创建一个临时构造函数`F()`，将传入的对象赋值给构造函数的原型，然后返回新对象的实例。本质上，`object`对传入的对象进行了一次<u>浅复制</u>。

ECMAScript5通过增加`Object.create()`方法将原型式继承的概念规范化了，并可以通过第二个可选参数添加新的属性或覆盖同名属性，

```javascript
let tom = {
  name: "Tom",
  friends: ["Amy", "Hado", "Van"],
};

let greg = Object.create(tom, {
  name: {
    value: "Greg",
    writable: true
  },
});
```

注意，第二个参数必须安装`Object.defineProperties`的写法，给对象定义新的属性，参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties)。

- 原型式继承的缺点也是引用值会在相关对象间共享。

**寄生式继承**

寄生式继承背后的思路很简单：**原型式继承+增强对象**，即创建一个实现继承的函数返回的对象，以某种方式增强对象，然后返回这个对象。

```javascript
function create(origin) {
  let clone = object(origin); // 创建新对象
  clone.newFunc = function() { // 增强对象
    // ...
  }
  return clone;
}
```

注意，`object`可以是任何一个返回新对象的函数。

- 优点：同原型式继承，主要关注对象，而不在乎类型和构造函数。
- 缺点：通过寄生式继承给对象添加的函数难以重用，与盗用构造函数模式类似。

**寄生组合式继承**

寄生组合式继承解决了组合继承的效率问题：通过寄生式继承来取得父类原型，然后将返回的新对象赋值给子类原型。寄生组合式继承的基本思路如下，

```javascript
function extend(subType, superType) {
  let prototype = object(superType.prototype); // 创建对象
  prototype.constructor = subType;             // 增强对象
  subType.prototype = prototype;               // 赋值对象
}
```

本质上，子类原型最终是要包含超类对象的所有实例属性，子类构造函数只要在执行时重写自己的原型就行了。

- 优点：只调用了一次超类的构造函数，避免了子类原型上存在不必要也用不到的属性，效率更高。可以说，寄生组合式继承是引用类型继承的最佳模式。

### React的继承

Vue通过[Vue.extend()](https://cn.vuejs.org/v2/api/?#Vue-extend)来继承组件，而React不提倡使用继承，

> 在 Facebook，我们在成百上千个组件中使用 React。我们并没有发现需要使用继承来构建组件层次的情况。
>
> Props 和组合为你提供了清晰而安全地定制组件外观和行为的灵活方式。注意：组件可以接受任意 props，包括基本数据类型，React 元素以及函数。
>
> ——[React官方文档：组合 vs 继承](https://zh-hans.reactjs.org/docs/composition-vs-inheritance.html)

当我们需要把一些组件看作是其他组件的特殊实例，比如 `WelcomeDialog` 可以说是 `Dialog` 的特殊实例。 React推荐通过组合来实现这一点。“特殊”组件可以通过 props 定制并渲染“一般”组件：

```jsx
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
    </FancyBorder>
  );
}

function WelcomeDialog() {
  return (
    <Dialog
      title="Welcome"
      message="Thank you for visiting our spacecraft!" />
  );
}
```

### ES6：类和继承的实现原理

[见这篇](https://cloud.tencent.com/developer/article/1500264)。

### 作用域

作用域决定了代码区块中变量和其他资源的可访问性。

**词法作用域**，也叫静态**作用域**，它的**作用域**是指在**词法**分析阶段就确定了，不会改变。 **动态作用域**是在运行时根据程序的流程信息来**动态**确定的，而不是在写代码时进行静态确定的。

JavaScript是词法（静态）作用域的语言。

**ES6 之前 JavaScript 没有块级作用域,只有全局作用域和函数作用域**。ES6的到来，为我们提供了“块级作用域”,可通过新增命令let和const来体现。

来看一个例子，

```javascript
// ES6以前
var x = 10;
function fn() {
  console.log(x);
}
function show(f) {
  var x = 20;
  return (function () {
    f();
  })();
}
show(fn); // 10，词法作用域
```

以上例子说明，函数的作用域链在定义时（词法分析阶段）就确定好了。

### 作用域与执行上下文

作用域和执行上下文是两个截然不同的概念。

JavaScript属于解释型语言，JavaScript的执行分为：解释和执行两个阶段,这两个阶段所做的事并不一样：

- 解释阶段：词法分析，语法分析，作用域规则确定；
- 执行阶段：创建执行上下文，执行函数代码，垃圾回收；

注意，`call`、`apply`、`bind`等方法改变的都是执行阶段时的执行上下文。

JavaScript遇到函数执行的时候，就会创建一个执行上下文

### 执行环境

某个执行环境中的所有代码执行完毕后，该环节被销毁，保存在其中的所有变量和函数定义也随之销毁，全局执行环境直到应用程序退出——例如关闭网页/浏览器时才会被销毁。

- 全局执行环境：最外围的一个执行环境。根据ECMAScript实现所在的宿主环境不同，表示执行环境的对象也不一样。在web浏览器中，全局执行环境被认为是window对象，因此所有全局变量和函数都是作为window对象的属性和方法创建的。
- 函数执行环境：每个函数都有自己的执行环境。当**执行**流进入一个函数时，函数的环境就会被推入一个环境栈中，而在函数执行之后，栈将其环境弹出，把控制权返回给之前的执行环境。ECMAScript程序中的执行流正由这个方便的机制控制着。

**执行环境与执行上下文几乎是一个意思，只是在叙述不同内容时的不同表达方式。**它们都意味着堆栈变化，以及`this`指向的改变。

### 作用域链

除了环境记录外，还需要保存对外部词法环境的引用。这里使用一个指向变量对象的指针列表——作用域链来记录。作用域链本质上是一个列表对象，线性、有次序的保存着变量对象的引用。作用域链的作用是保证对执行环境有权访问的所有变量和函数的有序访问。作用域链的前端，始终都是当前执行的代码所在环境的变量对象。如果这个环境是函数，则将其“活动对象”作为变量对象（活动对象最开始包含一个对象，即arguments）。作用域链的下一个变量对象来自包含（外部）环境，层层延续到全局执行环境，全局执行环境的变量对象始终都是作用域链中的最后一个对象。**标识符解析**是沿着作用域链一级一级地搜索标识符的过程，始终从作用域链的前端开始，然后逐级往后回溯，直到找到为止，如果找不到，通常会报错。另外，在js中， **内部环境可以通过作用域链访问所有的外部环境，但外部环境不能访问内部环境中的任何变量和函数**。因此，作用域链的线性和有序也体现在此，每个环境都可以向上搜索作用域链以查找变量和函数名，但不能往下。

首先，变量对象和作用域链我们能理解：作用域链是一个有次序保存相应执行环境的变量对象的引用的列表。其实，在调用函数时，会为函数创建一个执行环境压入栈顶，同时会创建一个预先包含全局变量对象的作用域链，这个作用域链被保存在内部的 **[[Scope]]属性** 中。再执行时，会通过 **复制函数的[[Scope]]属性中的对象构建起执行环境的作用域链，此后，又有一个活动对象被创建并被推入这个执行环境作用域链的顶端** 。无论什么时候在函数中访问一个变量时，都会从作用域链中搜索具有相应名字的变量。

### 闭包

> 闭包是函数和声明该函数的词法环境的组合。
>
> ——MDN
>
> 闭包就是由函数创造的一个词法作用域，里面创建的变量被引用后，可以在这个词法环境之外自由使用。闭包通常用来创建内部变量，使得这些变量不能被外部随意修改，同时又可以通过指定的函数接口来操作。
>
> —— [知乎高赞](https://www.zhihu.com/question/19554716)
>
> 闭包是 JavaScript 一个非常重要的特性，这意味着当前作用域**总是**能够访问外部作用域中的变量。 在ES6以前， [函数](https://bonsaiden.github.io/JavaScript-Garden/zh/#function.scopes)是 JavaScript 中唯一拥有自身作用域的结构，因此闭包的创建依赖于函数。
>
> —— [JS秘密花园](https://bonsaiden.github.io/JavaScript-Garden/zh/#function.closures)


一般来讲，当函数执行完毕后，局部活动对象就会被销毁，内存中仅保存全局作用域（全局执行环境的变量对象）。**闭包的情况有所不同：在另一个函数内部定义的函数会将包含函数（即外部函数）的活动对象添加到它的作用域链中。**

一个闭包的例子：

```javascript
function createComparisonFunction(propertyName) {
    return function(object1, object2){
        var value1 = object1[propertyName];
        var value2 = object2[propertyName];
        if (value1 < value2){
            return -1;
        } else if (value1 > value2){
            return 1;
        } else {
            return 0;
        }
    };
}
// 创建函数
var compareNames = createComparisonFuncion('name');
// 调用函数
var result = compareNames({name: 'zs'},{name: 'ls'});
// 解除对匿名函数的引用（以便释放内存）
compareNames = null;
```

闭包的引用一定要及时释放，否则会造成内存泄漏。（GC程序并不能自动清除）

**闭包的应用场景**

- 模拟私有变量

  ```javascript
  function Counter(start) {
      var count = start;
      return {
          increment: function() {
              count++;
          },
  
          get: function() {
              return count;
          }
      }
  }
  
  var foo = Counter(4);
  foo.increment();
  foo.get(); // 5
  ```

  因为 JavaScript 中不可以对作用域进行引用或赋值，因此没有办法在外部访问 `count` 变量。 唯一的途径就是通过那两个闭包。

- 处理循环的异步操作

  ```javascript
  for(var i = 0; i < 10; i++) {
      setTimeout(function() {
          console.log(i);  
      }, 1000);
  }
  ```

  上面的代码不会输出数字 `0` 到 `9`，而是会输出数字 `10` 十次。当 `console.log` 被调用的时候，*匿名*函数保持对外部变量 `i` 的引用，此时 `for`循环已经结束， `i` 的值被修改成了 `10`。

  为了正确的获得循环序号，最好使用 [匿名包装器](https://bonsaiden.github.io/JavaScript-Garden/zh/#function.scopes)（译者注：其实就是我们通常说的自执行匿名函数）。

  ```javascript
  for(var i = 0; i < 10; i++) {
      (function(e) {
          setTimeout(function() {
              console.log(e);  
          }, 1000);
      })(i);
  }
  ```

  外部的匿名函数会立即执行，并把 `i` 作为它的参数，此时函数内 `e` 变量就拥有了 `i` 的一个拷贝。当传递给 `setTimeout` 的匿名函数执行时，它就拥有了对 `e` 的引用，而这个值是**不会**被循环改变的。

### JavaScript的this指向

JavaScript 语言之所以有`this`的设计，跟内存里面的数据结构有关系。这样的结构是很清晰的，问题在于属性的值可能是一个函数。这时，引擎会将函数单独保存在内存中，然后再将函数的地址赋值给`foo`属性的`value`属性。

```javascript
{
  foo: {
    [[value]]: 5 // 如果foo是一个函数，这里的value就是函数的地址
    [[writable]]: true
    [[enumerable]]: true
    [[configurable]]: true
  }
}
```

由于函数是一个单独的值，所以它可以在不同的环境（上下文）执行。现在问题就来了，由于函数可以在不同的运行环境执行，所以需要有一种机制，能够在函数体内部获得当前的运行环境（context）。所以，`this`就出现了，它的设计目的就是在函数体内部，指代函数当前的运行环境。

> —— 阮一峰老师的[博客](https://www.ruanyifeng.com/blog/2018/06/javascript-this.html)。

### JavaScript的内存管理（堆栈溢出及内存泄漏）

- 理解JavaScript的内存分区；
- 什么是内存泄漏，JS内存泄漏的几种常见场景；
- 垃圾收集算法

### JavaScript模块化进化过程

**全局function模式**

```javascript
function m1(){
  //...
}
function m2(){
  //...
}
```

- 作用: 将不同的功能封装成不同的全局函数；
- 问题: 污染全局命名空间, 容易引起命名冲突或数据不安全，而且模块成员之间看不出直接关系；

**namespace模式 : 简单对象封装**

```javascript
let myModule = {
  data: 'www.baidu.com',
  foo() {
    console.log(`foo() ${this.data}`)
  },
  bar() {
    console.log(`bar() ${this.data}`)
  }
}
myModule.data = 'other data' //能直接修改模块内部的数据
myModule.foo() // foo() other data
```

- 作用: 减少了全局变量，解决命名冲突；
- 问题: 数据不安全(外部可以直接修改模块内部的数据)；

**IIFE模式：匿名函数自调用（闭包）**

```javascript
// module.js
(function (window) {
  let data = "www.baidu.com";

  function foo() {
    console.log(`foo() ${data}`);
  }
  function otherFun() {
    console.log("otherFun()");
  }

  // 提供接口
  window.myModule = { foo, bar }; //ES6写法
})(window);

```

```html
// index.html文件
<script type="text/javascript" src="module.js"></script>
<script type="text/javascript">
    myModule.foo()
    console.log(myModule.data) //undefined 不能访问模块内部数据
    myModule.data = 'xxxx'
    myModule.foo() //没有改变
</script>
```

- 作用：实现了数据私有；
- 问题：无法解决模块依赖；

**IIFE模式增强 : 引入依赖**

这就是JavaScript现代模块实现的基石；

```javascript
// module.js
(function (window, $) {
  let data = "www.baidu.com";

  function foo() {
    console.log(`foo() ${data}`);
    $("body").css("background", "red");
  }
  function bar() {
    console.log(`bar() ${data}`);
    otherFun();
  }
  function otherFun() {
    //内部私有的函数
    console.log("otherFun()");
  }
  //暴露行为
  window.myModule = { foo, bar };
})(window, jQuery);
```

```html
 // index.html文件
  <!-- 引入的js必须有一定顺序 -->
  <script type="text/javascript" src="jquery-1.10.1.js"></script>
  <script type="text/javascript" src="module.js"></script>
  <script type="text/javascript">
    myModule.foo()
  </script>
```

- 作用：有依赖的有序引入保证了模块的独立性，还使得模块之间的依赖关系变得明显；
- 问题：请求过多，依赖模糊，难以维护；

**CommonJS**

特点：

1. 所有代码都运行在模块作用域，不会污染全局作用域。
2. 模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就被缓存了，以后再加载，就直接读取缓存结果。要想让模块再次运行，必须清除缓存。
3. 模块加载的顺序，按照其在代码中出现的顺序。

- 暴露模块：`module.exports = value`或`exports.xxx = value`；
- 引入模块：`require(xxx)`,如果是第三方模块，xxx为模块名；如果是自定义模块，xxx为模块文件路径；

CommonJS规范规定，每个模块内部，module变量代表当前模块。这个变量是一个对象，它的exports属性（即module.exports）是对外的接口。加载某个模块，其实是加载该模块的module.exports属性。

**CommonJS模块的加载机制是，输入的是被输出的值的拷贝。也就是说，一旦输出一个值，模块内部的变化就影响不到这个值**。

**ES6模块化**

ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。

export命令用于规定模块的对外接口，import命令用于输入其他模块提供的功能。

- 暴露模块：`export`；
- 引入模块：`import`；

模块的暴露和引入有多种语法，详情见[ES6入门手册](https://es6.ruanyifeng.com/#docs/module)。

### JavaScript宏任务和微任务

宏任务和微任务分别维护一个队列，均采用先进先出的策略进行执行，同步执行的任务都在宏任务上执行。

具体的操作步骤如下：

- 从宏任务的头部取出一个任务执行；
- 执行过程中若遇到微任务则将其添加到微任务的队列中；
- 宏任务执行完毕后，微任务的队列中是否存在任务，若存在，则挨个儿出去执行，直到执行完毕，如果在执行微队列任务的过程中，又产生了`microtask`，那么会加入整个队列的队尾，也会在当前的周期中执行；
- GUI 渲染；
- 回到步骤 1，直到宏任务执行完毕；

前 4 步构成了一个事件的循环检测机制，即我们所称的（浏览器环境下的） EventLoop。

**JavaScript在浏览器环境下和Node环境下对宏任务、微任务的执行机制是类似的，都是对宏任务进行事件循环，在每次宏任务间隔循环执行所有的微任务，也就是上面的步骤1到4。**

> macrotask 和 microtask 这两个概念，表示**异步任务的两种分类**。在挂起任务时，JS引擎会将所有任务按照类别分到这两个队列中，首先在 macrotask 的队列（这个队列也被叫做 task queue）中取出第一个任务，执行完毕后取出 microtask 队列中的所有任务顺序执行；之后再取 macrotask 任务，周而复始，直至两个队列的任务都取完。
>
> 两个类别的具体分类如下：
>
> - **macro-task:** script（整体代码）, `setTimeout`, `setInterval`, `setImmediate`, I/O, UI rendering
> - **micro-task:** `process.nextTick`, `Promises`（这里指浏览器实现的原生 Promise）, `Object.observe`, `MutationObserver`

### Node的事件循环

![nodejsloop](https://cescdf.com/image/nodejseventloop.png)

- timers: 这个阶段执行定时器队列中的回调如  setTimeout() 和  setInterval()。
- I/O callbacks: 这个阶段执行几乎所有的回调。但是不包括 close 事件，定时器和 setImmediate()的回调。
- idle, prepare: 这个阶段仅在内部使用，可以不必理会。
- poll: 等待新的 I/O 事件，node 在一些特殊情况下会阻塞在这里。
- check: setImmediate()的回调会在这个阶段执行。
- close callbacks: 例如 socket.on('close', ...)这种 close 事件的回调。

**当个 v8 引擎将 js 代码解析后传入 libuv 引擎后，循环首先进入 poll 阶段。**

在给定的阶段中，无论何时调用`process.nextTick()`，传递给`process.nextTick()`的所有回调都将在事件循环继续之前解析。这可能会造成一些不好的情况，因为它允许您通过递归`process.nextTick()`调用来“饿死”I/O，从而阻止事件循环到达轮询阶段。

### 浏览器的事件循环

要理解浏览器的事件循环，先要理解DOM和WebAPI；当我们有了可以执行JS的引擎，但是我们的目标是`构建用户界面`，而传统的前端用户界面是基于DOM构建的，因此我们需要引入DOM。DOM是`文档对象模型`，其提供了一系列JS可以直接调用的接口，理论上其可以提供其他语言的接口，而不仅仅是JS。 而且除了DOM接口可以给JS调用，浏览器还提供了一些WEB API。 DOM也好，WEB API也好，本质上和JS没有什么关系，完全不一回事。JS对应的ECMA规范，V8用来实现ECMA规范，其他的它不管。 这也是JS引擎和JS执行环境的区别，V8是JS引擎，用来执行JS代码，浏览器和Node是JS执行环境，其提供一些JS可以调用的API即`JS bindings`。

![liulanqi](https://cescdf.com/image/liulanqi.jpeg)

由于浏览器的存在，现在JS可以操作DOM和WEB API了，看起来是可以构建用户界面啦。 有一点需要提前讲清楚，V8只有栈和堆，其他诸如事件循环，DOM，WEB API它一概不知。原因前面其实已经讲过了，因为V8只负责JS代码的编译执行，你给V8一段JS代码，它就从头到尾一口气执行下去，中间不会停止。

可以看出，浏览器是一个不断循环的帧环境，它不仅要处理JavaScript（仅仅是V8的工作），还要处理样式的计算、回流重绘、合成，可以参考这篇[《浏览器的帧原理》](https://mp.weixin.qq.com/s/aHFWQGH94NkxHKCro8eW9w)。

![browser-frame](https://cescdf.com/image/browser-frame.jpeg)

我会单独开一篇浏览器原理来详细介绍，这里点到为止。

### setTimeout和setInterval

`setTimeout()`的参数如下，

```javascript
var timeoutID = scope.setTimeout(function[, delay, arg1, arg2, ...]);
```

返回值`timeoutID`是一个正整数，表示定时器的编号。这个值可以传递给`clearTimeout()`。

同样的，`setInterval()`有如下参数，

```javascript
var intervalID = scope.setInterval(func, delay, [arg1, arg2, ...]);
```

此返回值`intervalID`也是一个非零数值，用来标识通过`setInterval()`创建的计时器，这个值可以用来作为`clearInterval()`的参数来清除对应的计时器 。

**使用setTimeout实现setInterval**

```javascript
function mySetInterval(func, millisec) {
  if (typeof func !== "function") {
    throw new TypeError("Callback must be a function.");
  }
  function fn() {
    func();
    setTimeout(() => fn(), millisec);
  }
  setTimeout(fn, millisec);
}

mySetInterval(() => console.log(new Date()), 1000);
```

基于上面的思路，可以继续实现 `clearInterval()`，大概思路是得到内部`setTimeout`返回的`timeid`，这里不细讲，详情参考[掘金博客](https://juejin.cn/post/6844903839934447629)。

### JavaScript的正则表达式

正则表达式通常被用来检索、替换那些符合某个模式(规则)的文本。

> 在 JavaScript中，正则表达式也是对象。这些模式被用于 `RegExp` 的 `exec` 和 `test` 方法, 以及 `String` 的 `match`、`matchAll`、`replace`、`search` 和 `split` 方法。
>
> —— MDN

创建正则表达式的两种方式，

```javascript
// 方法一：两个斜杠
var re = /ab+c/;
// 方法二：内置引用类型
var re = new RegExp("ab+c");
```

正则表达式对象自带两个常用方法，

- `test()` 方法执行一个检索，用来查看正则表达式与指定的字符串是否匹配，返回 `true` 或 `false`；
- `exec() `方法在一个指定字符串中执行一个搜索匹配，返回一个结果数组或`null`；

JS正则表达式的一些常用技巧，

| 字符               | 含义                                                         |
| :----------------- | ------------------------------------------------------------ |
| \                  | 转义字符                                                     |
| ^，$               | 匹配输入的开始，匹配输入的结束                               |
| *，+，？           | 匹配前一个表达式0次或多次，1次或多次，0次或1次               |
| {n}，{n, }，{a, b} | 匹配前一个表达式恰好n次，至少n次，a到b次                     |
| .                  | 匹配任意单个字符（除了换行符）                               |
| [xyz]，\[^xyz\]    | 匹配方括号中的任意字符，匹配除了方括号的任意字符（注意1.内部都可以用`-`连接表示范围2.只能匹配一次） |
| \d，\D，\s，\S     | 匹配一个数字（等价于[0-9]），匹配非数字，匹配空白，匹配非空白 |

来看两个常用的正则表达式，

```javascript
// 匹配 IP 地址
const ipReg = /((2[0-4]\d|25[0-5]|[01]?\d\d?)\.){3}(2[0-4]\d|25[0-5]|[01]?\d\d?)/;

// 匹配电子邮箱
const mailReg = /([a-z0-9_\.-]+)@([\da-z\.-]+)\.([a-z\.]{2,6})/;
```

对于IP地址，我们分段来看。

先看第一段 `"(2[0-4]\d|25[0-5]|[01]?\d\d?)\.)"`，这里用 "|" 分割了三种匹配情况。

第一种是 `"2[0-4]\d"`，三位数字，第一位是 2，第二位是 0 - 4 之间，第三位是任意数字。

第二种是 `"25[0-5]"`，三位数字，第一位是 2，第二位是 5，第三位是 0 - 5 之间。

第三种是 `"[01]?\d\d?"`，第一位是 0 或者 1，匹配零次或一次，第二位和第三位是任意数字，第三位数字匹配零次或一次，也就是一位数，两位数，三位数都可能满足这种情况。

第一段末尾是 `"\."` 作为分隔符。

再来看第二段 `"{3}"`，这个表示前面的子表达式重复三次，也就是 IP 地址的前三个字节。

再看第三段` "(2[0-4]\d|25[0-5]|[01]?\d\d?)"`，和第一段是一个意思。

对于邮箱地址的匹配，也是分段来看。

第一段 `"([a-z0-9_\.-]+)"`，表示匹配 a-z 范围内的字母，0-9 范围的数字，以及 "_"，"."，"-" 三个字符，"+" 表示至少有一个字符。

第二段 `"@"` 表示匹配 "@" 字符。

第三段 `"([\da-z\.-]+)"`，"\d" 表示匹配任意数字，a-z 范围的字母，"."，"-" 两个字符，"+" 至少有一个字符。

第四段 `"\."` 表示匹配 "." 字符。

第五段 `"([a-z\.]{2,6})"`，表示匹配 a-z 范围的字母，"." 字符，"{2,6}" 表示至少 2 个字符，至多 6 个字符。

### JavaSritp异常捕获

**`try...catch`**语句标记要尝试的语句块，并指定一个出现异常时抛出的响应。

`try`语句包含了由一个或者多个语句组成的`try`块, 和至少一个`catch`块或者一个`finally`块的其中一个，或者两个兼有， 下面是三种形式的`try`声明：

1. `try...catch`
2. `try...finally`
3. `try...catch...finally`

**注意，如果`finally`块返回一个值，该值会是整个`try-catch-finally`流程的返回值，不管在`try`和`catch`块中语句返回了什么。**

### JavaScript浅拷贝、深拷贝

先来看浅拷贝和深拷贝的定义，

- 浅拷贝：创建一个新对象，这个对象有着原始对象属性值的一份精确拷贝。如果属性是基本类型，拷贝的就是基本类型的值，如果属性是引用类型，拷贝的就是内存地址 ，所以如果其中一个对象改变了这个地址，就会影响到另一个对象。
- 深拷贝：将一个对象从内存中完整的拷贝一份出来,从堆内存中开辟一个新的区域存放新对象,且修改新对象不会影响原对象

**简单的浅拷贝**

```javascript
function clone(target) {
    let cloneTarget = {};
    for (const key in target) {
        cloneTarget[key] = target[key];
    }
    return cloneTarget;
};
```

没什么好说的。

**深拷贝：低级版**

```javascript
JSON.parse(JSON.stringify());
```

**深拷贝：简单递归**

```javascript
function clone(target) {
    if (typeof target === 'object') {
        let cloneTarget = {};
        for (const key in target) {
            cloneTarget[key] = clone(target[key]);
        }
        return cloneTarget;
    } else {
        return target;
    }
};
```

上面两种写法非常简单，而且可以应对大部分的应用场景，但是它还是有很大缺陷的，比如拷贝其他引用类型、拷贝函数、循环引用等情况。

**深拷贝：中级版**

```javascript
function clone(target, map = new WeakMap()) {
    if (typeof target === 'object') {
        let cloneTarget = Array.isArray(target) ? [] : {};
        if (map.get(target)) {
            return map.get(target);
        }
        map.set(target, cloneTarget);
        for (const key in target) {
            cloneTarget[key] = clone(target[key], map);
        }
        return cloneTarget;
    } else {
        return target;
    }
};
```

上面的最终版涉及到几个知识点：

- WeakMap：Map的作用是防止循环引用，WeakMap储存对象的弱引用，及时释放内存；
- Array：兼容了数组的情况；

这还不是全部，还要继续处理克隆其他类型，比如Map，Set甚至Function的情况，还可以用不同的遍历方式提高效率，完整版见[《如何写出一个惊艳面试官的深拷贝》](http://www.conardli.top/blog/article/JS%E8%BF%9B%E9%98%B6/%E5%A6%82%E4%BD%95%E5%86%99%E5%87%BA%E4%B8%80%E4%B8%AA%E6%83%8A%E8%89%B3%E9%9D%A2%E8%AF%95%E5%AE%98%E7%9A%84%E6%B7%B1%E6%8B%B7%E8%B4%9D.html#%E5%85%8B%E9%9A%86%E5%87%BD%E6%95%B0)。

### 小结

注意，所有关于**手动实现JavaScript某一个函数或者功能**的内容已全部转移到新的博客，详情见博客主页。

