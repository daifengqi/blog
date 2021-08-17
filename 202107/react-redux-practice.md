# react-redux：最佳实践

# React-redux: Best Practice

## 2021 年 7 月 1 日

## Jul 1 2021

### 前言

如果你要用redux作为状态管理库，并且同时你还在用React作为主框架，那么你没有理由不用[react-redux](https://react-redux.js.org/)这个库，因为这才是最佳实践。

下面我们讲一讲使用这个库的流程和原理。

![react-redux](https://redux.js.org/assets/images/ReduxAsyncDataFlowDiagram-d97ff38a0f4da0f327163170ccc13e80.gif)

### 前置知识

在使用react-redux之前，先要理解redux用到的一些理念，比如action，纯函数，dispatch等，可以先看我[之前的博客](https://cescdf.com/blog/202104/redux-quick-notes)。

### 一切始于reducer & store

store是一种状态管理通用的，或者说flux理念中的概念，即**单一的状态树**，这个对象维护了当前的状态。在redux中，store的参数就来自于reducer，一个reducer就是一个函数。

来看一个简单的reducer，

```javascript
const reducer = (state = {}, action) => {
  switch (action.type) {
    case "SET":
      return action.state;
    default:
      return state;
  }
};

export default reducer;
```

**reducer就是一个主体为`switch`的函数**，除此之外，没有其他意思了。

### 从reducer到store

只需要一个函数，

```javascript
import { createStore } from "redux";
import reducer from './reducer'

export default createStore(reducer);
```

有时候，我们可能需要多个状态树，以对应不同的状态管理，这会让我们定义多个reducer，然后把它们结合起来，

```javascript
import { combineReducers } from 'redux';
import posts from './posts';
import auth from './auth';

const reducers = combineReducers({ posts, auth })
export default createStore(reducer);
```

这就成功定义了一个store。

我们可以直接通过store的三个API来使用redux，但正如一开始所说，**在react里使用redux的最佳实践是通过react-redux库提供的API来做**，这样可以结合react虚拟DOM根据状态自动刷新的优势，而省去了使用`store.subscribe`引来的种种问题，下面我们看看怎么做。

### 基于Context的全局注入

下面是[redux官方](https://react-redux.js.org/introduction/getting-started)提供的全局注入store的简单案例，

```javascript
import React from 'react'
import ReactDOM from 'react-dom'

import { Provider } from 'react-redux'
import store from './store'

import App from './App'

const rootElement = document.getElementById('root')
ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  rootElement
)
```

此处，我们从`react-redux`这个包引入了全局Provider，作为组件树最外层的包装，很容易理解。

如果我们dispatch的行为是异步（async）的，那么需要用到`thunk`中间件，来看一下怎么使用呢，

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import { createStore, applyMiddleware, compose } from 'redux';
import thunk from 'redux-thunk';

import reducers from './reducers';

import App from './App';
import './index.css';

const store = createStore(reducers, compose(applyMiddleware(thunk)));

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root'),
);
```

最关键的是这一行，

```js
const store = createStore(reducers, compose(applyMiddleware(thunk)));
```

这里通过reducer来创建store的时候，在第二个参数传入了`compose(applyMiddleware(thunk))`，于是引入了thunk中间件，这个中间件是直接从'redux-thunk'库引入的。

这样一来，我们全局注入的store就可以接受异步的dispatch了。

> 以上，我们完成了状态store和reducer的定义，下面就是如何在组件中通过dispatch一些actions来改变状态了。

### Hooks

> 参考[React-redux Hooks](https://react-redux.js.org/api/hooks)。

- useDispatch()

React-redux提供`useDispatch`这个hook来取代`store.dispatch`的操作。一方面，这可以让dispatch自行融入到React的state设计中，符合组件渲染的预期。另一方面， 因为没有了导入的直接可以使用的`store`对象，而是将状态作为全局Context注入最上层组件树，所以需要通过一个东西来影响`store`，**useDispatch扮演的就是这个角色**。

基本使用方式，

```javascript
import React from 'react'
import { useDispatch } from 'react-redux'

export const CounterComponent = ({ value }) => {
  const dispatch = useDispatch()

  return (
    <div>
      <span>{value}</span>
      <button onClick={() => dispatch({ type: 'increment-counter' })}>
        Increment counter
      </button>
    </div>
  )
}
```

来看，我们从`react-redux`库里引入了`useDispatch`这个钩子，然后通过钩子初始化一个函数`dispatch`，那之后，我们就可以像用`store.dispatch`一样，来直接通过`dispatch`派发action。

- useSelector()

```js
const result: any = useSelector(selector: Function, equalityFn?: Function)
```

useSelector是从store提取状态数据的Hook。顾名思义`selector`用于从state中选择对应的reducer（的对象）出来，例如，

```jsx
import React from 'react'
import { useSelector } from 'react-redux'

export const CounterComponent = () => {
  const counter = useSelector((state) => state.counter)
  return <div>{counter}</div>
}
```

这里直接选择了`state`的`counter` Reducer，同样，你还可以利用某些变量，比如props，

```jsx
export const TodoListItem = (props) => {
  const todo = useSelector((state) => state.todos[props.id])
  return <div>{todo.text}</div>
}
```

这里选择的是`state.todos[props.id]`。**注意，`selector`传入的是 combine 后的状态，提取的是对应Reducer的对象。**

- useStore()

最直接的，你可以用useStore()返回全局`store`并使用。注意，这个store是具有三个API的store对象，而不是像useSelector那样返回状态内的变量。

### React.memo

最后提一下React.memo和useSelector在渲染优化上的区别。

默认情况下，在调度操作后运行selector函数时，useSelector()将对所选值进行引用相等比较，并且仅当所选值发生更改时，才会导致组件重新呈现。但是，与`connect()`不同的是，useSelector()不会因为组件的父级重新渲染而阻止组件重新渲染，即使组件的props没有更改。

**要避免由于父组件重新渲染带来的子组件渲染，应该用`React.memo`。**React官方的示例如下，

```jsx
function MyComponent(props) {
  /* render using props */
}
function areEqual(prevProps, nextProps) {
  /*
  return true if passing nextProps to render would return
  the same result as passing prevProps to render,
  otherwise return false
  */
}
export default React.memo(MyComponent, areEqual);
```

React.memo将包装一个组件，返回一个所谓的高阶组件。

### useContext

最后的最后，我们来看看react-redux实现这一切的原理的起点，就是React的Context机制。官方对useContext钩子给出了下面的示例，

```jsx
const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

const ThemeContext = React.createContext(themes.light);

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  );
}
```

不用多说了，其实就是类似的，使用useContext()钩子来获取来自Provider的全局Context。区别就在于没有提供dispatch和selector。

### useCallback和useMemo

这两个API都是做缓存，不同的是useCallback返回一个函数，而useMemo储存调用函数后的返回结果（对象），如下

```jsx
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);

onst memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

这和redux有什么联系呢？我们可以将dispatch包装在useCallback内部，这样就不用在每次渲染组件的时候都去生成一次函数，而是会稳定下来，

```jsx
const incrementCounter = useCallback(
  () => dispatch({ type: 'increment-counter' })
  , [dispatch])
```

这里来说，只要传入Provider的全局store对象是不变的（通常来讲是这样的），那么dispatch也不会改变，于是这个函数会被一直缓存，当作dispatch使用。这就是一种performance optimization。