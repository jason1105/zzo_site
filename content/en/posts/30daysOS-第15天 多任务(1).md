---
title: "30daysOS-第15天 多任务(1)"
date: 2021-04-03T16:49:00+08:00
tags: [学习笔记, 学习, 笔记]
draft: false
description: "30天自制操作系统"
source: "http://hrb.osask.jp/"
---
#操作系统 #30daysOS 

CPU 切换进程需要 0.0001s 左右, 这段时间用来切换上下文: CPU 将当前任务的上下文(寄存器的值)保存到内存中(保存上下文), 然后从内存中恢复另一个任务的上下文(也就是将另一组寄存器的值从内存恢复到寄存器中, 即恢复上下文). 

**TSS (Task status segment)  **

TSS 段寄存器就是用来保存和恢复上下文的. 有 16  位 和 32 位之分. 长度为 104 个字节.

![[Pasted image 20210406103119.png]]

TSS 中的 EIP (instruction pointer) 寄存器保存了下一条指令在内存中的地址. 随着程序执行, EIP 会自动累加. 

> JMP 指令实际上就是更改了 EIP 寄存器的内容. 但汇编中使用 `MOV EIP, 0x1234` 这样的写法是不行的, 只能用 far 模式的 JMP. 如果 **JMP 的地址是 TSS**, 那么就会进行任务切换.

TR 寄存器中保存的是当前正在执行的任务的 `任务 GDT 编号乘以8`. 使用 ltr 命令对 TR 进行赋值, JMP 的时候 TR 会自动保存. See bellow code `load_tr()`

```c
struct TASK *task_init(struct MEMMAN *memman)
{
	int i;
	struct TASK *task;
	struct SEGMENT_DESCRIPTOR *gdt = (struct SEGMENT_DESCRIPTOR *) ADR_GDT;
	taskctl = (struct TASKCTL *) memman_alloc_4k(memman, sizeof (struct TASKCTL));
	for (i = 0; i < MAX_TASKS; i++) {
		taskctl->tasks0[i].flags = 0;
		taskctl->tasks0[i].sel = (TASK_GDT0 + i) * 8;
		set_segmdesc(gdt + TASK_GDT0 + i, 103, (int) &taskctl->tasks0[i].tss, AR_TSS32);
	}
	task = task_alloc();
	task->flags = 2; /* 動作中マーク */
	taskctl->running = 1;
	taskctl->now = 0;
	taskctl->tasks[0] = task;
	load_tr(task->sel); /* Save GDT of current task, 
	                       the GDT use for saving TSS when switch task. */
	task_timer = timer_alloc();
	timer_settime(task_timer, 2);
	return task;
}
```

**多任务执行流程**

- 定义任务的 TSS, 并加入到 GDT
- 切换任务
	- 用 LTR 命令将任务编号装载到 TR (target register) (用汇编语言编写)
	- 使用 JMP 的 far 模式跳转指令 (用汇编语言编写) 跳转到 GDT 中定义的序号. 例如 `JMP 3*8:0`. 可以从 A 程序跳转到 B 程序, 再从 B 程序跳转到 A 程序, 也可以通过定时器中断在 AB 之间跳转.

> **JMP模式**
> 
> near 模式只改写 EIP, 例如:  `JMP 0x0000001b`
> 
> far 模式同时改写 CS, 例如:   `JMP DWORD 2 * 8 : 0x0000001b`  或者 `JMP FAR [ESP
> + 4]`

**任务切换自动化**

在定时器中断服务程序中实现任务交替执行.