---
layout: post
title: "TextView 常用功能总结"
modified: 2018-1-25 18:44:00
excerpt: "样式设置、滚动、Html..."
tags: [Android]
comments: true
---


### 一、设置样式

URL：<http://www.cnblogs.com/bdsdkrb/p/5715438.html>


### 二、添加滚动

```Kotlin
tv_read_text.movementMethod = ScrollingMovementMethod.getInstance()
```

### 三、Html.fromHtml() 支持的标签

URL：<http://zyoo005.iteye.com/blog/1523874>

```Java
TextView textView=(TextView)findViewById(R.id.hello);   
textView.setText(Html.fromHtml("Hello <b>World</b>,<font size=\"3\" color=\"red\">AnalysisXmlActivty!</font>"));   
```



```
<a href="...">  定义链接内容  
<b>  定义粗体文字   b 是blod的缩写  
<big>  定义大字体的文字  
<blockquote>  引用块标签   
属性:  
Common  -- 一般属性  
cite  -- 被引用内容的URI  
<br>   定义换行  
<cite>   表示引用的URI  
<dfn>   定义标签  dfn 是defining instance的缩写  
<div align="...">  
<em>  强调标签  em 是emphasis的缩写  
<font size="..." color="..." face="...">  
<h1>  
<h2>  
<h3>  
<h4>  
<h5>  
<h6>  
<i>   定义斜体文字  
<img src="...">  
<p>     段落标签,里面可以加入文字,列表,表格等  
<small>  定义小字体的文字  
<strike>   定义删除线样式的文字   不符合标准网页设计的理念,不赞成使用.   strike是strikethrough的缩写  
<strong>   重点强调标签  
<sub>   下标标签   sub 是subscript的缩写  
<sup>   上标标签   sup 是superscript的缩写  
<tt>   定义monospaced字体的文字  不赞成使用.  此标签对中文没意义  tt是teletype or monospaced text style的意思  
<u>   定义带有下划线的文字  u是underlined text style的意思  
```

