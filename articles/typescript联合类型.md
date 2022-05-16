# typescript联合类型

最近在看antd源码时，一个typescript的联合类型写法让我感到困惑。

正常思路写联合类型时应该是这样的，这样很容易理解，task类型就是task1-task3之一。

```typescript
type task = task1 | task2 | task3
```



令我困惑的写法是这样的：

```ty
type task = 
	| task1 
	| task2
	| task3
```

为什么会多了一个 “|”，而且百度、谷歌和typescript的文档都难以找到解释。



后来在stack overflow上找到答案，这种写法是为了**方便和美观**。其实上面两种写法功能是相同的。不过按照第一种写法，如果要换行(见下面代码)，会令人看得很不爽。

```typescript
type task = task1 
	| task2
	| task3
```

