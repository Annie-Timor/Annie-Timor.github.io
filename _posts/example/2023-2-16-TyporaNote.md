---
title: TyporaNote
author: lixinghui
date: 2023-2-16 12:00:00 +0800
categories: [Blogging, Demo]
tags: [example]
---



## 标题居中

```
<h1 align = "center">标题居中的方法</h1>
```



## 将网页内容保存到markdown

使用软件：typora

步骤：

> 1、找到需要复制的网页
>
> 2、全选，复制
>
> 3、在typora中进行粘贴
>
> 4、把广告删除
>
> 5、对部分内容进行修改



## Typora数学公式输入指南


公式大全的链接

[https://www.zybuluo.com/codeep/note/163962#mjx-eqn-eqsample](https://www.zybuluo.com/codeep/note/163962#mjx-eqn-eqsample)

[公式2](https://zhuanlan.zhihu.com/p/261750408)

[公式3](https://blog.csdn.net/Small_Tsky/article/details/106799380)

[公式4](https://blog.csdn.net/qq_45866781/article/details/122201266)

常用公式：

| 符号   | 输入       | 显示                   |
| ------ | ---------- | ---------------------- |
| 分号   | \frac{}{}  | $\frac{1}{2}$          |
| 求和   | \sum_{}^{} | $\sum_{n=0}^{N-1}$     |
| 求积分 | \int_{}^{} | $\int_{n=1}^{+\infty}$ |



## Typora链接图片

在静态网站中，您可以通过以下两种方式来链接图片：

1、相对路径链接：如果您的图片与页面在同一个目录下，或者在页面的子目录中，那么您可以使用相对路径来链接图片。例如，如果您的图片文件名为`image.png`，并且与页面在同一个目录下，那么您可以使用以下代码来在页面中显示图片：

```
![图片描述](image.png)
```

2、绝对路径链接：如果您的图片不在页面所在的目录中，您可以使用绝对路径来链接图片。对于 GitHub Pages 来说，您可以将图片上传到您的仓库中，并使用该仓库的 Raw 链接来链接图片。例如，如果您的图片在您的 GitHub 仓库中的`images`目录下，您可以使用以下代码来在页面中显示图片：

```
![图片描述](https://raw.githubusercontent.com/<username>/<repository>/<branch>/images/image.png)
例如：
![](https://raw.githubusercontent.com/Annie-Timor/Annie-Timor.github.io/master/user_img/add_windows.jpg)
```

其中，`<username>`是您的 GitHub 用户名，`<repository>`是您的 GitHub 仓库名，`<branch>`是您的 GitHub 仓库的分支名（通常是`main`或`master`）。

`raw.githubusercontent.com`是 GitHub 提供的一个公共 Raw 数据托管服务。通过该服务，您可以通过 URL 直接访问 GitHub 上的文件的原始内容，而不是下载该文件。这可以方便地用于将 GitHub 上的文件链接到静态网站、在线文档等场景中。

## Typora链接文件

在静态网站内，链接其他文章需要确定该md文件被转换成了什么。

[C语言格式输出符 - 欢迎来到我的博客 (annie-timor.github.io)](https://annie-timor.github.io/2023/02/20/C语言格式输出符/)

```
https://annie-timor.github.io/2023/02/20/C语言格式输出符/
https://annie-timor.github.io--是仓库名称
2023--年
02--月
20--日
<这三个日期是md文件开头的date，会被补全为2个字符>
C语言格式输出符--这个是md文件开头的title
```

[链接C文件](https://github.com/Annie-Timor/Data-Structure/blob/master/CFree/%E2%96%BC%E9%85%8D%E5%A5%97%E4%B9%A0%E9%A2%98%E8%A7%A3%E6%9E%90/%E2%96%BC02%20%E7%BA%BF%E6%80%A7%E8%A1%A8/%E2%96%BC%E4%B9%A0%E9%A2%98%E6%B5%8B%E8%AF%95%E6%96%87%E6%A1%A3-02/Question-2.11-main.c)

```
[显示名称](链接网址)
```

