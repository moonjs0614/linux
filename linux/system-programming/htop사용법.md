프로세스, 프로세서, 프로그램 구분하기 (프로세서가 뭐더라)

GUI냐 CLI냐에 따라 root processor 가 다르다

`kill` 은 시험에 나온다. (죽이는게 아니라 죽이는 신호만 보낸다. 보낸다고 죽는것도 아니다. 다른 프로세스에 signal을 보낸다.)

`ctrl + c` => SIGINT => 종료
`ctrl + z` => SIGSTOP => 중지

```bash
gihong@gihong-ssafy:~$ python3
Python 3.8.10 (default, Mar 13 2023, 10:26:41)
[GCC 9.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> # $ctrl + c
KeyboardInterrupt
>>> # $ctrl + z
[1]+  Stopped                 python3
```


프로세스에 signal 보내기

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

void gogo(int num)
{
    printf("GOGO\n");
}

int main()
{
    signal(SIGUSR1, gogo);
    while(1) sleep(1);

    return 0;
}
```


```bash
gihong@gihong-ssafy:~/workspace/signalAPI$ kill -10 9272
gihong@gihong-ssafy:~/workspace/signalAPI$ killall -SIGUSR1 ./result
```
