<h2>背景 &nbsp; &nbsp;</h2>
<p>&nbsp; &nbsp; &nbsp; JavaScript现在俨然已经是最火的语言，无论是2016年GitHub发布的最热门的10大语言还是2016年GitHub最受欢迎的开源项目，JavaScript已经独领风骚。如果当初Brendan Eich知道JavaScript现在会成为互联网第一大语言，估计当时也不会就花10天就把这门语言设计出来了，那现在也许就少不少坑了。</p>
<p>&nbsp; &nbsp; &nbsp; JavaScript在诞生以及设计之初就是一门宽松编程语言，当初网景公司对这门语言的要求就是看上去和Java相似，但是比Java简单，使非专业的网页作者也能快速上手。所以才搭了Java的快车，取了JavaScript这个名字。但是，我们都知道，其实两者什么关系都没有。事实证明，JavaScript的简单的确让它很快就流行起来。但是这其实也是一把双刃剑，一方面的确可以提高编程效率，但是另一方面也降低了严谨性。很多人都有感觉，就是JavaScript开发的软件越来越复杂庞大，同时也越来越难维护。所以现在TypeScript也得到了很多JavaScript开发者的青睐。当然这篇文章我还是想谈谈JavaScript。</p>
<h2>原因</h2>
<p>&nbsp; &nbsp; &nbsp; &nbsp;因为JavaScript的宽松，相对来说对开发者的限制就少很多，例如我们不用像写c一样每行必须换行符。这样，很多人在开发时都有自己的习惯，包括命名习惯，代码书写习惯等等，但是在团队开发中，其实这是不提倡的，我们希望大家的代码风格尽可能统一。个人认为这样做有以下好处：<br /><strong>1.提升代码阅读效率</strong><br />&nbsp; &nbsp; &nbsp; &nbsp;无论拿到谁的代码，都感觉就像自己写的。代码很工整，阅读起来神清气爽，酣畅淋漓。例如：有的团队规范指定变量使用驼峰命名，类命名也使用驼峰，但是第一个字母大写。这样我们只要一看到变量，就知道它是变量还是类。不用去寻找代码定义的地方了。<br /><strong>2.减少很多不必要的bug，保证代码质量</strong><br />&nbsp; &nbsp; &nbsp; &nbsp;通过规范的规则，我们可以做到很多事。例如：强制使用===做判断，检查未定义的变量，声明的变量未使用等等。这些看起来不起眼的规则其实很可能就是隐藏的bug，而且有可能是最容易被忽略的，也就是最难debug到的。<br /><strong>3.对个人发展友好</strong><br />&nbsp; &nbsp; &nbsp; &nbsp;团队代码规范一般是团队总结出来的或者业界比较认可的，是大家多年经验的结晶。对于个人来说，遵循这些代码规范，会养成一个比较好的习惯。虽然刚开始遵循这些规范会很不适应，但是一旦习惯后，就变得像强迫症一样，写代码时会不自觉遵守。<br /><strong>4.可以避免大括号是否换行的圣战~(逃</strong></p>
<h2>工具对比</h2>
<p>&nbsp; &nbsp; &nbsp; &nbsp; 要想在项目中使用统一的代码规范，首先我们需要的是好的代码检验工具。</p>
<p>&nbsp; &nbsp; &nbsp; &nbsp; JavaScript语言精粹一书作者Douglas Crockford大神在2002年推出了JSLint，这是上古js静态代码分析工具，Douglas Crockford制定了一套js编码规则，JSLint就是按照他制定的规则来检查和分析代码，任何违反规则的代码都会发出警告。刚开始JSLint非常流行，但是慢慢的，随着时代的发展，很多代码的写法也在发生着变化，一方面，大神的个性很固执，其实也是很自信，坚持认为自己的代码规范没问题，对于开源社区的反馈基本上也置之不理，另一方面，JSLint这款工具的规则是不可配置的，没有办法关闭想要关闭的规则。这对于开发者就很痛苦了，原本想用它写出更好的代码，却发现它成了限制自己的牢笼。</p>
<p>&nbsp; &nbsp; &nbsp; &nbsp; 这样，JSHint就横空出世。JSHint做的事情其实和JSLint是一样的，但是它最大的优势就是规则可配置。它有一系列规则，你可以通过配置来决定使用哪些规则。通过将配置写进配置文件，这样整个团队就可以很好的共用一份代码规范了。</p>
<p>&nbsp; &nbsp; &nbsp; &nbsp; 而近些年最流行的其实是ESLint。</p>
<p>&nbsp; &nbsp; &nbsp; &nbsp; 接下来，我将通过对比分析一下JSHint和ESLint这两款工具的优缺点。</p>
<h3>1、安装</h3>
<p>&nbsp; &nbsp; &nbsp; JSHint和ESLint安装方式差不多，都是可以通过npm进行安装。</p>
<div class="km_insert_code">
<pre><code><span class="hljs-built_in">npm</span> install -g jshint
<span class="hljs-built_in">npm</span> install -g eslint</code></pre>
</div>
<p><br />&nbsp; &nbsp; &nbsp; 现在主流编辑器像Sublime，WebStorm等等都提供了比较好的支持，大家如果想尝试集成的话，可以自行谷歌，这里就不详细讲了。</p>
<h3>2、使用</h3>
<p>&nbsp; &nbsp; &nbsp; 安装完成后，都可以在命令行直接调用。例如：</p>
<div class="km_insert_code">
<pre><code><span class="hljs-selector-tag">jshint</span> <span class="hljs-selector-tag">test</span><span class="hljs-selector-class">.js</span>
<span class="hljs-selector-tag">eslint</span> <span class="hljs-selector-tag">test</span><span class="hljs-selector-class">.js</span></code></pre>
</div>
<p>&nbsp; &nbsp; &nbsp; &nbsp;这里先贴一下我测试使用的最简单的配置</p>
<div class="km_insert_code">
<pre><code><span class="hljs-comment">// JSHint</span>
{
    <span class="hljs-string">"curly"</span>: <span class="hljs-literal">true</span>,  <span class="hljs-comment">// 循环或者条件语句必须使用花括号包围，即使只有一行代码</span>
    <span class="hljs-string">"eqeqeq"</span>: <span class="hljs-literal">true</span>, <span class="hljs-comment">// 强制使用===和!==</span>
    <span class="hljs-string">"undef"</span>: <span class="hljs-literal">true</span>,  <span class="hljs-comment">// 禁止使用未定义变量</span>
    <span class="hljs-string">"esversion"</span>: <span class="hljs-number">6</span>, <span class="hljs-comment">// 开启对ES6的支持</span>
    <span class="hljs-string">"node"</span>: <span class="hljs-literal">true</span>,   <span class="hljs-comment">// 对node环境的支持</span>
    <span class="hljs-string">"browser"</span>: <span class="hljs-literal">true</span> <span class="hljs-comment">// 对浏览器环境的支持</span>
}

<span class="hljs-comment">// ESLint</span>
{
    <span class="hljs-string">"env"</span>: {
        <span class="hljs-string">"browser"</span>: <span class="hljs-literal">true</span>,
        <span class="hljs-string">"node"</span>: <span class="hljs-literal">true</span>
    },
    <span class="hljs-string">"parserOptions"</span>: {
        <span class="hljs-string">"ecmaVersion"</span>: <span class="hljs-number">6</span>
    },
    <span class="hljs-string">"rules"</span>: {
        <span class="hljs-string">"no-undef"</span>: <span class="hljs-number">2</span>, <span class="hljs-comment">// 0表示关闭，1表示警告，2表示错误</span>
        <span class="hljs-string">"curly"</span>: <span class="hljs-number">2</span>,
        <span class="hljs-string">"eqeqeq"</span>: <span class="hljs-number">2</span>
    },
    <span class="hljs-string">"plugins"</span>: []
}

<span class="hljs-comment">//test.js</span>
<span class="hljs-keyword">if</span>(a === <span class="hljs-number">1</span>) 
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"ok"</span>);
<span class="hljs-keyword">let</span> b = <span class="hljs-number">1</span>;</code></pre>
</div>
<p>&nbsp;<br />&nbsp; &nbsp; &nbsp; 规则其实比较简单，下面我们看看两者运行结果：</p>
<p><img src="http://km.oa.com/files/photos/pictures/201703/1488354940_28_w624_h86.png" /></p>
<p><img src="http://km.oa.com/files/photos/pictures/201703/1488354892_40_w623_h134.png" /></p>
<p>&nbsp; &nbsp; 从结果看，两者效果其实是差不多的，都能准确提示错误。具体的差别我们文章后面再讲。</p>
<h3>3、配置</h3>
<p>&nbsp; &nbsp; &nbsp; 巧妇难为无米之炊，工具再好，不知道规则也是徒劳的，所以我们需要告诉这些工具我们想要怎样的规则。JSHint和ESLint都支持4种配置方式。</p>
<ol>
<li>通过-<strong>-config</strong>标记手动指定配置文件</li>
<li>使用特定文件。两者分别为<strong>.jshintrc</strong>和<strong>.eslintrc</strong></li>
<li>配置放到项目的package.json中，key分别为<strong>jshintConfig</strong>和<strong>eslintConfig</strong></li>
<li>内联配置。在文件中插入特殊语句，达到配置的效果</li>
</ol>
<p>&nbsp; &nbsp; &nbsp; 个人推荐是使用第二种方式，这样整个项目都可以使用这个配置，有特殊情况处理的可以搭配内联配置进行解决。</p>
<h3>4、规则</h3>
<p>&nbsp; &nbsp; &nbsp; 接下来就是最重要的部分了：规则。</p>
<p>&nbsp; &nbsp; &nbsp; JSHint的规则：<a href="http://jshint.com/docs/options/" target="_blank" rel="noopener">http://jshint.com/docs/options/</a>。</p>
<p>&nbsp; &nbsp; &nbsp; ESLint的规则：<a href="http://eslint.org/docs/rules/" target="_blank" rel="noopener">http://eslint.org/docs/rules/</a>。</p>
<p>&nbsp; &nbsp; &nbsp; 从规则上看，两者其实都定义了一些规则，数量上ESLint数量多一点，且进行了分类。有一些细节的是，JSHint正在将一些规则进行分离。例如indent，文档中就提到了这个即将被移除，要使用的请使用JSCS。</p>
<h3>5、区别</h3>
<p>&nbsp; &nbsp; &nbsp; 可以看到，到目前为止，一切进展都很顺利，两个工具的使用都很简单。可以看到它们的优点都是：规则可配置，支持配置文件，易于共享，支持ES6.</p>
<p>&nbsp; &nbsp; &nbsp; 那它们有什么区别呢，为什么ESLint现在已经逐渐替代了JSHint了呢？个人认为有以下几点：</p>
<h3>5.1、错误提示</h3>
<p>&nbsp; &nbsp; &nbsp; 回到刚刚那个运行结果图，我们可以看到JSHint只给出了哪个地方出了错，但是哪条规则报错并没有给出，对比之下ESLint则在每条错误的最后给出了出现错误的规则。</p>
<p><img src="http://km.oa.com/files/photos/pictures/201703/1488354892_40_w623_h134.png" /></p>
<p>&nbsp; &nbsp; &nbsp; 这对于开发者来说其实是很有帮助的，有时候我们知道有些写法可以接受或者必须这样写，那我们就可以直接关闭这个规则。当然前提是我们必须知道哪个规则有问题，这里ESLint就比较人性化了。</p>
<p>下面以JSHint文档中的一段代码为例</p>
<div class="km_insert_code">
<pre><code><span class="hljs-meta">"use strict"</span>;

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">b</span>(<span class="hljs-params"></span>) </span>{
<span class="hljs-meta">    "use strict"</span>;

}</code></pre>
</div>
<p>&nbsp;<br /> 这段代码在JSHInt运行下会报错，结果如下：</p>
<p><img src="http://km.oa.com/files/photos/pictures/201703/1488352492_66_w550_h64.png" /><br /> 而在eslint下就不会报错：</p>
<p><img src="http://km.oa.com/files/photos/pictures/201703/1488352518_74_w472_h50.png" /></p>
<p>&nbsp; &nbsp; &nbsp; 那如果我觉得这种写法是可以接受的，那我怎么屏蔽这个规则呢，我也没有开启这个规则啊，这个规则叫什么名字叫什么我也不知道啊，相信很多人这个时候如果不去查官方文档就会一脸蒙蔽。按照官方文档解释，我们需要在运行时加上--verbose选项，来查看这是哪一条规则</p>
<p><img src="http://km.oa.com/files/photos/pictures/201703/1488352553_19_w611_h73.png" /></p>
<p>接下来就是使用内联配置屏蔽这一规则了</p>
<div class="km_insert_code">
<pre><code><span class="hljs-comment">/* jshint -W034 */</span>
<span class="hljs-meta">"use strict"</span>;

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">b</span>(<span class="hljs-params"></span>) </span>{
<span class="hljs-meta">    "use strict"</span>;

}</code></pre>
</div>
<p>&nbsp;<img src="http://km.oa.com/files/photos/pictures/201703/1488352593_51_w499_h41.png" /><br /> 这个时候再运行世界就清静了~</p>
<p>&nbsp; &nbsp; &nbsp; 而且JSHint的规则里面还有强制选项和宽松选项，说实话，我觉得还是比较麻烦，对于开发者来说，自己决定哪些规则需要哪些不需要就可以了。像上面的w034，我还需要在每个用的地方加上这段代码，要是能在配置文件里直接屏蔽这个规则该多好啊。<br />&nbsp; &nbsp; &nbsp; 相比较之下，eslint就完胜了。它默认是关闭所有规则的，任何规则都是可以开启关闭的。这样即使有些规则开启了，我们可以通过错误信息看到引起错误的规则，在配置文件里，直接设为0就可以关闭了。如果不想直接全部屏蔽，也支持这种文件里的内联配置。</p>
<h3>5.2、可拓展性</h3>
<p>&nbsp; &nbsp; &nbsp; ESLint还有一个地方胜出的是它的设计思想。ESLint被设计为可插拔的，很容易拓展。就像现在的webpack构建工具，插件茫茫多，依靠社区大家的力量，这比一个人开发要好多了。而且插件可以满足不同人的需求。ESLint是支持JSX的。<br /> ESLint的默认解析器是Espree，但是这个是可配置的，我们可以指定其他解析器例如Babel-ESLint</p>
<div class="km_insert_code">
<pre><code>{
    <span class="hljs-attr">"parser"</span>: <span class="hljs-string">"babel-eslint"</span>,
    <span class="hljs-attr">"plugins"</span>: []
}</code></pre>
</div>
<p>&nbsp;<br /> 这样我们就可以使ESLint和Babel兼容了。</p>
<p><strong>因为ESLint的可拓展性，所以它是可以利用插件来支持React代码的，当然也可以支持Vue和AngularJs，在这三架马车大红大紫的现在，ESLint必然也会得到更多关注。</strong></p>
<p>这才是ESLint的大杀器啊，JSHint只能望而却步啊~</p>
<p>所以，对比之下，ESLint的确在很多方面做的比较好，所以下面的实践就以ESLint为例。</p>
<h2>规范推荐</h2>
<p>&nbsp; &nbsp; &nbsp; 很多团队也许都有自己的代码规范，但是说实话，制作代码规范还是一件比较麻烦的事，而且如果结合ESLint的话，最好的做法是做成插件使用，这个就更麻烦了，所以这里我推荐两个常用的代码风格规范，基于这两个风格规范进行定制，配置适当的rule，就可以很容易生成自己的代码规范。<br />&nbsp; &nbsp; &nbsp; 第一个规范是<a href="http://standardjs.com" target="_blank" rel="noopener">JavaScript Standard Style</a>,对应的拓展配置文件是eslint-config-standard。使用方法是</p>
<div class="km_insert_code">
<pre><code><span class="hljs-built_in">npm</span> install eslint-config-standard
<span class="hljs-regexp">//</span> 添加到.eslintrc文件
{
    <span class="hljs-string">"extends"</span>: <span class="hljs-string">"standard"</span>
}</code></pre>
</div>
<p>相应的规则大家可以去看官网介绍。</p>
<p><br />&nbsp; &nbsp; &nbsp; 第二个推荐的规范是<strong><a href="https://github.com/airbnb/javascript" target="_blank" rel="noopener">aibnb-javascript</a>，</strong>这个规范包含推荐的es5以及es6，react的代码规范，并且基本上都有相应的错误和正确代码以及为什么要这么写，个人觉得有时间去读一读还是很好的，虽然感觉有些规则也不是很赞同。<br />&nbsp; &nbsp; &nbsp; 配套的拓展配置文件是<a href="https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb" target="_blank" rel="noopener">eslint-config-airbnb</a>,里面有详细的安装使用说明。</p>
<div class="km_insert_code">
<pre><code>{
    <span class="hljs-attr">"extends"</span>: <span class="hljs-string">"airbnb"</span>
}</code></pre>
</div>
<p>&nbsp;<br />&nbsp; &nbsp; 其中默认是对React支持的，大家正在写React的可以去试一试~</p>
<p>&nbsp; &nbsp; 但是这个规范很严格，第一次检查时没多少行代码竟然报出几十个错误，简直吓死人。当然，很多React的规范很难去遵守，例如要求每个文件只写一个模块，但是很容易通过配置内联规则或者写进配置的rule中来个性化定制。</p>
<h2>实践</h2>
<p>&nbsp; &nbsp; &nbsp; 好了，现在工具和规则我们都有了，剩下的就是执行了，但是每次手动执行好像很蠢的样子。幸运的是，我们可以结合构建工具来执行，以<strong>gulp-eslint</strong>为例</p>
<div class="km_insert_code">
<pre><code><span class="hljs-keyword">var</span> gulp = <span class="hljs-built_in">require</span>(<span class="hljs-string">'gulp'</span>), 
eslint = <span class="hljs-built_in">require</span>(<span class="hljs-string">'gulp-eslint'</span>);

gulp.task(<span class="hljs-string">'lint'</span>, <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{ 
    <span class="hljs-keyword">return</span> gulp.src([<span class="hljs-string">'js/**/*.js'</span>]) // 文件位置可以修改
        .pipe(eslint())
        .pipe(eslint.format())
});

gulp.task(<span class="hljs-string">'default'</span>, [<span class="hljs-string">'lint'</span>], <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{ 

});</code></pre>
</div>
<p>&nbsp; &nbsp; &nbsp; 这样我们可以在build前或者每次变更都执行eslint。当然也可以和编辑器结合，写代码时就可以知道哪些地方有问题了。</p>
<p><br /> 如果团队对代码规范要求比较严格的话，推荐使用<a href="https://www.npmjs.com/package/pre-commit" target="_blank" rel="noopener">precommit</a>。安装成功后，配置大致如下</p>
<div class="km_insert_code">
<pre><code>{
    <span class="hljs-attr">"scripts"</span>: {
        <span class="hljs-attr">"eslint"</span>: <span class="hljs-string">"./node_modules/.bin/eslint --ext .js,.jsx dev/js/**"</span> <br />        // 具体文件路径以及后缀可自行更改
    },
    <span class="hljs-attr">"pre-commit"</span>: [
        <span class="hljs-string">"eslint"</span>
    ]
}</code></pre>
</div>
<p>&nbsp;<br />&nbsp; &nbsp; &nbsp; 这样配置后，每次运行git commit之前都会自动去执行这个eslint task，如果不通过，是没有办法提交。如下图：</p>
<p><img src="http://km.oa.com/files/photos/pictures/201703/1488353350_48_w743_h289.png" /></p>
<p><strong>&nbsp; &nbsp; &nbsp; 如果安装后commit没有运行你的配置，建议你去.git/hooks下看看有没有pre-commit这个文件，没有的话建议手动重新安装precommit一下</strong></p>
<h2><strong>总结</strong></h2>
<p>&nbsp; &nbsp; &nbsp; 终于，这篇文章也说的差不多了，当然，这只是个抛砖引玉的作用，推荐的两个代码规范也是目前业界相对比较流行的，并不代表一定就是最好的。<br /><br />&nbsp; &nbsp; &nbsp; JavaScript仍然会快速发展，开发中最重要的也就是质量和效率。过度遵循代码规范，追求代码严谨也许并不能带来预期的好处。所以做到怎样的程度还是需要大家自己拿捏的。如果有问题，欢迎大家斧正~</p>
<h2>参考</h2>
<p><a href="http://jshint.com" target="_blank" rel="noopener">http://jshint.com</a><br /><a href="http://eslint.org" target="_blank" rel="noopener">http://eslint.org</a><br /><a href="http://standardjs.com" target="_blank" rel="noopener">http://standardjs.com</a><br /><a href="https://github.com/airbnb/javascript" target="_blank" rel="noopener">https://github.com/airbnb/javascript</a></p>
