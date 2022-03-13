# React Hooks

## useCallback

```javascript
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

参数1: function，参数2:依赖数组。



useCallback解决什么问题？



```javascript
funtion App() {
    const [a, setA] = React.useState(0)
    function fn() {
        console.log('fn')
    }
    return <Child fn={fn}/>
}
    
function Child({fn}) {
    return 'child'
}
const Child = React.memo(Child)
```

在这个例子中，父组件APP中的函数fn作为props传给了子组件Child，父组件还有一个状态a。如果父组件调用setA，那么父组件会重新渲染。在渲染的过程中，会新建一个函数fn，这个新建的fn和之前传给Child组件的fn不是相同的引用。换句话说，Child接受的props发生了变化，因此重新渲染。然而fn虽然引用不一样，但功能是一样的，有没有办法可以在父组件App重新渲染时，其传给子组件的fn始终为相同引用？
答案是使用useCallback。
```javascript
funtion App() {
    const [a, setA] = React.useState(0)
    const fn = React.useCallback(
    	() => {console.log('fn')},
    	[]
    )

    return <Child fn={fn}/>
}
```
只要依赖数组中的依赖没发生变化，fn始终保持同一个引用。这个例子中依赖数组为空，因此fn始终保持相同引用。

什么情景需要用useCallback？

useCallback和React.memo搭配，避免吃性能的组件经常因为父组件重新渲染，参考链接2。

相关链接：

1. [React官方](https://react.docschina.org/docs/hooks-reference.html#usecallback)
2. [Your Guide to React.useCallback()](https://dmitripavlutin.com/dont-overuse-react-usecallback/)



## useMemo

useCallback解决函数引用发生变化的问题，而useMemo解决的是属性引用的问题。

```javascript
funtion App() {
    const [a, setA] = React.useState(0)
    const obj = {name: 'haha', age: 999}
    function fn() {
        console.log('fn')
    }
    return <Child fn={fn} obj={obj}/>
}
    
function Child({fn}) {
    return 'child'
}
const Child = React.memo(Child)
```

这个例子中，当父组件重新渲染时，obj对象的引用也发生了变化，导致子组件Child重新渲染。



使用useMemo

```javascript
funtion App() {
    const [a, setA] = React.useState(0)
    const [age, setAge] = React.useState(18)
    const obj = React.useMemo(() => {return {name: 'haha', age}}, [age])
    function fn() {
        console.log('fn')
    }
    return <Child fn={fn} obj={obj}/>
}
    
function Child({fn}) {
    return 'child'
}
const Child = React.memo(Child)
```

