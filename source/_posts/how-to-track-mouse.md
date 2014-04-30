---
title: 如何用OpenCV跟踪鼠标操作
date: '2014-05-01'
categories: ['Tips']
tags: [OpenCV, Mouse Control]

---

因为经常用到手动在图像上标记物体的操作，这里简单的介绍一些OpenCV中是如何实现鼠标响应信息的。OpenCV提供了`setMouseCallback`这个函数来响应鼠标的动作。其C++函数声明如下：
```
C++: void setMouseCallback(const string& winname, MouseCallback onMouse, void* userdata=0 )
```
* winname是窗口的名称
* onMouse是响应事件的函数
* userdata可选项，是用户提供给onMouse响应函数的指针。

onMouse响应函数的声明为;

```
void onMouse(int event, int x, int y, int flag, void *prama)
```
* event是鼠标响应事件，分别是：
	+ EVENT_MOUSEMOVE
	+ EVENT_LBUTTONDOWN
	+ EVENT_RBUTTONDOWN
	EVENT_MBUTTONDOWN
	EVENT_LBUTTONUP
	EVENT_RBUTTONUP	
	EVENT_MBUTTONUP
	EVENT_LBUTTONDBLCLK
	EVENT_RBUTTONDBLCLK
	EVENT_MBUTTONDBLCLK

鼠标动作主要有三个：点击(click)，松开(down)，滑动(move)。则onMouse函数对应的




#include "MouseCapture.h"
#include <iostream>

using namespace cv;
using namespace std;

void onMouse(int event, int x, int y, int flag, void *parma)
{
    MouseCapture *MC = (MouseCapture *)parma;
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

int MouseCapture::loadImg(string imgFile)
{
    img = imread(imgFile);
    return 0;
}

int MouseCapture::drawRect()
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
    return 0;
}


#include <iostream>
#include "MouseCapture.h"


int main()
{
    MouseCapture MC;
    MC.loadImg("lena.jpg");
    MC.drawRect();

    return 0;
}


#ifndef MOUSECAPTURE_H
#define MOUSECAPTURE_H

#include "opencv2/highgui/highgui.hpp"
#include "opencv2/core/core.hpp"
#include <string>

class MouseCapture
{
public:
    MouseCapture();
    int loadImg(std::string imgFile);
    int drawRect();
    friend void onMouse(int event, int x, int y, int flag, void *parma);

private:
    cv::Mat img;
    std::string winName;
    cv::Rect rect;
    bool isMarked;

};


#endif // MOUSECAPTURE_H
	
		
		
	







