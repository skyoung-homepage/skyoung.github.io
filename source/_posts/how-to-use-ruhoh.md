---
title: 如何使用ruhoh2.0.alpha搭建博客
date: '2013-04-24'
categories: "How to"
tags: [ruhoh, kramdown]

---

我只能说我最近比较闲，所以没事就是瞎折腾，刚用Jekyll Bootstrap构建的博客，没玩两天，就对ruhoh又产生了兴趣。[ruhoh](http://ruhoh.com/)是Jekyll Bootstrap的作者又重写的一个博客生成器，基于前几天对Jekyll Bootstrap的了解，来看这个ruhoh,感觉还是又很多相似之处的，特别是一些设置，主要感觉ruhoh好像是把代码搞得更整洁了，模块化更明显了，Jekyll Bootstrap总给人很乱的感觉。当然ruhoh的还有很多其他的特点，总的感觉就是简单易用。而且较为美观。

促使我改用ruhoh的还有一个原因就是Jekyll Bootstrap的首页的post预览是不带格式的，即像粗体，公式啥的直接是以文本的形式展现出来，让我感觉很不爽，本来想着改一下代码的，可惜自己能力实在有限，照葫芦画瓢弄了一阵没整出来，就想着直接换ruhoh算了，ruhoh提供一个叫summary的东西可以带格式的预览，而且新版的2.0.alpha提供了Posts Paginator功能，就是index页的翻页功能。

<!--more-->

###具体安装方法
我的系统是ubuntu12.04，主要的安装方法参考[coolaj86](http://blog.coolaj86.com/)的两篇博客：[Getting started with ruhoh (static blog) ](http://blog.coolaj86.com/articles/hosting-your-blog-on-ruhoh-com.html)和[Hosting your blog on GitHub Pages](http://blog.coolaj86.com/articles/hosting-your-blog-on-github-pages.html)。github pages支持Jekyll但不支持ruhoh，所以ruhoh的理念就是用git作为代码管理，而用github的[WebHooks](https://help.github.com/articles/post-receive-hooks)功能关联http://post.ruhoh.com ，这样就可以由ruhoh提供源代码的解析任务，ruhoh1.x就是这样的，具体方法参考[这里](http://ruhoh.com/docs/1/publish/)。但是目前http://post.ruhoh.com 暂时不支持ruhoh.2.0.alpha。所以就只能先编译好源代码，然后上传到github上实现博客的构建。这也是[Hosting your blog on GitHub Pages](http://blog.coolaj86.com/articles/hosting-your-blog-on-github-pages.html)这篇博客解决的问题。

首先，和ruhoh的官方要求一样，把源代码clone到本地，然后切换到2.0.alpha分支下，代码如下：
```
$ cd ~
$ git clone 'git://github.com/ruhoh/blog.git' 'blog-v2'
$ cd blog-v2
$ git checkout "2.0.alpha"
$ bundle install
```

注意这里，由于ruhoh的库里有三个分支（查看ruhoh的github库可以看到），分别是：master分支，1.0分支，2.0.alpha分支。其中，master下是 ruhohSpec v1.0；1.0分支下也是 ruhohSpec v1.0，但是从两个分支的readme中可以看出区别， master下的兼容 ruhoh gem 1.0.x.x，1.0下的兼容ruhoh gem v0.4.x。

ruhoh运行需要ruby1.9.2以上版本的支持，这里简单介绍一下如何安装ruby1.9.3,其实这个网上也有很多教程，我这里记录下来以防以后链接失效，我参考的是[在ubuntu中安装ruby 1.9.3](http://blog.sina.com.cn/s/blog_565e192a01013xjp.html)这篇博客，具体方法如下：

1. 安装 RVM：
```
$ curl -L get.rvm.io | bash -s stable
```
2. 将 rvm 加入PATH ：
```
$ source ~/.rvm/scripts/rvm
```
3. 查看需要安装的包：
```
$ rvm requirements
```
4. 安装上一步提示所需的包： 
```
$ sudo aptitude install build-essential openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison subversion
```
5. 安装 Ruby 1.9.3 ：
```
$ rvm install 1.9.3
```
6. 替换 gem 的 source ：
```
$ gem source -r http://rubygems.org/;gem source -a http://ruby.taobao.org
```
7. 安装 Bundler：
```
gem install bundler
```

这里需要注意第2步中，source加入环境变量只是对当前终端起作用，当终端关闭重启后，就失效了。所以需要在`.bashrc`中加入第2步的代码，即
```
$ gedit ~/.bashrc
```

加入
```
$ source ~/.rvm/scripts/rvm
```


下面就是介绍如何进行本地预览，这个在官方网站上也有，即：
```
$ bundle exec rackup -p 9292
```

然后就可以在<http://localhost:9292>上进行本地预览了，还可以先编译，编译好了到编译的文件夹下也可以本地预览，具体操作是：
```
$ bundle exec ruhoh compile
$ cd compiled
$ python -m SimpleHTTPServer 3000
```
 
然后在<http://localhost:3000>下进行本地预览。

然后就是把源代码上传到github中，首先在github上注册一个账号，然后新建一个以`username.github.io`为名字的库（repository），这里`username`就是你的账号名。为什么这样命名呢，这个也只能说是“王八的屁股——规定”，github pages上说对于个人或是组织的主页建立就是要这样命名，而且要master分支下。而对项目的主页要放到相应项目的gh-pages分支下，具体可以参考[这里](https://help.github.com/articles/user-organization-and-project-pages)。

正像前面所说，目前http://post.ruhoh.com 不支持2.0.alpha，所以只能通过先编译后上传才可以生成页面，这里先把源代码上传到那么这里上传`username.github.io`库的blog分支留作以后代码管理用，然后把编译好的文件上传到master分支用于生成页面。

1. 如果你没有编译的话,执行下面代码编译源代码到compile文件中
```
$ bundle exec ruhoh compile 
```

2. 忽略掉一些文件，.gitignore文件中存着忽略的文件名称，这里默认的应该已经包含compiled/*，保险起见，执行下面代码
```
$ echo "compiled/*" >> .gitignore
$ echo "*.lock" >> .gitignore
```

3. 新建一个分支blog，删除以前的origin，连接新的origin；把username换成自己的github账号名
```
$ git checkout -b blog
$ git remote rm origin
$ git remote add origin https://github.com/username/username.github.io.git
```
        
4. 添加到库中，并commit，并且push到库的blog分支下（push这一步不是必须的，只是用来存储源代码，目前post.ruhoh.com还不支持2.0.alpha），
```
$ git add .
$ git commit -m "first commit"
$ git push origin blog
```

5. 把编译好的文件上传到master分支下，正如前面所说，目前的master分支下存着ruhohSpec v1.0，所以需要先把这个分支删除或者重命名。这里采用重命名的方式：
```
$ git branch -m master masterold
$ git checkout blog
```

6. 建立新的分支master，删除所有的文件（.gitignoreg过的complie文件还在）
```
$ git checkout --orphan master
$ git rm -rf .
```

7. 忽略compiled文件，push到库
```
$ echo "compiled" >> .gitignore
$ git add .gitignore
$ git commit -m "ignore the compiled directory"
$ git push origin -u master
```

8. 更新你页面，以后你每次发博客，都要按此步骤，更新页面。
	```
#首先切换到blog分支
$ git checkout blog
#添加一些新的东西然后编译
$ git add .
$ git commmit -m "new post"
$ ruhoh compile
#切换到master分支
$ git checkout master
#同步compiled的文件到改分支下
$ rsync -a compiled/ ./
#删除compiled文件
$ rm -rf compiled/
$ git add .
$ git commit -m "updated compiled site"
$ git push origin master
	```

过大约十分钟，你就可以在https://username.github.io 下看到了。
以后每次更新页面只需要重复第8步就可以了，先切换到blog下添加新的post，然后编译，然后切换到master下进行推送即可。

如果想用自己的域名，首先，设置DNS服务器，具体设置可以参考[github pages help](https://help.github.com/articles/setting-up-a-custom-domain-with-pages)；设置好DNS后，只需要在master分支下建个文件CNAME，填入域名即可，具体命令如下，其中YOUR_DOMAIN是指你申请的域名。
```
$ git checkout master
$ echo "YOUR-DOMAIN" > CNAME
$ git add CNAME
$ git commit -m "hosting an empty blog on branch master"
$ git push origin -u master
$ git checkout blog
```

如果写博客的时候有公式的需要，可以把ruhoh的markdown解析器改为kramdown，ruhoh默认的解析器是Redcarpet，具体设置可以参考[ruhoh官网](http://ruhoh.com/docs/2/plugins/#toc_7)，这里简单介绍下，并说一下我遇到的问题和解决方法。

kramdown的设置是利用的ruhoh的plugins的功能，在plugins文件（如果没有，就建一个）构造如下文件结构
```
|-- plugins
    -- converters
       -- kramdown.rb
```

其中，kramdown.rb中输入一下内容：
    ```
class Ruhoh
    module Converter
	module Markdown
	   def self.extensions
		['.md', '.markdown']
	   end
	   def self.convert(content)
		require 'kramdown'
		Kramdown::Document.new(content).to_html
	   end
	end
    end
end
    ```

我当时按照这个步骤做了，但是出现了问题，提示我kramdown.rb中的第8行`require 'kramdown'`找不到kramdown，我当时是在本地预览出错的，后来我发现是问题出在Gemfile上，Gemfile是用来指定我用到的gem的，虽然在我已经安装了kramdown，但我的Gemfile中没有指定kramdown，所以会出错。在Gemfile加入下面这行代码即可修复此问题。
```
gem 'kramdown', "~> 1.0"
```

因为kramdown的公式渲染需要MathJax，所以也需要安装这个gem。而在服务器上使用需要在theme->twitter->layouts->default.html文件下加入一下代码：
```
<script type="text/javascript"
src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
```

剩下的一下设置基本在讲解Jekyll Bootstrap的那篇博客中有介绍，可以参考[万事开头难——我的博客折腾记](http://blog.skyoung.org/2013/04/21/it-is-the-first-step-that-is-troublesome/).包括google分析，comment设置等。

###总结
又一次折腾的结束，等到http://post.ruhoh.com 支持ruhoh2.0后，ruhoh的构建就简单了，以上只是权宜之计。
