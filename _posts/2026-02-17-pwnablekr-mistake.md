---
layout: post
title: "pwnable.kr -  mistake"
date: 2026-02-17
categories: pwnable wargame
---

# mistake
> 우리는 모두 실수를 해요, 넘어 갑시다. (너무 진지하게 받아들이지 마세요. 화려한 해킹 스킬같은건 전혀 필요없습니다.)
이 문제는 실제 사건에 기반합니다.

# 문제 개요
* pwnable.kr - Toddler’s Bottle
* 4 point
* files: `flag`, `mistake`, `mistake.c`, `password`

# 프로그램 분석
입력을 xor로 암호화한 과정을 거친 이후, 그것을 passward에 들어있는 파일과 비교합니다. 그래야 하는데...
```
#include <stdio.h>
#include <fcntl.h>

#define PW_LEN 10
#define XORKEY 1

void xor(char* s, int len){
        int i;
        for(i=0; i<len; i++){
                s[i] ^= XORKEY;
        }
}

int main(int argc, char* argv[]){

        int fd;
        if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0){
                printf("can't open password %d\n", fd);
                return 0;
        }

        printf("do not bruteforce...\n");
        sleep(time(0)%20);

        char pw_buf[PW_LEN+1];
        int len;
        if(!(len=read(fd,pw_buf,PW_LEN) > 0)){
                printf("read error\n");
                close(fd);
                return 0;
        }

        char pw_buf2[PW_LEN+1];
        printf("input password : ");
        scanf("%10s", pw_buf2);

        // xor your input
        xor(pw_buf2, 10);

        if(!strncmp(pw_buf, pw_buf2, PW_LEN)){
                printf("Password OK\n");
                setregid(getegid(), getegid());
                system("/bin/cat flag\n");
        }
        else{
                printf("Wrong Password\n");
        }

        close(fd);
        return 0;
}
```
# 설계 및 시나리오
별다른 취약점이라고는 보이지가 않아, gdb를 이용해 값을 살펴 보던중, fd에 1이 로드 되는것을 보게 되었고, 이것을 조사해보니 이런 일이 발생하였습니다.
`fd=open("/home/mistake/password",O_RDONLY,0400) < 0`<br>
여기서 두개의 연산자가 사용되었습니다. 그리고 두 연산자중 `<`이 가장 우선 순위가 높습니다.
따라서 fd에 담기는건 파일 디스크립터가 아니고 성공 여부입니다. 성공하면 0이 될태니, 결과적으로 fd에는 `0`, `stdin`이 담깁니다.
그럼 원본 password 값을 원하는대로 바꿀 수있습니다.
# 익스플로잇
$$
'A' \oplus 1 = '@'
$$
따라서
@@@@@@@@@@을 입력하고, AAAAAAAAAA을 입력합니다.
# 결과
```
mistake@ubuntu:~$ ./mistake
do not bruteforce...
@@@@@@@@@@
input password : AAAAAAAAAA
Password OK
Mommy_the_0perator_priority_confuses_me
```