- .vue文件的模板最终会被编译成render函数。
  - ```h(html标签或组件: String | Component,options: ?oBject,children?: Array)```函数的作用是创建元素,当然也可以传入插槽`() => {}`。在手写render函数时会用到h函数。h()函数最终会调用`createVNode()`来生成VNode(其实h函数就是createVNode函数的简单封装)



## 组合式API setup()

- setup()只会执行一次，且执行过程中不能访问data、computed和methods。在setup中可以用`reactive()和ref()`来定义响应式数据
  - 第一个参数是props。props是响应式的，在setup()中不能直接解构。可以用`toRefs`来解构
  - 第二个参数是context，是一个对象，包括三个属性：`attrs slot emit`
- `watchEffect(() => {})`中依赖到的数据发生变化时，箭头函数就会执行
- setup()返回一个对象时，对象的属性以及第一个参数props可以被模板访问。
- setup还可以返回一个render函数，render函数会渲染页面，并且当依赖发生改变时，render函数会重新渲染页面。

一个组件可能有非常多的功能，如果按照以前的options开发模式，一个功能的逻辑代码是分散的，多个功能的代码交错在一起。如果这个组件非常复杂、庞大，那么就很难维护。

setup()可以解决这个问题：把每个功能都写在一个单文件下面，在组件的setup()中引入功能。这样就能把每个功能的代码都放在一个文件，而不同功能之间的代码有序。



### ref和reactive

- 两者的共同作用是令数据变成响应式
- 不同之处：
  - reactive()接受一个对象，深度遍历这个对象，把每一个属性都变成响应式。注意：reactive不能使基本类型、数组变成响应式，所以才有ref。
  - ref()，返回的是一个对象```const foo = ref(0)```,通过`foo.value`可以访问到响应式数据“0“。foo实际上就是{value: 0}的一个引用，只不过这个对象已经是响应式的。
    - 值得注意的是：在模板中，我们并不需要要通过`foo.value`来获取数据，直接通过foo即可访问到数据，这是因为ref自动展开。
    - 还有一种情况，当 `ref` 作为响应式对象的 property 被访问或更改时，为使其行为类似于普通 property，它会自动展开内部值





## 为什么使用JSX

1. vue3提供更好的API支持：setup()返回render函数
2. typescript在编译时能对jsx进行错误检查
3. jsx语法灵活

坑：在.vue文件中写组件时，可能会对组件的props类型进行定义。而在使用组件的过程中，如果传入的props类型不对，ts在编译时会报错，但是编辑器不报错。原因ts不支持在.vue文件中定义类型





## TypeScript

优势：

1. 强类型
2. 更早地发现错误(运行前发现错误)
3. 是javascript的超集，支持javascript最新特性



使用：

typescript中尽量避免使用var



### typescript的新类型:

元组：

- 固定长度、固定类型的Array。
- 声明元组：`let tuple : [number, string] = [1, 'zack']`



联合类型(Union):

- 如果我们希望一个变量是数字或字符串：`let a : number | string = 1`
- 联合类型就是"或"的意思，用`|`来表示



字面量类型(literial):

- 联合类型可以指定一个变量是多种类型，字面量类型指定变量的具体值：`let color : 'green' | 'red' = 'green'`



枚举类型(enum):

exp:

```typescript
enum color = {
    red = 'red',
    green = 'green'
} 
console.log(color.red) // red

enum color2 = {
    red, // 0
    green // 1
}
console.log(color,green) // 1 
```



any 和 unknown:

- any是任意类型
- unknown不能保证类型判断，但能保证类型安全。使用unknown时，要手动进行类型判断后才执行相应的操作。



void、undefined和never：

这是三个都是函数的类型。

```typescript
// void 当函数没有返回值时用void。
function log(msg): void {
    console.log(msg)
}
// 虽然函数没有返回值，但是当打印函数时能获取到undefined
console.log(log(123)) // undefined

// undefined 当函数主动返回一个undefined的值时
function foo() : undefined {
    return
}

// never 当函数永远执行不完时(抛出异常或死循环)
function bar() : never {
    throw error()
} // 执行不到这里
```



### 类型适配(type assertions)

一个变量一开始声明为any，后来我们希望指定他为string类型时

```typescript
let msg : any = 'hello'
(<string>msg).split('') // 方法一
(msg as string).split('') // 方法二: 使用as
```

当我们知道一个数据的具体类型，而typescript不知道，此时可以用as来告诉typescript，此数据的类型就是这个。





### Provide和Inject的原理

父组件在setup中通过`provide(key, value)`可以向子孙组件传东西，子孙组件通过`inject(key)`获取父组件提供的东西。那么其背后是如何工作的？

父组件provide相当于向下传递了一个对象，子孙组件inject的就是这个对象。

Q：如果父组件和子组件都有自己的provide，那么如何把这两个provide都传到孙组件？如果父子的provide有冲突该如何处理？

A：如果只有父组件有provide，那么孙组件拿到的就是父组件provide的对象。如果子组件也有provide，那么子组件provide的对象 = Object.create(父组件provide的对象)。简单地说，父子组件的provide具有原型链的关系，子provide.proto === 父provide。孙组件在inject()某个属性时，先从子组件的provide中找，找不到的话会顺着原型链去父组件的provide中找。
