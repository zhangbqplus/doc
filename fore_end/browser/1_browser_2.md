# 浏览器的工作原理（解析代码和DOM树的构建）

#### 解析过程如图：

![Image text](https://github.com/zhangbqplus/doc/blob/master/img/qianduan/_5.jpg)

#### 解析代码

我们日常开发需要的 90% 的HTML“词”（指编译原理的术语 token，表示最小的有意义的单元），种类大约只有标签开始、属性、标签结束、注释、CDATA 节点几种。

HTML 支持 SGML 语法。如：<?  和 <% 。如果html发生报错也不会有任何提示。

1. ##### 词（token）是如何被拆分的

   标准标签语法

   ```
   <p class="a">text text text</p>
   ```

   第一个词（token) 是 <p

   拆分如下:

   ```
   <p "标签开始"的开始
   class = "a" 属性
   > "标签开始"的结束
   text text text 文本
   </p> 标签结束
   ```

   ![Image text](https://github.com/zhangbqplus/doc/blob/master/img/qianduan/_6.jpg)

逻辑思想：

1、代码从 HTTP 协议收到的字符流读取字符

2、在接受第一个字符之前是无法判断这是哪一个词（token）的，但是随着接受的字符越来越多，拼出其他的内容可能性就越来越少。

我们根据逻辑思想可以看出在我们每读入一个字符时，其实都要做一次决策，而且这些决定是跟“当前状态”有关的。在这样的条件下，浏览器工程师要想实现把字符流解析成词（token），最常见的方案就是使用状态机。

#### 状态机

HTML 词法状态机官方文档：https://html.spec.whatwg.org/multipage/parsing.html#tokenization

为了方便理解，我么用JavaScript来模拟一下（状态机）代码：

```
var data = function(c){
    if(c=="&") {
        return characterReferenceInData;
    }
    if(c=="<") {
        return tagOpen;
    }
    else if(c=="\0") {
        error();
        emitToken(c);
        return data;
    }
    else if(c==EOF) {
        emitToken(EOF);
        return data;
    }
    else {
        emitToken(c);
        return data;
    }
};
var tagOpenState = function tagOpenState(c){
    if(c=="/") {
        return endTagOpenState;
    }
    if(c.match(/[A-Z]/)) {
        token = new StartTagToken();
        token.name = c.toLowerCase();
        return tagNameState;
    }
    if(c.match(/[a-z]/)) {
        token = new StartTagToken();
        token.name = c;
        return tagNameState;
    }
    if(c=="?") {
        return bogusCommentState;
    }
    else {
        error();
        return dataState;
    }
};
//……
```

状态机的两个状态示例：data 为初始状态，tagOpenState 是接受了一个“ < ” 字符，来判断标签类型的状态。

每一个状态是一个函数，通过“if else”来区分下一个字符做状态迁移。即通过当前状态函数返回下一个状态函数。

状态迁移代码：

```
var state = data;
var char
while(char = getInput())
state = state(char);
```

这段代码的关键一句是“ state = state(char) ”，不论我们用何种方式来读取字符串流，我们都可以通过 state 来处理输入的字符流，这里用循环是一个示例，真实场景中，可能是来自 TCP 的输出流。

状态函数通过代码中的 emitToken 函数来输出解析好的 token（词），我们只需要覆盖 emitToken，即可指定对解析结果的处理方式。

词法分析器代码：

```
function HTMLLexicalParser(){

    //状态函数们……
    function data() {
        // ……
    }

    function tagOpen() {
        // ……
    }
    // ……
    var state = data;
    this.receiveInput = function(char) {
        state = state(char);
    }
}
```

至此就把字符流拆成了词（token）了。

#### 构建 DOM 树

接下来我们要把这些简单的词变成 DOM 树，这个过程我们是使用栈来实现的，任何语言几乎都有栈，我们还是用 JavaScript 来模拟一下，在JavaScript 中的栈只要用数组就好了。

```
function HTMLSyntaticalParser(){
    var stack = [new HTMLDocument];
    this.receiveInput = function(token) {
        //……
    }
    this.getOutput = function(){
        return stack[0];
    }
}
```

我们这样来设计 HTML 的语法分析器，receiveInput 负责接收词法部分产生的词（token），通常可以由 emitToken 来调用。在接收的同时，即开始构建 DOM 树，所以我们的主要构建 DOM 树的算法，就写在 receiveInput 当中。当接收完所有输入，栈顶就是最后的根节点，我们 DOM 树的产出，就是这个 stack 的第一项。

为了构建 DOM 树，我们需要一个 Node 类，接下来我们所有的节点都会是这个 Node 类的实例。

在完全符合标准的浏览器中，不一样的 HTML 节点对应了不同的 Node 的子类，我们为了简化，就不完整实现这个继承体系了。我们仅仅把 Node 分为 Element 和 Text（如果是基于类的 OOP 的话，我们还需要抽象工厂来创建对象），

```
function Element(){
    this.childNodes = [];
}
function Text(value){
    this.value = value || "";
}
```

前面我们的词（token）中，以下两个是需要成对匹配的：

- tag start
- tag end

根据一些编译原理中常见的技巧，我们使用的栈正是用于匹配开始和结束标签的方案。

对于 Text 节点，我们则需要把相邻的 Text 节点合并起来，我们的做法是当词（token）入栈时，检查栈顶是否是 Text 节点，如果是的话就合并 Text 节点。

同样我们来看看直观的解析过程：

```
<html maaa=a >
    <head>
        <title>cool</title>
    </head>
    <body>
        <img src="a" />
    </body>
</html>
```

通过这个栈，我们可以构建 DOM 树：

- 栈顶元素就是当前节点；
- 遇到属性，就添加到当前节点；
- 遇到文本节点，如果当前节点是文本节点，则跟文本节点合并，否则入栈成为当前节点的子节点；
- 遇到注释节点，作为当前节点的子节点；遇到 tag start 就入栈一个节点，当前节点就是这个节点的父节点；
- 遇到 tag end 就出栈一个节点（还可以检查是否匹配）。

当我们的源代码完全遵循 XHTML（这是一种比较严谨的 HTML 语法）时，这非常简单问题，然而 HTML 具有很强的容错能力，奥妙在于当 tag end 跟栈顶的 start tag 不匹配的时候如何处理。

W3C 规则都整理：http://www.w3.org/html/wg/drafts/html/master/syntax.html#tree-construction