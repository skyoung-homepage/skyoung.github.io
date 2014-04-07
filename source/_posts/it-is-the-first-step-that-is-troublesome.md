---
title: 万事开头难——我的博客折腾记
date: '2013-04-21'
categories: "How to" 
tags: [Jekyll, git]
---

####起因
为什么要整个博客呢？其实这也不是我一时兴起，很久以前就想过搞个自己的个人主页，只是因为没有足够的时间而作罢了。现在临近毕业，论文基本搞定，算是终于有时间折腾一下了。感觉博客这东西既可以督促自己及时总结自己在学习心得，又可以与别人分享交流，所以整整还是不错的。好了，闲话少说，切入正题。

####过程
在讲如何构建博客之前，我需要先说明一下博客运行的原理，首先我们需要一个叫做[虚拟主机](http://baike.baidu.com/view/7383.htm)的东西，简单点说就是网络上提供一个虚拟的服务器来支撑网站的运行，提供一些基本的功能。这个东西可以在[github](https://github.com/)上申请个帐号就可以了，它提供这样一个功能。github要求构建个人主页要建一个和自己帐号名字相同的repository，且在master分支下，即如果你的帐号名是example，repository的名字就得是example.github.io；而对于项目主页则要建立一个gh-pages的分支，具体一些其他方面的细节问题可以参见[github help pages](https://help.github.com/categories/20/articles)。然后就是如何构建自己的博客页面信息等内容，这个东西就用到了[Jekyll](http://jekyllrb.com/) ,github支持Jekyll构建网页，所以我们需要做的就是用它构建好自己的博客，然后把博客上传的github上就可以了。原始的构建Jekyll还是比较麻烦的，所以就有了一些基于Jekyll的构建博客框架，比较流行的就是[Jekyll Bootstrap](http://jekyllbootstrap.com/),从这个名字可以看出，这个东东和[Bootstrap](http://twitter.github.io/bootstrap/index.html)肯定有关系，[Bootstrap](http://twitter.github.io/bootstrap/index.html)也是一个构建网页的东西，提供了许多简单易用的功能，降低了构建网页的难度，是个很炫的东西。除此之外还有[Octopress](http://octopress.org/)也是基于Jekyll的一个博客构建框架，这个我没有具体玩过，所以不好多说。[ruhoh](http://ruhoh.com/)是[Jekyll Bootstrap](http://jekyllbootstrap.com/)作者搞的另一构建博客的东西，本来想着构建自己博客的时候用这东西的，但是不知怎么的试了几次都不成功，各种不工作，最后放弃了，还是用了[Jekyll Bootstrap](http://jekyllbootstrap.com/)。

<!--more-->

博客构建好了后，还需要别人能搜到你的网站，这样才能有读者啊，或者专业一点叫推广。这里其中一条是让别人能在搜索引擎中搜到你的网站信息，所以要设置一个叫做[google网站站长](http://www.google.cn/intl/zh-CN_cn/webmasters/)的东西,通过设置这个东西，就可以让google抓取到你的网页，这个只需要在jekyll bootstrap的_include->theme->twitter下找到default.html加入元标记就可以了，当然首先要用自己的google帐号登陆google网站站长，按照提示来就可以。还有就是设置一个叫[google分析](http://www.google.cn/intl/zh-CN/analytics/)，用来监测你的网站的访问情况，设置也比较简单，首先也是加入一些代码到default.html文件中，按照google分析的要求一步一步的做就可以，然后就是在 _config.yml中设置一下analytics下的google的tracking-id如下：
```
	analytics :
	    provider : google 
	    google : 
	    tracking_id : 'your id'     其中``your id``在google分析的账户中有提供。
```
博客的撰写是一种轻量级标记语言markdown，易于书写，而且还有一些它的扩展语言也是支持的，像[kramdown](http://kramdown.rubyforge.org/index.html),它可以支持Latex公式输入，只需要在用两个$$括起来就可以撰写Latex，但是公式的渲染需要[MathJax](http://www.mathjax.org/)的支持，利用MathJax的服务器可以实现Jekyll对kramdown的支持，只需要在default.html中加入如下代码
```
	<script type="text/javascript"
	src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
	</script>
```
然后设置_config.yml文件如下：
```
	markdown: kramdown
```
同时Jekyll也支持[textile](http://redcloth.org/textile),比markdown语言实现的功能相对多一点，这个用什么写看个人习惯，我是用kramdown，考虑到其公式的渲染能力。

有了以上的功能，基本上就可以安心写博客了，当然如果你对博客的主题还有更高的要求，可以参考[Bootstrap](http://twitter.github.io/bootstrap/index.html),调整一下自己的主题，让其符合自己的要求，或者干脆直接写个主题，Jekyll Bootstrap也支持调用自己写的主题，主题主要需要撰写的是三个布局网页default.html，post.html，page.html，更多的个人博客模板参考[这里](https://github.com/mojombo/jekyll/wiki/Sites)。如果你对JB架构的某些地方不满意，也可以重写JB的API，然后放在_include的custom文件下，然后把include JB/the-file-you-need改为include custom/the-file-you-need,同时设置 _config.yml文件下的
```
	JB :
	    tags_list :
		provider : "custom"

```
最后，说一下域名申请。从github中构建的个人网页的网址形式是：``http://username.github.io``,其中``username``是你的账户名。如果你对这个域名不满意，可以注册一个自己喜欢的域名，去相关的域名注册网站很容易就可以购买到域名，一般``.com``的域名相对难购买一点，``org``的还是蛮容易的。一般的域名服务商也会提供域名的解析服务，如果不提供，可以去免费的域名解析服务器申请个账户，像[DNSPod](https://www.dnspod.cn/)。这样申请完域名，就可以把这个域名解析到自己的博客网址上了。具体方法参见[这里](https://help.github.com/articles/setting-up-a-custom-domain-with-pages)

####总结
折腾了好几天的博客终于搞定了，中间也是遇到了很多问题，从完全不懂，到逐渐明白一些东西，感觉还是蛮不错的。虽然目前博客还比较简陋，但基本符合我的要求了，当然以后最重要还是博客的内容。所谓万事开头难，现在头也开的差不多了，剩下的就是持久力了。


