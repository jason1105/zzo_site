---
title: "30daysOS-第xx天 "
date: 2021-04-08T15:27:00+08:00
tags: [学习笔记, 学习, 笔记]
draft: false
description: "30天自制操作系统"
source: "http://hrb.osask.jp/"
---
#操作系统 #30daysOS 

# 任务自动切换

**多任务管理结构体**

定义一个任务池 `tasks0[]`, 这个任务池的作用是限制最大任务数量. 任务池中有 MAX 个任务, 这些任务是裸任务, 并未与应用程序绑定, 也无法执行. 把它们的 TSS 注册到 GDT 中.

定义一个任务队列 `*tasks[]`, 保存正在运行的任务.

# 任务休眠

- 主任务初始化硬件中断缓冲
- 主任务加入任务队列
- 子任务加入任务队列
- **Start:** 主任务停止中断, 判断中断缓冲中是否有数据, 如果缓冲是空的, 进入休眠 
- 子任务开启中断继续执行 ( Set Interrupt Flag using  )
- 并行处理继续执行
- 主任务继续执行, 处理硬件中断数据, 处理完毕后, 跳转到 **Start**

> 子程序如果需要响应中断, 需要和主程序共用中断缓冲区

**并行处理部分**

- 硬件发出中断
- CPU 调用中断处理
- 中断处理程序往中断缓冲区写数据, 如果中断缓冲区中有需要回复的任务, 则恢复任务(将主任务加入任务队列)

IF will be set to 1 when switch task because EFLAGS of TSS of every task was initialized to 0x00000202  in method task_alloc(), and `0x00000202` means `0000 0000 0000 0000 0000 0010 0000 0010`, in other words, the 9th bit.

![[Pasted image 20210414215134.png]]

```c
struct TASK *task_alloc(void)
{
	int i;
	struct TASK *task;
	for (i = 0; i < MAX_TASKS; i++) {
		if (taskctl->tasks0[i].flags == 0) {
			task = &taskctl->tasks0[i];
			task->flags = 1; /* 使用中マーク */
			task->tss.eflags = 0x00000202; /* IF = 1; <---- THIS */
			task->tss.eax = 0; /* とりあえず0にしておくことにする */
			task->tss.ecx = 0;
			task->tss.edx = 0;
			task->tss.ebx = 0;
			task->tss.ebp = 0;
			task->tss.esi = 0;
			task->tss.edi = 0;
			task->tss.es = 0;
			task->tss.ds = 0;
			task->tss.fs = 0;
			task->tss.gs = 0;
			task->tss.ldtr = 0;
			task->tss.iomap = 0x40000000;
			return task;
		}
	}
	return 0; /* もう全部使用中 */
}
```

# 优先级

**设定优先级**

- 在 TASK 结构体中加入 priority 变量, 1 最低, 10 最高.
- 在切换任务时, 将 timer 的超时时间设置为 priority, 这样一来, priority 越高的任务, 其超时间越久.

问题: 如果两个相同优先级的任务怎么办? 

**任务级别分组**


