# Webpack模块化打包原理

概述：本文通过阅读webpack的打包结果，来理解webpack的模块化打包原理。

阅读本文前需要知道的知识：

- webpack的[核心概念](https://webpack.js.org/concepts/)，主要了解入口(entry point)，出口(output)和loader的基本概念即可。
- [依赖图](https://webpack.js.org/concepts/dependency-graph/)
- 模块化规范commonjs、esModule和[webpack的模块概念](https://webpack.js.org/concepts/modules/)



什么是webpack？

官方文档提到：**webpack** 是一个用于现代 JavaScript 应用程序的 *静态模块打包工具*。说白了，webpack的核心功能是把我们平时模块化开发的代码打包成一个整体包，这个整体包可以在浏览器中运行。



webpack的打包结果：

举个例子：现在有三个js文件:entry.js、message.js和name.js

```javascript
// enrty.js
import message from './message.js';
console.log(message)

// message.js
import {name} from './name.js';
export default `hello ${name}!`;

// name.js
export const name = 'world';
```

现在指定entry.js为入口文件，让webpack帮助我们打包，在mode为none的情况下，打包的结果bundle.js如下：

```javascript
/*******************************依赖图*****************************************/
(() => {
  "use strict";
    
  // __webpack_modules__是一个存储了所有依赖关系的数组，一个依赖就是一个模块。
  // 数组的每个元素都是一个函数，一个函数代表一个模块，函数有三个参数：
  // 参数1 -> __unused_webpack_module 这个参数与模块缓存有关，在这个例子没有作用，因此不讨论
  // 参数2 -> __webpack_exports__ 是一个对象，存储了这个模块的所有导出信息，相当于nodejs中的            module.exports 对象
  // 参数3 -> __webpack_require__ 是一个函数。当一个模块需要引入其他模块时，用此方法引入。这个函数接      受某个模块的唯一标识moduleId,返回这个的导出(__webpack_exports__)。在第二部分还会对这个函数有更      详细的介绍。
  // 总结：用函数表示模块，一个模块应该由导入、导出和模块主要代码所组成。
  var __webpack_modules__ = ([
/* 0 这里其实是entry.js模块，这个模块在此为空，原因是其被拿出来单独执行了(看最下面的加载入口文件那块代码)，实际上放这里也完全没问题*/,
    /* 1 message.js*/
    ((__unused_webpack_module, __webpack_exports__, __webpack_require__) => {
	  // 标记exports对象 -> exports.__esModule = true,在这个例子中无作用，因此可以忽略
      __webpack_require__.r(__webpack_exports__); 
      // 通过代理导出
      __webpack_require__.d(__webpack_exports__, { 
        "default": () => (__WEBPACK_DEFAULT_EXPORT__)

      });
      // 导入name.js，实际上导入的是name.js模块的第二个参数__webpack_exports__
      var _name_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(2);

		
      const __WEBPACK_DEFAULT_EXPORT__ = (`hello ${_name_js__WEBPACK_IMPORTED_MODULE_0__.name}!`); // 暴露


    }),
    /* 2 name.js*/
    ((__unused_webpack_module, __webpack_exports__, __webpack_require__) => {
	  // 标记，可忽略
      __webpack_require__.r(__webpack_exports__);
      // 代理导出
      __webpack_require__.d(__webpack_exports__, {
        "name": () => (/* binding */ name)

      });
      // 模块主代码
      const name = 'world';
    })
  ]);
  /********************************全局变量和方法****************************************/
  // The module cache
  // 模块缓存，一个模块第一次被导入时，会执行这个模块的相关代码，并把这个模块的导出放到缓存对象中。
  // 当模块第二次被导入时，直接从缓存对象中找出这个模块的导出，而不会再次执行模块代码。
  var __webpack_module_cache__ = {};

  // The require function
  // 作用：引入一个模块
  // 逻辑：先从缓存找某个模块，找不到的话从依赖图找，并把这个模块的导出放进缓存。第二次引入这个模块时就能在      缓存中找到。
  // 函数返回被引入模块的导出
  function __webpack_require__(moduleId) {
    // Check if module is in cache
    var cachedModule = __webpack_module_cache__[moduleId];
    if (cachedModule !== undefined) {
      return cachedModule.exports;

    }
    // Create a new module (and put it into the cache)
    var module = __webpack_module_cache__[moduleId] = {
      // no module.id needed
      // no module.loaded needed
      exports: {}

    };

    // Execute the module function
    __webpack_modules__[moduleId](module, module.exports, __webpack_require__);

    // Return the exports of the module
    return module.exports;

  }

  /************************************************************************/
  /* webpack/runtime/define property getters */
  // 立即执行函数:声明了一个全局变量__webpack_require__.d。这是一个函数，作用是把某个模块的导出内容代理到模块的第二个参数__webpack_exports__。
  // __webpack_exports__ 和 __webpack_require__函数中return的 module.exports是同一个对象。
  (() => {
    // define getter functions for harmony exports
    __webpack_require__.d = (exports, definition) => {
      for (var key in definition) {
        if (__webpack_require__.o(definition, key) && !__webpack_require__.o(exports, key)) {
          Object.defineProperty(exports, key, { enumerable: true, get: definition[key] });

        }

      }

    };

  })();

  /* webpack/runtime/hasOwnProperty shorthand */
  (() => { // 查看对象是否有某个属性
    __webpack_require__.o = (obj, prop) => (Object.prototype.hasOwnProperty.call(obj, prop))

  })();

  /* webpack/runtime/make namespace object */
  (() => { // 给exports对象加标签和属性，表示exports是模块化对象
    // define __esModule on exports
    __webpack_require__.r = (exports) => {
      if (typeof Symbol !== 'undefined' && Symbol.toStringTag) {
        Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });

      }
      Object.defineProperty(exports, '__esModule', { value: true });

    };

  })();

  /*************************************加载入口文件***********************************/
  var __webpack_exports__ = {};
  // This entry need to be wrapped in an IIFE because it need to be isolated against other modules in the chunk.
  // 立即执行函数
  (() => {
    // 入口文件没有导出内容，因此是个空对象
    __webpack_require__.r(__webpack_exports__); // 标记导出对象
    // 导入依赖模块
    var _message_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(1); 
    // 模块主代码
    console.log(_message_js__WEBPACK_IMPORTED_MODULE_0__["default"]);
  })();


})()
  ;
```

上面的代码为打包结果，有完整易懂的中文注释。通过分析打包结果，可以得出以下结论：

1. 模块化代码打包后，并没有使用commonjs和esModule相关的模块化规范，而是自己仿照commonjs的模块化规范，用js实现了\_\_webpack_require\_\_和\_\_webpack_exports\_\_来达到模块的导入和导出功能。说白了就是**自己用javascript实现模块导入和导出,解决运行环境不支持某种模块化规范的问题。**
2. **利用函数的作用域，使各个模块的代码互不影响**，这也是为什么依赖图中用函数表示某个模块。
3. A模块要导入B模块，其实就是把B模块的导出对象作为参数传入A模块中。
4. 每个模块的导出内容都放在一个对象中，供其他模块导入。



从中也可以窥探出webpack的打包原理：

1. 生成依赖图
2. 实现一个通用的“模块化系统”，主要功能有三个：导出、导入、每个模块有独立的作用域。
3. 从入口文件开始运行代码。当遇到导入、导出时模块化系统能通过依赖图正确地处理。



当然本文的打包例子十分简单，没有涉及到模块缓存和循环引用的问题，旨在让读者理解webpack打包的核心原理。

至于webpack打包的核心原理实现，webpack官方文档中推荐了一个项目：[minipack][https://github.com/ronami/minipack]。这个项目通过简单的代码实现了webpack的核心打包逻辑，且有大量注释，推荐阅读。
