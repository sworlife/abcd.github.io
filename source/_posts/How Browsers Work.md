---
title: How Browsers Work
date: 2019-07-14 16:33:00
tags: 翻译
categories: 前端技术
comments: true
---

[原文链接](http://taligarsiel.com/Projects/howbrowserswork1.htm)


<a name="6fcf16c7"></a>
## 介绍（Introduction）
Web 浏览器很可能是使用最广泛的应用软件。在这本书里，我将解释其底层是如何工作的。当你在浏览器地址栏里输入 “google.com” ，直到在浏览器里展示出谷歌网页使，我们将清楚浏览器是如何实现的。
<a name="e0a2c782"></a>
### 关于浏览器（The browsers we will talk about）
目前主要有5种使用较广泛的浏览器：Internet Explorer, Firefox, Safari, Chrome and Opera。<br />我将提供一些来自于 Firefox、Chrome 和 Safari 中的部分源码示例。<br />根据 W3C 对浏览器使用情况的统计（2009），使用 Firefox、Safari 和 Chrome 共占比接近 60%。<br />因此，目前开源浏览器在整个浏览器市场中是重要的组成部分。
<a name="3d3e844f"></a>
### 浏览器主要功能（The browser's main functionality）
浏览器主要功能就是把你选择的存在的 web 资源，通过请求从服务器展现到浏览器窗口。资源的格式通常为HTML，但也包括 PDF、图片等。资源的地址由使用者使用 URI 地址指定。更多详情参加相关网络章节。<br />浏览器解析和展现 HTML 文件的方法在 HTML 和 CSS 规范有详细说明。哪些规范由 W3C（Word Wide Web Consortium）组织维护，它是 web 的官方组织。<br />HTML 最新版本是 [4](http://www.w3.org/TR/html401/)。版本 5 正在制定中。CSS 最新版本是 [2](http://www.w3.org/TR/CSS2/)，版本 3 正在制定中。<br />多年来各浏览器只遵从一部分规范，然后开发他们自己的扩展功能，这为 web 开发者导致了严重的兼容性问题。今天大多数浏览器或多或少遵从了规范。<br />各浏览器的用户界面彼此之间有很多相似点，常见的用户界面元素有如下几种：

- 用于填写 URI 的地址栏
- 后退和前进的按钮
- 书签选项
- 一个刷新按钮提供更新功能，一个停止按钮为正在加载中的文档提供停止更新功能
- 首页按钮提供回到首页功能

奇怪的是，浏览器的用户界面没有任何正式的规范来详细说明，它仅仅是过去几年最佳实践的经验以及浏览器间相互模仿造成的。HTML5 规范没有定义一个浏览器必须具备哪些 UI 元素，但列出了一些公共的元素。地址栏、状态栏和工具栏这些位列其中。当然，还有某些浏览的独有功能，例如 Firefox 的下载管理器。更多详情参见用户界面章节。
<a name="3f3258a4"></a>
### 浏览器的高级结构（The browser's high level structure）
浏览器有以下主要组成部分：

1. 用户界面 - 它包括了地址栏、后退/前进按钮、书签菜单等。除了用来展示网页的主窗口以为，其余没一部分都会展示出来。
1. 浏览器引擎 - 用于查询和操作渲染引擎的接口。
1. 渲染引擎 - 负责显示指定的内容。例如，如果要求显示的内容是 HTML，它就负责解析 HTML 和 CSS 并且在屏幕上显示出解析出的内容。
1. 网络 - 用于网络呼叫，例如：HTTP 请求。它有独立于平台的接口和每个平台的底层实现。
1. UI 后端 - 用于绘制基础部件，例如：组合框和窗口。它暴露出未特定平台的通用接口。底层实现使用了操作系统的用户界面方法。
1. JavaScript 解释器 - 用于解析和执行 JavaScript 代码
1. 数据存储 - 这是一个持久性层。浏览器需要再硬盘上保存所有类型的数据，例如：cookies。新 HTML 规范（HTML5）定义的 “web database”在浏览器中是完整（虽然是轻量的）的数据库。

![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1550392520218-f9497554-05ad-412b-8406-b33903f79a2a.png#align=left&display=inline&height=339&name=image.png&originHeight=339&originWidth=500&size=46828&status=done&width=500)<br />**Figure 1：Browser main components.**

值得注意的是，Chrome 不同于大多数浏览器，它的渲染引擎拥有多实例 - 每个页签对应一个实例。每个页签拥有一个自己的进程。<br />我将把以上每个部分都用一个章节来详细讲解。
<a name="fb599fcd"></a>
### 各组成部分之间的联系（Communication between the components）
Firefox 和 Chrome 都开发有特定通信设施。它们将在一个特别的章节进行讲解

<a name="2e656cba"></a>
## 渲染引擎（The rendering engine）
渲染引擎的职责是渲染，就是在浏览器屏幕上呈现出指定的内容。<br />渲染引擎默认能呈现 HTML 和 XML 文档，以及图片。通过一个 plug-in（浏览器扩展）也能呈现其他类型。例如：PDF 使用一个 PDF 查看器扩展就能呈现。我们在一个特别的章节讨论这些插件和扩展。在这一章节我们集中在主要使用的部分——呈现使用 CSS 格式化过的 HTML 和 图片。

<a name="6185d257"></a>
### 渲染引擎（Rendering engines）
我们提到的浏览器：Firefox、Chrome 和 Safari 都建立在两个渲染引擎之上，Firefox 使用了 Gecko（一个“自制”的 Mozilla 渲染引擎）。Safari 和 Chrome 使用了 Webkit。<br />Webkit 是一个开源渲染引擎，它最初是 Linux 平台的一个引擎，后来被 Apple 修改为支持 Mac 和 Windows。在 [https://webkit.org/](https://webkit.org/) 可以看到更多详情。

<a name="8eaf48bf"></a>
### 主要流程（The main flow）
渲染引擎从网络层获取到指定文档的内容时开始工作。这通常在8K的块（每次从网络层中获取的内容）中完成。<br />之后渲染引擎处理的主要流程如下：

    ![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1550462641043-27d55467-8301-4caf-af80-9fae77b2d23d.png#align=left&display=inline&height=66&name=image.png&originHeight=66&originWidth=600&size=18211&status=done&width=600)    <br />**Figure 2：Rendering engine basic flow.**<br />**<br />渲染引擎将以解析 HTML 文档和把标签转换为树（content tree）中的 DOM 节点开始，将解析样式数据，包含外部 CSS 文件和内联样式。样式信息与 HTML 中的视觉指令将被用来生成另一种树 - 渲染树。<br />渲染树包含了一个个带有视觉属性，类似颜色和尺寸的长方形盒子。这些长方形盒子被有序地呈现在屏幕上。<br />渲染树构造好之后，将通过一个布局（layout）的过程。这意味着将给每个节点一个在屏幕上出现的真实的坐标。下一阶段就是绘制（painting）- 渲染树将被遍历以及每个节点都将使用 **UI 后端层**进行绘制。<br />
<br />重要的是要理解这是一个渐进的过程（gradual process）。为了更好的用户体验，渲染引擎会尝试尽快地在屏幕上呈现出内容。它不会等到所有HTML都被解析之后才开始构建和布局渲染树。内容的一部分在解析和呈现，同时也在持续接受网络层传来的剩余内容。<br />**
<a name="553afa95"></a>
### 主要流程示例（Main flow examples）
![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1550470060151-5eb33d8f-2ad9-45cb-ab52-32e1e2bd79a2.png#align=left&display=inline&height=289&name=image.png&originHeight=289&originWidth=624&size=45038&status=done&width=624)<br />**Figure 3: Webkit main flow**<br />**<br />**![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1550470148590-afa67184-1d25-4ba5-ad60-1861e1953cad.png#align=left&display=inline&height=290&name=image.png&originHeight=290&originWidth=624&size=112605&status=done&width=624)**<br />**Figure 4: Mozilla's Gecko rendering engine main flow**<br />**<br />从图3和图4可以看出，Webkit 和 Gecko 使用了略微不同的术语，流程本质上还是一致的。<br />Gecko 把格式化后视觉元素的树称为框架树（Frame tree）。每个元素就是一个框架（Frame）。Webkit 则用术语渲染树（Render Tree）和其组成部分渲染对象（Render Objects）来表示。Webkit 使用术语布局（Layout）表示元素的位置，Gecko 则称为回流（Reflow）。连接器（Attachment）是 Webkit 用来连接DOM节点和可视化信息来创建渲染树的术语。一个较小的非语义的差异就是 Gecko 在 HTML 和 DOM tree 之间额外多了一层，称为内容槽（content sink），是一个标记 DOM 元素的工厂。下面将讨论流程汇总的每一个部分：<br />**
<a name="b1a33cb4"></a>
### 解析和 DOM 树结构（Parsing and DOM tree construction）
<a name="76eff1b6"></a>
#### 常规解析（Parsing - general）
由于解析在渲染引擎中是一个非常重要的过程，因此我们将走得更深入一点。让我们开始简短的介绍下解析部分（paring）。<br />解析一个文档意味着将其翻译成一些有意义的结构 - 一些代码能够理解和使用。解析的结果通常是一个表示文档结构的结点树。我们称为解析树或者语法树。<br />例如：解析表达式“2 + 3 - 1”能返回这个树：<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1550476117790-08dbf402-e174-4085-a9fb-9e048f417a95.png#align=left&display=inline&height=155&name=image.png&originHeight=155&originWidth=400&size=20276&status=done&width=400)<br />**Figure 5: mathematical expression tree node**

<a name="Grammars"></a>
##### Grammars
解析基于文档遵守的语法规则（书写的语言或格式）。每种格式都必须解析出由词汇表和语法规则组成的确定性的语法。这被称为上下文无关语法（context free grammar）。人类语言不是这类语言，因此无法传统的解析技术进行解析。

<a name="9ae131a7"></a>
##### （解析器 - 结合词法分析器）Parser - lexer combination
解析可以被分为两个子流程：词法分析（lexical analysis）和语法分析（syntax analysis）。<br />词法分析是将输入拆分为词（token）的过程。词就是语言中的词汇表（有效构造块的集合）。在人类语言中，就是该语言在词典中出现的所有单词组成的。<br />语法分析是语言语法规则的应用。<br />解析器通常分为两部分：负责将输入拆分为有效词汇的词法分析器（lexer）或分词器（tokenizer），以及负责遵循语言规则分析出文档结构，进而构造出解析树（parse tree）的解析器。词法分析器知道如何去除无关紧要的符号，例如：空格和换行符。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1550482562198-3c0d4d3d-c644-486d-98a7-5240264baae5.png#align=left&display=inline&height=300&name=image.png&originHeight=300&originWidth=101&size=19139&status=done&width=101)<br />**Figure 6: from source document to parse trees**<br />**<br />解析过程是迭代的，解析器通常向词法分析器请求新的词（token），然后尝试匹配一条语法规则，如果与一条规则匹配，与之对应的结点将被添加到解析树中，然后将继续请求另一个词（token）。<br />如果没有匹配到规则，解析器将在内部存储这个词（token），然后继续请求词（token）直到找到匹配内部存储的所有词的规则。如果没有找到规则，那么解析器将引发一个异常，这意味着该文档无效并且包含有语法错误。

<a name="adc66e18"></a>
##### 翻译（Translation）
多数时候解析树不是最终产品，翻译中常使用解析 - 将输入文本翻译为另一种格式。例如汇编。把源码编译成机器码的编译程序首先将其解析成解析树，然后翻译把解析树翻译成机器码文档。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1550484895530-a863f78e-2b6d-410a-bf25-5c25a035bd23.png#align=left&display=inline&height=400&name=image.png&originHeight=400&originWidth=104&size=25384&status=done&width=104)<br />**Figure 7: compilation flow**

<a name="8469736f"></a>
##### 解析示例（Parsing example）
图5中，我们构建了一个数学表达式的解析树，让我们尝试定义一个简单的数学语言来看看解析流程。<br />词汇表：我们的语言包括整数、加号和减号。<br />语法：

    1. 该语言语法构建块是表达式、术语和操作
    1. 该语言能包括任意数量的表达式
    1. 表达式（expression）定义为一个术语（term）后面跟着一个操作（operation），再跟着另一个术语（term）
    1. 操作（operation）是指加号（plus token）或者减号（minus token）
    1. 术语是指一个整数（integer）或者一个表达式（expression）

让我们解析输入“2 + 3 - 1”。<br />第一个子串匹配到规则的是“2”，根据规则#5，它是一个术语。第二次匹配是“2 + 3”匹配到规则#2 - 一个术语后跟着一个操作再跟着另一个术语。下一次匹配将只会在输入的末尾被命中。“2 + 3 - 1”是一个表达式因为我们已经知道“2 + 3”是一个术语，因此我们拥有了一个术语后跟着一个操作再跟着另一个术语。“2 + +”将不能匹配任何规则因此是无效的输入。

<a name="226d81c1"></a>
##### 正式定义词汇和语法（Formal definitions for vocabulary and syntax）
词汇表通常由正则表达式（[regular expressions](https://www.regular-expressions.info/)）表示。<br />例如我们的语言将被定义为：<br />INTEGER：0|[1-9][0-9]*<br />PLUS: +<br />MINUS: -<br />正如你所看到的，整数被定义为一个正则表达式。<br />语法通常被定义为一种 [BNF](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form)（Backus–Naur form） 格式。我们的语言将被定义为：<br />expression := term operation term<br />operation := PLUS | MINUS<br />term := INTEGER | expression

我们说一种语言如果它的语法是无上下文语法，则能通过常规解析器被解析。无上下文语法可简单定义为语法能完全用 BNF 格式表示。正式的定义参加[http://en.wikipedia.org/wiki/Context-free_grammar](http://en.wikipedia.org/wiki/Context-free_grammar)

<a name="3867ff5d"></a>
##### 解析器的类型（Types of parsers）
解析器有两个基本类型 - 自上而下解析器（top down parsers）和自下而上解析器（bottom up parsers）。自上而下解析器可简单定义为解析器查看语法的高级结构，并尝试匹配其中的一个。自下而上解析器从输入开始，逐步转换为语法规则，从低级规则开始，直到能匹配高级语法为止。<br />来看看这两种解析器如何解析我们的示例：<br />自上而下解析器从最高级规则开始，它将定义“2 + 3”是一个表达式。然后定义“2 + 3 - 1”是一个表达式（定义表达式的过程会逐步匹配其他规则，但起点是最高级规则）。<br />自下而上解析器将浏览输入直到与一个规则匹配，然后用该规则替换匹配的输入。这个过程将持续到输入的结束。部分匹配出的表达式放在了解析栈上。

| **Stack** | **Input** |
| :---: | :---: |
|  | 2 + 3 - 1 |
| term | 2 + 3 - 1 |
| term operation | 3 - 1 |
| expression | -1 |
| expression operation | 1 |
| expression |  |

这种自下而上解析器被称为渐变减少解析器（shif reduce parser）,因为输入被向右平移（想象下指针从输入的开始向右移动）并逐步简化为语法规则。

<a name="eb0a8c3e"></a>
##### 自动生产的解析器（Generation parsers automatically）
有些工具能为你生成解析器。它们被称为解析生成器。你提供语言的语法给它们 - 词汇表和语法规则，然后就生成一个能工作的解析器。创建一个解析器需要深入理解解析原理并且不容易手动创建出优化的解析器，因此解析生成器非常有用。<br />Webkit 使用了两个非常好的解析生成器 - 创建词法分析程序的Flex和创建解析器的Bison（你可能会遇到它们的名称是Lex 和 Yacc）。Flex 的输入是一个包含用正则表达式定义词（token）的文件。Bison 的输入是用 BNF 格式表示的语言语法。

<a name="3fc06d7f"></a>
#### HTML 解析器（HTML Parser）
HTML 解析器的工作是把 HTML 标记解析成解析树。

<a name="b0887e49"></a>
##### HTML 语法定义（The HTML grammer definition）
HTML 的词汇表和语法[规范](http://taligarsiel.com/Projects/howbrowserswork1.htm#w3c)由 W3C 组织创建。现在的版本是HTML4，HTML5正在制定中。

<a name="0355a922"></a>
##### 不是上下文无关语法（Not a context free grammer）
正如我们在解析器介绍中看到，语法的语法能使用 BNF 等格式进行正式定义。<br />不幸的是所有传统的解析器主题都不适用于 HTML （我提到它们不是为了好玩，它们将被用于 CSS 解析和 JavaScript 解析）。HTML 不能简单地由解析器需要的上下文无关语法进行定义。<br />DTD（Document Type Definition）是一个定义 HTML 的正式格式，但不是一个上下文无关语法。<br />这在第一个站点上看起来很奇怪——HTML 非常接近 XML，有许多可用的 XML 的解析器。HTML-XHTML是一个 XML 的变体，因此它们最大区别是什么呢？<br />区别就是 HTML 更“宽容”，它默默地加上你疏忽掉特定的标记，有时省略开始或结束标记等。总的来说，它是一种“软”语法，与 XML 的硬和苛刻的语法相反。<br />很明显，这个看起来很微小的不同使得世界变得不同。一方面这是 HTML 如此流行的主要原因——它忽略你的错误，使得 web 作者的生活更容易；另一方面，它使得语法格式很难写。总结下——HTML 不容易被解析，自从它的语法不是上下文无关语法开始，既不适用于传统解析器，也不适用于 XML 解析器。

<a name="c3a151fc"></a>
##### HTML DTD
HTML定义在一个 DTD 格式里。这种格式被用于定义 [SGML](https://en.wikipedia.org/wiki/Standard_Generalized_Markup_Language) 家族语言。这种格式包含所有元素及其属性和层次结构的定义。正如我们前面看到的，HTML DTD 不是上下文无关语法。<br />DTD 有几种变体。严格模式严格遵守规范，但其他几种模式包含对浏览器对过去使用的标记的支持。目的是向后兼容旧的内容。现在的严格 DTD 地址：[http://www.w3.org/TR/html4/strict.dtd](http://www.w3.org/TR/html4/strict.dtd)

<a name="DOM"></a>
##### DOM
输出的树——解析树是 DOM 元素和属性的节点组成的树。DOM是 Document Object Model 的简称。它是 HTML 文档的对象表示和 HTML 元素与外部世界（如 JavaScript）的接口。<br />这个树的根节点是 [Document](https://www.w3.org/TR/1998/REC-DOM-Level-1-19981001/level-one-core.html#i-Document) 对象。<br />DOM 与标记几乎是一对一的关系。例如，这些标记：
```html
<html>
	<body>
		<p>
			Hello World
		</p>
		<div> <img src="example.png"/></div>
	</body>
</html>
```

将被翻译为下面这样的DOM树：

    ![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1550566879833-9e2a9d27-ec5f-4a17-8669-cf8a6d42c474.png#align=left&display=inline&height=219&name=image.png&originHeight=219&originWidth=400&size=36649&status=done&width=400)<br />**Figure 8: DOM tree of the example markup**

如何 HTML，DOM 也由 W3C 组织具体说明，可见：[https://www.w3.org/DOM/DOMTR](https://www.w3.org/DOM/DOMTR)。这是一个操作文档的通用规范。特定的模块描述 HTML 特定元素。HTML 规范可在这里找到：[https://www.w3.org/TR/2003/REC-DOM-Level-2-HTML-20030109/idl-definitions.html](https://www.w3.org/TR/2003/REC-DOM-Level-2-HTML-20030109/idl-definitions.html)。<br />当我看说树中包含了 DOM 节点，我的意思是树由某个 DOM 接口的元素构建的。浏览器使用浏览器内部具有其它属性的具体实现。

<a name="ff00c3f3"></a>
##### 解析算法（The parsing algorithm）
正如在前面几节中看到的，HTML 不能被通常的自上而下的解析器解析。<br />原因如下：

    1. 语言的宽容本性
    1. 浏览器需要具有传统的错误容忍度，以支持已知的无效HTML的事实。
    1. 解析过程可重入（可重入(**Reentrant**)：函数可以由多于一个线程并发使用，而不必担心数据错误。可重入函数可以在任意时刻被中断，稍后再继续运行，不会丢失数据。）。通常资源在解析过程中不会改变，但在 HTML 中，包含“document.write”的 script 标签能添加额外的词（tokens），因此解析过程实际上会修改输入。

由于不能使用常规解析器技术，浏览器为解析 HTML 创建了自定义的解析器。<br />这个解析算法描述的是 HTML5 规范的细节。这个算法由**标记**（tokenization）和构建树两个阶段组成。<br />标记是把输入通过词法分析、解析转变成**标记**（tokens）。**HTML 标记**包含开始标记、结束标记、属性名称和属性值。<br />分词器识别出标记，将其交给构建树，然后使用下一个字符识别出下一个标记，以此类推，直到输入结束。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1550629215999-29aa6106-2b07-4b47-a084-6cacd12a174c.png#align=left&display=inline&height=400&name=image.png&originHeight=400&originWidth=308&size=35956&status=done&width=308)<br />Figure 6：HTML parsing flow（taken from HTML5 spec）<br />

<a name="7c52a2b1"></a>
##### 标记算法（The tokenization algorithem）
该算法输入一个 HTML 词，该算法用状态机（state machine）表示。每个状态消耗输入流的一个或多个字符，并且根据那些字符更新到下一个状态。该决策收到当前标记化状态和树构造状态的影响。这意味着根据当前的状态，消耗同一个字符将为正确的下一个状态产出不同的结果。该算法太复杂而不能完全引入，因此让我们通过一个简单的示例帮助我们理解原理。<br />基础示例 - 标记下面的 HTML:
```html
<html>
	<body>
		Hello world
	</body>
</html>
```

<br />初始状态是“Data state”，当遇到字符“<”时，状态将被改变为“Tag open state”。继续消耗一个“a-z”的字符会创建一个“Start tag token”（开始标记），状态变为“Tag name state”。一直保持在这个状态直到字符“>”被消耗。每个字符都会附件到新的标记名称上。在我们的例子中，创建的标记是一个 “html”标记。<br />当“>”标签到达是，当前标记被发出并且状态变回到“Data state”。“body” 标签将被相同的步骤处理。到目前为止，“html”和“body”都已发出。现在回到“Data state”状态。消费“Hello world”中的“H”将创建并发出一个字符标记，一直持续到“</body>”中的“<”到达。这期间我们发出了“Hello world”中每一个字符的标记。现在我们回到“Tag open state”状态。继续消费下一个输入“/”将创建一个“end tag token”（结束标记）并且移动到“Tag name state”状态。再一次保持这个状态知道“>”到来。然后一个新标签标记将被发出，且回到“Data state”状态。输入的“</html>”将像之前的例子一样被处理。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1550632809798-c9530ab3-047c-45b9-8e72-28721ead5e19.png#align=left&display=inline&height=387&name=image.png&originHeight=387&originWidth=627&size=26067&status=done&width=627)<br />**Figure 9: Tokenizing the example input**<br />**
<a name="3340002d"></a>
##### 构造树算法（Tree construction algorithm）
当解析器被创建时，文档对象也被创建。在树构造阶段，将修改文档根目录中的 DOM 树，并向其添加元素。分词器发出的每个节点都将由树构造处理。每个标记都有 DOM 元素的规范定义与之对应并为其构建。除非添加到DOM 树的元素也添加开放元素的堆栈中了。这个栈被用于纠正嵌套不匹配和未关闭的标签。该算法也描述为一个状态机。这些状态被称为插入模式（insertion modes）。<br />让我们来看看实例输入的树构造过程：
```html
<html>
	<body>
		Hello world
	</body>
</html>
```
树构造阶段输入是一系列来自标记阶段的标记，第一个模式是初始模式（initial mode）。接收到这个html 标记将导致切换到“before html”模式并且在该模式下对标记重新处理。这将导致HTMLHtmlElement 元素诞生并且它将被添加到文档对象（Document Object）的根节点。<br />状态将切换到“before head”。我们接收到“body”标记，尽管我们没有“head”标记，HTMLHeadElement 元素仍会被隐含的创建并添加到树中。<br />现在我们切换到“in head”模式，然后切换到“after head”。这个 body 标记被重新加工，HTMLBodyELement 被创建并插入，且模式转换为“in body”。<br />“Hello world” 字符串的字符标记现在被接收到，第一个标记将导致一个“Text”节点被创建和插入，其他字符将被添加到这个节点上。<br />接收到body的结束标记将导致转换到“after body”模式。我们将接收到html 结束标签将切换到“after after body”模式。接收到文件结束的标记将结束解析。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1550647262722-5ddbc38a-858a-48c1-9da2-6407eff10001.png#align=left&display=inline&height=769&name=image.png&originHeight=769&originWidth=532&size=33483&status=done&width=532)<br />**Figure 10: tree construction of example html**

<a name="02ddcf2a"></a>
##### 解析完成之后的行为（Actions when then parsing is finished）
解析完成之后浏览器将文档标记为交互式的，并开始解析“deferred”模式中的脚本（scripts）——哪些应该在文档被解析之后被运行。文档状态之后将被设置为“complete”，且“load”事件将被激活。<br />你将在[HTML5 规范](https://www.w3.org/TR/html5/syntax.html#html-parser)中看到标记、书构造的完整算法。
<a name="ae17b7a0"></a>
##### 浏览器容错处理（Browsers error tolerance）
你绝不会在一个HTML页面获取到“Invalid Syntax”错误。浏览器将修复无效的内容，然后继续。拿这段HTML举例：

```html
<html>
  <mytag>
  </mytag>
  <div>
  <p>
  </div>
  	Really lousy HTML
  </p>
</html>
```
我必须违反上百万的规则（“mytag”不是标准标签，“p”的“div”错误嵌套），但浏览器仍然正确的展示并且没有提出抗议。因此很多解析代码都是为了修复HTML作者的错误。<br />在不同浏览器中，错误处理非常一致，但令人惊讶的是它不是HTML当前规范的一部分。就像书签和后退/前进按钮，它只是多年来浏览器开发出来的东西。他们知道在很多站点重复出现无效的HTML结构，然后和其它浏览器以一种符合要求的方式尝试去修复它们。<br />HTML5 规范确实定义其中一些需求。Webkit 在HTML 解析类开头的注释中很好的总结这一点。
> 这个解析器把标记化的输入解析到文档中，创建出文档树。如果这个文档格式良好，就直接地解析它。
> 不幸的是，我们必须处理很多那些格式不够良好的文档，因此解析器必须能够容忍错误。
> 我们必须至少关心下面几种错误情形：
> 1. 正添加到某些外部标签的元素是被显示禁止的。这种情况，我们应该关闭被禁止元素之前的所有标签，然后将元素添加其后。
> 1. 我们不允许直接地添加元素。可能写文档的人忘记了中间的某些标签（或其中的标签是可选的）。这可能是以下标签的情况：HTML HEAD BODY TR TD LI （我还忘记了哪些？）
> 1. 我们想添加块级元素到一个行级元素中。关闭所有行级元素直到下一个更高级的块元素。
> 1. 如果这没有用，关闭元素直到允许被添加的元素或者忽略这个标签。


让我们看一些Webkit 容错处理的例子：<br />**</br> 替代 <br>**<br />一些网站使用</br>替代<br>。为了被IE兼容，FIrefox Webkit 如此处理像<br>等标签。
```cpp
if (t->isCloseTag(brTag) && m_document->inCompatMode()) {
     reportError(MalformedBRError);
     t->beginTag = true;
}
```
注意——这个错误处理是内部的——它不会提供给用户。

- 走失的表格（a stray table）

走失的表格是指一个表格放在了另一个表格中，但确没有放在单元格中，举例如下：

```html
<table>
	<table>
		<tr><td>inner table</td></tr>
         </table>
	<tr><td>outer table</td></tr>
</table>
```

Webkit 将修改这个结构变成两个兄弟表格：

```html
<table>
	<tr><td>outer table</td></tr>
</table>
<table>
	<tr><td>inner table</td></tr>
 </table>
```

代码如下：

```cpp
if (m_inStrayTableContent && localName == tableTag)
        popBlock(tableTag);
```

Webkit 使用一个栈来保存当前元素内容——它将把里面的table从外部table栈中弹出。两个table之后便是兄弟关系了

- 嵌套的表单元素（Nested form elements）

这种情况是指用户将一个form 放入另一个form 中，第二个form 会被忽略。代码如下：

```cpp
if (!m_currentFormElement) {
        m_currentFormElement = new HTMLFormElement(formTag,    m_document);
}
```


- 太深的标签层次结构（A too deep tag hierarchy）

这个注释不言自明。
> www.liceo.edu.mx 这个站点就是一个例子，它达到了 1500个标记的嵌套级别，全部来自遗传 <b>。我们仅允许大约 20 个相同类型的嵌套标签，然后将它们全部忽略。

```cpp
bool HTMLParser::allowNestedRedundantTag(const AtomicString& tagName)
{

unsigned i = 0;
for (HTMLStackElem* curr = m_blockStack;
         i < cMaxRedundantTagDepth && curr && curr->tagName == tagName;
     curr = curr->next, i++) { }
return i != cMaxRedundantTagDepth;
}
```

- 错误放置的html或body的结束标签（Misplaced html or body end tags）
同样，也有相应的注释说明。
> 为支持不完整的html
> 我们从不关闭body标签，除非一些愚蠢的web页面在文档实际结束之前关闭它。我们依靠 end() 的调用来关闭这些东西。
> 

```cpp
if (t->tagName == htmlTag || t->tagName == bodyTag )
        return;
```
因此 web 作者要小心——除非你想在Webkit 容错处理的代码中作为实例出现——编写格式良好的HTML。

<a name="e055ea7a"></a>
#### CSS 解析（CSS parsing）
还记得介绍中的解析概念么？哦，这不像 HTML，CSS 是一个上下文无关的语法，也能使用介绍中的那类解析器来解析。事实上，CSS 规范定义了 CSS 词汇和语法（[http://www.w3.org/TR/CSS2/grammar.html](http://www.w3.org/TR/CSS2/grammar.html)）。

让我们来看一些实例：<br />词汇语法（词汇）通过正则表达式定义了每一个词：

```
comment		\/\*[^*]*\*+([^/*][^*]*\*+)*\/
num				[0-9]+|[0-9]*"."[0-9]+
nonascii	[\200-\377]
nmstart		[_a-z]|{nonascii}|{escape}
nmchar		[_a-z0-9-]|{nonascii}|{escape}
name			{nmchar}+
ident			{nmstart}{nmchar}*

"ident" 是 identifier 的缩写，例如一个类名。"name" 是元素 id（它由"#"引用）
```

语法使用 BNF 描述
> - *****: 0 or more
> - **+**: 1 or more
> - **?**: 0 or 1
> - **|**: separates alternatives
> - **[ ]**: grouping

```
ruleset
  : selector [ ',' S* selector ]*
    '{' S* declaration [ ';' S* declaration ]* '}' S*
  ;
selector
  : simple_selector [ combinator selector | S+ [ combinator selector ] ]
  ;
simple_selector
  : element_name [ HASH | class | attrib | pseudo ]*
  | [ HASH | class | attrib | pseudo ]+
  ;
class
  : '.' IDENT
  ;
element_name
  : IDENT | '*'
  ;
attrib
  : '[' S* IDENT S* [ [ '=' | INCLUDES | DASHMATCH ] S*
    [ IDENT | STRING ] S* ] ']'
  ;
pseudo
  : ':' [ IDENT | FUNCTION S* [IDENT S*] ')' ]
  ;
declaration
  : property ':' S* expr prio?
  ;
property
  : IDENT S*
  ;
prio
  : IMPORTANT_SYM S*
  ;
expr
  : term [ operator? term ]*
  ;
term
  : unary_operator?
    [ NUMBER S* | PERCENTAGE S* | LENGTH S* | EMS S* | EXS S* | ANGLE S* |
      TIME S* | FREQ S* ]
  | STRING S* | IDENT S* | URI S* | hexcolor | function
  ;
function
  : FUNCTION S* expr ')' S*
  ;
/*
 * There is a constraint on the color that it must
 * have either 3 or 6 hex-digits (i.e., [0-9a-fA-F])
 * after the "#"; e.g., "#000" is OK, but "#abcd" is not.
 */
hexcolor
  : HASH S*
  ;
```

说明：ruleset 是下面这样的结构：

```css
div.error, a.error {
	color: red;
  font-weight: bold;
}
```

div.error 和 a.error 都是 selector。在花括号中的部分包含通过这个 ruleset 应用的规则。它的结构被正式定义在这个描述中：

```
ruleset
  : selector [ ',' S* selector ]*
    '{' S* declaration [ ';' S* declaration ]* '}' S*
  ;
```

这意味着一个 ruleset 是一个 selector 或者由逗号和空格分割的多个 selector（S 代表空格）组成。一个 ruleset 包含一对花括号且花括号中间是一个 declaration 或者由分号分割的多个 declaration。"declaration" 和 "selector" 将被定义在接下来的 BNF 定义中。

<a name="5cda2ac8"></a>
##### Webkit CSS 解析器（Webkit CSS parser）
Webkit 使用 [Flex and Bison](http://taligarsiel.com/Projects/howbrowserswork1.htm#parser_generators) 生成器根据 CSS 语法文件自动创建解析器，正如你从解析器介绍中回想起的一样，Bison 创建一个自下而上的渐变减少解析器（shif reduce parser）。Firefox 使用手动编写的自上而下的解析器。两种情况都会把每个 CSS 文件解析成一个样式表对象（StyleSheet Object），每个对象包括了 CSS 规则。CSS 规则又包含了选择器（selector）、声明对象（declaration objects）以及其他 CSS 语法相关的对象。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1551749965864-b738d12d-7324-4c32-8348-f52989b8d99c.png#align=left&display=inline&height=393&name=image.png&originHeight=393&originWidth=500&size=50124&status=done&width=500)<br />**Figure 7：parsing CSS**<br />**
<a name="5f765268"></a>
#### 脚本解析（Parsing scripts）
这部分将在 JavaScript 章节处理。

<a name="ec21b793"></a>
#### 处理脚本和样式表的顺序（The order of processing scripts and style sheets）
<a name="8f465117"></a>
##### 脚本（Scripts）
web 的模型是同步的。作者期望当解析到 <script> 标记时，脚本将被解析并立即执行。将使文档的解析停止，直到脚本被执行。如果是外部脚本，该资源必须第一时间从网络抓取进来——做这些仍然是同步的。使解析停止，直到资源被请求到。这是多年来的模式，并被定义在 HTML4 和 5的规范中。

作者能把脚本标记为 "defer" ，因此它不会使文档解析停止并且将在解析完成之后才被执行。HTML5 新增一个选项用来标记异步脚本，因此它将通过**不同的线程**被解析和执行。

<a name="4c9371ec"></a>
##### 投机解析（Speculative parsing）
Webkit 和 Firefox 都做了优化，当执行脚本时，其他线程解析文档剩下的部分并且找出需要从网络中加载的其他资源并且加载它们。这些方式使得资源可以并行加载并且总体速度更好。注意——投机解析不会改变 DOM 树结构，并将其留给主解析器，它仅解析指定的外部资源，如外部脚本、样式表和图片。

<a name="55291784"></a>
##### 样式表（Style sheets）
另一方面，样式表具有不同的模型，概念上来讲，样式表似乎不会改变 DOM 树，也就没有理由停止文档解析来等待样式表。然而，在文档解析阶段，脚本需要样式信息。如果样式还没有被加载和解析，脚本将得到错误的结果并且这明显地导致很多问题。这似乎是一个边缘情况但确很常见。当样式表被加载和解析时，Firefox 将阻塞所有脚本。Webkit 只在脚本试图访问某些确定样式属性时，才会阻塞脚本，这些属性可能会受到未加载的样式表的影响。

<a name="0771ae8d"></a>
### 构造渲染树（Render tree construction）
当DOM 树被构建好后，浏览器将构建另一个树，渲染树（render tree）。这个树是有特定顺序的会被呈现的视觉元素。这个视图代表了文档。这个树的目的是能以正确的顺序绘制内容。

Firefox 称渲染树中的这些元素为"frames"。Webkit 使用的术语是"renderer" 或者 "render object"。

一个 "renderer" 知道如何布局（layout）和 绘制（paint）它自己以及他的子级（children）。

Webkits RenderObject 类，也就是 "renderers" 的基类定义如下：

```cpp
class RenderObject{
	virtual void layout();
	virtual void paint(PaintInfo);
	virtual void rect repaintRect();
	Node* node;  //the DOM node
	RenderStyle* style;  // the computed style
	RenderLayer* containgLayer; //the containing z-index layer
}
```

每一个 renderer 相当于一个长方形区域，通常与节点的 CSS 盒相符，这在 CSS2 规范中有相应的描述。它包含了几何信息，如宽、高和位置。这个盒子的类型受与节点有关的 "display" 样式属性的影响（参见[style computation](http://taligarsiel.com/Projects/howbrowserswork1.htm#style_computation) 部分）。下面是一份 Webkit 代码，它通过"display"属性来决定渲染器（renderer）应该为一个DOM节点创建什么类型。

```cpp
RenderObject* RenderObject::createObject(Node* node, RenderStyle* style)
{
    Document* doc = node->document();
    RenderArena* arena = doc->renderArena();
    ...
    RenderObject* o = 0;

    switch (style->display()) {
        case NONE:
            break;
        case INLINE:
            o = new (arena) RenderInline(node);
            break;
        case BLOCK:
            o = new (arena) RenderBlock(node);
            break;
        case INLINE_BLOCK:
            o = new (arena) RenderBlock(node);
            break;
        case LIST_ITEM:
            o = new (arena) RenderListItem(node);
            break;
       ...
    }

    return o;
}
```

元素类型也需要被考虑，例如表单控件（controls）和表格（tables）有特殊的框架。在 Webkit 中，如果一个元素想创建一个特殊的渲染器（renderer），它将覆盖 "createRenderer" 方法。这些渲染器 （"renderers"）指向包含非几何信息的样式对象。

<a name="9aac2e2a"></a>
#### 渲染树与DOM树的关系（The render tree relation to the DOM tree）
渲染器与DOM树想对应，但不是一一对应。不可见的DOM元素不会被插入渲染树中，例如："head" 元素。display 属性值为"none"的元素也不会出现在树中（visibility 属性值为"hidden"的元素会出现在树中）。

DOM 元素对应几个视觉对象。通常具有复杂结构的元素不能通过单一的长方形进行描述。例如，"select" 元素有3个渲染器——一个用于呈现区域，一个用于下拉列表，一个用于按钮。当文本因宽度不够而打断成多行时，被添加的新行需要额外的渲染器。多渲染器的另一个例子是损坏的HTML。根据 CSS 规范，一个行内元素必须只包含块级元素或行内元素。对于混合的内容，将创建一个匿名的块级渲染器来包裹行内元素。

一些 render objects 对应一个 DOM 结点，但不在树中的同一位置。浮动和决定定位的元素脱离出正常流，放置在树中的不同位置，并映射到一个真实的框架。一个占位框架出现在该出现的地方。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1551925579994-391aa64b-a523-4268-a353-877c2cc7ab42.png#align=left&display=inline&height=396&name=image.png&originHeight=396&originWidth=731&size=55206&status=done&width=731)

**Figure 11：渲染树和 匹配的DOM树。"Viewport" 是初始内容块。在 Webkit 中它就是"RenderView" 对象。**<br />**
<a name="205ce413"></a>
#### 构建树的流程（The flow of constructing the tree）
在 FIrefox 中，其外表（presentation）被注册为监听DOM更新的侦听器。外表（presentation）把框架（frame）创建委托给"FrameConstructor"，并且由它来解决样式（参看 style computation）和创建框架。

在 Webkit 中，把解决样式和创建渲染器（renderer）称为 "attachment"。每个 DOM 节点都有 "attach" 方法。Attachment 是同步的，节点插入 DOM 树时，调用新节点的 "attach" 方法

在构建过程中，把 html 和 body 标签处理成渲染树的根节点。根节点的渲染对象与CSS 规范中的包裹块（containing block）相匹配，——是包含所有其他块的最顶层块。它的尺寸就是视窗——浏览器窗口显示区域的尺寸。Firefox 称之为 ViewPortFrame，Webkit 称之为 RenderView。这就是文档指向的渲染对象。树的其余部分在DOM节点插入时构造。参看 CSS2 的这个主题页——[http://www.w3.org/TR/CSS21/intro.html#processing-model](http://www.w3.org/TR/CSS21/intro.html#processing-model)

<a name="f410803d"></a>
#### 样式计算（Style Computation）
创建渲染树需要计算每个渲染对象的视觉特性。这是通过计算每个元素的样式特性来完成的。

样式包含各种来源的样式表，行内样式元素和HTML中的视觉特性（如“bgcolor”属性）。后者被转换为匹配CSS样式的属性。

这些样式表来源都是浏览器的默认样式表。样式表通过页面作者和用户样式表——这些样式表由浏览器用户提供（浏览器允许你定义自己喜欢的样式。例如在 Firefox 中，通过将样式表放在“Firefox Profile”文件夹中来实现）来提供。

样式计算带来以下几点困难：

1. 样式数据有非常大的结构，持有大量样式属性可能导致内存问题。
1. 为每个元素找出匹配的规则时，如果不做优化，将导致性能问题。遍历全部的规则列表以找出每个元素的匹配项是一个繁重的任务。选择器可以有复杂的结构，这会导致匹配过程从一个看起来有希望的路径开始，而这个路径但被证明是无效的，而必须尝试另一个路径。例如——下面的复合选择器：

```html
div div div div{
	...
}
```

表示这个规则适用于 "<div>"，它是 3 个 divs 的后代。假设你想检查是否规则适用于给定的 "div" 元素。你选择树上树上的一个特定的路径去检查。你可能需要去遍历节点树，仅找到两个 divs 且规则不适用。你可能需要尝试树中的其他路径。
3. 应用规则涉及十分复杂的级联规则，这些级联规则定义规则的层次结构。

让我们来看看浏览器如何面对这些问题：

<a name="0458e9d9"></a>
##### 共享样式数据（Sharing style data）
Webkit 节点引用样式对象（style Objects）（RenderStyle），在某些条件下这些对象由不同节点共享。这些节点都是同级关系，并且：

1. 这些元素必须处于相同的鼠标状态（例如：不愿写一个是 :hover 状态，而另一个不是）
1. 任何元素都没有 id 
1. 标签名称应该匹配
1. 类属性应该匹配
1. 映射属性集必须相同
1. 连接状态必须匹配
1. 集合状态必须匹配
1. 没有元素应该被属性选择器影响，这里说的受影响是指在选择器中任意位置使用属性选择器的选择器匹配。
1. 元素中不能有 inline 样式属性
1. 不能有同级选择器，WebCore 在遇到任何同级选择器时，只会引发一个全局开关，并停用整个文档的样式共享（如果存在）。这包括 + 选择器以及 :first-child 和 :last-child 等选择器。
<a name="3042eaae"></a>
##### Firefox 规则树（Firefox rule tree）
FIrefox 为了更容易计算样式，有两个额外的树——规则树(rule tree)和样式上下文树(style context tree)。Webkit 也有样式对象，但它们没有用树结构（比如：样式上下文树）进行存储，仅用DOM 节点指向其相关的样式。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1552185108339-7ecd8d4c-d059-4c13-ad40-ef354618b485.png#align=left&display=inline&height=407&name=image.png&originHeight=407&originWidth=640&size=13250&status=done&width=640)<br />**Figure 13: Firefox style context tree**<br />样式上下文包含终值。这些值是用正确的顺序应用了匹配出的所有规则进行计算，并且执行了把逻辑值转换为具体值的操作。比如：如果逻辑值是屏幕的百分比，它将被计算和转换成绝对单位。规则树的想法非常聪明。它使得节点之间能共享这些值，避免了再次计算它们。这也节省了空间。

所有匹配出的规则都被存储在一个树结构中。一个路径中底部节点拥有更高权重。这个树包含了能找到的与规则匹配的所有路径。存储这些规则是惰性的。该树不会在一开始就为每一个节点计算的，而是节点样式需要被计算时，才将计算出的路径添加到树中。

这个方法可看成把树结构的路径作为词汇表中的词。来看看这个已经计算好的规则树。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1552187524853-7493af40-b25a-4661-956a-87ae0586d997.png#align=left&display=inline&height=261&name=image.png&originHeight=261&originWidth=400&size=25826&status=done&width=400)<br />假设我们需要为内容树中另一个元素匹配出规则，并且找到的匹配规则（正确的顺序）是 B-E-I。我们在树中已经有这个路径了，因为我们已经计算过 A-B-E-I-L 的路径。我们现在将有少部分工作可做。

让我们看看这个树结构如何减少部分工作。

- **结构体划分（Division into structs）**

样式上下文被拆分成结构体。这些结构体包含特定类型（如：border 或者 color）的样式信息。结构体中的所有属性都是继承的或非继承的。继承属性除非元素定义的属性，否则继承制它们的父级。非继承属性（称为重置属性）如果没有定义则使用默认值。<br />该树结构通过把所有结构体（包括计算出的终值）存储在树结构中来帮助我们。这个方法就是如果底部节点没有为结构体应用一个定义，在上部节点中的一个存储结构能被使用。

- **使用规则树计算样式上下文（Computing the style contexts using the rule tree）**

当为确定的元素计算样式上下文时，我们首先计算出在规则树中的路径或者使用一个已经存在的。然后开始在路径中应用规则，填充结构体到我们的新的样式上下文中。我们从路径中底部节点开始——有最高优先级的那个（通常是最明确的选择器），然后遍历树结构直到结构体被填满。如果该节点的结构体没有规范，我们就能够大大的优化——去树结构上，直到我们找到明确说明的节点然后简单的指向它——这就是最好的优化——所有结构体都被共享。这节省了最终值的计算和内存。<br />如果我们找到了部分定义，则向上遍历这个树结构指定结构体填充完毕。<br />如果我们没有为结构体找到任何定义，这种情况的结构体是 "inherited" 类型——我们指向这个结构体的在内容树中的父级，这时我们也成功的共享了结构体。如果它是 reset 结构体，就会使用默认值。<br />如果最特定的节点添加了值，我们就需要一些额外的计算来把它转换为实际值。然后存储结果到这个树的节点，让它能被其子节点使用。<br />一个元素有指向同一个树节点的同级或同级元素，则它们之间可以共享整个样式上下文。

来看一个示例：假如有下面的 HTML

```html
<html>
<body>
  <div class="err" id="div1">
    <p>
    this is a <span class="big"> big error </span>
    this is also a
    	<span class="big"> very  big  error</span> error
    </p>
  </div>
  <div class="err" id="div2">another error</div>
</body>
</html>
```
并且有以下规则

```css
div {margin:5px;color:black}
.err {color:red}
.big {margin-top:3px}
div span {margin-bottom:4px}
#div1 {color:blue}
#div 2 {color:green}
```
为简化过程，我们只需要填充两个结构体——color 结构体 和 margin 结构体。color 结构体只包含一个成员—— color，margin 结构体包含了四个边。

规则树的结果看起来像这样（这些节点用“节点名称 ：所指向的规则”来标记）<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1552205532294-dde631c2-b026-47b9-bc08-fba3722a6863.png#align=left&display=inline&height=294&name=image.png&originHeight=294&originWidth=500&size=32298&status=done&width=500)<br />**Figure 12：The rule tree**<br />上下文树如下所示（节点名称：指向规则的节点）：<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1552206261003-6ac07e27-5147-4078-ac45-67397eea00f9.png#align=left&display=inline&height=305&name=image.png&originHeight=305&originWidth=400&size=35349&status=done&width=400)<br />**Figure 13: 上下文树**<br />**<br />假如我们解析HTML到第二个 <div> 标签，我们需要为和这个节点创建一个样式上下文，并填充他的样式结构体。

我们将匹配规则，发现为这个 <div> 标签匹配出的规则是1，2 和 6。也就是说，在树结构中的路径已经存在了，因此我们的元素可以使用，我们仅需要为规则6添加另一个节点到结构树中（规则树中的F节点）。

我们将创建一个样式上下文并把它放到上下文树中。这个新的样式上下文将指向规则树中的F节点。

我们现在需要填充样式结构体。我们将从填充 margin 结构体开始。因最后的规则节点（F）没有添加 margin  结构体，我们可以向上追溯规则树，直到找到一个之前插入节点计算出的缓存结构体并使用它。我们将在节点 B 找到它，它是最上层节点定义的 margin 规则。

我们已经有一个color 结构体的定义了，因此不能使用缓存的结构体。因 color 有一个属性，我们不需要向上追溯规则树来填充其他属性。我们将计算终值（字符 转换为 RGB 等）并在这个节点上缓存计算出的结构体。

解析第二个 <span> 元素的工作甚至更简单。我们将匹配这些规则并且最终它指向规则 G，就像之前的 span。因我们有指向同一个节点的同级元素，能够共享全部样式上下文，仅需指向之前 span 的上下文即可。

为了包含继承自父级规则的结构体，在上下文树上做缓存（color 属性实际上是继承的，但 FIrefox 作为 reset 处理它，并且在规则树上缓存它）。

例如，如果我们在一个段落中位字体增加一些规则：
```css
p {font-family:Verdana;font size:10px;font-weight:bold}
```
然后在上下文树中，div 元素作为该段落的子级，能够与它的父级共享一样的 font 结构体。前提是没有为这个 div 定义 font 规则。

在 Webkit 中没有规则树，匹配声明被遍历4次。首先不重要且高优先级属性（因其他属性依赖，而被首先应用的属性——如：display）被应用，接着高优先级且重要，然后正常优先级且不重要，再然后正常优先级其重要的规则。这就是说，多次出现的属性将根据正确的层叠顺序被解析。最后出现的生效。

因此概况为——共享样式对象（结构体中全部或者一些）解决了问题 1 和 3。FIrefox 的规则树也有助于以正确的顺序应用属性。
<a name="6045ba44"></a>
##### 操纵规则以简化匹配（Manipulating the rules for an easy match）
样式规则有以下几种资源：

- CSS 规则，要么在外部样式表中，要么在 style 元素汇总。
```css
p {color:blue}
```

- 内联样式属性，例如：
```html
<p style="color:blue" />
```

- HTML 可视化属性（被映射成相关的样式规则）
```html
<p bgcolor="blue" />
```

最后两个很容易和元素匹配，因为元素拥有样式属性，并且 HTML 属性以元素为 key 就能被映射。

正如之前在问题 #2 中提到的，CSS 规则匹配很困难。为解决这个困难，为更容易访问，对规则进行了操作。

解析样式表之后，根据选择器规则被添加到几个散列表中，通过 id，类名，标签名进行映射，也有不符合这些类别的通用映射。如果选择器是一个 id，规则将被添加到 id 映射中，如果是类名则被添加到类名映射中，等等。

这个操作使得更容易匹配规则，将不需要查看每个的声明——我们能从映射表中为一个元素提取规则。这个优化提出95%以上的规则，以至于它们甚至不需要考虑匹配过程。

让我们以下面的样式规则举例：
```css
p.error {color:red}
#messageDiv {height:50px}
div {margin:5px}
```

第一个规则将被插入类名映射，第二个插入 id 映射，第三个插入 标签映射。

HTML 片段如下

```html
<p class="error">an error occurred </p>
<div id=" messageDiv">this is a message</div>
```

我们首先尝试为元素 p 查找规则。类名映射将包含一个 “error” 键，这个键下可以找到 “p.error” 规则、div 元素相关的规则在 id 映射表（键名就是 id ）和 标签映射表中。因此剩余的工作仅需找出通过键名提取出的规则中那些真正匹配的。

例如：如果 div 的规则如下
```css
table div {margin:5px}
```

它仍然从标签名映射表中提取出来，因为键名是选择器的最右边，但它不匹配我们的 div 元素，它没有table 的祖先。

Webkit 和 FIrefox 都做这种操作。

<a name="31a1d2db"></a>
##### 按正确的层叠顺序应用规则（Applying the rules in the correct cascade order）
样式对象的属性对应于每个可视属性（所有css 属性，但更通用）。如果没有通过任何匹配的规则定义——那么一些属性能从其父级元素的样式对象中继承。其他属性使用默认值。

当不只一个定义时，问题就开始了——接下来是解决这个问题的层叠顺序。

- 样式表层叠顺序（Style sheet cascade order）

一个样式属性的声明允许出现在多个样式表中，以及在一个样式表中的多次出现。这意味着应用规则的顺序非常重要。这就被称为“层叠”顺序。根据 CSS2 规范，层叠顺序就是（从低到高）：

  1. 浏览器声明
  1. 用户普通声明
  1. 作者普通声明
  1. 作者重要声明
  1. 用户重要声明

浏览器声明重要性最低，如果声明被标记为 important，用户才能覆盖作者的。<br />具有相同顺序的声明将按[规范](https://www.yuque.com/longtengsong/frontend/how_browsers_work/edit)进行排序，然后按指定顺序排序。HTML 可视化属性被转换为匹配的 CSS 声明。它们按视为低优先级的作者规则。<br />

- Specifity

选择器规范被定义在 [CSS2 规范](https://www.w3.org/TR/CSS2/cascade.html#specificity)中，如下：

  - 如果声明来自 'style' 属性，而不是带有选择器的规则，则计数 1，否则计数 0 （=a）
  - 计算选择器中 ID 属性的数量（=b）
  - 计算选择器中其他属性和伪类的数量（=c）
  - 计算选择器中元素和伪元素的数量（=d）

把四个数字 a-b-c-d 连接起来（在基数很大的数制中），得到这个特性。<br />数制需要你使用类别中最大的数字来定义，例如：<br />如果 a = 14 你能使用 16 进制数。在不大可能的情况下，当 a = 17 时，你需要 17 进制数。后面这种情形可能发生，选择器像这样：html body div div p ...(在你的选择器总有17个标签。不是很可能)。<br />一些示例：
```css
*             {}  /* a=0 b=0 c=0 d=0 -> specificity = 0,0,0,0 */
 li            {}  /* a=0 b=0 c=0 d=1 -> specificity = 0,0,0,1 */
 li:first-line {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
 ul li         {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
 ul ol+li      {}  /* a=0 b=0 c=0 d=3 -> specificity = 0,0,0,3 */
 h1 + *[rel=up]{}  /* a=0 b=0 c=1 d=1 -> specificity = 0,0,1,1 */
 ul ol li.red  {}  /* a=0 b=0 c=1 d=3 -> specificity = 0,0,1,3 */
 li.red.level  {}  /* a=0 b=0 c=2 d=1 -> specificity = 0,0,2,1 */
 #x34y         {}  /* a=0 b=1 c=0 d=0 -> specificity = 0,1,0,0 */
 style=""          /* a=1 b=0 c=0 d=0 -> specificity = 1,0,0,0 */
```


- 规则排序 - Sorting the rules

匹配到规则以后，则根据层叠规则进行排序。Webkit 对小列表使用冒泡排序，大列表使用合并排序。Webkit 通过重写规则的 “>” 运算符实现排序:
```cpp
static bool operator >(CSSRuleData& r1, CSSRuleData& r2)
{
    int spec1 = r1.selector()->specificity();
    int spec2 = r2.selector()->specificity();
    return (spec1 == spec2) : r1.position() > r2.position() : spec1 > spec2; 
}
```


<a name="dd5e0339"></a>
#### 渐进过程（Gradual process）
Webkit 使用标记是否已加载所有顶级样式表 (包括 “导入”) 的标志。当合成渲染树时，如果样式没有完全加载——持有的位置被使用并且标记在文档中，一旦样式表加载完成，它们将重新计算。

<a name="5baaa446"></a>
### 布局（Layout）
当渲染器被创建并添加到树结构中时，它还没有位置和大小。计算这些值被称为 layout 或者 reflow。

HTML 使用基于流的布局模型，这意味着大多数时间都可以在一个通道中计算几何图形。 “在流中” 后面的元素通常不会影响"在流中"之前的元素的几何图形，所以布局能从左至右，从上至下通过文档。有些例外——例如：HTML 表格可能需要多个通道。

坐标系统和根部框架有关系，使用顶部坐标和左边坐标。

布局是一个递归过程。它从根部的渲染器开始，与html 文档的元素相对应。布局继续递归，经过一些或者全部层叠的框架，为每一个需要的渲染器计算几何信息。

根渲染器的位置是0,0，尺寸是视窗——浏览器窗口的可视部分。

所有渲染器都有一个"layout" 或者 "reflow" 方法，每个渲染器调用其需要布局的子级的布局方法。

<a name="8786ab91"></a>
#### 脏位系统（Dirty bit system）
为了使每一个小的变化，不做一次完整的布局，浏览器使用了“脏位”系统。改变或者新增的渲染器把自己和子级标记为“dirty”——需要布局。

有两个标记：“dirty” 和 “children are dirty”。Children are dirty 意味着其渲染器本身可能没有改变，但至少有一个子级需要布局。

<a name="555f1346"></a>
#### 全局和增量布局（Global and incremental layout）
布局能被整个渲染树触发——这就是“全局”布局。这将导致一些结果：

1. 全局样式变化影响所有渲染器，如字体大小变化。
1. 屏幕将被重新计算

布局也可能是增量的，仅修改过的渲染器被布局（这可能会导致一些损坏，这将需要额外的布局）。

当渲染器被修改时，增量布局被触发（异步）。例如，新的渲染器被添加到渲染树后，额外的内容来自网络，并被添加到DOM 树。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1552694567559-e81fef8f-ae06-430b-9c06-4b53de36eeb5.png#align=left&display=inline&height=341&name=image.png&originHeight=341&originWidth=326&size=7736&status=done&width=326)<br />**Figure 20：增量布局——仅dirty 渲染器和他们的子级被布局**

<a name="8bf4bb5f"></a>
#### 异步和同步布局（Asynchronous and Synchronous layout）
增量布局是异步的，Firefox 用放置增量布局的队列“reflow commands”和一个调度器来触发这些命令的批处理执行。Webkit 也有一个计时器来执行一个增量布局——遍历树，“dirty” 渲染器被布局。

脚本要求样式信息，如“offsightHeight”能触发同步增量布局。

全局布局通常被同步的触发。

有时布局会在初始布局之后作为一个回调被触发，因为一些属性，如滚动位置改变。

<a name="Optimizations"></a>
#### Optimizations
当布局被“resize” 或者渲染器位置（不是大小）的改变激活，渲染的大小来自缓存而不是重新计算。<br />在一些情况下——仅子树被修改，布局是不会从跟节点开始。在本地改变且不影响周围的情况下，才可能发生——例如文本插入文本域（每个按键都将触发一个从跟节点开始的布局）。

<a name="63964773"></a>
#### 布局流程（The layout process）
布局通常包含以下几个部分：

1. 父级渲染器确定自己的宽度。
1. 父级遍历自己并：
  1. 确定子级渲染器的位置（设置它的x 和 y）
  1. 如果需要则调用子级的布局函数（它们是脏的或者我们在全局布局中，又或者一些其他原因）——这是计算子级的高度。
3. 父级使用子级累积的高度和 margin 和 padding 的高度来设置它自己的高度——这会用在父级渲染器的父级。
3. 设置它的脏位为 false。

Firefox 使用“状态”对象（nsHTMLReflowState）作为参数来布局（术语：“reflow”）。除其他外，状态包括了父级的宽度。<br />FIrefox布局的输出是一个“metrics”对象（nsHTMLReflowMetrics）。它会包含渲染器计算的高度。

<a name="1c5caf48"></a>
#### 宽度计算（Width calculation）
渲染器宽度的计算使用容器块的宽度，渲染器样式“width”属性，留白和边框。

例如：下面 div 的宽度计算：
```html
<div style="width: 30%" />
```

Webkit 将按下列顺序进行计算（RenderBox 类的方法 calcWidth）：

- 容器宽度是容器可用宽度和 0 的最大值。在这种情况下，可用宽度就是内容宽度，计算为：

clientWidth() - paddingLeft() - paddingRight()<br />clientWidth 和 clientHeight 相当于一个内部对象，包括了 border 和 scrollbar。

- 元素宽度是 “width” 样式属性。通过计算容器宽度的百分比，它会被计算成一个绝对值。
- 现在加上水平方向的 borders 和 paddings

到目前为止，这是 “优先宽度” 的计算。现在最小和最大宽度将被计算。

如果优先宽度高于最大宽度——最大宽度被使用。如果小于最小宽度（最小不可破坏单元），那么最小宽度将被使用。

如果需要布局，但宽度不会更改，则缓存值。

<a name="dcd1f22f"></a>
#### Line Breaking
当正在布局中的渲染器需要被断开时，它会停止和传播到需要被打断的父级。父级将创建一个额外的渲染器并调用其布局函数。

<a name="6e9ccdb0"></a>
### 绘制（Painting）
在绘制阶段，递归渲染树并调用“paint”方法用来在屏幕上显示他们的内容。绘制使用UI基础设施组件。更多详情查看UI相关的章节。

<a name="3339ab3e"></a>
#### 全局和增量（Global and Incremental）
就像布局，绘制也可以是全局性——绘制整个树——或者增量。在增量绘制中，一些渲染器以不影响整个树的方式更改。改变的渲染器使它在屏幕上的长方形区域无效。这导致 OS 察觉到它是一个“dirty region”并产生一个“paint”事件。OS 聪明地把几个 region 合并为一个。在 Chrome 中它更复杂，因为渲染器在一个不同于主进程的进程中。Chrome 在一定程度上模拟 OS 的行为。侦听这些事件并将消息委托给跟节点的渲染器。遍历这个树直到到达相关的渲染器。它将绘制它自己（通常是它的子级）。

<a name="127b6bc1"></a>
#### The painting order
CSS2 定义绘制流程的顺序——[http://www.w3.org/TR/CSS21/zindex.html](http://www.w3.org/TR/CSS21/zindex.html)。真实的顺序在元素堆叠在上下文栈中。该顺序影响绘制，从栈的后面向前面进行绘制。一个block 渲染器的栈顺序是：

1. background color
1. background image
1. border
1. children
1. outline

<a name="20c9ff13"></a>
#### Firefox display list
Firefox 遍历渲染树并为绘制长方形建立展现列表。它包括渲染器相关的长方形，正确的绘画顺序（渲染器的背景，然后边框等）。

那种方式，渲染树仅需要为重绘遍历一次，代替了几次——绘制所有背景，然后所有图片，然后所有边框等。FIrefox 通过不增加隐藏元素优化了流程，就像元素完整在其他不透明元素的下方。

<a name="e284a7db"></a>
#### Webkit 矩形存储（Webkit rectangle storage）
重绘之前，Webkit 保存了老的长方形作为位图。它之后仅需绘制新的或老的交叉的部分。

<a name="02423c12"></a>
### 动态改变（Dynamic changes）
浏览器尝试在响应更改时执行尽可能少的操作。因此元素颜色的变化仅会引起元素的重绘。元素位置的改变会引起元素、它的子节点和可能的兄弟节点的布局和重绘。新增一个DOM节点会导致节点的布局和重绘。重要的改变，例如“html”元素的字体大小改变，将导致缓存无效，布局和重绘整个树。

<a name="9605f8cd"></a>
### 渲染引擎的线程（The rendering engine's threads）
渲染引擎是单线程。几乎所有事情都发现在单线程中，网络操作除外。在 FIrefox 和 Safari 中，这是浏览器的主线程。在 Chrome 中，它是标签进程的主线程。

网络操作可以由多个并行线程执行，并行连接的数量有限（通常2-6个连接，FIrefox 3个，例如：使用6个）。

<a name="bf580377"></a>
#### 事件循环（Event loop）
浏览器主线程是一个事件循环。它是一个无限循环，使进程保持活力。它等待事件（例如布局和绘制事件）并处理它们。这是 FIrefox 事件循环的主要代码：
```cpp
while (!mExiting)
    NS_ProcessNextEvent(thread);
```

<a name="5540ebe2"></a>
### CSS2视觉模型（CSS2 visual model）
<a name="7c4f91be"></a>
#### The canvas
根据 CSS2 [规范](https://www.w3.org/TR/CSS21/intro.html#processing-model)，术语 canvas 描述“呈现格式结构的空间。”——浏览器绘制内容的地方。

canvas 的空间每个维度都是无限的，但浏览器根据视窗的尺寸选择初始宽度。

根据 [http://www.w3.org/TR/CSS2/zindex.html](http://www.w3.org/TR/CSS2/zindex.html)，如果 canvas 包含在另一个里，canvas 是透明的 ，如果不是，则给定一个浏览器定义的颜色。

<a name="2c6f9512"></a>
#### CSS 盒模型（CSS Box model）
[CSS box model](http://www.w3.org/TR/CSS2/box.html) 定义了为文档树中的元素生成的矩形盒子并根据视觉格式模型布局出来。<br />每个盒子都有一个内容区域（例如：文本、图片等等）和周期可选的 padding、border、和 margin 区域。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1553051247330-c63ab743-74da-461d-a869-1fbb0e372c14.png#align=left&display=inline&height=348&name=image.png&originHeight=348&originWidth=509&size=62400&status=done&width=509)<br />**Figure 14：css2 box model**<br />**<br />每个节点生成 0 - n 个这样的盒子。<br />所有元素都有一个 “display” 属性用来定义它们将要创建的盒子类型。例如：
```
block  - generates a block box.
inline - generates one or more inline boxes.
none - no box is generated.
```

默认值是 inline，但浏览器样式表设置了其他默认值。例如：“div” 元素的默认的 display 是 block。<br />你可以在 [http://www.w3.org/TR/CSS2/sample.html](http://www.w3.org/TR/CSS2/sample.html) 找到一个默认样式表。

<a name="a0dc7b43"></a>
#### 定位方案（Positioning scheme）
有三种方案：

1. 正常——对象根据其在文档中的位置进行定位——这意味着它在渲染树中的位置就像它在 dom 树中的位置一样，并根据其盒子的类型和尺寸进行布局。
1. 浮动——对象先像正常流一样布局，然后尽可能移动到最左侧或最右侧。
1. 绝对——对象在渲染树中的位置与其在dom树中的位置不一样

该定位方案通过“position”属性和“float”属性设置。

- static 和 relative 导致正常流
- absolute 和 fixed 导致 绝对定位

在 static 定位中，不定义位置并使用默认定位。其他方案中，作者定义position ——top、bottom、left、right。

盒子的布局方式是由以下信息决定的：

- 盒子类型
- 盒子尺寸
- 定位方案
- 外部信息——例如图片大小和屏幕大小

<a name="a8cfbf39"></a>
#### Box types
块级盒子：形状是一个块——在浏览器窗口上有它们自己的矩形。<br />
![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1553063265266-5e18f13b-6f7b-4c7c-a54a-b43f6f1f6020.png#align=left&display=inline&height=127&name=image.png&originHeight=127&originWidth=150&size=4624&status=done&width=150)<br />**Figure 15：块级盒子**<br />**<br />内联盒子：没有自己的块，但被包裹在块中。<br />
![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1553063359794-16bbb0db-b14f-4569-8ef0-837f9ea877a6.png#align=left&display=inline&height=233&name=image.png&originHeight=233&originWidth=300&size=14095&status=done&width=300)<br />**Figure 15：内联盒子**

块在垂直方向一个接一个被格式化。行级在水平方向格式化。<br />
![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1553063513712-06fe3242-3610-4e6a-83d7-9d4161fa5d8c.png#align=left&display=inline&height=324&name=image.png&originHeight=324&originWidth=350&size=22725&status=done&width=350)<br />**Figure 16：块和行格式化**

行级盒子放置在行内或行级盒子里。这些行至少和最高的盒子一样高，但是可以更高，当盒子以“baseline”进行对齐时，意思是元素的底部部分与其他盒子的底部对齐。这种情况容器宽度不够，这些行内盒子将放置成多行。这通常在段落中发生。

![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1553064101870-a36481ff-4af9-49c3-8b5f-55cad404eb89.png#align=left&display=inline&height=277&name=image.png&originHeight=277&originWidth=400&size=34538&status=done&width=400)<br />**Figure 17：行**

<a name="ef4f8692"></a>
#### 定位（Positioning）
<a name="ea733563"></a>
##### 相对（Relative）
相对定位——像通常一样定位，然后按要求的位置移动

![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1553064694798-54ab7b71-7599-427a-8a39-b009d21de76a.png#align=left&display=inline&height=261&name=image.png&originHeight=261&originWidth=500&size=28221&status=done&width=500)<br />**Figure 18:相对定位**<br />**
<a name="Floats"></a>
##### Floats
浮动的盒子被挪动的行的左边或右边。这个有趣的特性就是其他盒子围绕它流动，HTML：

```html
<p>
<img style="float:right" src="images/image.gif" width="100" height="100">Lorem ipsum dolor sit amet, consectetuer...
</p>
```
看起像这样<br />
![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1553065599543-e25ed63d-0631-4e05-8387-bc085c65cd43.png#align=left&display=inline&height=203&name=image.png&originHeight=203&originWidth=444&size=10403&status=done&width=444)<br />**Figure 19:Float**<br />**
<a name="f7dc1be2"></a>
##### 绝对和固定（Absolute and fixed）
这种布局被定义为完全不理会正常流。元素不参加正常流。尺寸是相对于容器的。固定定位——容器就是视窗。<br />
![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1553066037531-82965a95-f48b-40a4-8774-5e88224cec4a.png#align=left&display=inline&height=343&name=image.png&originHeight=343&originWidth=500&size=29304&status=done&width=500)<br />**Figure 20:Fixed positioning**
> 注意事项——固定盒子不会移动，即使文档被滚动。


<a name="69b75e00"></a>
#### 分层表示（Layered representation）
这是通过CSS属性 z-index 来定义的。它相对于盒子的第三个维度，它的位置属于“z 轴”。

盒子被分割成栈（叫作上下文栈）。这些盒子被分割为堆栈 (称为堆栈上下文)。在每个堆栈中，后面的元素将首先绘制，顶部的前向元素将更接近用户。如果重叠，将隐藏前元素。

堆栈根据 z-index 属性进行排序。带有 z-index 属性的盒子来自一个本地栈。视窗是独立于栈之外的。<br />例如：

```html
<style type="text/css">
  div { 
    position: absolute; 
    left: 2in; 
    top: 2in; 
  }
</style>

<p>
	<div style="z-index: 3;background-color:red; width: 1in; height: 1in; "></div>
	<div style="z-index: 1;background-color:green;width: 2in; height: 2in;"></div>
</p>
```
展示结构如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/189476/1553068582490-7c9d9293-77c8-4d79-9f31-badb8897abc4.png#align=left&display=inline&height=227&name=image.png&originHeight=227&originWidth=254&size=1908&status=done&width=254)<br />**Figure 20:固定定位**<br />**<br />虽然绿色的 div 是在红色的 div 之前，并且在正常流之前已经被绘制，但 z-index 属性更高，因此，它在根节点盒子持有的堆栈中更靠前。

<a name="Resources"></a>
### Resources

1. Browser architecture
  1. Grosskurth, Alan. A Reference Architecture for Web Browsers. http://grosskurth.ca/papers/browser-refarch.pdf.
2. Parsing
  1. Aho, Sethi, Ullman, Compilers: Principles, Techniques, and Tools (aka the "Dragon book"), Addison-Wesley, 1986
  1. Rick Jelliffe. The Bold and the Beautiful: two new drafts for HTML 5. http://broadcast.oreilly.com/2009/05/the-bold-and-the-beautiful-two.html.
3. Firefox
  1. L. David Baron, Faster HTML and CSS: Layout Engine Internals for Web Developers. http://dbaron.org/talks/2008-11-12-faster-html-and-css/slide-6.xhtml.
  1. L. David Baron, Faster HTML and CSS: Layout Engine Internals for Web Developers(Google tech talk video). http://www.youtube.com/watch?v=a2_6bGNZ7bA.
  1. L. David Baron, Mozilla's Layout Engine. http://www.mozilla.org/newlayout/doc/layout-2006-07-12/slide-6.xhtml.
  1. L. David Baron, Mozilla Style System Documentation. http://www.mozilla.org/newlayout/doc/style-system.html.
  1. Chris Waterson, Notes on HTML Reflow. http://www.mozilla.org/newlayout/doc/reflow.html.Chris Waterson, Gecko Overview. http://www.mozilla.org/newlayout/doc/gecko-overview.htm.
  1. Alexander Larsson, The life of an HTML HTTP request. https://developer.mozilla.org/en/The_life_of_an_HTML_HTTP_request.
4. Webkit
  1. David Hyatt, Implementing CSS(part 1). http://weblogs.mozillazine.org/hyatt/archives/cat_safari.html.
  1. David Hyatt, An Overview of WebCore. http://weblogs.mozillazine.org/hyatt/WebCore/chapter2.html.
  1. David Hyatt, WebCore Rendering. http://webkit.org/blog/114/.
  1. David Hyatt, The FOUC Problem. http://webkit.org/blog/66/the-fouc-problem/.
5. W3C Specifications
  1. HTML 4.01 Specification. http://www.w3.org/TR/html4/.
  1. HTML5 Specification. http://dev.w3.org/html5/spec/Overview.html.
  1. Cascading Style Sheets Level 2 Revision 1 (CSS 2.1) Specification. http://www.w3.org/TR/CSS2/.
6. Browsers build instructions
  1. Firefox. https://developer.mozilla.org/en/Build_Documentation
  1. Webkit. http://webkit.org/building/build.html
