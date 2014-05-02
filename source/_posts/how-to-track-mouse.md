---
title: 如何用OpenCV跟踪鼠标操作
date: '2014-05-01'
categories: ['Tips']
tags: [OpenCV, Mouse Control]

---

在视频第一帧手动标记出目标的位置是在线视觉跟踪中最基本的一个操作，实现这个操作需要检测鼠标的移动和点击事件。OpenCV提供了`setMouseCallback`这个函数来响应鼠标的动作，并返回鼠标在绑定窗口上的坐标位置。下面就这个函数的使用做一个简单的介绍。

首先，`setMouseCallback`的C++函数声明如下：
```
C++: void setMouseCallback(const string& winname, MouseCallback onMouse, void* userdata=0 )
```
* **winname**是窗口的名称
* **onMouse**是响应事件的函数
* **userdata**可选项，是用户提供给`onMouse`响应函数的指针。

<!--more-->
这个函数是负责绑定鼠标响应的窗口，这样函数就只会响应关于该窗口的鼠标操作。`onMouse`定义的针对鼠标不同操作，用户需要做出的操作。

`onMouse`响应函数的声明为;

```
void onMouse(int event, int x, int y, int flag, void *param)
```
* **event**是鼠标响应事件，分别是：
	+ EVENT_MOUSEMOVE		滑动
	+ EVENT_LBUTTONDOWN		左击
	+ EVENT_RBUTTONDOWN		右击
	+ EVENT_MBUTTONDOWN		中键点击
	+ EVENT_LBUTTONUP		左键放开
	+ EVENT_RBUTTONUP		右键放开
	+ EVENT_MBUTTONUP		中键放开
	+ EVENT_LBUTTONDBLCLK		左键双击
	+ EVENT_RBUTTONDBLCLK		右键双击
	+ EVENT_MBUTTONDBLCLK		中键双击
* **x**是鼠标在图像上的横坐标
* **y**是鼠标在图像上的纵坐标
* **flag**是标记鼠标某一event下，其附加的状态，比如EVENT_MOUSEMOVE下，flag为EVENT_FLAG_LBUTTON，左键拖拽状态。flag的状态有：
	+ EVENT_FLAG_LBUTTON		左键拖拽
	+ EVENT_FLAG_RBUTTON		右键拖拽
	+ EVENT_FLAG_MBUTTON		中键拖拽
	+ EVENT_FLAG_CTRLKEY		按住Ctrl不放
	+ EVENT_FLAG_SHIFTKEY		按住Shift不放
	+ EVENT_FLAG_ALTKEY		按住Alt不放
* **param**是用户传入的指针， 即`setMouseCallback`中的**userdata**。

下面给出一个画矩形框的代码实例来详细讲解。首先定义了一个`MouseCapture`的类，存放在`MouseCapture.h`中。定义的私有成员有存储的图像`img`，窗口的名称`winName`，所画方框`rect`的以及是否结束画框的标志符`isMarked`。成员函数分别是读取图像的函数`loadImg()`，画方框函数`drawRect()`以及鼠标响应事件函数`onMouse()`,注意这里`onMouse`定义为`static`成员函数，是因为`setMouseCallBack()`不是`MouseCapture`类的成员函数，如果onMouse为普通成员函数，则会因为函数声明不同而无法使用，而声明为`static`成员函数后，就可以直接在`MouseCapture::`作用域里直接访问该静态成员函数`onMouse`

```cpp MouseCapture.h
#ifndef MOUSECAPTURE_H
#define MOUSECAPTURE_H

#include "opencv2/highgui/highgui.hpp"
#include "opencv2/core/core.hpp"
#include <string>

class MouseCapture
{
public:
    MouseCapture();
    void loadImg(std::string imgFile);
    void drawRect();
    static void onMouse(int event, int x, int y, int flag, void *parma);

private:
    cv::Mat img;
    std::string winName;
    cv::Rect rect;
    bool isMarked;

};

#endif // MOUSECAPTURE_H
```
接下面列出类`MouseCapture`的具体定义，在文件`MouseCapture.cpp`中。`onMouse`定义了当鼠标左击时，记录下矩形左上角的点，当鼠标抬起时，记录下矩形的宽高，当鼠标滑动且左击拖拽的时候，记录下矩形的宽高。注意这里**flag**的使用。指针**param**是`setMouseCallBack()`调用的时候传入的**this**指针（参见50行），这个参数的传入负责把跟踪鼠标所画矩形的框的位置大小信息写入类的成员**rect**中。
```cpp MouseCapture.cpp
#include "MouseCapture.h"
#include <iostream>

using namespace cv;
using namespace std;

void MouseCapture::onMouse(int event, int x, int y, int flag, void *param)
{
    MouseCapture *MC = (MouseCapture *)param;
    switch(event)
    {
    case EVENT_LBUTTONDOWN:
        MC->rect.x = x;
        MC->rect.y = y;
        break;

    case EVENT_LBUTTONUP:
        MC->rect.width = x - MC->rect.x;
        MC->rect.height = y - MC->rect.y;
        break;

    case EVENT_RBUTTONDOWN:
        MC->isMarked = true;
        break;

    case EVENT_MOUSEMOVE:
        if(flag & EVENT_FLAG_LBUTTON)
        {
            MC->rect.width = x - MC->rect.x;
            MC->rect.height = y - MC->rect.y;
        }
        break;
    default:
        break;
    }

}

MouseCapture::MouseCapture():winName("MouseControl"),rect(Rect(0,0,0,0)),isMarked(false)
{}

void MouseCapture::loadImg(string imgFile)
{
    img = imread(imgFile);
}

void MouseCapture::drawRect()
{
    namedWindow(winName);
    setMouseCallback(winName, onMouse, this);
    Mat tmpImg;
    while(!isMarked)
    {
        tmpImg = img.clone();
        rectangle(tmpImg, rect, Scalar(0,0,255), 2);

        string str;
        stringstream strStream;
        strStream<<"Marked Region:"<<rect.x<<":"<<rect.y<<":"<<rect.width<<":"<<rect.height;
        str = strStream.str();
        putText(tmpImg, str, Point(20,20), CV_FONT_HERSHEY_COMPLEX, .8,Scalar(0,255,0),2);

        imshow(winName, tmpImg);
        if(waitKey(10) == 'q')
            break;
    }
    cout<<"Mark finished!"<<endl;
}
```
最后是主函数的调用：
```cpp main.cpp
#include <iostream>
#include "MouseCapture.h"

int main()
{
    MouseCapture MC;
    MC.loadImg("lena.jpg");
    MC.drawRect();
    return 0;
}
```
最后，对于`onMouse()`鼠标响应事件函数的声明问题还有另外一种解决方案，就是把`onMouse()`声明为类`MouseCapture`的友函数，而不把`onMouse`声明为类的成员函数。

	
		
		
	








