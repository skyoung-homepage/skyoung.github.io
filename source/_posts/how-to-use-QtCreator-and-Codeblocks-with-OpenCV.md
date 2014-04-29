---
title: "如何在CodeBlocks和Qt Creator开发环境下使用OpenCV（Ubuntu12.04）"
date: '2014-03-06'
categories: ['How to']
tags: [Ubuntu, Qt Creator, CodeBlocks, OpenCV]

---

自从切换到ubuntu系统后，就一直在寻找合适的C++开发环境。看了网上的一些介绍，大体就这么几个选择：Eclipse， [CodeBolocks]()， [Qt Creator]()。后来自己试用了一下CodeBlocks和Qt Creator，感觉这两个开发环境还是不错，打开速度都很快。但是由于CodeBlocks的`Find declaration`功能针对OpenCV老是不灵，这让我很不爽，所以果断换掉去尝试Qt Creator。Qt Creator给人焕然一新的感觉，界面简洁，布局合理，常用的功能在左侧排列，不会给人杂乱的感觉。

因为要开发OpenCV项目，所以首先需要知道开发环境配置第三方库的方法。虽然最后没有选择CodeBlocks，但是毕竟也摸索过一段时间，这里以OpenCV为实例一并介绍其如何配置第三方库。


###CodeBlocks下配置OpenCV

CodeBlocks的安装主要可以参考[官网的教程](http://www.codeblocks.org/downloads)。主要两种方式，一是下载源代码，自己编译安装；再就是安装别人编译好的，对于ubuntu系统参考[这里](https://launchpad.net/~pasgui/+archive/ppa/)。把这个源加入自己的系统，即执行
	
	sudo add-apt-repository ppa:pasgui/ppa

然后执行
	
	sudo apt-get install codeblocks 
这样 就可以CodeBlocks就安装好了。

<!-- more -->

关于CodeBlocks配置第三方库的方法，网上也有很多，无非是在配置include和lib路径，但这些过程实在是繁琐，这不是我想要的。因为ubuntu下有pkg-config工具，有了这个东西，配置OpenCV不再是麻烦事。OpenCV 的安装的时候，会生成一个`*.pc`文件，pkg-config通过这个文件就可以自动获取OpenCV的include和lib文件的位置，实现自动配置。具体的OpenCV安装方法可以参考我的另一篇博客[ubuntu 12.04 LTS下安装OpenCV，Boost等常用库](http://blog.skyoung.org/how-to/ubuntu-12-04-lts%E4%B8%8B%E5%AE%89%E8%A3%85opencv-boost%E7%AD%89%E5%B8%B8%E7%94%A8%E5%BA%93/)。里面讲解了在OpenCV安装完成后，如何配置pkg-config的搜寻路径。只有pkg-config工具找到opencv的`*.pc`文件，才能获取到OpenCV的头文件和库的安装位置。

CodeBlocks支持pkg-config工具，只需要在工程的`build options`配置相关参数即可，以下操作参考[这里](http://aravindev.blogspot.jp/2013/06/configuring-and-installing-opencv-using.html)。

1. 依次找到`Compiler settings->Other options`，输入

		`pkg-config --cflags opencv`
如下图所示，这个命令是配置OpenCV的include路径的。
 
	![](/media/How-to/compiler.png)

2. 依次找到`Liner settings->Other linker options`, 输入

		`pkg-config --libs opencv`
如下图所示，这个命令是配置OpenCV的lib路径和连接的动态库。

	![](/media/How-to/linker.png)

**注意:** 这两个命令里面的`opencv`替换为你的`pkgconfig`文件下实际的文件名。

###Qt Creator下配置OpenCV

Qt Creator的安装非常简单，取[Qt官网](http://qt-project.org/downloads)下载linux版本的文件，双击运行即可。注意需要设置文件属性为可执行文件，右键`属性->权限`里面的`允许作为程序执行文件`打钩，或者打开终端，输入相应的修改文件属性的命令。Qt Creator目前有个小问题是在调试的时候，提示“ptrace 不允许的操作”，解决这个问题可以参考这篇[博客](http://www.deder.at/wordpress/?p=307)。首先打开`/etc/sysctl.d/10-ptrace.conf`这个文件，
	
	sudo gedit /etc/sysctl.d/10-ptrace.conf

然后把`kernel.yama.ptrace_scope = 1`改为`kernel.yama.ptrace_scope = 0`重启电脑即可。

Qt的下配置OpenCV更为简单，只需添加一个地方即可。右键单击工程，选择`Add Library`,

![](/media/How-to/addlib.png)

选择`System package`,点击`next`

![](/media/How-to/package.png)

输入`pkgconfig`文件下的`*.pc`文件名，注意不要输入文件后缀，文件名即可。

就这么简单，这样OpenCV的配置就完了。

###总结

个人感觉，由于有了pkg-config这种工具，ubuntu下使用OpenCV是那么的方便。虽然windows下的vs也可以通过制作工程模板，来简化OpenCV工程的配置，但是总感觉似乎不如ubuntu下来的智能。其实，不光是OpenCV，其他第三方库，只要提供了`*.pc`配置文件，就可以使用pkg-config，这样使得第三方库的使用异常便捷。





