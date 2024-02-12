# void

POSIX Thread API 는 void * 를 사용하기 때문에, void * 부터 학습한 후 Thread 수업을 시작한다.

## void 포인터는 만능 포인터이다.

- 어떤 타입의 주소도 모두 다 저장할 수 있는 만능 포인터

```c
int x = 0;
char t = 'A';

int* p1 = &x;
char* p2 = &t;

```

위를 아래처럼 바꿀 수 있다.

```c
int x = 0;
char t = 'A';

void* p1 = &x;
void* p2 = &t;

```

만능 포인터의 기능 : 주소 저장만 가능
- 주소를 저장할 수 있다.
- 하지만 사용할 수 없다. 저장만 할 수 있다.

# Thread

운영체제는 여러 프로그램을 실행할 수 있지만, 펌웨어는 하나밖에 실행하지 못 한다.

스레드는 함수단위로 두 개 이상의 함수를 동시에 실행시키고 싶을 때 사용한다.

```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

void *abc() { // return 값이 void pointer
    while(1) {
        printf("ABC\n");
        sleep(1);
    }

    return 0;
}

void *bts() {
    while(1) {
        printf("BTS\n");
        sleep(1);
    }

    return 0;
}

int main()
{
    pthread_t t1, t2; // t1, t2가 쓰레드. 이 안에 abc랑 bts가 담긴다.

    pthread_create(&t1, NULL, abc, NULL); // params: 쓰레드의 주소, 옵션, 기준이 되는 함수(메인함수), 함수에 넣을 파라미터
    pthread_create(&t2, NULL, bts, NULL);

    pthread_join(t1, NULL); // abc가 끝나길 대기한다
    pthread_join(t2, NULL); // bts가 끝나길 대기한다
}

```

```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

void *DD1()
{
	while (1)
	{
		printf("[DD1] A\n");
		usleep(300 * 1000);
		printf("[DD1] B\n");
		usleep(300 * 1000);
		printf("[DD1] C\n");
		usleep(300 * 1000);
	}
}

void *DD2()
{
	while (1)
	{
		printf("[DD2] %d\n", 0);
		usleep(200 * 1000);
		printf("[DD2] %d\n", 1);
		usleep(200 * 1000);
		printf("[DD2] %d\n", 2);
		usleep(200 * 1000);
		printf("[DD2] %d\n", 3);
		usleep(200 * 1000);
		printf("[DD2] %d\n", 4);
		usleep(200 * 1000);
		printf("[DD2] %d\n", 5);
		usleep(200 * 1000);
	}
}

void *DD3()
{
	while (1)
	{
		for (char i = 'A'; i <= 'Z'; i++)
		{
			printf("[DD3] %c\n", i);
			usleep(300 * 1000);
		}
	}
}

int main()
{
	pthread_t t1, t2, t3;

	pthread_create(&t1, NULL, DD1, NULL);
	pthread_create(&t2, NULL, DD2, NULL);
	pthread_create(&t3, NULL, DD3, NULL);

	pthread_join(t1, NULL);
	pthread_join(t2, NULL);
	pthread_join(t3, NULL);
	return 0;
}

```

# Thread 인자값 넘기기

Thread에 인자 값을 넘길 수 있다.
- create 시 변수의 주소를 넘기는 방식 사용
- 구조체 변수를 사용하면 더 많은 값을 넘길 수 있다.

```c
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>

typedef struct _Node_
{
	int y;
	int x;
} Node;

void *abc(void *p)
{
	Node *val = (Node *)p; // 보이드 포인터로 받은걸 노드 포인터로 형변환해서
							// 노드 포인터 타입으로 val을 받는다.
	while(1) {
		printf("%d %d\n", val->x, val->y); // 포인터이기 때문에 '.' 이 아니라 '->' 로
		sleep(1);
	}
}

int main()
{
	pthread_t tid;
	Node val = {2, 4};

	pthread_create(&tid, NULL, abc, &val);
	pthread_join(tid, NULL);

	return 0;
}

```

각기 다른 스레드에서 증감하는 변수 참조하기
=> 스레드가 준비되는 속도가 느리기 때문에 비교적 늦은 시간에서의 변수 값을 참조하게 된다.

```c
#include <stdio.h>
#include <pthread.h>

pthread_t tid[4];

void *run(void *arg)
{
	int a = *(int *)arg;
	printf("%d", a);
}

int main()
{
	for (int i = 0; i < 4; i++)
	{
		pthread_create(&tid[i], NULL, run, &i);
		// thread가 준비되는 것에 비해 i가 증가하는 속도가 더 빠르기 때문에
		// 4개의 스레드의 파라미터로 4에 가까운 값이 비교적 많이 들어가게 된다
	}
	for (int i = 0; i < 4; i++)
		pthread_join(tid[i], NULL);

	return 0;
}

```

해결책 : 각기 다른 값을 참조하면 된다

```c
#include <stdio.h>
#include <pthread.h>

pthread_t tid[4];

void *run(void *arg)
{
	int a = *(int *)arg;
	printf("%d", a);
}

int main()
{
	int id[4]; // 각기 다른 값(숫자의 배열)을 만들어줘 각각 다른 값을 참조하면 된다.
	for (int i = 0; i < 4; i++)
	{
		id[i] = i;
		pthread_create(&tid[i], NULL, run, &id[i]);
		// thread가 준비되는 것에 비해 i가 증가하는 속도가 더 빠르기 때문에
		// 4개의 스레드의 파라미터로 4에 가까운 값이 비교적 많이 들어가게 된다
	}
	for (int i = 0; i < 4; i++)
	{
		pthread_join(tid[i], NULL);
	}
	return 0;
}

```

각각의 스레드 번호를 출력하고, 모든 스레드가 작동 완료 하면 "END"를 출력

```c
#include <stdio.h>
#include <pthread.h>

pthread_t tid[37];

void *run(void *arg)
{
	int a = *(int *)arg;
	printf("%d ", a);
}

int main()
{
	int id[37]; // 각기 다른 값(숫자의 배열)을 만들어줘 각각 다른 값을 참조하면 된다.
	for (int i = 0; i < 37; i++)
	{
		id[i] = i + 1;
		pthread_create(&tid[i], NULL, run, &id[i]);
		// thread가 준비되는 것에 비해 i가 증가하는 속도가 더 빠르기 때문에
		// 4개의 스레드의 파라미터로 4에 가까운 값이 비교적 많이 들어가게 된다
	}
	for (int i = 0; i < 37; i++)
	{
		pthread_join(tid[i], NULL);
	}

	printf("END\n");
	return 0;
}

```

[도전] 4개 스레드 만들기

```c
#include <stdio.h>
#include <pthread.h>

void *abc(void *p)
{
	int a = *(int *)p;
	if (a == 1)
	{
		printf("ABC\n");
	}
	else if (a == 2)
	{
		printf("MINMIN\n");
	}
	else if (a == 3)
	{
		printf("COCO\n");
	}
	else if (a == 4)
	{
		printf("KFCKFC\n");
	}
	else
	{
		printf("WRONG ID NUM\n");
	}
}

int main()
{
	pthread_t tid[4];

	int id[4] = {1, 2, 3, 4};

	for (int i = 0; i < 4; i++)
	{
		pthread_create(&tid[i], NULL, abc, &id[i]);
	}

	for (int i = 0; i < 4; i++)
	{
		pthread_join(tid[i], NULL);
	}
}

```

# Thread 메모리공유

요약: 스택은 공유 안된다
- .text, .bss, .data, .heap -> 공유 가능
- .stack -> 공유 안됨

# Mutex

## 용어 정리

### Race Condition

- situation (상황)
- Thread / Process 의 타이밍에 따라 결과 값이 달라질 수 있는 상태

### Critical Section (임계영역)

- spot (장소)
- HW 장치를 사용하는 곳 / Share Resource (전역 변수 등)

> 누군가 논문에 만들어놓은걸 프로그래밍적으로 구현시킨 것이다. 프로그래밍은 이런식으로 발전한다.

```c
#include <stdio.h>
#include <pthread.h>

pthread_t tid[4];
int cnt;

void *run()
{
	for (int i = 0; i < 10000; i++)
	{
		cnt++;
	}
}

int main()
{
	for (int i = 0; i < 4; i++)
	{
		pthread_create(&tid[i], NULL, run, &i);
	}
	for (int i = 0; i < 4; i++)
	{
		pthread_join(tid[i], NULL);
	}

	printf("%d\n", cnt);
}

```

```c
#include <stdio.h>
#include <pthread.h>

pthread_t tid[4];
int cnt;

void *run()
{
	for (int i = 0; i < 10000; i++)
	{
		cnt++;
	}
}

int main()
{
	for (int i = 0; i < 4; i++)
	{
		pthread_create(&tid[i], NULL, run, &i);
		pthread_join(tid[i], NULL); // 이러면 결국엔 스레드 하나가 돌아가는거나 같음

	}

	printf("%d\n", cnt);
}

```

뮤텍스 활용법

```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

pthread_mutex_t mlock;
pthread_t t[4];
int cnt;

void *run()
{
	pthread_mutex_lock(&mlock);
	for (int i = 0; i < 10000; i++)
	{
		cnt++;
	}
	pthread_mutex_unlock(&mlock);
}

int main()
{
	pthread_mutex_init(&mlock, NULL);
	for (int i = 0; i < 4; i++)
	{
		pthread_create(&t[i], NULL, run, NULL);
		usleep(100);
	}

	for (int i = 0; i < 4; i++)
	{
		pthread_join(t[i], NULL);
	}

	pthread_mutex_destroy(&mlock);
	printf("%d\n", cnt);

	return 0;
}

```