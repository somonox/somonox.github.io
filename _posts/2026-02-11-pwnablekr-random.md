---
layout: post
title: "pwnable.kr - random"
date: 2026-02-11
categories: pwnable wargame
---
# random
> 아빠! 프로그래밍에서 랜덤값을 어떻게 사용하는지 가르쳐주세요!

# 문제 개요
* pwnable.kr - Toddler's Bottle
* 1 point
* files: `flag`, `random`, `random.c`

```
[*] '/home/random/random'
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        PIE enabled
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
```
# 프로그램 분석 
프로그램 자체는 굉장히 단순합니다.. key ^ 랜덤값이 cafebabe일 경우 flag를 출력합니다... 다행이도 key나 랜덤값 둘중하나만 알아도 다른쪽을 구할 수있다는 것입니다. 하지만 우리는 둘다 모릅니다. <br>
하지만 여기서 알아야할것은 컴퓨터 대부분의 난수 생성기는 유사 난수 생성기라는것입니다. 즉, seed값이 같으면 같은 시퀀스로 값이 출력됩니다! 표준 libc 구현을 기준으로 seed값은 1, 첫번째로 출력되는 값은 1804289383입니다.<br>
왜 그런지는  여기를 보시면됩니다
https://en.cppreference.com/w/c/numeric/random/rand
https://github.com/lattera/glibc/blob/master/stdlib/random_r.c
(__random_r 함수를 보시면 됩니다 thread safety와 관련하여 여러 문제 때문에 내부 구현이 분리되어있습니다. 그리고 기본적으로 TYPE_3의 가산 지연 피드백을 사용합니다!)

gdb로 확인해도 동일한 값이 출력됩니다!
```
...
   0x55f9763b6223 <main+26>    xor    eax, eax                        EAX => 0
   0x55f9763b6225 <main+28>    mov    eax, 0                          EAX => 0
   0x55f9763b622a <main+33>    call   rand@plt                    <rand@plt>
 ► 0x55f9763b622f <main+38>    mov    dword ptr [rbp - 0x1c], eax     [0x7fff9070af14] <= 0x6b8b4567
...(생략)
pwndbg> p $rax
$2 = 1804289383
```
```
#include <stdio.h>

int main(){
        unsigned int random;
        random = rand();        // random value!

        unsigned int key=0;
        scanf("%d", &key);

        if( (key ^ random) == 0xcafebabe ){
                printf("Good!\n");
                setregid(getegid(), getegid());
                system("/bin/cat flag");
                return 0;
        }

        printf("Wrong, maybe you should try 2^32 cases.\n");
        return 0;
}
```
# 설계 및 시나리오
이제 처음 랜덤 값을 알아 내었으니 xor의 가역성을 이용하여 key를 구할 수있습니다!
<br>
$$
key \oplus rand = \mathrm{0x}CAFEBABE\\\\
\mathrm{0x}CAFEBABE\oplus rand = key \\\\
\mathrm{0x}CAFEBABE\oplus 1804289383 = 2708864985
$$

# 익스플로잇
```
random@ubuntu:~$ python -c "print(0xcafebabe ^ 1804289383)"
2708864985
```

# 결과
```
random@ubuntu:~$ ./random
2708864985
Good!
m0mmy_I_can_predict_rand0m_v4lue!
```
ISO C가 돈받고 파는지 처음 알았습니다...