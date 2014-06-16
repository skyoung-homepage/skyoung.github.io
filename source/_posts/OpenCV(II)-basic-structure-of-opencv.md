---
title: OpenCV系列(二)：基本数据结构
date: '2014-02-28'
categories: ['OpenCV']
tags: [OpenCV, Basic Sturcture]
toc: true
---


常用的OpenCV数据结构像Mat，Point等我们并不陌生，但最近仔细阅读OpenCV手册后发现有些数据结构的一些便捷操作以及一些新加入的功能自己并没有掌握，这里做个记录算是学习笔记。

###众多的类模板

我们平时常用的数据结构像Point，Rect其实是类模板某一数据类型的实例。比如`Point`对应`Point_<int>`，`Rect`对应`Rect_<int>`。
这里就简单罗列一下Point对应的定义：
```
typedef Point_<int> Point2i;
typedef Point2i Point;
typedef Point_<float> Point2f;
typedef Point_<double> Point2d;
```
而且，Point也重载了加减乘除等操作运算符，比如：
```
Point pt1,pt2,pt3;
int a;
pt1 = pt2 + pt3， pt1 = pt2 - pt3;
pt1 = pt2 * a， pt1 = a * pt2;
pt1 += pt2， pt1 -= pt2， pt1 *= a;
double value = norm(pt); // L2 norm
pt1 == pt2， pt1 != pt2;
```
<!-- more -->
对于三维空间的点有`Point3_<>`对应:
```
typedef Point3_<int> Point3i;
typedef Point3_<float> Point3f;
typedef Point3_<double> Point3d;
```

`Size_<>`的定义：

```
typedef Size_<int> Size2i;
typedef Size2i Size;
typedef Size_<float> Size2f;
```

`Rect_<int>`的对应`Rect`的操作有：
```
Rect rect, rect1, rect2;
Point pt1;
Size sz1;
rect = rect + pt1, rect = rect - pt1;; //平移pt1的位置
rect = rect + sz1, rect = rect - sz1; //加上sz1对应width和height的值来放大或者缩小rect的尺寸
rect += pt1, rect -= pt1 rect +=sz1, rect -=sz1; 
rect = rect1 & rect2; //两个矩形的交集
rect = rect1 | rect2; //能包含两个矩形的最小方框
rect &= rect1, rect |= rect1; 
rect == rect1, rect != rect1; //两个矩形是否相同
```
	
###旋转矩形

OpenCV为旋转矩形提供了类RotatedRect，这个类用三个参数来描述旋转矩形，其构造函数为

```
RotatedRect(const Point2f& center, const Size2f& size, float angle)
```

其中center是矩形的中心，size是矩形的长宽尺寸，angle是沿着中心顺时针旋转的角度。

这个类的两个成员函数操作分别是：

```
void RotatedRect::points(Point2f pts[]) const
Rect RotatedRect::boundingRect() const
```

第一个成员函数`points`的获取旋转矩形的四个点，第二个成员函数`boundingRect`是获取包含旋转矩形最小方框。
下面举个OpenCV手册中简单例子:

```
Mat image(200, 200, CV_8UC3, Scalar(0));
RotatedRect rRect = RotatedRect(Point2f(100,100), Size2f(100,50), 30);

Point2f vertices[4];
rRect.points(vertices);
for (int i = 0; i < 4; i++)
	line(image, vertices[i], vertices[(i+1)%4], Scalar(0,255,0));

Rect brect = rRect.boundingRect();
rectangle(image, brect, Scalar(255,0,0));

imshow("rectangles", image);
waitKey(0);
```

###TermCriteria提供迭代终止条件

OpenCV提供了一个类，里面包含迭代终止的两个条件：迭代次数和迭代精度。

	TermCriteria::TermCriteria(int type, int maxCount, double epsilon)
其中type包括`TermCriteria::COUNT, TermCriteria::EPS or TermCriteria::COUNT + TermCriteria::EPS.`。第一个是仅在迭代次数达到`maxCount`时下终止，第二个是只仅在迭代精度达到`epsilon`时终止，第三个是随便哪个条件满足了都终止。

为什么OpenCV要提供这样一个类呢？你可能会觉得这可有可无啊，两个条件，自己当参数传入也可以啊。其实我个人感觉原因主要是为了方便OpenCV中一些算法函数接口的统一。比如KL算法，EM算法等就需要用的这些迭代条件作为参数传入。

###Matx的使用

除了Mat，OpenCV还提供了一个矩阵类Matx，主要用于在编译时就已知矩阵的大小的小矩阵。它的定义为：

	template<typename _Tp, int m, int n> class Matx {...};

它拥有和Matlab中矩阵类似的操作方式。比如你可以直接在定义的时候初始化矩阵的元素。可以采用`Mat(i，j)`的方式访问矩阵元素，例如：
```
Matx22d m(1,2,
	  3,4);
cout << m(0,1) << endl;
```

这里`Matx22d`是`Matx<double, 2, 2>`的别名，类似的比如：
```
typedef Matx<float, 1, 2> Matx12f;
typedef Matx<double, 1, 2> Matx12d;
```

但需要注意不是每个尺寸都有这样的别名，比如，`Matx77f`或者`Matx55f`就会出现编译错误，因为OpenCV中根本没有这个别名定义。

###Vec和Scalar_

Vec是一维的向量，其模板定义为：

	template<typename _Tp, int n> class Vec : public Matx<_Tp, n, 1> {...};
常见的别名定义有：
```
typedef Vec<uchar, 2> Vec2b;
typedef Vec<short, 4> Vec4s;
typedef Vec<int, 4> Vec4i;
typedef Vec<float, 6> Vec6f;
typedef Vec<double, 6> Vec6d;
```

Scalar_是Vec的一种特殊情况,来自`Vec<_Tp, 4>`的派生。


###类似Matlab操作的Range类
```
class CV_EXPORTS Range
{
public:
	Range();
    	Range(int _start, int _end);
    	Range(const CvSlice& slice);
    	int size() const;
    	bool empty() const;
    	static Range all();
    	operator CvSlice() const;

    	int start, end;
};
```

`Range(a,b)`相当于Matlab中的`a:b`，`Range::all()`相当于`:`，举例说明如下：
```
Mat img = imread("lean.jpg");
Mat img1 = img(Range(10:20), Range(2:15));//取行10-20，列2-15对应的数据
Mat img2 = img(Range(1:30), Range::all());//取1-30行的数据
```

###Mat类的相关操作

```
class CV_EXPORTS Mat
{
public:
   	...
    	int flags;
    	int dims;
    	int rows, cols;
    	uchar* data;
    	...
};
```

设M为一个Mat对象，其数据存放在M.data里面，其数据的布局是有M.step[]决定的。假设M有d维，则数据的寻址方式为：

$$addr(M_{i_0,...,i_{d-1}}) = M.data + i_0*M.step[0] + ... + i_{d-1}*M.step[d-1] $$

例如$Img$是一个二维三通道矩阵，则，

$$addr(Img_{i_0,i_1}) = M.data + i_0*M.step[0] + i_1*M.step[1] $$

构造Mat的方式有很多种，下面一一列出：

1. 使用构造函数` Mat(nrows, ncols, type[, fillValue])`,例如,
```
// 构建3×2的4通道8位矩阵，每个元素初始值为（1,2,3,4）
Mat M(3,2,CV_8UC4,Scalar(1,2,3,4));
```

2. 使用`M.create(nrows,ncols,type)`，例如，
```
//构建100×100的10通道8位矩阵
M.create(100,100,CV_8UC(10))
```

3. 构建多维的矩阵，
```
//构建一个100×100×100的8位三维矩阵
int sz[] = {100,100,100}
Mat Cube(3, sz, CV_32F, Scalar::all(0))
```

4. 使用复制构造函数或者赋值操作符
```
Mat A(B);
Mat C = B;
```

5. 单独对矩阵的某一行某一列进行操作
```
//第4行加上第6行的3倍赋值给第4行
M.row(3) = M.row(3) + M.row(5)*3;

// 把第8列拷贝到第2列，通过 M.col(1) = M.col(7)是不起作用的，应该:
Mat M1 = M.col(1);
M.col(7).copyTo(M1);
```

6. 构建矩阵的ROI区域，单独操作ROI区域的值
```
Mat img(Size(320,240),CV_8UC3);
Mat roi(img, Rect(10,10,100,100));
roi = Scalar(0,255,0);
```

	确定矩阵在原矩阵中的相对位置，使用locateROI，
	```
Mat A = Mat::eye(10, 10, CV_32S);
Mat B = A(Range::all(), Range(1, 3));
Mat C = B(Range(5, 9), Range::all());
Size size; Point ofs;
//得出ofs为（1,5），size为（10,10），为什么是（10,10）？目前没搞清楚
C.locateROI(size, ofs);
	```

7. 对于外部数据输入，进行初始化
```
//外部输入一个一维数组
void process_video_frame(const unsigned char* pixels, int width, int height, int step)
{
    Mat img(height, width, CV_8UC3, pixels, step);
    GaussianBlur(img, img, Size(7,7), 1.5, 1.5);
}

//用二维数组初始化矩阵
double m[2][3] = { {1, 2, 3}, {4, 5, 6} };
Mat M = Mat(2, 3, CV_64F, m);
```

8. IplImage和CvMat转换成Mat
```
IplImage* img = cvLoadImage("lena.jpg", 1);
Mat mtx(img); // IplImage* -> Mat
IplImage* img1 = mtx; //Mat -> IplImage*
CvMat oldmat = mtx; // Mat -> CvMat
Mat mtx1(oldmat);  //CvMat -> Mat
```
9. 类似Matlab方式和`<<`赋值
```
//类似Matlab中的单位矩阵等
M = Mat::ones(10, 10, CV_64F);
M = Mat::eye(10, 10, CV_64F);
M = Mat::zeros(10, 10, CV_64F);

//使用`Mat_`和`<<`配合
Mat M = (Mat_<double>(3,3) << 1, 0, 0, 0, 1, 0, 0, 0, 1);
```

获取Mat中的元素方法主要有：

1. 使用M.at<typename>(i,j)
```
M.at<double>(i,j)
```
2. 对于二维矩阵，可以采取逐行获取的方式：
```
double sum=0;
for(int i = 0; i < M.rows; i++)
{
    const double* Mi = M.ptr<double>(i);
    for(int j = 0; j < M.cols; j++)
        sum += std::max(Mi[j], 0.);
}
```

	对于不在乎矩阵的形状，只是简单的遍历矩阵的元素的，可以采用更快速的方法,首先检查元素排列是否连续，如果是，可以看成一个一维数组访问。

	```
double sum=0;
int cols = M.cols, rows = M.rows;
if(M.isContinuous())
{
    cols *= rows;
    rows = 1;
}
for(int i = 0; i < rows; i++)
{
    const double* Mi = M.ptr<double>(i);
    for(int j = 0; j < cols; j++)
        sum += std::max(Mi[j], 0.);
}
	```

3. 仿照STL中，使用迭代器访问：
```
	double sum=0;
	MatConstIterator_<double> it = M.begin<double>(), it_end = M.end<double>();
	for(; it != it_end; ++it)
	    sum += std::max(*it, 0.);
```

这个矩阵的迭代器可以传给STL的算法，例如`std::sort()`。

###总结

OpenCV2.0以上版本的数据结构的参考了很多C++的特性，比如模板类，迭代器，智能指针等。而且，还仿照Matlab的一些矩阵操作，给出了一些类似Matlab的更为方便的操作。OpenCV的新的数据结构Mat和OpenCV1.0的一些数据结构可以相互转换的，使其兼容老的版本。此外，OpenCV还实现了对外部输入数据的处理。




		
		
		
	








