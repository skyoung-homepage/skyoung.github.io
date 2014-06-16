---
title: 浅析霍夫变换检测直线
date: '2013-08-07'
categories: ['Computer Vision']
tags: [OpenCV, Hough Transform]
toc: true
---

霍夫变换在检测直线，圆和椭圆等形状有着很广泛的应用，它通过一个投票过程实现对不同形状的参数的估算。这里这个投票过程是指把二值图像空间的点映射到参数空间，记录参数空间中不同参数对应的点的累积，从而实现形状上的点对参数的投票过程，然后选取局部累积最大值作为检测到的形状的参数输出。

那为什么叫它霍夫变换，当然是和霍夫有关系的。1962年保罗**.**霍夫（Paul.Hough）写了一个叫*Method and Means for Recognizing Complex Patterns*的专利，其中有用到该方法检测粒子物理中云图中的直线和曲线。目前广泛应用的霍夫变换是1972年由 Richard Duda 和 Peter Hart 提出的[[1]](http://dl.acm.org/citation.cfm?id=361242),该方法提出了用 **angle-radius** 替换 **slope-intercept** 作为霍夫变换的参数空间，简化了计算。后来 [Dana H. Ballard](http://www.cs.utexas.edu/~dana/)的一篇 *Generalizing the Hough transform to detect arbitrary shapes* 使得霍夫变换在计算机视觉领域开始广泛应用。

<!--more-->

###基本原理

![](/media/Computer-Vision/hough-transform.jpg)

如上图所示，一条直线可以表示为y=kx+b，其中k是直线的斜率，b是直线的截距。在一幅图像中，我们只知道一些离散的二位点，如果想检测到这些点构成的直线的话，就需要求出直线的参数(k,b)，目前点的坐标是已知，我们把k，b看成参数空间中的两个坐标，就可以求出一个参数空间下的hough图像，图像的强度值是该参数下对应点的个数。这样由于在同一个直线的点的参数(k,b)就是一样的，所以就会在hough图上形成强点，从而求出直线的参数。这其实就是一个已知点对参数的一个投票过程(voting procedure)。

上面提到直线表示方式是我们表示直线的常用方法，这种表示方式叫做“斜率-截距”(**slope-intercept**)的方式，其参数空间表示为(k,b)。但是这种表示方式不足的地方在于直线垂直x坐标轴时（如x=a），k变得无法表示。所以 Duda和Hart提出了用“角度-半径”(**angle-radius**)的参数空间表示方式，这样直线方程变为  $$ xcos\theta+ysin\theta=r $$ 
其参数空间表示为$(\theta,r)$，当$\theta\in[0,\pi)$时，$r\in R$。


###霍夫变换检测直线在OpenCV中的实现
OpenCV中实现了两种检测直线的方法，一种是Standard Hough Transform(SHT)，对应OpenCV中的Houghlines函数；一种是Probabilistic Hough Transform(PHT)，对于openCV中的HoughlinesP函数。

####Standard Hough Transform
首先介绍HoughLines函数，其C++函数声明如下：
```
void HoughLines(InputArray image, OutputArray lines, double rho, double theta, int threshold, double srn=0, double stn=0 )
```
其中


- **image**是图像输入；
- **lines**是直线参数输出；
- **rho**是距离r的分辨率，即每个像素对应的距离；
- **theta**是角度$ \theta $的分辨率，即每个像素对应的角度；
- **threshold**是累计值的阈值，大于这个阈值输出相应的直线； 
- **srn**是多尺度霍夫变换中rho的分母；
- **stn**是多尺度霍夫变换中的theta的分母；对于标准霍夫变换，srn和stn都设置为0。

这里除了Standard Hough Transform，还有个Multi-scale Hough Transform，具体源代码没怎么看懂，感觉好像是针对rho和theta这两分辨率做了一些调整。

下面贴出OpenCV中Houghlines实现的核心源代码，并作详细的注释：
```
static void
icvHoughLinesStandard( const CvMat* img, float rho, float theta, int threshold, CvSeq *lines, int linesMax )
{
    //_accum是存放hough图中累积值，及对应hough图的二维矩阵
    //_sort_buf是存放hough图局部极值用来排序的
    cv::AutoBuffer<int> _accum, _sort_buf;
    //这两个变量用来存放离散的sin(theta)和cos(theta)
    cv::AutoBuffer<float> _tabSin, _tabCos;
    //存放图像数据
    const uchar* image;
    //步长，宽，高
    int step, width, height;
    //numangle是离散的角度的个数，即hough图的高度
    //numrho是离散的半径的个数，即hough图的宽度
    int numangle, numrho;
    //统计局部极值的个数
    int total = 0;
    int i, j;
    float irho = 1 / rho;
    double scale;

    //判断图像是否是矩阵并且图像类型是8位单通道
    CV_Assert( CV_IS_MAT(img) && CV_MAT_TYPE(img->type) == CV_8UC1 );
    image = img->data.ptr;
    step = img->step;
    width = img->cols;
    height = img->rows;
    //利用theta和rho这两个分辨率，计算hough图的高宽
    numangle = cvRound(CV_PI / theta);
    numrho = cvRound(((width + height) * 2 + 1) / rho);

    //分配内存
    _accum.allocate((numangle+2) * (numrho+2));
    _sort_buf.allocate(numangle * numrho);
    _tabSin.allocate(numangle);
    _tabCos.allocate(numangle);
    int *accum = _accum, *sort_buf = _sort_buf;
    float *tabSin = _tabSin, *tabCos = _tabCos;
	
    //内存赋值为零
    memset( accum, 0, sizeof(accum[0]) * (numangle+2) * (numrho+2) );

    //计算离散的sin(theta)和cos(theta)
    float ang = 0;
    for(int n = 0; n < numangle; ang += theta, n++ )
    {
	tabSin[n] = (float)(sin((double)ang) * irho);
	tabCos[n] = (float)(cos((double)ang) * irho);
    }

    // stage 1. fill accumulator
    // 重点来了，计算hough图，注意这里的theta是0到pi，计算出来的r是
    //[-(numrho-1)/2,(numrho-1)/2]，需要转换到[1,numrho-1]
    for( i = 0; i < height; i++ )
	for( j = 0; j < width; j++ )
	{
	    if( image[i * step + j] != 0 )
		for(int n = 0; n < numangle; n++ )
		{
		    int r = cvRound( j * tabCos[n] + i * tabSin[n] );
		    r += (numrho - 1) / 2;
		    accum[(n+1) * (numrho+2) + r+1]++;
		}
	}
	
    // stage 2. find local maximums
    //寻找局部极值，同时满足累积值大于threshold
    for(int r = 0; r < numrho; r++ )
    for(int n = 0; n < numangle; n++ )
    {
	int base = (n+1) * (numrho+2) + r+1;
	if( accum[base] > threshold &&
	    accum[base] > accum[base - 1] && accum[base] >= accum[base + 1] &&
	    accum[base] > accum[base - numrho - 2] && 
	    accum[base] >= accum[base + numrho + 2] )
   	            sort_buf[total++] = base;
    }

    // stage 3. sort the detected lines by accumulator value
    // 排序局部极值
    icvHoughSortDescent32s( sort_buf, total, accum );

    // stage 4. store the first min(total,linesMax) lines to the output buffer
    //存储局部极值较高的参数对应的直线，通过linesMax限制直线个数的输出
    linesMax = MIN(linesMax, total);
    scale = 1./(numrho+2);
    for( i = 0; i < linesMax; i++ )
    {
	CvLinePolar line;
	int idx = sort_buf[i];
	int n = cvFloor(idx*scale) - 1;
	int r = idx - (n+1)*(numrho+2) - 1;
	line.rho = (r - (numrho - 1)*0.5f) * rho;
	line.angle = n * theta;
	cvSeqPush( lines, &line );
    }
}	
```

SHT是相对比较简单的，其核心内容是累积图像上点对hough图中参数的投票。因为在参数空间中要想表示hough图像，其参数必定是不连续的，需要离散化相应的参数，所以需要theta和rho的分辨率，这两个值越小，其参数离散的越精细，相对计算出的参数就会越准确，当然计算量也会增大。

####Probabilistic Hough Transform

PHT的原理介绍可以参考[Jiri. Matas](http://cmp.felk.cvut.cz/~matas/)的*Robust Detection of Lines Using the Progressive Probabilistic Hough Transform*，基本思想是：不断地随机选取图像中的点进行投票，直到参数空间某个点$(\theta,r)$的累积值达到一个设定阈值后， 搜寻该参数对应直线上的点，基于一个gap阈值（判断两个点的连通性）寻找到直线的起始点（所以PHT输出的是线段），然后删除图像上对应该直线上的点并收回这些点对其他参数点的投票。基本的算法流程如下：

1. 检查图像是否所有像素值为零，如果是退出；
2. 随机选取一个点，更新hough图中的累积值；
3. 在图像上删除所选取的随机点；
4. 检查目前hough图上最大值是否大于阈值thr(N)，如果有，则输出该参数对应的直线，否则转到1；
5. 沿着该直线搜寻图像上的点，基于gap阈值，判断改点是否属于该线段；
6. 删除该线段对应的图像上的点；
7. 撤销该线段上点对hough图上参数的投票；
8. 如果线段的长度大于length，把该线段加入到输出列表中
9. 转到1。

OpenCV上的具体实现是函数HoughlinesP，其C++声明如下：
```
void HoughLinesP(InputArray image, OutputArray lines, double rho, double theta, int threshold, double minLineLength=0, double maxLineGap=0 )
```

其中，


- **image**是图像输入；
- **lines**是直线参数输出；
- **rho**是距离r的分辨率，即每个像素对应的距离；
- **theta**是角度$\theta$的分辨率，即每个像素对应的角度；
- **threshold**是累计值的阈值，大于这个阈值输出相应的直线；
- **minLineLength**是输出直线的最小长度阈值；
- **maxLineGap**是判断直线上两个点是否连续的最大间隔阈值。

其核心代码的具体实现如下所示，下面作了详细注释：

```c
static void
icvHoughLinesProbabilistic( CvMat* image, float rho, float theta, int threshold, int lineLength, 
int lineGap, CvSeq *lines, int linesMax )
{
    //accum：存储hough累积图 mask：存放大于零的图像点，即二值图像
    cv::Mat accum, mask;
    //存放离散sin和cos
    cv::vector<float> trigtab;
    cv::MemStorage storage(cvCreateMemStorage(0));

    //存放所有的点的坐标
    CvSeq* seq;
    CvSeqWriter writer;
    int width, height;
    int numangle, numrho;
    float ang;
    int r, n, count;
    CvPoint pt;
    float irho = 1 / rho;
    CvRNG rng = cvRNG(-1);
    const float* ttab;
    uchar* mdata0;

    CV_Assert( CV_IS_MAT(image) && CV_MAT_TYPE(image->type) == CV_8UC1 );

    width = image->cols;
    height = image->rows;

    numangle = cvRound(CV_PI / theta);
    numrho = cvRound(((width + height) * 2 + 1) / rho);

    accum.create( numangle, numrho, CV_32SC1 );
    mask.create( height, width, CV_8UC1 );
    trigtab.resize(numangle*2);
    accum = cv::Scalar(0);
    //计算离散的cos和sin
    for( ang = 0, n = 0; n < numangle; ang += theta, n++ )
    {
	trigtab[n*2] = (float)(cos(ang) * irho);
	trigtab[n*2+1] = (float)(sin(ang) * irho);
    }
    ttab = &trigtab[0];
    mdata0 = mask.data;
    //把非零点坐标写入内存
    cvStartWriteSeq( CV_32SC2, sizeof(CvSeq), sizeof(CvPoint), storage, &writer);
	
    // stage 1. collect non-zero image points
    for( pt.y = 0, count = 0; pt.y < height; pt.y++ )
    {
	const uchar* data = image->data.ptr + pt.y*image->step;
	uchar* mdata = mdata0 + pt.y*width;
	for( pt.x = 0; pt.x < width; pt.x++ )
	{
	    if( data[pt.x] )
	    {
		mdata[pt.x] = (uchar)1;
		CV_WRITE_SEQ_ELEM( pt, writer );
	    }
	    else
		mdata[pt.x] = 0;
	}
    }
    //把非零点赋值给seq
    seq = cvEndWriteSeq( &writer );
    count = seq->total;

    // stage 2. process all the points in random order
    for( ; count > 0; count-- )
    {
	// choose random point out of the remaining ones
	int idx = cvRandInt(&rng) % count;
	int max_val = threshold-1, max_n = 0;
	CvPoint* point = (CvPoint*)cvGetSeqElem( seq, idx );
	CvPoint line_end[2] = { {0,0}, {0,0} };
           
	float a, b;
	int* adata = (int*)accum.data;
	int i, j, k, x0, y0, dx0, dy0, xflag;
	int good_line;
	const int shift = 16;

	i = point->y;
	j = point->x;

	// "remove" it by overriding it with the last element
	// 用最后一个元素覆盖掉被选中的点，但是这样会不会加大最后一个点被选中
	// 的概率，以及次数？？？
	*point = *(CvPoint*)cvGetSeqElem( seq, count-1 );
	
	// check if it has been excluded already (i.e. belongs to some 	other line)
	// 如果该点值为零，说明该点已经在某条直线上了，所以被删除了
	if( !mdata0[i*width + j] )
	    continue;

	// update accumulator, find the most probable line
	for( n = 0; n < numangle; n++, adata += numrho )
	{
	    r = cvRound( j * ttab[n*2] + i * ttab[n*2+1] );
	    r += (numrho - 1) / 2;
	    int val = ++adata[r];
	    if( max_val < val )
	    {
		max_val = val;
		max_n = n;
	    }
	}

	// if it is too "weak" candidate, continue with another point
	if( max_val < threshold )
	    continue;

	// from the current point walk in each direction
	// along the found line and extract the line segment
	// 当该点update累积值后，某个参数累积值大于某一值时，
	// 以当前点为起点，沿直线两个方向搜索直线上的点
	a = -ttab[max_n*2+1];
	b = ttab[max_n*2];
	x0 = j;
	y0 = i;
	//此处判断是考虑到离散直线的表示问题，因为沿直线方向，x坐标移动一个像
	//素，对应的y坐标应该移动|b/a|个像素，而且|b/a|应该小于1，而当|b/a|>1
	//的时候，我们按照y坐标移动一个像素的方式进行搜寻，对应x坐标移动|a/b|
	//个像素，对应|a/b|<1。这种搜寻方式是为了保证在斜线方向上，离散像素的
	//距离保持最近。
	if( fabs(a) > fabs(b) )
	{
	    xflag = 1;
	    dx0 = a > 0 ? 1 : -1;
	    //这里以及后面的shift是考虑到dy0是int型，而b/fabs(a)是float型，
	    //直接赋值会丢失数据，b/fabs(a)<1，直接赋值的话，dy0=0
	    dy0 = cvRound( b*(1 << shift)/fabs(a) );
	    y0 = (y0 << shift) + (1 << (shift-1));
	}
	else
	{
	    xflag = 0;
	    dy0 = b > 0 ? 1 : -1;
	    dx0 = cvRound( a*(1 << shift)/fabs(b) );
	    x0 = (x0 << shift) + (1 << (shift-1));
	}

	for( k = 0; k < 2; k++ )
	{
	    int gap = 0, x = x0, y = y0, dx = dx0, dy = dy0;

	    if( k > 0 )
		dx = -dx, dy = -dy;
	
	    // walk along the line using fixed-point arithmetics,
	    // stop at the image border or in case of too big gap
	    //该段循环是实现线段起始点的搜寻，并且
	    //据此判断线段的长度是否符合要求
	    for( ;; x += dx, y += dy )
	    {
		uchar* mdata;
		int i1, j1;

		if( xflag )
		{
		    j1 = x;
		    i1 = y >> shift;
		}
		else
		{
		    j1 = x >> shift;
		    i1 = y;
		}

		if( j1 < 0 || j1 >= width || i1 < 0 || i1 >= height )
		    break;
		mdata = mdata0 + i1*width + j1;
	
		// 对于每个非零点，更新线段的起始点，重置gap为零
		if( *mdata )
		{
		    gap = 0;
		    line_end[k].y = i1;
		    line_end[k].x = j1;
		}
		else if( ++gap > lineGap )
		    break;
	}
    }
			
    //判断线段的长度是否足够长
	    good_line = abs(line_end[1].x - line_end[0].x) >= lineLength ||
		abs(line_end[1].y - line_end[0].y) >= lineLength;
	
    //该段循环和上段循环类似，但是目标不同，这里是为了清除掉
    //good_line对应的图像的点，即赋值为零，并且撤回这些点对
    //其他accum的投票；此段循环的前提是已经判断出了good_line，
    //而good_line是由上段循环完成的。
    for( k = 0; k < 2; k++ )
    {
	int x = x0, y = y0, dx = dx0, dy = dy0;

	if( k > 0 )
	    dx = -dx, dy = -dy;

	// walk along the line using fixed-point arithmetics,
	// stop at the image border or in case of too big gap
	for( ;; x += dx, y += dy )
	{
	    uchar* mdata;
	    int i1, j1;

	    if( xflag )
	    {
		j1 = x;
		i1 = y >> shift;
	    }
	    else
	    {
		j1 = x >> shift;
		i1 = y;
	    }

	    mdata = mdata0 + i1*width + j1;

	    //对于good_line上的每个非零点，撤销其对应accum的投票，并置零
	    if( *mdata )
	    {
		if( good_line )
		{
		    adata = (int*)accum.data;
		    for( n = 0; n < numangle; n++, adata += 	numrho )
		    {
			r = cvRound( j1 * ttab[n*2] + i1 * ttab[n*2+1] );
			r += (numrho - 1) / 2;
			adata[r]--;
		    }
		}
		*mdata = 0;
	    }
	
	    if( i1 == line_end[k].y && j1 == line_end[k].x )
	    break;
	}
    }
			
    //输出good_line
    if( good_line )
    {
	CvRect lr = { line_end[0].x, line_end[0].y, line_end[1].x, line_end[1].y };
	cvSeqPush( lines, &lr );
	if( lines->total >= linesMax )
	    return;
    }
}
```

	
整体来说，PHT的思路还是比较清晰的，代码开始比较迷惑我的地方在于沿直线的搜寻问题上，后来查阅OpenCV画直线的函数，也发现其直线上离散点的搜寻规律，从而理解了这个部分。

###总结
STH和PTH都有着各自的优势，STH在图像中直线上点比较稀疏的情况下，检测效果较好，而PTH针对这种情况会出现同一直线分成多条段的情况，主要是由于PTH的直线搜索策略对于稀疏点效果不好，当点稍微偏离直线时就容易搜不到，这样的话，只能通过降低其累积阈值以及最小长度，从而导致一条直线分成多条线段的问题。但PTH在噪声较大的情况其计算效率相对较高，而且可以输出直线的起始点，即输出的是线段，这在边缘较为稠密的图像中检测效果较好。

本来打算把圆的检测也介绍下的，但写道这看到篇幅已然很大，而且圆的检测还没有仔细研究彻底，所以决定下次再讲。
