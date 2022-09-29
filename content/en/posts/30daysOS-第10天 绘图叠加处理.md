---
title: "30daysOS-第08天 鼠标控制 32位模式切换"
date: 2021-04-1T11:00:00+08:00
tags: [学习笔记, 学习, 笔记]
draft: false
description: "30天自制操作系统"
source: "http://hrb.osask.jp/"
---
#操作系统 #30daysOS 

# 内存管理(续)

## 舍入运算

**向上舍入**

```c
unsigned int memman_alloc_4k(struct MEMMAN *man, unsigned int size)
{
	unsigned int a;
	size = (size + 0xfff) & 0xfffff000; // 希望是 0x1000 的倍数, 向上取整, 0xfff是对边界的处理, 使用 size + 0x1000 也可以, 但可能造成浪费. 例如: (0x2000 + 0x1000) & 0xfffff000 = 0x3000, 本来申请0x2000就可以了, 现在多出了 0x1000 空间, 造成了浪费. 
	a = memman_alloc(man, size);
	return a;
}
```

**向下舍入**

```c
  size = size & 0xfffff000  // 这就可以了
```

**十进制的向下舍入**

`size = size / 10 * 10`

或者

`size = size - size % 10`

# 叠加

- 将所有图层保存在数组中
- 某个图层更新后, 按照数组中图层的高度, 从 0 开始全部重绘一遍

改进

- 某个图层更新后, 记录更新的画面区域
- 按照数组中图层的高度, 从 0 开始全部重绘一遍该画面区域