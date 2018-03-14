---
layout:     post
title:      "Altium Designer 原理图库绘制技巧"

date:       2018-03-14
author:     "XiongWei"
header-img: "img/post-bg-2018.jpg"
catalog: true
tags:
    - PCB
---
#  这篇blog是一些绘制原理图时容易忽略的技巧

## 1，	芯片供电端加*0.1uf*的滤波电容。
![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/11.png)


## 2，电源端加*10uf电解电容和0.1uf钽电容*的滤波电容。

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/12.png)

## 3，	网络标号热点连接成功时显示小*白点*，则成功。

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/13.png)


## 4，当手册写到有些引脚为浮空时（floating），我们将将其接一个0.1uf的电容，可以将一些干扰源引到地上。（如果出现问题也可不焊接方便电路调试）

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/14.png)

## 5，模拟地与数字地隔离，使用磁珠，类似电感，滤波。

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/15.png)

## 6，	电源隔离，不允许数字和模拟接一起，需要经过隔离电源模块。
如果标注Vcc模拟地vdd数字地隔离
手册会将模拟地和数字地标注隔离。

## 7，	核心器件FPGA的放置，从网上下载对应的器件库，直接拖出来partA，partB。。。。
然后分别对io端口和芯片配置端口进行绘制。最后分为两页进行。

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/16.png)

## 8，给FPGA芯片配置时，我们通过查找器件手册来获取引脚的配置信息

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/17.png)

图中相关引脚的配置信息，我们查阅Xilinx官网的芯片手册，找到对应手册，仔细阅读，得到相关信息。

## 9，画好原理图时，电阻电容还有器件都没有标号，所以我们需要使用快捷键T+A来实现给原理图做标注（annotate schematics），然后每一个器件都有标号，所以，之前我们都标为U？，R？

## 10，怎样把原理图里面的库抠下来，变成自己用的原理图库。

Design-〉MakeSchematic Library

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/18.png)

## 11，原理图画完，编译一下，如果没有反应说明没有问题。

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/19.png)


## 12，利用封装向导来画器件的封装

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/20.png)

	按键的2就是2D模型，3是3D模型。
	器件的参数就填写真实的参数，软件会自动为我们将焊盘留长，方便我们焊接。

### 1，电阻电容电感等两个引脚的使用CHIP 

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/21.png)


![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/22.png)


### 2，画完一个之后再向我们的库中添加器件。

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/23.png)


### 3，画有极性器件（二极管）封装时添加丝印层。

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/24.png)

### 4，双击中间网状可以修改3D封装，当调整完后，看不到确定键事，按tab键在enter来确认。

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/25.png)

### 5，在画封装时，一定要把原理图库的引脚的引脚和pcb库的引脚对应上。一遇到原理图库的引脚改动时，必须要进行更新原理图。

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/26.png)

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/27.png)

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/28.png)

	当datasheet只表明min而没有标出max时，一般加一个1/2左右足够。
	芯片背面放置一个大焊盘是GND，默认为芯片的一个引脚，只不过一般没有写出。


## 13, 可以将其他project里面的器件封装copy过来，

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/29.png)

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/30.png)

## 14, 当遇到芯片内部有的引脚是相连的，需要改动引脚封装

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/31.png)

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-03-14/32.png)