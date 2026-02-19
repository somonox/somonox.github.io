---
layout: post
title: "pwnable.kr - lotto"
date: 2026-02-19
categories: pwnable wargame
---
# lotto
> 엄마! 숙제로 로또 프로그램을 만들었는데 해보실레요?

# 문제 개요
* pwnable.kr - Toddler's Bottle
* 2 point
* files: `flag`, `lotto`, `lotto.c`

# 프로그램 분석
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>

unsigned char submit[6];

void play(){

        int i;
        printf("Submit your 6 lotto bytes : ");
        fflush(stdout);

        int r;
        r = read(0, submit, 6);

        printf("Lotto Start!\n");
        //sleep(1);

        // generate lotto numbers
        int fd = open("/dev/urandom", O_RDONLY);
        if(fd==-1){
                printf("error. tell admin\n");
                exit(-1);
        }
        unsigned char lotto[6];
        if(read(fd, lotto, 6) != 6){
                printf("error2. tell admin\n");
                exit(-1);
        }
        for(i=0; i<6; i++){
                lotto[i] = (lotto[i] % 45) + 1;         // 1 ~ 45
        }
        close(fd);

        // calculate lotto score
        int match = 0, j = 0;
        for(i=0; i<6; i++){
                for(j=0; j<6; j++){
                        if(lotto[i] == submit[j]){
                                match++;
                        }
                }
        }

        // win!
        if(match == 6){
                setregid(getegid(), getegid());
                system("/bin/cat flag");
        }
        else{
                printf("bad luck...\n");
        }

}

void help(){
        printf("- nLotto Rule -\n");
        printf("nlotto is consisted with 6 random natural numbers less than 46\n");
        printf("your goal is to match lotto numbers as many as you can\n");
        printf("if you win lottery for *1st place*, you will get reward\n");
        printf("for more details, follow the link below\n");
        printf("http://www.nlotto.co.kr/counsel.do?method=playerGuide#buying_guide01\n\n");
        printf("mathematical chance to win this game is known to be 1/8145060.\n");
}

int main(int argc, char* argv[]){

        // menu
        unsigned int menu;

        while(1){

                printf("- Select Menu -\n");
                printf("1. Play Lotto\n");
                printf("2. Help\n");
                printf("3. Exit\n");

                scanf("%d", &menu);

                switch(menu){
                        case 1:
                                play();
                                break;
                        case 2:
                                help();
                                break;
                        case 3:
                                printf("bye\n");
                                return 0;
                        default:
                                printf("invalid menu\n");
                                break;
                }
        }
        return 0;
}
```
프로그램 자체는 정상 작동하는 로또 프로그램으로 보이나 같은 자리에 같은 글자가 있는것을 검사하는것이 아닌, 한 글자가 배열 전체에 있는지 검사합니다.

# 설계 및 시나리오
위와 같은 논리적 오류가 있으므로 정답 로또에 ascii값 45를 가지는 글자가 1개라도 있고 입력된 로또가 전부 ascii값 45를 가지도록 채워져있다면 6점을 손쉽게 얻을 수있습니다.

# 익스플로잇
랜덤 값중에 45가 있을때까지 계속 돌리면 플래그를 얻을 수있으므로 ------를 계속 넣으면됩니다. pwntool까지 사용하기는 귀찮기 때문에 6바이트만 읽는 것을 이용하여 로또 번호를 입력하는 동시에 메뉴 선택까지 하게 만들어 복사 붙여넣기만으로 작동하게 하였습니다.

# 결과 
```
...(생략)
Submit your 6 lotto bytes : ------1
Lotto Start!
bad luck...
- Select Menu -
1. Play Lotto
2. Help
3. Exit
Submit your 6 lotto bytes : ------1
Lotto Start!
Sorry_mom_1_Forgot_to_check_duplicates
- Select Menu -
1. Play Lotto
2. Help
3. Exit
Submit your 6 lotto bytes :
```