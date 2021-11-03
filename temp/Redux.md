# Redux

redux是一个状态管理库。三大原则：

- **单一数据源**: 整个应用的 state 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 store 中。
- **State 是只读的**: 唯一改变 state 的方法就是触发 action，action 是一个用于描述已发生事件的普通对象。
- **使用纯函数来执行修改**: 为了描述 action 如何改变 state tree ，你需要编写 reducers。



## Elm架构[The Elm Architecture](https://guide.elm-lang.org/architecture/	)

redux的工作流程参考了Elm架构，因此我们先了解一下Elm架构。

![](https://guide.elm-lang.org/architecture/buttons.svg)

三个重要概念：

- **Model**: 应用的状态
- **View**: 由状态输出的HTML
- **Update**: 状态的更新逻辑

用户的一次交互流程：用户点击按钮，View发送Msg到Elm，Elm的Update根据Msg来更新Model。Model改变后，View也随之变化。



## Redux + React
![](https://www.ruanyifeng.com/blogimg/asset/2016/bg2016091802.jpg)

- Store: 存储State的地方。**redux 应用只有一个单一的 store**。
- Action: 是一个对象`{type, ...}`。Store接受Action，根据Reducer和Action来更新state。
- Reducer: `(state, action) => state`。是一个纯函数，返回新的state。

Redux的工作流程：React组件dispatch action 到store，store的reducer根据action来更新Model。Model后，由React组件组成的UI也发生改变。
与Elm的工作流程对比，可以发现Elm的三个基本概念分别与Redux的概念一一对应:

- Mdel -> Store
- Vew -> Ract
- Udate -> Rducer
   还有Msg对应的是Action

