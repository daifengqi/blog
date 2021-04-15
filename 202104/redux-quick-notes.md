# Redux入门速查笔记

# Redux Quick Notes

## 2021 年 4月 8 日

## Apr 8 2021

---

### 引言

Redux是一个全局状态管理框架，其不可变状态树（Inmutable State Tree）可看作单个JavaScript对象，在每个时刻，Redux都只有一个确定的状态（current state）。虽然这个状态可以非常复杂，有时候内部会嵌套许多JavaScript对象，但是Redux的宗旨始终是明确的：即单一的状态管理树。

### Actions

State不可变的意思是不可以通过赋值直接修改状态的值。在Redux中，使用action是唯一的修改状态的方式。action对象通过包括一个type属性和一个payload属性。

- type：字符串，即action的名称或标志；
- payload：本次action需要携带的数据，可以包含多个变量。

Actions描述了状态的修改方式。

### 纯函数（pure function）

纯函数的输出完全牢固地依赖于输入的参数。概括地说，具有以下特征的函数可称为纯函数：

- 函数为相同的参数返回相同的结果。
- 它的执行没有副作用，也就是说，它不会改变输入数据。
- 没有局部和全局变量的作用域改变。
- 不依赖于任何外部状态，比如全局变量。

关于纯函数和函数式编程的讨论是一个很大的话题，详情可以参考这个[JSConf会议报告](https://www.youtube.com/watch?v=e-5obm1G_FY&t=1475s)，以及知乎上的这篇[反对函数式编程的政治正确](https://zhuanlan.zhihu.com/p/51563817)。

Redux中，对状态的修改必须使用纯函数。这里简单地说明一下为什么：

1. Redux接受给定的状态（对象）并将其传递给循环中的每个reducer。如果有任何变化，它期望reducer提供一个全新的对象。如果没有任何变化，它也希望能将旧对象取回。Redux只是通过比较两个对象的内存位置来检查旧对象是否与新对象相同。因此，如果在reducer中改变旧对象的属性，“new state”和“old state”都将指向同一个对象。因此，Redux认为一切都没有改变！所以这行不通。
2. 为什么要这样设计？在JavaScript中，只有一种方法可以知道两个对象是否具有相同的属性，去深入递归地比较每个属性的引用是否相同，这样做是非常低效率的。Redux通过简单地检查新对象是否拥有的新的内存位置来判断状态是否改变了，这是一种相比而言更高效的设计思想。

到此，你应该明白了为什么以上的策略依赖于状态的修改必须使用纯函数的假设。

### Reducer

Reducer本质就是一个函数，需要要接受两个输入：旧的状态（old state）和派发的行为（dispatching action），然后返回一个新对象代表的新状态（new state）。

创建一个简单Reducer的例子如下：

```js
import { createStore } from Redux;

const counter = (state = 0, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1;
    case 'DECREMENT':
      return state - 1;
    default:
      return state;
  }
}

const store = createStore(counter);
```

### 三个重要方法

- getState()：返回现在的状态;
- dispatch()：派发行为对象;
- subscribe()：添加订阅者;

### 源码：实现createStore

如下。

```js
const createStore = (reducer) => {
  let state;
  let listeners = [];
  
  const getState = () => state;
  
  const dispatch = (action) => {
    state = reducer(state, action);
    listeners.forEach(listener => listener());
  };
  
  const subscribe = (listener) => {
    listeners.push(listener);
    return () => {
      listeners = listeners.filter(lis => lis!== listener);
    }
  }
  
  dispatch({});
  
  return {getState, dispatch, subscribe}
}
```

这便是一个简单的Redux框架。

### Array的典型纯函数和非纯函数方法

- 非纯函数方法：push, splice
- 纯函数方法：slice, concat, ...spread

### Object的典型纯函数和非纯函数操作

- 非纯函数操作：直接对属性赋值
- 纯函数操作：`Object.assign({}, old_state, changed_property)`、或者...spread

总之，灵活运用...spread特性来对数组和对象进行增、删、改是良好的纯函数编程习惯。

### Reducer Composition

要记住的是，Redux框架里的Reducer就是函数，也仅仅是函数。这意味着在JavaScript执行环境里，Reducer可以被当作一等对象拷贝、复制、作为参数传递和返回。在函数式编程的背景下，这个特性可以被充分利用，来解耦和组织你的代码架构。一个Reducer里可以包含多个在其他地方定义的Reducer，这样以来，就可以根据代码的逻辑，把功能实现在不同的函数里，然后通过某单个函数整合。

实际上，Reducer把这样的功能封装在[combineReducers](https://redux.js.org/api/combinereducers)这个函数里了，下面我们来看看如何实现一个combineReducers()。作为一个函数，它接受一串封装在对象里的reducers作为参数，并且返回单个Reducer：

```js
const combineReducers = (reducers) => {
  return (state = {}, action) => {
    return Object.keys(reducers).reduce(
      (nextState, key) => {
        nextState[key] = reducers[key](
        state[key],
        action
        );
        return nextState;
      },
      {}
    );
  };
};
```

这里的一个注意点是用了一个`reduce`函数，并传入一个空对象`{}`作为起始参数。

