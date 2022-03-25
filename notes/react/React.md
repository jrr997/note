# React

## 并发模式(Concurrent Mode)

1. 正常情况下，在页面渲染的过程中，浏览器不能响应用户的交互(例如输入内容、点击按钮等)，这是浏览器渲染进程和JS脚本执行不能同时进行。

   浏览器的刷新率一般为60帧，就是每隔16.6ms刷新一次。在16.6ms中要处理完JS执行和页面渲染，如果JS执行时间过长导致页面渲染时间未完成就会让用户感觉到卡顿。

   并发模式如何解决这个问题？

   在每一帧中，预留足够的时间渲染页面，JS进程可中断。例如每一帧中只允许JS的执行时间是5ms，现在有一个JS任务要处理10ms，那么这个任务分为两帧完成。第一帧执行5ms的JS，然后中断JS的执行，剩余时间渲染页面。第二帧继续执行剩余的JS任务和页面渲染。

2. 在页面跳转时，通常需要发送请求获取第二个页面的资源。如果请求时间过长，会导致页面空白，一般我们会进行首屏优化。在并发模式下，当页面要发生跳转时，会立刻请求新页面的资源，但不会立刻跳转，而是先在当前页面停留一段时间。当“这一小段时间”足够短时，用户是无感知的。如果请求时间超过一个范围，再显示`loading`的效果。



## Context

React是单向数据流，当嵌套多层的组件需要上层组件的数据时，需要把props从上到下逐层传递。

什么是context? 何时需要用context?

context是react组件传递props的一种方式，类似于广播。当**多个组件**都需要相同的data时，在父组件中可通过context传递。

应用场景就是"多个组件"，例如现在有三层组件(父、子、孙)，当两个子组件和一个孙组件需要用到父组件的data时，需要用到context。再例如多个子组件都要使用父组件的data。

考虑这种场景：

仍然有父、子、孙三层组件，只有一个孙组件需要用到父组件的data。按照单向数据流的思想，父组件的data的流向是父组件 -> 子组件 -> 孙组件。这里嵌套不深，如果只有少量data，这样传数据时没问题的。

如果这个孙组件需要用到父组件的多个data(例如5个)，那父组件需要以props的形式传5个data到子组件，然后子组件以相同的方式把接收到的数据传到孙组件。这样造成了一个问题:子组件接收了很多自身用不到的props，这会增加子组件的复杂度。

如果使用context也有点大材小用，因为只有一个组件需要这个数据。

React官方推荐一种解决方式是：在父组件中把孙组件传递到子组件，这样孙组件能使用父和子组件的data，并且父组件无需向子组件传递无用的props。

context的基本使用(官方例子):

```javascript
// Context lets us pass a value deep into the component tree
// without explicitly threading it through every component.
// Create a context for the current theme (with "light" as the default).
const ThemeContext = React.createContext('light');

class App extends React.Component {
  render() {
    // Use a Provider to pass the current theme to the tree below.
    // Any component can read it, no matter how deep it is.
    // In this example, we're passing "dark" as the current value.
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

// A component in the middle doesn't have to
// pass the theme down explicitly anymore.
function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  // Assign a contextType to read the current theme context.
  // React will find the closest theme Provider above and use its value.
  // In this example, the current theme is "dark".
  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}
```

注意：这个例子中```const ThemeContext = React.createContext('light');```是这个文件的全局变量，被父组件和孙组件引用。在日常开发时，提供context的组件和使用context组件很可能在不同的文件中，我们要解决一个问题：如何让这两个在不同文件中的组件都能引入`React.createContext()`。

解决的方法是新建一个context.js来存储context，并把其export出去。这样提供者和使用者都能通过import引入同一个context。



## Refs

- 什么是refs?

  refs是DOM nodes或者React组件实例

- refs有什么用?什么时候会用到refs？如何使用？

当某个组件想获取其他组件的实例，或者获取某个原生的DOM元素时可以用refs。例如一个父组件想手动调用子组件的方法，父组件可以通过refs来获取子组件的实例，然后调用其方法或访问其属性。代码如下：

```javascript
export default class Father extends PureComponent {
  constructor(props) {
    super(props)
    this.childRef = React.createRef()
    this.fnChildRef = React.createRef()
  }
  handleClick = () => {
    alert(this.childRef.current.state.text);
  }
  render() {
    return (
      <div>
        <Child ref={this.childRef} />
        <button onClick={this.handleClick}>点击</button>
      </div>
    )
  }
}

// class component 
class Child extends PureComponent {
  constructor(props) {
    super(props)
  }
  handleChange = (e) => {
    this.setState({ text: e.target.value })
  }
  render() {
    return (
      <input type="text" onChange={this.handleChange} />
    )
  }
}
```

上面这个例子体现了父组件获取Class子组件实例，然而有时候子组件可能是函数式组件，函数式组件是没有实例的，因此传入ref属性没有效果。

```javascript
function MyFunctionComponent() {
  return <input />;
}

class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = React.createRef();
  }
  render() {
    // This will not work!
    return (
      <MyFunctionComponent ref={this.textInput} />
    );
  }
}
```



如何获取函数式组件中的state、方法等？

1. 如果只想获取函数式组件return的某个元素的ref，可以使用`React.forwardRef()`，例子如下：


```javascript
// function component
const FnChild = React.forwardRef((props, ref) => {
  return (
    <div>
      <input type="text" ref={ref} />
    </div>
  )
})

export default class Father extends PureComponent {
  constructor(props) {
    super(props)
    this.fnChildRef = React.createRef()
  }
  handleClick = () => {
    alert(this.fnChildRef.current.value);
  }
  render() {
    return (
      <div>
        <FnChild ref={this.fnChildRef} />
        <button onClick={this.handleClick}>点击</button>
      </div>
    )
  }
}
```

2. 如果想获取函数式组件的内部的属性、方法等，可以使用：

   ```javascript
   useImperativeHandle(ref, createHandle, [deps])
   ```

   例子：

   ```javascript
   const FnChild2 = React.forwardRef((props, ref) => {
     const inputRef = useRef()
     useImperativeHandle(ref, () => ({
       value: () => inputRef.current.value
     }))
     return (
       <div>
         <input type="text" ref={inputRef} />
       </div>
     )
   })
   ```

- 上面这些用法都有一个前提：可以在子组件内部加代码。如果不能修改子组件，但是又想拿到子组件中的DOM，可以使用[`findDOMNode()`](https://reactjs.org/docs/react-dom.html#finddomnode)。

- Callback Refs：利用闭包，父组件往子组件的ref属性传自身的方法。React会在子组件mount时调用ref 回调，并且回调函数的第一个参数是自身ref，回调函数如下(官方例子https://reactjs.org/docs/refs-and-the-dom.html)：

  ```javascript
      this.setTextInputRef = element => {
        this.textInput = element; // element 为返回的ref
      };
  ```

  **注意只能用在DOM和Class组件，不能用于函数式组件。**



## 受控组件和非受控组件

React在处理原生表单时，可以实现与表单元素的"双向绑定"，这就是受控组件。并不是所有表单元素都受控，比如说上传文件。

```javascript
<textarea value={this.state.value} onChange={this.handleChange} />
```

简单来讲，受控组件就是给组件提供一个value， 一个onChange。value 用于渲染当前的UI视图，在用户触发操作，需要更新value的时候，组件调用onChange，由上层组件负责更新状态。（这就是双向绑定）

如果不通过双向绑定，如何获取表单元素中的值？

用ref直接从DOM元素中获取值。这就是非受控组件。

**总结：**受控指的是表单元素受所在的组件控制，即表单元素能与组件进行双向绑定。非受控组件是指组件通过ref获取表单元素的值。

