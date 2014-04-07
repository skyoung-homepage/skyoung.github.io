---
title: "一个关于visual tracking的benchmark"
date: '2013-09-08'
categories: ['Computer Vision']
tags: [Online Visual Tracking, Benchmark]

---

自从工作后，就很少接触visual tracking这个领域了。最近下班没事，浏览博客看到几篇CVPR2013关于visual tracking的paper，其中有一篇名为**Online Object Tracking: A Benchmark**的paper对目前该领域几乎大部分优秀算法做了一个分析和评估，提出了更为严谨合理的评估方法。作者是University of California at Merced的一个博士后，以前好像是自动化所的一个博士。他针对29个算法，测试了50个视频，并且标记了所有视频的groundtruth，其耐心和毅力值得敬佩。

### 分析算法特性
当然，作者不是简单的罗列出所有的算法，而是针对目前的存在的算法，总结出四个模块：Representation Scheme，Search Mechanism，Model Update，Context and Fusion of Trackers。个人觉得前三个模块的确是visual tracking最重要的部分，Representation Scheme方面，算法的提出相对热闹些：基于图像原始像素值（如Lucas and Kanade稀疏点的光流法），直方图（如color histograms, histograms of oriented gradients (HOG)等），Harr-like特征，binary feature，基于sparse representation等等。Search Mechanism方面，可以用的方法相对就会比较单一。对于跟踪算法是基于最优化函数框架的算法，跟踪物体的位置通过local optimum search确定位置。而对于算法是基于binary classifier的，方法主要有两种：particle filter和dense sampling search。Model Update方面，对于binary classifier就是训练样本的更新。对于subspace-based tracking就是物体外观图像块的update。

<!--more-->

###评估方法
作者的主要贡献应该还是在评估方法上的创新，针对视频中出现的各种情况，对所有测试视频标上相应的属性标签，这些标签包括：光照变化，尺寸变化，遮挡，形变，运动模糊，快速运动，平面内的转动，平面外的转动，背景有相似目标，低分辨率。这样就可以看出算法对不同场景的处理能力。

目前最常用的精度评估方法就是求重叠率，即算法得出的结果方框（tracked bounding box）与真值（ground truth bounding box）重叠的面积除以两个方框的并集的面积，
$$S=\frac{R\_t\bigcap R\_g}{R\_t \bigcup R\_g}$$ 
但是大部分算法都是简单的求视频所有帧的重叠率的均值作为对算法的评估，而这篇paper的作者采用了更为合理的方法——AUC（Area Under Curve）。顾名思义，就是曲线下的面积大小，这里的曲线作者叫它success plot，其实就是[ROC](http://en.wikipedia.org/wiki/Receiver_operating_characteristic)（receiver operating characteristic），经常用来判定分类器性能好坏。曲线的横坐标是判定跟踪是否成功的重叠率阈值threshold，范围是[0,1]，纵坐标是对应的success rate，当然随着这个阈值threshold的增大，success rate逐渐减小，就形成了一个递减的曲线。这里如果这个曲线是向上凸，说明分类器的性能较好，如果向下凸，分类性能就会较差，反应到AUC上，就是向上凸，AUC变大，向下凸，AUC变小。所以AUC的大小正好反应了分类器性能的好坏。这种评估算法性能的方法能更综合的评估算法的鲁棒性。

我们知道，有些online visual tracking的算法对于初始化的方框是非常敏感的，如果初始化方框选的不同，会很大影响算法后续的跟踪效果。所以作者提出了两个策略来评估这个问题：

-  **TRE**（temporal robustness evaluation）是指把视频分成若干个段，分别跟踪，然后对这些段跟踪的结果求均值。
-   **SRE**（spatial robustness evaluation）是指初始方框的位置是不同的，基于目标位置向周围做微小的偏移（10%），以测试算法的鲁棒性。

除此之外，作者还做了针对不同尺寸的初始化方框的情况测试。

###总结
这篇paper对该领域的大部分算法作了较为详细分析和评估，对于该领域研究还是有一定指导意义的。具体的数据集，paper和代码可以参考作者的网站[Online Object Tracking: A Benchmark](http://visual-tracking.net/#)，好像得翻墙才能打开。



