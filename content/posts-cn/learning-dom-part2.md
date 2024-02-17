---
title: "DOM API详解（二）"
date: 2016-01-26 15:58:34
tags: 
- DOM
categories: 
- DOM
---

###四、API详解-DOM CORE

#### DOM标准的层级

DOM标准实际由很多的子模块组成，包括Core module，XML module，Events module，User interface Events module，Mouse Events module，Text Events module，Keyboard Events module，Mutation Events module，Mutation name Events module，HTML Events module，Load and Save module，Asynchronous load module，Validation module，和XPath module。

<!--more-->

具体的层级结构如下图所示：
![dom-architecture](https://www.w3.org/TR/2004/REC-DOM-Level-3-Core-20040407/images/dom-architecture.png)
*image credit: w3.org*

#### DOM Core-导语

在我们把目光转向DOM Core模块之前，我想先讲讲如何阅读DOM的W3C Specification。首先我们可以阅读官方的导语-[What is the Document Object Model?](https://www.w3.org/TR/2004/REC-DOM-Level-3-Core-20040407/introduction.html)。

这一节中介绍了DOM的相关概念。比如关于DOM的结构，我们通常认为是一颗树，但实际上用森林来形容更为贴切，因为虽然以`Document`为根节点的文档树最多只能有一颗，但我们还有一颗可选的`doctype`节点树，以及`comments`节点等等。

更重要的是，导语中还介绍了接下来文档中的一些惯例。比如因为DOM是独立于编程语言的，所以在文档中使用了IDL（Interactive Data Language）来描述接口。然后我们需要注意的一点就是，DOM可以用于HTML和XML，因此在前端同学看来，DOM中会有一些不太熟悉的概念，比如`Namespace`和`DOM URIs`，这些正是为了兼容XML而加入到DOM中的特性。不用过分在意。

下面是一个简化的IDL示例：

```
interface Element : Node {
  readonly attribute DOMString       tagName;
  void               removeAttribute(in DOMString name)
                                        raises(DOMException);
  Attr               getAttributeNode(in DOMString name);
  NodeList           getElementsByTagName(in DOMString name);
};
```
*IDL中主要描述了一个接口的继承关系，以及接口的方法和属性，及相应的参数和返回值*

下面我们首先来看DOM Core模块。从“Core”这个名词就可以看出这个模块在整个DOM标准中处于中心地位。DOM Core模块主要定义了DOM最关键的一组接口和对象。其中最顶层的一个接口就是`Node`。直接继承`Node`的接口有`Element`、`DocumentType`、
`Attr`、`ProcessingInstruction`、`Comment`、
`Text`、`CDATASection `和`Notation `。

#### Node接口

`Node`接口是整个DOM中最顶层的接口，DOM中的每一个节点都是一个`Node`。从`Node`接口分化出其他的更具体的接口。

`Node`接口的属性：

| **属性名** | **返回值** | **备注** |
| ---------------------------------------- | :--------------------------------------- |:--------------------------------------- |
|nodeName | DOMString | 只读 |
|nodeValue| DOMString |只读|
|nodeType|unsigned short |只读|
|parentNode|Node  |只读|
|childNodes| NodeList|只读|
|firstChild|Node  |只读|
|lastChild| Node |只读|
|previousSibling| Node|只读|
|nextSibling| Node |只读|
|attributes|NamedNodeMap|只读|
|ownerDocument| Document |只读|

值得注意的是，虽然这些属性都定义在Node接口上，但是不是所有的底层接口实现都有对应的值。如`Text`和`Comment`节点都没有子元素，因此它们的`childNodes`为`null`。同理，`Element`接口的`nodeValue`值就为`null`。

`nodeType`属性的值被定义为一组常量：

```
// NodeType
  const unsigned short      ELEMENT_NODE                   = 1;
  const unsigned short      ATTRIBUTE_NODE                 = 2;
  const unsigned short      TEXT_NODE                      = 3;
  const unsigned short      CDATA_SECTION_NODE             = 4;
  const unsigned short      ENTITY_REFERENCE_NODE          = 5;
  const unsigned short      ENTITY_NODE                    = 6;
  const unsigned short      PROCESSING_INSTRUCTION_NODE    = 7;
  const unsigned short      COMMENT_NODE                   = 8;
  const unsigned short      DOCUMENT_NODE                  = 9;
  const unsigned short      DOCUMENT_TYPE_NODE             = 10;
  const unsigned short      DOCUMENT_FRAGMENT_NODE         = 11;
  const unsigned short      NOTATION_NODE                  = 12;
```
这就是我们平时用来检测节点类型时使用的数值了。

下面从`Node`接口的方法中选几个主流的API进行简单的介绍，对于W3C中为标准但在WHATWG中被废弃的，这里不进行介绍（在主流的浏览器中也没有相应的实现）。对于一些在继承Node的接口中有更具体实现的API，这里也不进行介绍（如`hasAttributes`在Element接口中有更具体的实现）。

+ `compareDocumentPosition`

  这是DOM3中引入的一个很有意思的API。这个API将两个Node的位置进行比较，然后返回`DocumentPosition`这组常量中的一种。这组常量是：
  
  ```
  // 节点被另一个节点包含。被包含的节点是父节点的后继
  DOCUMENT_POSITION_CONTAINED_BY
  // 节点包含另一个节点。父节点是子节点的前驱
    DOCUMENT_POSITION_CONTAINS
DOCUMENT_POSITION_DISCONNECTED
// 后继关系
DOCUMENT_POSITION_FOLLOWING
DOCUMENT_POSITION_IMPLEMENTATION_SPECIFIC
// 前驱关系
DOCUMENT_POSITION_PRECEDING

  ```

+ `appendChild`&&`removeChild`  

 这两个API是比较常用的，`appendChild`在DOM子元素列表的最后插入元素，如果这个元素已经存在DOM树中，就先移除这个元素再插入。`removeChild`则从DOM子元素列表中移除元素。
+ `insertBefore`  

 这个API有两个参数，一个是`newChild`，一个是`refChild`。`newChild`既是要插入的节点，而`refChild`则是一个已经在`Node`所在DOM树中的参考子节点。`newChild`会插入到`refChild`之前，如果`refChild`是`null`，那么`newChild`会插入到`Node`子节点列表的末尾。
 
 这个API的记忆很简单，命名是`insertBefore`，必然是有一个参考节点的，不然插到谁的前面去呢？
 
+ `isEqualNode`
 
 这个API的作用是判断两个DOM是否相等。这里的相等并不是指指向同一个对象，而是指属性和值方面是否相同。
 
 对于两个节点来说比较的算法如下：
 
 + 两个节点是否是相同的类型
 + `nodeName, localName, namespaceURI, prefix, nodeValue`这些属性是否相同
 + 节点的属性集合（在DOM中的数据类型是`NamedNodeMaps`）是否相同，这里集合中属性的顺序可以是随意的
 + 子节点集合（`NodeLists`类型）是否相同，这里会考虑节点的index，如果不一样则不相同。
 
#### Element接口

#### Document接口

#### Attr接口

#### Text，Comment和CharacterData接口

#### DOM中的基础数据类型
+ `DOMString`  
+ `DOMTimeStamp`
+ `DOMUserData`
+ `DOMObject`