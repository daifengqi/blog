# 手动造轮子之React全解析

# Build Your Own React

## 2021 年 5 月 18 日

## May 18 2021



### Reference

点击[这里](https://pomb.us/build-your-own-react/)。

### Review

**Let's start with 3 lines codes, to remove all React specific code and replace it with vanilla JavaScript.**

- first line: define a HTML **element** with JSX
- second line: get a node or **container** from DOM
- third line: **render the element into the container**

see below,

```JSX
const element = <h1 title="foo">Hello</h1>;
const container = document.getElementById("root");
ReactDOM.render(element, container);
```

JSX is transformed to JS by build tools like Babel. The transformation is usually simple: replace the code inside the tags with a call to `createElement`, passing the three things as parameters,

- the tag name
- the props
- **the children (many things can be children; think about it)**

```javascript
const element = React.createElement(
  "h1", // tag name
  {title: "foo"}, // props
  "Hello"
)

// it will be transferred to
const element = {
  type: "h1",
  props: {
    title: "foo",
    children: "Hello",
  }
}
```

The second line is already vanilla JS; then comes with the third line,

```javascript
// before
ReactDOM.render(element, container);

// after
// 1. create DOM node
const node = document.createElement(element.type);
node['title'] = element.props.title;
// 2. add text to node
const text = document.createTextNode("");
text["nodeValue"] = element.props.children;
// 3. append children
node.appendChild(text);
container.appendChild(node);
```

### Step 1: The `createElement` Function

Let's see another example of JSX,

```JSX
const element = (
	<div id="foo">
  	<a>bar</a>
    <b />
 	</div>
)
const container = document.getElementById("root");
ReactDOM.render(element, container);

// the JSX will be
const element = React.createElement(
	"div",
  {id: "foo"},
  React.createElement("a", null, "bar"),
  React.createElement("b")
)
```

As we remember, `element = type + props`, so the only thing our function needs to do is to create that object.

Let's write the `createElement` function,

```javascript
// input 'children' must be array
function createElement(type, props, ...children) {
  return { // note that we return an object
    type,
    props: {
      ...props,
      children,
    },
  }
}
```

we want to simplify our code with a `createTextElement` function to wrap primitive values such as number and string, see

```javascript
function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: [],
    },
  }
}
```

Note that React doesn't do this, we do because we prefer simple code here.

### Step 2: The `render` Function

> `render`阶段就是把`fiber`挂载到真实DOM上。

Next, we will replace this,

```javascript
ReactDOM.render(element, container);
```

For now, we only care about **adding**, and we will cover **updating** and **deleting** later.

We start by creating the DOM node with `type` attribute and append it to DOM tree.

```javascript
function render(element, container) {
  // text or tag
  const dom = element.type === 'TEXT_ELEMENT'
  			? document.createTextNode("")
        : document.createElement(element.type);
  // assign each prop to the node
  Object.keys(element.props)
    .filter( (key) => key !== "children")
  	.forEach(name => {
    dom[name] = element.props[name];
  })
  // recursively do the same for children
  element.props.children.forEach(child => 
  	render(child, dom);
  );
  
  container.appendChild(dom);
}
```

We now have the codes that can render JSX to the DOM.

Note that we have a problem here: **the recursive part won't stop until completing the whole element tree.** Sometimes it is not necessary.

### Step 3: Concurrent Mode

We are going to break the work into small units. **After we finish each unit we want to let the browser interrupt the rendering procedure, if there's anything else that happens.**

Firstly let's read the code and comment below,

```javascript
let nextUnitOfWork = null;

// @deadline: 
// check how much time left until the browser needs to take control again
function workLoop(deadline) {
  let shouldYield = false;
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    shouldYield = deadline.timeRemaining() < 1; // check here
  }
  requestIdleCallback(workLoop);
}

// make a loop
requestIdleCallback(workLoop);

function performUnitOfWork(nextUnitOfWork) {
  // ... todo
}
```

 the browser will run the `requestIdleCallback` when the main thread is idle(空闲的).

> *React [doesn’t use requestIdleCallback anymore](https://github.com/facebook/react/issues/11171#issuecomment-417349573). Now it uses the [scheduler package](https://github.com/facebook/react/tree/master/packages/scheduler). But for this use case it’s conceptually the same.*



### Step 4: Fibers

We'll introduce <u>fiber tree</u> here. **We have one fiber for each element and each fiber holds a unit of work**.

![fiber1](https://cescdf.com/image/fiber1.png)

When render, we create the <u>root fiber</u> and set it as the `nextUnitOfWork`. The rest of the work will hapen on the `performUnitOfWork` function.

There are three things happending on each fiber:

1. add the element to DOM;
2. create fibers for the element's children;
3. select next unit of work;

**To find the next unit of work, each fiber has a link to its first child, then it links to the next sibling; the sibling fiber at the end has a link to its parent.**

The way the fiber goes is as below,

```
if (child) {
	goto(child);
} else if (sibling) {
	goto(sibling);
} else {
	goto(parent)
}
```

It's definitely a `DFS` traversing. A complete workflow for `render` is to **start from root fiber and end at root fiber.** (一次完整的`render`就是完整走一遍fiber tree)

OK!

Let's rewrite the `render` code part.

Firstly, to create a DOM,

```javascript
function createDOM(fiber) {
  const dom = 
        fiber.type === 'TEXT_ELEMENT'
  			? document.createTextNode("")
        : document.createElement(element.type);
  
  Object.keys(fiber.props)
  	.filter(key => key !== "children")
  	.forEach(name => {
    	dom[name] = fiber.props[name];
  	});
  
  return dom;
}
```

> `createDOM`就是下面式子里到函数f：
>
> dom = f (fiber)

In the `render` function we set `nextUnitOfWork` to the fiber tree root.

```javascript
function render(element, container) {
  nextUnitOfWork = {
    dom: container,
    props: {
      children: [element],
    },
  }
}
```

review the `workLoop` function, where we start working on the root.

```javascript
function workLoop(deadline) {
  let shouldYield = false;
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    );
    shouldYield = deadline.timeRemaining() < 1; // check here
  }
  requestIdleCallback(workLoop);
}

// make a loop
requestIdleCallback(workLoop);
```

for `performUnitOfWork`, see the code and comment,

```javascript
function performUnitOfWork(fiber) {
  // 1. add dom node
  if (!fiber.dom) {
    fiber.dom = createDOM(fiber);
  }
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom);
  }
  
  // 2. create new fibers for each child
  const elements = fiber.props.children;
  let index = 0;
  let prevSibling = null;
  
  while (index < elements.length) {
    const element = elements[index];
    
    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }
    
    // _add to fiber tree as child or sibling
    // | child: the first element
    // | sibling: not a child
    if (index === 0) {
      fiber.child = newFiber;
    } else {
      prevSibling.sibling = newFiber;
    }
    
    prevSibling = newFiber;
    index++;
  }
  
  // 3. return next unit of work
  if (fiber.child) {
    return fiber.child;
  }
  let nextFiber = fiber;
  while (nextFiber) {
    // work on all siblings
    if (nextFiber.sibling) {
      return nextFiber.sibling;
    }
    // after that, return to parent
    nextFiber = nextFiber.parent;
  }
}
```

That's our `performUnitOfWork`. A huge part, Uh?

> 1. `performUnitOfWork`是在`workLoop`里被调用的，`workLoop`会循环完成每一个 Unit of Work。
> 2. 注意看`performUnitOfWork`这个函数，可以知道`fiber`树是在遍历过程中一边遍历一边创建的。（即`const newFiber = {...}`）

### Step 5: Render and Commit Phases

Let's see this part we have written in `performNextUnitOfWork`,

```javascript
if (fiber.parent) {
  fiber.parent.dom.appendChild(fiber.dom);
}
// need to remove above codes
```

We add a new node toe the DOM each time we work on a fiber. Plus, the browser could interrupt the work before rendering a whole tree, **so the user will see an incomplete UI. And we don't want that.**

We need to remove that part and figure out a new way.

**We split the process to two parts: render + commit**.

> 之所以把render阶段拆分为render+commit，是因为希望在最后（即commit阶段）一并提交渲染完成的DOM树，以得到更好的用户UI体验。

Instead, we use a `wipRoot` variablt to keep track of the traversing of fiber tree.

```javascript
function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element],
    }
  };
  nextUnitOfWork = wipRoot;
}

let nextUnitOfWork = null;
let wipRoot = null;
```

Once we finish all the work (when there isn't a next unit of work), we commit the whole fiber tree to the DOM.

```javascript
function workLoop(deadline) {
  let shouldYield = false;
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    );
    shouldYield = deadline.timeRemaining() < 1;
  }
  
  if (!nextUnitOfWork && wipRoot) {
    commitRoot(); // commit the whole fiber tree to the DOM
  }
}
```

### Step 6: Reconciliation

**Let's implement the** `commitRoot` **function.**

```javascript
function commitRoot() {
  // call commitWork n times to delete old fibers,
  // where n = deletions.length
  deletions.forEach(commitWork);

  commitWork(wipRoot.child);
  currentRoot = wipRoot; // save a reference
  wipRoot = null;
}

// recursive
function commmitWork(fiber) {
  if (!fiber) {
    return;
  }
  
  const domParent = fiber.parent.dom;
  
  // determine the `effectTag`
  if (fiber.effectTag === "PLACEMENT" && fiber.dom !== null) {
    domParent.appendChild(fiber.dom);
  } else if (fiber.effectTag === 'UPDATE' && fiber.dom !== null) {
    updateDom(fiber.dom, fiber.alternate.props, fiber.props); // implement later
  } else if (fiber.effectTag === "DELETION") {
    domParent.removeChild(fiber.dom);
  }

  // DFS to commit the tree
  commitWork(fiber.child);
  commitWork(fiber.sibling);
}

function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element],
    },
    alternate: currentRoot, // a link to the old fiber
  }
  deletions = []; // keep tract of the nodes need to remove
  nextUnitOfWork = wipRoot;
}

let nextUnitOfWork = null;
let currentRoot = null;
let wipRoot = null;
let deletions = null;
```

> `render`函数只是把`fiber root`赋值给了`nextUnitOfWork`，这是一个开始；真正的挂载是在`workLoop`里调用`performUnitOfWork`完成的。

Now we look at `performUnitOfWork` that creates the new fibers,

```javascript
function performUnitOfWork(fiber) {
  // 1. add dom node
  if (!fiber.dom) {
    fiber.dom = createDOM(fiber);
  }
  
  // 2. create new fibers for each child
  const elements = fiber.props.children;
  reconcileChildren(fiber, elements); // move to a new function
  
  // 3. return next unit of work
  if (fiber.child) {
    return fiber.child;
  }
  let nextFiber = fiber;
  while (nextFiber) {
    // work on all siblings
    if (nextFiber.sibling) {
      return nextFiber.sibling;
    }
    // after that, return to parent
    nextFiber = nextFiber.parent;
  }
}
```

> 调和（reconciliation）阶段是在commit之前，这个阶段负责完成dom树的添加、更新和删除。

Here we reconcile(调和) the old fibers with the new elements.

```javascript
function reconcileChildren(wipFiber, elements) {
  let index = 0;
  let oldFiber = wipFiber.alternate && wipFiber.alternate.child;
  let prevSibling = null;
  
  while (index < elements.length || oldFiber != null) {
    const element = elements[index];
    let newFiber = null;
  }
}
```

**The `element` is the thing we want to render to the DOM and the `oldFiber` is what we rendered the last time.**

We need to compare them to see if there's any change we need to apply to the DOM.

- **A.** If the old fiber and the new element have the same type, we keep the DOM node and update the props.
- **B.** If the type is different and there's a new element, we need to create a new DOM node.
- **C.** If the type is diffferent and there's  an old fiber, we need to remove the old node.
- **B and C can happen together.**

```javascript
function reconcileChildren(wipFiber, elements) {
  // ...
  
  const sameType = oldFiber && element && element.type === oldFiber.type;
  
  if (sameType) {
    // update the node
    newFiber = {
      type: oldFiber.type,
      props: element.props,
      dom: oldFiber.dom,
      parent: wipFiber,
      alternate: oldFiber,
      effectTag: "UPDATE", // use this in commit phase
    }
  }
  if (element && !sameType) {
    // add this node
    newFiber = {
      type: element.type,
      props: element.props,
      dom: null,
      parent: wipFiber,
      alternate: null,
      effectTag: "PLACEMENT",
    }
  }
  if (oldFiber && !sameType) {
    // delete the oldFiber's node
    oldFiber.effectTag = 'DELETION';
    deletions.push(oldFiber)
  }
}
```

> React uses keys to make a better reconciliation. For example, it detects when children change places in the element array.

Let's implement `updateDom` now,

note that you must know the two things in JavaScript,

- Double arrow function (curried funciton);
- `in` operator: The **`in` operator** returns `true` if the specified property is in the specified object or its prototype chain.

```javascript
const isProperty = key => key !== "children";
const isNew = (prev, next) => key => { // curried function
  prev[key] !== next[key];
}
const isGone = (prev, next) => key => !(key in next);

function updateDom(dom, prevProps, nextProps) {
  // Remove old properties
  Object.keys(prevProps)
  	.filter(isProperty)
  	.filter(isGone(prevProps, nextProps))
  	.forEach(name => {
    	dom[name] = "";
  })
  
  // Set new or changed properties
  Object.keys(nextProps)
  	.filter(isProperty)
  	.filter(isNew(prevProps, nextProps))
  	.forEach(name => {
    	dom[name] = nextProps[name];
  })
}
```

One special kind of prop that we need to update are event listeners, so if the prop name starts with the “on” prefix we’ll handle them differently.

```javascript
const isEvent = key => key.startsWith("on");
const isProperty = key => {
  key !== "children" && !isEvent(key);
}
```

Let's see,

```javascript
function updateDom(dom, prevProps, nextProps) {
  // Remove old or changed event listeners
  Object.keys(prevProps)
  	.filter(isEvent)
  	.filter(key => !(key in nextProps) || isNew(prevProps, nextProps)[key])
  	.forEach(name => {
    const eventType = name.toLowerCase().substring(2);
    dom.removeEventListener(evnetType, prevProps[name])
  })
  // Add new event listeners
  Object.keys(nextProps)
  	.filter(isEvent)
  	.filter(isNew(prevProps, nextProps))
  	.forEach(name => {
    	const eventType = name.toLowerCase().substring();
    dom.addEventListener(evnetType, prevProps[name])
  })
  
  // Remove old properties
  // ...
  // ...
}
```

Try the version with reconciliation on [codesandbox](https://codesandbox.io/s/didact-6-96533).

Let's review the main thread of our code,

```javascript
function createElement(type, props, ...children) {}
function createTextElement(text) {}
function createDom(fiber) {}

var isEvent;
var isProperty;
var isNew;
var isGone;

function updateDom(dom, prevProps, nextProps) {}
function commitRoot() {}
function commitWork(fiber) {}
function render(element, container) {}

let nextUnitOfWork = null
let currentRoot = null
let wipRoot = null
let deletions = null

function workLoop(deadline) {}

requestIdleCallback(workLoop)

function performUnitOfWork(fiber) {}
function reconcileChildren(wipFiber, elements) {}
```

Finally,

```JSX
const Didact = {
  createElement,
  render,
}

/** @jsx Didact.createElement */
const container = document.getElementById("root")

const updateValue = e => {
  rerender(e.target.value)
}

const rerender = value => {
  const element = (
    <div>
      <input onInput={updateValue} value={value} />
      <h2>Hello {value}</h2>
    </div>
  )
  Didact.render(element, container)
}

rerender("World")
```



### Step 7: Function Components

The next thing we need to add is support for function components.

Function components are differents in two ways:

- the fiber from a function component doesn’t have a DOM node.
- and the children come from running the function instead of getting them directly from the `props`.

In `performUnitOfWork`, we check if the fiber type is a function,

```javascript
function performUnitOfWork(fiber) {
  const isFunctionComponent = fiber.type instanceof Function;
  
  if (isFunctionComponent) {
    updateFunctionComponent(fiber);
  } else {
    updateHostComponent(fiber); // the same as before
  }
  
  if (fiber.child) {
    return fiber.child;
  }
  let nextFiber = fiber;
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling;
    }
    nextFiber = nextFiber.parent;
  }
}
```

In `updateFunctionComponent`, see,

```javascript
function updateFunctionComponent(fiber) {
  const children = [fiber.type(fiber.props)];
  reconcileChildren(fiber, children);
}
```

 in `updateFunctionComponent`, we run the function to get the children. For our example, the `fiber.type` is the `App` function it returns the `h1` element.

Once we have the children, the reconciliation works in the same way, we don’t need to change anything there.

We need to change  `commitWork` function. First, to find the parent of a DOM node we need to go up the fiber tree until we find a fiber with a DOM node.

```javascript
let domParentFiber = fiber.parent
while (!domParentFiber.dom) {
  domParentFiber = domParentFiber.parent
}
const domParent = domParentFiber.dom

// determine the `effectTag`
```

Secondly, when removing a node we also need to keep going until we find a child with a DOM node.

```javascript
function commitDeletion(fiber, domParent) {
  if (fiber.dom) {
    domParent.removeChild(fiber.dom)
  } else {
    commitDeletion(fiber.child, domParent)
  }
}
```



### Step 8: Hooks

Last step. Now that we have function components let’s also add state.

Here's an example,

```JSX
function Counter() {
  const [state, setState] = Didact.useState(1)
  return (
    <h1 onClick={() => setState(c => c + 1)}>
      Count: {state}
    </h1>
  )
}
const element = <Counter />
const container = document.getElementById("root")
Didact.render(element, container)
```

We need to initialize some global variables before. First we set the work in progress fiber. We also add a `hooks` array to the fiber to support calling `useState` several times in the same component. And we keep track of the current hook index.

```javascript
let wipFiber = null
let hookIndex = null

function updateFunctionComponent(fiber) {
  // 1. set the work in progress fiber.
  wipFiber = fiber
  hookIndex = 0 // keep track of current hook index
  wipFiber.hooks = [] // keep tack of all hooks
  const children = [fiber.type(fiber.props)]
  reconcileChildren(fiber, children)
}
```

Let's turn to `useState` function,

```javascript
function useState(initial) {
  // 2. each time we call useState, we need to check old hook
  const oldHook =
    wipFiber.alternate &&
    wipFiber.alternate.hooks &&
    wipFiber.alternate.hooks[hookIndex];
  
  // if we have old hook, we add old state from old to new hook
  // otherwise, we set it to initial
	const hook = {
      state: oldHook ? oldHook.state : initial,
      queue: [],
	};
	
  // run the actions the next time we are rendering the component
  const actions = oldHook ? oldHook.queue : [];
  actions.forEach(action => {
    hook.state = action(hook.state)
  })
  
  // push the action to the hook queue,
  // set a new work in progress root as the next unit of work,
  // so the work loop can start a new render phase
  const setState = action => {
    hook.queue.push(action)
    wipRoot = {
      dom: currentRoot.dom,
      props: currentRoot.props,
      alternate: currentRoot,
    }
    nextUnitOfWork = wipRoot
    deletions = []
  }
	
  // add the hook to the fiber,
  // increment the hook index by one, and return the state.
  wipFiber.hooks.push(hook);
  hookIndex++;
  return [hook.state, setState]
}
```

Read the comment carefully to get the essence of hook design.

下面是卡颂老师实现`useState` Hook的例子，可以结合起来学习，参考[bilibili视频](https://www.bilibili.com/video/BV1iV411b7L1)。

```javascript
// 区分组件处于mount还是update状态
let isMount = true;
// 指向hook链表中当前处理的hook
let workInProgressHook = null;

// 一个fiber对象
// 每个函数组件对应一个fiber
const fiber = {
  stateNode: App,
  memorizedState: null, // 对应hooks的数据，如果有多个hooks的就保存一个链表
};

// 实现useState
function useState(initialState) {
  let hook;

  if (isMount) {
    hook = {
      // hook上还有一个memorizedState
      // react团队就是喜欢用同名变量
      memorizedState: initialState,
      next: null,
      // 更新状态的队列，使用队列是为了按顺序多次更新
      queue: {
        pending: null,
      },
    };
    if (!fiber.memorizedState) {
      fiber.memorizedState = hook;
    } else {
      workInProgressHook.next = hook;
    }
    workInProgressHook = hook;
  } else {
    hook = workInProgressHook;
    workInProgressHook = workInProgressHook.next;
  }

  // 剪开环状链表，计算新的状态
  // 获得上一次的状态，和update，得出本次的新状态
  let baseState = hook.memorizedState;
  if (hook.queue.pending) {
    let firstUpdate = hook.queue.pending.next;
    // 遍历update链表
    do {
      const action = firstUpdate.action;
      baseState = action(baseState);
      firstUpdate = firstUpdate.next;
    } while (firstUpdate !== hook.queue.pending.next);

    hook.queue.pending = null; // 清空链表
  }

  hook.memorizedState = baseState;
  return [baseState, dispatchAction.bind(null, hook.queue)];
}

// 更新useState的状态
function dispatchAction(queue, action) {
  // 真正的update是双向环状链表：不仅要计算新的update，并且每次更新是有优先级的
  const update = {
    action,
    next: null,
  };

  // 当前hook还没有触发的更新
  if (queue.pending === null) {
    // 形成环
    update.next = update;
  } else {
    // 多次调用，后面的调用会进入else的逻辑
    // queue.pending保存最后一个update, .next就保存第一个
    update.next = queue.pending.next;
    queue.pending.next = update;
  }

  // 每次执行后，确保queue.pending指向update的最后一个
  queue.pending = update;

  // dispatchAction会触发一次更新
  schedule();
}

// 调度
function schedule() {
  workInProgressHook = fiber.memorizedState; // 复位，重新指向第一个hook
  const app = fiber.stateNode(); // 触发组建的render
  isMount = false; // render过后就不处于mount状态了
  return app;
}

function App() {
  const [num, updateNum] = useState(0);

  console.log("isMount?", isMount);
  console.log("num", num);

  return (
    <>
      <h1>Make Your Own Hooks!</h1>
      <button
        style={{ width: "360px", height: "200px", fontSize: "50px" }}
        onClick={() => updateNum((num) => num + 1)}
      >
        {num}
      </button>
    </>
  );
}

window.app = schedule();
```

>  我们用尽可能少的代码模拟了`Hooks`的运行，但是相比`React Hooks`，他还有很多不足。以下是他与`React Hooks`的区别：
>
> 1. `React Hooks`没有使用`isMount`变量，而是在不同时机使用不同的`dispatcher`。换言之，`mount`时的`useState`与`update`时的`useState`不是同一个函数。
> 2. `React Hooks`有中途跳过`更新`的优化手段。
> 3. `React Hooks`有`batchedUpdates`，当在`click`中触发三次`updateNum`，`精简React`会触发三次更新，而`React`只会触发一次。
> 4. `React Hooks`的`update`有`优先级`概念，可以跳过不高优先的`update`。

