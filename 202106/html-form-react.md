# HTML表单和React的受控组件

# HTML Form And React's Controlled Components

## 2021 年 6 月 13 日

## Jun 13 2021



### HTML表单基础

HTML 表单用于搜集（不同类型的）用户输入。

`<form>`用于定义表单元素，`<input>`是最重要的表单元素。

通过`<form>`元素的`action`属性定义提交表单时执行的操作。

`<input>` 元素根据不同的 *type* 属性，可以变化为多种形态。

**🌟form的子标签**

`<form>`标签内可以定义如下内容，

- `<input>`
- `<select name="selectName">` + `<option value="val">`
- `<textarea name="textName">`
- `<button type="button/reset/submit">`

**🌟input的所有类型**

- `<input type="text">`
- `<input type="password">`
- `<input type="submit">`
- `<input type="radio">`：单选框
- `<input type="checkbox"> `：复选框
- `<input type="button">`

HTML5 增加了多个新的输入类型：

- color
- date
- datetime
- datetime-local
- email
- month
- number
- range
- search
- tel
- time
- url
- week

以上每个输入类型都有自己的属性可以用，需要时请查阅。

### 受控组件、非受控组件

**🌟受控组件**

> 关于受控组件，请查阅[React官方文档](https://reactjs.org/docs/forms.html#controlled-components)。

React认为，在HTML中，`<input>`、`<textarea>`和`<select>`等表单元素通常维护自己的状态，并根据用户输入进行更新。而在React中，可变状态应该保存在组件的state属性中，并且仅使用setState更新。

那么，React怎样解决这个问题呢，答案就是，让React的state成为这些表单状态的 “**single source of truth**”，即唯一真相源。

> An input form element whose value is controlled by React in this way is called a “controlled component”.

举一个例子，

```jsx
function NameForm(props) {
  
  const [val, setVal] = useState("")

  function handleChange(e) {
    setVal(e.target.value);
  }

  function handleSubmit(e) {
    // do something
    event.preventDefault();
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Name:
        <input type="text" value={val} onChange={handleChange} />
      </label>
      <input type="submit" />
    </form>
  );
}
```

其实就是在`<input>`标签里，配合value和onChange属性，一个设置为状态，一个设置为状态对应的函数，这样就能通过React状态实时跟踪`<input>`的状态。

`<form>`标签的onSubmit属性不是必要的，但必要时也可以使用。

同样的，select标签也可以这样使用，

```jsx
function FlavorForm(props) {
  
  const [value, setValue] = useState("grapefruit");
  
  /*
  ...
  */
  
  return (
  <form onSubmit={handleSubmit}>
  <label>
    Pick your favorite flavor:
    <select value={value} onChange={handleChange}>
      <option value="grapefruit">Grapefruit</option>
      <option value="lime">Lime</option>
      <option value="coconut">Coconut</option>
      <option value="mango">Mango</option>
    </select>
  </label>
  <input type="submit" value="Submit" />
</form>)
}
```

**🌟非受控组件**

React举了一个非受控组件的例子，

```html
<input type="file" />
```

没了。

React也允许使用非受控组件的方式来处理组件的值，如下，

```jsx
function NameForm(props) {
  
  const input = useRef(null);
  const [value, setValue] = useState("");
	
  function handleSubmit(e) {
    e.preventDefault();
    setValue(input.current.value);
  }
  
  return (
    <form onSubmit=(handleSubmit)>
      <label htmlFor="input">
        Name:
        <input type="text" defaultValue={value} ref={input} name="input"/>
      </label>
      <input type="submit" value="Submit" />
    </form>
  );
}
```

### React之Ref

Refs 提供了一种方式，允许我们访问 DOM 节点或在 render 方法中创建的 React 元素。

下面是几个适合使用 refs 的情况：

- 管理焦点，文本选择或媒体播放。
- 触发强制动画。
- 集成第三方 DOM 库。

避免使用 refs 来做任何可以通过声明式实现来完成的事情。你可能首先会想到使用 refs 在你的 app 中“让事情发生”。如果是这种情况，请花一点时间，认真再考虑一下 state 属性应该被安排在哪个组件层中。通常你会想明白，让更高的组件层级拥有这个 state，是更恰当的。查看 [状态提升](https://zh-hans.reactjs.org/docs/lifting-state-up.html) 以获取更多有关示例。

**🌟同步地使用DOM**

我发现，最希望用到refs的场景是在获取DOM时，希望在function组件语句里同步地获得，而不是在DOM渲染结束后通过useEffect+document API来获得节点，这样做有很多原因，比如有一些函数组件的变量在useEffect里无法获取，或者修改了之后无法返回到函数组件中，这就希望在DOM创建之前就拿到这个节点。

但是我这样做的时候，常常出BUG，也不知道是什么原因，有时间要测试一下。

