---
title: "30daysOS-第02天 汇编语言和Makefile入门"
date: 2021-03-07T18:33:00+08:00
tags: [学习笔记, 学习, 笔记]
draft: false
description: "30天自制操作系统"
source: "http://hrb.osask.jp/"
---
#操作系统 #30daysOS 

# 目的

制作512k启动区.

## 汇编

![[Pasted image 20211121105805.png]]

**常用的16位寄存器**

CPU 的这些小弟都是16位的, 加在一起不过16个字节.

```ASM
AX —— accumulator  # X 的意思是 extend
CX —— counter
DX —— data
BX —— base. The base of memory that program can use.  
SP —— stack pointer. Point to stack of program, used in conjunction with SS.
BP —— base pointer. Point to stack segment
SI —— source index. Point to memory location in the data segment addressed by DS
DI —— destination index. Same to SI, access memory location address by ES
```

BX/BP/SI/DI can be used for read or write memory, e.g. `MOV AL, BYTE [BX]`

**8个8位寄存器**

注意啊, 这8个寄存器实际上使用的还是 AX / CX / DX/ BX, CPU 并没有因为这8个寄存器而多了8个字节.

```ASM
AL —— accumulator low
CL —— counter low
DL —— data low
BL —— base low
AH —— accumulator high
CH —— counter high
DH —— data high
BH —— base high
```

**32位寄存器**

在16位寄存器的基础上, 扩展了32位寄存器, 他们的低位使用的是上面的16位寄存器, 高位则没有相应的寄存器. 如果想用高位, 可以移位.

```ASM
EAX —— accumulator  # X 的意思是 extend
ECX —— counter
EDX —— data
EBX —— base
ESP —— stack pointer
EBP —— base pointer
ESI —— source index
EDI —— destination index
```

**段寄存器**

这些是16位段寄存器,

```ASM
ES —— extra segment
CS —— code segment  IP为指令指针寄存器。CPU将CS:IP指向的内容当作当前的指令。
SS —— stack segment
DS —— data segment
FS —— segment part 2
GS —— segment part 3
```

> 关于段寄存器的介绍: https://www.cxyzjd.com/article/hallyz945/88352520

## 伪指令

DB 指令是伪指令, 伪指令不会被编译为机器码, 而是由汇编器（MASM、TASM等）直接解释执行. 而 JMP 这样的指令是被编译成为机器码.

ORG 同样也是伪指令.

> https://zhuanlan.zhihu.com/p/139785404


## Makefile

Makefile 是 make 命令的配置文件, 可以执行批量处理.

```
# 定义 ipl.bin 文件
	../z_tools/nask.exe ipl.nas ipl.bin ipl.lst

# 定义 helloos.img 文件
		wbinimg src:ipl.bin len:512 from:0 to:0 imgout:helloos.img
```

> <span style='background-color:#ff0000;color:#fff;'>注意</span>: 缩进必须使用tab.


- 执行 `../z_tools/nask.exe ipl.nas ipl.bin ipl.lst` 生成相应 bin 和 lst 文件.

`make -r helloos.img` 的执行过程:

- 找到 `helloos.img`
- 找 `ipl.bin Makefile` 这两个文件, 发现缺少 `ipl.bin`
	- 继续找 `ipl.bin`
- 执行 `../z_tools/edimg.exe imgin:../z_tools/fdimg0at.tek 
		wbinimg src:ipl.bin len:512 from:0 to:0 imgout:helloos.img` 生成相应的 img 文件.

可以把其他的命令 例如: install / run 等等都加入到 Makefile 中.
```

default :
	../z_tools/make.exe img
ipl.bin : ipl.nas Makefile
	../z_tools/nask.exe ipl.nas ipl.bin ipl.lst
	
helloos.img : ipl.bin Makefile
	../z_tools/edimg.exe imgin:../z_tools/fdimg0at.tek \
		wbinimg src:ipl.bin len:512 from:0 to:0 imgout:helloos.img

# Command
	../z_tools/make.exe -r helloos.img
install :
	../z_tools/make.exe img
	../z_tools/imgtol.com w a: helloos.img

run :
	../z_tools/make.exe img
	copy helloos.img ..\z_tools\qemu\fdimage0.bin
	../z_tools/make.exe	-C ../z_tools/qemu
clean :
	-del ipl.bin
	-del helloos.img

```

现在可以通过 `make run` 和 `make clean` 命令来启动和清除文件了.



## 内存分配

### 启动区程序

将启动区程序装载到下面的地址中.

0x00007c00 ~ 0x00007dff

### 整体分配

开机时系统会以实模式进入，此时可访问的内存只有1M大小，这时的内存分配情况如下所示(此时由bios主导这一M内存的使用情况)：
```
0x 0 0 0 0 0
｜
｜ 10x64K=640K; 基本内存
0x 9 F F F F
0x A 0 0 0 0
｜
｜ 2x64K=128K;　　作为显存使用
｜　　　　　　　0xb0000-0xb8000 Mono text video buffer
｜　　　　　　　0xb8000-0xc0000 CGA/EGA+ chroma text video buffer
｜
｜
0x B F F F F
｜
｜ 4x64K=264K;　　由bios使用，地址如何利用由其自己决定
｜
```

### 基本内存分配

```
基本内存的分配方式如下(由bios分配):
0x 0 0 0 0 0
｜
0x 0 0 3 F F
｜
｜
0x 0 0 4 F F
0x 0 0 5 0 0
｜  0x8000 ~ 0x81ff  给启动区预留的内存
｜  0x8200 ~ 0x9fff   给应用使用
0x 9 F F F F
```


### BIOS使用部分
```
而通常情况下，bios对属于自己的地址空间的划分方式如下：
0x C 0 0 0 0
｜
｜　　　　　　　0.5X64k=32k; 显卡bios使用
｜
0x C 7 F F F
.
.
.
.
0x F 0 0 0 0
｜
｜ 1x64K=64K; 系统bios使用
｜
```

使用 `INT` 指令调用. 例如: [0x10 号函数](http://www.ablmcc.edu.hk/~scy/CIT/8086_bios_and_dos_interrupts.htm#int10h_0Eh) 是控制显卡的程序. 我们可以调用这个函数将字符显示在屏幕上.

```
mov ah, 0eh
mov bh, 0h
mov bl, 15
```
[附加程序](30daysOS-%E7%AC%AC02%E5%A4%A9%20%E6%B1%87%E7%BC%96%E8%AF%AD%E8%A8%80%E5%92%8CMakefile%E5%85%A5%E9%97%A8.md#%E9%99%84%E5%8A%A0%E7%A8%8B%E5%BA%8F) 中的 `putloop` 部分使用 10 号命令输出了一个字符串.


## 参考
[BIOS中断调用](https://zh.wikipedia.org/wiki/BIOS%E4%B8%AD%E6%96%B7%E5%91%BC%E5%8F%AB#BIOS_%E4%B8%AD%E6%96%B7%E5%90%91%E9%87%8F%E8%A1%A8)
[本课练习](https://gitee.com/lv-wei/os-30days/tree/main/tolset_h/helloos4)
[0x10 号函数](http://www.ablmcc.edu.hk/~scy/CIT/8086_bios_and_dos_interrupts.htm#int10h_0Eh) 

## 附加程序 boot.asm

启动过程:

1. BIOS 判断是否为引导盘, 如果不是则退出
2. 如果是, BIOS开始加载 boot 程序: 将第一个扇区(MBR)的内容读到内存中的 0x00007c00 ~ 0x00007dff 
3. 跳转到 boot 程序, 执行 boot 程序
4. boot 程序将 Loader.bin 程序加载到内存中的 0x1000
5. 跳转到 Loader程序. 执行 Loader 程序
6. Loader 将操作系统内核 Kernel.dll 加载到内存中, 启动 CPU 保护模式和分页机制.
7. 跳转到 Kernel.dll 入口函数开始执行

> https://zhuanlan.zhihu.com/p/32280478

下面的程序经过汇编编译器。会将下面的db指令的数据写到 img 文件中。同时将 cpu指令翻译成机器码，也写入到 img 文件当中。也就是说 img 文件当中存在两种数据，一种是cpu可以识别的机器指令，另一种是数据。然后将 img 写到磁盘, 用磁盘启动系统. 在系统启动的过程中，Bells可以。发现该磁盘是几斤到磁盘，然后将该磁盘的前512个字节，也就是第一扇区的数据加载到内存当中。这些数据包含了机器指令，例如jmp。Mov同时也包含了使用 DB 伪指令写入的数据。


```
; hello-os
; TAB=4

		ORG		0x7c00			; 本程序起始位置, 必须从这里开始. 这是伪指令.


		JMP		entry
		DB		0x90
		DB		"HELLOIPL"		; ブートセクタの名前を自由に書いてよい（8バイト）
		DW		512				; 1セクタの大きさ（512にしなければいけない）
		DB		1				; クラスタの大きさ（1セクタにしなければいけない）
		DB		2				; FATの個数（2にしなければいけない）
		DW		224				; ルートディレクトリ領域の大きさ（普通は224エントリにする）
		DW		2880			; このドライブの大きさ（2880セクタにしなければいけない）
		DB		0xf0			; メディアのタイプ（0xf0にしなければいけない）
		DW		9				; FAT領域の長さ（9セクタにしなければいけない）
		DW		2				; ヘッドの数（2にしなければいけない）
		DD		0				; パーティションを使ってないのでここは必ず0
		DD		2880			; このドライブ大きさをもう一度書く
		DB		0,0,0x29		; よくわからないけどこの値にしておくといいらしい
		DD		0xffffffff		; たぶんボリュームシリアル番号
		DB		"HELLO-OS   "	; ディスクの名前（11バイト）
		DB		"FAT12   "		; フォーマットの名前（8バイト）

; プログラム本体

entry:
		MOV		AX,0			; レジスタ初期化
		MOV		SS,AX
		MOV		SP,0x7c00
		MOV		DS,AX
		MOV		ES,AX

putloop:
		MOV		AL,[SI]         ; []意思是把内容作为内存地址, 取得其值, 这里的 SI 定义了输出内容的首地址
		ADD		SI,1			; SI加1, 即将指针移动到下一个内存地址, 为下一次循环做准备
		CMP		AL,0            ; 比较
		JE		fin             ; 如果 AL==0, 结束处理
		MOV		AH,0x0e			; 否则, 说明 AL 中有一个文字, 把AH设置为 0x0e, 准备输出 
		MOV		BX,15			; 设置输出文字的颜色
		JMP		putloop         ; 循环处理
fin:
		HLT						; 何かあるまでCPUを停止させる, CPU进入待机状态.
		JMP		fin				; 無限ループ

msg:
		DB		0x0a, 0x0a		; 改行を2つ
		DB		"hello, world"
		DB		0x0a			; 改行
		DB		0
		RESB	0x7dfe-$		; 0x7dfeまでを0x00で埋める命令, 这个也应该是伪指令.

		DB		0x55, 0xaa

; 以下はブートセクタ以外の部分の記述

		DB		0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
		RESB	4600
		DB		0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
		RESB	1469432

```