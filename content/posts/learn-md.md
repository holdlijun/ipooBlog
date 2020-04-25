---
title: "Markdown 基础语法与进阶使用"
description : "Markdown 常用语法，进阶使用"
date: 2020-03-18
tags : ["Markdown"]
categories : [
    "Markdown",
]
type : ["posts","post"]
KeyName : "Markdown基础教程,Markdown语法,Markdown进阶使用"

---


<!-- MarkdownTOC -->

- [标题](#%E6%A0%87%E9%A2%98)
- [段落](#%E6%AE%B5%E8%90%BD)
- [换行符](#%E6%8D%A2%E8%A1%8C%E7%AC%A6)
- [文本](#%E6%96%87%E6%9C%AC)
- [插入图片](#%E6%8F%92%E5%85%A5%E5%9B%BE%E7%89%87)
- [引用区块](#%E5%BC%95%E7%94%A8%E5%8C%BA%E5%9D%97)
- [列表](#%E5%88%97%E8%A1%A8)
- [代码块](#%E4%BB%A3%E7%A0%81%E5%9D%97)
	- [代码块中语法高亮](#%E4%BB%A3%E7%A0%81%E5%9D%97%E4%B8%AD%E8%AF%AD%E6%B3%95%E9%AB%98%E4%BA%AE)
- [链接](#%E9%93%BE%E6%8E%A5)
- [网址和电子邮件地址](#%E7%BD%91%E5%9D%80%E5%92%8C%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6%E5%9C%B0%E5%9D%80)
- [转义字符](#%E8%BD%AC%E4%B9%89%E5%AD%97%E7%AC%A6)
- [Markdown使用表情符号](#markdown%E4%BD%BF%E7%94%A8%E8%A1%A8%E6%83%85%E7%AC%A6%E5%8F%B7)

<!-- /MarkdownTOC -->


#  标题

| Markdown语法 | HTML | 
| ------ | ------ | 
| # Heading level 1	 | 	一级
| ## Heading level 2	| 二级
| ### Heading level 3	| 	三级
| #### Heading level 4	 | 	四级
| ##### Heading level 5	 | 	五级
| ###### Heading level 6	 | 	六级

# 段落

- 创建段落，使用一行或多行分割。

- 不要缩进带有空格或制表符的段落。

# 换行符
- 一个段落后面加` <br>  ` ,后面要有两个空格
```txt
I really like using Markdown.<br>  
I think I'll use it to format all of my documents from now on.
```

# 文本

- 加粗文本
	请在单词或短语的前后添加`**`或`_`。要加粗一个单词的中部以强调，请在字母周围添加`**`，且各空格之间没有空格。
- 斜体
	要斜体显示文本，请在单词或短语前后添加`*`或`_`。要斜体突出单词的中间部分，请在字母周围添加`*`，中间不要带空格。  

# 插入图片
- 添加感叹号`!`，然后在括号中添加替代文本，并在括号中添加图像资源的路径或URL。您可以选择在括号中的URL之后添加标题。
```txt
![testImages](https://....com/images/ipoovx.jpg "ipoo") 
```
- 效果:
![testImages](http://oss.ipooli.com/images/ipoovx.jpg?x-oss-process=image/resize,w_300,h_300 "ipoo") 

# 引用区块
- 在段落前面添加一个 `>`
> Dorothy followed her through many of the beautiful rooms in her castle.

- 多个段落的引用，换行空白处加 `>`

```txt
	> Dorothy followed her through many of the beautiful rooms in her castle.
	>
	> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.
```

- 嵌套块的引用

在要嵌套的段落前面添加一个 	`<<`
```txt
> Dorothy followed her through many of the beautiful rooms in her castle.
>
>> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.
```

# 列表
- 有序列表
```txt
	1. First item
	2. Second item
	3. Third item
	4. Fourth item
```
效果:
1. First item
2. Second item
3. Third item
4. Fourth item
- 无序列表

要创建无序列表，请在订单项前添加破折号（-），星号（*）或加号（+）。缩进一个或多个项目以创建嵌套列表
```txt
- First item
- Second item
- Third item
- Fourth item
```
效果:
- First item
- Second item
- Third item
- Fourth item
- 在列表中添加元素

要在保留列表连续性的同时在列表中添加另一个元素，请将该元素缩进四个空格或一个制表符，如以下示例列表中加一个引用。
```txt
*   This is the first list item.
*   Here's the second list item.

    > A blockquote would look great below the second list item.

*   And here's the third list item.
```

# 代码块
- 在代码块的每一行缩进至少四个空格或一个制表符。
- 不缩行的代码块，在代码块之前和之后的行上使用三个反引号(```)或三个波浪号`~~~`
```txt
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```

## 代码块中语法高亮
在开始的反引号指定语言,如：`txt`,`java`,`go`,`json`

~~~json
```txt
语法高亮
```
~~~

# 链接
- 请将链接文本括在方括号（例如[ipoo Blog]）中，然后立即在URL后面加上括号（例如(https://ipooli.com)）中的URL。<br>  
```txt
精美的blog: [ipoo Blog](https://ipooli.com).
```
效果: <br>  
ipoo的blog: [ipoo Blog](https://ipooli.com).
- 可以选择为链接添加标题。当用户将鼠标悬停在链接上时，将显示.
```txt
ipoo的blog: [ipoo Blog](https://ipooli.com "最好的博客").
```
效果: <br>  
ipoo的blog: [ipoo Blog](https://ipooli.com "最好的博客").
# 网址和电子邮件地址
- 要将URL或电子邮件地址快速转换为链接，请将其用`<>`括起来。
```txt
<https://ipooli.com>
```
效果:

<https://ipooli.com>
- 链接放到段落里
```txt
链接放在段落里，参考[ipoo blog link][1]

[1]<https://ipooli.com> "ipoo blog link"
```
效果：

链接放在段落里，参考[ipoo blog link][1]

[1]: <https://ipooli.com> "ipoo blog link"

- 自动转网址链接 直接输入链接，许多Markdown处理器会自动将URL转换为链接

# 转义字符
- 请在字符前面添加`\`
```txt
\* Without the backslash, this would be a bullet in an unordered list.
```

# Markdown使用表情符号
- 表情符号以冒号开头和结尾，并包含表情符号的名称。
可以参考[表情简码](https://www.webfx.com/tools/emoji-cheat-sheet/)

:green_heart:

本篇完。


**注：** 本篇大部分为直译 *[Markdown Guide](https://www.markdownguide.org)*. 只供个人参考学习。



