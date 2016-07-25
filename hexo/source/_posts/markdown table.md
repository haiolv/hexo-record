---
title: MarkDown Leaner(表格以及锚).md
date: 2016-07-25 17:03:39
tags: [MarkDown,文本编辑]
categories: [文本编辑,MarkDown] # 配置categories
---
## MarkDown 常用技巧<a id="begin"></a>

[markdown](http://markdownpad.com/ "http://markdownpad.com/")是一种轻量级的「标记语言」，其语法简单有效。常用的标记符号也不超过十个，这种相对于更为复杂的 HTML 标记语言来说，Markdown 可谓是十分轻量的，学习成本也不需要太多，且一旦熟悉这种语法规则，会有一劳永逸的效果。

这里对于简单的语法就不做阐述了，由各位客官自行揣摩，本篇小文主要是正对两个方面做一记录和阐述：

* [MarkDown 表格语法](#markdown_table)；
* [MarkDown 页内跳转](#markdown_page_link)；

这里是测试业内跳转的部分：

* 测试一<a id="test1"></a>
* <span id="test2">测试二</span>
* <span name="test3">测试三</span>

### MarkDown 简单语法入门

参考常用的Markdown语法：

- [Markdown 语法说明 (简体中文版)](http://wowubuntu.com/markdown/#list "http://wowubuntu.com/markdown/#list")
- [创始人 John Gruber 的 Markdown 语法说明](http://daringfireball.net/projects/markdown/syntax "http://daringfireball.net/projects/markdown/syntax")
- [Markdown常见语法](https://learn.getgrav.org/content/markdown "https://learn.getgrav.org/content/markdown")
- [为什么我们要学习Markdown的三个理由](http://news.cnblogs.com/n/139649/ "http://news.cnblogs.com/n/139649/")

工具集合

- [马克飞象，专为印象笔记打造的Markdown编辑器，非常推荐](https://maxiang.io/ "https://maxiang.io/")
- [cloudapp ： Easily share anything, anywhere](https://www.getcloudapp.com/ "https://www.getcloudapp.com/")
- [Markdown 工具补完](http://www.appinn.com/markdown-tools/ "http://www.appinn.com/markdown-tools/")

简要说明，使用Markdown的有点：

* 使你你专注文字内容而不是排版样式，安心写作。
* 使你轻松的导出 HTML、PDF 和本身的 .md 文件，利于编辑以及共享。
* 使你轻松入门，学习成本较低，并且与各大知名网站兼容，如 Github等，以及各种blog格式，如博客园等，其可读、直观、学习成本低。


[markdown](http://markdownpad.com/ "http://markdownpad.com/")常用语法很简单，可以从上面说的入门中进行学习。
下面主要针对 表格和页内跳转进行整理。

### <span id="markdown_table">MarkDown 表格语法</span>

Markdown的表格语法如下：

	| Option | Description |
	| ------ | ----------- |
	| data   | path to data files to supply the data that will be passed into templates. |
	| engine | engine to be used for processing templates. Handlebars is the default. |
	| ext    | extension to be used for dest files. |

>Tables are created by adding pipes as dividers between each cell, and by adding a line of dashes (also separated by bars) beneath the header. Note that the pipes do not need to be vertically aligned.

其对应的Html如下：

	<table>
	  <tr>
	    <th>Option</th>
	    <th>Description</th>
	  </tr>
	  <tr>
	    <td>data</td>
	    <td>path to data files to supply the data that will be passed into templates.</td>
	  </tr>
	  <tr>
	    <td>engine</td>
	    <td>engine to be used for processing templates. Handlebars is the default.</td>
	  </tr>
	  <tr>
	    <td>ext</td>
	    <td>extension to be used for dest files.</td>
	  </tr>
	</table>

也就是说，上述两种方式都可以实现简单的表格。

如果，你足够细心，你可能会想到，表格单元中的内容对其方式，上述默认对齐方式是按照左对齐的。

但是如果你想让表格中的内容右对齐，或者居中显示呢？其实实现也很简单：

* 右对齐

	| Option | Description |
	| ------: | -----------: |
	| data   | path to data files to supply the data that will be passed into templates. |
	| engine | engine to be used for processing templates. Handlebars is the default. |
	| ext    | extension to be used for dest files. |

* 居中对齐

	| Option | Description |
	| :------: | :-----------: |
	| data   | path to data files to supply the data that will be passed into templates. |
	| engine | engine to be used for processing templates. Handlebars is the default. |
	| ext    | extension to be used for dest files. |

不知道有没有看到修改之处 

	"| :------: | :-----------: |"

核心就在于这里的修改之处,冒号的添加，如果右边有，就居右显示，左边有就局左显示，左右都有，就居中显示，默认为居左显示。

来个混合的，看看效果

	| Tables        | Are           | Cool  |
	| ------------- |:-------------:| -----:|
	| col 3 is      | right-aligned | $1600 |
	| col 2 is      | centered      |   $12 |
	| zebra stripes | are neat      |    $1 |


参考：[Markdown之表格的处理](http://www.ituring.com.cn/article/3452 "http://www.ituring.com.cn/article/3452")，然后得到转化后的html，复制粘贴到markdown就行。



当然有的时候，我们需要将需要在文章中插入一段表格，或者 将EXCEL表格插入到Markdown中，当然可以用截图，但是截图显然不合适，而且截图不好控制，比如表格过长的情况下。

针对将 将EXCEL表格插入到Markdown中，我们可以通过 [将EXCEL/Word表格转化成HTML格式的在线网站](http://pressbin.com/tools/excel_to_html_table/index.html "http://pressbin.com/tools/excel_to_html_table/index.html")

例如：经excel复制表格内容如下：

	编号	城市	国家
	1	北京	中国
	2	西安	中国
	3	纽约	美国
	4	巴黎	法国

生成html代码：

	<table>
	   <tr>
	      <td>编号</td>
	      <td>城市</td>
	      <td>国家</td>
	   </tr>
	   <tr>
	      <td>1</td>
	      <td>北京</td>
	      <td>中国</td>
	   </tr>
	   <tr>
	      <td>2</td>
	      <td>西安</td>
	      <td>中国</td>
	   </tr>
	   <tr>
	      <td>3</td>
	      <td>纽约</td>
	      <td>美国</td>
	   </tr>
	   <tr>
	      <td>4</td>
	      <td>巴黎</td>
	      <td>法国</td>
	   </tr>
	   <tr>
	      <td></td>
	   </tr>
	</table>

效果如下：

<table>
   <tr>
      <td>编号</td>
      <td>城市</td>
      <td>国家</td>
   </tr>
   <tr>
      <td>1</td>
      <td>北京</td>
      <td>中国</td>
   </tr>
   <tr>
      <td>2</td>
      <td>西安</td>
      <td>中国</td>
   </tr>
   <tr>
      <td>3</td>
      <td>纽约</td>
      <td>美国</td>
   </tr>
   <tr>
      <td>4</td>
      <td>巴黎</td>
      <td>法国</td>
   </tr>
</table>


参考： 

- [markdown做表格](http://www.wooaii.com/archives/3434.html "http://www.wooaii.com/archives/3434.html")
- [Excel/World表格格式转换成Html格式](http://pressbin.com/tools/excel_to_html_table/index.html "http://pressbin.com/tools/excel_to_html_table/index.html")


### <span id="markdown_page_link">MarkDown 页内跳转</span>

如果亲们写了一篇很长的文章，要是没有目录的话。阅读起来十分不便。 页内跳转 就能很好的解决这个问题。

业内跳转 这个概念在Html中就有，当然Markdown的语法也可以实现。

#### Html中的页内跳转

即HTML 锚（anchor)的实现，当使用命名锚（named anchors）时，我们可以创建直接跳至该命名锚（比如页面中某个小节）的链接，这样使用者就无需不停地滚动页面来寻找他们需要的信息了。

	<a name="label">锚（显示在页面上的文本）</a>

提示：锚的名称可以是任何你喜欢的名字。

提示：您可以使用 id 属性来替代 name 属性，命名锚同样有效。

例如：

首先，我们在 HTML 文档中对锚进行命名（创建一个书签）：

	<a name="tips">基本的注意事项 - 有用的提示</a>

然后，我们在同一个文档中创建指向该锚的链接：

	<a href="#tips">有用的提示</a>


>参考：[w3school:HTML 链接](http://www.w3school.com.cn/html/html_links.asp "http://www.w3school.com.cn/html/html_links.asp")

#### MarkDown的具体实现

arkDown的具体实现其实和Html的很类似，也是需要定义类似 锚 的属性，然后通过

	[需要显示的名称](#锚的属性)

实现跳转。

例如我们点击跳转到开始：

* [跳转到首页](#begin)
* [跳转到测试一](#test1)
* [跳转到测试二](#test2)
* [跳转到测试三](#test3)