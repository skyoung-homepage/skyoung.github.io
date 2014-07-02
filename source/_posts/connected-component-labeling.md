---
title: 几种联通区域标记的算法介绍
date: '2013-08-15'
categories: ['Computer Vision']
tags: [OpenCV, Connected Component Labeling]

---

联通区域标记(Connected Component Labeling)是图像处理里面常用的一个技术，它是用来检测二值图像中联通的区域，在许多跟踪检测算法中充当目标区域检测的作用。常见的CCL(Connected Component Labeling)包括Two-Pass的方法和One-Pass的方法，这里pass就是扫描的遍数，Two-Pass就是扫描两遍的算法，One-Pass就是扫描一遍的算法。下面仅就这两类算法中有代表性的算法做一个简单的介绍。

###Two-Pass

####传统的Two-Pass：

关于该算法的一个形象的描述可以参考博客[Labelling connected components – Example](http://www.aishack.in/2010/03/labelling-connected-components-example/)。具体的流程是：

1. 逐行扫描图像上的点，检查每个点是否是前景点，如果不是，继续扫描；
2. 检查该前景点的左边的点和上边的点
  - 如果有一个为前景点，则为当前扫描点标记和此点相同编号；
  - 如果两个都是前景点，则选择标号小的，为当前扫描点标记编号；这里注意如果两个都是前景点且标号不同，需要把标号小的点设为标号大的点的父节点，以方便第二遍扫描的时候用并查集算法（参考[利用不相交集实现等价元素的聚类](http://blog.skyoung.org/2013/08/09/union-find-algorithm/)）合并;
  - 如果都不是，则赋值目前标号label，label++（label初始值为零）；

3. 再次扫描图像，利用并查集算法合并联通的不同的label的区域。

<!--more-->

####run-based的Two-pass：

该算法定义每一行联通的一个区域叫做**run**，把run看成一个整体，逐行搜寻时，完成一个run扫描好后再进行整个run的标记，标记的准则是按照上一行是否有run和自己联通来判断如何标记当前的run；

每个run有两个label：representative label和一个provisional label，这里我们用R和P表示。当出现上一行n个run联通当前行的run的时候，更改n个run的R为最小的R，我们设为Rmin，同时标记当前run的P和R为Rmin；

该算法的优势在于减少了第一遍扫描时标号的数量，从而减少了第二遍中联通区域的合并任务。具体算法可以参考**“A Run-Based Two-Scan Labeling Algorithm”**。

###One-Pass

####基于区域生长的方法

单次扫描的算法有一种基于区域生长的联通区域标记算法，基本的思路可以参考博客[二值图像连通区标记之区域生长法](http://blog.csdn.net/yihandk666/article/details/6934664)。为防止链接失效，这里简单描述下流程：


1. 输入待标记图像bitmap，初始化一个与输入图像同样尺寸的标记矩阵labelmap，一个队列queue以及标记计数labelIndex；
2. 按从左至右、从上至下的顺序扫描bitmap，当扫描到一个未被标记的前景像素p时，labelIndex加1，并在labelmap中标记p（相应点的值赋为labelIndex），同时，扫描p的八邻域点，若存在未被标记的前景像素，则在labelmap中进行标记，并放入queue中，作为区域生长的种子；
3. 当queue不为空时，从queue中取出一个生长种子点p1，扫描p1的八邻域点，若存在未被标记过的前景像素，则在labelmap中进行标记，并放入queue中；
4. 重复3直至queue为空，一个连通区标记完成；
5. 转到2，直至整幅图像被扫描完毕，得到标记矩阵labelmap和连通区的个数labelIndex。

具体的代码也顺便贴出来，原博客中有个小bug，在进行八邻域的搜索时，没有考虑首尾相连的边界问题。
```
//定义结构体部分

//八邻域
static int NeighborDirection[8][2] = { {0,1},{1,1},{1,0},{1,-1},{0,-1},{-1,-1},{-1,0},{-1,1} };
typedef unsigned char Byte;

//定义队列
typedef struct QNode
{
    int data;
    struct QNode *next;
}QNode;
//采用的链式队列的结构
typedef struct Queue
{
    struct QNode* first;
    struct QNode* last;
}Queue;
 

//主调用函数

int ConnectedComponentLabeling(Byte *binaryImage, int width, int height, int *labemap)
{
    int cx, cy, popIndex , labelIndex = 0;
    int index;
    Queue *queue = NULL;
    queue = (Queue*)malloc(sizeof(Queue));
    queue->first = NULL;
    queue->last = NULL;
    for(cy = 1; cy < height - 1; cy++)
    {
	for(cx = 1; cx < width - 1; cx++)
	{
	    index = cy * width + cx;
       	    if(bitmap[index] == 255 && labelmap[index] == 0)
	    {
		labelIndex++;
	        SearchNeighbor(bitmap, width, height, labelmap, labelIndex, index, queue);
	        popIndex = PopQueue(queue);
		while(popIndex > -1)
		{
	            SearchNeighbor(bitmap, width, height, labelmap, labelIndex, popIndex, queue);
		    popIndex = PopQueue(queue);
		}
	    }
	}
    }
    free(queue);
    return labelIndex;
}


void SearchNeighbor(Byte *bitmap, int width, int height, int *labelmap, int labelIndex, int pixelIndex, Queue *queue)
{
    int searchIndex, i, length;
    int cx;
    labelmap[pixelIndex] = labelIndex;
    length = width * height;
    for(i = 0;i < 8;i++)
    {	
	//此处是处理边缘像素的情况下的搜索区域
	cx = pixelIndex%width;
	searchIndex = pixelIndex + NeighborDirection[i][0] * width + NeighborDirection[i][1];
	if(cx == 0 || cx == width - 1 && abs(searchIndex%width - cx) >1)
	    continue;
		
	if(searchIndex > 0 && searchIndex < length && 
	bitmap[searchIndex] == 255 && labelmap[searchIndex] == 0)
	{
	    labelmap[searchIndex] = labelIndex;
	    PushQueue(queue, searchIndex);
	}
    }
}

//推出队列
int PopQueue(Queue *queue)
{
    QNode *p = NULL;
    int data;
    if(queue->first == NULL)
    {
	return -1;
    }
    p = queue->first;
    data = p->data;
    if(queue->first->next == NULL)
    {
	queue->first = NULL;
	queue->last = NULL;
    }	
    else
    {
	queue->first = p->next;
    }
    free(p);
    return data；
}

//推入队列
void PushQueue(Queue *queue, int data)
{
    QNode *p = NULL;
    p = (QNode*)malloc(sizeof(QNode));
    p->data = data;
    if(queue->first == NULL)
    {
	queue->first = p;
	queue->last = p;
	p->next = NULL;
    }
    else
    {
	p->next = NULL;
	queue->last->next = p;
	queue->last = p;
    }
}
```

####基于轮廓跟踪的方法

还有一种经典的单次扫描的的算法是由台湾的中央研究院资讯科学研究所的[张复](http://www.iis.sinica.edu.tw/pages/fchang/)提出的，源代码可以在其[实验室网站](http://ocrwks11.iis.sinica.edu.tw/~dar/Download/WebPages/Component.htm)下载到，而且OpenCV的Wiki网站还有专门一个库[cvBlobsLib](http://opencv.willowgarage.com/wiki/cvBlobsLib),就是实现的这个算法。算法的详细介绍可以参考**“A Linear-Time Component-Labeling Algorithm Using Contour Tracing Technique”**，这里作一个简单的介绍。

逐行扫描图像，设当前像素为P

1. 如果P为为背景点，则继续，直到扫描到前景点；
2. 如果P是**unlabeled**的，且正上方的像素为背景像素（如图1所示），认为p是一个新遇见的**外轮廓**的起点，然后以P为起点，开始进行contour tracing跟踪外轮廓，为该轮廓上的所有点标记为新的的标号C，然后C=C+1；       
![图 1](/media/Computer-Vision/ccl1.png) 
3. 如果P的正下方的像素是**unmarked**（这里marked是指在contour tracing中外围轮廓周围的背景点会被mark）的背景像素。这里有两种情况：如果p是labeled的（如图2所示），则P既是外轮廓点，又是内轮廓点；如果P是unlabled的（如图3所示），则P左边的点N一定是labeled，则设置P和N相同的标号。无论是那种情况，都以P为起点进行contour tracing跟踪内轮廓，并把轮廓上所有的点标记和P相同的标号。        
![ 图 2](/media/Computer-Vision/ccl2.png)             ![图 3](/media/Computer-Vision/ccl3.png) 
4. 如果P不是以上两种情况，即P不是轮廓点，则P的左边点N一定是标记点，把P标记成和N相同标号即可。

#####Contour tracking

当遇到一个外轮廓或是内轮廓的起点时，首先通过**Tracker**搜寻轮廓的下一个点，直到输出点又一次为起点位置，从而实现轮廓的跟踪。

#####Tracker

如图4所示，Tracker的目的是搜寻当前点P周围八邻域的点，找到轮廓的下一个点。按照顺时针方向搜寻，对于一个外轮廓的起点，其起始搜寻点为7,因为6已经知道是个背景点，而对于一个内轮廓点，其起始搜寻点为3,因为2已经知道是个背景点。对于轮廓上的任意一个点，其下一个点的搜寻起始位置为(d+2) mod 8, 这里d是当前点的前一点的在当前的八邻域的位置，如图5所示。一旦起始搜寻点确定，按照顺时针方向搜寻第一个前景点就是下一个轮廓的点。但是cvBlobsLib上的实现方法是采用逆时针搜寻的方式，所以代码上和原论文中稍有不同。

 ![图 4](/media/Computer-Vision/tracker.png)    ![图 5](/media/Computer-Vision/tracker1.png) 

###总结
以上仅是我这几天查阅文献得到的一个初步总结，所以难免有疏漏，这里记下来仅仅是做个笔记，以备后来查阅。对于轮廓的检测在OpenCV中也有相应的函数findContours，这里列出供参考。

