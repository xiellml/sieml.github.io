---
title: JavaScript Basic Introduction
date: 2018-03-29 14:56:30
tags: script
---


#### JavaScript历史

在上个世纪的1995年，当时的网景公司正凭借其Navigator浏览器成为Web时代开启时最著名的第一代互联网公司。

由于网景公司希望能在静态HTML页面上添加一些动态效果，于是Brendan Eich这哥们在两周之内设计出了JavaScript语言。你没看错，这哥们只用了10天时间。

为什么起名叫JavaScript？原因是当时Java语言非常红火，所以网景公司希望借Java的名气来推广，但事实上JavaScript除了语法上有点像Java，其他部分基本上没啥关系

#### JavaScript的作用

- 操作网页元素： 静态和动态效果
- 传递数据：与后台交互数据
- 响应用户输入：处理各种输入事件

#### JavaScript的内容范围

归结起来就两个东西 **对象** 和 **事件**

#### JavaScript组成部分

- JavaScript核心语法

  > 变量，常量，运算符，控制语句，函数，对象

- BOM ：浏览器对象模型，其实就是JavaScript里面有操作浏览器的知识

- DOM：文档对象模型 ，HTML DOM 定义了所有 HTML 元素的*对象*和*属性*，以及访问它们的*方法*。 换言之，HTML DOM 是关于如何获取、修改、添加或删除 HTML 元素的标准。

注： DOM包含三层意思(HTML文档被浏览器解析后就是一棵DOM树)：

1. DOM是一个操作HTML的API接口
2. DOM是一个HTML结构模型
3. DOM代指该HTML模型中的对象

总结： 简单地说，BOM和DOM一样，只不过DOM操作的是HTML中的元素，BOM是浏览器的API、操作的是浏览器（即控制浏览器的行为）

### JavaScript引入方式

- 外部引用：把另外一个js文件引入到这个html文件中

```
<body>  
        引入外部js  
        <button id="button">实验</button>  
        <script src="demo.js">     
        </script>  
 </body>
```

注：外部js嵌入另一个外部js，document.write(““)

- 嵌入方式：可以写在head中，也可以写在body末尾

```
<body>  
       直接在HTML中嵌入
       <button id="button">实验</button>  
       <script>  
           button.onclick=function()  
           {  
               alert("引入方法一：直接嵌入script标签");  
           }  
       </script>  
 
</body>
```

- 事件监听处调用

```
<body>  
js的引入方式实验  
<button id="button" onclick="javascript:alert('js引入的方式3：事件定义')">实验</button>  
</body>
```

- 域名或者是重定向

```
<a href="javascript:"></a>
<form action="javascript:">
</form>
```

### 基本语法

**JS的注释：**

JS单行注释是用//，多行注释以 / *开始，以* / 结尾。

结束*/

**JS的输出：**

alert() 弹出对话框；

使用 **document.write()** 方法将内容写到 HTML 文档中；

**JS的变量：**

var是JS中的通用类型变量，可以随便存储其它类型的值，可以直接使用，不用定义。

var name;name 为变量名，可以有字母，数字 ，下划线组成，但不能用数字 开头，不区分 大小写

**数字类型：**

int :整数型

string;字符型

double;双精度

flout;单精度

boolen;布尔型

attr=array()数姐

类型的转换：

将其他类型转换成整数型：parseint()

将其他 类型转换成小数型：parsefloat()

判断是否是一个合法的数字类型：isNaN()

**运算符：**

数学运算符有：+, - ,* ,/,%,++,–

关系运算符有：== ！= => =<

逻辑运算符有：&& （并） ||（与） ！（非）

给定 x=6 以及 y=3，下表解释了逻辑运算符：

(x < 10 && y > 1) 为 true//两个条件同时满足为 true，有一个不满足为 false

(x==5 || y==5) 为 false//两个条件有一个满足为 true，都不满足为 false

!(x==y) 为 true//如果不满足为 true，满足为 false

运算符 = 用于赋值。

运算符 + 用于加值。

### 第三方js库

**jQuery** “Write Less, Do More“

- 消除浏览器差异：你不需要自己写冗长的代码来针对不同的浏览器来绑定事件，编写AJAX等代码；
- 简洁的操作DOM的方法：写`$('#test')`肯定比`document.getElementById('test')`来得简洁；
- 轻松实现动画、修改CSS等各种操作。

**underscore** 函数式编程
Object没有这些方法。此外，低版本的浏览器例如IE6～8也没有这些方法. 所以:
使用统一开源库的函数来实现map()、filter()等等操作

**node.js** 异步IO缺陷的应用
在2009年，Ryan正式推出了基于JavaScript语言和V8引擎的开源Web服务器项目，命名为Node.js

**React** 用于构建用户界面的 JavaScript 库


参考文章：

[JavaScript 快速入门](http://docs.cocos.com/creator/manual/zh/scripting/javascript-primer.html)