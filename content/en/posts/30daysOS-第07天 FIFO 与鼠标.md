---
title: "30daysOS-第07天 FIFO 与鼠标"
date: 2021-03-29T11:20:00+08:00
tags: [学习笔记, 学习, 笔记]
draft: false
description: "30天自制操作系统"
source: "http://hrb.osask.jp/"
---
#操作系统 #30daysOS 

# 环形 FIFO

中断处理过程, 包含下面两个过程, 环形 FIFO 在这里起到了 缓存数据 / 解耦程序 / 消息订阅 的功能. 

**CPU处理中断**

- CPU 接收到中断
	- 中断处理开始
	- CPU 调用汇编程序
		- 汇编程序开始
		- 保存现场
		- 调用中断处理服务程序
			- 中断处理服务程序开始
			- 告诉 PIC 我会处理中断的
			- 读取键盘数据保存到 环形 FIFO
			- 中断处理服务程序结束
		- 恢复现场
		- 返回IRETD
		- 汇编程序结束
	- 中断处理结束

**主程序**

1. 设置中断标志=0, CPU 不接受任何中断
2. 如果环形 FIFO 是空的, 则设置中断标志=1, CPU 开始响应中断
3. 如果环形 FIFO 中有数据, 则处理数据 ( 这些数据可能是键盘输入鼠标移动 )
4. JUMP TO 1

# 初始化鼠标

鼠标的中断号码是: IRQ12

键盘的中断号码是: IRQ1

**初始化鼠标控制电路**

```c
#define PORT_KEYDAT				0x0060
#define PORT_KEYSTA				0x0064
#define PORT_KEYCMD				0x0064
#define KEYSTA_SEND_NOTREADY	0x02
#define KEYCMD_WRITE_MODE		0x60
#define KBC_MODE				0x47

void init_keyboard(void)
{
	/* 初始化键盘控制电路 */
	wait_KBC_sendready();  // 等待键盘可用
	io_out8(PORT_KEYCMD, KEYCMD_WRITE_MODE);  // 向键盘控制器发送指令: 0x60
	wait_KBC_sendready(); // 等待键盘可用
	io_out8(PORT_KEYDAT, KBC_MODE); // 向键盘控制器发送指令的参数:  0x47 (使用鼠标模式)
	return;
}
```


**激活鼠标**

```c
#define KEYCMD_SENDTO_MOUSE		0xd4
#define MOUSECMD_ENABLE			0xf4

void enable_mouse(void)
{
	/* マウス有効 */
	wait_KBC_sendready();
	io_out8(PORT_KEYCMD, KEYCMD_SENDTO_MOUSE); // 向键盘控制器发送指令: 0xd4 (发送数据到鼠标)
	wait_KBC_sendready();
	io_out8(PORT_KEYDAT, MOUSECMD_ENABLE); // 向键盘控制器发送指令: 0xf4 (开启鼠标)
	return; /* 鼠标收到后, 将会立刻发送ACK(0xfa)中断 */
}

```