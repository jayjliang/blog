上一篇和大家分享了PDF格式的相关内容，其中有提到内嵌字体的相关内容，但是限于篇幅没有展开细讲，那本篇文章就用常见的TTF字体格式抛砖引玉，帮助大家了解字体的一些知识。如果大家有时间还是建议阅读一下前篇文章，特别是如何查看二进制数据的部分，本篇文章会用得到。

### 一、背景

作为前端开发，相信大家经常使用`css`属性`font-family`，并且值经常是很长的一串，类似下面：
> -apple-system,system-ui,Segoe UI,Roboto,Ubuntu,Cantarell,Noto Sans,sans-serif,BlinkMacSystemFont,"Helvetica Neue","PingFang SC","Hiragino Sans GB","Microsoft YaHei",Arial;

为什么要写这么一大串fallback逻辑，其实最主要原因就是因为系统对字体的处理。windows机器上一般会自带微软雅黑字体，mac机器上一般会自带苹方字体，那如果我们在windows机器上告诉程序想用苹方字体去渲染一个字符串，系统不一定能给你做到。假如你的windows机器恰好安装了苹方字体，那万事大吉，假如没有，那可能就用系统默认字体给你渲染了（取决于你的程序逻辑）。所以我们经常遇到word文档在我自己电脑上看到好好的，但是别人打开咋就不一样了。

PDF格式相比于其他格式来说，有一个很大的优点，就是跨平台，也就是一个文档无论你在windows还是mac抑或手机上看，它的格式都是一样的，这其中也包括字体。PDF文件不存在字体问题，背后其实也没有什么黑魔法，就是把字体文件嵌入了文件当中，渲染的时候直接使用嵌入的字体而不是系统字体，那问题就迎刃而解了。

### 二、常见的字体格式

- TTF：TTF (TrueType Font) 字体格式是由苹果和微软为 PostScript 而开发的字体格式，差不多是大家最熟悉或者说听的最多的格式了。
- OTF：OTF (OpenType Font) 由 TTF 演化而来，是 Adobe 和微软共同努力的结果。OTF 字体包含一部分屏幕和打印机字体数据。OTF 有几个独家功能，包括支持多平台和扩展字符集。我理解这是TTF字体的超集。
- WOFF：WOFF (Web Open Font Format) 本质上是 metadata + 基于 SFNT 的字体（如 TTF、OTF 或其他开放字体格式）。该格式完全是为了 Web 而创建，由 Mozilla 基金会、微软和 Opera 软件公司合作推出。 WOFF 字体均经过 WOFF 的编码工具压缩，文件大小一般比 TTF 小 40%，加载速度更快，可以更好的嵌入网页中。
- WOFF2：WOFF2 是 WOFF 的下一代。 WOFF2 格式在原有的基础上提升了 30% 的压缩率。

上面列举了一些常见的字体格式，可以看到，TTF不仅是我们最熟悉的，也和其他字体有比较深的渊源，所以本篇文章的重点也是介绍TTF字体格式。

### 三、TTF文件结构

#### 3.1 概览

大部分TTF文件都是`.ttf`后缀，但是也有部分文件是`.ttc`后缀，TTC代表TrueType Collection，也就是一个文件中包含多个TTF文件，主要是为了更好的共享一些信息来减小文件体积。

接下来我们以Arial字体为例，和大家一起揭开TTF的神秘面纱。


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8c6f2988bc8147f885d15145634c2cc5~tplv-k3u1fbpfcp-watermark.image?)

上面的图是使用二进制文件查看器查看Arial.ttf的截图，二进制文件看起来就是这样，ascii的模式下看起来很多就是乱码。个人觉得二进制一个比较大的优点是体积小，没有冗余的信息，如果你把一个JSON序列化后，就会存在很多冗余信息，但是缺点就是不够灵活，JSON的数组可以存储任意类型元素，但是二进制里面如果有数组的概念，那就不能元素是两种不一样的类型，除非再借助其他信息，这也导致JSON的序列化和反序列化要比二进制慢。

在这种限制下，一个二进制的文件解析规则就是固定的，比如TTF文件就有它自己的规则，你必须按照它的规则来解析才能得到正确结果。

#### 3.2 目录

我们都知道书一般有目录和正文两个比较重要的部分，目录的主要作用是快速找到对应的正文。而TTF文件也有类似目录的概念，叫`Font Directory`，它里面主要的内容就是这个TTF文件正文有多少部分，每个部分在哪里。而TTF文件的正文是`Font Tables`，每一个Table都有它自己的作用，这个后面会展开来讲。那接下来我们就根据目录的规则对比真实文件来一起解析一下。


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/79193adff5d0477b81df19bfcf73976c~tplv-k3u1fbpfcp-watermark.image?)

可以看到，文件的前4个字节表示的文件类型。0x74727565（true）或者0x00010000都是TTF格式，可以看到这个文件是0x00010000。再接下来两个字节是tables的数量，可以看到这个文件一共有24个。剩余的6个字节我们暂时不关心。


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6b834ecc76c043dbbf661a9e2ab7890f~tplv-k3u1fbpfcp-watermark.image?)

这只是我们的文件信息总览，接下来就是24个tables的信息，主要是内容偏移位置，定义如下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d85e1d56bc14777ba8b2171d558ad7c~tplv-k3u1fbpfcp-watermark.image?)

这里我们就解析一下第一个tables

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b33664c73108475891c60139aebb2a2e~tplv-k3u1fbpfcp-watermark.image?)

我们把解析的信息翻译一下：
- tag：0x44534947 = DSIG
- checksum： 0x7232A231
- offset： 0x000BA844 = 763972
- length： 0x00002430 = 9264
也就是说DSIG这个table的内容从763932这个位置开始，总长度是9264，但是里面的内容具体怎么解析就需要根据DSIG这个table的规则来解析了。

可以看到，这就是目录的作用，也凸显出了二进制的优势，那就是可以随机按需访问。我可以只解析部分文件，试想一下JSON文件可以只解析一部分嘛。

#### 3.3 Font Tables

这里每一个table都有自己的作用，很多table也很复杂，限于篇幅，不能全部展开，想了解详细内容可以参阅官方文档。这里只介绍一些基础或者常用的table。

官方文档提到必须的table有以下几个。


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa9af4a3634044a3b90ec76f5871a2a3~tplv-k3u1fbpfcp-watermark.image?)

##### glyf

这个是字体的核心内容，表示一个文字是如何被绘制的。例如下面的字母b，其实就是利用直线以及贝塞尔曲线来绘制出来的，那有每条线的坐标信息就是存在glyf里面的。当然，glyf本身比较复杂，分简单和复杂两种模式，这里也不展开。


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de82913640b340d6ab1c1864091fd859~tplv-k3u1fbpfcp-watermark.image?)

##### cmap

从我们代码的角度出发，以字母b为例，我们只知道它的codepoint是98，那我们怎么根据98来找到对应的glyf呢，最简单的方式就是glyf是一个数组，第98个正好是b。

但是这种场景就会存在非常多的空间浪费，如果字体文件只有一个glyf，cdePoint是65536，那前面65535岂不是都浪费了，而且假如字符是不连续的，例如97，107，207，那中间的间隔也会造成浪费。

所以我们需要一个映射，将codepint映射到glyf数组的下标。而这个映射表就是cmap。这里我只介绍其中一个比较巧妙的format4。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fed99c868f174512ae355c792bb536ea~tplv-k3u1fbpfcp-watermark.image?)

按照文档说法，它适用于连续区间或者空白比较多的场景，说白了就是最大压缩连续区间。例如字符是从10到100，如果一个个映射，那需要存储90个数，但是如果使用上面这种算法，只需要存储开始数字startCode 10，结束数字endCode 100。

我们结合官方文档的例子来理解一下：


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/650afb6094d1470fa53557644663edf8~tplv-k3u1fbpfcp-watermark.image?)

我们现在有三个连续区间的字符，codepoint分别在10-20，30-90，100-153，总共126个字符，那最终glyf的数组长度就是127（第0个glyf保留，谁也不能用），也就是下标是从1到126，是连续的，但是我们的字符的codepoint不是完全连续的。

根据上面的算法，我们用startCode表示区间起点，endCode表示区间终点，idDelta表示codePoint和glyf的index之间的差距，

> codepoint 10映射到的glyfindex就是10-9=1

> codepoint 20映射到的glyfindex就是20-9=11

> codepoint 30映射到的glyfindex就是30-18=12

> codepoint 90映射到的glyfindex就是90-18=72

> codepoint 100映射到的glyfindex就是100-27=73

> codepoint 153映射到的glyfindex就是153-27=126

可以看到最终映射的值就是我们想要的从1到126，同时只用了三条数据就完成了126个字符的信息存储，是不是比较巧妙！

##### local
有了glyf的下标后，我们就需要知道它的内容偏移信息，这个就是存在local里面的，所以理论上local数组的长度和glyf数组的长度是一致的。

##### head
字体全局信息，包含字体版本号、创建时间、修改时间，还有bounding box的坐标，unitsPerEm可以理解为基准单位吧，假如这个值是1000，那如果你的字符高度是2000，那计算真实高度时是需要把2000除1000的。

##### hhea
水平排版的字体信息，包括ascent、descent、lineGap

#### 3.4 字体样式
这里的字体样式主要是指加粗和斜体，可以看到，在字体文件本身信息里面基本上没有这个方面的信息。实际上。加粗和斜体都需要单独的字体文件，例如`Arial.ttf`表示正常字体，`Arial Bold.ttf`表示粗体，`Arial Italic.ttf`表示斜体，`Arial Bold Italic.ttf`表示加粗的斜体，但是实际上很多字体设计的时候都没有设计斜体，例如很多中文字体。但是我们在chrome里面却可以对中文字体设置斜体，原因是因为浏览器的斜体是假斜体，前面提到过字体是用线来绘制的，那在绘制的时候就可以使用transform矩阵来对影响它。

`css
transform: matrix(1, 0, -0.2, 1, 0, 0);
`

感兴趣的试试上面这个样式会让你的文字发生什么变化～

### 四、工具

上面介绍了TTF字体格式的相关内容，可以看到，这些二进制文件都靠自己去读肯定效率很低，所以还是需要借助一些解析工具的。这里目前业界比较出名的当属[opentype.js](https://opentype.js.org/)，它支持TTF、OTF、WOFF，功能比较强大，很多库对字体的操作都是基于它的，比如字体裁剪工具：[font-carrier](http://purplebamboo.github.io/font-carrier/)。

但是因为它对字体的解析基本上是全量的（glyf的解析后来被优化成按需的了），所以解析速度相对慢，内存占用大。

回到一开始提到的PDF文件嵌入字体，大家都知道字体文件很大，特别是中文字体，基本上都是10M以上，如果只是暴力的把所有用到的字体都嵌入到PDF文档中，那最后体积就会变得很庞大，所以需要基于使用的文字对字体做裁剪。

### 五、总结
本篇文章介绍的也只是TTF字体格式的冰山一角，除了格式本身，笔者也感受到了二进制的冲击，当没有了map，怎么使用二进制文件来存储复杂数据同时极致的压缩体积，背后都隐藏着很多学问。最后对于不对的地方欢迎斧正～


参考：
- [Web 字体简介: TTF, OTF, WOFF, EOT & SVG](https://zhuanlan.zhihu.com/p/28179203)
- [TTF官方文档](https://developer.apple.com/fonts/TrueType-Reference-Manual/RM01/Chap1.html)
- [OTF官方文档](https://docs.microsoft.com/en-us/typography/opentype/spec/)
