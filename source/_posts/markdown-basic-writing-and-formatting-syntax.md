---
title: Markdown basic writing and formatting syntax
date: 2018-11-11 00:51:28
tags: [markdown,tools,study]
categories: Tools
---
### 1.标题
在标题文本前添加1到6个`#`符号来创建一个标题，`#`符号的个数决定标题的大小，从1到6标题依次减小。
<!-- more -->
例如：
```
## The second largest heading
###### The smallest heading
```
效果如下：
## The second largest heading
###### The smallest heading

### 2.文本样式

|样式|语法|示例|输出|
| :------:| :------: | :------: |:-:|
| 加粗 | `** **`or `__ __` | `**This is bold test**` |**This is bold test**|
|斜体|`* *` or `_ _`|`*This text is italicized*`|*This is italicized*|
|删除线|`~~ ~~`|`~~This was mistaken text~~`|~~This was mistaken text~~|
|斜体加粗|`** **` and `_ _`|`**This text is _extremely_ important**`|**This text is _extremely_ important**|

### 3.引用
你可以用符号`>`来引用文本，例如：
```
In the words of Abraham Linconln:
> Pardon my French
```
效果如下：
In the words of Abraham Linconln:
> Pardon my French

你可以用一个反引号\`在语句中标出代码或者命令，反引号内的文本不会被格式化。
```
Use `git status` to list all new or modified files that haven't yet been commited.
```
Use `git status` to list all new or modified files that haven't yet been commited.
用三个反引号来格式化代码块或者文本块。
~~~
Some basic Git commands are:
```
git status  
git add  
git commit  
```
~~~
``` bash
Some basic Git commands are:
git status  
git add  
git commit  
```
### 4.链接
可以通过在中括号`[ ]`包装链接文本来创建一个内部链接，然后用大括号`( )`包装URL。也可以通过键盘快捷键`command + k`来创建一个链接。  
`This site was built using [Github Pages](https://pages.github.com).`  
This site was built using  [Github Pages](https://pages.github.com).
#### 相对链接
你可以在你的渲染文件里定义相对链接和图像路径来帮助读者导航到代码仓库的其他文件。

相对链接是相对于当前文件的链接。例如，如果你在仓库的根目录下有一个README文件，docs目录下有另一个文件docs/CONTRIBUTING.md，在README文件中CONTRIBUTING.md文件的相对链接可能如下：  
`[Constribution guidelines for this project](docs/CONTRIBUTING.md)`

GitHub将会根据你当前所在的分支自动转换你的相对链接或图像路径，所以链接或路径一直是有效的。你可以使用所有的相对链接操作符，比如`./`和`../`。  
相对链接使用户复制你的仓库更容易。绝对链接可能在你的仓库的副本中失效-建议在仓库中用相对链接来指向其他文件。
#### 插入图片
基础格式：
`![Alt text](图片链接 "optional title")`
> **Alt text**: 图片的Alt标签，用来描述图片的关键词，可以不写。   
  **图片链接**：可以是本地地址或者网络地址。   
  **optional title**：鼠标置于图片上会出现的标题文字，可以不写。
### 5.列表
你可以通过在一行文字前加上`-`或者`*`来创建一个无序列表。例如  
~~~
- George Washington
- John Adams
- Thomas Jefferson
~~~
效果如下：
- George Washington
- John Adams
- Thomas Jefferson

要使列表有序，在行前添加一个数字。例如：  
~~~
1. James Madison
2. James Monroe
3. John Quincy Adams
~~~
#### 嵌套列表
你可以通过在一个或多个列表下缩进来创建一个嵌套列表。  
如果要用GitHub上的web编辑器或者采用等宽字体的文本编辑器来创建一个嵌套列表的话，你可以直接对齐列表。在嵌套列表前边添加空格直到列表标识字符（`-`或者`*`）在文本的第一个字符正下方。例如：  
~~~
1. First list item
   - First nested list item
     - Second nested list item
~~~  
效果如下：
1. First list item
   - First nested list item
     - Second nested list item      


### 6.任务列表
要创建一个任务列表，可以在列表项前添加`[ ]`，用正则间隔符号填充。标记一个已完成的任务，可以用`[x]`。
```
- [x] Finish my changes
- [ ] Push my commits to GitHub
- [ ] Open a pull request
```
- [x] Finish my changes
- [ ] Push my commits to GitHub
- [ ] Open a pull request      

如果一个任务列表项已括号开始，你需要使用转义字符`\`;
```
- [ ] \(Optional) Open a followup issue
```
- [ ] \(Optional) Open a followup issue

更多信息参考"[About task lists](https://help.github.com/articles/about-task-lists/)."
### 7.插入公式
#### 行内公式
用`$`符号包括公式,例如：
输入`$E = mc^2$`,显示为：$E = mc^2$

#### 行间公式
用两个`$$`符号包括公式，例如：
输入`$$E = mc^2$$`，显示为:
$$E = mc^2$$   

### 8.更多阅读
- [GitHub Flavored Markdown Spec](https://github.github.com/gfm/).
- [About writing and formatting on GitHub](https://help.github.com/articles/about-writing-and-formatting-on-github/)
- [Working with advanced formatting](https://help.github.com/articles/working-with-advanced-formatting/)
- [Mastering Markdown](https://guides.github.com/features/mastering-markdown/)
