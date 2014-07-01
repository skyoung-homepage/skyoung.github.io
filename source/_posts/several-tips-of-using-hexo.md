title: 使用hexo构建博客时几个需要注意的地方
date: 2014-05-11 10:05:29
tags: [hexo, blog]
categories: [Tips]
---

前段时间把博客转到[hexo](http://hexo.io/)，个人感觉hexo的使用还是很方便的，特别是布局到github上很方便，而且使用Nodejs，生成页面速度很快。但是在具体使用的过程还是遇到了很多问题，这里做一些记录以备以后查阅。

###安装Nodejs

按照hexo上面的方法安装Nodejs总是出现无法找到npm命令的问题，一开始以为是环境变量设置的问题，但后来设置了也没有作用，不知道什么原因，所以最后还是按照Nodejs的官方教程安装。这里有安装有两种方法：

* 第一种方法，也是最简单的方法，即直接下载`linux binaries`，然后解压后，把这些文件直接复制到`/usr/local`目录下就可以了。这里需要注意由于向`/usr/local`下复制文件需要管理员权限，所以可以使用`sudo nautilus`打开文件管理窗口，然后执行复制操作。
* 第二种方法就是直接编译源码，下载`source code`,然后解压，按照里面的`readme`逐步执行以下操作即可。
```
$./configure
$ make
$ make install
```

<!--more-->
###安装hexo
这一步相对简单，直接使用nodejs的管理模块npm直接安装即可，即执行
```
$ npm install hexo -g
```

但似乎执行上述操作总会出现网络问题，所以这里需要设置一下npm的源地址，改为国内镜像的地址即可，执行
```
$ npm config set registry http://registry.cnpmjs.org 
```

###使用pandoc渲染html

因为[pandoc](http://pages.tzengyuxio.me/pandoc/)扩展了markdown很多实用的功能，而hexo目前还不支持pandoc，所以需要通过使用hexo的plugin使用pandoc。首先执行如下命令安装pandoc

```
$ sudo apt-get install pandoc
```

然后cd到相应的hexo博客文件夹下,执行
```
$ npm install hexo-renderer-pandoc --save
```

###npm代理设置
如果你使用代理上网，则需要设置npm的代理。这里为了便于描述，假设代理地址为`http：//proxy.website.com`,端口为`8080`,账户名为`skyoung`,密码为`123456`，则，执行如下命令
```
$ npm config set proxy="http://skyoung:123456@proxy.website.com:8080/"
```

这里顺便说一下ubuntu系统下如何设置代理上网，对于普通的网页浏览，需要打开依次打开`系统设置->网络->网络代理`，填入一下信息即可
```
skyoung:123456@proxy.website.com:8080
```

如何要设置系统的更新源，则需要执行下列命令打`apt.conf`文件
```
$  sudo gedit /etc/apt/apt.conf
```

写入：
```
Acquire::http::proxy "http://skyoung:123456@proxy.website.com:8080/";
Acquire::https::proxy "http://skyoung:123456@proxy.website.com:8080/";
```

要想使整个系统环境下都可以上网，比如使用`git clone`从github上面下载文件，还需要设置`bash.bashrc`文件，即执行
```
$ sudo gedit /etc/bash.bashrc
```

写入：
```
export http_proxy=http://skyoung:123456@proxy.website.com:8080/
export https_proxy=http://skyoung:123456@proxy.website.com:8080/
```

###总结
一开始觉得hexo使用应该很简单，但是实际操作中还是遇到各种问题，特别是nodejs的安装和hexo安装中问题多多，很是折腾。这次把这些问题记录下来，也省的别人遇到相应问题再折腾了。
