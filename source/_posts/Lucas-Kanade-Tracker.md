---
title: 'Lucas-Kanade（LK）算法原理介绍及OpenCV代码实现分析'
date: '2014-04-01'
categories: [Computer Vision]
tags: [Tracking, OpenCV]
---

Lucas-Kanade跟踪算法是视觉跟踪中一个很经典的基于点的逐帧跟踪算法。起初这个算法是用来求解stero matching[^1]的，后来经过Carlo Tomasi[^2]和Jianbo Shi[^3]等人的发展渐趋成熟。Jianbo Shi提出了一种筛选跟踪点特征的方法，使得特征的跟踪更可靠。Jean-Yves Bouguet[^4]详细阐述了如何采用金字塔方式实现LK算法以处理两帧之间特征点的较大位移。

###问题阐述
首先我们来看一下我们要解决的问题是什么？LK算法是基于特征点的跟踪，而这里的特征点就是每个点对应的一个小窗口图像块，LK所要解决的是求解连续两帧图像相同特征点的位移问题。这里我们假设$I$和$J$为连续两帧图像，其$(x,y)$点的灰度值分别对应$I(x,y), J(x,y)$。设$\mathbf{u}=[u_x,u_y]^T$是图像$I$上一点，LK算法的目标是在图像$J$找到一点$\mathbf{v}= \mathbf{u} + \mathbf{d} = [u_x+d_x,u_y+d_y]^T$使得点$I(\mathbf{u})$和点$J(\mathbf{v})$是同一个位置。为了求解这样的点，LK求解这两个点对应的小窗口内像素的相似度。设$\omega_x$和$\omega_y$分别是点左右扩展的窗口范围，这样我们可以定义如下residual function：为

$$\epsilon(\mathbf{d})=\epsilon(d_x,d_y)=\sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{x=u_y-\omega_y}^{u_y+\omega_y}(I(x,y)-J(x+d_x,y+d_y))^2$$

窗口大小为$(2\omega_x+1)\times (2\omega_y+1)$,通常情况下$\omega_x$和$\omega_y$的值为2,3,4,5,6,7。

###标准LK算法
针对上述最优化问题，求解方法是求解$\epsilon(\mathbf{d})$关于向量$\mathbf{d}$的偏导使其等于0,即

$$\frac{\partial \epsilon (\mathbf{d})}{\partial \mathbf{d}}|_{\mathbf{d}=\mathbf{d}_{opt}}=\begin{bmatrix}
0 & 0
\end{bmatrix}$$

这样可以推到出其偏导结果：

$$
\begin{equation}
\begin{split} 
\frac{\partial \epsilon (\mathbf{d})}{\partial \mathbf{d}}&=-2\sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{x=u_y-\omega_y}^{u_y+\omega_y}(I(x,y)-J(x+d_x,y+d_y))\frac{\partial J(x+d_x,y+d_y)}{\partial \mathbf{d}} \\
&=-2\sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{x=u_y-\omega_y}^{u_y+\omega_y}(I(x,y)-J(x+d_x,y+d_y))\begin{bmatrix}\frac{\partial J}{\partial x} & \frac{\partial J}{\partial y}\end{bmatrix}
\end{split}
\end{equation}
$$

利用泰勒级数展开$J(x+d_x,y+d_y)$得,

$$
\begin{equation}
\begin{split} 
J(x+d_x,y+d_y) &= J(x,y)+ ( d_x\frac{\partial}{\partial x}+d_y\frac{\partial}{\partial y} )J(x,y)
+ \frac{1}{2!}( d_x\frac{\partial}{\partial x}+d_y\frac{\partial}{\partial y} )^2J(x,y)+... \\
&\approx J(x,y)+ ( d_x\frac{\partial}{\partial x}+d_y\frac{\partial}{\partial y} )J(x,y) \\
&\approx J(x,y)+ \begin{bmatrix}\frac{\partial J}{\partial x} & \frac{\partial J}{\partial y} \end{bmatrix}\mathbf{d}
\end{split}
\end{equation}
$$

这样得出residual function为，

$$
\begin{equation}
\begin{split} 
\frac{\partial \epsilon (\mathbf{d})}{\partial \mathbf{d}} \approx -2\sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{x=u_y-\omega_y}^{u_y+\omega_y}(I(x,y)-J(x,y) - \begin{bmatrix}\frac{\partial J}{\partial x} & \frac{\partial J}{\partial y} \end{bmatrix}\mathbf{d})\begin{bmatrix}\frac{\partial J}{\partial x} & \frac{\partial J}{\partial y}\end{bmatrix}
\end{split}
\end{equation}
$$

这里关于图像$J(x,y)$的偏导可以通过求解$I(x,y)$的偏导近似计算。设
$$\triangledown I=\begin{bmatrix}I_x\\ I_y\end{bmatrix}=\begin{bmatrix}\frac{\partial J}{\partial x} & \frac{\partial J}{\partial y}\end{bmatrix}^T$$, $$\delta I=I(x,y)-J(x,y)$$
这样residual function变为
$$
\begin{equation}
\begin{split} 
\frac{1}{2}\frac{\partial \epsilon (\mathbf{d})}{\partial \mathbf{d}} &\approx \sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{x=u_y-\omega_y}^{u_y+\omega_y}(\triangledown I^T\mathbf{d}-\delta I)\triangledown I^T \\
&\approx \sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{x=u_y-\omega_y}^{u_y+\omega_y}(\triangledown I^T\mathbf{d}\triangledown I^T-\delta I\triangledown I^T)
\end{split}
\end{equation}
$$

等式两边取倒置
$$
\begin{equation}
\begin{split} 
\frac{1}{2} \left [ \frac{\partial \epsilon (\mathbf{d})}{\partial \mathbf{d}} \right ]^T &\approx \sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{x=u_y-\omega_y}^{u_y+\omega_y}(\triangledown I\mathbf{d}^T \triangledown I-\delta I \triangledown I) \\
&\approx \sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{x=u_y-\omega_y}^{u_y+\omega_y}(\triangledown I \triangledown I^T \mathbf{d}-\delta I  \triangledown I) \\
&\approx \sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{x=u_y-\omega_y}^{u_y+\omega_y}(\begin{bmatrix}
I^2_x & I_{xy}\\ 
I_{xy} & I^2_y
\end{bmatrix} \mathbf{d}-\begin{bmatrix}
\delta II_x\\ 
\delta II_y
\end{bmatrix})
\end{split}
\end{equation}
$$

我们用简单符号替代其中的两个部分，分别设
$$
\begin{align}
G&=\sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{x=u_y-\omega_y}^{u_y+\omega_y}\begin{bmatrix}
I^2_x & I_{xy}\\ 
I_{xy} & I^2_y
\end{bmatrix} \\
\mathbf{b}&=\sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{x=u_y-\omega_y}^{u_y+\omega_y} \begin{bmatrix}
\delta II_x\\ 
\delta II_y
\end{bmatrix}
\end{align}
$$

现在residual function变成了，

$$
\frac{1}{2} \left [ \frac{\partial \epsilon (\mathbf{d})}{\partial \mathbf{d}} \right ]^T 
\approx \sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{x=u_y-\omega_y}^{u_y+\omega_y}(G \mathbf{d}-\mathbf{b})
$$

使上式等于0,得出位移$\mathbf{d}$为，

$$\mathbf{d}=G^{-1}\mathbf{b}$$

这里必须保证$G$是可逆的，也就是保证图像$I(x,y)$在$x$和$y$方向上的梯度值必须不是0。

以上便是基本的LK算法的推到过程，但是上述推到的一个基本假设是点特征的位移是很小的，这样才能满足泰勒展开式中只保留前两项近似操作。

###多分辨率匹配（图像金字塔）





[^1]: Bruce D. Lucas and Takeo Kanade. _**An Iterative Image Registration Technique with an Application to Stereo Vision**_. International Joint Conference on Artificial Intelligence, pages 674-679, 1981.
[^2]: Carlo Tomasi and Takeo Kanade. _**Detection and Tracking of Point Features**_. Carnegie Mellon University Technical Report CMU-CS-91-132, April 1991.
[^3]: Jianbo Shi and Carlo Tomasi. _**Good Features to Track**_. IEEE Conference on Computer Vision and Pattern Recognition, pages 593-600, 1994.
[^4]: Jean-Yves Bouguet. _**Pyramidal Implementation of the Lucas Kanade Feature Tracker**_.
