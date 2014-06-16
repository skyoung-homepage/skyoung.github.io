---
title: TexMacs札记(IV) ——插入链接、图片、目录等内容
date: '2013-08-17'
categories: [TexMacs]
tags: [TexMacs]

---

距上一次写关于TexMacs的博客已经有一段时间了，因为毕业的原因，自己在家休整了一段时间，最近终于空闲下时间继续TexMacs手册的撰写。这篇博客主要讲解如何插入链接，图片，目录，参考文献等内容。

###插入标签、参考

这里标签、参考是指论文写作的时候常用的公式索引啊，或者章节索引之类的应用。像LaTex中一样，在公式或者章节处插入标签`label`，然后在索引的地方插入`reference`。注意这里标签的插入只能是在标题、公式、表格或者图片等处插入，而正文下插入label是不起作用的。具体插入方式：

* 插入label：使用菜单栏`Insert->Link->Label`；也可以通过工具栏{% img inline /media/TexMacs/link.png 20 20 %}进行插入，或者直接输入`\label`按回车。最后按回车激活该操作。如下图所示,`|`后面的**section1**是label的名称，以便后面ref参考使用。![](/media/TexMacs/label.png)\ 

* 插入reference：使用菜单栏`Insert->Link->Reference`；也可以通过工具栏{% img inline /media/TexMacs/link.png 20 20 %}进行插入，或者直接输入`\ref`按回车。最后按回车激活该操作。如下图所示，`|`后面输入要引用的label名称      
{% img /media/TexMacs/ref.png %} 

<!--more-->

###插入超链接

这个相对比较简单，就是通过菜单栏`Insert->Link->HyperLink`；也可以通过工具栏{% img inline /media/TexMacs/link.png 20 20 %}进行插入。如下图所示，后面有两个参数，第一个`|`后面是超链接的名称，第二个`|`是超链接的网址。最后按回车激活该操作。     
{% img /media/TexMacs/hyperlink.png %}

###插入图片

插入图片可以通过菜单栏`Insert->Image`实现,或者是工具栏{% img inline /media/TexMacs/link.png 20 20 %}进行插入。如下图所示，图片的大小以及位置可以通过弹出的对话框设置，目前支持的视频格式有ps、eps、tif、pdf、pdm、gif、ppm、xpm、以及fig，可惜不支持jpg。而且对于其他格式，我试了一下pdf和tif格式，发现老出现莫名的错误，看来TexMacs对图片的支持效果不好。     
{% img /media/TexMacs/insert-image.png 600 600 "插入图片" "插入图片" %}

###自动生成目录

这里插入目录和word的插入目录有点相似，只需要把光标放到你要插入目录的位置，然后通过菜单栏`Insert->Automatic->Table of contents`就可以插入目录，如下图所示。这里需要注意，因为TexMacs默认页面类型是**papyrus**，是没有页码的，需要先调整页面的类型，即`Document->Page->Type`修改为**paper**类型。当页码因为编辑发生变化时，可以通过`Document->Update->Table of contents`或者是`Document->Update->All`进行更新，这里注意，如果第一遍通过`Insert->Automatic->Table of contents`插入目录后没有反应，那就再通过`Document->Update->Table of contents`再 **Update** 一下就好。这里出了自动生成的内容都是类似的模式。包括自动生成图目录，表目录等。    
{% img /media/TexMacs/table-content.png 600 600 "生成目录" "生成目录" %}


###插入参考文献
TexMacs和Latex一样用的是**bibtex**来编译参考文献的。首先是按照Latex的要求编写**.bib**文件，然后在对应的文献引用处插入标号，即`Insert->Link->Citation`或者是插入隐形标号`Insert->Link->Invisible citation`。当然也可以通过工具栏{% img inline /media/TexMacs/link.png 20 20 %}进行插入。然后把光标放在你要生成参考文献的位置（一般是文章的末尾），通过菜单栏`Insert->Automatic->Bibliograpy`自动生成文献。当文献发生改变的时候，通过`Document->Updae->Bibliography`进行更新。此外，参考文献的格式是存放在 **~/.TeXmacs/system/bib** 下的。

###插入索引、术语表

这个和插入目录是很相似的，这里只需要在需要索引的或者某个术语位置插入相应的标记，然后通过自动生成即可。索引的插入标记方法是：`Insert->Link->Index entry`，术语的插入标记的方法是`Insert->Link->Glossary`。或者通过工具栏{% img inline /media/TexMacs/link.png 20 20 %}进行插入。然后把光标放在需要插入索引或者是术语表的位置，通过菜单栏的`Insert->Automatic->Index/Glossary`进行插入。同样当发生改变时通过`Document->Updae->Index/Glossary/All`进行更新。

###总结
总的来说，TexMacs在这个部分做的和Latex还是很像的，只不过好多东西更图形化了，更直观了，有点像word那种感觉。到此为止TexMacs的基本功能已经算是介绍完了，当然TexMacs还有presentation的功能，但相比较微软的ppt，其效果相对就比较鸡肋了。个人觉得TexMacs的重点在于其对科学文献的撰写，其排版如Latex般优美，界面如word般友好。但是目前许多paper的写作模板还是以Latex居多，而利用TexMacs自己编写模板还是有点麻烦，所以权且把TexMacs作为一个记事本使用，或者是撰写文献的草稿。而且TexMacs也支持导出Latex，html，xml，bibtex，pdf等多种格式的文件，其使用范围还是很广的。

