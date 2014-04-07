---
title: 'OpenCV系列(一)：简介'
date: '2014-01-10'
categories: [OpenCV]
tags: [OpenCV]
---

一直都想系统全面地学习一下OpenCV，但却迟迟懒于行动，虽然平时也用OpenCV的一些东西，但毕竟没有对其没有一个全面的把握。这次借此机会打算参考OpenCV手册系统地整理一下自己的学习心得。

###OpenCV主要模块

OpenCV主要包括core，imgproc，video，cliab3d，feature2d，objdetect，highgui，gpu等几个模块，其中

* **core** 主要是图像的一些基本数据结构如Mat，Point，Rect等及其相应的操作;
* **imgproc** 主要包括一些针对图像的基本处理，像线性或非线性的图像滤波，几何形变操作（如尺寸变化，仿射变换等）;
* **video** 主要包括运动估计，背景差分法跟踪，以及一些其他目标跟踪算法;
* **calib3d** 主要是多视角几何，单目相机和双目相机的参数标定，目标姿态估计，立体匹配和三维重建方面的算法;
* **feature2d** 主要包括基于特征点的检测器，特征点的提取和匹配。objdetect主要包括检测目标或是特定的目标（像人脸，行人等）;
* **highgui** 包括视频和图像的读写，以及一些简单的UI功能;
* **gpu** 主要是一些视觉算法基于GPU加速的实现。
<!--more-->

###OpenCV的命名空间:cv

所有的OpenCV中的类和函数的都是定义在cv这个命名空间中的。所以在使用这些类和函数时，要在前面加上`cv::`或者在程序开始部分写上`using namespace cv`

目前OpenCV中有些函数名或是类的名和C++标准库中的名字是冲突的。所以为了解决这个问题，可以直接在函数或是类前面加入相应的命名标示符，比如：

```c
		cv::Mat a(100, 100, CV_32F);
		cv::randu(a, Scalar::all(1), Scalar::all(std::rand()));
		cv::log(a, a);
		a /= std::log(2.);
```
这里，`randu`和`log`在OpenCV和C++标准库中都有实现，所以为了区分，可以在前面加上相应的命名空间，当然如果两个函数的形参不同编译器也可以通过函数的参数个数或者参数类型实现区分，但是加上相应的命名空间可以避免很多潜在的隐患。

###自动内存管理机制

OpenCV的内存分配和释放是自动的。例如对于Mat类数据，他的内存释放时考虑到内存的共享，即若有其他指针指向同一片内存区域，则对应的Mat数据不会被释放。矩阵的赋值实际上并没有数据的拷贝，而只是被赋值指针指向赋值指针所指的区域，即共享同一个对象。而`Mat::clone`提供了Mat数据就的拷贝过程，相当于拷贝一份新的数据给别人。

举个OpenCV手册中的例子如下：
```
	Mat A(1000, 1000, CV_64F);
	//建一个矩阵头B，B和A指向同一个对象，没有复制操作
	Mat B = A;
	//建一个矩阵头C，指向B的第三排，也即A的第三排
	Mat C = B.row(3);
	//把B指向的对象，也即A的对象复制一份给D
	Mat D = B.clone();
	//把B指向的对象，也即A的对象的第5排拷贝一份给C，也即A的第三排
	B.row(5).copyTo(C);
	//A指向D指向的对象，而A原来指向的对象被B和C指向的对象不变
	A = D;
	//此操作只是让B指向空，而A被C指向的部分仍然存在
	B.release();
	//把C的内容拷贝到C，这样A被C指向的部分就没有指针指向了，就会被自动释放掉了。
	C = C.clone();
```
我们知道C++ TR1中有个智能指针`std::shared_ptr`,它的作用和普通的指针没有什么不同，但是会记录有多少个`shared_ptr`共同指向一个对象，即引用计数，一旦最后一个这样的指针被销毁后，引用计数变为0时，这个对象就会自动被释放。同样OpenCV中的数据结构也具有类似的内存管理机制。而且对于普通的指针对象，OpenCV也提供了`Ptr<>`实现智能指针。

###自动分配输出矩阵的内存

OpenCV不但可以自动释放内存，而且可以为函数中输出参数自动分配内存，比如输入一个`cv::Mat`类型的矩阵，其输出矩阵会自动分配一个和输入矩阵相同尺寸和类型的矩阵。例如：
```
	#include "cv.h"
	#include "highgui.h"

	using namespace cv;

	int main(int, char**)
	{
	   VideoCapture cap(0);
	   if(!cap.isOpened()) return -1;

	   Mat frame, edges;
	   namedWindow("edges",1);
	   for(;;)
	   {
		cap >> frame;
		cvtColor(frame, edges, CV_BGR2GRAY);
		GaussianBlur(edges, edges, Size(7,7), 1.5, 1.5);
		Canny(edges, edges, 0, 30, 3);
		imshow("edges", edges);
		if(waitKey(30) >= 0) break;
	   }
	   return 0;
	}
```
`frame`没有分配内存，但是当`cap>>frame`的时候，由于`cap`以及知道图像的尺寸和深度，会自动分配一个相应图像矩阵给`frame`。还有`cvtColor(frame, edges, CV_BGR2GRAY)`的时候，`edges`会自动分配一个和输入图像`frame`相同大小的图像，`channel`取决于`CV_BGR2GRAY`，所以是1。这里内存的分配只是在循环的第一次分配，后续只有在视频的大小发生变化的时候才重新分配。

这个机制的关键在于`Mat`类自带的成员函数`Mat::create`。这个函数可以根据提供的类型和尺寸分配相应的内存，如果这个矩阵已经是指定的尺寸和类型了，就什么也不做，如果不是，则释放掉原来的内存（若存在的话，当然内存的释放服从上面提到的内存管理机制），分配新的内存。而因为OpenCV的函数会自动的为输出`Mat`调用`create`，所以才会有这个自动为输出矩阵分配内存的特性。

但是目前仍然有些函数是不具有这个功能的，需要注意，比如`cv::mixChnnels, cv::RNG::fill`。

###如何处理数据类型溢出

我们知道一个8位图像的像素值范围是[0,255],当我们为一个这样类型的的像素赋一个超过这个范围的数值时，就会发生溢出，这样就不是我们想要的结果。以8位无符号整型数据为例，实际在对一个矩阵元素$I(x,y)$赋值$r$的时候是进行了如下操作$$I(x,y)=\min(\max(round(r),0),255)$$
但需要注意对于32位整型没有溢出处理机制。

###矩阵元素的的数据类型

OpenCV的矩阵元素的数据类型包括以下几种：

* 8-bit unsigned integer (uchar)
* 8-bit signed integer (char)
* 16-bit unsigned integer (ushort)
* 16-bit signed integer (short)
* 32-bit signed integer (int)
* 32-bit floating-point number (float)
* 64-bit floating-point number (double)

这些基本的数据类型分别表示为

	enum { CV_8U=0, CV_8S=1, CV_16U=2, CV_16S=3, CV_32S=4, CV_32F=5, CV_64F=6 };

对于多通道，就是相应的在后面加上`C(n)`即可，例如

	CV_8UC(1) == CV_8U CV_8UC(2) == CV_8UC2 ... //对于n=[1,4],可以去掉括号
	CV_16SC(5) CV_32FC(10) ...

通道的最大值由`CV_CN_MAX`定义，目前是512。

不同的CV算法对图像的元素数据类型支持是有限制的，下面列出一下情况：

* 人脸识别算法仅支持8位的灰度或彩色图像
* 大部分机器学习算法和线性函数仅支持`float`类型的矩阵
* 基本的函数，如`cv::add`,支持所有类型
* 颜色空间转换的函数仅支持8为无符号，16无符号和32位`float`

###InputArray和OutputArray

许多OpenCV的函数一般的参数是`cv::Mat`类型，但有些时候为了更方便的直接传入`std::vector<>`（比如一个数组）或者是`Matx<>`。为了避免重复多个函数的API，函数的接口的形参类型中，输入参数变为`InputArray`，输出参数为`OutputArray`。这样的话，函数就不仅仅可以传入`Mat`类型的参数，还以可以传入`std::vector<>,Matx<>,Vect<>`或者`Scalar`类型参数。如果函数有个可选参数，如果你不想传入这个参数，可以传一个`noArray()`。

###错误处理机制

OpenCV错误处理机制是使用了类`cv::Exception`,它其实是派生的STL中的异常处理`std::exception`，简单使用方法如下：
```
	try
	{
	    ... // call OpenCV
	}
	catch( cv::Exception& e )
	{
	    const char* err_msg = e.what();
	    std::cout << "exception caught: " << err_msg << std::endl;
	}
```

	




