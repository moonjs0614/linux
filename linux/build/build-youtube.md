# 빌드란?

- 소스코드에서 실행 가능한 software로 변환하는 과정 (Process) 또는 결과물

# C언어 빌드 과정

1. Compile & Assemble
    - 하나의 소스코드 파일이 0과 1로 구성된 Object 파일이 만들어진다.

2. Linking
   - 만들어진 Object 파일들 + Library 들을 모아 하나로 합침

> 이 두가지 과정은 `빌드의 대표적인 역할`이다. 더 세부적인 과정은 임베디드 C언어 시간에 다룸.

 ## 1. 각각의 파일을 Compile & Assemble
  
- 각각의 C언어 파일을 컴파일(+Assemble) 한다.
- 명령어 수행 (-c 옵션 : Compile and Assemble)
  
`$gcc -c ./green.c` , `$gcc -c ./yellow.c`

## 2. Linking

- 만들어진 Object 파일들과 라이브러리 함수들을 하나로 합친다.
  - `$gcc ./green.o ./yellow.o -o ./go`
  - -o 옵션 : output 파일 지정 (안해주면 default 파일명으로 a.out)

cf. gcc가 똑똑해서 아래와 같이 해도 되지만, 학습을 위해 단계를 나누어서 수행한다.
> - green.c / yellow.c 를 삭제 후 해야한다.

```bash
$ gcc ./*.c
$ ./a.out
```

# 빌드 자동화 스크립트

## build script 제작하기