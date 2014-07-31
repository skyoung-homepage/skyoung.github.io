title: OpenCV系列(三)：Mat详解
date: '2014-03-26'
categories: ['OpenCV']
tags: [OpenCV, Mat]
toc: true
---


Mat类是OpenCV最基本的一个数据类型，它可以表示一个多维的多通道的数组。Mat常用来存储图像，包括单通道二维数组——灰度图，多通道二维数组——彩色图。当然也可以用来存储点云，直方图等等，对于高维的数组可以考虑存储在SparseMat中。对于一个Mat对象M，其数据布局是由M.step[]决定的，数据存放在M.data里面，假设M有d维，则数据的寻址方式为：

$$addr(M_{i_0,...,i_{d-1}}) = M.data + i_0*M.step[0] + ... + i_{d-1}*M.step[d-1] $$

例如$Img$是一个二维三通道矩阵，则，

$$addr(Img_{i_0,i_1}) = M.data + i_0*M.step[0] + i_1*M.step[1] $$

这里需要说明的是各个维度的步长满足如下关系：`M.step[i] >= M.step[i+1]*M.size[i+1] `,也就是二维数组的数据的存放是一行一行的，三维数组数据存放是一面一面的。

<!-- more -->
下面给出OpenCV中Mat类的一个粗略定义如下：
```cpp
class CV_EXPORTS Mat
{
public:
    // ... a lot of methods ...
    ...

    /*! includes several bit-fields:
         - the magic signature
         - continuity flag
         - depth
         - number of channels
     */
    int flags;
    //! the array dimensionality, >= 2
    int dims;
    //! the number of rows and columns or (-1, -1) when the array has more than 2 dimensions
    int rows, cols;
    //! pointer to the data
    uchar* data;

    //! pointer to the reference counter;
    // when array points to user-allocated data, the pointer is NULL
    int* refcount;

    // other members
    ...
};
```

###构造Mat的方法
构造Mat的方式有很多种，下面把常用的方法一一列出：

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

8. IplImage，CvMat和Mat相互转换
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

###获取Mat元素的方法
构造好矩阵后，剩下一个很重要的事情就是如何快速准确的获取矩阵Mat中的元素，下面列出几种常用的获取Mat中的元素方法：

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


###Mat提供的常用成员函数

1. **赋值操作符'='**    
	除了普通的矩阵赋值外，如果一个Scalar赋值给一个Mat,则表示把Mat的所有元素赋值为这个Scalar值。

2. **矩阵的取行列及对角线操作**    
	A.row(i)，A.col(j)这些操作返回矩阵A的第i行和第j列。A.rowRange(m,n)和A.colRange(m,n)分别取的是A的第m行到第n行（包括m行，不包括n行）和A的第m列到第n列（包括m列，不包括n列）。这里需要注意一个问题,对于
```
A.row(i) = A.row(j)
```

	这一操作，并不能把第j行复制到第i行，因为A.row()返回的只是矩阵的头，以上操作仅仅相当于两个指针的操作，所指内存其实是没有发生变化的。如果想把第j行复制到第i行，可以
```
A.row(j).copyTo(A(i))
```

	当右边的矩阵发生操作后，是可以赋值的，比如
```
A.row(i) = A.row(j)*a
A.row(i) = A.row(j) + Scalar(0,0,0);
```

	A.diag(i)取的是矩阵的对角线，这里i=0代表最中间的对角线，i=1是偏右上一行的对角线，i=-1是左下一行的对角线，例如:
```
Mat A = (Mat_<float>(3,3)<< 1,9,3,
		            7,5,0,
		            7,3,9);
```

	A.diag(0)取得是{1,5,9}，A.diag(1)取得是{9,0}，A.diag(-1)取得是{7,3}。 

3. **复制函数** 

	```
A.clone()//返回A的拷贝。
A.copyTo(B)//执行把A拷贝到B矩阵中。
A.copyTo(B,mask)//进拷贝mask对应的部分
```

4. **转换矩阵元素的数据类型** 

	```
A.converTo(B,tpye,scale)//把A的类型转换为type并且按照scale缩放A到B矩阵中
assignTo(A,type)//更改A的元素数据类型
```

5. **设定矩阵的值** 

	```
A.setTo(s)//把A中所有的值赋值为s
```

6. **更改矩阵的通道数和行数**

	A.reshape()改变通道数，A.resize()改变行数。其中A.reshape()这个操作不改变ros*cols*channels的个数，仅仅相当于重构这些元素，例如：
```
vector<Point> vec;//vec是N个Point
Mat pointMat = Mat(vec); //pointMat是一个三通道的N×1的矩阵
pointMat.reshape(1)//pointMat变为一个单通道N×3的矩阵
```

	更改矩阵的行数如下
```
A.resize(sz) //A变为sz行
```

7. **locateROI和adjustROI**

	这两个函数主要是对submatrix的操作，即通过A.row(),A(Range(i,j),Range::all())等操作获得的submatrix在原始矩阵中位置。例如：
```
Mat A = (Mat_<float>(3,3)<< 1,9,3,
		            7,5,0,
		            7,3,9);
Mat B = A(Range(0,2),Range(1,3));//B变为 {9,3,
				 //      5,0}
Size sz;
Point p1;
B.locateROI(sz, p1);
cout<<sz<<" "<<p1<<endl;  //sz是原矩阵的大小3×3,p1是B在A中位置(1,0)
B.adjustROI(0,1,0,0); //四个参数分别是上下左右平移的像素数，这里是把B向下平移1行，
		      //最后得出B为{9,3,
		      //           5,0，
		      //    	   3,9}
cout<<B<<endl;
```

8. **Mat的各项属性**

	```
A.total() //元素的个数
A.elemSize() //元素的大小，如果是8UC3的话，返回3*sizeof(uchar)
A.elemSize1() //如果是8UC3的话，返回sizeof(uchar)
A.type() //元素的数据类型
A.depth()//元素的位数
A.channels()//矩阵的通道数
A.step1() //矩阵的每一行元素的个数，A.step/A.elemSize1
A.size() //矩阵的尺寸
//注意以下是成员变量不是成员函数
A.step //矩阵的一行的字节数
A.rows //矩阵的行数，即高
A.cols //矩阵的列数，即宽
```

###总结
Mat提供了关于矩阵的一些基本操作，这对图像的操作打下了坚实的基础，更多复杂的算法都是基于这些操作实现的。
		

		
		
		
		
		
		
		
		
	








