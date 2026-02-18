---
layout: post
title: "pwnable.kr -  coin1"
date: 2026-02-19
categories: pwnable wargame
---
# coin1
> 엄마! 게임을 하고싶어요!

# 문제 개요
* pwnable.kr - Toddler’s Bottle
* 5 point
* files: `readme`

# 프로그램 분석
프로그램 소스는 없습니다. 간단히 프로그램을 설명하자면, 정해진 기회내에 단 한개의 위조 동전의 위치를 원하는 만큼의 동전 무게를 재어서 찾아내야합니다.

# 설계 및 시나리오
문제를 대충 보고 떠오른것은 이진탐색을 이용하면 될것같다! 였습니다. 따라서 동전의 수에 $log_2$를 취하여 확인했더니 기회와 거의 동일한 수가 나와서 확신하고 알고리즘을 작성하였습니다.

# 익스플로잇
```
from pwn import *

r = remote("0", 9007)

r.recvuntil(b"- Ready? starting in 3 sec... -")
r.recvline()

for i in range(100):
    data = r.recvuntil(b"N=").decode()
    line = r.recvline().decode()
    
    parts = line.split()
    N = int(parts[0])
    C = int(parts[1].split('=')[1])
    print(f"Target: N={N}, C={C}")

    left = 0
    right = N - 1

    for i in range(C):
        mid = (left + right) // 2
        
        payload = " ".join(map(str, range(left, mid + 1)))
        r.sendline(payload.encode())
        
        res = r.recvline().decode().strip()

        weight = int(res)

        if weight % 10 != 0:
            right = mid
        else:
            left = mid + 1

    r.sendline(str(left).encode())
    result = r.recvline().decode().strip()
    print(f"Result: {result}")
r.interactive()
```
pwntool을 이용해 간단히 짜주었습니다!
# 결과
```
...(생략됨)
Target: N=276, C=9
Result: Correct! (99)
[*] Switching to interactive mode
Congrats! get your flag
b1naRy_S34rch1Ng_1s_3asy_p3asy
```