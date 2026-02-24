
---
layout: post
title: "pwnable.kr - cmd2"
date: 2026-02-24
categories: pwnable_kr command_injection
---
# cmd2

> Daddy bought me a system command shell. but he put some filters to prevent me from playing with it without his permission... but I wanna play anytime I want!

# 문제 개요
* pwnable.kr
* 9 points
* files: cmd2, cmd2.c, flag

# 프로그램 분석
c
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
    for(p=environ; *p; p++) memset(*p, 0, strlen(*p));
}

int main(int argc, char* argv[], char** envp){
    delete_env();
    putenv("PATH=/no_command_execution_until_you_become_a_hacker");
    if(filter(argv[1])) return 0;
    printf("%s\
", argv[1]);
    setregid(getegid(), getegid());
    system( argv[1] );
    return 0;
}

입력된 `argv[1]`은 `filter` 함수를 통해 특정 문자열(`=`, `PATH`, `export`, `/`, `` ` ``, `flag`)을 포함하는지 검사합니다. 특히 `/` 문자가 필터링되어 절대 경로를 이용한 명령어 실행이 제한됩니다. `PATH` 환경 변수도 `no_command_execution_until_you_become_a_hacker`로 설정되어 있습니다. 필터링된 후 `system(argv[1])`을 통해 명령어가 실행되므로, 필터를 우회하여 `/` 문자를 포함하는 명령어를 주입하는 것이 목표입니다.

# 설계 및 시나리오
프로그램은 `/` 문자를 필터링하고 `PATH` 환경 변수를 제한하여 절대 경로 및 일반적인 명령어 실행을 어렵게 합니다. 하지만 `system()` 함수를 사용하므로 셸 인젝션 취약점이 존재합니다.
이 취약점을 이용하여 `/` 필터를 우회하고 플래그를 획득하기 위한 시나리오는 다음과 같습니다.
1.  먼저 `flag` 파일에 접근할 수 있는 디렉토리(예: `/tmp`)에서 `ln -s flag ddong` 명령어를 실행하여 `flag` 파일의 심볼릭 링크 `ddong`을 생성합니다. 이렇게 하면 `flag` 문자열 필터를 우회할 수 있습니다.
2.  `cat` 명령어를 현재 디렉토리(예: `/tmp`)로 복사하여 (`cp /bin/cat .`) `PATH` 제한에 상관없이 `cat`을 실행할 수 있도록 준비합니다.
3.  `/` 문자가 필터링되므로, 셸의 `printf` 명령어를 사용하여 ASCII octal 값 `\\057`로 `.`과 `cat` 사이에 `/`를 삽입합니다. 즉, `.\\$(printf '\\057')cat ddong` 형태의 명령어를 구성합니다.
4.  이 명령어는 `system()` 함수로 전달될 때, 셸에 의해 `$(printf '\\057')`이 `/`로 해석되어 최종적으로 `./cat ddong` 명령어가 실행됩니다. 이를 통해 `ddong` (원본 `flag`) 파일의 내용을 출력하여 플래그를 획득할 수 있습니다.

# 익스플로잇
`/` 문자를 `printf '\\057'`로 우회하여 `cat` 명령어를 실행합니다. `flag` 파일은 `ddong`으로 심볼릭 링크한 후 `./cat ddong` 명령어를 구성하여 플래그를 읽습니다.
bash
/home/cmd2/cmd2 ".\\$(printf '\\057')cat ddong"


# 결과
bash
cmd2@ubuntu:/tmp/aa$ /home/cmd2/cmd2 ".\\$(printf '\\057')cat ddong"
.$(printf '\\057')cat ddong
Shell_variables_can_be_quite_fun_to_play_with!

