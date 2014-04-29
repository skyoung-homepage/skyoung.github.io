title: 如何使用pkg-config工具编译链接第三方库
date: 2014-04-28 16:38:04
tags: [pkg-config]
categories: [Tips]
---

我们在使用第三方库（如OpenCV,Boost等）时，难免会遇到相关的库的编译连接问题，而每次重复繁杂的配置的确让人懊恼，而pkg-config[^1]工具提供了一个相对简单的库的编译链接方法。这里就简单介绍一下pkg-config的使用方法。

我们知道，第三方库的使用主要涉及头文件的路径设置，库的路径设置以及动态库的环境变量设置。当然有的库的使用是仅仅需要头文件，例如Eigen以及大部分Boost，这样就不需要设置库路径和动态库的环境变量。pkg-config通过读取一个`*.pc`的文件，获取了库的头文件位置和库的路径等信息，然后告知编译器，实现库的自动使用。一般来说，`*.pc`文件的大体内容如下格式：


	prefix=/usr/local
	exec_prefix=${prefix}
	includedir=${prefix}/include
	libdir=${exec_prefix}/lib

	Name: foo
	Description: The foo library
	Version: 1.0.0
	Cflags: -I${includedir}/foo
	Libs: -L${libdir} -lfoo

<!--more-->

其中，

* `prefix`一般是指定库的默认安装路径
* `exec_prefix`一般是指库的另外指定的安装路径
* `inludedir`指定库的头文件路径
* `libdir`指定库的lib文件的路径
* `Name`指定库的名称，比如OpenCV，Boost等
* `Description`表示库的描述
* `Version`是版本号
* `Cflags`是gcc链接头文件的指令,以`-I`紧接头文件路径设置
* `Libs`是gcc链接lib文件的指令, 是`-L`紧接lib文件路径，`-l`紧接所使用的lib的名字。除此之外还有另外一种设置方法就是直接写成所引用的库文件路径加库的名称，即`Libs: ${libdir}/foo.a`,OpenCV提供的`*.pc`文件就是这样写的。

当然并不是每个库都会提供写好的`*.pc`文件，OpenCV的linux版就提供了`opencv.pc`文件，而Boost就没有提供，这里就需要自己写一个，下面贴出一个我写好的`boost.pc`文件作为参考。boost库大部分库是不需要编译的，仅需头文件的，而我最近正好用到需要编译的库文件`filesystem`和`system`，所以这里的示例一个单独使用这两个库文件的方法。

	prefix=/usr/local
	exec_prefix=${prefix}/boost-1.55.0
	libdir=${exec_prefix}/lib
	includedir=${exec_prefix}/include

	Name: Boost
	Description: A C++ Libraries
	Version: 1.55.0
	Cflags: -I${includedir}
	Libs:  -L${libdir}/ -lboost_filesystem -lboost_system

这里需要注意一个问题，就是boost库的Libs设置在使用`-l`方法是需要去掉前面的`lib`前缀（实际生成的这两个文件分别是`libboost_filesystem.a`和`libboost_system.a`，原因可参考[Link Your Program to a Boost Library](http://www.boost.org/doc/libs/1_55_0/more/getting_started/unix-variants.html#link-your-program-to-a-boost-library)），而在使用第二种方法的时候则不需要，即

	Libs:  ${libdir}/libboost_filesystem.a  ${libdir}/libboost_system.a

上面的方法使用的是boost的静态库，而对于第一种方法链接的是动态库，这是需要设置环境变量告知系统动态库的位置，即使用ldconfig工具，以boost为例，首先打开一个`boost.conf`文件，

	$ sudo gedit /etc/ld.so.conf.d/boost.conf

加入boost的库文件路径,例如
	

	/usr/local/boost-1.55.0/lib

然后执行

	$ sudo ldconfig

把boost库路径读入缓存，使系统能搜寻到库的位置。这里有个问题需要注意，你可能发现在实际使用中，比如Qt开发环境下，没有设置这些环境变量，仍然可以运行，其实这些路径的设置在pkg-config工具中已经临时设置好了，所以可以执行，但当你直接执行可执行程序时，就会出现`cannot open shared object file: No such file or directory`的报错。

另一个很重的问题是告诉pkg-config工具你为库所写的`*.pc`文件的路径，这样pkg-config才能找到`*.pc`文件并读取库的路径链接信息。设置环境变量

	$ sudo gedit /etc/profile

加入

	export PKG_CONFIG_PATH=/usr/local/Boost/pkgconfig:$PKG_CONFIG_PATH

这里假设Boost的pkg-config文件`boost.pc`放在了`/usr/local/Boost/pkgconfig`下。

以上所有步骤准备好了以后就可以使用pkg-config了。以上面已经写好的`boost.pc`文件文件为例，在命令行下直接输入

	$ pkg-config --cflags boost
其输出为

	-I/usr/local/boost-1.55.0/include

输入

	$ pkg-config --libs boost

输出

	-L/usr/local/boost-1.55.0/lib/ -lboost_filesystem -lboost_system

对于链接静态库时，可以加入`--static`选项，即

	$ pkg-config --libs --static boost 
这样通过这些命令就可以很方便的实现程序的编译链接，例如使用g++编译个foo文件，调用了boost库，则可以执行如下命令

	$ g++ `pkg-config --cflags --libs boost` -o foo foo.cpp

以上语句相当于用`pkg-config --cflags --libs boost`替换了`-I/usr/local/boost-1.55.0/include -L/usr/local/boost-1.55.0/lib/ -lboost_filesystem -lboost_system`，其实还是g++的那些命令选项。

对于支持g++编译命令选项的开发环境，使用pkg-config也是非常简单的，比如CodeBlocks和QtCreator就支持g++，gcc编译命令，具体使用方法可以参考另一篇博客[如何在CodeBlocks和Qt Creator开发环境下使用OpenCV](http://skyoung.github.io/2014/03/06/how-to-use-QtCreator-and-Codeblocks-with-OpenCV/)


[^1]:[Guide to pkg-config](http://people.freedesktop.org/~dbn/pkg-config-guide.html)
