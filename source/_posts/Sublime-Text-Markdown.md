---
title: 使用Sublime Text 3写Markdown
abbrlink: 0001
date: 2020-03-02 13:00
summary: '目前Sublime Text 3已经支持高亮显示原始Markdown和MultiMarkdown语法，如果不需要预览功能或是GitHub Flavored Markdown支持，可以直接使用。'
categories: 软件工具
tags:
- Sublime Text
- Markdown
---

## 首先介绍几个Markdown语法说明
- 1：Markdown 语法说明（详解版）
- 2：Markdown 语法说明 (简体中文版)
- 3：GitHub 风格的 Markdown 语法
- 4：GitHub 上的书写方式

目前Sublime Text 3已经支持高亮显示原始Markdown和MultiMarkdown语法，如果不需要预览功能或是GitHub Flavored Markdown支持，可以直接使用。

安装中需要使用Package Control，没有安装的可以看这边。

## 安装Markdown Preview 插件
Mardown Preview不仅支持在浏览器中预览markdown文件，还可以导出html代码。下面我们通过Package Control安装。

### 1.安装
通过按组合键Ctrl+Shift+P或是点击Preference->Package Control调出命令面板，然后再输入 install，选择 Package Control: install package。

![install package](/medias/sublime/sublime-install.png)

在插件安装面板输入markdown找到Markdown Preview并点击安装即可。

![install markdown](/medias/sublime/sublime-markdown.png "安装MarkdownPreview")

### 2.使用
通过按组合键Ctrl+Shift+P或是点击Preference->Package Control调出命令面板，输入mdp，下图中红框圈出的就是在浏览器中预览markdown文件。

![运行 markdown](/medias/sublime/sublime-mdp.png "运行MarkdownPreview")

选中后，你将见到两个选项：GitHub和Mardown。GitHub选项意味着使用GitHub的在线API来解析.md文件。它的解析速度取决于你的联网速度。据称有每天60次访问的限制。[2]但能免费获得GFM格式的语法支持和EMOJI表情的支持。

另外一个常用功能是图中第五个，Export HTML in Sublime Text，即导出html文件到sublime text。

### 3.快捷键设置
Sublime Text支持自定义快捷键，markdown preview默认没有快捷键，我们可以自己为preview in browser设置快捷键。方法是在Preferences -> Key Bindings User打开的文件的中括号中添加以下代码(可在Key Bindings Default找到格式)：

```bash
{ "keys": ["alt+m"], "command": "markdown_preview", "args": {"target": "browser", "parser":"markdown"} }
```

这里：
"alt+m"可设置为自己喜欢的按键。
"parser":"markdown"也可设置为"parser":"github"，改为使用Github在线API解析markdown。

### 4.设置语法高亮和mathjax支持
语法高亮跟编辑器的主题有关，可以在Preferences ->Color Scheme找自己喜欢的主题。

关于目录生成，只要文章是按照markdown语法写作的。在需要生成目录的地方写[TOC]即可。

设置mathjax支持需要在Preferences ->Package Settings->Markdown Preview->Setting User中增加如下代码[5]：

```bash
{
    /*
        Enable or not mathjax support.
    */
    "enable_mathjax": true,

    /*
        Enable or not highlight.js support for syntax highlighting.
    */
    "enable_highlight": true,
}
```

### 打印成pdf
将markdown转换为pdf应该有很多种方法的。可直接用谷歌浏览器虚拟打印功能生成。[3]
利用Markdown Preview的Preview in Browser功能可以在浏览器上看到htm效果。在页面右键->打印->另存为pdf->调节页边距即可将pdf文件下载下来。

## 安装使用Markdown Editing
Github项目地址：SublimeText-Markdown/MarkdownEditing

### 1.安装
如果Sublime安装了Package Control,直接使用组合键Command+Shift+P调出命令面板，输入 “package install” 从列表中选择 “install Package” 然后回车。再输入MarkdownEditing，找到后点击即可自行安装，重启便可使用。

### 2.使用
除了高亮显示语法，MarkdownEditing 还提供了一些快捷键用于快速插入markdown 标记。引用6中有比较详细的使用方式，这里仅作简要介绍。
常用的(更多的快捷键请参阅其官方文档)有：

插入链接：Ctrl + Win + K
插入图片：Shift + Win + K

### 3.code snippet
输入 “mdi + tab” 会自动插入下面的图片标记

```bash
![Alt text](/path/to/img.jpg "Optional title")

![图片alt](图片地址 ''图片title'')

图片alt就是显示在图片下面的文字，相当于对图片内容的解释。
图片title是图片的标题，当鼠标移到图片上时显示的内容。title可加可不加
```

输入 “mdl + tab” 会自动生成下面的链接标记

```bash
[](link)

[超链接名](超链接地址 "超链接title")
title可加可不加
```

## 另外的插件：OmniMarkupPreviewer
这个插件貌似功能很强大，用于markdown这是其中的一种功能。因为上两种插件已经够用，就不再研究它了，仅仅Mark下。
