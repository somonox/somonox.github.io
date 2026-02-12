---
layout: post
title: "pwnable.kr - input"
date: 2026-02-12
categories: pwnable wargame
---
# input2
> 엄마! 제 입력을 어떻게 컴퓨터에게 줄 수 있어요?

# 문제 개요
* pwnable.kr - Toddler's Bottle
* 10 point
* files: `flag`, `input2`, `input2.c`

# 프로그램 분석 
프로그램 인자, 표준 입출력, 환경변수, 파일, 네트워크를 이용하여 사용자의 입력을 받고 모두 검증 성공하면 flag를 내주는 방식입니다. 파이썬을 사용해서 차근 차근 만족시키면 되겠군요.
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>

int main(int argc, char* argv[], char* envp[]){
        printf("Welcome to pwnable.kr\n");
        printf("Let's see if you know how to give input to program\n");
        printf("Just give me correct inputs then you will get the flag :)\n");

        // argv
        if(argc != 100) return 0;
        if(strcmp(argv['A'],"\x00")) return 0;
        if(strcmp(argv['B'],"\x20\x0a\x0d")) return 0;
        printf("Stage 1 clear!\n");

        // stdio
        char buf[4];
        read(0, buf, 4);
        if(memcmp(buf, "\x00\x0a\x00\xff", 4)) return 0;
        read(2, buf, 4);
        if(memcmp(buf, "\x00\x0a\x02\xff", 4)) return 0;
        printf("Stage 2 clear!\n");

        // env
        if(strcmp("\xca\xfe\xba\xbe", getenv("\xde\xad\xbe\xef"))) return 0;
        printf("Stage 3 clear!\n");

        // file
        FILE* fp = fopen("\x0a", "r");
        if(!fp) return 0;
        if( fread(buf, 4, 1, fp)!=1 ) return 0;
        if( memcmp(buf, "\x00\x00\x00\x00", 4) ) return 0;
        fclose(fp);
        printf("Stage 4 clear!\n");

        // network
        int sd, cd;
        struct sockaddr_in saddr, caddr;
        sd = socket(AF_INET, SOCK_STREAM, 0);
        if(sd == -1){
                printf("socket error, tell admin\n");
                return 0;
        }
        saddr.sin_family = AF_INET;
        saddr.sin_addr.s_addr = INADDR_ANY;
        saddr.sin_port = htons( atoi(argv['C']) );
        if(bind(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0){
                printf("bind error, use another port\n");
                return 1;
        }
        listen(sd, 1);
        int c = sizeof(struct sockaddr_in);
        cd = accept(sd, (struct sockaddr *)&caddr, (socklen_t*)&c);
        if(cd < 0){
                printf("accept error, tell admin\n");
                return 0;
        }
        if( recv(cd, buf, 4, 0) != 4 ) return 0;
        if(memcmp(buf, "\xde\xad\xbe\xef", 4)) return 0;
        printf("Stage 5 clear!\n");

        // here's your flag
        setregid(getegid(), getegid());
        system("/bin/cat flag");
        return 0;
}
```
# 설계 및 시나리오
### 1. Program argument
우선 프로그램이 원격에 있으면 불편하므로 scp를 사용해 가져오겠습니다.
```
scp -P 2222 input2@pwnable.kr:/home/input2/input2 .
```
이런 종류의 문제를 푸는데 탁월한 파이썬 패키지인 `subprocess`를 이용하겠습니다. <br>
우선, 첫번째로 argument가 100개 여야합니다
그리고 argc(argument count)는 자기 자신이 포함되므로, 99개의 인자가 들어가야겠죠. <br>
그리고 'A'번째의 argument는 `\x00`, 'B'번째의 argument는 `\x20\x0a\x0d`가 되어야합니다.
```
import subprocess

args = ['./input2'] + [b'A'] * 99
args[ord('A')] = b''
args[ord('B')] = b'\x20\x0a\x0d'

subprocess.call(args)
```
이렇게 실행한다면 1단계는 클리어 입니다!
### 2. Standard Input Output
`"\x00\x0a\x00\xff"`와 `"\x00\x0a\x02\xff"`를 전달 해야합니다 하지만 subprocess 파이프를 사용한 stderr 쓰기는 제한되기 때문에 os를 사용하여 pipe를 연결하면됩니다.
```
from subprocess import *
import os

args = ['./input2'] + [b'A'] * 99
args[ord('A')] = b''
args[ord('B')] = b'\x20\x0a\x0d'

r, w = os.pipe()

p1 = Popen(args, stdin=PIPE, stderr=r)
os.write(w, b"\x00\x0a\x02\xff")
p1.stdin.write(b"\x00\x0a\x00\xff")
p1.stdin.flush()

os.close(r)
os.close(w)
```
### 3. ENV
0xdeadbeef를 키로, 0xcafebabe를 값으로 하여 env를 주입하면됩니다.
```
from subprocess import *
import os

args = ['./input2'] + [b'A'] * 99
args[ord('A')] = b''
args[ord('B')] = b'\x20\x0a\x0d'

r, w = os.pipe()

my_env = {}
my_env[b"\xde\xad\xbe\xef"] = b"\xca\xfe\xba\xbe"

p1 = Popen(args, stdin=PIPE, stderr=r, env=my_env)
os.write(w, b"\x00\x0a\x02\xff")
p1.stdin.write(b"\x00\x0a\x00\xff")
p1.stdin.flush()

os.close(r)
os.close(w)

```

### 4. File
홈화면에는 파일을 만들권한이 없으므로 /tmp를 사용해 만들어준 다음 조건을 만족하기 위해 null byte를 4개 넣어놓읍시다
```
from subprocess import *
import os
import time

tmp_dir = "/tmp/faah"
if not os.path.exists(tmp_dir):
    os.makedirs(tmp_dir)

os.chdir(tmp_dir)

args = ['/home/input2/input2'] + [b'A'] * 99
args[ord('A')] = b''
args[ord('B')] = b'\x20\x0a\x0d'
args[ord('C')] = "6767"

file_name = "\x0a"

with open(file_name, "wb") as f:
    f.write(b"\x00\x00\x00\x00")

r, w = os.pipe()
my_env = os.environ.copy()
my_env[b"\xde\xad\xbe\xef"] = b"\xca\xfe\xba\xbe"

p1 = Popen(args, stdin=PIPE, stderr=r, env=my_env, stdout=PIPE)
os.write(w, b"\x00\x0a\x02\xff")
p1.stdin.write(b"\x00\x0a\x00\xff")
p1.stdin.flush()

os.close(r)
os.close(w)
```
### 5. Socket communication
마지막 단계입니다 'C'번째 arg에서 포트를 읽어오므로 아무도 쓰지 않을거 같은 포트를 넣은 다음 조건을 만족하기 위해 연결을 맺은 후, deadbeef를 전송합니다
```
from subprocess import *
from socket import *
import os
import time

host = "0.0.0.0"
port = 6767

tmp_dir = "/tmp/faah"
if not os.path.exists(tmp_dir):
    os.makedirs(tmp_dir)

os.chdir(tmp_dir)

args = ['/home/input2/input2'] + [b'A'] * 99
args[ord('A')] = b''
args[ord('B')] = b'\x20\x0a\x0d'
args[ord('C')] = "6767"

file_name = "\x0a"

with open(file_name, "wb") as f:
    f.write(b"\x00\x00\x00\x00")

r, w = os.pipe()
my_env = os.environ.copy()
my_env[b"\xde\xad\xbe\xef"] = b"\xca\xfe\xba\xbe"

p1 = Popen(args, stdin=PIPE, stderr=r, env=my_env, stdout=PIPE)
os.write(w, b"\x00\x0a\x02\xff")
p1.stdin.write(b"\x00\x0a\x00\xff")
p1.stdin.flush()

os.close(r)
os.close(w)

time.sleep(0.5)

clientSocket = socket(AF_INET, SOCK_STREAM)
clientSocket.connect((host ,port))
clientSocket.send(b"\xde\xad\xbe\xef")

out, err = p1.communicate()

print(out.decode())
```

# 익스플로잇
```
input2@ubuntu:/tmp/faah$ vim exploit.py
input2@ubuntu:/tmp/faah$ python3 exploit.py
```

# 결과
```
input2@ubuntu:/tmp/faah$ python3 exploit.py
Mommy_now_I_know_how_to_pa5s_inputs_in_Linux
Welcome to pwnable.kr
Let's see if you know how to give input to program
Just give me correct inputs then you will get the flag :)
Stage 1 clear!
Stage 2 clear!
Stage 3 clear!
Stage 4 clear!
Stage 5 clear!
```
오타만 없었으면 쉬운 문제!