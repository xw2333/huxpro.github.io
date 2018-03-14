---
layout:     post
title:      "Altium Designer 器件封装"

date:       2018-03-14
author:     "XiongWei"
header-img: "img/post-bg-2018.jpg"
catalog: true
tags:
    - PCB
---
#  上一篇blog是一些绘制原理图时容易忽略的技巧，这篇则是封装的绘制

## 1，	切换到丝印层，然后以原点画一条线。

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/33.png)


## 2，	切换到mil单位，线宽为6mil。

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/34.png)

## 3，  再切换到mm单位，填写坐标为3.2mm。

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/35.png)


## 4，	Ctrl+M测量线的长度。



![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/36.png)

## 5，	放好丝印后，再放置焊盘，注意焊盘的形状（长方形），大小（比手册大一点）孔径（这里填0），和放置的层（顶层）。

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/37.png)


## 6，	调整跳动距离

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/38.png)

	Ctrl+上下左右可以微调焊盘位置。对称移动焊盘。

## 7，	丝印层的线不能挡住焊盘，所以移动玩焊盘后需要再去调整丝印。并且标注1号端口。

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/39.png)

## 8，一般可以从互联网上直接下载一些器件的封装，然后直接copy来，用ctrl+M来测量尺寸，看看是否满足我们的要求。

