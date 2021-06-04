# JavaScript函数专题与this指向

# JavaScript Topic of Function

## 2021 年 5 月 25 日

## May 25 2021



> 参考[JavaScript专题系列文章](https://github.com/mqyqingfeng/Blog/tree/master/articles/%E4%B8%93%E9%A2%98%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0)



### 柯里化

柯里化是一种将使用多个参数的一个函数转换成一系列使用一个（或更少）参数的函数的技术。

一个柯里化实际用途的例子，

```javascript
// 示意而已
function ajax(type, url, data) {
    var xhr = new XMLHttpRequest();
    xhr.open(type, url, true);
    xhr.send(data);
}

// 虽然 ajax 这个函数非常通用，但在重复调用的时候参数冗余
ajax('POST', 'www.test.com', "name=kevin")
ajax('POST', 'www.test2.com', "name=kevin")
ajax('POST', 'www.test3.com', "name=kevin")

// 利用 curry
var ajaxCurry = curry(ajax);

// 以 POST 类型请求数据
var post = ajaxCurry('POST');
post('www.test.com', "name=kevin");

// 以 POST 类型请求来自于 www.test.com 的数据
var postFromTest = post('www.test.com');
postFromTest("name=kevin");
```

**curry 的这种用途可以理解为：参数复用。本质上是降低通用性，提高适用性。**

一个经常会看到的 curry 函数的实现为：

```javascript
// 第一版
var curry = function (fn) {
  	// 这一行可以用es6的写法：args = [...arguments];
    var args = [].slice.call(arguments, 1);
    return function() {
        var newArgs = args.concat([].slice.call(arguments));
        return fn.apply(this, newArgs);
    };
};
```

还不够，来看下面这个，

```javascript
// 第二
function curry(fn, args) {
    length = fn.length;

    args = args || [];

    return function() {

        var _args = args.slice(0),

            arg, i;

        for (i = 0; i < arguments.length; i++) {

            arg = arguments[i];

            _args.push(arg);

        }
        if (_args.length < length) {
            return curry.call(this, fn, _args);
        }
        else {
            return fn.apply(this, _args);
        }
    }
}


var fn = curry(function(a, b, c) {
    console.log([a, b, c]);
});

fn("a", "b", "c") // ["a", "b", "c"]
fn("a", "b")("c") // ["a", "b", "c"]
fn("a")("b")("c") // ["a", "b", "c"]
fn("a")("b", "c") // ["a", "b", "c"]
```

### 偏函数

柯里化是将一个多参数函数转换成多个单参数函数，也就是将一个 n 元函数转换成 n 个一元函数。

局部应用则是固定一个函数的一个或者多个参数，也就是将一个 n 元函数转换成一个 n - x 元函数。

如果说两者有什么关系的话，引用 [functional-programming-jargon](https://github.com/hemanth/functional-programming-jargon#partial-application) 中的描述就是：

> Curried functions are automatically partially applied.

最简单的偏函数是使用`bind`固定函数的第一个参数，

```javascript
function add(a, b) {
    return a + b;
}
var addOne = add.bind(null, 1);
addOne(2) // 3
```

然而使用`bind`会改变`this`指向，所以我们尝试写一个不改变`this`指向的偏函数方法，

```javascript
function partial(fn) {
    var args = [].slice.call(arguments, 1);
    return function() {
        var newArgs = args.concat([].slice.call(arguments));
        return fn.apply(this, newArgs);
    };
};
```

**Bang，竟然完全和柯里化的代码一模一样！**

更复杂的便函数可以使用占位符，如下，

```javascript
// 第二版
var _ = {};

function partial(fn) {
    var args = [].slice.call(arguments, 1);
    return function() {
        var position = 0, len = args.length;
        for(var i = 0; i < len; i++) {
            args[i] = args[i] === _ ? arguments[position++] : args[i]
        }
        while(position < arguments.length) args.push(argumetns[position++]);
        return fn.apply(this, args);
    };
};
```

有兴趣的话可以自己验证。

### 惰性函数

我们现在需要写一个 foo 函数，**这个函数返回首次调用时的 Date 对象，注意是首次**。

现在有如下解决方法：

1. 普通方法：问题有两个，一是污染了全局变量，二是每次调用 foo 的时候都需要进行一次判断。
2. 闭包：没有解决调用时都必须进行一次判断的问题。
3. 函数对象属性：依旧没有解决调用时都必须进行一次判断的问题。
4. 惰性函数

来看惰性函数的实现，

```javascript
var foo = function() {
    var t = new Date();
    foo = function() {
        return t;
    };
    return foo();
};

```

不错，惰性函数就是解决每次都要进行判断的这个问题，解决原理很简单，重写函数。

> 参考资料：[Lazy Function Definition Pattern](http://peter.michaux.ca/articles/lazy-function-definition-pattern)。

### 函数组合

略。

### 函数记忆

略。

### 函数递归之尾调用

尾调用，是指函数内部的最后一个动作是函数调用。该调用的返回值，直接返回给函数。

举个例子：

```javascript
// 尾调用
function f(x){
    return g(x);
}
```

然而

```javascript
// 非尾调用
function f(x){
    return g(x) + 1;
}
```

并不是尾调用，因为 `g(x)` 的返回值还需要跟 1 进行计算后，`f(x)`才会返回值。

两者又有什么区别呢？答案就是执行上下文栈的变化不一样。

为了模拟执行上下文栈的行为，让我们定义执行上下文栈是一个数组：

```
ECStack = [];
```

我们模拟下第一个**尾调用函数**执行时的执行上下文栈变化：

```
// 伪代码
ECStack.push(<f> functionContext);

ECStack.pop();

ECStack.push(<g> functionContext);

ECStack.pop();
```

我们再来模拟一下第二个**非尾调用函数**执行时的执行上下文栈变化：

```
ECStack.push(<f> functionContext);

ECStack.push(<g> functionContext);

ECStack.pop();

ECStack.pop();
```

**也就是说，尾调用函数执行时，虽然也调用了一个函数，但是因为原来的的函数执行完毕，执行上下文会被弹出，然而非尾调用函数，就会创建多个执行上下文压入执行上下文栈。**

函数调用自身，称为递归。如果尾调用自身，就称为尾递归。

所以我们只用把阶乘函数改造成一个尾递归形式，就可以避免创建那么多的执行上下文。

阶乘函数的尾调用优化如下，

```javascript
function factorial(n, res) {
    if (n == 1) return res;
    return factorial2(n - 1, n * res)
}

console.log(factorial(4, 1)) // 24
```

还可以继续柯里化，

```javascript
var newFactorial = curry(factorial, _, 1)

newFactorial(5) // 24
```

> 思考：斐波那契数列怎样进行尾调用优化？

### ES6箭头函数

> 箭头函数表达式的语法比函数表达式更简洁，并且没有自己的this，arguments，super或new.target。箭头函数表达式更适用于那些本来需要匿名函数的地方，并且它不能用作构造函数。
>
> —— MDN

箭头函数不会创建自己的`this,它只会从自己的作用域链的上一层继承this`。

在下面的代码中，传递给`setInterval`的函数内的`this`与封闭函数中的`this`值相同：

```javascript
function Person(){
  this.age = 0;

  setInterval(() => {
    this.age++; // |this| 正确地指向 p 实例
  }, 1000);
}

var p = new Person();
```

### this

**当JavaScript代码执行一段可执行代码(executable code)时，会创建对应的执行上下文(execution context)。**

对于每个执行上下文，都有三个重要属性

- 变量对象(Variable object，VO)
- 作用域链(Scope chain)
- this

这一节重点讲讲 this。

>因为我们要从 ECMASciript5 规范开始讲起。
>
>先奉上 ECMAScript 5.1 规范地址：
>
>英文版：http://es5.github.io/#x15.1
>
>中文版：http://yanhaijing.com/es5/#115

先来看JS的类型，

> ECMAScript 的类型分为语言类型和规范类型。
>
> ECMAScript 语言类型是开发者直接使用 ECMAScript 可以操作的。其实就是我们常说的Undefined, Null, Boolean, String, Number, 和 Object。
>
> 而规范类型相当于 meta-values，是用来用算法描述 ECMAScript 语言结构和 ECMAScript 语言类型的。规范类型包括：Reference, List, Completion, Property Descriptor, Property Identifier, Lexical Environment, 和 Environment Record。

**其中的 Reference 类型，与 this 的指向有着密切的关联。**

Reference 类型是用来解释诸如 delete、typeof 以及赋值等操作行为的。

> Reference 是一个 Specification Type，也就是 “只存在于规范里的抽象类型”。它们是为了更好地描述语言的底层行为逻辑才存在的，但并不存在于实际的 js 代码中。
>
> —— Evan You

Reference由三个组成部分，分别是：

- base value：属性所在的对象或者就是 EnvironmentRecord，它的值只可能是 undefined, an Object, a Boolean, a String, a Number, or an environment record 其中的一种；
- referenced name：属性的名称；
- strict reference

一个例子，

```javascript
var foo = 1;

// 对应的Reference是：
var fooReference = {
    base: EnvironmentRecord,
    name: 'foo',
    strict: false
};
```

每个Reference Type的有一些方法，比如`GetBase`和`IsPropertyReference`，

- GetBase(V). Returns the base value component of the reference V.
- IsPropertyReference(V). Returns true if either the base value is an object or HasPrimitiveBase(V) is true;

还有一个用于从 Reference 类型获取对应值的方法： `GetValue`。

```javascript
// 继续上面的例子
GetValue(fooReference) // 1;
```

**MemberExpression**

我们还要了解一下所谓的MemberExpression，

MemberExpression :

- PrimaryExpression // 原始表达式
- FunctionExpression // 函数定义表达式
- MemberExpression [ Expression ] // 属性访问表达式
- MemberExpression . IdentifierName // 属性访问表达式
- new MemberExpression Arguments // 对象创建表达式

```javascript
// 举个例子
function foo() {
    console.log(this)
}

foo(); // MemberExpression 是 foo

function foo() {
    return function() {
        console.log(this)
    }
}

foo()(); // MemberExpression 是 foo()

var foo = {
    bar: function () {
        return this;
    }
}

foo.bar(); // MemberExpression 是 foo.bar
```

所以，简单理解 MemberExpression 其实就是()左边的部分。

**重点来了，如何确定this的值呢**，

让我们描述一下：

1. 计算 MemberExpression 的结果赋值给 ref；

2. 判断 ref 是不是一个 Reference 类型；

   2.1 如果 ref 是 Reference，并且 IsPropertyReference(ref) 是 true, 那么 this 的值为 GetBase(ref)；

   2.2 如果 ref 是 Reference，并且 base value 值是 Environment Record, 那么this的值为 	ImplicitThisValue(ref)；

   2.3 如果 ref 不是 Reference，那么 this 的值为 undefined；

一些例子，

```javascript
var value = 1;

var foo = {
  value: 2,
  bar: function () {
    return this.value;
  }
}

//示例1
console.log(foo.bar()); // 2
// foo.bar返回一个Reference类型，且 IsPropertyReference(ref) 是 true，那么 this 的值为 GetBase(ref)
//示例2
console.log((foo.bar)()); // 2
// 忽略括号，实际上 () 并没有对 MemberExpression 进行计算
//示例3
console.log((foo.bar = foo.bar)()); // 1
// 有赋值操作符，会导致使用 GetValue，所以返回的值不是 Reference 类型，this指向undefined，而非严格模式下会转换成全局环境，所以this.value === 1
//示例4
console.log((false || foo.bar)()); // 1
// 同3
//示例5
console.log((foo.bar, foo.bar)()); // 1
// 同3
```

> 参考资料：[JavaScript深入之从ECMAScript规范解读this](https://github.com/mqyqingfeng/Blog/issues/7#)

