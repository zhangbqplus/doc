# 浏览器DOM

DOM API 是最早被设计出来的一批 API，也是用途最广的 API，常常用 DOM 来泛指浏览器中所有的 API。不过这里我们要介绍的 DOM，指的就是狭义的**文档对象模型**。

#### DOM API 介绍

文档对象模型：用来描述文档。特指 HTML 文档（也用于 XML 文档，但我们这里不讨论）。同时它又是一个“对象模型”，这意味着它使用的是对象这样的概念来描述 HTML 文档。

HTML 文档是一个由标签嵌套而成的树形结构，因此，DOM 也是使用树形的对象模型来描述一个 HTML 文档。

DOM API 大致会包含 4 个部分：

- 节点：DOM 树形结构中的节点相关 API。
- 事件：触发和监听事件相关 API。
- Range：操作文字范围相关 API。
- 遍历：遍历 DOM 需要的 API。

本篇文章重点会为你介绍节点和遍历相关 API。DOM API 数量很多，我希望给你提供一个理解 DOM API 设计的思路，避免单靠机械的方式去死记硬背。

#### 节点

DOM 的树形结构所有的节点有统一的接口 Node，我们按照继承关系来节点的类型。

![Image text](https://github.com/zhangbqplus/doc/blob/master/img/qianduan/_7.jpg)

在这些节点中，除了 Document 和 DocumentFrangment，都有与之对应的 HTML 写法

```

Element: <tagname>...</tagname>
Text: text
Comment: <!-- comments -->
DocumentType: <!Doctype html>
ProcessingInstruction: <?a 1?>
```

我们在编写 HTML 代码并且运行后，就会在内存中得到这样一棵 DOM 树，HTML 的写法会被转化成对应的文档模型，而我们则可以通过 JavaScript 等语言**去访问这个文档模型**。

常用的节点：Document、Element、Text 

#### Node

Node 是 DOM 树继承关系的根节点，它定义了 DOM 节点在 DOM 树上的操作，首先，Node 提供了一组属性，来表示它在 DOM 树中的关系，它们是：

- parentNode	
- childNodes
- firstChild
- lastChild
- nextSibling
- previousSibling

上面这组属性为我们提供了前、后、父、子关系，有了这几个属性，我们可以很方便地根据**相对位置**获取元素。

Node 中也提供了操作 DOM 树的 API，主要有下面几种：

- appendChild
- insertBefore
- removeChild
- replaceChild

这几个 API 的设计只有 before，没有 after，而 jQuery 等框架都对其做了补充。

实际上：appendChild 和 insertBefore 的这个设计，是一个“最小原则”的设计，这两个 API 是满足插入任意位置的必要 API，而 insertAfter，则可以由这两个 API 实现出来。

通过上面Node 提供的 API 已经（基本）可以很方便地对树进行增、删、遍历等操作了。

Node 还提供了一些**高级 API**，

- compareDocumentPosition 	是一个用于比较两个节点中关系的函数。
- contains  检查一个节点是否包含另一个节点的函数。
- isEqualNode  检查两个节点是否完全相同。
- isSameNode  检查两个节点是否是同一个节点，实际上在 JavaScript 中可以用“===”。
- cloneNode  复制一个节点，如果传入参数 true，则会连同子元素做深拷贝。

DOM 标准规定了节点必须从文档的 create 方法创建出来，不能够使用原生的 JavaScript 的 new 运算。于是 document 对象有这些方法。

- createElement
- createTextNode
- createCDATASection
- createComment
- createProcessingInstruction
- createDocumentFragment
- createDocumentType

#### Element 与 Attribute

当我们比较关注的是元素的时候，Element就是表示元素，它是 Node 的子类。

元素对应了 HTML 中的**标签**，它**既有子节点**，**又有属性**。所以 Element 子类中，有一系列操作属性的方法。

我们把元素的 Attribute 当作字符串来看待（返回字符串，即取得属性值），这样就有以下的 API：

- getAttribute
- setAttribute
- removeAttribute
- hasAttribute

还可以把 Attribute 当作节点（返回属性节点，是一个对象）：

- getAttributeNode
- setAttributeNode

另外还有attributes 对象：document.body.attributes.class = “a” 等效于 document.body.setAttribute(“class”, “a”)

#### 查找元素

document 节点提供了查找元素的能力。比如有下面的几种

- querySelector
- querySelectorAll
- getElementById
- getElementsByName
- getElementsByTagName
- getElementsByClassName

getElementById、getElementsByName、getElementsByTagName、getElementsByClassName，这几个 API 的性能高于 querySelector。

getElementsByName、getElementsByTagName、getElementsByClassName 获取的集合并非数组，而是一个能够动态更新的集合。

#### 遍历

通过 Node 的相关属性，我们可以用 JavaScript 遍历整个树。

DOM API 中还提供了 NodeIterator 和 TreeWalker 来遍历树。比起直接用属性来遍历，NodeIterator 和 TreeWalker 提供了**过滤功能**，还可以把*属性节点也包含在遍历之内*。

NodeIterator 的基本用法示例：

```
var iterator = document.createNodeIterator(document.body, NodeFilter.SHOW_TEXT | NodeFilter.SHOW_COMMENT, null, false);
var node;
while(node = iterator.nextNode())
{
    console.log(node);
}
```

TreeWalker 的用法示例：

```
var walker = document.createTreeWalker(document.body, NodeFilter.SHOW_ELEMENT, null, false)
var node;
while(node = walker.nextNode())
{
    if(node.tagName === "p")
        node.nextSibling();
    console.log(node);
}
```

建议需要遍历 DOM 的时候，直接使用递归和 Node 的属性， 不要使用TreeWalker 和 NodeIterator 这两个 API。

下面就来见到那介绍一下这两个API：

 NodeIterator： NodeIterator循环并没有类似“hasNext”这样的方法，而是直接以 nextNode 返回 null 来标志结束， NodeIterator的第二个参数是掩码，这两个设计都是传统 C 语言里比较常见的用法。通常掩码型参数，我们都是用按位或运算来叠加。而针对这种返回 null 表示结束的迭代器，我使用了在 while 循环条件中赋值，来保证循环次数和调用 next 次数严格一致（但这样写可能违反了某些编码规范）。

TreeWalker： 比起 NodeIterator，TreeWalker 多了在 DOM 树上自由移动当前节点的能力，一般来说，这种 API 用于“跳过”某些节点，或者重复遍历某些节点。

#### Range

Range API 是一个比较专业的领域，比较适用于富文本编辑类的业务。

Range API 表示一个 HTML 上的范围，这个范围是以文字为最小单位的，所以 Range 不一定包含完整的节点，它可能是 Text 节点中的一段，也可以是头尾两个 Text 的一部分加上中间的元素。

通过 Range API 可以比节点 API 更精确地操作 DOM 树，凡是 节点 API 能做到的，Range API 都可以做到，而且可以做到更高性能，但是 Range API 使用起来比较麻烦，所以在实际项目中，并不常用，只有做底层框架和富文本编辑对它有强需求。

创建 Range 一般是通过设置它的起止来实现

```
var range = new Range(),
    firstText = p.childNodes[1],
    secondText = em.firstChild
range.setStart(firstText, 9) // do not forget the leading space
range.setEnd(secondText, 4)
```

通过 Range 也可以从用户选中区域创建，这样的 Range 用于处理用户选中区域

```
var fragment = range.extractContents()
range.insertNode(document.createTextNode("aaaa"))
```

下面这个例子展示了如何使用 range 来取出元素和在特定位置添加新元素

```
var range = new Range(),
    firstText = p.childNodes[1],
    secondText = em.firstChild
range.setStart(firstText, 9) // do not forget the leading space
range.setEnd(secondText, 4)

var fragment = range.extractContents()
range.insertNode(document.createTextNode("aaaa"))
```

#### 命名空间

在 HTML 场景中，需要考虑命名空间的场景不多。最主要的场景是 SVG。

创建元素和属性相关的 API 都有带命名空间的版本：

- documen
  - tcreateElementNS
  - createAttributeNS
- Element
  - getAttributeNS
  - setAttributeNS
  - getAttributeNodeNS
  - setAttributeNodeNS
  - removeAttributeNS
  - hasAttributeNS
  - attributes.setNamedItemNS
  - attributes.getNamedItemNS
  - attributes.removeNamedItemNS

若要创建 Document 或者 Doctype，也必须要考虑命名空间问题。DOM 要求从 document.implementation 来创建。

- document.implementation.createDocument
- document.implementation.createDocumentType

除此之外，还提供了一个快捷方式

- document.implementation.createHTMLDocument