title: ubuntu14.04安装TexLive2014
tags: [texlive, texmaker, LaTex]
categories: [How to]
date: 2014-07-21 13:14:33
---

最近买了一本Latex方面的书，打算系统学习一下Latex，虽然以前也大概了解过LaTex，但是都是很零碎的学习，不够系统，而且现在大多也都忘记了。目前Tex的发行版主要有两种：CTex和TexLive，前者是Windows下的软件，后者则可以在多种常见系统下运行，包括Windows,MacOSX,Linux
各版本。这里仅就TexLive在ubuntu14.04下的安装做一个简单的介绍。

安装TexLive的方法有：通过CD，通过在线安装，通过ISO挂载（linux）这里仅就最后一种进行介绍。首先，要去[Acquiring TeX Live as an ISO image](https://www.tug.org/texlive/acquire-iso.html)下载最新的TexLive。这里建议网络不稳定的用户采用torrent下载，支持断点，否则很容易中间断掉，笔者就悲剧的出现这种情况，最后还是使用torrent下载的。 cd到TexLive的ISO文件所在文件夹下，然后挂载到`/mnt`上
```
mount -t iso9660 -o ro,loop,noauto texlive2014.iso /mnt
```

或者直接右键`使用磁盘映像挂载器打开`，相当于挂载到`/media/usrname/TeXLive2014`下面，这里`usrname`是你的电脑用户名。因为TexLive的图形化安装界面需要Perl/Tk模块的支持，所以首先需要安装相应模块，
```
sudo apt-get install perl-tk
```

然后执行，
```
cd /mnt
./install-tl -gui
```
<!--more-->

出现如下图形化安装界面，

![](/media/How-to/tex1.png)

接下来就很简单了，点击`安装TexLive`即可，接下来的画面是这样：

![](/media/How-to/tex2.png)

安装完成后需要设置环境变量，即在`～/profile`下加入
```
PATH=/usr/local/texlive/2014/bin/x86_64-linux:$PATH; export PATH
MANPATH=/usr/local/texlive/2014/texmf-dist/doc/man:$MANPATH; export MANPATH
INFOPATH=/usr/local/texlive/2014/texmf-dist/doc/info:$INFOPATH; export INFOPATH
```

这里注意，第一个PATH中的`/x86_64-linux`因各个电脑不同而不同，因为笔者的电脑是64位，所以是`/x86_64-linux`，32位应该是`/i386-linux`，可以自行去相应文件夹下查询。

最后需要配置一下字体，否则无法支持中文显示。

1. 将`/usr/local/texlive/2014/texmf-var/fonts/conf`下的`texlive-fontconfig.conf`复制到`/etc/fonts/conf.d`目录下，并且改名为`09-texlive.config`，
也可以执行如下命令,
```
sudo cp /usr/local/texlive/2014/texmf-var/fonts/conf/texlive-fontconfig.conf /etc/fonts/conf.d/09-texlive.conf
```

2. 刷新fontconfig的字体缓存，执行如下命令：
```
sudo fc-cache -fsv
```

这样就为系统配置了TexLive自带的字体。

3. 为了让pdfTex,dvipdfmx等程序能正确找到系统的安装的字体，需要设置OSFONTDIR变量，在`/usr/local/texlive/2014/`下的配置文件`texmf.cnf`中，加入如下变量
```
OSFONTDIR = /usr/share/fonts//;/usr/local/share/fonts//;~/.fonts//
```

TexLive安装完成后，可以考虑安装一些针对Latex的专门编辑器，比如Texmaker，TeXstudio，TeXworks等。比如笔者的使用的就是Texmaker，感觉很方便，去官网上下载相应的软件安装就可以。这里有一个问题，安装`Ubuntu 14.04 Qt5 amd64 package : texmakerQT5_ubuntu_14.04_4.2_amd64.deb (Qt5)`这个版本，中文无法输入，所以笔者安装的是`Ubuntu 14.04 amd64 package : texmaker_ubuntu_14.04_4.2_amd64.deb (Qt4)`这个版本，中文是可以正常输入。
