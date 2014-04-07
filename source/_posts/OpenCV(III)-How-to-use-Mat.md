title: OpenCV系列(三)：Mat的相关操作
date: '2014-03-26'
categories: ['OpenCV']
tags: [OpenCV, Mat]

---


Mat类是OpenCV最基本的一个数据类型，涉及Mat的相关操作比较繁多，[上一篇博客](http://blog.skyoung.org/opencv/opencv%E7%B3%BB%E5%88%97-%E4%BA%8C-%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/)结尾处有介绍Mat的构造方法和获取元素的方法，这里将详细讲解一些其他的操作。

###矩阵的基本操作

OpenCV提供了矩阵常用的一些操作包括矩阵的相加，相乘，逐元素相乘，逐元素相除，求逆等。要意识到Scalar类型是矩阵Mat的元素，而一个实数可以看成是Scalar的特例，即注意矩阵的多通道问题。设为A，B为Mat类型，s是Scalar类型，a是一个实数。

* **矩阵加减：** A+B，A-B，A+s，A-s，s+A，s-A,-A.
* **矩阵乘以实数：** A\*a，a\*A
* **逐元素乘除：** A.mul(B)，A/B，a/A
* **矩阵倒置：** A.t()
* **矩阵的逆：** A.inv()
* **矩阵比较：** A comp B，A comp a，a comp A。这里comp包括 >， >=，==，!=，<=，<。得出的结果是一个单通道8位的矩阵，元素的值为255或0。
* **矩阵位操作：** A logic B， A logic s，s logic A。这里logic包括：&，|，^
* **逐元素最大最小值：** min(A,B)，min(A,a)，max(A,B)，max(A,a)。
* **矩阵的绝对值：** abs(A)
* **向量的差乘和内积：** A.cross(B)，A.dot(B);

关于Mat的操作，如果返回结果的话一定是Mat或者Saclar类型。矩阵相乘时，数据类型必须至少是float型才可以，否则会报错。

<!-- more -->

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
		B.adjustROI(0,1,0,0); //把B向下平移1行，最后得出B为{9,3,
		                      //                        5,0，
		                      //	                     3,9}			
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
		

		
		
		
		
		
		
		
		
	








