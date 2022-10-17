# Umi第一个插件

前言：umi插件开发的新手，通过手写一个简单的umi插件来学习umi插件的开发。



目标：写一个插件，在umi页面中插入一个标题。



1.初始化npm

`npm init -y`



2.创建一个umi页面

在项目根目录中创建`pages/index.js`:

```jsx
export default () => <div>Hello Umi!</div>
```



3.安装umi

`yarn add umi@3.x`



4.在根目录下创建插件目录`umi-plugin-hello`



5.创建插件的`package.json`文件，并且终端运行`yarn install`安装依赖

主要是利用`babel`把代码转成`es5`，插件的代码`src/index.js`中，终端运行`yarn build`后，会在`lib`目录中输出`es5`代码。如果不这么做会报错。

```json
{
  "name": "umi-plugin-hello",
  "version": "1.0.0",
  "description": "",
  "main": "./lib",
  "scripts": {
    "build": "babel src -d lib"
  },
  "babel": {
    "presets": [
      "@babel/preset-env"
    ]
  },
  "devDependencies": {
    "@babel/cli": "^7.19.3",
    "@babel/core": "^7.19.3",
    "@babel/preset-env": "^7.19.4"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}

```



6.在`src/index.js`中编写插件的代码：

插件的功能是在`<body>`的最前面插入一个标题"hello umi plugin"

```js
export default (api) => {
  api.modifyHTML($ => {
    $("body").prepend(`<h1>hello umi plugin</h1>`);
    return $;
  })
}
```



7.在插件根目录下运行`yarn build`，生成`lib`目录。



8.在项目根目录安装插件

`yarn add ./umi-plugin-hello`



9.启动项目

`yarn umi dev`



注意：在 Umi@3 中，当插件使用 `@umijs` 或者 `umi-plugin` 开头，只要安装就会被默认使用。如果插件名称不满足这个要求，需要在umi中配置

```javascript
// umirc.js
import { defineConfig } from 'umi';

export default defineConfig({
  plugins: ['you-plugin-name'],
});
```



总结：umi插件就是一个js函数:

```js
export default (api) => {
  // your plugin code here
};
```

接收一个参数`api`，我们利用`api`注册很多钩子函数。umi会在正确的时机执行这些钩子。



插件的API文档: [Plugins Api](https://v3.umijs.org/plugins/api)

代码地址 :https://github.com/jrr997/first-umi-plugin