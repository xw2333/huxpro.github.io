---
layout:     post
title:      "VIVADO和SDK联合调试"

date:       2018-04-20
author:     "XiongWei"
header-img: "img/post-bg-2018.jpg"
catalog: true
tags:
    - FPGA
---


 哈哈哈，终于在组会前一天把调试搞定了，本来还愁汇报什么呢？经过我几天来不吃不喝（当然健身房还是不能落下的，hhh），总算把zynq的软硬件调试搞通了，但是这些仅仅是对付一些一般复杂性的系统，当系统变得庞大时，可能里面需要更多的调试技巧，但是掌握zynq的调试技巧对zynq的开发来说，重要性不言而喻。废话不多说，下面进入正题：

# 一、这是该系统的Top框图。 #

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-04-19/50.png)

# 二、调试模型图 #


![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-04-19/51.png)


1，因为GPIO_LED是8位信号，如果只分配了4位，还有4位没有用到，编译会报错，所以要加入约束条件：
当未分配引脚报错时，加入以下的约束条件来生成bitstream。

	set_property SEVERITY {Warning} [get_drc_checks NSTD-1]
	set_property SEVERITY {Warning} [get_drc_checks UCIO-1]	

2，对MATH的reg0和reg1进行赋值，这是arm通过AXI总线来执行写操作。所以我们设置W_VAILD为触发信号。

如图：![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-04-19/52.bmp) 点击等待触发（）

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-04-19/53.png)

3，在SDK中设置断点，执行这两行代码：

	Xil_Out32(MATH_IP_BASE+MATH_REG0,0X42);
	Xil_Out32(MATH_IP_BASE+MATH_REG1,0X12);

这两句的意思就是对FPGA的MATH核两个寄存器写入数据

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-04-19/54.png)

4，放大的写axi总线的时序（发现地址和数据同时有效，因为独立的地址和数据通道。）

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-04-19/55.png)

5，将reg2的数据读出

	val = Xil_In32(MATH_IP_BASE+MATH_REG2);

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-04-19/56.png)

放大的读时序（当RVALID有效时RDATA总线上的数据就有效）

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-04-19/57.png)

6，对LED的操作。设置自动触发。

	XGpio_axi_WriteReg(XPAR_GPIO_LITE_ML_0,GPIO_LITE_ML_REG0,1<<i);

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-04-19/58.png)

***Warning：***
当设置多个触发条件时，需要设置OR触发条件（默认是AND，这样就不会显示波形）

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-04-19/59.png)

7，MATH核的功能：当sel有效时把虚拟输入放在总线上，反之是ip的执行输入

    ain_i <= ain_vio WHEN sel='1' ELSE ain; 
	bin_i <= bin_vio WHEN sel='1' ELSE bin;

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-04-19/60.png)

![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-04-19/61.png)


# 三、最后，读者可以实现一个8路输出的pwn波形，利用自定义ip #

以下是我核心代码和加入ila来观测到的波形图。

	Xil_Out32(XPAR_PWN_V1_0_0_BASEADDR,99);//pwm0 fre
	Xil_Out32(XPAR_PWN_V1_0_0_BASEADDR+4,10);//pwm0 wav

	Xil_Out32(XPAR_PWN_V1_0_0_BASEADDR+8,99);//pwm1 fre
	Xil_Out32(XPAR_PWN_V1_0_0_BASEADDR+12,20);//pwm1 wav

	Xil_Out32(XPAR_PWN_V1_0_0_BASEADDR+16,99);//pwm2 fre
	Xil_Out32(XPAR_PWN_V1_0_0_BASEADDR+20,40);//pwm2 wav

	Xil_Out32(XPAR_PWN_V1_0_0_BASEADDR+24,99);//pwm3 fre
	Xil_Out32(XPAR_PWN_V1_0_0_BASEADDR+28,80);//pwm3 wav


	Xil_Out32(XPAR_PWN_V1_0_0_BASEADDR+32,49);//pwm0 fre
	Xil_Out32(XPAR_PWN_V1_0_0_BASEADDR+36,10);//pwm0 wav

	Xil_Out32(XPAR_PWN_V1_0_0_BASEADDR+40,49);//pwm1 fre
	Xil_Out32(XPAR_PWN_V1_0_0_BASEADDR+44,20);//pwm1 wav

	Xil_Out32(XPAR_PWN_V1_0_0_BASEADDR+48,49);//pwm2 fre
	Xil_Out32(XPAR_PWN_V1_0_0_BASEADDR+52,30);//pwm2 wav

	Xil_Out32(XPAR_PWN_V1_0_0_BASEADDR+56,49);//pwm3 fre
	Xil_Out32(XPAR_PWN_V1_0_0_BASEADDR+60,40);//pwm3 wav


![](http://githubblogpic.oss-cn-huhehaote.aliyuncs.com/2018-04-19/62.png)

图中可以看出利用ip核实现了8路的pwn输出，无需cpu的参与，pl端自动输出，在大型系统中可以减少cpu的负载。

# 总结： #
自定义ip时，一般是新建一个工程，然后在里面打开tool->create and package IP 
选择AXI_LITE总线来生成模板，加入自己需要挂载总线上的ip，然后将寄存器和ip的输入输出线连接起来。
写完代码，一定要综合该IP，因为他不会自动报错，等加入到工程时就会出现一堆问题，所有一定要保证IP的正确性。
最后在执行一次tool->create and package IP 。这时选择的是Package your current project,并且包含.xci files。最后再添加到自己需要用的工程中去。

源代码可以发我邮箱1010944344@qq.com索取。
