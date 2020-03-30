<p></p>
<p>&nbsp; &nbsp; 直出现在已经是性能优化的标配了，关于直出的好处我相信大家都已经很了解了。进行直出的缘由除了直出带来的性能优化外，还有良好的SEO，当然，也是为了提高考核分数。简单来说，直出分为两种，分别是数据直出和同构直出，本次直出的限定范围是同构直出。</p>
<p>&nbsp; &nbsp; vue的直出其实官方已经给出了比较详细的<a href="https://ssr.vuejs.org/zh/" target="_blank" rel="noopener">文档</a>和一个比较全的<a href="https://github.com/vuejs/vue-hackernews-2.0" target="_blank" rel="noopener">demo</a>，社区也有一个开箱即用的方案<a href="https://nuxtjs.org" target="_blank" rel="noopener">Nuxt.js</a>。官方demo是vue全家桶（vue+vuex+vue-router）的直出方案，直出部分也是运用了直出端的很多插件，然而很多时候我们自己的项目也许用不到这么多东西，如果之前你对vue直出不了解的话，直接参考他们的demo做起来会感觉很麻烦，而网上关于简单直出的demo或者文章又不多。本篇文章就将从最简单的直出方式说起，直到做到和官方demo一样的直出方式。</p>
<p>&nbsp; &nbsp; 首先来看群这边项目的技术栈，由于群这边的业务单页面居多，所以没有使用vue-router，很多项目就是单纯的基于vue，甚至没有使用vuex。经过尝试后，我们发现配合vuex，直出工作会变得简单很多（从后面的直出说明大家会有体会），所以最终技术栈是vue+vuex。</p>
<p>&nbsp; &nbsp; 对项目的直出改造大概分为三个部分，分别为前端，构建以及后端。接下来的几种直出方式里面会对这三个部分需要进行的改动进行说明。</p>
<p>&nbsp; &nbsp; 首先我们需要对前端进行改造。改造前的文件结构如下:</p>
<div class="km_insert_code">
<pre><code>.index
|   index.vue
|   main.js
|   main.html
+---css
+---img
+---_components<br /></code></pre>
</div>
<p>&nbsp;改造后的文件结构如下：</p>
<div class="km_insert_code">
<pre><code>.index<br />|   app.js
|   index.vue
|   main.js
|   main.html<br />|   server.js
+---css
+---img
+---_components<br /><span style="color: #f8f8f2; font-family: Consolas, Monaco, monospace; font-size: 14px; white-space: pre; background-color: #23241f;">+---store</span><br /></code></pre>
</div>
<p>&nbsp; &nbsp; &nbsp;我们可以看到主要的变化是抽象出app.js，同时增加服务器端入口文件server.js以及引入store层。那app.js中到底应该写入什么内容呢，我们看下面的代码对比：</p>
<p>改造前的main.js:</p>
<div class="km_insert_code">
<pre><code><span class="hljs-keyword">import</span> Vue <span class="hljs-keyword">from</span> <span class="hljs-string">'vue'</span>
<span class="hljs-keyword">import</span> Main <span class="hljs-keyword">from</span> <span class="hljs-string">'./index.vue'</span>
<span class="hljs-keyword">import</span> AlloyFinger <span class="hljs-keyword">from</span> <span class="hljs-string">'../../common/alloy_finger'</span>;
Vue.use(AlloyFinger);

<span class="hljs-keyword">new</span> Vue({
    <span class="hljs-attr">el</span>: <span class="hljs-string">'#main'</span>,
    <span class="hljs-attr">render</span>: <span class="hljs-function"><span class="hljs-params">h</span> =&gt;</span> h(Main)
});</code></pre>
</div>
<p>&nbsp;改造后的app.js:</p>
<div class="km_insert_code">
<pre><code><span class="hljs-keyword">import</span> Vue <span class="hljs-keyword">from</span> <span class="hljs-string">'vue'</span>
<span class="hljs-keyword">import</span> Main <span class="hljs-keyword">from</span> <span class="hljs-string">'./index.vue'</span>
<span class="hljs-keyword">import</span> { createStore } <span class="hljs-keyword">from</span> <span class="hljs-string">'./store/store'</span>

<span class="hljs-keyword">import</span> AlloyFinger <span class="hljs-keyword">from</span> <span class="hljs-string">'../../common/alloy_finger'</span>;
Vue.use(AlloyFinger);

<span class="hljs-keyword">export</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">init</span>(<span class="hljs-params"></span>) </span>{
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"call init"</span>)
    <span class="hljs-keyword">const</span> store = createStore()

    <span class="hljs-keyword">const</span> app = <span class="hljs-keyword">new</span> Vue({
        store,
        <span class="hljs-attr">render</span>: <span class="hljs-function"><span class="hljs-params">h</span> =&gt;</span> h(Main)
    })

    <span class="hljs-keyword">return</span> { app, store }
}</code></pre>
</div>
<p>&nbsp; &nbsp; &nbsp;从代码对比我们可以看到，app.js主要是改造于之前的main.js，但是去除了挂载el元素，同时引入使用了vuex的store层，最后暴露出了app和store对象。</p>
<p>&nbsp; &nbsp; 接下来我们看看改造后的main.js是怎样的：</p>
<div class="km_insert_code">
<pre><code><span class="hljs-keyword">import</span> { init } <span class="hljs-keyword">from</span> <span class="hljs-string">'./app'</span>
<span class="hljs-keyword">const</span> { app, store } = init()
app.$mount(<span class="hljs-string">'#main'</span>)
<span class="hljs-keyword">if</span>(<span class="hljs-built_in">window</span>.__INITIAL_STATE__) {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"init state"</span>)
    store.replaceState(<span class="hljs-built_in">window</span>.__INITIAL_STATE__);
} <span class="hljs-keyword">else</span> {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"no state"</span>);
    store.dispatch(<span class="hljs-string">'loadData'</span>);
}</code></pre>
</div>
<p>&nbsp; &nbsp; &nbsp;可以看到，核心逻辑就是调用mount方法，同时根据是否有后端插入到前端的数据来决定是否请求首屏cgi。这里在后端已经插入数据的情况下，我们使用vuex的replaceState方法来初始化store，如果这里没有引入vuex，可能我们就要在首屏cgi里面去做这个操作。</p>
<p>&nbsp; &nbsp; 最后我们看看main.html是如何进行改造的：</p>
<div class="km_insert_code">
<pre><code>改造前：
<span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span>
<span class="hljs-tag">    &lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"main"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">src</span>=<span class="hljs-string">"//open.mobile.qq.com/sdk/qqapi.js?_bid=152"</span>&gt;</span><span class="undefined"></span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">src</span>= <span class="hljs-string">"//s.url.cn/pub/js/alloyreport.js?_bid=2231"</span> &gt;</span><span class="undefined"></span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">src</span>=<span class="hljs-string">"js/index.js"</span>&gt;</span><span class="undefined"></span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">body</span>&gt;</span>
改造后：
<span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"main"</span>&gt;</span><span class="hljs-comment">&lt;!--vue-ssr-outlet--&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">src</span>=<span class="hljs-string">"//open.mobile.qq.com/sdk/qqapi.js?_bid=152"</span>&gt;</span><span class="undefined"></span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">src</span>= <span class="hljs-string">"//s.url.cn/pub/js/alloyreport.js?_bid=2231"</span> &gt;</span><span class="undefined"></span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">src</span>=<span class="hljs-string">"js/index.js"</span>&gt;</span><span class="undefined"></span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">body</span>&gt;</span></code></pre>
</div>
<p>&nbsp; &nbsp; &nbsp;这里实际上工作很简单，加了个占位符&lt;! --vue-ssr-outlet--&gt;来告诉vue应该把直出生成的html放到哪里去。</p>
<p>&nbsp; &nbsp; 到这里，我们实际上就可以开始进行直出了，那接下来就进行第一种直出。</p>
<p>&nbsp; &nbsp; 首先我们需要对构建进行改造，主要目的是对server.js进行打包，一般做法是增加webpack.node.js配置文件。我们看看构建的核心逻辑：</p>
<div class="km_insert_code">
<pre><code><span class="hljs-keyword">var</span> baseConfig = {
    <span class="hljs-attr">context</span>: configWebpack.path.src,
    <span class="hljs-attr">entry</span>: configWebpack.nodeEntry,
    <span class="hljs-attr">output</span>: {
        <span class="hljs-attr">path</span>: isProduction ? path.join(configWebpack.path.dist, <span class="hljs-string">'node'</span>) : path.join(configWebpack.path.dev, <span class="hljs-string">'node'</span>),
        <span class="hljs-attr">filename</span>: <span class="hljs-string">'[name].js'</span>
    },
    <span class="hljs-attr">target</span>: <span class="hljs-string">'node'</span>,
    <span class="hljs-attr">node</span>: {
        <span class="hljs-attr">__filename</span>: <span class="hljs-literal">false</span>,
        <span class="hljs-attr">__dirname</span>: <span class="hljs-literal">false</span>
    }
｝</code></pre>
</div>
<p>&nbsp; &nbsp; &nbsp;需要注意的是我们需要指定target为node，毕竟这个文件最后是使用node去运行的。这里我们推荐使用我们团队成员heyli开发的steamer-vue脚手架进行开发，内建对直出的支持，非常好用！</p>
<p>&nbsp; &nbsp; 接下来我们来看server.js的内容：</p>
<div class="km_insert_code">
<pre><code><span class="hljs-built_in">module</span>.exports = <span class="hljs-function"><span class="hljs-keyword">function</span>* (<span class="hljs-params">req, res</span>) </span>{
    <span class="hljs-comment">// 使用vue-server-renderer.createRenderer来进行直出<br /></span>    <span style="color: #75715e; font-family: Consolas, Monaco, monospace; font-size: 14px; white-space: pre; background-color: #23241f;">//</span> <font color="#75715e">我们需要的是html文件内容和app,store对象</font>
    <span class="hljs-keyword">const</span> { createRenderer } = <span class="hljs-built_in">require</span>(<span class="hljs-string">'vue-server-renderer'</span>)
    <span class="hljs-keyword">const</span> apps = <span class="hljs-built_in">require</span>(<span class="hljs-string">'../../page/index/app'</span>)
    <span class="hljs-keyword">var</span> fileContent = <span class="hljs-string">''</span>;
    fileContent = <span class="hljs-built_in">require</span>(<span class="hljs-string">'../../../dist/index.html'</span>);
   
    <span class="hljs-keyword">let</span> info = fileContent.split(<span class="hljs-string">'&lt;!--vue-ssr-outlet--&gt;'</span>)
    <span class="hljs-keyword">const</span> { app, store } = apps.init();
    <span class="hljs-keyword">const</span> getData = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
        <span class="hljs-keyword">return</span> store.dispatch(<span class="hljs-string">'loadData'</span>, req.header)
    }
    <span class="hljs-keyword">const</span> renderer = createRenderer({}, {
        <span class="hljs-attr">template</span>: fileContent
    })

    <span class="hljs-keyword">var</span> renderHTML = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function">(<span class="hljs-params">resolve, reject</span>) =&gt;</span> {
            getData()
                .then(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">data</span>) </span>{
                    app.state = <span class="hljs-built_in">JSON</span>.stringify(store.state)
                    renderer.renderToString(app, (err, html) =&gt; {
                        <span class="hljs-keyword">if</span> (err) {
                            <span class="hljs-keyword">throw</span> err;
                        }
                        resolve(info[<span class="hljs-number">0</span>] + html + info[<span class="hljs-number">1</span>])
                    });
                })
                .catch(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">data</span>) </span>{
                    reject(info[<span class="hljs-number">0</span>] + info[<span class="hljs-number">1</span>])
                })
        })
    }
    res.set(<span class="hljs-string">'Content-Type'</span>, <span class="hljs-string">'text/html'</span>);
    res.set(<span class="hljs-string">'Set-Cookie'</span>, req.headers.cookie);
    <span class="hljs-keyword">const</span> html = <span class="hljs-keyword">yield</span> renderHTML()
    res.body = html
};</code></pre>
</div>
<p>&nbsp; &nbsp; &nbsp;这里的server.js已经不仅仅是服务端入口js了，同时也是玄武需要使用的路由中间件了。我们对这个sever.js进行打包，然后直接丢到IDC机器上，启动玄武，配置好路由文件，就可以直出了。</p>
<p>&nbsp; &nbsp; 这种直出方式优点很明显，所有需要的文件全部打包在一起，最后输出只有一个文件，部署起来很方便，构建改动相对也比较小。但是这也是缺点，实际上vue-server-renderer是不推荐打包进来的，这样文件体积会增大。另外一个缺点是路由中间件代码不具有通用性，里面混合了业务逻辑代码。</p>
<p>&nbsp; &nbsp; 那么接下来我们就尝试第二种直出方式。</p>
<p>&nbsp; &nbsp; 首先来看前端部分的改造。对于server.js，我们要做的第一件事就是代码拆分，我们希望把业务相关的代码放在这里面，路由中间间放到router.js里面去。这样router.js就能做到通用。看看改造后的server.js:</p>
<div class="km_insert_code">
<pre><code><span class="hljs-meta">'use strict'</span>;
<span class="hljs-keyword">import</span> { init } <span class="hljs-keyword">from</span> <span class="hljs-string">'./app'</span>

<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> context =&gt; {
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function">(<span class="hljs-params">resolve, reject</span>) =&gt;</span> {
        <span class="hljs-keyword">const</span> { app, store } = init()
        store.dispatch(<span class="hljs-string">'loadData'</span>, context)
            .then(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">data</span>) </span>{
                context.state = store.state;
                resolve(app);
            }).catch(reject);
    });
}</code></pre>
</div>
<p>&nbsp; &nbsp; &nbsp;首屏cgi请求我们拆分到server.js中，同时将state变量赋值到context中去，这样页面就会自动注入init_state。</p>
<p>构建端：</p>
<div class="km_insert_code">
<pre><code><span class="hljs-keyword">var</span> baseConfig = {
    <span class="hljs-attr">context</span>: configWebpack.path.src,
    <span class="hljs-attr">entry</span>: configWebpack.nodeEntry,
    <span class="hljs-attr">output</span>: {
        <span class="hljs-attr">path</span>: isProduction ? path.join(configWebpack.path.dist, <span class="hljs-string">'node'</span>) : path.join(configWebpack.path.dev, <span class="hljs-string">'node'</span>),
        <span class="hljs-attr">filename</span>: <span class="hljs-string">'[name].js',</span>
        <font color="#f92672">libraryTarget</font>: <span class="hljs-string">'commonjs2'</span>
    },
    <span class="hljs-attr">target</span>: <span class="hljs-string">'node'</span>,
    <span class="hljs-attr">node</span>: {
        <span class="hljs-attr">__filename</span>: <span class="hljs-literal">false</span>,
        <span class="hljs-attr">__dirname</span>: <span class="hljs-literal">false</span>
    }
｝</code></pre>
</div>
<p>&nbsp; &nbsp; &nbsp;这里相对于第一种直出方式的构建而言，只是增加了libraryTarget这一个配置，因为打包出来的js是需要在node端进行require的。</p>
<p>最后我们来看看服务端需要使用的router.js:</p>
<div class="km_insert_code">
<pre><code><span class="hljs-meta">'use strict'</span>
<span class="hljs-keyword">const</span> fs = <span class="hljs-built_in">require</span>(<span class="hljs-string">'fs'</span>)
<span class="hljs-keyword">const</span> path = <span class="hljs-built_in">require</span>(<span class="hljs-string">'path'</span>)
<span class="hljs-keyword">const</span> resolve = <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">file</span>) </span>{
    <span class="hljs-keyword">return</span> path.resolve(__dirname, file)
}<br /><span style="color: #f8f8f2; font-family: Consolas, Monaco, monospace; font-size: 14px; white-space: pre; background-color: #23241f;"> </span><span class="hljs-comment" style="color: #75715e; font-family: Consolas, Monaco, monospace; font-size: 14px; white-space: pre;">// 使用vue-server-renderer.<span style="color: #f8f8f2; font-family: Consolas, Monaco, monospace; font-size: 14px; white-space: pre; background-color: #23241f;">createBundleRenderer</span>来进行直出<br /></span><span style="color: #f8f8f2; font-family: Consolas, Monaco, monospace; font-size: 14px; white-space: pre; background-color: #23241f;"> </span><span style="color: #75715e; font-family: Consolas, Monaco, monospace; font-size: 14px; white-space: pre; background-color: #23241f;">//</span><span style="color: #f8f8f2; font-family: Consolas, Monaco, monospace; font-size: 14px; white-space: pre; background-color: #23241f;"> </span><font color="#75715e" style="font-family: Consolas, Monaco, monospace; font-size: 14px; white-space: pre;">我们需要的是html文件内容和app,store对象</font>
<span class="hljs-keyword">const</span> createBundleRenderer = <span class="hljs-built_in">require</span>(<span class="hljs-string">'vue-server-renderer'</span>).createBundleRenderer

global.window = <span class="hljs-literal">undefined</span>
global.mqq = <span class="hljs-literal">undefined</span>

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">createRender</span>(<span class="hljs-params">bundle, options</span>) </span>{
    <span class="hljs-keyword">return</span> createBundleRenderer(bundle, <span class="hljs-built_in">Object</span>.assign(options, {
        <span class="hljs-attr">runInNewContext</span>: <span class="hljs-literal">false</span>
    }));
}

<span class="hljs-keyword">let</span> renderer, template, bundle

template = fs.readFileSync(resolve(<span class="hljs-string">'../webserver/index.html'</span>), <span class="hljs-string">'utf-8'</span>);
bundle = fs.readFileSync(resolve(<span class="hljs-string">'./index.js'</span>), <span class="hljs-string">'utf-8'</span>);
renderer = createRender(bundle, {
    template
})


<span class="hljs-built_in">module</span>.exports = <span class="hljs-function"><span class="hljs-keyword">function</span> *(<span class="hljs-params">req, res</span>) </span>{<br />    // 传入header和url，这样我们可以在直出时在前端拿到这些信息
    <span class="hljs-keyword">let</span> context = {
        <span class="hljs-attr">headers</span>: req.header,
        <span class="hljs-attr">url</span>: req.originalUrl,
    }
    res.set(<span class="hljs-string">'Content-Type'</span>, <span class="hljs-string">'text/html'</span>)
    <span class="hljs-keyword">var</span> renderHTML = <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function">(<span class="hljs-params">resolve, reject</span>) =&gt;</span> {
            renderer.renderToString(context, (err, html) =&gt; {
                <span class="hljs-keyword">if</span> (err) {
                    <span class="hljs-comment">// throw err;</span>
                    <span class="hljs-built_in">console</span>.log(err)
                    resolve(template)
                }
                resolve(html)
            });
        })
    }
    <span class="hljs-keyword">const</span> html = <span class="hljs-keyword">yield</span> renderHTML()
    res.body = html
}
</code></pre>
</div>
<p>&nbsp; &nbsp; &nbsp;router.js的改动还是比较大的，首先，我们使用createBundleRenderer来进行直出，直出方式一是使用createRenderer来进行直出的，那么使用createBundleRenderer有哪些好处呢，下面摘录<a href="https://ssr.vuejs.org/zh/bundle-renderer.html" target="_blank" rel="noopener">官方文档</a>的说明：</p>
<ul style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: antialiased; font-size: 15px; margin-top: 1.2em; margin-bottom: 1.2em; padding: 0px 0px 0px 1.5em; word-spacing: 0.05em; line-height: 1.6em; color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif;">
<li style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: antialiased; font-size: inherit;">内置的 source map 支持（在 webpack 配置中使用&nbsp;<code style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-family: 'Roboto Mono', Monaco, courier, monospace; font-size: 0.85em; background-color: #f8f8f8; color: inherit; padding: 0.2em; margin: 0px; border-radius: 2px; white-space: nowrap; break-inside: avoid; direction: ltr; border: none;">devtool: 'source-map'</code>）</li>
<li style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: antialiased; font-size: inherit;">在开发环境甚至部署过程中热重载（通过读取更新后的 bundle，然后重新创建 renderer 实例）</li>
<li style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: antialiased; font-size: inherit;">关键 CSS(critical CSS) 注入（在使用&nbsp;<code style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-family: 'Roboto Mono', Monaco, courier, monospace; font-size: 0.85em; background-color: #f8f8f8; color: inherit; padding: 0.2em; margin: 0px; border-radius: 2px; white-space: nowrap; break-inside: avoid; direction: ltr; border: none;">*.vue</code>&nbsp;文件时）：自动内联在渲染过程中用到的组件所需的CSS。更多细节请查看&nbsp;<a href="https://ssr.vuejs.org/zh/css.html" style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: antialiased; text-decoration: none; color: #42b983; font-weight: 600; font-size: inherit; background: 0px 0px;">CSS</a>&nbsp;章节。</li>
<li style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: antialiased; font-size: inherit;">使用&nbsp;<a href="https://ssr.vuejs.org/zh/api.html#clientmanifest" style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: antialiased; text-decoration: none; color: #42b983; font-weight: 600; font-size: inherit; background: 0px 0px;">clientManifest</a>&nbsp;进行资源注入：自动推断出最佳的预加载(preload)和预取(prefetch)指令，以及初始渲染所需的代码分割 chunk。</li>
</ul>
<p><font color="#333333"><span style="font-size: 15px; word-spacing: 0.75px;">&nbsp; &nbsp; 其中第三点关键css的提取我想很多人还是很感兴趣的，<span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">它可以在 HTML 中收集和内联（使用&nbsp;</span><code style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-family: 'Roboto Mono', Monaco, courier, monospace; font-size: 0.85em; background-color: #f8f8f8; color: #333333; padding: 0.2em; margin: 0px; border-radius: 2px; white-space: nowrap; break-inside: avoid; direction: ltr; border: none; orphans: 3; widows: 3; word-spacing: 0.75px;">template</code><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">&nbsp;选项时自动处理）组件的 CSS。在客户端，当第一次使用该组件时，</span><code style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-family: 'Roboto Mono', Monaco, courier, monospace; font-size: 0.85em; background-color: #f8f8f8; color: #333333; padding: 0.2em; margin: 0px; border-radius: 2px; white-space: nowrap; break-inside: avoid; direction: ltr; border: none; orphans: 3; widows: 3; word-spacing: 0.75px;">vue-style-loader</code><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">&nbsp;会检查这个组件是否已经具有服务器内联(server-inlined)的 CSS - 如果没有，CSS 将通过&nbsp;</span><code style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-family: 'Roboto Mono', Monaco, courier, monospace; font-size: 0.85em; background-color: #f8f8f8; color: #333333; padding: 0.2em; margin: 0px; border-radius: 2px; white-space: nowrap; break-inside: avoid; direction: ltr; border: none; orphans: 3; widows: 3; word-spacing: 0.75px;">&lt;style&gt;</code><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">&nbsp;标签动态注入。</span></span></font></p>
<p><font color="#333333"><span style="font-size: 15px; word-spacing: 0.75px;">&nbsp; &nbsp; 具体的配置<a href="https://ssr.vuejs.org/zh/css.html" target="_blank" rel="noopener">官方文档</a>也是比较详细的。</span></font></p>
<p>&nbsp; &nbsp; createBundleRenderer还有一个好处是它支持runInNewContext配置。<a href="https://ssr.vuejs.org/zh/api.html#createrendereroptions" target="_blank" rel="noopener">官方说明</a>是：</p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">&nbsp; &nbsp; 默认情况下，对于每次渲染，bundle renderer 将创建一个新的 V8 上下文并重新执行整个 bundle。这具有一些好处 - 例如，应用程序代码与服务器进程隔离，我们无需担心文档中提到的</span><a href="https://ssr.vuejs.org/zh/structure.html#avoid-stateful-singletons" style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: antialiased; text-decoration: none; color: #42b983; font-weight: 600; font-size: 15px; background-image: initial; background-position: 0px 0px; background-size: initial; background-repeat: initial; background-attachment: initial; background-origin: initial; background-clip: initial; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; orphans: 3; widows: 3; word-spacing: 0.75px;">状态单例问题</a><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">。然而，这种模式有一些相当大的性能开销，因为重新创建上下文并执行整个 bundle 还是相当昂贵的，特别是当应用很大的时候。</span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">&nbsp; &nbsp; 出于向后兼容的考虑，此选项默认为true，但建议你尽可能使用false或者once配置。</span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">&nbsp; &nbsp; 回到我们的直出，第二种直出方式它是自动化的，自动帮我们插入html和注入init_state，我们的服务器端路由中间件router.js也更具有通用性，整体结清晰。当然它也有一些缺点，html文件是没有打包的，你需要自己发布到IDC机器上，同时相关的node_modules也需要你进行打包发布至少一次。</span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">&nbsp; &nbsp; 到这里可能有人会感觉这样应该差不多了吧，其实不然，官方直出库能做的事实际上比我们想象的还要多，接下来我们看第三种直出方式，这种直出方式也是和官方demo基本上一模一样的。</span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">&nbsp; &nbsp; 首先看前端，然而前端在第二种直出的基础上我们不需要任何改变。那我们看构建端，首先是webpack.client.js：</span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><img src="http://km.oa.com/files/photos/captures/201709/1505023471_92_w807_h278.png" /></span></p>
<p style="orphans: 3; widows: 3;"><font color="#333333"><span style="font-size: 15px; word-spacing: 0.75px;">webpack.server.js：</span></font></p>
<p style="orphans: 3; widows: 3;"><font color="#333333"><span style="font-size: 15px; word-spacing: 0.75px;"><img src="http://km.oa.com/files/photos/captures/201709/1505023497_81_w717_h434.png" /></span></font></p>
<p style="orphans: 3; widows: 3;"><font color="#333333"><span style="font-size: 15px; word-spacing: 0.75px;">&nbsp; &nbsp; 其实主要改变就是引入client-plugin和server-plugin。上面两个构建是我从官方demo中精简出来的，所以入口文件和打包文件的写法和第二种自出方式略有差异。关于这两个插件我们同样可以参考<a href="https://ssr.vuejs.org/zh/api.html#webpack-plugins" target="_blank" rel="noopener">官方文档</a>：</span></font></p>
<p style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: antialiased; font-size: 15px; orphans: 3; widows: 3; margin-top: 1.2em; margin-bottom: 1.2em; word-spacing: 0.05em; line-height: 1.6em; color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif;">webpack 插件作为独立文件提供，并且应当直接 require：</p>
<pre style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-family: 'Roboto Mono', Monaco, courier, monospace; font-size: 15px; background: #f8f8f8; break-inside: avoid; direction: ltr; margin-top: 0px; margin-bottom: 0px; padding: 1.2em 1.4em; border: none; color: #333333; overflow: auto; word-wrap: normal; line-height: 1.5em;"><code class="lang-js" style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-family: 'Roboto Mono', Monaco, courier, monospace; font-size: 0.8em; background-position: 0px 0px; background-color: initial; color: inherit; padding: 0px; margin: 0px; border-radius: 2px; position: relative; break-inside: avoid; direction: ltr; border-top: none; border-right: none; border-bottom: none; border-image: initial; max-width: initial; overflow: initial; line-height: inherit;"><span class="hljs-keyword" style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-size: inherit; color: #8959a8;">const</span> VueSSRServerPlugin = <span class="hljs-built_in" style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-size: inherit; color: #f5871f;">require</span>(<span class="hljs-string" style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-size: inherit; color: #718c00;">'vue-server-renderer/server-plugin'</span>)
<span class="hljs-keyword" style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-size: inherit; color: #8959a8;">const</span> VueSSRClientPlugin = <span class="hljs-built_in" style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-size: inherit; color: #f5871f;">require</span>(<span class="hljs-string" style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-size: inherit; color: #718c00;">'vue-server-renderer/client-plugin'</span>)
</code></pre>
<p style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: antialiased; font-size: 15px; orphans: 3; widows: 3; margin-top: 1.2em; margin-bottom: 1.2em; word-spacing: 0.05em; line-height: 1.6em; color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif;">生成的默认文件是：</p>
<ul style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: antialiased; font-size: 15px; margin-top: 1.2em; margin-bottom: 1.2em; padding: 0px 0px 0px 1.5em; word-spacing: 0.05em; line-height: 1.6em; color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif;">
<li style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: antialiased; font-size: inherit;"><code style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-family: 'Roboto Mono', Monaco, courier, monospace; font-size: 0.85em; background-color: #f8f8f8; color: inherit; padding: 0.2em; margin: 0px; border-radius: 2px; white-space: nowrap; break-inside: avoid; direction: ltr; border: none;">vue-ssr-server-bundle.json</code>&nbsp;用于服务器端插件；</li>
<li style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: antialiased; font-size: inherit;"><code style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-family: 'Roboto Mono', Monaco, courier, monospace; font-size: 0.85em; background-color: #f8f8f8; color: inherit; padding: 0.2em; margin: 0px; border-radius: 2px; white-space: nowrap; break-inside: avoid; direction: ltr; border: none;">vue-ssr-client-manifest.json</code>&nbsp;用于客户端插件。</li>
</ul>
<p style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: antialiased; font-size: 15px; orphans: 3; widows: 3; margin-top: 1.2em; margin-bottom: 1.2em; word-spacing: 0.05em; line-height: 1.6em; color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif;">创建插件实例时可以自定义文件名：</p>
<pre style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-family: 'Roboto Mono', Monaco, courier, monospace; font-size: 15px; background: #f8f8f8; break-inside: avoid; direction: ltr; margin-top: 0px; margin-bottom: 0px; padding: 1.2em 1.4em; border: none; color: #333333; overflow: auto; word-wrap: normal; line-height: 1.5em;"><code class="lang-js" style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-family: 'Roboto Mono', Monaco, courier, monospace; font-size: 0.8em; background-position: 0px 0px; background-color: initial; color: inherit; padding: 0px; margin: 0px; border-radius: 2px; position: relative; break-inside: avoid; direction: ltr; border-top: none; border-right: none; border-bottom: none; border-image: initial; max-width: initial; overflow: initial; line-height: inherit;"><span class="hljs-keyword" style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-size: inherit; color: #8959a8;">const</span> plugin = <span class="hljs-keyword" style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-size: inherit; color: #8959a8;">new</span> VueSSRServerPlugin({
  filename: <span class="hljs-string" style="box-sizing: border-box; -webkit-tap-highlight-color: transparent; text-size-adjust: none; -webkit-font-smoothing: initial; font-size: inherit; color: #718c00;">'my-server-bundle.json'</span>
})</code></pre>
<p style="orphans: 3; widows: 3;"><font color="#333333"><span style="font-size: 15px; word-spacing: 0.75px;">那这两个插件生成的json文件大概是什么样的呢，下面看截图</span></font></p>
<p style="orphans: 3; widows: 3;"><font color="#333333"><span style="font-size: 15px; word-spacing: 0.75px;">vue-ssr-client-manifest.json:</span></font></p>
<p style="orphans: 3; widows: 3;"><font color="#333333"><span style="font-size: 15px; word-spacing: 0.75px;"><img src="http://km.oa.com/files/photos/captures/201709/1505023770_58_w585_h631.png" /></span></font></p>
<p style="orphans: 3; widows: 3;"><font color="#333333"><span style="font-size: 15px; word-spacing: 0.75px;">vue-ssr-server-bundle.json:</span></font></p>
<p style="orphans: 3; widows: 3;"><font color="#333333"><span style="font-size: 15px; word-spacing: 0.75px;"><img src="http://km.oa.com/files/photos/captures/201709/1505023779_91_w659_h284.png" /></span></font></p>
<p>那么配合这两个文件我们的router.js又该如何写呢：<span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"></span></span></p>
<p><img src="http://km.oa.com/files/photos/captures/201709/1505023819_75_w705_h698.png" /></p>
<p>&nbsp; &nbsp; 看完你会发现非常简单，就是将原来打包出来的index.js的读取换成<span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">vue-ssr-server-bundle.jsond的读取，同时读取<span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">vue-ssr-client-manifest.json的内容传给BundleRenderer的配置项。</span></span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">&nbsp; &nbsp; 需要注意的是，这两个插件只有使用createBundleRenderer才支持哦。</span></span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">&nbsp; &nbsp; 可以看到这两个插件的接入是很容易的，改动很小，但是它们有什么好处呢？最大的好处就是自动帮你做资源的预加载，资源预加载效果图如下：</span></span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><img src="http://km.oa.com/files/photos/captures/201709/1505024253_36_w688_h635.png" /></span></span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">&nbsp; &nbsp; 从这张图中我们可以看到第二条红线，这里面的内容就是之前提到的关键css的注入。</span></span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">到这里三种直出方式都已经讲完了，个人觉得会有一种由浅入深的过程，如果刚开始就让你直接用第三种直出，你可能会觉得一头雾水，不明所以。但是经过前两种的铺垫，就会有一种水到渠成的感觉。</span></span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">&nbsp; &nbsp; 当然选择哪种直出方式是没有什么准则的，选择自己觉得最合适的就行，甚至可以发挥你自己的想象，进行一些组合或者变换。</span></span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">&nbsp; &nbsp; 你以为这样这篇文章就结束了吗？那你就太naive了~其实在直出过程中，前端还有很多兼容性问题需要处理，接下来我就简单提几点以及我们的处理方式。</span></span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">1.避免重复加载cgi</span></span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">&nbsp; &nbsp; 在后端注入数据后，前端应该是不需要加载cgi的，使用vuex后，我们直接使用replaceState就搞定了。</span></span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><img src="http://km.oa.com/files/photos/captures/201709/1505024587_38_w581_h327.png" /></span></span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">2.解决前后端差异</span></span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">&nbsp; &nbsp; 前端已有的很多逻辑在后端是无法运行的，主要涉及window，cookie，url以及数据上报。这时建议进行分端处理。</span></span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">&nbsp; &nbsp; 首先我们进行变量注入：</span></span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><img src="http://km.oa.com/files/photos/captures/201709/1505024689_27_w718_h226.png" /></span></span></p>
<p><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;"><span style="color: #333333; font-family: 'Source Sans Pro', 'Helvetica Neue', Arial, sans-serif; font-size: 15px; orphans: 3; widows: 3; word-spacing: 0.75px;">然后在代码里进行判断是客户端还是服务端，从而进行不同操作：</span></span></p>
<div class="km_insert_code">
<pre><code><span class="hljs-keyword">const</span> isClient = process.env.VUE_ENV === <span class="hljs-string">'client'</span>
....

if(!isClient) {
        header = context.headers
        url = context.url
}
<span class="hljs-keyword">let</span> bkn = isClient ? Util.getBkn() : getBkn(header.cookie)
<span class="hljs-keyword">let</span> gc = isClient ? Util.getParameter(<span class="hljs-string">'gc'</span>) : getParameter(<span class="hljs-string">'gc'</span>, url)
<span class="hljs-keyword">let</span> uin = isClient ? Util.getParameter(<span class="hljs-string">'uin'</span>) : getParameter(<span class="hljs-string">'uin'</span>, url)
isClient &amp;&amp; retcodeReport);</code></pre>
</div>
<p>&nbsp;</p>
<p style="orphans: 3; widows: 3;"><font color="#333333"><span style="font-size: 15px; word-spacing: 0.75px;">&nbsp; &nbsp; 上面的代码就是当服务端时，获取bkn，gc以及uin的代码是和客户端不一样的，也不进行数据上报。</span></font></p>
<p style="orphans: 3; widows: 3;"><font color="#333333"><span style="font-size: 15px; word-spacing: 0.75px;">这里有个小细节就是我们巧妙的使用了context传入的url和headers信息。当然这种方式可能看起来有些hack，也许大家会有更好的解决方法，欢迎来交流。</span></font></p>
<p style="orphans: 3; widows: 3;"><font color="#333333"><span style="font-size: 15px; word-spacing: 0.75px;">3.网络请求</span></font></p>
<p style="orphans: 3; widows: 3;"><font color="#333333"><span style="font-size: 15px; word-spacing: 0.75px;">&nbsp; &nbsp; 网络请求也是我们需要格外注意的一个点，建议是前后端封装不同的ajax库。</span></font></p>
<div class="km_insert_code">
<pre><code><span class="hljs-meta">'use strict'</span>

<span class="hljs-keyword">import</span> { get <span class="hljs-keyword">as</span> clientGet,post <span class="hljs-keyword">as</span> clientPost,jsonp <span class="hljs-keyword">as</span> clientJsonp } <span class="hljs-keyword">from</span> <span class="hljs-string">'./client/ajax'</span>
<span class="hljs-keyword">let</span> serverGet, get
<span class="hljs-keyword">if</span>(process.env.VUE_ENV === <span class="hljs-string">'server'</span>) {
    serverGet = <span class="hljs-built_in">require</span>(<span class="hljs-string">'./node/ajax'</span>).get
    get = serverGet
} <span class="hljs-keyword">else</span> {
    get = clientGet
}

<span class="hljs-keyword">export</span> { get }</code></pre>
</div>
<p>&nbsp; &nbsp; &nbsp;在导出文件时，根据当前环境来导出不同的ajax库，这样我们的核心代码就可以保持不变。特别是对于内部需要接入L5的需求来说，这样封装后，代码结构就清晰很多。</p>
<p>&nbsp; &nbsp; 前端的问题大概就是这样了，接下来我们谈一谈如何接入玄武。此处省略一万字。玄武是我们团队开发的基于koa的框架，用它做直出非常方便，但是限于篇幅，这里就不展开了，大家有兴趣的可以联系我们进行尝试。</p>
<p>&nbsp; &nbsp; 所以到这里，这篇文章就是真的结束了。可以看到，我们的直出还是有限制的，它依赖与vuex库帮我们做了很多事情。这也给我们留下一些思考，没有store层的项目怎样接入才是最优雅的，函数式组件？还是其他更好的方法。vue直出的性能是怎样的，这里没有进行压测，所以没有数据统计与分析，这也是后面我们需要进行的一些工作。</p>
