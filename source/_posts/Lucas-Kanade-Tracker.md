---
title: 'Lucas-Kanade（LK）算法原理介绍及OpenCV代码实现分析'
date: '2014-04-13'
categories: [Computer Vision]
tags: [Tracking, OpenCV]
toc: true
---

Lucas-Kanade跟踪算法是视觉跟踪中一个很经典的基于点的逐帧跟踪算法。起初这个算法是用来求解stero matching[^1]的，后来经过Carlo Tomasi[^2]和Jianbo Shi[^3]等人的发展渐趋成熟。Jianbo Shi提出了一种筛选跟踪点特征的方法，使得特征的跟踪更可靠。Jean-Yves Bouguet[^4]详细阐述了如何采用金字塔方式实现LK算法以处理两帧之间特征点位移较大的情况。

###问题阐述
首先我们来看一下我们要解决的问题是什么？LK算法是基于特征点的跟踪，而这里的特征点就是每个点对应的一个小窗口图像块，LK所要解决的是求解连续两帧图像相同特征点的位移问题。这里我们假设$I$和$J$为连续两帧图像，其$(x,y)$点的灰度值分别对应$I(x,y), J(x,y)$。设$\mathbf{u}=[u_x,u_y]^T$是图像$I$上一点，LK算法的目标是在图像$J$找到一点$\mathbf{v}= \mathbf{u} + \mathbf{d} = [u_x+d_x,u_y+d_y]^T$使得点$I(\mathbf{u})$和点$J(\mathbf{v})$是同一个位置。为了求解这样的点，LK求解这两个点对应的小窗口内像素的相似度。设$\omega_x$和$\omega_y$分别是点左右扩展的窗口范围，这样我们可以定义如下residual function为

$$\epsilon(\mathbf{d})=\epsilon(d_x,d_y)=\sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{y=u_y-\omega_y}^{u_y+\omega_y}(I(x,y)-J(x+d_x,y+d_y))^2$$

窗口大小为$(2\omega_x+1)\times (2\omega_y+1)$,通常情况下$\omega_x$和$\omega_y$的值为2,3,4,5,6,7。

<!--more-->

###标准LK算法
针对上述最优化问题，求解方法是求解$\epsilon(\mathbf{d})$关于向量$\mathbf{d}$的偏导使其等于0,即

$$\frac{\partial \epsilon (\mathbf{d})}{\partial \mathbf{d}}|_{\mathbf{d}=\mathbf{d}_{opt}}=\begin{bmatrix}
0 & 0
\end{bmatrix}$$

这样可以推到出其偏导结果：

$$
\begin{equation}
\begin{split} 
\frac{\partial \epsilon (\mathbf{d})}{\partial \mathbf{d}}&=-2\sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{y=u_y-\omega_y}^{u_y+\omega_y}(I(x,y)-J(x+d_x,y+d_y))\frac{\partial J(x+d_x,y+d_y)}{\partial \mathbf{d}} \\
&=-2\sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{y=u_y-\omega_y}^{u_y+\omega_y}(I(x,y)-J(x+d_x,y+d_y))\begin{bmatrix}\frac{\partial J}{\partial x} & \frac{\partial J}{\partial y}\end{bmatrix}
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
\frac{\partial \epsilon (\mathbf{d})}{\partial \mathbf{d}} \approx -2\sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{y=u_y-\omega_y}^{u_y+\omega_y}(I(x,y)-J(x,y) - \begin{bmatrix}\frac{\partial J}{\partial x} & \frac{\partial J}{\partial y} \end{bmatrix}\mathbf{d})\begin{bmatrix}\frac{\partial J}{\partial x} & \frac{\partial J}{\partial y}\end{bmatrix}
\end{split}
\end{equation}
$$

这里关于图像$J(x,y)$的偏导可以通过求解$I(x,y)$的偏导近似计算。设
$$\triangledown I=\begin{bmatrix}I_x\\ I_y\end{bmatrix}=\begin{bmatrix}\frac{\partial J}{\partial x} & \frac{\partial J}{\partial y}\end{bmatrix}^T$$, $$\delta I=I(x,y)-J(x,y)$$
这样residual function变为
$$
\begin{equation}
\begin{split} 
\frac{1}{2}\frac{\partial \epsilon (\mathbf{d})}{\partial \mathbf{d}} &\approx \sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{y=u_y-\omega_y}^{u_y+\omega_y}(\triangledown I^T\mathbf{d}-\delta I)\triangledown I^T \\
&\approx \sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{y=u_y-\omega_y}^{u_y+\omega_y}(\triangledown I^T\mathbf{d}\triangledown I^T-\delta I\triangledown I^T)
\end{split}
\end{equation}
$$

等式两边取倒置
$$
\begin{equation}
\begin{split} 
\frac{1}{2} \left [ \frac{\partial \epsilon (\mathbf{d})}{\partial \mathbf{d}} \right ]^T &\approx \sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{y=u_y-\omega_y}^{u_y+\omega_y}(\triangledown I\mathbf{d}^T \triangledown I-\delta I \triangledown I) \\
&\approx \sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{y=u_y-\omega_y}^{u_y+\omega_y}(\triangledown I \triangledown I^T \mathbf{d}-\delta I  \triangledown I) \\
&\approx \sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{y=u_y-\omega_y}^{u_y+\omega_y}(\begin{bmatrix}
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
G&=\sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{y=u_y-\omega_y}^{u_y+\omega_y}\begin{bmatrix}
I^2_x & I_{xy}\\ 
I_{xy} & I^2_y
\end{bmatrix} \\
\mathbf{b}&=\sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{y=u_y-\omega_y}^{u_y+\omega_y} \begin{bmatrix}
\delta II_x\\ 
\delta II_y
\end{bmatrix}
\end{align}
$$

现在residual function变成了，

$$
\frac{1}{2} \left [ \frac{\partial \epsilon (\mathbf{d})}{\partial \mathbf{d}} \right ]^T 
\approx \sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{y=u_y-\omega_y}^{u_y+\omega_y}(G \mathbf{d}-\mathbf{b})
$$

使上式等于0,得出位移$\mathbf{d}$为，

$$\mathbf{d}=G^{-1}\mathbf{b}$$

这里必须保证$G$是可逆的，也就是保证图像$I(x,y)$在$x$和$y$方向上的梯度值必须不是0。

以上便是基本的LK算法的推导过程，具体实现的时候需要多次的迭代才能得到一个较准确的点的位移矢量，类似[牛顿-拉弗森方法](http://zh.wikipedia.org/wiki/%E7%89%9B%E9%A1%BF%E6%B3%95)（Newton-Raphson method）的迭代过程，这是一个逐渐趋近最优值的过程。下面详细介绍迭代的过程，针对第$k(k\geqslant 1)$次迭代：

设第$k-1$次迭代的位移$\mathbf{d}^{k-1}=[d_x^{k-1},d_y^{k-1}]$，则我们利用第$k-1$次迭代的位移作为第$k$次迭代位移的初始化值，即当前次迭代的$J(x,y)$变为

$$J(x,y) = J(x+d_x^{k-1},y+d_y^{k-1})$$

residual function变为
$$\epsilon(\mathbf{d})=\epsilon(d_x,d_y)=\sum_{x=u_x-\omega_x}^{u_x+\omega_x}\sum_{y=u_y-\omega_y}^{u_y+\omega_y}(I(x,y)-J(x+d_x^k,y+d_y^k))^2$$

通过一次标准的LK算法，得出第$k$次的位移

$$\mathbf{d}^k = G^{-1}\mathbf{b}_k$$

这里我们发现每次迭代中，G是不变的，通过$I(x,y)$计算，唯一变化的是$\mathbf{b}$，每次迭代图像$J(x,y)$对应的窗口都会向所要求的位置点靠近一点点（即上一次迭代的位移作为初始化），而$\mathbf{b}$的计算与$J(x,y)$有关，所以每次迭代都会发生变化，这样每次迭代需要计算的就只有$\mathbf{b}$。

假设进行了K次迭代后收敛，最终位移的结果为
$$\mathbf{d} = \sum_{k=1}^{K}\mathbf{d}^k$$

对于第一次迭代其对应的初始化位移为：
$$\mathbf{d}^0 = \begin{bmatrix}0 & 0\end{bmatrix}^T$$


但是上述推导的一个基本假设是点特征的位移是很小的，这样才能满足泰勒展开式中只保留前两项的近似操作。而为了能处理较大的位移情况，则需要基于图像金字塔在不同分辨率的图层下进行跟踪。

###图像金字塔跟踪

首先举一个简单的例子，比如知道一个点前后两幅图像的位移为16个像素，这么大的位移直接使用标准LK算法是很难计算出来结果的，而如果在图像分辨率降低到原来一半后，其位移就变为8个像素，再降低一半，则为4个像素，如果金字塔的层数是3,则在最底层，点的位移只有两个像素，这样就满足了小位移的假设。这样首先在最底层进行标准LK算法，得出一个位移后乘以2作为上一层的初始位移，再进行标准LK算法，以此类推，最终得到点的位移。

设图像金字塔层数为$L =0,1,2...L_m$,跟踪是从图像金字塔的最底层$L_m$开始的，对于图像从第$L+1$层到$L$层的跟踪流程，和标准LK算法的迭代有点类似，第$L$层的初始化位置是基于第$L+1$层计算出来的。

设$\mathbf{g}^L=\begin{bmatrix}g_x^L & g_y^L\end{bmatrix}^T$是第$L$层的初始化位移，它是通过第$L+1$层的位移计算得到的。这样第$L$层的residual function就变成了

$$\epsilon^L(\mathbf{d}^L)=\epsilon(d_x^L,d_y^L)=\sum_{x=u_x^L-\omega_x}^{u_x^L+\omega_x}\sum_{x=u_y^L-\omega_y}^{u_y^L+\omega_y}(I^L(x,y)-J^L(x+g_x^L+d_x^L,y+g_y^L+d_y^L))^2$$

从上式可以看出，第$L$层的$J^L(x,y)$由于有了$\mathbf{g}^L$作为初始化位置使得要求解的位移$\mathbf{d}^L$变得很小，也就很适合用标准的LK算法计算了。

$\mathbf{g}^L$的计算是通过第$L+1$层的位移和初始化位置计算的，

$$\mathbf{g}^L = 2(\mathbf{g}^{L+1} + \mathbf{d}^{L+1} )$$

对于最底层的初始化位置设为，
$$\mathbf{g}^{L_m} = \begin{bmatrix}0 & 0 \end{bmatrix}^T$$

最后得出点的位移为，
$$\mathbf{d}^0 = \mathbf{g}^0 + \mathbf{d}^0$$

图像金字塔的构建是通过首先对上一层图像进行去边缘滤波，然后下采样得到的，具体的实现参考下面[章节](#图像金字塔的构建)。

###OpenCV代码实现分析

这个算法的实现主要分为三个重要的部分：图像金字塔的构建，图像梯度图的计算，标准LK算法的迭代。

####图像金字塔的构建
OpenCV是通过下采样和去边缘滤波器两个流程生成的多分辨率的图像金字塔，当然为了程序的优化，这两个流程是同时进行的。滤波器采用的kernel是
$$\frac{1}{256}\begin{bmatrix}
1 & 4 & 6 & 4 & 1\\ 
4 & 16 & 24 & 16 &4 \\ 
6 &  24& 36 & 24 & 6\\ 
4 & 16 & 24 & 16 & 4\\ 
1 & 4 & 6 & 4 & 1
\end{bmatrix}
$$

程序实现的核心函数是 

```
void pyrDown(InputArray src, OutputArray dst, const Size& dstsize=Size(), int borderType=BORDER_DEFAULT )
```
其OpenCV的源代码如下（OpenCV-2.4.8/video/pyramids.cpp/line:187）。这个函数实现的大体思路是以**目标图像**的长宽为基准同时实现对**源图像**的去边缘滤波以及下采样操作。因为图像在滤波后还要做下采样，如果这两步骤是分开做的话，前面滤波到的像素就会额外计算了一半下采样后根本不需要的像素，浪费了计算，所以这里仅仅是滤波了下采样中保留下来的像素。

滤波器的实现最直接的想法就是每次计算所有窗口里面的像素值然后求均值，但这样做会重复计算前后两行重叠的部分像素和，代码效率并不高，实际上代码中的实现是首先计算每一个像素对应的前后两行的行和，然后存储下这5个行和，每行所有像素计算完所有5个行和并存储好后，再用一个for循环求和每个像素的5个行和，这样就可以避免重复计算前后两行重复的行和而提高了效率。如下图所示，中间红色像素对应窗口为1-5行，蓝色像素对应窗口为2-6行，其中其中2-5行的和都是重复的，不需要重复计算。

{% img /media/Computer-Vision/pyrDown.png "pyrDown实现示意图" "pyrDown实现示意图"%}

这里有几个需要解释的地方：

第一，代码第54行的for循环实现的是求解每个元素对应的各个行和。这里它采用了一种循环存取机制。例如，当计算目标图像第k行像素（即源图像第2k行像素）时，其需要求解的行和分别是对应源图像上的2k-2，2k-1，2k，2k+1，2k+2行，这时假设内存中是按照顺序方式存储的，当计算目标图像k+1行像素（即源图像第2k+2行像素）时，我们只需要再计算2k+3和2k+4的**行和**并且存储在本来存储2k-2和2k-1行的**行和**的内存中，这样的计算和存储开销是最小的。这样说可能有点抽象，如下图所示，左边是计算目标图像第k行像素时，数组中5个元素存储的内容，右边是计算目标图像k+1行像素存储的内容，仅仅是把原来存储2k-2和2k-1行的元素替换成新计算出来的2k+3和2k+4行的行和，这样在访问这些行和时，顺序就会发生一定的变化，由左边的1,2,3,4,5变成右边的4,5,1,2,3。这样在计算k+2行像素时，只需把原来存储2k-2和2k-1的内存替换为2k+3和2k+4的行和即可，然后依次类推。查看程序第57行`WT* row = buf + ((sy - sy0) % PD_SZ)*bufstep;`，采用的就是这种循环存储方法。那这样在取这些存在数组中的行和是，顺序也是对应的顺序，因为每个行和要乘以的权重不一，自然顺序不能错。顺序的计算方法查看程序第122行` rows[k] = buf + ((y*2 - PD_SZ/2 + k - sy0) % PD_SZ)*bufstep;`

{% img /media/Computer-Vision/order.png "存储示意图" "存储示意图"%}


第二，程序在处理边缘问题是采用了`borderInterpolate`这个函数，主要涉及对称边缘图像或直接复制边缘图像等方式添加虚拟边缘。当`boderType = BORDER_REPLICATE`时，是简单的复制边缘图像，当`boderType = BORDER_REFLECT`时，就是以边缘为中心对称复制边缘里层的图像。而pyrDown采用的就是这种边缘处理方式。


```cpp
template<class CastOp, class VecOp> void
pyrDown_( const Mat& _src, Mat& _dst, int borderType )
{
    //滤波器的窗口大小
    const int PD_SZ = 5;
    typedef typename CastOp::type1 WT;
    typedef typename CastOp::rtype T;

    CV_Assert( !_src.empty() );
    Size ssize = _src.size(), dsize = _dst.size();
    int cn = _src.channels();
    int bufstep = (int)alignSize(dsize.width*cn, 16);
    AutoBuffer<WT> _buf(bufstep*PD_SZ + 16);
    WT* buf = alignPtr((WT*)_buf, 16);
    //处理左右边界
    int tabL[CV_CN_MAX*(PD_SZ+2)], tabR[CV_CN_MAX*(PD_SZ+2)];
    AutoBuffer<int> _tabM(dsize.width*cn);
    int* tabM = _tabM;
    //存储PD_SZ行的和，用于第二次循环列的求和
    WT* rows[PD_SZ];
    CastOp castOp;
    VecOp vecOp;

    CV_Assert( std::abs(dsize.width*2 - ssize.width) <= 2 &&
               std::abs(dsize.height*2 - ssize.height) <= 2 );
    int k, x, sy0 = -PD_SZ/2, sy = sy0, width0 = std::min((ssize.width-PD_SZ/2-1)/2 + 1, dsize.width);

    for( x = 0; x <= PD_SZ+1; x++ )
    {
        int sx0 = borderInterpolate(x - PD_SZ/2, ssize.width, borderType)*cn;
        int sx1 = borderInterpolate(x + width0*2 - PD_SZ/2, ssize.width, borderType)*cn;
        for( k = 0; k < cn; k++ )
        {
            tabL[x*cn + k] = sx0 + k;
            tabR[x*cn + k] = sx1 + k;
        }
    }

    ssize.width *= cn;
    dsize.width *= cn;
    width0 *= cn;

    for( x = 0; x < dsize.width; x++ )
        tabM[x] = (x/cn)*2*cn + x % cn;
    
    //基于目标图像的高度
    for( int y = 0; y < dsize.height; y++ )
    {
        T* dst = (T*)(_dst.data + _dst.step*y);
        WT *row0, *row1, *row2, *row3, *row4;

        // fill the ring buffer (horizontal convolution and decimation)
        //水平方向求解各个行和
        for( ; sy <= y*2 + 2; sy++ )
        {
            //循环存储行和
            WT* row = buf + ((sy - sy0) % PD_SZ)*bufstep;
            int _sy = borderInterpolate(sy, ssize.height, borderType);
            const T* src = (const T*)(_src.data + _src.step*_sy);
            int limit = cn;
            const int* tab = tabL;

            for( x = 0;;)
            {
                for( ; x < limit; x++ )
                {
                    row[x] = src[tab[x+cn*2]]*6 + (src[tab[x+cn]] + src[tab[x+cn*3]])*4 +
                        src[tab[x]] + src[tab[x+cn*4]];
                }

                if( x == dsize.width )
                    break;

                if( cn == 1 )
                {
                    for( ; x < width0; x++ )
                        row[x] = src[x*2]*6 + (src[x*2 - 1] + src[x*2 + 1])*4 +
                            src[x*2 - 2] + src[x*2 + 2];
                }
                else if( cn == 3 )
                {
                    for( ; x < width0; x += 3 )
                    {
                        const T* s = src + x*2;
                        WT t0 = s[0]*6 + (s[-3] + s[3])*4 + s[-6] + s[6];
                        WT t1 = s[1]*6 + (s[-2] + s[4])*4 + s[-5] + s[7];
                        WT t2 = s[2]*6 + (s[-1] + s[5])*4 + s[-4] + s[8];
                        row[x] = t0; row[x+1] = t1; row[x+2] = t2;
                    }
                }
                else if( cn == 4 )
                {
                    for( ; x < width0; x += 4 )
                    {
                        const T* s = src + x*2;
                        WT t0 = s[0]*6 + (s[-4] + s[4])*4 + s[-8] + s[8];
                        WT t1 = s[1]*6 + (s[-3] + s[5])*4 + s[-7] + s[9];
                        row[x] = t0; row[x+1] = t1;
                        t0 = s[2]*6 + (s[-2] + s[6])*4 + s[-6] + s[10];
                        t1 = s[3]*6 + (s[-1] + s[7])*4 + s[-5] + s[11];
                        row[x+2] = t0; row[x+3] = t1;
                    }
                }
                else
                {
                    for( ; x < width0; x++ )
                    {
                        int sx = tabM[x];
                        row[x] = src[sx]*6 + (src[sx - cn] + src[sx + cn])*4 +
                            src[sx - cn*2] + src[sx + cn*2];
                    }
                }

                limit = dsize.width;
                tab = tabR - x;
            }
        }

        // do vertical convolution and decimation and write the result to the destination image
        //循环取得各个行和的指针
        for( k = 0; k < PD_SZ; k++ )
            rows[k] = buf + ((y*2 - PD_SZ/2 + k - sy0) % PD_SZ)*bufstep;
        row0 = rows[0]; row1 = rows[1]; row2 = rows[2]; row3 = rows[3]; row4 = rows[4];

        x = vecOp(rows, dst, (int)_dst.step, dsize.width);
        //每一行结束的位置，把该行所有像素点的滤波结果求解出来
        for( ; x < dsize.width; x++ )
            dst[x] = castOp(row2[x]*6 + (row1[x] + row3[x])*4 + row0[x] + row4[x]);
    }
}
```


####图像梯度图的计算

图像梯度的实现也要设计到窗口计算问题，所以和上面提到的方法有类似的地方，也是先计算行和，再计算列和。程序采用了Sharr算子进行梯度的计算，x方向的梯度图算子是
$$
\begin{bmatrix}
-3 & 0 & 3 \\ 
-10 & 0 & 10 \\ 
-3 & 0 & 3
\end{bmatrix}
$$
y方向的梯度算子是
$$
\begin{bmatrix}
-3 & -10 & -3 \\ 
0 & 0 & 0 \\ 
3 & 10 & 3
\end{bmatrix}
$$

OpenCV详细代码如下，注意这里程序计算出的x和y方向的梯度值分别存放在了一个双通道的目标图像中，每个通道占用一个方向的梯度值。

```
static void calcSharrDeriv(const cv::Mat& src, cv::Mat& dst)
{
    using namespace cv;
    using cv::detail::deriv_type;
    int rows = src.rows, cols = src.cols, cn = src.channels(), colsn = cols*cn, depth = src.depth();
    CV_Assert(depth == CV_8U);
    dst.create(rows, cols, CV_MAKETYPE(DataType<deriv_type>::depth, cn*2));

#ifdef HAVE_TEGRA_OPTIMIZATION
    if (tegra::calcSharrDeriv(src, dst))
        return;
#endif

    int x, y, delta = (int)alignSize((cols + 2)*cn, 16);
    AutoBuffer<deriv_type> _tempBuf(delta*2 + 64);
    deriv_type *trow0 = alignPtr(_tempBuf + cn, 16), *trow1 = alignPtr(trow0 + delta, 16);

#if CV_SSE2
    __m128i z = _mm_setzero_si128(), c3 = _mm_set1_epi16(3), c10 = _mm_set1_epi16(10);
#endif

    for( y = 0; y < rows; y++ )
    {
        const uchar* srow0 = src.ptr<uchar>(y > 0 ? y-1 : rows > 1 ? 1 : 0);
        const uchar* srow1 = src.ptr<uchar>(y);
        const uchar* srow2 = src.ptr<uchar>(y < rows-1 ? y+1 : rows > 1 ? rows-2 : 0);
        deriv_type* drow = dst.ptr<deriv_type>(y);

        // do vertical convolution
        x = 0;
#if CV_SSE2
        for( ; x <= colsn - 8; x += 8 )
        {
            __m128i s0 = _mm_unpacklo_epi8(_mm_loadl_epi64((const __m128i*)(srow0 + x)), z);
            __m128i s1 = _mm_unpacklo_epi8(_mm_loadl_epi64((const __m128i*)(srow1 + x)), z);
            __m128i s2 = _mm_unpacklo_epi8(_mm_loadl_epi64((const __m128i*)(srow2 + x)), z);
            __m128i t0 = _mm_add_epi16(_mm_mullo_epi16(_mm_add_epi16(s0, s2), c3), _mm_mullo_epi16(s1, c10));
            __m128i t1 = _mm_sub_epi16(s2, s0);
            _mm_store_si128((__m128i*)(trow0 + x), t0);
            _mm_store_si128((__m128i*)(trow1 + x), t1);
        }
#endif
        for( ; x < colsn; x++ )
        {
            int t0 = (srow0[x] + srow2[x])*3 + srow1[x]*10;
            int t1 = srow2[x] - srow0[x];
            trow0[x] = (deriv_type)t0;
            trow1[x] = (deriv_type)t1;
        }

        // make border
        int x0 = (cols > 1 ? 1 : 0)*cn, x1 = (cols > 1 ? cols-2 : 0)*cn;
        for( int k = 0; k < cn; k++ )
        {
            trow0[-cn + k] = trow0[x0 + k]; trow0[colsn + k] = trow0[x1 + k];
            trow1[-cn + k] = trow1[x0 + k]; trow1[colsn + k] = trow1[x1 + k];
        }

        // do horizontal convolution, interleave the results and store them to dst
        x = 0;
#if CV_SSE2
        for( ; x <= colsn - 8; x += 8 )
        {
            __m128i s0 = _mm_loadu_si128((const __m128i*)(trow0 + x - cn));
            __m128i s1 = _mm_loadu_si128((const __m128i*)(trow0 + x + cn));
            __m128i s2 = _mm_loadu_si128((const __m128i*)(trow1 + x - cn));
            __m128i s3 = _mm_load_si128((const __m128i*)(trow1 + x));
            __m128i s4 = _mm_loadu_si128((const __m128i*)(trow1 + x + cn));

            __m128i t0 = _mm_sub_epi16(s1, s0);
            __m128i t1 = _mm_add_epi16(_mm_mullo_epi16(_mm_add_epi16(s2, s4), c3), _mm_mullo_epi16(s3, c10));
            __m128i t2 = _mm_unpacklo_epi16(t0, t1);
            t0 = _mm_unpackhi_epi16(t0, t1);
            // this can probably be replaced with aligned stores if we aligned dst properly.
            _mm_storeu_si128((__m128i*)(drow + x*2), t2);
            _mm_storeu_si128((__m128i*)(drow + x*2 + 8), t0);
        }
#endif
        for( ; x < colsn; x++ )
        {
            deriv_type t0 = (deriv_type)(trow0[x+cn] - trow0[x-cn]);
            deriv_type t1 = (deriv_type)((trow1[x+cn] + trow1[x-cn])*3 + trow1[x]*10);
            drow[x*2] = t0; drow[x*2+1] = t1;
        }
    }
}
```
####标准LK算法的迭代

这个部分的实现需要注意的地方主要是subpixel的计算，因为每次计算出的位移都很小，考虑到计算的精度，必须的精确到小数位，所以需要注意如何计算一个小数位置的像素值，这个就和线性插值是类似的。如下图所示，小数位置的像素值是有四个相邻像素拟合出来的。设中间蓝色像素点的坐标为$(x_{sub},y_{sub})$，四周四个整数位置的像素点自分别为$(x_0,y_0),(x_0,y_1),(x_1,y_0),(x_1,y_1)$，中间蓝色像素离其他四个像素水平和垂直方向上的像素距离分别为$w_{00},w_{01},w_{10},w_{11}$,如图中所标。

{% img /media/Computer-Vision/subpixel.png 300 300 "subpixel的计算" "subpxiel的计算"%}

subpxiel的计算公式为
$$
I(x_{sub},y_{sub})=w_{11}w_{01}I(x_0,y_0)+w_{10}w_{01}I(x_0,y_1)+w_{11}w_{00}I(x_1,y_0)+w_{10}w_{00}I(x_1,y_1)
$$

在计算权重$w_{i,j}$的时候，为了避免浮点计算，会对$w_{i,j}$乘以一定的倍数使用整数运算。用的subpixel的地方主要有两个地方，一个是在计算矩阵$G$的时候，需要取梯度图的值，但是点的位置不一定是整数，所以一定要使用subpixel取值；还有一个地方是计算$b$的时候，因为要取前后两幅图像的像素值，而这两个点的位置也不一定是整数，所以也要用到subpixel。考虑到这部分的OpenCV代码比较长，而理解相对没有那么困难，这里就不再贴出，仅列出上面需要注意的地方，需要的可以参考`oepncv2.4.8/video/lkpramid.cpp/line:159-483`

###尾声
LK算法的实现除了以上所讲的OpenCV的实现外，还有几个其他的版本，分别是由Stan Birchfield实现的版本[KLT](http://www.ces.clemson.edu/~stb/klt/)，速率相比OpenCV慢一些；一个GPU加速实现的版本[GPU KLT](http://cs.unc.edu/~ssinha/Research/GPU_KLT/)；一个Matlab实现的版本[Matlab KLT](http://www.ri.cmu.edu/research_project_detail.html?project_id=515&menu_id=261)以及一个Java实现的版本[Java KLT](http://boofcv.org/index.php?title=Applet_Feature_Tracking)。

[^1]: Bruce D. Lucas and Takeo Kanade. _**An Iterative Image Registration Technique with an Application to Stereo Vision**_. International Joint Conference on Artificial Intelligence, pages 674-679, 1981.
[^2]: Carlo Tomasi and Takeo Kanade. _**Detection and Tracking of Point Features**_. Carnegie Mellon University Technical Report CMU-CS-91-132, April 1991.
[^3]: Jianbo Shi and Carlo Tomasi. _**Good Features to Track**_. IEEE Conference on Computer Vision and Pattern Recognition, pages 593-600, 1994.
[^4]: Jean-Yves Bouguet. _**Pyramidal Implementation of the Lucas Kanade Feature Tracker**_.
