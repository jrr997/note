# React 生命周期

## constructor

1. 官网介绍：只有在初始化state或把方法绑定到实例上才需要constructor。个人认为只有需要**初始化state**时用到constructor。
2. constructor不能有副作用。
3. 第一步要super(props)。



## render

1. render是纯函数，不能有副作用。相同的props和state，return相同的结果。
2. render return的不只有react element(组件或DOM)，还有很多其他类型如String/Number/Boolean等。



## componentDidMount()

1. 在第一次render后执行，可以在这发送网络请求、添加订阅、setState
2. 在这里setState会导致调用两次render()方法，虽然两次调用会发生在浏览器渲染页面之前，但仍可能导致性能问题，所以谨慎使用。一个使用场景是模态框的大小依赖于 DOM 节点的大小或位置。



## componentDidUpdate()

`componentDidUpdate(prevProps, prevState, snapshot)`

在newProps、setState()、forceUpdate()后触发。





生命周期图：https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/



挂载时：

- [**`constructor()`**](https://zh-hans.reactjs.org/docs/react-component.html#constructor)
- [`static getDerivedStateFromProps()`](https://zh-hans.reactjs.org/docs/react-component.html#static-getderivedstatefromprops)
- [**`render()`**](https://zh-hans.reactjs.org/docs/react-component.html#render)
- [**`componentDidMount()`**](https://zh-hans.reactjs.org/docs/react-component.html#componentdidmount)

关于getDerivedStateFromProps：

getDerivedStateFromProps在render调用前调用，包括挂载时和更新时。实际上只要父级重新渲染时，这个生命周期函数就会重新调用，不管 props 有没有“变化”。



更新时：

- [`static getDerivedStateFromProps()`](https://zh-hans.reactjs.org/docs/react-component.html#static-getderivedstatefromprops)
- [`shouldComponentUpdate()`](https://zh-hans.reactjs.org/docs/react-component.html#shouldcomponentupdate)
- [**`render()`**](https://zh-hans.reactjs.org/docs/react-component.html#render)
- [`getSnapshotBeforeUpdate()`](https://zh-hans.reactjs.org/docs/react-component.html#getsnapshotbeforeupdate)
- [**`componentDidUpdate()`**](https://zh-hans.reactjs.org/docs/react-component.html#componentdidupdate)



卸载时：componentWillUnmount

