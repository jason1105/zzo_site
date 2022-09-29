---
title: "30daysOS-第04天 C语言与画面显示的练习"
date: 2021-03-26T10:47:00+08:00
tags: [学习笔记, 学习, 笔记]
draft: false
description: "30天自制操作系统"
source: "http://hrb.osask.jp/"
---
#操作系统 #30daysOS 

# C语言

**指针**

\*(p+i) 可以写成  p[i]

```
int i[4]  = {1,2,3,4};    // 这里的 i 可以理解为 int *i 里的 i. 值是内存地址.
printf("%d", i[0]);   // i[0]是 *(i+0) 的简写形式,  如果写成 0[i] 也可以
```

**结构体**

保存了启动信息

```
struct BOOTINFO {
	char cyls, leds, vmode, reserve; // 引导区字节 / 键盘灯 / 显示模式 / 保留
	short scrnx, scrny; // 分辨率
	char *vram; // 显存地址
};
```

# 调色板

屏蔽中断是什么意思?

```c
// 设置系统调色板
void set_palette(int start, int end, unsigned char *rgb)
{
	int i, eflags;
	eflags = io_load_eflags();	/* 保存中断标志位 */
	io_cli(); 					/* 禁止中断 */
	io_out8(0x03c8, start);   // 将起始调色板号写入0x03c8
	for (i = start; i <= end; i++) {     // 顺序写入颜色的rgb即可
		io_out8(0x03c9, rgb[0] / 4);  // 记录色号对应的r
		io_out8(0x03c9, rgb[1] / 4);  // 记录色号对应的g
		io_out8(0x03c9, rgb[2] / 4);  // 记录色号对应的b
		rgb += 3;
	}
	io_store_eflags(eflags);	/* 回复中断标志位 */
	return;
}
```


直接用机器语言就可以在屏幕上画矩形了. VRAM 是显示用的内存, 每个字节对应屏幕上的一个像素, 只要往 VRAM 里面写入一些内容, 画面上就能显示出来. 我们使用一个简单的汇编语言写的函数:

```asm
_write_mem8:	; void write_mem8(int addr, int data);
		MOV		ECX,[ESP+4]		; [ESP+4]中有addr, 读到 ECX 中.
		MOV		AL,[ESP+8]		; [ESP+8]中有 data, 读到 AL中
		MOV		[ECX],AL
		RET
```

# 汇编

**CLI**

Clear interrupt flag 

清除中断标志, 即将中断标志置0, CPU 将会忽略所有中断.

**STI**

set interrupt flag 

设置中断标志, 即将中断标志置1, CPU 将会处理所有中断.

**EFLAGS 标志寄存器**

![[Pasted image 20210403161531.png]]