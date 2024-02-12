# 2장. Device Driver 기초

## 커널 빌드

### 커널 빌드하기

동일한 환경에서 Device Driver를 개발파기 위한 준비

1. 라즈베리파이 준비

   - 라즈베리파이 환경 준비하기
2. Linux Kernel 을 Download / Build

   - 모두 같은 버전의 Linux 에서 실습을 하자

라즈베리파이 OS 커널 빌드 (공식 홈페이지)

- https://www.raspberrypi.com/documentation/computers/linux_kernel.html

### 빌드 순서

1. 종속성 도구 설치

`$ sudo apt install git bc bison flex libssl-dev make`

2. 커널 소스코드 다운로드

   - `$ cd /usr/src`
   - `$ sudo git clone --depth=1 https://github.com/raspberrypi/linux`
3. config 파일 생성

   - `make` 시 `sudo` 붙여주자

```bash
$ cd linux
$ KERNEL=kernel7l
$ sudo make bcm2711_defconfig
```

4. Build 하기
   - zImage : Linux 커널
   - modules : 커널에서 사용하는 Device Driver
   - dtbs : 디바이스 드라이버 세부 정보

```bash
$ make -j4 zImage modules dtbs
```

> 약 60분 이후 빌드가 완료된다. 커널 빌드 성공 메세지는 따로 없고 에러 메세지가 없이 끝난다면 성공한 것

5. Build 된 결과물 Copy
   1. modules_install : 커널이 사용하는 디바이스 드라이버들이 적절한 곳에 설치
   2. dtb 들을 SD 카드(/boot)에 복사
   3. Build된 Kernel을 SD카드 (/boot)에 복사

```bash
$ sudo make modules_install
$ sudo cp arch/arm/boot/dts/*.dtb /boot/
$ sudo cp arch/arm/boot/dts/overlays/*.dtb* /boot/overlays/
$ sudo cp arch/arm/boot/dts/overlays/README /boot/overlays/
$ sudo cp arch/arm/boot/zImage /boot/$KERNEL.img
```

### 커널 버전 확인

리셋 전 `uname -r`

- `5.15.84-v71+` (커널 버전은 이것과 다를 수 있다.)

리셋 후

- `6.1.21-v71+` (커널 버전은 이것과 다를 수 있다.)

리셋 전 후로 버전이 달라져야 한다.

### Trouble Shooting

만약 커널 빋르 성공했지만, 리셋 후에도 똑같은 버전인 경우

- 마지막 복사 단계가 제대로 안 된 것이므로 아래와 같이 입력하자.

```bash
$ KERNEL=kernel7l
$ sudo cp arch/arm/boot/zImage /boot/$KERNEL.img
```

- `reboot` 후 `uname -r` 로 결과 학인하기

### Device Driver 개발 시 필요한 것 : Kernel Header

커널 빌드를 하는 이유는 실습 환경을 위해 모든 사람의 버전을 일치시켜주는 것도 있찌만, 커널 빌드를 하면 커널 헤더라는 부산물이 생기기 때문이기도 하다.
단, 빌드 과정이 워낙 오래 걸려 커널 헤더만 따로 다운로드 받을 수 도 있다.

### 실습 환경

OS : 32bit

- `$ getconf LONG_BIT`

커널버전 : `*-v7l+`

- `uname -r`

아키텍처 : armv7l

- `uname -m`

## volatile

### volatile

- 최적화를 방해하는 범인은 `컴파일러`

### 컴파일러가 최적화 하는 과정

- 어떻게 동작하는 코드일까?

```c
#include <stdio.h>

int main()
{
    int a = 10;
    a = 20;
    a = 30;
    a = 40;
    printf("%d", a);

    return 0;
}
```

### 컴파일러의 역할

사람이 짠 `.c` 코드를 보고 기계어로 변환한다

컴파일러는 생각보다 똑똑하다.

> 코드는 생각보다 길고, 컴파일러는 조금만 일하고 싶어한다.

컴파일러는 `release` 할 때, 불필요한 절차를 다 생략하고 최적화 해서 컴파일한다.

### volatile 추가

- 변수 타입 앞에 `volatile` 을 붙이고 다시 디버깅하자

```c
#include <stdio.h>

int main()
{
    volatile int a = 10;
    a = 20;
    a = 30;
    a = 40;
    print("%d", a);

    return 0;
}
```

**volatile 은 컴파일러가 최적화하는 걸 방해한다.**

`volatile` 을 안 썼을 경우 디셈블리어

```
--- c:\users\username\source\repos\project1\project1\main.cpp ---------------------
	int a = 10;
	a = 20;
	a = 30;
	a = 40;
	printf("%d", a);
00271040  push        28h  
00271042  push        offset string "%d" (02720F8h)  
00271047  call        printf (0271010h)  
0027104C  add         esp,8  

	return 0;
0027104F  xor         eax,eax  
}
00271051  ret  
```

`volatile` 을 썼을 경우 디셈블리어

```
--- c:\users\ssafy\source\repos\project1\project1\main.cpp ---------------------
#include <stdio.h>

int main()
{
004D1040  push        ebp  
004D1041  mov         ebp,esp  
004D1043  push        ecx  
	volatile int a = 10;
004D1044  mov         dword ptr [ebp-4],0Ah  
	a = 20;
004D104B  mov         dword ptr [a],14h  
	a = 30;
004D1052  mov         dword ptr [a],1Eh  
	a = 40;
004D1059  mov         dword ptr [a],28h  
	printf("%d", a);
004D1060  mov         eax,dword ptr [a]  
004D1063  push        eax  
004D1064  push        offset string "%d" (04D20F8h)  
004D1069  call        printf (04D1010h)  
004D106E  add         esp,8  

	return 0;
004D1071  xor         eax,eax  
}
004D1073  mov         esp,ebp  
004D1075  pop         ebp  
004D1076  ret  
```

### volatile 용도

임베디드에서 H/W 의 특정 주소 값을 이용해 제어하는 경우

```c
device = 0xAABB;
while (device != 0xAABB){

}
```

인터럽트 핸들러를 사용할 때

```c
uint8_t input_value[2] = {0x0, 0x0};

void interrupt port_read(void) {
    input_value[0] = PORTA;
    input_value[1] = PORTB;
}

int main() {
    if (input_value[0] != input_value[1]) {
        // 알람을 울린다.
    }

    return 0;
}
```

그 이외에도 volatile이 필요한 경우는

- 메모리 주소를 가진 I/O 레지스터
- 인터럽트 핸들러가 값을 변경하는 전역 변수
- 등등 최적화에 의해 오류가 발생할 가능성이 있는 변수에 쓰인다.

> 너무 남발하면 오히려 코드 최적화를 할 수 없으므로 적재적소에 volatile을 붙인다.

## app과 driver 개발

### 수업 준비하기

이하 insmod, rmmod 과정 생략

### 편리한 개발을 위한 세팅

shell script를 작성하자 (alias.sh)

- 쉬뱅은 필요 없다
- 디바이스 드라이버 커널 적재와 제거 (insmod / rmmod)
- alias : 별명

`alias.sh`

```sh
alias sc='sudo insmod nobrand.ko'
alias sd='sudo rmmod nobrand'
```

```bash
user@HOSTNAME:~/work$ ls
Makefile alias.sh app.c nobrand.c
```

## read / write

### 지금까지 배운 내용

H/W 없이, app -> Device Driver 로 open / close system call 을 보내 Device File 을 열고 닫기만 하였다.

이제 우리는 데이터를 보내고 받는 내용에 대해 학습할 것이다.
=> `system call` 을 이용해 `Device File`에 데이터를 읽고 쓰는 것을 할 것이다.

- `read` / `write` / `ioctl`

### `chrdev` 는 문자 기반의 디바이스 파일

`data` 를 주고 받을 때 사용하는 `syscall`

- `read` / `write` : byte 단위 값 전달 받기
- `ioctl` : 제어, 설정 용도

### `app.c` 코드에 read syscall 추가

Byte 단위로 값을 보내고 받을 때 `write` / `read` syscall 사용

- `write(fd, 메세지, 사이즈)`
- `read(fc, buf, buf사이즈)`

먼저 read를 구현하자

```c
int main()
{
    int fd = open("/dev/nobrand", O_RDWR);
    if (fd < 0)
    {
        printf("ERROR\n");
        close(fd);
        exit(1);
    }

    // read char
    char buf[100];
    read(fd, buf, 100);

    printf("%s\n", buf);

    close(fd);
    return 0;
}
```

### Driver 코드에 read/write 추가

`nobrand_read(파일포인터, buf, 길이, 오프셋);`

- return count?

```c
static ssize_t nobrand_read(struct file *flip, char *buf, size_t count, loff_t *ppos)
{
    buf[0] = 'H';
    buf[1] = 'I';
    buf[2] = '\0';
    return count;
}

static struct file_operations fops =
{
    .open = nobrand_open,
    .release = nobrand_release,
    .read = nobrand_read,
};

```

## ioctl

### ioctl 이란?

- ioctl (intput output control)

  - `<sys/ioctl.h>` 필요
  - 하드웨어를 제어하기 위한 함수
  - 예약된 동작 번호 지정
- `ioctl (파일 디스크립터, cmd, arg)`

  - ex. `ioctl(fd, _IO(0,3), 0);`
  - `_IO(type, num)` : 명령 코드 중 하나
    - `type` : 매직 번호, 특별한 의미가 있는 것은 아님. 가급적 다른 디바이스 드라이버와 구별
    - 3번 명령을 하자

### app에 코드를 추가하자

- 헤더 추가

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/ioctl.h>

int main() {
    int fd = open("/dev/nobrand", O_RDWR);
    if ( fd<0 ) {
        printf("ERROR\n");
        close(fd);
        exit(1);
    }

    ioctl(fd, _IO(0, 3), 0);
    ioctl(fd, _IO(0, 4), 0);
    ioctl(fd, _IO(0, 5), 0);

    printf("Is it now?\n");

    close(fd);

    return 0;
}
```

### driver에 코드 추가

- nobrand_ioctl 추가 / fops 에 연결

```c
static long nobrand_ioctl(struct file *flip, unsigned int cmd, unsigned long arg) {
    printk(KERN_ALERT "command number : %d\n", cmd);
    switch(cmd) {
        case _IO(0, 3):
            printk(KERN_INFO "HI - ioctl 3\n");
            break;
        case _IO(0, 4):
            printk(KERN_INFO "HI - ioctl 4\n");
            break;
        case _IO(0, 5):
            printk(KERN_INFO "HI - ioctl 5\n");
            break;
    }

    return 0;
}

static struct file_operations fops = {
	.open = nobrand_open,
	.release = nobrand_release,
    .read = nobrand_read,
    .write = nobrand_write,
    .unlocked_ioctl = nobrand_ioctl,
};
```

### ioctl 테스트

```bash
user@hostname~/work$ sudo insmod nobrand.ko
user@hostname~/work$ ./app
Is it now?
user@hostname~/work$ sudo rmmod nobrand
```

커널 로그

```bash
[ 7950.925672] hi
[ 7955.234559] welcome
[ 7955.234562] command number : 3
[ 7955.234565] HI - ioctl 3
[ 7955.234565] command number : 4
[ 7955.234572] HI - ioctl 4
[ 7955.234572] command number : 5
[ 7955.234573] HI - ioctl 5
[ 7955.234653] release
[ 7964.781816] bye
```

## cmd parameter

### ioctl 의 쉬운 사용

- ioctl()의 _IO 매크로 없이 그냥 사용할 수 도 있다.

`app.c`

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/ioctl.h>

int main(){
	int fd = open("/dev/nobrand", O_RDWR); // app 은 디바이스 파일과 연결한다. nobrand.ko 와는 아무 상관 없다. cf. mknod 로 nobrand 디바이스 파일이 생성됨
	if ( fd<0 ){
		printf("ERROR\n");
		exit(1);
	}

    ioctl(fd, _IO(0, 3), 1234);
    ioctl(fd, _IO(0, 4), 9012);
    ioctl(fd, _IO(0, 5), 5678);

    printf("Is it now?\n");

	close(fd);
	return 0;
}


```

### driver 에 코드 추가

- case 도 3, 4, 5 로 구분

```c
static long nobrand_ioctl(struct file *flip, unsigned int cmd, unsigned long arg) {
    printk(KERN_ALERT "command number : %d\n", cmd);
    switch(cmd) {
        case _IO(0, 3):
            printk(KERN_INFO "HI - ioctl 3\n");
            break;
        case _IO(0, 4):
            printk(KERN_INFO "HI - ioctl 4\n");
            break;
        case _IO(0, 5):
            printk(KERN_INFO "HI - ioctl 5\n");
            break;
    }

    return 0;
}

```

### 동작 테스트

- 결과가 똑같다!
  - 그럼 왜 귀찮게 `_IO` 를 사용할까?

### cmd 규칙에 어긋난 코딩 방식

- cmd 규칙이 있다. -> 권장사항
  - cmd format 에 맞춰서 사용해야 한다.

### ioctl 동작의 네 가지 분류

- simple ioctl

  - 구분 번호만 주어지는 경우
- read ioctl

  - 데이터를 읽는 경우
- write ioctl

  - 데이터를 쓰는 경우
- r/w ioctl

  - 읽고 쓰는 ioctl

### cmd parameter

cmd 구성

- 약속된 cmd 변수의 비트 단위 Format (32bit)

  - Direction : R/W
  - Size : 데이터 크기
  - Type : 매직넘버
  - Number : 구분 번호
- 이걸 일일이 지켜가며 코드를 작성하는 것은 매우 어렵다.

Direction (2 bit) | Size (14 bit) | Type (8 bit) | Number (8 bit)

### 그래서 Macro 가 제공된다.

쉽게 Format을 사용하기 위한 Kernel 이 제공하는 Macro 함수

- `_IO(type, number)`
- `_IOR(type, number, 전송 받을 데이터 타입)`
- `_IOW(type, number, 전송 보낼 데이터 타입)`
- `_IOWR(type, number, 전송 주고 받을 데이터 타입)`

## API return

### ioctl 의 역할

- ioctl 을 이용해 H/W 에 여러 옵션을 줘서 명령을 할 수 있다.
  - app 에서 cmd 에 정수 값을 보냄
  - driver 에서 cmd 값에 따라 select 문으로 분기를 태움
  - cmd 값에 따른 코드 추가 가능
  - args 에 포인터를 전달하여 app의 다량의 데이터를 driver로 전달 가능

위의 과정 예시

```c
ret = ioctl(fd, CMD, &args);

int device_ioctl(struct inode *inode, struct file *filep, unsigned int cmd, unsigned long arg)
{
    return ret;
}

```

### return 값이란

하드웨어 개발응ㄹ 하면 코드 한 줄 한 줄 api를 쓸 때마다 에러의 위험이 있다.
하드웨어는 보통 메모리를 직접 건드릴 가능성이 매우 높기 때문에 에러의 위험성이 상당히 크게 다가온다.
따라서 개발자는 에러가 날 수 있는 모든 곳에 에러 검출을 위한 코드를 작성한다.

### return 값 활용 방법

app.c 에서 코드를 작성하자

- 사용자로부터 cmd 값을 입력받는다.
- cmd 값을 3 ~ 5 만 입력 받아야 하고
- 그 이외의 값을 입력하면 에러를 검출한다

`app.c`

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/ioctl.h>

int main(){
	int fd = open("/dev/nobrand", O_RDWR); // app 은 디바이스 파일과 연결한다. nobrand.ko 와는 아무 상관 없다. cf. mknod 로 nobrand 디바이스 파일이 생성됨
	if ( fd<0 ){
		printf("ERROR\n");
		exit(1);
	}

    int n;
    while(1) {
        scanf("%d", &n);
        int ret = ioctl(fd, _IO(0, n), 0);
        if (ret < 0) {
            printf("command invalid!\n");
            break;
        }
    }

	close(fd);
	return 0;
}
```

드라이버.c 파일의 switch 문에 default를 추가

- `-EINVAL` : Invalid argument

```c
static long nobrand_ioctl(struct file *flip, unsigned int cmd, unsigned long arg) {
    printk(KERN_ALERT "command number : %d\n", cmd);
    switch(cmd) {
        case _IO(0, 3):
            printk(KERN_INFO "HI - ioctl 3\n");
            break;
        case _IO(0, 4):
            printk(KERN_INFO "HI - ioctl 4\n");
            break;
        case _IO(0, 5):
            printk(KERN_INFO "HI - ioctl 5\n");
            break;
        default:
            return -EINVAL;
    }

    return 0;
}
```

### make 후 테스트

app 을 실행해 범위를 벗어난 값을 입력하니, 커널에도 로그가 남 지 않고, app에서 확인할 수 있다.

bash 명령어

```bash
username@HOSTNAME:~/workspace$ sudo insmod nobrand.ko
username@HOSTNAME:~/workspace$ ./app
3
4
5
6
command invalid!
username@HOSTNAME:~/workspace$ sudo rmmod nobrand
```

커널 로그

```bash
[ 9333.532193] hi
[ 9336.057645] welcome
[ 9340.110907] command number : 3
[ 9340.110911] HI - ioctl 3
[ 9340.887042] command number : 4
[ 9340.887047] HI - ioctl 4
[ 9341.390967] command number : 5
[ 9341.390972] HI - ioctl 5
[ 9341.780533] command number : 6
[ 9341.780550] release
[ 9347.980564] bye
```

### ioctl argument

수업 skip

## kernel log

### [참고] 커널 로그 레벨

커널이 발생시키는 메시지를 중요도에 따라 분류하여 출력하는 방법

- `printk()` 에 메시지의 중요도를 나타낼 수 있다.
- 커널에 출력할 로그 레벨을 설정해, 설정된 로그 레벨 이상의 메시지만 출력할 수 있다.
- 디버깅에 도움이 된다.

| 로그 레벨 | DEFINE | 의미 |
| --------- | ------ | ---- |
|           |        |      |
|           |        |      |
|           |        |      |
|           |        |      |
|           |        |      |
|           |        |      |
|           |        |      |

### kernel log 확인

- 로그 레벨에 따라 kernel 에 뜨는 메시지를 확인하자.

## kernel 디버깅

수업 skip

### copy_from/to_user

여러 개의 메시지를 보내려면?

- 문자열 / 구조체를 전달하면 된다.

단, 그냥 전송하면 안된다.

- user space 와 kernel space 는 메모리가 구분되어 있다.

### Linux Layer

리눅스 layer 는 여러 개

- user space
- kernel space
- hardware

Subsystems

- 커널의 모듈들
  - 메모리 관리
  - 프로세스 관리
  - 파일 시스템
  - 등등

### [참고] OS architecture (면접용 장표)

- Monolithic 모놀리식 : 커널의 영향이 큼
- Microkernel 마이크로커널 : RTOS 처럼 커널의 영향 최소

https://yozm.wishket.com/magazine/detail/1813/

### 접근할 수 없는 메모리

- User space 와 Kernel space 는 메모리 접근이 안된다!
  - 위험 / 금지! -> user space 메모리 주소와 kernel space 의 메모리 주소가 다름
  - 그럼 어떻게 할까?

### copy from/to user 가 해결해준다

- `<linux/uaccess.h>` 헤더 필요
  - user space -> kernel space 로 메시지 전송 : copy_from_user
  - kernel space -> user space 으로 메시지 전송 : copy_to_user

> [user space]  --`copy_from_user()`--> [kernel space]
> [user space] <--`copy_to_user()`--    [kernel space]

### copy_from_user

- buf 에 있는 값을 ioctl 로 전송

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/ioctl.h>

char buf[30] = "THIS IS APP MEMORY";
int main()
{
    int fd = open("/dev/nobrand", O_RDWR);
    if (fd < 0)
    {
        printf("ERROR\n");
        exit(1);
    }

    ioctl(fd, _IO(0, 3), buf);
    close(fd);
    
    return 0;
}

```


> Native Compile 과 Cross compile의 차이점 시험에 나옴 (109페이지)
> Cross Compile은 한마디로 가상 환경의 CPU에서 컴파일 한다고 생각하면 된다. (CPU 회사에서 가상 CPU 환경(IDE)을 제공해준다.)

