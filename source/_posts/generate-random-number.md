---
title: 几种生成随机数的方法
date: '2013-08-27'
categories: ['Tips']
tags: [随机数, 均匀分布, 高斯分布, OpenCV]
toc: true
---
随机数在很多程序中经常被用到，但每次用到的时候，由于没有系统的总结，总是得重新查阅，所以这次打算系统的介绍一下c/c++中如何产生随机数，以备以后查阅。随机数服从的分布常用的有均匀分布和高斯分布，所以这里仅介绍这两种。

###均匀分布
这里介绍三种生成服从均匀分布随机数的方法：线性同余法，梅森旋转算法和Multiply-With-Carry算法。

####线性同余法
[线性同余法](http://en.wikipedia.org/wiki/Linear_congruential_generator)是一个很古老的随机数生成算法，它的数学形式如下：

$$X\_{n+1} = (aX\_n + c)(\mod{m})$$ 
其中，$$m>0,0<a<m, 0<c<m$$ 

<!--more-->

这里${X\_n}$这个序列生成一系列的随机数，$X\_0$是种子。随机数产生的质量与m，a，c三个参数的选取有很大关系。这些随机数并不是真正的随机，而是满足在某一周期内随机分布，这个周期的最长为m。根据Hull-Dobell Theorem，当且仅当：

1. c和m互素;
2. a-1可被所有m的质因数整除;
3. 当m是4的整数倍，a-1也是4的整数倍

时，周期为m。所以m一般都设置的很大。c语言标准库就是采用的这个方法实现随机数生成的，其中各个参数的设置如下：$$m=2^{31}, a=1103515245, c=12345$$ 这个算法的优势在计算速度快，内存消耗少。但是，对于随机数质量要求高的应用，像基于蒙塔卡罗思想的算法粒子滤波器等就不适合用这种方法，因为相邻的随机数并不独立，序列关联性较大。

下面详细介绍下c语言下随机数生成函数的使用方法，声明在头文件`stdlib.h`（c++中建议用`cstdlib`）中的函数
```
	int rand()
```
提供随机数的生成，这个均匀分布的范围是[0,RAND\_MAX]， RAND_MAX的大小和系统有关。因为rand()是按照指定顺序生成随机数的，所以每次生成的随机数都是一致的。如果想每次产生的随机数不同，则需要函数 
```
	void srand(unsigned seeed)
```
（来自`stdlib.h`）为随机数生成器播散种子。如果在调用rand()前不执行srand()，则默认产生的种子是1，即执行srand(1)和不执行srand的结果是一样的。至于种子怎么选，一般常用的方法是利用time函数（在头文件`time.h`）来获得系统时间，它的返回值为从1970年1月1日零时零分零秒到目前为止所经过秒数，然后将time_t型数据转化为unsigned型再传给srand函数，即
```
	srand((unsigned) time(NULL))
```

里注意time函数的调用，它的函数声明如下：
```
	time_t time(time_t * timer)
```
既可以通过函数参数timer返回现在的时间，也可以通过函数的返回值返回现在的时间，所以这里srand用到的是返回值返回的时间，用不到参数返回时间，故使用空指针作为参数。

如果想产生其他范围的随机数，例如[a,b]，则需要利用**求余运算（%）**：
```
	rand()%(b-a)+a
```
如果是产生[0,1]范围内的浮点型随机数，则使用**除法运算（/）**:
```
	(double)rand()/RAND_MAX
```
####梅森旋转算法

[梅森旋转算法](http://en.wikipedia.org/wiki/Mersenne_twister)目前生成随机数较好的算法，他的名字来源于其周期为$2^{19937}-1$,而这数是个[梅森素数](http://en.wikipedia.org/wiki/Mersenne_prime)。常用的两个变体是32位的MT19937和，64位的MT19937-64,对于k位字长，其服从均匀分布的范围是[$2^k-1$]。
梅森旋转算法适合基于蒙塔卡罗思想的算法，而且周期很长（$2^{19937}-1$）远高于线性同余算法(一般为$2^{31}$或$2^{32}$)。目前Python，Ruby，Matlab（[Mersenne Twister](http://www.mathworks.com/matlabcentral/fileexchange/6614)）,最新的C++11（[std::mersenne_twister_engine](http://en.cppreference.com/w/cpp/numeric/random/mersenne_twister_engine)）以及Boost库（[boost random/mersenne_twister](http://www.boost.org/doc/libs/1_49_0/boost/random/mersenne_twister.hpp)）中都有这个算法的实现。此外，还有一个不错的实现在一个叫[RandomLib](http://randomlib.sourceforge.net/)的库中。


####Multiply-With-Carry算法

[Multiply-With-Carry算法](http://en.wikipedia.org/wiki/Multiply-with-carry)是[George Marsaglia](http://en.wikipedia.org/wiki/George_Marsaglia)发明的一系列随机算法中的一个。其特点在于计算速度快，周期长，OpenCV中均匀分布的随机数产生器就是用的这个方法。下面详细介绍下OpenCV中随机数产生器的使用。

OpenCV中随机数生成器是由一个叫[RNG](http://docs.opencv.org/modules/core/doc/operations_on_arrays.html?highlight=rng#RNG)的类实现的。如果要产生服从均匀分布的随机数，使用`RNG::uniform`,该函数有三个重载函数，分别针对不同数据类型：

```
	int RNG::uniform(int a, int b)
	float RNG::uniform(float a, float b)
	double RNG::uniform(double a, double b)
```
其中，**a，b**是均匀分布是范围。这里范围的边界为[a,b)，即包含a，不包含b。

使用的时候需要注意一些问题，下面是OpenCV文档中给出的几个实例：
```
	RNG rng;
	// 总是产生 0, 因为[0,1)之间的整数只有0
	double a = rng.uniform(0, 1);
	
	// 产生 [0, 1)的double型的随机数
	double a1 = rng.uniform((double)0, (double)1);

	// 产生 [0, 1)的float型的随机数
	double b = rng.uniform(0.f, 1.f);

	// 产生 [0, 1)的double型的随机数
	double c = rng.uniform(0., 1.);

	// 造成编译器混淆，产生错误。
	//  RNG::uniform(0, (int)0.999999)? or RNG::uniform((double)0, 0.99999)?
	double d = rng.uniform(0, 0.999999);
```

###高斯分布

产生服从高斯的分布的随机数方法主要有四种，具体可以参考[这里](http://blog.chinaunix.net/uid-22666248-id-357093.html)：中心极限定理法，Box-Muller算法，Marsaglia polar算法以及Ziggurat算法。这四种算法都是基于给定均匀分布的前提下实现的。

####中心极限定理法

首先给出中心极限定理的定义，相信这个大家都还有印象吧。

设随机变量$ X_1,X_2,...,X_n $  相互独立，具有相同的分布，$ E(X_k)=\mu, D(X_k)=\sigma $，记
$$ Y=\frac{\sum\_{k=1}^{n}X\_k-n\mu}{\sqrt{n\mu^2}}= \frac{\overline{X}-\mu}{\mu/\sqrt{n}} $$ 则
$$\lim_{n\rightarrow \infty }Y_n \sim N(0,1)$$


因为c语言已经提供了生成服从均匀分布随机数的函数`rand()`，那么我们就可以利用服从均匀分布的随机数生成服从高斯分布的随机数。这里首先回顾一下什么是均匀分布，均匀分布概率密度函数为：

$$f(x)=1/(b-a), x\in [a,b]$$

其期望为：$E(x)=(a+b)/2$，方差为：$D(x)^2=(b-a)^2/12$。

首先，我们选取n个[0,1]范围内的随机数，设为$ x\_1,x\_2,...x_n $， 则该均匀分布的期望为 $E(x)=1/2,$ 方差为 $D(x)^2=1/12$。这样当n足够大的时候，
$$ \overline{x}=\frac{\sum\_{k=1}^{n}x\_k-nE(x)}{\sqrt{nD(x)^2}}=\frac{\sum\_{k=1}^{n}x\_k-n/2}{\sqrt{n/12}} \sim N(0,1) $$

代码如下：
```
	double random_guassian_central_limit()
	{
		const int N = 20;
		double sum = 0;
		for( int i = 0; i < N; i++)
		{
			sum += (double)rand()/RAND_MAX;
		}
		sum -= N/2.0;
		sum /= sqrt(N/12.0);

		return sum;
	}
```
####Box-Muller算法
[Box-Muller算法](http://en.wikipedia.org/wiki/Box-Muller_transform)可以基于服从均匀分布的随机产生服从高斯分布的随机数。基本的形式如下：

设$U\_1，U\_2$是两个相互对立的服从均匀分布[0,1]的两个随机数，则
$$Z\_0=R\cos(\theta)=\sqrt{-2ln{U\_1}}\cos(2\pi U\_2)$$
及 
$$Z\_1=R\sin(\theta)=\sqrt{-2ln{U\_1}}\sin(2\pi U\_2)$$
是两个相互独立的服从高斯分布N(0,1)
其$$R^2=-2ln{U\_1}, \theta=2\pi U\_2$$

代码如下：

```
	#define PI 3.1415926
	double random_guassian_box_muller()
	{
	    static double  U1, U2;
	    double Z;
	    int flag = 0;
	    
	    if(flag == 0)
	    {
			U1 = (double)rand()/RAND_MAX;
			U2 = (double)rand()/RAND_MAX;
			Z = sqrt(-2*log(U1))*sin(2*PI*U2);
	    }
	    else
	    {
			Z = sqrt(-2*log(U1))*cos(2*PI*U2);
	    }

	    flag = 1 - flag;
	    return Z;
	}
```
####Marsaglia polar算法

[Marsaglia polar算法](http://en.wikipedia.org/wiki/Marsaglia_polar_method)的实现不需要使用三角函数的运算，其公式表示如下


设$x,y$服从均匀分布[-1,1] 则，
$$x\sqrt{\frac{-2ln(s))}{s}} , y\sqrt{\frac{-2ln(s))}{s}}$$ 是两个服从高斯分布N(0,1)是随机数，其中$s=x^2+y^2$。

我们可以先利用`rand()`生成[0,1]范围内的随机数，然后变换到[-1,1]范围内。设$X$服从[0,1]均匀分布，则$2X-1$服从[-1,1]均匀分布。

代码如下：
```
	double random_guassian_marsaglia_polar()
	{
	    static double V1, V2, S;
	    int flag = 0;
	    double X;
	    if(flag == 0)
	    {
			do
			{
		    	double U1 = (double)rand() / RAND_MAX;
		    	double U2 = (double)rand() / RAND_MAX;
		 
		    	V1 = 2 * U1 - 1;
		    	V2 = 2 * U2 - 1;
		    	S  = V1 * V1 + V2 * V2;
			}while( S >= 1 || S ==0);

    	    
			X = V1 * sqrt (-2 * log(S) / S);
	    }
	    else
	    {
			X = V2 * sqrt(-2 * log(S) / S);
	    }

	    flag = 1 - flag;
	    return X;
	}
```
####Ziggurat算法
[Ziggurat算法](http://en.wikipedia.org/wiki/Ziggurat_algorithm)也是[George Marsaglia](http://en.wikipedia.org/wiki/George_Marsaglia)发明的一系列随机算法中的一个。OpenCV中产生服从高斯分布的随机数就是采用的这种方法。RNG类中提供`RNG::gaussian`返回一个随机数，它的函数声明如下：
```
	double RNG::gaussian(double sigma)
```
其中，**sigma**是方差，这里期望默认为0。

此外，OpenCV还提供了`fill`，`randu`和`randn`用来生成填充一个数组的随机数。`randu`是生成服从均匀分布，`randn`是生成服从高斯分布，`fill`是二者都可，取决于里面的参数。

###总结
由于时间和精力有限，博客中提到的有些算法并没有深究其原理，仅仅做个简单的介绍。如果有需要，请自行查阅相关资料，网上的资料还是很丰富。而且目前几乎各种经典的随机数生成算法都有现成的实现，所以无需自己coding，而且也不建议自己coding，既费时，又不一定有好的效果和优化。

	

