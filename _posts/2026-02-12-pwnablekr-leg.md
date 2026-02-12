---
layout: post
title: "pwnable.kr - leg"
date: 2026-02-12
categories: pwnable wargame
---
# leg
> 아빠가 ARM 아키텍쳐를 공부해야한다고 하셧어요. 하지만 전 인텔 아키텍쳐를 알고 둘은 비슷할거에요. 왜 귀찮게 ARM 아키텍쳐를 공부해요?

Download : http://pwnable.kr/bin/leg.c<br>
Download : http://pwnable.kr/bin/leg.asm

# 문제 개요
* pwnable.kr - Toddler's Bottle
* 5 point
* files: `flag`, `leg`, `leg.c`, `leg.asm`

# 프로그램 분석 
`key1`, `key2`, `key3`함수의 최종 반환 값들의 합을 구한 값과 입력을 비교하여 만약 같다면 값을 출력합니다.
```
#include <stdio.h>
#include <fcntl.h>
int key1(){
	asm("mov r3, pc\n");
}
int key2(){
	asm(
	"push	{r6}\n"
	"add	r6, pc, $1\n"
	"bx	r6\n"
	".code   16\n"
	"mov	r3, pc\n"
	"add	r3, $0x4\n"
	"push	{r3}\n"
	"pop	{pc}\n"
	".code	32\n"
	"pop	{r6}\n"
	);
}
int key3(){
	asm("mov r3, lr\n");
}
int main(){
	int key=0;
	printf("Daddy has very strong arm! : ");
	scanf("%d", &key);
	if( (key1()+key2()+key3()) == key ){
		printf("Congratz!\n");
		int fd = open("flag", O_RDONLY);
		char buf[100];
		int r = read(fd, buf, 100);
		write(0, buf, r);
	}
	else{
		printf("I have strong leg :P\n");
	}
	return 0;
}
```
```
(gdb) disass main
Dump of assembler code for function main:
   0x00008d3c <+0>:	push	{r4, r11, lr}
   0x00008d40 <+4>:	add	r11, sp, #8
   0x00008d44 <+8>:	sub	sp, sp, #12
   0x00008d48 <+12>:	mov	r3, #0
   0x00008d4c <+16>:	str	r3, [r11, #-16]
   0x00008d50 <+20>:	ldr	r0, [pc, #104]	; 0x8dc0 <main+132>
   0x00008d54 <+24>:	bl	0xfb6c <printf>
   0x00008d58 <+28>:	sub	r3, r11, #16
   0x00008d5c <+32>:	ldr	r0, [pc, #96]	; 0x8dc4 <main+136>
   0x00008d60 <+36>:	mov	r1, r3
   0x00008d64 <+40>:	bl	0xfbd8 <__isoc99_scanf>
   0x00008d68 <+44>:	bl	0x8cd4 <key1>
   0x00008d6c <+48>:	mov	r4, r0
   0x00008d70 <+52>:	bl	0x8cf0 <key2>
   0x00008d74 <+56>:	mov	r3, r0
   0x00008d78 <+60>:	add	r4, r4, r3
   0x00008d7c <+64>:	bl	0x8d20 <key3>
   0x00008d80 <+68>:	mov	r3, r0
   0x00008d84 <+72>:	add	r2, r4, r3
   0x00008d88 <+76>:	ldr	r3, [r11, #-16]
   0x00008d8c <+80>:	cmp	r2, r3
   0x00008d90 <+84>:	bne	0x8da8 <main+108>
   0x00008d94 <+88>:	ldr	r0, [pc, #44]	; 0x8dc8 <main+140>
   0x00008d98 <+92>:	bl	0x1050c <puts>
   0x00008d9c <+96>:	ldr	r0, [pc, #40]	; 0x8dcc <main+144>
   0x00008da0 <+100>:	bl	0xf89c <system>
   0x00008da4 <+104>:	b	0x8db0 <main+116>
   0x00008da8 <+108>:	ldr	r0, [pc, #32]	; 0x8dd0 <main+148>
   0x00008dac <+112>:	bl	0x1050c <puts>
   0x00008db0 <+116>:	mov	r3, #0
   0x00008db4 <+120>:	mov	r0, r3
   0x00008db8 <+124>:	sub	sp, r11, #8
   0x00008dbc <+128>:	pop	{r4, r11, pc}
   0x00008dc0 <+132>:	andeq	r10, r6, r12, lsl #9
   0x00008dc4 <+136>:	andeq	r10, r6, r12, lsr #9
   0x00008dc8 <+140>:			; <UNDEFINED> instruction: 0x0006a4b0
   0x00008dcc <+144>:			; <UNDEFINED> instruction: 0x0006a4bc
   0x00008dd0 <+148>:	andeq	r10, r6, r4, asr #9
End of assembler dump.
(gdb) disass key1
Dump of assembler code for function key1:
   0x00008cd4 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
   0x00008cd8 <+4>:	add	r11, sp, #0
   0x00008cdc <+8>:	mov	r3, pc
   0x00008ce0 <+12>:	mov	r0, r3
   0x00008ce4 <+16>:	sub	sp, r11, #0
   0x00008ce8 <+20>:	pop	{r11}		; (ldr r11, [sp], #4)
   0x00008cec <+24>:	bx	lr
End of assembler dump.
(gdb) disass key2
Dump of assembler code for function key2:
   0x00008cf0 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
   0x00008cf4 <+4>:	add	r11, sp, #0
   0x00008cf8 <+8>:	push	{r6}		; (str r6, [sp, #-4]!)
   0x00008cfc <+12>:	add	r6, pc, #1
   0x00008d00 <+16>:	bx	r6
   0x00008d04 <+20>:	mov	r3, pc
   0x00008d06 <+22>:	adds	r3, #4
   0x00008d08 <+24>:	push	{r3}
   0x00008d0a <+26>:	pop	{pc}
   0x00008d0c <+28>:	pop	{r6}		; (ldr r6, [sp], #4)
   0x00008d10 <+32>:	mov	r0, r3
   0x00008d14 <+36>:	sub	sp, r11, #0
   0x00008d18 <+40>:	pop	{r11}		; (ldr r11, [sp], #4)
   0x00008d1c <+44>:	bx	lr
End of assembler dump.
(gdb) disass key3
Dump of assembler code for function key3:
   0x00008d20 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
   0x00008d24 <+4>:	add	r11, sp, #0
   0x00008d28 <+8>:	mov	r3, lr
   0x00008d2c <+12>:	mov	r0, r3
   0x00008d30 <+16>:	sub	sp, r11, #0
   0x00008d34 <+20>:	pop	{r11}		; (ldr r11, [sp], #4)
   0x00008d38 <+24>:	bx	lr
End of assembler dump.
(gdb) 
```
# 설계 및 시나리오
우선 소스들에 겉보기에는 반환값이 없으니 gdb disass를 봅시다..

### 사전 지식
우선 대부분의 함수에서 공통적으로 시작부분에 다음이 사용됩니다.
```
push	{r11}
add	r11, sp, #0 
```
r11은 rbp정도로 생각하면됩니다 sp는 rsp입니다.
`add	r11, sp, #0`은 조금 햇갈릴 수있지만 `mov rbp, rsp`와 동일합니다.
따라서 처음에 함수 스택 프레임을 설정하는 로직입니다.
arm 아키텍쳐는 레지스터 0, 즉, r0을 반환값을 담는 레지스터로 사용합니다 rax와 같은 역할입니다!
그리고 key 1, 2, 3번 함수 모두 최종적으로 r0에 유효한 값을 담고 있으므로 반환값을 쉽게 추적할 수있습니다.
pc는 program counter 로써 rip와 비슷한 동작을 수행하지만 프로그램 실행 인출-해독-실행 중에서 인출쪽을 가르키므로 코드 주소에서 +0x4(Thumb 모드) 혹은 +0x8(ARM 모드)한 만큼을 가집니다.

### key1
```
0x00008cdc <+8>:mov	r3, pc
```
결국 모든 코드가 마지막에 `mov	r0, r3`을 포함하므로 우선 asm 안에 있는 코드만 분석하면 됩니다.
pc를 r3으로 옮기네요! 4바이트 단위로 명령어가 쓰인 ARM 모드이므로 최종적으로 0x00008cdc + 0x8을 반환 하겠네요!
### key2
```
   0x00008cf8 <+8>:	push	{r6}
   0x00008cfc <+12>:	add	r6, pc, #1
   0x00008d00 <+16>:	bx	r6
   0x00008d04 <+20>:	mov	r3, pc
   0x00008d06 <+22>:	adds	r3, #4
   0x00008d08 <+24>:	push	{r3}
   0x00008d0a <+26>:	pop	{pc}
   0x00008d0c <+28>:	pop	{r6}
```
r6에 pc + 1만큼을 넣은후 bx라는 명령어를 수행합니다.
왜 이 동작을 수행하는지 이해하기 위해서는 thumb mode 전환에 대해 알 필요가 있는데, ARM에서는 bx 명령어를 수행할때 점프할 주소의 LSB가 1, 즉, 홀수라면 -1한다음 그 주소로 점프하여 thumb mode로 전환합니다.
그리고 pc에는 thumb mode이므로 0x00008d04 + 0x4가 담기게 됩니다.
이후 0x00008d04 + 0x4 + 0x4를 수행하여 thumb mode를 탈출합니다. 
최종적으로 0x8D0C가 담기겠네요!
### key3
```
   0x00008d28 <+8>:	mov	r3, lr
   0x00008d2c <+12>:	mov	r0, r3
   
   Meanwhile...(caller)
   0x00008d7c <+64>:	bl	0x8d20 <key3>
   0x00008d80 <+68>:	mov	r3, r0
```
lr은 linkage register로 return address를 나타냅니다! 따라서 caller의 다음주소를 보면되겠네요! 0x00008d7c에서 호출했으니, 그 다음인 0x00008d80이 답입니다!



# 익스플로잇
$$
8D0C_{16} + 8D80_{16} + 8CE4_{16} = 1A770_{16} \\\\
1A770_{16} = 108400_{10}
$$

# 결과
```
/ $ ./leg
Daddy has very strong arm! : 108400
Congratz!
daddy_has_lot_of_ARM_muscl3
```
ARM instruction set을 공부해보고 싶게 만드는 문제였습니다!