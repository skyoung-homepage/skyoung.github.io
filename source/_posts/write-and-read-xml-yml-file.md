title: OpenCV读取XML/YML文件的方法
date: 2014-04-25 15:47:36
tags: [XML YML]
categories: Tips
---

在平时写一些程序的时候，需要一些配置参数的文件，最简单的方法是使用txt文件存储参数，但当参数较多时，为了让存储的参数更美观有序且方便读取，一些其他库提供的一些数据存储方法就显得很方便了，例如[libconfig](http://www.hyperrealm.com/libconfig/)。当然这里并不打算着重讲解这个库，而是主要说一说OpenCV中关于数据存储的方法——读取XML/YML文件。[XML](http://zh.wikipedia.org/zh/XML)是一种可扩展标记语言,YML和它类似，但是更适合人阅读。关于XML格式的文件除了OpenCV可以读写外，还有其他一些专门的库提供了读写的API，例如比较轻量级的[TinyXML](http://www.grinninglizard.com/tinyxml/index.html)。

###XML/YML的写

文件的写的过程相对简单，类似C++中数据流的写的过程，使用`<<`。文件的创建是用类`FileStorage`，释放时调用` FileStorage::release()`并且关闭文件。列举一个OpenCV手册中例子如下：

<!--more-->
```
#include "opencv2/opencv.hpp"
#include <time.h>

using namespace cv;

int main(int, char** argv)
{
    FileStorage fs("test.yml", FileStorage::WRITE);

    fs << "frameCount" << 5;
    time_t rawtime; time(&rawtime);
    fs << "calibrationDate" << asctime(localtime(&rawtime));
    Mat cameraMatrix = (Mat_<double>(3,3) << 1000, 0, 320, 0, 1000, 240, 0, 0, 1);
    Mat distCoeffs = (Mat_<double>(5,1) << 0.1, 0.01, -0.001, 0, 0);
    fs << "cameraMatrix" << cameraMatrix << "distCoeffs" << distCoeffs;
    fs << "features" << "[";
    for( int i = 0; i < 3; i++ )
    {
        int x = rand() % 640;
        int y = rand() % 480;
        uchar lbp = rand() % 256;

        fs << "{:" << "x" << x << "y" << y << "lbp" << "[:";
        for( int j = 0; j < 8; j++ )
            fs << ((lbp >> j) & 1);
        fs << "]" << "}";
    }
    fs << "]";
    fs.release();
    return 0;
}
```

这样得到的`test.yml`文件中的内容存储情况如下：

```
%YAML:1.0
frameCount: 5
calibrationDate: "Fri Apr 25 16:55:42 2014\n"
cameraMatrix: !!opencv-matrix
   rows: 3
   cols: 3
   dt: d
   data: [ 1000., 0., 320., 0., 1000., 240., 0., 0., 1. ]
distCoeffs: !!opencv-matrix
   rows: 5
   cols: 1
   dt: d
   data: [ 1.0000000000000001e-01, 1.0000000000000000e-02,
       -1.0000000000000000e-03, 0., 0. ]
features:
   - { x:103, y:166, lbp:[ 1, 0, 0, 1, 0, 1, 1, 0 ] }
   - { x:115, y:113, lbp:[ 1, 1, 1, 1, 1, 1, 1, 1 ] }
   - { x:586, y:12, lbp:[ 1, 0, 0, 1, 0, 1, 0, 0 ] }
```

OpenCV产生的yml文件中有两种collection，分别是mappings和sequences，mapping中的元素是有名字，名字对应其数值，其数值的获取是通过名字得到的，如上面`test.yml`文件所示的feature下对应的几个mappings。而sequences是没有名字的，通过索引获取数值的，例如上面的cameraMatrix里面的data对应的数值以及feature里面的lbp对应的数值以及feature本身。

当写入mapping元素的时候，使用`<<`依次输入数据的名字和数值，当输入sequences元素的时候，只需要依次输入数据的数值即可。

当写入mapping的时候，首先写入`{`然后依次写入各个mapping元素然，最后写入`}`。写入sequence的时候首先写入`[`，然后依次写入各个sequence元素，最后写入`]`。在`{`或者`[`后面加上`:`可以让这些数据呈inline模式显示，即一行，否则会每个元素都换行。

###XML/YML的读

文件的读的过程有三种方法，一种是和文件的写对应的，即使用`>>`读取，如下面的示例代码第8行；另一种是使用类型强制转换成相应数据，如下面实例代码第4行；最后一种是针对多个数据的读取，使用`FileNode`对应的迭代器`FileNodeIterator`进行逐个读取数据，如下面实例代码第28行。

```
FileStorage fs2("test.yml", FileStorage::READ);

// first method: use (type) operator on FileNode.
int frameCount = (int)fs2["frameCount"];

std::string date;
// second method: use FileNode::operator >>
fs2["calibrationDate"] >> date;

Mat cameraMatrix2, distCoeffs2;
fs2["cameraMatrix"] >> cameraMatrix2;
fs2["distCoeffs"] >> distCoeffs2;

cout << "frameCount: " << frameCount << endl
     << "calibration date: " << date << endl
     << "camera matrix: " << cameraMatrix2 << endl
     << "distortion coeffs: " << distCoeffs2 << endl;

FileNode features = fs2["features"];
FileNodeIterator it = features.begin(), it_end = features.end();
int idx = 0;
std::vector<uchar> lbpval;

// iterate through a sequence using FileNodeIterator
for( ; it != it_end; ++it, idx++ )
{
    cout << "feature #" << idx << ": ";
    cout << "x=" << (int)(*it)["x"] << ", y=" << (int)(*it)["y"] << ", lbp: (";
    // you can also easily read numerical arrays using FileNode >> std::vector operator.
    (*it)["lbp"] >> lbpval;
    for( int i = 0; i < (int)lbpval.size(); i++ )
        cout << " " << (int)lbpval[i];
    cout << ")" << endl;
}
fs.release();
```

之前一直使用libconfig作为参数配置的库，但是既然OpenCV也有这方面的功能，考虑到不需要安装额外的库，还是打算转向使用这个东西，这里做个简单介绍算是以后查询使用。