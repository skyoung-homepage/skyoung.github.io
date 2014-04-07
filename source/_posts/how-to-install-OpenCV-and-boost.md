---
title: ubuntu 12.04 LTS下安装OpenCV，Boost等常用库
date: '2013-05-01'
categories: [How to]
tags: [OpenCV, Boost, Eigen, libconfig]

---
最近在搞一个视觉的程序，要用到一些库，就想着把这些库的安装细节记录一下。其实以前也有弄过，只是一旦安装好了，时间长了就容易忘记，特别是一些小细节的设置。如果碰上换电脑或是重装系统，又得重新研究，所以这里权当是个笔记，以备以后用到时查找。这里主要介绍**OpenCV，Boost，Eigen，libconfig**这几个库的安装和设置，其实过程很类似，但考虑到以后用起来省劲，这里也不厌其烦的全部列出了。

####OpenCV
OpenCV的安装在官网上有个教程，但是按照那个教程总是会出现问题，不尽如人意，这里记录一个参考[Installing OpenCV 2.4.1 in Ubuntu 12.04 LTS](http://www.samontab.com/web/2012/06/installing-opencv-2-4-1-ubuntu-12-04-lts/)这篇博文的OpenCV的安装教程备忘。我安装的是OpenCV2.4.5，其他相近的版本方法应该也是类似。我们希望OpenCV能支持一些第三方的库，如Qt，OpenGL，Python，TBB等，所以这里需要提前安装一些相应的依赖包，然后在cmake的时候做一些设置。

<!--more-->

1. 保证系统中各个软件和软件源都是更新过的，执行下面命令：

        #更新源
        $ sudo apt-get update 
        #更新已安装的包
        $ sudo apt-get upgrade 
    
2. 然后需要安装一些依赖包，像用于图像读写，摄像头驱动之类的包（v4l），执行下面命令：
    
        $ sudo apt-get install build-essential libgtk2.0-dev libjpeg-dev libtiff4-dev libjasper-dev libopenexr-dev cmake python-dev python-numpy python-tk libtbb-dev libeigen2-dev yasm libfaac-dev libopencore-amrnb-dev libopencore-amrwb-dev libtheora-dev libvorbis-dev libxvidcore-dev libx264-dev libqt4-dev libqt4-opengl-dev sphinx-common texlive-latex-extra libv4l-dev libdc1394-22-dev libavcodec-dev libavformat-dev libswscale-dev

3. 去[OpenCV官网](http://opencv.org/)下载源代码，解压后，进入相应的目录文件夹。因为我们希望OpenCV支持一些第三方库，所以这里需要设置一些东西，我们采用cmake的外部编译方法，先新建一个build文件，然后进入该文件夹进行编译。因为OpenCV在debug和release两种模式下生产的库文件不尽相同，debug模式因为添加了很多调试信息，文件大小相对会大一些，所以这里分别编译debug和release两种模式下的OpenCV。 对于debug模式，执行如下命令：

        $ mkdir build
        $ cd build
        $ cmake -D WITH_TBB=ON -D BUILD_NEW_PYTHON_SUPPORT=ON -D WITH_V4L=ON -D INSTALL_C_EXAMPLES=ON -D BUILD_EXAMPLES=ON -D WITH_QT=ON -D WITH_OPENGL=ON -D CMAKE_BUILD_TYPE=DEBUG -D CMAKE_INSTALL_PREFIX=/usr/local/OpenCV/Debug ..

    这里有个地方需要注意，命令最后是两个点，是上一层目录的意思，因为Cmakelist.txt在build的上层目录中。安装路径通过`CMAKE_INSTALL_PREFIX`设置为`/usr/local/OpenCV/Debug`，默认是`/usr/local`。而编译类型通过`CMAKE_BUILD_TYPE=DEBUG`设为DEBUG，默认是RELEASE。对于Release下的编译，只需要更改这两个部分即可，即：

		$ cmake -D WITH_TBB=ON -D BUILD_NEW_PYTHON_SUPPORT=ON -D WITH_V4L=ON -D INSTALL_C_EXAMPLES=ON -D BUILD_EXAMPLES=ON -D WITH_QT=ON -D WITH_OPENGL=ON -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local/OpenCV/Release ..

4. cmake结束后，检查输出的各个选项是否符合要求。特别注意FFMPEG（用于视频的读写）和V4L（摄像头的驱动），因为这俩是我最常用的功能，我这里特别点出。我的配置的结果如下图：

    ![](/media/How-to/terminal.png)

5. 编译安装OpenCV，命令如下：
       
        $ make
        $ sudo make install

6. 设置库的搜索路径，在`/etc/ld.so.conf.d/`加入对于库的`.conf`文件，首先新建`opencv.conf`
        
        $ sudo gedit /etc/ld.so.conf.d/opencv.conf
    
    在`opencv.conf`中加入函数库所在的目录：

		/usr/local/OpenCV/Debug/lib
		/usr/local/OpenCV/Release/lib

    也可以在`ld.so.conf`文件中
        
        $ sudo gedit /etc/ld.so.conf
    
    加入上述目录。其实`ld.so.conf`中写着如下代码：
    
        $ include /etc/ld.so.conf.d/*.conf
    
    说明`ld.so.conf`是包含`ld.so.conf.d`中所有的`.conf`文件的。

7. 接下来利用`ldconfig`命令，将`/etc/ld.so.conf.d`中的数据读入缓存，

        $ sudo ldconfig

8. 然后添加pkg-config环境变量，便于pkg-config找到`*.pc`文件。因为我们生成了两个版本的OpenCV，为了便于区分，分别把Debug和Release下`lib/pkgconfig`下对应的`*.pc`文件改名为`opencv-debug`和`opencv-release`。然后把这两个路径设置在环境变量中，

        $ sudo gedit /etc/profile
    
    把以下代码写入文件的最后并保持。

		export PKG_CONFIG_PATH=/usr/local/OpenCV/Debug/lib/pkgconfig:$PKG_CONFIG_PATH
		export PKG_CONFIG_PATH=/usr/local/OpenCV/Release/lib/pkgconfig:$PKG_CONFIG_PATH

或者，直接把这两文件放到一起，然后放到某个位置，再把这个路径设置到环境变量中。pkg-config的默认搜索路径是`/usr/local/lib/config`，所以可以直接放在这个路径下，无需设置环境变量。

这样OpenCV的安装就完成了，可以参考[Using OpenCV with gcc and CMake](http://docs.opencv.org/doc/tutorials/introduction/linux_gcc_cmake/linux_gcc_cmake.html#linux-gcc-usage)中讲解的OpenCV使用方法尝试测试几个简单的程序试试。

使用动态库的程序在运行的时候需要搜寻动态库的位置，这里第6-7步设置库的搜索路径就是为了告诉程序动态库的位置，以确保程序的正常运行。第8步的作用是告诉[pkg-config](http://zh.wikipedia.org/wiki/Pkg-config)工具`*.pc`文件的位置，以便pkg-config得到编译预处理参数（宏定义，头文件位置等）和链接参数（库及依赖的库的位置，文件名等），方便程序的编译和链接。具体的理解可以参考这篇博客[理解 pkg-config 工具](http://www.chenjunlu.com/2011/03/understanding-pkg-config-tool/)。


####Boost

Boost的大部分文件是不需要编译的，因为都是头文件，直接把头文件复制到你想要的地方就可以了，然后使用的时候就把include头文件链接到程序中就可以了。这里着重讲讲需要编译的部分，官方网站给出如下需要编译的文件：

* Boost.Filesystem
* Boost.GraphParallel
* Boost.IOStreams
* Boost.MPI
* Boost.ProgramOptions
* Boost.Python (see the Boost.Python build documentation before building and installing it)
* Boost.Regex
* Boost.Serialization
* Boost.Signals
* Boost.System
* Boost.Thread
* Boost.Wave

而我正好用到Boost.Filesystem和Boost.System，所以要编译，就把这个部分记录一下。我主要参考的是官网中如下部分：[Easy Build and Install](http://www.boost.org/doc/libs/1_53_0/more/getting_started/unix-variants.html#easy-build-and-install)

1. 去官网下载[Boost](http://www.boost.org/)，解压到某个目录下，然后进入这个目录，执行如下命令，查看一下命令选项的设置
    
        $ ./bootstrap.sh --help

2. 设置boost的安装路径，比如`/usr/local/boost-1.53.0`，则命令如下：

        $ ./bootstrap.sh --prefix=/usr/local/boost-1.53.0
        
3. 安装boost
    
        $ ./b2 install

4. 设置库的路径，和上面OpenCV一样，有两种方法，这里选择第一种：

        $ sudo gedit /etc/ld.so.conf.d/boost.conf
        
    加入函数库目录：
    
        /usr/local/boost-1.53.0/lib
        
这样Boost的安装就完成了，非常简单。

####Eigen
其实，Eigen也是只有头文件，直接复制到合适的地方就可以。还有一种方法是INSTALL文件提到的利用cmake的安装的方法，这里简单介绍一下：

1. 首先是外部编译，建立新的文件夹build，设置安装路径，比如我的是/usr/local/Eigen-3.1.3

        $ mkdir build
        $ cd build
        $ cmake -D CMAKE_INSTALL_PREFIX=/usr/local/Eigen-3.1.3 ..
        
2. 安装Eigen到指定的路径，由于没有需要编译的东西，就不需要make了。

        $ sudo make install

这样Eigen就安装结束了。

####libconfig

libconfig没有采用cmake构建makefile，而是有一个configure文件负责makefile的生成。具体步骤如下：

1. 先输入如下命令
        
        $ ./configure --help
    
    查看一下configure里面的一些选项，看一看有没有自己用得到的，其中常用的就是`--prefix`选项，用来设置安装路径的，假设我们设为`usr/local/libconfig-1.4.9`，则

        $ ./configure --prefix=usr/local/libconfig-1.4.9
        
2. 清除上一次编译的目标文件，这一步不是必须的，但做了没坏处，防止源代码中存在上一次编译的目标文件。
            
        $ make clean

3. 编译并安装，
    
        $ make
        $ sudo make install

4. 设置库的路径，

        $ sudo gedit /etc/ld.so.conf.d/libconfig.conf
    
    加入函数库目录：
    
        $ /usr/local/libconfig-1.4.9/lib

####总结

总的来说，库的安装还是比较相似的，一般是通过cmake或者自带的configure文件安装。而且库在发布的时候，会在readme或者intall文件中大体阐述库的安装方法，也可以去官网查询。具体思路一般就是先生成makefile，然后按照安装软件的方式安装即可。
    







    
    



