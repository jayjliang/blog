前言：

> webpack5的令人激动的新特性Module federation可能并不会让很多开发者激动，但是对于深受多应用伤害的腾讯文档来说，却是着实让人眼前一亮，这篇文章就带你了解腾讯文档的困境以及Module federation可以如何帮助我们走出这个困境。



## 0x1 腾讯文档的困境

### 1.1 多应用场景背景

腾讯文档从功能层面上来说，用户最熟悉的可能就是word、excel、ppt、表单这四个大品类，四个品类彼此独立，可能由不同的团队主要负责开发维护，那从开发者角度来说，四个品类四个仓库各自独立维护，好像事情就很简单，但是现实情况实际上却复杂很多。我们来看一个场景：

##### 通知中心的需求

![image-20200329125332068](https://pub.idqqimg.com/pc/misc/files/20200329/d90ed585ae566ec82052f2480c228042.png)

对于复杂的权限场景，为了让使用者能快速能获得最新状态，我们实际上有一个通知中心的需求，在pc的样式大致就是上图里面的样子。这里是在腾讯文档的列表页看到的入口，实际上在上面提到的四大品类里面，都需要嵌入这样的一个页面。

那么问题来了，为了最小化这里的开发和维护成本，肯定是各个品类公用一套代码是最好的，那最容易想到的就是使用独立npm包的方式来引入。确实，腾讯文档的内部很多功能现在是使用npm包的方式来引入的，但是实际上这里会遇到一些问题：

##### 问题一：历史代码

腾讯文档的历史很复杂，简而言之，在刚开始的时候，代码里面是不支持写ES6的，所以没办法引入npm包。一时半会想改造完成是不现实的，产品需求也不会等你去完成这样的改造

##### 问题二：发布效率

这里的问题实际上也是现在我们使用npm包的问题，其实还是我们懒，想投机取巧。以npm包的方式引入的话，一旦有改动，你需要改5个仓库（四个品类+列表页）去升级这里的版本，实际上发布成本是蛮大的，对于开发者来说其实也很痛苦

### 1.2 我们的解决方案

为了能在不支持ES6代码的环境下快速引入React来加速需求开发，我们想出了一个所谓的Script-Loader（下面会简称SL）的模式。

整体架构如图：

![image-20200329131110457](https://pub.idqqimg.com/pc/misc/files/20200329/45f6d7f4a870bf4b4525f331741fcd3d.png)

简单来说就是，参考jquery的引入方式，我们用另外一个项目去实现这些功能，然后把代码打包成ES5代码，对外提供很多接口，然后在各个品类页，引入我们提供的加载脚本，内部会自动去加载文件，获取每个模块的js文件的CDN地址并且加载。这样做到各个模块各自独立，并且所有模块和各个品类形成独立。

在这种模式下，每次发布，我们只需要去发布各个改动的模块以及最新的配置文件，其他品类就能获得自动更新。

这个模式并不一定适合所有项目，也不一定是最好的解决方案，从现在的角度来看，有点像微前端的概念，但是实际上却也是有区别的，这里就不展开了。这种模式目前确实能解决腾讯文档这种多应用复用代码的需求。

### 1.3 遇到的问题

这种模式本质上目前没有很严重的问题，但是有一个很痛点一直困扰我们，那就是品类代码和SL的代码共享问题。举个例子：

> Excel品类改造后使用了React，SL的模块A、模块B、模块C引入了React

因为SL的模块之间是各自独立的，所以React也是各自打包的，那就是说当你打开Excel的时候，如果你用了模块A、B、C，那你最终页面会加载四份React代码，虽然不会带上什么问题，但是对于有追求的前端来说，我们还是想去解决这样的问题。

##### 解决方案： External

对于React来说，我们可以默认品类是加载了React，所以我们直接把SL里面的React配置为External，这样就不会打包了，但是实际上情况没有这么简单：

###### 问题一：模块可能独立页面

就以上面的通知中心来说，在移动端上面就不是嵌入的了，而且独立页面，所以这个独立页面需要你手动引入React

###### 问题二：公共包不匹配

简单来说，就是SL依赖的包，在品类里面可能并没有使用，例如Mobx或者Redux

###### 问题三：不是所有包都可以直接配置External

这里的问题是说像React这种包我们可以通过配置External为window.React来达到共用，但是不是所有包都可以这样的，那对于不能配置为全局环境的包来说，还没法解决这里的代码共享问题

基于这些问题，我们目前的选择是一种折中方案，我们把可以配置全局环境的包提取出来，每个模块指明依赖，然后在SL内部，加载模块代码之前会去检测依赖，依赖加载完成才会加载执行实际模块代码。

这种方式有很大问题，你需要手动去维护这样的依赖，每个共享包实际上你都是需要单独打包成一个CDN文件，为的是当依赖检测失败的时候，可以有一个兜底加载文件。因此，实际上目前也只有React包做了这个共享。



**那么到这里，核心问题就变成了品类代码和SL如何做到`代码共享`。对于其他项目来说，其实也就是多应用如何做到`代码共享`。**



## 0x2 webpack的打包原理

为了解决上面的问题，我们实际上想从webpack入手，去实现这样的一个插件帮我们解决这个问题。核心思路就是hook webpack的内部require函数，在这之前我们先来看一下webpack打包后的一些原理，这个也是后面理解Module federation的核心。如果这里你比较熟悉，也可以快速跳过到第三节，但是不熟悉的同学还是建议认真了解一下。

### 2.1 chunk和module

webpack里面有两个很核心的概念，叫chunk和module，这里为了简单，只看js相关的，用笔者自己的理解去解释一下他们直接的区别：

> module：每一个源码js文件其实都可以看成一个module

> chunk：每一个打包落地的js文件其实都是一个chunk，每个chunk都包含很多module

默认的chunk数量实际上是由你的入口文件的js数量决定的，但是如果你配置动态加载或者提取公共包的话，也会生成新的chunk。

### 2.2 打包代码解读

有了基本理解后，我们需要去理解webpack打包后的代码在浏览器端是如何加载执行的。为此我们准备一个非常简单的demo，来看一下它的生成文件。

```js
src
---main.js
---moduleA.js
---moduleB.js

/**
* moduleA.js
*/
export default function testA() {
    console.log('this is A');
}


/**
* main.js
*/
import testA from './moduleA';

testA();

import('./moduleB').then(module => {

});

```

非常简单，入口js是`main.js`，里面就是直接引入`moduleA.js`，然后动态引入 `moduleB.js`，那么最终生成的文件就是两个chunk，分别是:

1. `main.js`和`moduleA.js`组成的`bundle.js`
2. ``moduleB.js`组成的`0.bundle.js`

如果你了解webpack底层原理的话，那你会知道这里是用mainTemplate和chunkTemplate分别渲染出来的，不了解也没关系，我们继续解读生成的代码

##### import变成了什么样

整个`main.js`的代码打包后是下面这样的

```js
(function (module, __webpack_exports__, __webpack_require__) {

    "use strict";
    __webpack_require__.r(__webpack_exports__);
    /* harmony import */
    var _moduleA__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__( /*! 		./moduleA */ "./src/moduleA.js");


    Object(_moduleA__WEBPACK_IMPORTED_MODULE_0__["default"])();

    __webpack_require__.e( /*! import() */ 0).then(__webpack_require__.bind(null, /*! ./moduleB 			*/ "./src/moduleB.js")).then(module => {

    });

})
```

可以看到，我们的直接import moduleA最后会变成webpack_require，而这个函数是webpack打包后的一个核心函数，就是解决依赖引入的。

##### webpack_require是怎么实现的

那我们看一下webpack_require它是怎么实现的：

```js
function __webpack_require__(moduleId) {
    // Check if module is in cache
    // 先检查模块是否已经加载过了，如果加载过了直接返回
    if (installedModules[moduleId]) {
        return installedModules[moduleId].exports;
    }
    // Create a new module (and put it into the cache)
    // 如果一个import的模块是第一次加载，那之前必然没有加载过，就会去执行加载过程
    var module = installedModules[moduleId] = {
        i: moduleId,
        l: false,
        exports: {}
    };
    // Execute the module function
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
    // Flag the module as loaded
    module.l = true;
    // Return the exports of the module
    return module.exports;
}
```

如果简化一下它的实现，其实很简单，就是每次require，先去缓存的installedModules这个缓存map里面看是否加载过了，如果没有加载过，那就从`modules`这个所有模块的map里去加载。

##### modules从哪里来的

那相信很多人都有疑问了，**modules这么个至关重要的map是从哪里来的呢**，我们把`bundle.js`生成的js再简化一下：

```js
(function (modules) {})({
    "./src/main.js": (function (module, __webpack_exports__, __webpack_require__) {}),
    "./src/moduleA.js": (function (module, __webpack_exports__, __webpack_require__) {})
});
```

所以可以看到，这其实是个立即执行函数，`modules`就是函数的入参，具体值就是我们包含的所有module，到此，一个chunk是如何加载的，以及chunk如何包含module，相信大家一定会有自己的理解了。

##### 动态引入如何操作呢

上面的chunk就是一个js文件，所以维护了自己的局部`modules`，然后自己使用没啥问题，但是动态引入我们知道是会生成一个新的js文件的，**那这个新的js文件`0.bundle.js`里面是不是也有自己的`modules`呢？那`bundle.js`如何知道`0.bundle.js`里面的`modules`呢**？

先看动态import的代码变成了什么样：

```
__webpack_require__.e( /*! import() */ 0).then(__webpack_require__.bind(null, /*! ./moduleB 			*/ "./src/moduleB.js")).then(module => {

});
```

从代码看，实际上就是外面套了一层webpck_require.e，然后这是一个promise，在then里面再去执行webpack_require。

实际上webpck_require.e就是去加载chunk的js文件`0.bundle.js`，具体代码就不贴了，没啥特别的。

等到加载回来后它认为**`bundle.js`里面的`modules`就一定会有了`0.bundle.js`包含的那些`modules`**，这是如何做到的呢？

我们看`0.bundle.js`到底是什么内容，让它拥有这样的魔力：

```js
(window["webpackJsonp"] = window["webpackJsonp"] || []).push(
    [
        [0],
        {
            "./src/moduleB.js": (function (module, __webpack_exports__, __webpack_require__) {})
        }
    ]
);
```

拿简化后的代码一看，大家第一眼想到的是jsonp，但是很遗憾的是它不是一个函数，却只是向一个全局数组里面push了自己的模块id以及对应的`modules`。那看起来魔法的核心应该是在`bundle.js`里面了，事实的确也是如此。

```js
var jsonpArray = window["webpackJsonp"] = window["webpackJsonp"] || [];
var oldJsonpFunction = jsonpArray.push.bind(jsonpArray);
jsonpArray.push = webpackJsonpCallback;
jsonpArray = jsonpArray.slice();
for(var i = 0; i < jsonpArray.length; i++) webpackJsonpCallback(jsonpArray[i]);
var parentJsonpFunction = oldJsonpFunction;
```

在`bundle.js`的里面，我们看到这么一段代码，其实就是说**我们劫持了push函数**，那`0.bundle.js`一旦加载完成，我们岂不是就会执行这里，那不就能拿到所有的参数，然后把`0.bundle.js`里面的所有module加到自己的`modules`里面去！

### 2.3 总结一下

如果你没有很理解，可以配合下面的图片，再把上面的代码读几遍。

![image-20200329143727089](https://pub.idqqimg.com/pc/misc/files/20200329/d2fe2f484b5720f50a7ea2a6fad9a8cc.png)

其实简单来说就是，对于mainChunk文件，我们维护一个`modules`这样的所有模块map，并且提供类似webpack_require这样的函数。对于chunkA文件（可能是因为提取公共代码生成的、或者是动态加载）我们就用类似jsonp的方式，让它把自己的所有`modules`添加到主chunk的`modules`里面去。



### 2.4 如何解决腾讯文档的问题？

基于这样的一个理解，我们就在思考，那腾讯文档的多应用代码共享能不能解决呢？

具体到腾讯文档的实际场景，就是如下图：

![image-20200329143446668](https://pub.idqqimg.com/pc/misc/files/20200329/7fb72a7f07d4cad3b41e5ebf02ea0db3.png)

因为是独立的项目，所以webpack打包也是有两个mainChunk，然后有各自的chunk（其实这里会有chunk覆盖或者chunk里面的mudule覆盖问题，所以id要采用md5）。

那问题的核心就是如何打通两个mainChunk的`modules`？

如果是自由编程，我想大家的实现方式可就太多了，但是在webpack的框架限制下面，如何快速的实现这个，我们也一直在思考方案，目前想到的方案如下：

> SL模块内部的webpack_require被我们hack，每次在`modules`里面找不到的时候，我们去Excel的`modules`里面去找，这样需要把Excel的`modules`作为全局变量

但是对于Excel不存在的模块我们需要怎么处理？

这种很明显就是运行时环境，我们需要做好加载时的失败降级处理，但是这样就会遇到同步转异步的问题，本来你是同步引入一个模块的，但是如果它在Excel的modules不存在的时候，你就需要先一步加载这个module对应的chunk，变成了类似动态加载，但是你的代码还是同步的，这样就会有问题。

所以我们需要将**依赖前置**，也就是说在加载SL模块后，它知道自己依赖哪些共享模块，然后去检测是否存在，不存在则依次去加载，所有依赖就位后才开始执行自己。

## 0x3 webpack5的Module federation

说实话，webpack底层还是很复杂的，在不熟悉的情况下而且定制程度也不能确定，所以我们也是迟迟没有去真正做这个事情。但是偶然的机会了解到了wbepack5的Module federation，通过看描述，感觉和我们想要的东西很像，于是我们开始一探究竟！

### 3.1 Module federation的介绍

关于Module federation是什么，有什么作用，现在已经有一些文章去说明，这里贴一篇，大家可以先去了解一下

[Module federation allows a JavaScript application to dynamically run code from another bundle/build, on both client and server](https://indepth.dev/webpack-5-module-federation-a-game-changer-in-javascript-architecture/)

简单来说就是允许运行时动态决定代码的引入和加载。

### 3.2 Module federation的demo

我们最关心的还是Module federation的的实现方式，才能决定它是不是真的适合腾讯文档。

这里我们用已有的demo：

[module-federation-examples/basic-host-remote](https://github.com/module-federation/module-federation-examples/tree/master/basic-host-remote)

在此之前，还是需要向大家介绍一下这个demo做的事情

```
app1
---index.js 入口文件
---bootstrap.js 启动文件
---App.js react组件

app2
---index.js 入口文件
---bootstrap.js 启动文件
---App.js react组件
---Button.js react组件
```

这是文件结构，其实你可以看成是两个独立应用app1和app2，那他们之前有什么爱恨情仇呢？

```js
/** app1 **/
/**
* index.js
**/
import('./bootstrap');

/**
* bootstrap.js
**/
import('./bootstrap');
import App from "./App";
import React from "react";
import ReactDOM from "react-dom";

ReactDOM.render(<App />, document.getElementById("root"));


/**
* App.js
**/
import('./bootstrap');
import React from "react";

import RemoteButton from 'app2/Button';

const App = () => (
  <div>
    <h1>Basic Host-Remote</h1>
    <h2>App 1</h2>
    <React.Suspense fallback="Loading Button">
      <RemoteButton />
    </React.Suspense>
  </div>
);

export default App;

```

我这里只贴了app1的js代码，app2的代码你不需要关心。代码没有什么特殊的，只有一点，app1的App.js里面：

```js
import RemoteButton from 'app2/Button';
```

也就是关键来了，跨应用复用代码来了！app1的代码用了app2的代码，但是这个代码最终长什么样？是如何引入app2的代码的？

### 3.3 Module federation的配置

先看我们的webpack需要如何配置：

```js
/**
 * app1/webpack.js
 */
{
    plugins: [
        new ModuleFederationPlugin({
            name: "app1",
            library: {
                type: "var",
                name: "app1"
            },
            remotes: {
                app2: "app2"
            },
            shared: ["react", "react-dom"]
        })
    ]
}
```

这个其实就是Module federation的配置了，大概能看到想表达的意思：

1. 用了远程模块app2，它叫app2
2. 用了共享模块，它叫shared

remotes和shared还是有一点区别的，我们先来看效果。

生成的html文件：

```html
<html>
  <head>
    <script src="app2/remoteEntry.js"></script>
  </head>
  <body>
    <div id="root"></div>
  <script src="app1/app1.js"></script><script src="app1/main.js"></script></body>
</html>
```

ps：这里的js路径有修改，这个是可以配置的，这里只是表明从哪里加载了哪些js文件



app1打包生成的文件：

```js
app1/index.html
app1/app1.js
app1/main.js
app1/react.js
app1/react-dom.js
app1/src_bootstrap.js
```

**ps: app2你也需要打包，只是我没有贴app2的代码以及配置文件，后面需要的时候会再贴出来的**



最终页面表现以及加载的js：

![image-20200329152614947](https://pub.idqqimg.com/pc/misc/files/20200329/4324ad92466e30764981541020a384b0.png)



从上往下加载的js时序其实是很有讲究的，后面将会是解密的关键：

```js
app2/remoteEntry.js
app1/app1.js
app1/main.js
app1/react.js
app1/react-dom.js
app2/src_button_js.js
app1/src_bootstrap.js
```



这里最需要关注的其实还是每个文件从哪里加载，在不去分析原理之前，看文件加载我们至少有这些结论：

1. remotes的代码自己不打包，类似external，例如app2/button就是加载app2打包的代码
2. shared的代码自己是有打包的

### Module federation的原理

在讲解原理之前，我还是放出之前的一张图，因为这是webpack的文件模块核心，即使升级5，也没有发生变化

![image-20200329152252834](https://pub.idqqimg.com/pc/misc/files/20200329/f194a736fb613663a0627da06838b54e.png)

app1和app2还是有自己的`modules`，所以实现的关键就是两个`modules`如何同步，或者说如何注入，那我们就来看看Module federation如何实现的。

##### 3.3.1 import变成了什么

```js
// import源码
import RemoteButton from 'app2/Button';

// import打包代码 在app1/src_bootstrap.js里面
/* harmony import */
var app2_Button__WEBPACK_IMPORTED_MODULE_1__ = __webpack_require__( /*! app2/Button */ "?ad8d");
/* harmony import */
var app2_Button__WEBPACK_IMPORTED_MODULE_1___default = /*#__PURE__*/ __webpack_require__.n(app2_Button__WEBPACK_IMPORTED_MODULE_1__);
```

从这里来看，我们好像看不出什么，因为还是正常的webpack_require，难道说它真的像我们之前所设想的那样，重写了webpack_require吗？

遗憾的是，从源码看这个函数是没有什么变化的，所以核心点不是这里。

但是你注意看加载的js顺序：



```js
app2/remoteEntry.js
app1/app1.js
app1/main.js
app1/react.js
app1/react-dom.js
app2/src_button_js.js // app2的button竟然先加载了，比我们的自己启动文件还前面
app1/src_bootstrap.js
```

回想上一节我们自己的分析

> 所以我们需要将依赖前置，也就是说在加载SL模块后，它知道自己依赖哪些共享模块，然后去检测是否存在，不存在依次去加载，所以依赖就位后才开始执行自己。

所以它是不是通过依赖前置来解决的呢？



##### 3.3.2 main.js文件内容

因为html里面和app1相关的只有两个文件：app1/app1.js以及app1/main.js

那我们看看main.js到底写了啥

```js
(() => { // webpackBootstrap
    var __webpack_modules__ = ({})

    var __webpack_module_cache__ = {};

    function __webpack_require__(moduleId) {

        if (__webpack_module_cache__[moduleId]) {
            return __webpack_module_cache__[moduleId].exports;
        }
        var module = __webpack_module_cache__[moduleId] = {
            exports: {}
        };
        __webpack_modules__[moduleId](module, module.exports, __webpack_require__);
        return module.exports;
    }
    __webpack_require__.m = __webpack_modules__;

    __webpack_require__("./src/index.js");
})()
```

可以看到区别不大，只是把之前的`modules`换成了`webpack_modules`，然后把这个`modules`的初始化由参数改成了内部声明变量。

那我们来看看webpack_modules内部的实现:

```js
var __webpack_modules__ = ({

    "./src/index.js": ((__unused_webpack_module, __unused_webpack_exports, __webpack_require__) => {
        __webpack_require__.e( /*! import() */ "src_bootstrap_js").then(__webpack_require__.bind(__webpack_require__, /*! ./bootstrap */ "./src/bootstrap.js"));
    }),

    "container-reference/app2": ((module) => {
        "use strict";
        module.exports = app2;
    }),

    "?8bfd": ((module, __unused_webpack_exports, __webpack_require__) => {
        "use strict";
        var external = __webpack_require__("container-reference/app2");
        module.exports = external;
    })
});
```

从代码看起来就三个module：

```js
./src/index.js 这个看起来就是我们的app1/index.js，里面去动态加载bootstrap.js对应的chunk src_bootstrap_js
container-reference/app2 直接返回一个全局的app2，这里感觉和我们的app2有关系
?8bfd 这个字符串是我们上面提到的app2/button对应的文件引用id
```

**那在加载src_bootstrap.js之前加载的那些react文件还有app2/button文件都是谁做的呢？通过debug，我们发现秘密就在webpack_require__.e("src_bootstrap_js")这句话**

在第二节解析webpack加载的时候，我们得知了：

> 实际上webpck_require.e就是去加载chunk的js文件`0.bundle.js`，等到加载回来后它认为`bundle.js`里面的`modules`就一定会有了`0.bundle.js`包含的那些`modules`

也就是说原来的webpack_require__.e平淡无奇，就是加载一个script，以致于我们都不想去贴出它的代码，但是这次升级后一切变的不一样了，它成了关键中的关键！



##### 3.3.3 webpack_require__.e做了什么

```js
__webpack_require__.e = (chunkId) => {
    return Promise.all(Object.keys(__webpack_require__.f).reduce((promises, key) => {
        __webpack_require__.f[key](chunkId, promises);
        return promises;
    }, []));
};
```

看代码，的确发生了变化，现在底层是去调用webpack_require.f上面的函数了，等到所有函数都执行完了，才执行promise的then

那问题的核心又变成了webpack_require.f上面有哪些函数了，最后发现有三个函数：

一：overridables

```js
/* webpack/runtime/overridables */
__webpack_require__.O = {};
var chunkMapping = {
    "src_bootstrap_js": [
        "?a75e",
        "?6365"
    ]
};
var idToNameMapping = {
    "?a75e": "react-dom",
    "?6365": "react"
};
var fallbackMapping = {
    "?a75e": () => {
        return __webpack_require__.e("vendors-node_modules_react-dom_index_js").then(() => () => __webpack_require__("./node_modules/react-dom/index.js"))
    },
    "?6365": () => {
        return __webpack_require__.e("vendors-node_modules_react_index_js").then(() => () => __webpack_require__("./node_modules/react/index.js"))
    }
};
__webpack_require__.f.overridables = (chunkId, promises) => {}
```

二：remotes

```js
/* webpack/runtime/remotes loading */
var chunkMapping = {
    "src_bootstrap_js": [
        "?ad8d"
    ]
};
var idToExternalAndNameMapping = {
    "?ad8d": [
        "?8bfd",
        "Button"
    ]
};
__webpack_require__.f.remotes = (chunkId, promises) => {}
```

三：jsonp

```js
/* webpack/runtime/jsonp chunk loading */
var installedChunks = {
    "main": 0
};


__webpack_require__.f.j = (chunkId, promises) => {}
```

这三个函数我把核心部分节选出来了，其实注释也写得比较清楚了，我还是解释一下：

1. overridables 可覆盖的，看代码你应该已经知道和shared配置有关
2. remotes 远程的，看代码非常明显是和remotes配置相关
3. jsonp 这个就是原有的加载chunk函数，对应的是以前的懒加载或者公共代码提取

##### 3.3.4 加载流程

知道了核心在webpack_require.e以及内部实现后，不知道你脑子里是不是对整个加载流程有了一定的思路，如果没有，容我来给你解析一下

1. 先加载src_main.js，这个没什么好说的，注入在html里面的
2. src_main.js里面执行webpack_require("./src/index.js")
3. src/index.js这个module的逻辑很简单，就是动态加载src_bootstrap_js这个chunk
4. 动态加载src_bootstrap_js这个chunk时，经过overridables，发现这个chunk依赖了react、react-dom，那就看是否已经加载，没有加载就去加载对应的js文件，地址也告诉你了
5. 动态加载src_bootstrap_js这个chunk时，经过remotes，发现这个chunk依赖了?ad8d，那就去加载这个js
6. 动态加载src_bootstrap_js这个chunk时，经过jsonp，就正常加载就好了
7. 所有依赖以及chunk都加载完成了，就去执行then逻辑：webpack_require src_bootstrap_js里面的module：./src/bootstrap.js

到此就一切都正常启动了，**其实就是我们之前提到的依赖前置，先去分析，然后生成配置文件，再去加载**。

看起来一切都很美好，但其实还是有一个关键信息没有解决！



##### 3.3.5 如何知道app2的存在

上面的第4步加载react的时候，因为我们自己实际上也打包了react文件，所以当没有加载的时候，我们可以去加载一份，也知道地址

但是第五步的时候，当页面从来没有加载过app2/Button的时候，我们去什么地址加载什么文件呢？

这个时候就要用到前面我们提到的main.js里面的`webpack_modules`了

```js
var __webpack_modules__ = ({
        
    "container-reference/app2": 
        ((module) => {
            "use strict";
            module.exports = app2;
        }),
        
    "?8bfd":
        ((module, __unused_webpack_exports, __webpack_require__) => {
        "use strict";
            var external = __webpack_require__("container-reference/app2");
            module.exports = external;
        })
});
```

这里面有三个module，我们还有 **?8bfd、container-reference/app2** 没有用到，我们再看一下remotes的实现

```js
/* webpack/runtime/remotes loading */
var chunkMapping = {
    "src_bootstrap_js": [
        "?ad8d"
    ]
};
var idToExternalAndNameMapping = {
    "?ad8d": [
        "?8bfd",
        "Button"
    ]
};
__webpack_require__.f.remotes = (chunkId, promises) => {
    if (__webpack_require__.o(chunkMapping, chunkId)) {
        chunkMapping[chunkId].forEach((id) => {
            if (__webpack_modules__[id]) return;
            var data = idToExternalAndNameMapping[id];
            promises.push(Promise.resolve(__webpack_require__(data[0]).get(data[1])).then((factory) => {
                __webpack_modules__[id] = (module) => {
                    module.exports = factory();
                }
            }))
        });
    }
}
```

当我们加载src_bootstrap_js这个chunk时，经过remotes，发现这个chunk依赖了?ad8d，那在运行时的时候：

```js
id = "?8bfd"
data = [
   "?8bfd",
   "Button"
]
// 源码
__webpack_require__(data[0]).get(data[1])
// 运行时
__webpack_require__('?8bfd').get("Button")
```

结合main.js的module ?8bfd的代码，那最终就是app2.get("Button")

这不就是个全局变量吗？看起来有些蹊跷啊！

##### 3.3.6 再看app2/remoteEntry.js

我们好像一直忽略了这个文件，它是第一个加载的，必然有它的作用，带着对全局app2有什么蹊跷的疑问，我们去看了这个文件，果然发现了玄机！

```js
var app2;
app2 =
    (() => {
        "use strict";
        var __webpack_modules__ = ({
            "?8619": ((__unused_webpack_module, exports, __webpack_require__) => {
                var moduleMap = {
                    "Button": () => {
                        return __webpack_require__.e("src_Button_js").then(() => () => __webpack_require__( /*! ./src/Button */ "./src/Button.js"));
                    }
                };
                var get = (module) => {
                    return (
                        __webpack_require__.o(moduleMap, module) ?
                        moduleMap[module]() :
                        Promise.resolve().then(() => {
                            throw new Error("Module " + module + " does not exist in container.");
                        })
                    );
                };
                var override = (override) => {
                    Object.assign(__webpack_require__.O, override);
                }

                __webpack_require__.d(exports, {
                    get: () => get,
                    override: () => override
                });
            })
        });
        return __webpack_require__("?8619");
    })()
```

如果你细心看，就会发现，这个文件定义了全局的app2变量，然后提供了一个get函数，里面实际上就是去加载具体的模块

所以app2.get("Button")在这里就变成了app2内部定义的get函数，随后执行自己的webpack_require

是不是有种焕然大悟的感觉！

原来它是这样在两个独立打包的应用之间，通过全局变量去建立了一座彩虹桥！

当然，app2/remoteEntry.js是由app2根据配置打包出来的，里面实际上就是根据配置文件的导出模块，生成对应的内部`modules`



##### 你可能忽略的bootstrap.js

细心的读者如果注意的话，会发现，在入口文件index.js和真正的文件app.js之间多了一个bootstrap.js，而且里面内容就是异步加载app.js

那这个文件是不是多余的，笔者试了一下，直接把入口换成app.js或者这里换成同步加载，整个应用就跑不起来了

其实从原理上分析后也是可以理解的：

因为依赖需要前置，并且等依赖加载完成后才能执行自己的入口文件，如果不把入口变成一个异步的chunk，那如何去实现这样的依赖前置呢？毕竟实现依赖前置加载的核心是webpack_require.e

##### 3.3.7 总结

至此，Module federation如何实现shared和remotes两个配置我相信大家都有了理解了，其实还是逃不过在第二节末尾说的问题：

1. **如何解决依赖问题，这里的实现方式是重写了加载chunk的webpack_require.e，从而前置加载依赖**

2. **如何解决`modules`的共享问题，这里是使用全局变量来hook**

整体看起来实现还是挺巧妙的，不是webpack核心开发者，估计不能想到这样解决，实际上改动也是蛮大的。

这种实现方式的优缺点其实也明显：

优点：做到代码的运行时加载，而且shared代码无需自己手动打包

缺点：对于其他应用的依赖，实际上是强依赖的，也就是说app2有没有按照接口实现，你是不知道的

至于网上一些其他文章所说的app2的包必须在代码里面异步使用，这个你看前面的demo以及知道原理后也知道，根本没有这样的限制！



## 0x4 总结

对于腾讯文档来说，实际上更需要的是目前的shared能力，对一些常见的公共依赖库配置shared后就可以解决了，但是也只是理想上的，实际上还是会遇到一些可见的问题，例如：

1. 不同的版本生成的公共库id不同，还是会导致重复加载
2. app2的remotEntry更新后如何获取最新地址
3. 如何获知其他应用导出接口

但是至少带来了解决这个问题的希望，remotes配置也让我们看到了多应用共享代码的可能，所以还是会让人眼前一亮，期待webpack5的正式发布！

最后，如果有写的不正确的地方，欢迎斧正～

