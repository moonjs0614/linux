# Low Level API

## System Call (Syscall)

리눅스 App이 리눅스 커널에게 어떤 부탁을 하기 위해 만들어진 API를 System Call 이라고 한다.

> `System Call` 을 줄여서 `Syscall` (`시스콜`) 이라고 한다.

## Low Level 파일처리 API

*`printf`, `scanf` 는 `System Call` 로 만들어졌을까?*

> - 그렇다.
>
> - `printf`
>   - 디스플레이 장치에 글자가 출력 되어야 한다.
>   - 커널에게 그래픽 카드를 통해 글자를 출력해 달라는 부탁을 `Syscall` 로 해야함.
>
> - `scanf`
>   - 키보드라는 입력장치가 동작을 읽어야한다.
>   - 이런 장치들은 커널이 관리하므로, 커널에게 부탁해야 한다.

*파일에 값을 읽고 쓰기 위한 `fopen` 은 `Syscall` 로 구현되어 있을까?*

>
> - `fopen()` 은 `Syscall` 을 포함한 Wrapper(감싼) 함수이다.
> - `High Level API` 라고 부른다.
>
> - `open()` 은 `Syscall` 중의 하나로, `Low Level API` 라고 부른다.
>
> `fopen()` 은 Syscall 중 하나인 `open()` 을 <U>사용</U>한다.

*`open()` 은 ...*

> - 커널은 모든 장치들을 관리한다.
> `open()` 은 `syscall` 로 커널에게 부탁한다.
> - ex. Disk 저장장치에 저장된 값을 읽어주세요.

# 임베디드 개발자에게 파일처리 syscall의 의미

## 리눅스에서 파일의 의미

리눅스는 모든 장치들을 파일로 관리한다.

- 마우스 장치 파일
- 키보드 장치 파일
- 메모리 장치 파일

> 만약 리눅스에서 마우스 라는 장치가 갖고 있는 클릭 여부의 값을 알고싶다면
> => 리눅스 `cat` 명령어로 마우스 장치 파일을 읽어보면 된다.
>
> 만약 마우스 라는 장치의 클릭 여부를 출력해주는 App 을 만드려면
> => `open()`, `read()` 등 syscall 을 사용하여 장치파일을 읽어서 장치가 갖고 있는 정보를 알 수 있다.
>   - 장치파일은 `fopen()` 을 쓰면 안되고, `Low Level API` 를 써야만 한다.

차후 커널 프로그래밍 에서 "리눅스 디바이스 드라이버" 라는 플랫폼을 써서 장치를 제어한다.
이 때도 파일처리 syscall 을 사용한다.

# `open`, `read`, `write`, `close`

## [실습] 파일 오픈하기

## `vi` 에서 `man` 확인

## File Descriptor

## `open()` system call arguments

```c
int open(path, flag, mode);
```

### argument1 : path

읽을 파일의 경로

### argument2 : flag

### argument3 : mode

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main()
{
    int fd = open("./test.txt", O_WRONLY | O_TRUNC, 0664 );

    char buf[256] = "HI SYSTEM CALL";
    // char* buf = "HI SYSTEM CALL"; // 이렇게 작성하면 문자열의 길이만큼만 메모리를 차지한다.

    write(fd, buf, strlen(buf));

    if (fd<0) {
        printf("[%s :: %d] FILE OPEN ERROR\n", __FILE__, __LINE__);
        exit(1);
    }

    close(fd);

    return 0;
 }
```

```c
int main()
{
    int fd = open("./cal.txt", O_RDONLY, 0); // 열고
    char buf[1000] = {0};
    read(fd, buf, 1000);
    close(fd); // 닫고

    int num = atoi(buf);
    num *= 2;
    printf("\n==={%d}===\n", num);
    sprintf(buf, "%d", num);

    fd = open("./cal.txt", O_WRONLY, 0); // 열고
    write(fd, buf, strlen(buf));
    close(fd); // 닫고
}

```

# read와 write 응용 - 구조체변수 저장, 읽기

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main()
{
    int fd = open("./text.txt", O_RDONLY, 0);
    if (fd < 0) {
        printf("FILE OPEN ERROR\n");
        exit(1);
    }

    char buf[10] = {0};
    read(fd, buf, 10);

    printf("====READ====\n");
    printf("%s\n", buf);
    printf("====END====\n");

    close(fd);
    return 0;
}

```

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main()
{
    int fd = open("./test.txt", O_RDONLY, 0);
    if ( fd < 0 ) {
        printf("NOT READ\n");
    }

    char buf[10] = { 0 };
    ssize_t i = read(fd, buf, 10);

    printf("============\n");
    printf("%s\n", buf);
    printf("============\n");
    printf("%lu\n", i);

    close(fd);
    return 0;
}
```

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main()
{
    int fd = open("./test.txt", O_RDONLY, 0);
    if (fd < 0) {
        printf("FILE OPEN ERROR\n");
        exit(1);
    }

    char buf[11] = {0};
    read(fd, buf, 10);
    close(fd);

    int n;

    n = lseek(fd, 0, SEEK_CUR);
    printf("%d\n", n);
    read(fd, buf, 10);
    printf("%s\n", buf);

    n = lseek(fd, 0, SEEK_CUR);
    printf("%d\n", n);
    memset(buf, 0, 10);
    read(fd, buf, 5);
    printf("%s\n", buf);

    n = lseek(fd, 0, SEEK_CUR);
    printf("%d\n", n);

    return 0;
}


```

# Write 연습하기

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>
#include <unistd.h>

int main()
{
    int fd = open("test.txt", O_WRONLY | O_TRUNC, 0666);

    char buf[10] = "[NEW]";
    int n = strlen(buf);

    write(fd, buf, n);

    close(fd);
    return 0;
}

```