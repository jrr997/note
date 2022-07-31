# Redux中间件

## Redux Thunk

如果没有中间件，redux的dispatch方法接受一个Action，Action是一个对象。

redux-thunk是redux的中间件，可以使dispatch接受一个Action Creator，Action Creator是一个函数，其签名如下。前两个参数是Redux提供的dispatch和getState方法，默认情况下Action Creator只接受这两个参数。

```javascript
(dispatch, getState, ...extraArguments) => Action
```

通过配置，也可以让Action接收自定义参数:

```javascript
import { configureStore } from '@reduxjs/toolkit'
import rootReducer from './reducer'
import { myCustomApiService } from './api'

const store = configureStore({
  reducer: rootReducer,
  middleware: getDefaultMiddleware =>
    getDefaultMiddleware({
      thunk: {
        extraArgument: myCustomApiService
      }
    })
})

// later
function fetchUser(id) {
  // The `extraArgument` is the third arg for thunk functions
  return (dispatch, getState, api) => {
    // you can use api here
  }
}
```



redux-thunk的用法：

主要逻辑集中在Action Creator中。假如我们要发请求获取数据，可以在Creator中同步dispatch一个Action表示请求已发出。然后正式发起请求，在获取到数据时再dispatch一个Action表示数据以获取。

```javascript
const fetchPosts = postTitle => (dispatch, getState) => {
  dispatch(requestPosts(postTitle));
  return fetch(`/some/API/${postTitle}.json`)
    .then(response => response.json())
    .then(json => dispatch(receivePosts(postTitle, json)));
  };
};

// 使用方法一
store.dispatch(fetchPosts('reactjs'));
// 使用方法二
store.dispatch(fetchPosts('reactjs')).then(() =>
  console.log(store.getState())
);
```

这里fetchPosts返回一个Action Creator，传进了store.dispatch中，redux会执行这个Action Creator，先dispatch一个Action（`requestPosts(postTitle)`），然后进行异步操作。拿到结果后，先将结果转成 JSON 格式，然后再发出一个 Action（ `receivePosts(postTitle, json)`）。

总结：redux-thunk使store.dispatch方法可以接受一个函数(Action Creator)，我们可以在Action Creator中进行异步操作。



## Redux Saga

redux-saga用generator进行自动流程控制。genrator函数何时进行下一步操作完全取决于外部的调度时机，且其内部执行状态也由外部的输入决定。

redux-saga中的一些用于创建effects的api如put、take等，其实只是创建了一个对象，其执行逻辑在generator外部的函数中。