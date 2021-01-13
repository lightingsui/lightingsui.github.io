---
title: 51单片机系列-LED
date: 2020-12-30 23:45:47
tags: 51单片机
categories: 51单片机
---
# 51单片机系列-LED

单片机点亮LED是一项基本的操作，也是入门操作，通过分析电路以及原理来点亮LED。

<img src="https://raw.githubusercontent.com/lightingsui/Pic/master/img/20201230233209.png" style="zoom:50%;" />

可以看到单片机的8个LED接在了P2口上，那么通过操作P2口的高低电平就可以点亮或者关闭LED。

当P2口输出低电平时，可以点亮LED，输出高电平时，会熄灭LED。那么接下来看一下代码

```c
#include <reg51.h>

sbit LED1 = P2^0;

void main() {
	while(1) {
		LED1 = 0;
	}
}
```

直接通过sbit定义一个单片机的引脚，然后通过赋值低电平使其点亮。

## 流水灯

流水灯就涉及到了位操作，即左移与右移，使用函数\_crol\_可以实现左移，使用函数\_cror\_可以实现右移动

但是，使用这两种函数就需要引入头文件#include <intrins.h>

代码如下：

```c
#include <reg51.h>
#include <intrins.h>

typedef unsigned char u8;
typedef unsigned int u16;

#define LED P2

void delay_ms(u16 time)
{
	while(time--);
}

void main()
{	
	
	while(1)
	{
		int i = 0,j = 0;
		LED = 0xfe;
		delay_ms(50000);
		
		for(; i < 3; i++) 
		{
			LED = _crol_(LED, 2);
			delay_ms(50000);
		}
		
		// 1111 1101
		LED = 0xfd;
		delay_ms(50000);
		
		for(; j < 3; j++) 
		{
			LED = _crol_(LED, 2);
			delay_ms(50000);
		}
	}

}
```


