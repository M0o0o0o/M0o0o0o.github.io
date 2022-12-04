---
layout: post
title:  "쉘 스크립트 1"
date:   2022-11-30 12:27:00 +0900
categories: Shell-Script
---

** 인용부호 내의 공백문자는 일반문자로 사용된다.


## echo
- **'-e'** 옵션은 특수문자를 사용할 때 사용한다. 
  - echo -e "hello world \n\n\n"
  
## 명령의 구조 
- 쉘에서는 기본적으로 명령어 뒤에는 옵션을 넣을 수 있다.
  - ex: <span style="color:red;">**ls -al**</span>
  
## glob
- '*'를 glob 문자라 한다.
- 파일을 조회할 때 '*'를 자주 사용하는데, glob 문자에 의해 매칭된 파일로 치환되는 것을 globbing이라고 합니다.

## 인용문
- ex: rm 'lee moo yeol.txt'
- 공백문자를 포함하는 경우에는 인용문(single quote, double quote)를 사용해야 한다.

## [..]와 테스트
- 쉘에는 데이터 타입이 없기 때문에 명령문의 모든 문자는 String이며, 산술연산과 같은 작업은 별도의 확장이나 명령을 통해 제공된다.
- '[]'는 테스트 명령어로 리눅스 쉘에서는 '[]'는 안에 있는 명령어를 테스트 한다는 뜻으로 사용된다.
- '[]'안에 있는 명령어의 참, 거짓
- if-else문에서 많이 사용된다.

## read
> read 명령어는 사용자로부터 입력을 받는다.
- read -p "what is your adress : " v
  - 입력한 값이 변수명 v에 들어간다.(변수 이름은 자유롭게 지정이 가능)
- read -n -1 -p "Are you over 26?" v
  - 한 글자만 입력 가능하다.
- read -s -n 1 -p "Are you over 26?" v
  - '-s' 옵션은 사용자의 입력을 터미널 상에 나타나지 않게 한다. (silent)
- read -s -n -1 -t 3 "Are you over 26?" v
  - 3초 동안 입력이 없으면 입력을 기다리지 않고 종료된다.

## 실행파일을 사용하는 방법(4가지)
1. 현재 경로에서 실행하기
2. 실행 파일의 경로를 환경 변수에 추가하기
3. 실행 파일의 전체 경로를 표현하기
4. 기존 실행 파일들이 있는 경로에 복사하여 사용하기 

---

## 쉘 스크립트란?
- 쉘은 명령 인터프리터로 사용자가 OS에 대화식으로 명령을 내리거나, 명령을 일괄적으로 실행할 수 있는 기능을 제공하는 응용 프로그램이다.

## 쉘 스크립트의 실행방법
1. chmod +x script.sh
2. bash script.sh
3. source script.sh
4. ./script.sh


## 쉘 변수
- 쉘 변수를 선언할 때는 대입연산자(=) 양쪽에 **공백 문자가 있으면 안된다.**
```shell
color=white
echo $color
```
