相信大家对于PDF格式的文件应该不陌生，但是可能大部分人都只是知道。最近因为一些需求涉及到了PDF，所以做了一些相对来说深入的了解，这个过程中也发现现在网上相关的资源比较少，官方文档是很全，但是很长且全英文不易于解读，所以这里把一些经验分享给大家，希望帮助到有需要的同学。

### 一、如何查看PDF文件

工欲善其事，必先利其器。PDF文件本质上算是二进制文件，所以直接使用vscode打开，会有部分是可读的文本，部分看起来像是乱码。就像下图：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a1254630eca3407b9dd7bf744f3d52f4~tplv-k3u1fbpfcp-watermark.image)

但是如果你使用二进制文件查看工具（比如我用的是Hex Fiend），大概就长下面这样：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f3d432b4489d4181b5f7cab2ffb6ad26~tplv-k3u1fbpfcp-watermark.image)

很多同学看到这张截图，是不是顿时就会产生放弃的想法，在不知道PDF结构的时候，的确会产生这样的感觉，但是相信我，读完接下来的内容，你再回头看的话，真的不难～

回到正题，个人在查看工具上是这样建议的：
1. 如果你只是简单的查看的话，使用vscode是完全没有问题的
2. 如果你想要修改或者复制的话，那还是建议使用二进制文件查看文件，使用vscode修改二进制文件还有复制都会有问题

### 二、PDF文件结构解析

解下来我从一个真实的文件出发，带大家一起了解一下PDF的格式。这个文件是我使用pdfkit这个工具生成的文件，一共有两页，第一页就有几个文字，第二页是空白的，内容如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6dee99dbcc8e44819952caf681909675~tplv-k3u1fbpfcp-watermark.image)

为了方便大家阅读，我直接使用vscode打开。


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6565cd0de8d3410c9fc20d6ebabd4a00~tplv-k3u1fbpfcp-watermark.image)

参考PDF官方文档，我们可以从以下四个方面去了解PDF文件：
- Objects 对象
- File structure 物理文件结构
- Document structure 文档结构
- Content streams 内容流

#### 2.1 你必须要知道的obj

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b9eff8dd2658474985fecd361d92f955~tplv-k3u1fbpfcp-watermark.image)

这是第一屏的数据，可以看到很多都是可读文本。obj我认为是PDF文件的基本构成，一个PDF文件就是由很多obj组成的。
最开头的`%PDF-1.3`是PDF版本号，接下来跳过几个乱码后，就是重点了：obj。

```text
7 0 obj
<<
/Type /Page
/Parent 1 0 R
/MediaBox [0 0 612 792]
/Contents 5 0 R
/Resources 6 0 R
>>
endobj
```
obj的格式是以【数字1 数字1 obj】内容【endobj】，其中数字1表示obj的序号，每个obj都不一样。数字2是叫生成号，按照PDF规范，如果一个PDF文件被修改，那这个数字是累加的，实际上这个生成号一般看到的都是0。中间的是这个obj的内容，最后`endobj`表示这个obj的结束。

这个东西为了简单理解，你可以把它看成js中的object，我把上面的内容转成js的obj你再看看：

```json
{
    "type": "Page",
    "Parent": "1 0 R",
    "MediaBox": "[0 0 612 792]",
    "Contents": "5 0 R",
    "Resources": "6 0 R"
}
```

是不是看起来熟悉了很多，其实这也就是obj的大致语法了。那上面这段表示什么意思呢，我给你们翻译一下：

        我是一个Page，我的父对象是序号为1的obj，我的页面大小是左上角：0，0，宽度612，高度792。
        我的内容是序号为5的obj，我的资源是序号为6的obj。

我们接下来看文件，是序号为6的obj：
```text
6 0 obj
<<
/ProcSet [/PDF /Text /ImageB /ImageC /ImageI]
/Font <<
/F2 8 0 R
>>
>>
endobj
```
这里需要注意一个嵌套，我翻译成object你们就懂了：

```json
{
    "ProcSet": "[/PDF /Text /ImageB /ImageC /ImageI]",
    "Font": {
        "F2": "8 0 R"
    }
}
```
6 0 obj是上面的page的资源，有点像是css 样式，比如这里说的就是字体信息，其实`F2`这个字体的内容在8 0 R。

接下来就是page的内容5 0 R
```text
5 0 obj
<<
/Length 113
>>
stream
1 0 0 -1 0 792 cm
1 0 0 -1 0 792 cm
BT
1 0 0 1 100 691 Tm
/F2 20 Tf
[<00010002000300040004000400040004> 0] TJ
ET

endstream
endobj
```
这里稍微有一点不一样的是，这里除了有一个Length外，还有stream。stream的格式是：【stream 内容 endstream】，这里stream里面的具体内容就是我们看到的文字的绘制命令，我们在接下来的章节详细介绍，这里先跳过。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0485811e919a42158b9cf35c14c048dd~tplv-k3u1fbpfcp-watermark.image)

接下来就是第二页的内容了，这里我们就不展开了，其实和第一页差不多，大家对比阅读一下就可以了。

#### 2.2 物理文件结构
大家会读obj后，接下来讲物理文件结构就会简单很多。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d8ef5b96cb2454bbfa3303589476b4d~tplv-k3u1fbpfcp-watermark.image)

按照官方文档，一个PDF文件从物理结构上来说可以分为四个部分：
- Header 头部，也就是上面说过的PDF版本号，就那么多
- Body 内容，Body是由很多obj组成的，上面讲obj的时候那些例子都是Body的一部分
- Cross-reference table 交叉引用表
- File Trailer 文件尾


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d496c07e3ad42d9b76eb4d81354048d~tplv-k3u1fbpfcp-watermark.image)

可以看看我们这个测试文件的交叉引用表和文件尾。

##### 交叉引用表
交叉引用表的作用是什么呢，它的设计初衷是为了快速访问obj，里面标记了它的起始位置。

`xref`表示是交叉引用表，每个交叉引用表又可以分为若干个子段，我们这个例子只有一个子段。接下来的`0 20`表示这个子段开始于0，一共有20个对象。解析来的每行有三个部分，格式为【相对头文件偏移地址 生成号 对象是否使用】，生成号和obj的生成号类似，也是修改的时候使用，n表示对象有被使用，这个值还可能是f，表示被删除或没有用。

但是在实际过程中，你的对象偏移地址即使不对，PDF也能正常解析，所以这一块你基本上可以不用关注，知道就行。

##### 文件尾
文件尾的作用主要是快速找到交叉引用表的位置，还有一些文件信息，像加密，入口对象（type为Catalog的obj）。这个文件一般关注也不多，因为我们不是解析PDF，生成的工作这些库都帮你做了。

#### 2.3 文档结构
有了上面这些，你知道怎么读懂PDF文件，也能从物理上把每个obj解析出来。但是不同obj之间如何配合以及如何影响PDF的这些你就不知道了，这也是这一节的主要内容。接下来看官方文档：


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d88aaffa7627422f95dd90c11f0760f4~tplv-k3u1fbpfcp-watermark.image)

实际上官方文档很全也很详细，但是我们这里只关注主要功能，所以主要关注Page tree这一部分。

##### Document catlog
这个在文件尾也提到了，其实就是表示文档的入口，看我们的测试文件：

```text
3 0 obj
<<
/Type /Catalog
/Pages 1 0 R
/Names 2 0 R
>>
endobj
```
内容很简单，就是先声明type是catalog，然后pages在1 0 R，names在2 0 R

##### Pages
Pages字段是所有页面的描述集合，看我们的测试文件：
```text
1 0 obj
<<
/Type /Pages
/Count 2
/Kids [7 0 R 11 0 R]
>>
endobj
```
这里的意思是我们一共有两页，第一页在7 0 R，第二页在11 0 R。7 0 obj和11 0 obj我们在介绍obj的时候都介绍过了。

到这里，我们对整个文件的结构应该有了一些清晰了解，至少知道每一页在哪，内容大概是些啥，归纳一下如下：

```text
Catalog 3 0 R
    Pages 1 0 R
        Kids1: 7 0 R
            Parent: 1 0 R
            Contents: 5 0 R
            Resources 6 0 R
        Kids2: 11 0 R
            Parent: 1 0 R
            Contents: 9 0 R
```

#### 2.4 Stream
还记得我们在obj一节中讲到了Stream，但是没有详细展开讲，这一节稍微展开描述一些，因为Stream有一个编码的概念，很多文件的Stream编码后看起来都是乱码，基本上没有办法阅读。

Stream主要是存储大量数据的，例如内嵌的字体、图片还有绘制命令。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9ea7fd096b84f34b1d321782467ba7d~tplv-k3u1fbpfcp-watermark.image)
像我们文件的16 0 obj，如果你去找每个obj的关系的话，会发现它最后表示的内嵌的字体。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d98c1a881f11494e862b063413e7df15~tplv-k3u1fbpfcp-watermark.image)
这个7 0 obj是从另外一个文件中截取的，它表示的也是绘制命令，和我们的5 0 obj是差不多的，但是你看起来却是乱码，主要是因为多了Filter设置，也就是编码，目前有的编码如下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a821815b90074d11838f6cc674b2ef78~tplv-k3u1fbpfcp-watermark.image)
可以看到，这个例子中用到是`FlatDecode`，也就是zlib压缩算法。

这样的内容很影响我们分析PDF文件，所以下面提供一段Python脚本解压缩（因为是从github上抄的，感兴趣可以写js版本）

```python
import re
import zlib

pdf = open("test.pdf", "rb").read()
stream = re.compile(b".*?FlateDecode.*?stream(.*?)endstream", re.S)

allStream = stream.findall(pdf)
for index in range(len(allStream)):
    s = allStream[index].strip(b'\r\n')
    try:
      if index == 0:
        print(zlib.decompress(s))
    except:
        pass
    
```
PS：这段脚步目前比较简陋，需要手动置顶那个Stream，主要是因为像字体这种Stream解压出来也看不懂，所以一般找到了绘制命令的Stream，再解压分析。

#### 2.5 绘制
到这里，已经扫清了一切障碍，我们可以开始认真分析绘制相关的内容了，也就是你看到的文字、图片、形状啥的是怎么描述的。

在PDF文档里面，这一章节叫`Graphics`，也就是图形。具体长啥样呢，看下面：
```text
5 0 obj
<<
/Length 113
>>
stream
1 0 0 -1 0 792 cm
1 0 0 -1 0 792 cm
BT
1 0 0 1 100 691 Tm
/F2 20 Tf
[<00010002000300040004000400040004> 0] TJ
ET

endstream
endobj
```
就是我们的stream中的内容。这一段内容啥意思呢，我人工翻译一下：

    做一次transform，参数是1 0 0 -1 0 792，再做一次
    开始绘制文字了，设置文字位置 1 0 0 1 100 691
    设置字体为F2，字号为20
    绘制文字，内容是<00010002000300040004000400040004>
    绘制文字结束
其中文字内容我们看到的是【你好 wwwww】，那很多人就懵了，这里绘制的是一串啥玩意，其实主要是因为我们用了自定义字体，所以这里的内容不是直接就是你绘制的文字内容的编码，这里设计到自定义字体相关的，比较复杂，有机会再单独一篇文章讲。如果我换成默认字体，你应该就能看懂了，例如绘制一个文字1:
```text
BT
1 0 0 1 125.090185 758.640014 Tm
/F1 9.749988 Tf
[<31> 0] TJ
ET
```
什么，不是说好绘制1的嘛，怎么是31？其实1的charCode是49，那49的16进制是不是就是31啦。

好了，例子看完了，其实PDF的绘制和Canvas很类似，或者说很多操作可以一一对应，这也就是PDF.js这个库为什么能展示PDF，因为它把PDF的绘制指令转成了Canvas的绘制命令。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d20c2c54f0245f3b1041ebf25b262a2~tplv-k3u1fbpfcp-watermark.image)
这是官方文档列出来的一些指令，其实还有一些没有列出来，详细信息可以去官方文档仔细阅读。可以简单说几个Path相关的，例如q表示save、Q表示restore、re表示rect，c表示贝塞尔曲线、w表示ineWidth、m表示moveTo、l表示lineTo、f表示fill、S表示stroke、n表示clip、cm表示transform，更多更详细的指令还是建议去官方文档了解，很多都是用到了再用。

就是通过这些指令，PDF预览程序进行绘制才形成了我们最后看到的样子。

### 三、如何生成PDF

了解了PDF的格式后，我们可以顺便了解一下如何生成PDF。

#### 3.1 前端生成

如果前端生成，其实相关的库很多，这里推荐两个：
1. [pdfkit](https://github.com/foliojs/pdfkit)
2. [jspdf](https://github.com/MrRio/jsPDF)

像很多人都需要的诉求是将html转成pdf，那就可以使用html2canvas，在使用jspdf转成PDF，html2pdf就是这么干的。但是我自己的使用以及对比来说，pdfkit虽然star少一点，但是兼容性做的比jspdf好，例如rgba颜色jspdf就不支持，但是pdfkit支持，还有自定义字体这块，jspdf目前的实现比较消耗内存。

前端生成最大的问题其实是**字体问题**，因为PDF对于使用到的字体，除了标准的14种字体外，其他字体需要进行内嵌，而这标准的14种字体都不支持中文。如果要把字体内嵌，一方面你需要注意字体的法律风险问题，另一方面，字体文件普遍比较大，你还需要对字体进行裁剪，这就需要你对字体文件例如ttf格式有了解。

#### 3.2 服务端生成

如果是服务端生成的话，最常见的使用puppeteer无头浏览器。对于一些使用canvas的页面，可以考虑使用node-canvas，其实底层就是使用了skia或者cario这些绘图引擎来实现了CanvasRenderContext。

后端生成pdf的话，字体文件可以通过服务端安装来解决。

### 四、总结
本篇文章介绍的也只是PDF的冰上一角，主要是帮助大家从整体上理解PDF，其实到很多细节的话，还是有很多内容的。如果读到这里，再去读开始的二进制文件，大家已经不像一开始那么懵了，那本篇文章的作用也就达到了，最后对于不对的地方欢迎斧正～

参考：
- [PDF官方规范](https://www.adobe.com/content/dam/acom/en/devnet/pdf/pdfs/PDF32000_2008.pdf)
- [一文搞懂PDF格式](https://cloud.tencent.com/developer/article/1575759)
