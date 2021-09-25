# State依赖Props

State依赖props在开发中很常见。一个组件的state依赖于props，每次props变化都要更新state。

在Vue中主要通过computed和watch来处理这种情况。



在react中如何实现？

`memoize()`:这类似于vue的computed。memorization接收一个函数，只有在函数的参数与上一次不同的情况下，才会执行这个函数，否则使用缓存。

[`static getDerivedStateFromProps()`](https://react.docschina.org/docs/react-component.html#static-getderivedstatefromprops)也可以在props变化时更新state。但这个生命周期并不只是**props变化时执行**，在组件挂载时和更新(newProps、`steState()`和`forceUpdate()`)时都会执行。



如果是state是通过props来计算出来的，建议用`memoize()`。

