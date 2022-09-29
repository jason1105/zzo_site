---
title: "30daysOS-第22天 用C语言编写应用程序"
date: 2021-04-22T09:33:00+08:00
tags: [study, note, cs]
draft: false
description: "30天自制操作系统"
source: "http://hrb.osask.jp/"
---
#操作系统 #30daysOS 

C应用调用系统函数的四部分:

- C **应用程序** `hello.c`
- C 应用程序**链接文件** `a_nask.nas`, 用于调用下面的系统函数代理程序 `_asm_hrb_api` .
- 操作系统函数的**代理程序** ( `_asm_hrb_api` in `nasfunc.nas` that  was called by INT 0x0040)
- 操作**系统函数**(` hrb_api()` in console.c )

**执行C应用程序**

系统进程运行中, 在命令行输入待执行的应用程序 (例如: hello.hrb, 这个是由C编译而来), 下面是系统后续流程:

- 找到应用程序 (hello.hrb)
- 为应用分配内存空间: 代码段和数据段
- 将应用程序加载到代码段
- 将代码段和数据段注册到GDT
- 把代码段中的数据保存到数据段
- 调用 \_start_app
	- Function \_start_app {
		- 保存现场 ( 系统堆栈 )
		- 保存系统堆栈段寄存器`SS`和堆栈指针`ESP`到 `tss.esp0` ( CPU 借此可以禁止应用程序向系统段写入数据 )
		- 设置应用程序使用的段寄存器
		- 将应用程序的 `SS`, `ESP` 和 `CS`, `EIP` 入栈.
		- `RETF`, 将会跳转到C应用程序 (x86 规定不能使用 far-JMP 或 far-CALL 从操作系统调用应用程序段)
	- }


**C应用程序调用 API**

C 应用程序已经运行起来了, 下面调用系统API:

- C 应用程序调用 nas应用程序 \_api\_putchar (系统 API putchar 的门面)
	- Function \_api\_putchar {
		- 保存现场 (保存应用程序的现场)
		- 取得 C 传递过来的参数, 并设置到寄存器中
		- **使用 `INT` 调用中断 0x40** [^1]
			- Function asm_hrb_api {
				- 保护现场
				- **切换到系统段** (切换之前是应用的数据段.)
				- 调用 \_hrb\_api
				- 恢复现场
				- 返回 `IRETD`
			- }
		- 返回 `RET`
	- }


![[Pasted image 20210422134558.png]]

应用侧: C 应用程序调用汇编函数, 汇编函数使用 INT 调用
系统侧: 汇编函数通过 `INT` 指令调用API 处理程序

[^1]: 0x40 对应的是 asm_hrb_api, 在`IDT`中设置, asm_hrb_api 是操作系统提供的API的门面, 调用时需设置 EDX 寄存器, 用来标识具体要调用的功能. 例如 EDX=5 表示绘制窗口, 并返回窗口的句柄. 使用 INT 的好处是, 应用程序不需要知道系统API的地址.