---
title: "30daysOS-第05天 分割编译与中断"
date: 2021-03-28T17:19:00+08:00
tags: [学习笔记, 学习, 笔记]
draft: false
description: "30天自制操作系统"
source: "http://hrb.osask.jp/"
---
#操作系统 #30daysOS 

# 分段结构

为什么分段, [[30daysOS-第05天 结构体, 文字显示与GDT／IDT#GDT IDT]]

```
struct SEGMENT_DESCRIPTOR {
	short limit_low, base_low;
	char                   base_mid, access_right;
	char limit_high, base_high;
};
```

这个数据结构必须是8个字节, 这个是由CPU决定的.

**段地址**

段的起始地址, 大小为32位, 由三部分组成, 这样做的目的是为了与80286兼容.

- base_low   段的起始地址低16位
- base_mid  段的起始地址中间8位
- base_high  段的起始地址高8位

**段容量上限**

段的容量上限是20位, 由两个变量组成, 只能表示1MB, 但是如果可以修改它的单位就可以表示更多的了, 其单位根据段属性中的Gbit而定, 1的场合单位是 page, 1 page = 4KB, 容量上限: $4KB * 2^{20} = 4GB$.  0的场合单位是Byte, 容量上限是1MB.

- limit_low   段容量上限的低16位
- limit_high  段容量上限的高4位 (只用到了 limit_high 的低4位, 0000xxxx, limit_high 的高4位用于段属性)

**段属性**

段属性是12位二进制, 其中高4位存储在 limit_high 的高4位中.

- access_right   段属性的低8位
- limit_high    的高4位

段属性是由 limit_high 的高4位 连接上 access_right, 所以会表示为 xxxx0000xxxxxxxx

高4位的构成:  Gbit(0:MB, 1:Page(默认))\_D(0:16位模式, 1:32位模式(默认))\_x\_x

低8位:

```asm
00000000(0x00)	# 未使用
10010010(0x92)	# 系统用: 可读写, 不可执行
10011010(0x9a)	# 系统用: 只可执行
11110010(0xf2)	# 应用使用: 可读写, 不可执行
11111010(0xfa)	# 应用使用: 只可执行
```

CPU 有两种模式: 系统模式(ring0) 和 应用模式(ring3), 操作系统和应用程序使用不同的模式.

# 初始化 PIC

Programmable interrupt controller

PIC 是将8个中断信号集成为一个中断信号的装置. PIC 监视着输入管脚的中断信号, 一旦有信号, 就告诉 CPU, PIC 可以级联扩充.

![[Pasted image 20210328184604.png]]

编写中断处理程序:

```c
void inthandler21(int *esp)
/* PS/2キーボードからの割り込み */
{
	struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	// 覆盖上次的输出
	boxfill8(binfo->vram, binfo->scrnx, COL8_000000, 0, 0, 32 * 8 - 1, 15);
	// 本次输出
	putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, "INT 21 (IRQ-1) : PS/2 keyboard");
	for (;;) {
		io_hlt();
	}
}
```

中断处理完成后, 必须使用汇编中的 IRET 返回, 所以上面的处理程序需要在汇编中调用.

```asm
_asm_inthandler21:
		PUSH	ES
		PUSH	DS
		PUSHAD
		MOV		EAX,ESP
		PUSH	EAX
		MOV		AX,SS
		MOV		DS,AX
		MOV		ES,AX
		CALL	_inthandler21
		POP		EAX
		POPAD
		POP		DS
		POP		ES
		IRETD
```

CPU 执行完中断服务程序后, 返回触发中断的程序, 这一过程需要使用中断返回: 

- IRET   返回
- IRETW  返回字
- IRETD   返回双字

该指令执行的过程基本上是INT指令的逆过程，具体如下：  

- 从栈顶弹出内容送入IP；  
- 再从新栈顶弹出内容送入CS；  
- 再从新栈顶弹出内容送入标志寄存器；


INT 命令接收中断. `INT 0x20~0x2f` 对应 `IRQ 0~15`