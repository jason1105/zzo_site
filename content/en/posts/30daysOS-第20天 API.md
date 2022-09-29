---
title: "30daysOS-第20天 API"
date: 2021-04-19T13:47:00+08:00
tags: [study, note, cs]
draft: false
description: "30天自制操作系统"
source: "http://hrb.osask.jp/"
---
#操作系统 #30daysOS 


JMP use for running another code segment that no return.

CALL use for running another code segment that have a return .


- find app.nas
- load app.nas to memory that was registered with 0x1003 to segment registers.
- far-CALL app.nas



**Using INT to call a API**

- get interrupt No for API through registering API to IDT `set_gatedesc(idt + 0x40, (int)asm_cons_putchar, 2 * 8, AR_INTGATE32);`
- call API by sort of `MOV AL, 'A'` then ` INT 0x40`

```asm
_asm_cons_putchar:
		STI
		PUSH	1
		AND		EAX,0xff	; AHやEAXの上位を0にして、EAXに文字コードが入った状態にする。
		PUSH	EAX
		PUSH	DWORD [0x0fec]	; メモリの内容を読み込んでその値をPUSHする
		CALL	_cons_putchar
		ADD		ESP,12		; スタックに積んだデータを捨てる
		IRETD

```

We don't need to specify CS or DS, because `_asm_cons_putchar` have used same segment register with system.

```asm
; hlt.nas
[BITS 32]
		MOV		AL,'A'
		CALL    2*8:0xbe3 ; 2*8 is number of segment of system. 0xbe3 is the address of _asm_cons_putchar
fin:
		HLT
		JMP		fin

```

`hlt.nas` was loaded into specified memory address that defined in GDT with 0x1003.



