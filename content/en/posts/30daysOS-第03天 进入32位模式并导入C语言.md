---
title: "30daysOS-第03天 进入32位模式并导入C语言"
date: 2021-03-09T20:48:00+08:00
tags: [学习笔记, 学习, 笔记]
draft: false
description: "30天自制操作系统"
source: "http://hrb.osask.jp/"
---
#操作系统 #30daysOS 


## BIOS

cpu 32位模式不能调用 BIOS, bios使用16位机器语言写的.




## 引导区程序 (IPL)

IPL 指的是第0个磁头第0个柱面的第1个扇区.(第一个扇区是启动区), IPL 被自动加载并执行, 通过在IPL中指定我们自己的程序, 从而引导系统.


**段寄存器**

`MOV CX, [1234]` 等同于  `MOV CX, [DS:1234]` , 含义是: 读取 DS * 16 + 1234 这个地址的内容, 赋值给CX.  DS 是段寄存器. ES 也是段寄存器.

**读盘**

```
; 读盘

		MOV		AX,0x0820  ; 将数据装入该内存地址, 因为前面512个字节是启动区, 所以从 0x8200开始, ES是段寄存器, 使用的时候要乘以10倍(即一个数量级, 十进制)
		MOV		ES,AX   ; ES:BX 是读入数据的缓冲地址, 即保存读入数据的地址
		MOV		CH,0			; 柱面0
		MOV		DH,0			; 磁头0
		MOV		CL,2			; 扇区2 (第1个扇区是启动区, 这里读第二个)

		MOV		AH,0x02			; AH=0x02 : 读盘 <--- 02表示读盘
		MOV		AL,1			; 读取1个扇区
		MOV		BX,0    ; ES:BX 是读入数据的缓冲地址, 即保存读入数据的地址
		MOV		DL,0x00			; A驱动器
		INT		0x13			; ディスクBIOS呼び出し
		JC		error  ; 判断读盘的返回值(FLACS.CF), 0 正常, 1 错误
```


**复位**

```
; 复位

		MOV		AX,0x0820  ; 将数据装入该内存地址
		MOV		ES,AX   ; ES:BX 是读入数据的缓冲地址, 即保存读入数据的地址
		MOV		CH,0			; 柱面0
		MOV		DH,0			; 磁头0
		MOV		CL,2			; 扇区2 (第1个扇区是启动区, 这里读第二个)

		MOV		AH,0x00			; AH=0x00 : 复位  <--- 00表示复位
		MOV		AL,1			; 1个扇区
		MOV		BX,0    ; ES:BX 是读入数据的缓冲地址, 即保存读入数据的地址
		MOV		DL,0x00			; A驱动器
		INT		0x13			; ディスクBIOS呼び出し
		JC		error  ; 判断读盘的返回值(FLACS.CF), 0 正常, 1 错误
```

**JBE指令**

jump if below or equal

**读取10个柱面**

```
next:
        ; 遍历所有18个扇区.读取到内存
		MOV		AX,ES			; アドレスを0x200進める
		ADD		AX,0x0020  ; 缓存区指向下一个512数据块 (ES 是段寄存器, 用于计算内存地址)
		MOV		ES,AX			; ADD ES,0x020 という命令がないのでこうしている
		ADD		CL,1			; CLに1を足す, 指向下一个扇区1
		CMP		CL,18			; CLと18を比較
		JBE		readloop		; CL <= 18 だったらreadloopへ, 继续读取, 直到读完所有扇区
		; 遍历两个磁头, 继续读取所有扇区
		MOV		CL,1 ; 指向1号扇区
		ADD		DH,1 ; 移动到下一个磁头
		CMP		DH,2 
		JB		readloop		; DH < 2 だったらreadloopへ, 继续读取
		; 遍历10个柱面
		MOV		DH,0  ; 使用0号磁头
		ADD		CH,1  ; 移动到下一个柱面
		CMP		CH,CYLS
		JB		readloop		; CH < CYLS だったらreadloopへ, 继续读取
```


### memory

0x8000 ~ 0x81ff    IPL  引导区, 占512个字节.
0x8200 ~ 0x9fff     给应用使用的内存

### VRAM

There are areas used for VRAM in memory for different video modes.

0xa0000 ~ 0xaffff is used for video of 320 * 200 * 8.

## makefile

using variables in makefile.

```
TOOLPATH = ../z_tools/
MAKE     = $(TOOLPATH)make.exe -r
NASK     = $(TOOLPATH)nask.exe
EDIMG    = $(TOOLPATH)edimg.exe
IMGTOL   = $(TOOLPATH)imgtol.com
COPY     = copy
DEL      = del

# デフォルト動作

default :
	$(MAKE) img

# ファイル生成規則

ipl.bin : ipl.nas Makefile
	$(NASK) ipl.nas ipl.bin ipl.lst

haribote.img : ipl.bin Makefile
	$(EDIMG)   imgin:../z_tools/fdimg0at.tek \
		wbinimg src:ipl.bin len:512 from:0 to:0   imgout:haribote.img

# コマンド

asm :
	$(MAKE) ipl.bin

img :
	$(MAKE) haribote.img

run :
	$(MAKE) img
	$(COPY) haribote.img ..\z_tools\qemu\fdimage0.bin
	$(MAKE) -C ../z_tools/qemu

install :
	$(MAKE) img
	$(IMGTOL) w a: haribote.img

clean :
	-$(DEL) ipl.bin
	-$(DEL) ipl.lst

src_only :
	$(MAKE) clean
	-$(DEL) haribote.img

```



## haribote.sys

- write nas file - haribote.nas
- compile nas to sys - haribote.sys
- copy haribote.sys file to drive a that applied with img file which include boot sector.


notice 

- `0x002600` is where file name was saved in memory
- `0x004200` is where file content was saved in memory

### run app

; 読み終わったのでharibote.sysを実行だ！

		MOV		[0x0ff0],CH		; IPLがどこまで読んだのかをメモ
		JMP		0xc200


![[Pasted image 20210418212934.png]]