---
layout: post
title: "pwnable.kr - cmd1"
date: 2026-02-19
categories: pwnable wargame
---
# cmd1

> 엉마! 리눅스 PATH 환경변수가 뭐에요?

# 문제 개요

* pwnable.kr - Toddler's Bottle
* 1 point
* files: `cmd1`, `cmd1.c`, `readme`

# 프로그램 분석

제공된 `cmd1.c` 소스 코드를 확인하면 사용자로부터 인자(`argv[1]`)를 입력받아 `system()` 함수로 실행합니다. 하지만 실행 전 `filter` 함수를 통해 다음 키워드들을 검사합니다.

* `flag`
* `sh`
* `tmp`

만약 입력한 문자열에 위 단어가 포함되어 있으면 실행되지 않습니다. 또한, `putenv`를 통해 `PATH` 환경 변수를 `/thankyouverymuch`으로 초기화하여 `ls`, `cat` 같은 기본 명령어를 바로 사용할 수 없게 설정되어 있습니다.

# 설계 및 시나리오

1. **환경 변수 우회**: `PATH`가 초기화되었으므로 실행하려는 명령어의 **절대 경로**(`/bin/cat`)를 사용해야 합니다.
2. **필터링 우회 (Wildcard)**: `flag`라는 단어를 직접 쓸 수 없으므로, 쉘의 와일드카드(`*`) 문자를 활용합니다. `/bin/cat f*`와 같은 방식을 사용하면 `flag`라는 문자열을 직접 언급하지 않고도 파일을 읽을 수 있습니다.


# 익스플로잇

쉘에서 직접 인자를 전달하여 익스플로잇을 수행합니다.

```
./cmd1 "/bin/cat f*"

```

위의 방법 중 가장 간단한 와일드카드 방식을 사용하여 플래그를 획득합니다.

# 결과

```
cmd1@pwnable:~$ ./cmd1 "/bin/cat f*"
PATH_environment?_Now_I_really_g3t_it,_mommy!

```