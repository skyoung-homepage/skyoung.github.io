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




		
		
		
	








