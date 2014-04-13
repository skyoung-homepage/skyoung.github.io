---
title: TexMacs札记(III) ——输入表格
date: '2013-05-29'
categories: [TexMacs]
tags: [TexMacs]

---

好久没有更新博客了，月初去德国开了个会议，回来后又赶上毕业答辩，折腾了许久，然后就是毕业后的各种表格的填写，搞得我头都大了。今天终于搞得差不多了，闲下来继续关于TexMacs的讲解。这一讲主要说一下TexMacs表格的输入方法。

####有框无框表格的输入

表格输入是通过`Insert->Table`进行插入的，可以先插入一个带标题的表格，这里分为两种一种是大标题，一种是小标题，分别是`Insert->Table->Big table`和`Insert->Table->Small table`。然后可以在这种带标题的方框内插入普通的表格，这里分为有框和无框两种，每一种又分为居中和普通的两种，有框称为`block`，无框的是`tabular`。

![](/media/TexMacs/table1.png "图 1")

<!--more-->

如上图所示，列出了各种表格的示意图。

* table 1.是无框普通小标题，插入方式：`Insert->Table->Plain tabular`;
* table 2.是无框居中小标题，插入方式：`Insert->Table->Centered tabular`;
* table 3.是有框普通小标题，插入方式：`Insert->Table->Plain block`;
* table 4.是有框居中小标题，插入方式：`Insert->Table->Centered block`;
* table 5.是大标题。
* 这里注意，以上带标题的表格首先需要插入一个`Big table`或者`Small table`，然后按照上述方式插入各种类型的插入各种格式的表格。

####矩阵行列式的输入
除此之外，table还可以结合公式输入矩阵，行列式，大括号选择以及普通的堆叠，如下图所示。

![](/media/TexMacs/table2.png "图 2")

首先通过`Insert->Mathematics->Formula`使输入模式为公式状态，然后点击`Insert->Table`下拉列表进行选择

* table 6.是矩阵输入，插入方式：`Insert->Table->Matrix`,通过工具栏![](/media/TexMacs/matrix.png)进行增加或缩减矩阵中元素的个数；
* table 7.是行列式输入，插入方式： `Insert->Table->Determinant`，元素个数的增减方法同上；
* table 8.是选择的输入，插入方法：`Insert->Table->Choice`，元素个数的增减方法同上；
* table 9.是堆叠的输入方法，插入方法：`Insert->Table->Stack`，它和table 2.的不同在于每个元素的间隔是不能改变的。

以上便是对表格输入基本的介绍，更多的功能可以参考TexMacs手册-[Tabular material](http://www.texmacs.org/tmweb/manual/webman-table.en.html)。但是需要注意这个手册有些内容更新不及时，需要择优参考。
