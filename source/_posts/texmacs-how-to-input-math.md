---
title: TexMacs札记(II)
tagline: ——输入数学公式
date: '2013-04-27'
categories: [TexMacs]
tags: [TexMacs]

---
上一篇博客初步介绍了TexMacs的基本使用方法，这次着重介绍一些TexMacs的公式输入。TexMacs的公式输入非常便捷，而且其“Tab键式”的公式输入方法非常有意思，也非常有想象力。

###如何在文档中插入数学公式
TexMacs提供了三种插入数学公式的方法：

* 用`Insert->Mathematics->Formula`插入内联公式。还可以使用快捷键`$`，即`Shift+4`或者工具栏{% img inline /media/TexMacs/input.png 25 25%}下拉菜单选择。
* 用`Insert->Mathematics->Equation`插入显示公式。还可以使用快捷键`Alt+$`,即`Alt+Shift+4`或工具栏{% img inline /media/TexMacs/input.png 25 25%}下拉菜单选择。使用`Ctrl+#`可以为公式编号，使用`Ctrl+Tab`可以在内联公式与显示公式之间切换。
* 用`Insert-.Mathematics->Equations`插入多行公式。还可以是用快捷键`Alt+&`,即`Alt+Shift+7`或者工具栏{% img inline /media/TexMacs/input.png 25 25%}下拉菜单选择

<!--more-->

###如何插入数学符号
TexMacs的数学符号输入非常有特点，对于希腊字母的输入采用键盘上相似的键输入，然后按Tab键进行切换，比如你想输入$\alpha$，只需按下键盘A键，然后按Tab键进行切换，这时就会出现$\alpha$字母，如果是$\beta$,那就先输入B，然后Tab切换，这时就会出现$\beta$，再按一下Tab就会出现$\flat$。其他数学符号的输入也很有特点，例如，输入`<`可以得到<，按一下`Tab`键就可以得到$\in$。由此，我们可以看出TexMacs的输入是非常的人性化和方便的。其他符号的输入可以参考这个。当然这些快捷键是针对常用的符号输入，我们没有必要记住所有的快捷键；对于一些不常用的符号输入，可以使用工具栏上的一些符号输入（如下图），非常的方便快捷。
![](/media/TexMacs/equation.png)

下面列出一些常用符号的快捷键输入方式一些规则：

* `F5`键是用来组合输入希腊字母的，例如`F5+A`生成$\alpha$，`F5+B`生成$\beta$。**工具栏对应**{% img inline /media/TexMacs/T.png 20 20 %}。
* `F5+Shift`键是用来输入常用大写希腊字母，如'F5+G'生成$\gamma$，则`F5+Shift+G`生成大写的$\Gamma$。
* `F6`键是用来组合生成加粗的希腊字母，例如`F6+A`生成$\boldsymbol{\alpha}$。**工具栏对应**{% img inline /media/TexMacs/B2.png 20 20 %}
* `F7+Shift`键是用来组合生成书法类字母的。例如`F7+Shift+A`生成{% img inline /media/TexMacs/A.png 20 20 %}**工具栏对应**{% img inline /media/TexMacs/E.png 20 20 %}。
* `F8`键用来组合生成哥特体字母，例如`F8+P`生成{% img inline /media/TexMacs/p.png %})，也有通过`F8+Shift`组合键生成的，例如`F8+Shifit+B`生成{% img inline /media/TexMacs/B.png 20 20%}。**工具栏对应**{% img inline /media/TexMacs/F.png 25 25%}。
* `Shift+F6`键用来组合插入黑板字母，例如`Shift+F6+D`可以生成{% img inline /media/TexMacs/D.png %}。**工具栏对应**{% img inline /media/TexMacs/B1.png 20 20%}。

一些组合键可以得到相应的数学符号，如依次输入`>，=`可以得到$\geqslant$，还可以通过Tab键切换到$\geq$。依次输入`-,-,>`可以得到$\longrightarrow$。还有很多其他特殊符号的输入也是有快捷键的，个人觉得记住一些常用的快捷键就可以了，其他符号的输入可以借助数学公式的工具栏。

###主要数学架构的输入

这里好多输入习惯和Latex是很相似的，当然也可以用Latex命令行的方式输入，比如输入分数，可以直接输入`\frac`，然后an`Enter`键就可以了，或者是用快捷键`Alt+F`，亦或是用工具栏。

以下列出几个常用的快捷方式：

* `Alt+F`：分数输入
* `Alt+S`：平方根输入
* `Alt+Shift+S`：n次方根的输入
* `Alt+N`：否定某个式子，比如{% img inline /media/TexMacs/negation.png 30 30 %}
* `'`：上撇号，例如$y'$
* `` ` ``：左上撇号，$\text{}^{\backprime} y$
* `_`：下标
* `^`：上标
* `Alt+L,_`：左下标
* `Alt+L,^`：左上标

###定界符号的的输入

定界号指数学中的圆括号，中括号，大括号等符号。默认情况下，TexMacs的定界符号输入是一旦你输入左半边，右半边会自动加载。如果你不习惯这样，可以在`Edit->Preference->Keyboard->Automatic brackets->Disable`设置。`Ctrl+*`键可以自动放大或缩写定界符号的大小。定量的设置定界符号的大小可以用`Format->Transform->Resize object`进行设置。

###大运算符的输入

大运算符主要包括，积分号，求和号，求积号等。输入的方式，一种是通过工具栏上的{% img inline /media/TexMacs/sum.png 20 20 %}键，一种是用快捷键，由于快捷键较为复杂，个人感觉使用工具栏较为快捷划算。所以这里就不列快捷键了，而且工具栏中也有快捷键对应，感兴趣可以看看。

###重音符号的输入

重音符号主要包括$\widehat{a},\overline{a},\overrightarrow{a}$等，输入的方式可以采用工具栏上的{% img inline /media/TexMacs/accent.png 20 20 %}键。

###总结
以上便是TexMacs数学输入的基本的介绍，需要注意的是所有这些快捷键的输入都是在公式环境下才有作用，在使用时一定要检查输入环境。对于具体的介绍可以参照TexMacs英文手册[Mathematical formulas](http://www.texmacs.org/tmweb/manual/webman-math.en.html)部分。


