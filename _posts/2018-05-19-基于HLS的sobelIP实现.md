---
layout:     post
title:      "基于HLS的sobelIP实现"

date:       2018-05-19
author:     "XiongWei"
header-img: "img/post-bg-2018.jpg"
catalog: true
tags:
    - FPGA
---

以个人的理解，xilinx将HLS（高层次综合）定位于更方便的将复杂算法转化为硬件语言，通过添加某些配置条件HLS工具可以把可并行化的C/C++的代码转化为vhdl或verilog,相比于纯人工使用vhdl实现图像算法，该工具综合出的代码的硬件资源占用可能较多，但并没有相差太大。
但是却能提高我们的效率，缩短开发周期。
下面开始介绍我实现的一个sobel检测，可以把这个模块换成其它的各个加速算法，
Sobel 原理介绍
索贝尔算子（Sobel operator）主要用作边缘检测，在技术上，它是一离散性差分算子，用来
运算图像亮度函数的灰度之近似值。在图像的任何一点使用此算子，将会产生对应的灰度矢量或是
其法矢量
Sobel 卷积因子为：
![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-05-19/80.png)

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-05-19/81.png)


该算子包含两组 3x3 的矩阵，分别为横向及纵向，将之与图像作平面卷积，即可分别得出横向
及纵向的亮度差分近似值。如果以 A 代表原始图像， Gx 及 Gy 分别代表经横向及纵向边缘检测的图
像灰度值，其公式如下：

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-05-19/82.png)

Sobel 算子根据像素点上下、左右邻点灰度加权差，在边缘处达到极值这一现象检测边缘。对
噪声具有平滑作用，提供较为精确的边缘方向信息，边缘定位精度不够高。当对精度要求不是很高
时，是一种较为常用的边缘检测方法.
Sobel的实现在MATLAB和OpenCV的实现都是相对简单的，可是我们却无法保证执行的效率，众所周知，使用FPGA进行加速处理能对效率的提高带来显著的提升。
Sobel 算子在 HLS 上的实现
核心代码：

#include "top.h"

	void hls_sobel(AXI_STREAM& INPUT_STREAM, AXI_STREAM& OUTPUT_STREAM, int rows, int cols)
	{
    //Create AXI streaming interfaces for the core
		#pragma HLS INTERFACE axis port=INPUT_STREAM
		#pragma HLS INTERFACE axis port=OUTPUT_STREAM

		#pragma HLS RESOURCE core=AXI_SLAVE variable=rows metadata="-bus_bundle CONTROL_BUS"
		#pragma HLS RESOURCE core=AXI_SLAVE variable=cols metadata="-bus_bundle CONTROL_BUS"
		#pragma HLS RESOURCE core=AXI_SLAVE variable=return metadata="-bus_bundle CONTROL_BUS"

		#pragma HLS INTERFACE ap_stable port=rows
		#pragma HLS INTERFACE ap_stable port=cols

    	RGB_IMAGE img_0(rows, cols);
	    RGB_IMAGE img_1(rows, cols);
	    RGB_IMAGE img_2(rows, cols);
	    RGB_IMAGE img_3(rows, cols);
	    RGB_IMAGE img_4(rows, cols);
   		RGB_IMAGE img_5(rows, cols);
    	RGB_PIXEL pix(50, 50, 50);

		#pragma HLS dataflow
    	hls::AXIvideo2Mat(INPUT_STREAM, img_0);
   	 	hls::Sobel<1,0,3>(img_0, img_1);
    	hls::SubS(img_1, pix, img_2);
   	 	hls::Scale(img_2, img_3, 2, 0);
   	 	hls::Erode(img_3, img_4);
    	hls::Dilate(img_4, img_5);
    	hls::Mat2AXIvideo(img_5, OUTPUT_STREAM);
	}

我们基本用 hls 自带的视频处理库函数方向的边缘检测， ug902 技术手册中找到这些函数的介绍。

检测结果如下：

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-05-19/83.png)

在加入各种优化后，综合后内部的一些资源消耗。

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-05-19/84.png)

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-05-19/85.png)

于是将我们的sobel HLS封装成一个IP，然后在我们的zynq工程中去调用改IP，实现我们的加速算法。
整体框架图：

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-05-19/86.png)

硬件系统如此搭建，下面就是来驱动改hls的IP核。
1，首先在内存空间里定义一段内存：

	#define SOBEL_S2MM		0x08000000
	#define SOBEL_MM2S		0x0A000000

使用了两个 VDMA，其中 VDMA 只有一个写通道，负责完成显示的缓存功能， VDMA1 有两个通道，负责将我们取模的数组送入内存和 SOBEL
处理后的数据缓存的作用。
2，初始化显示vdma：

	vdmaConfig = XAxiVdma_LookupConfig(VGA_VDMA_ID);
	Status = XAxiVdma_CfgInitialize(&vdma, vdmaConfig, vdmaConfig->BaseAddress);

3，初始化显示的模块，

	Status = DisplayInitialize(&dispCtrl, &vdma, DISP_VTC_ID, DYNCLK_BASEADDR,&pFrames[2], 640*4);

&pFrames[2]就是我们需要显示的内存空间的起始地址。

4，以上是驱动负责显示的vdma，下面就是驱动sobel

	status = XHls_sobel_Initialize(&sobel,XPAR_XHLS_SOBEL_0_S_AXI_CONTROL_BUS_BASEADDR );
	vdmaConfig = XAxiVdma_LookupConfig(SOBEL_VDMA_ID);
	Status = XAxiVdma_CfgInitialize(&vdma, vdmaConfig, vdmaConfig->BaseAddress);

5，启动sobel：

	SOBEL_Setup();

在生成的bsp中能找到对应的驱动函数，实际上就是相应的寄存器写入数据。

	void SOBEL_Setup()
	{
	XHls_sobel_SetRows(&sobel, SOBEL_COL);
	XHls_sobel_SetCols(&sobel, SOBEL_ROW);
	XHls_sobel_DisableAutoRestart(&sobel);
	XHls_sobel_InterruptGlobalDisable(&sobel);
	SOBEL_VDMA_setting(SOBEL_ROW,SOBEL_COL,SOBEL_S2MM,SOBEL_MM2S);
	SOBEL_DDRWR(SOBEL_MM2S,SOBEL_ROW,SOBEL_COL);
	XHls_sobel_Start(&sobel);
	}

整个框架下来就是 VDMA1 将取模的数组刷入
内存，通过 VDMA1 的写通道送给 SOBEL 处理，然后将处理后的结果放在 VDMA1 的读通道中，然后再
操作 VDMA0 的写通道将 VDMA1 S2MM 里 SOBEL 的结果显示在屏幕上，思想比较简单易懂。
由于代码是仅仅对一个方向进行sobel检测，所以我便开始查阅hls手册，进行x，y连个方向检测，然后实现一个add。
修改代码如下：

	void hls_sobel(AXI_STREAM& INPUT_STREAM1,AXI_STREAM& OUTPUT_STREAM, int rows, int cols)
	{
    //Create AXI streaming interfaces for the core
		#pragma HLS INTERFACE axis port=INPUT_STREAM1
		#pragma HLS INTERFACE axis port=OUTPUT_STREAM

		#pragma HLS RESOURCE core=AXI_SLAVE variable=rows metadata="-bus_bundle CONTROL_BUS"
		#pragma HLS RESOURCE core=AXI_SLAVE variable=cols metadata="-bus_bundle CONTROL_BUS"
		#pragma HLS RESOURCE core=AXI_SLAVE variable=return metadata="-bus_bundle CONTROL_BUS"

		#pragma HLS INTERFACE ap_stable port=rows
		#pragma HLS INTERFACE ap_stable port=cols

   	 	RGB_IMAGE img_0(rows, cols);
   	 	RGB_IMAGE img_1(rows, cols);
		RGB_IMAGE img_2(rows, cols);
   	 	RGB_IMAGE img_3(rows, cols);
    	RGB_IMAGE img_4(rows, cols);
    	RGB_IMAGE img_5(rows, cols);
    	RGB_IMAGE img_6(rows, cols);
    	RGB_IMAGE img_7(rows, cols);
    	RGB_IMAGE img_8(rows, cols);
    	RGB_IMAGE img_9(rows, cols);
    	RGB_PIXEL pix(50, 50, 50);

		#pragma HLS dataflow
    	hls::AXIvideo2Mat(INPUT_STREAM1, img_0);
	/Copies the input image src to two output images dst1 and dst2, for divergent point of two datapaths.
    	hls::Duplicate(img_0,img_1,img_2);

    	hls::Sobel<0,1,3>(img_1, img_3);
    	hls::Sobel<1,0,3>(img_2, img_4);

    	hls::AddWeighted(img_3,1,img_4,1,0,img_5);

    	hls::SubS(img_5, pix, img_6);
    	hls::Scale(img_6, img_7, 2, 0);
    	hls::Erode(img_7, img_8);
   	 	hls::Dilate(img_8, img_9);
   	 	hls::Mat2AXIvideo(img_9, OUTPUT_STREAM);
	}
由于hls映射出来就是具体的电路，所以就不能只定义几个变量来存储对应的img，要看有几个过程，就定义几个RGB_IMAGE，每一次处理的结果都放到对应的RGB_IMAGE中去，简而言之就是在数据写入RGB_IMAGE之前，必须保证里面是empty。
这里用到了一个关键函数就是

	hls::Duplicate(img_0,img_1,img_2);

他就是Copies the input image src to two output images dst1 and dst2, for divergent point of two datapaths.
然后我又把数据加起来

	hls::AddWeighted(img_3,1,img_4,1,0,img_5);

然后进行腐蚀和膨胀。
综合如下：
因为又加入了y方向的检测，因此资源消耗可能会增加。

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-05-19/87.png)

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-05-19/88.png)

软件仿真结果如下：
![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-05-19/89.png)

硬件驱动HDMI如下：

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-05-19/90.jpg)

代码工程可以发我qq邮箱1010944344@qq.com索取