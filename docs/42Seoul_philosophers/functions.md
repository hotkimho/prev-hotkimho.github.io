---
layout: default
title: Functions
parent: Philosophers

---

# __1. gettimeofday__
## 1. 헤더파일
```c
#include <sys/time.h>
```

## 2. 함수 원형
```c
int gettimeofday(struct timeval *tv, struct timezone *tz);
```

## 3. 반환 값
성공 시 `0`
실패 시 `-1`

## 4. 함수 설명
`gettimeofday` 함수는 시간을 측정할 때 사용하는 함수입니다 2개의 매개변수를 받으며 각각 `timeval`, `timezone` 구조체의 주소를 받습니다.
`timezon` 구조체는 현재 사용되지 않고 있으며 이 필드가 사용된 경우 버그로 판단되어 무시된다고 합니다. 그래서 호출할 때 `NULL`값을 넘겨줍니다.
`timeval` 구조체는 다음과 같이 정의되어 있습니다.
 ```c
struct timeval
{
    long tv_sec;  // 초
    long tv_usec; // 마이크로초
}
 ```
각각 `sec`, `usec` 변수에는 `1970-01-01 00:00:00 +0000 (UTC)` 이후 현재까지 경과된 초와 마이크로초를 저장합니다.

## 5. 예시 코드
```c
#include <stdio.h>
#include <sys/time.h>
#include <stdlib.h>

int main()
{
	struct timeval startTime, endTime;
	double time;

	gettimeofday(&startTime, NULL);
	_sleep(2700);
	gettimeofday(&endTime, NULL);
	time = endTime.tv_sec + endTime.tv_usec / 1000000.0 - startTime.tv_sec - startTime.tv_usec / 1000000.0;
	printf("%.2f\n", time);
}
```
![1.return value](/42Seoul_philosophers/images/gettimeofday_1.png)

`timeval` 구조체 `startTime`, `endTime`를 만들고 시작할 때, 끝날 때 각각 함수를 호출 했습니다.
`startTime` 시작할 때 시간값을 가지고 있고, `endTime`에는 `sleep(2700)` 시간 이후의 시간값을 가지고 있습니다.
`endTime`에서 `startTime`을 빼면 두 함수 사이의 걸린 시간을 알 수 있습니다. 위 결과에선 `2.7`의 값을 가지고 있으며 약간의 오차가 있었습니다.

# __2. pthread_create__
## 1. 헤더파일
```c
#include <pthread.h>
```

## 2. 함수 원형
```c
int pthread_create(pthread_t *restrict thread, const pthread_attr_t *restrict attr, void *(*start_routine)(void *), void *restrict arg);
```

## 3. 반환 값
성공 시 `0` 반환
실패 시 `error number` 반환

## 4. 함수 설명
현재 `프로세스`에서 `스레드`를 생성하고 `루틴(함수)`을 실행시킵니다. 4개의 매개변수를 가지고 있습니다
1. 쓰레드가 생성되었을 때, 생성된 쓰레드의 식별자(id)입니다
2. 쓰레드와 관련된 특성을 지정하는데 사용합니다. `NULL` 값이 들어오면 기본값으로 쓰레드가 생성됩니다
3. 쓰레드를 실행할 함수입니다.
4. 3번의 실행할 함수에 인자값입니다.

`fork`처럼 쓰레드도 쓰레드를 생성한 `부모` 쓰레드가 종료되면 자식 쓰레드 또한 종료됩니다. 그외에 아래와 같은 종료 조건이 있습니다
1. pthread_exit(3) 함수를 호출하여 종료합니다.
2. 세 번째 매개변수인 routine 함수를 리턴합니다. 이 종료는 pthread_exit(3)와 동일합니다.
3. pthread_cancel(3) 함수에 의한 취소

## 5. 예시 코드
```c
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>

void *routine(void *param)
{
	int *arr = (int *)param;
	int i = 0;

	for (int i = 0; i < 5; i++)
	{
		printf("%d\n", arr[i]);
		sleep(1);
	}
}

int main()
{
	pthread_t thread_id;
	int state;

	int main_Arr[5] = { 1, 2, 3 ,4 ,5};
	int thread_Arr[5] = { 6, 7, 8 , 9, 10};
	pthread_create(&thread_id, NULL, routine, (void *)thread_Arr);
	routine((void *) main_Arr);
	pthread_join(thread_id, (void **)&state);
}
```
![return value2](/42Seoul_philosophers/images/create_1.png)

코드를 보면 생성한 자식 쓰레드에 `thread_Arr` 배열을 넘겨줘서 출력을 실행하고, 부모 쓰레드에서 `main_Arr` 배열을 출력합니다. 한번 출력 시 1초의 텀을 줘서 위 사진처럼 순차적으로 출력됩니다.


# __3. pthread_join__
## 1. 헤더파일
```c
#include <pthread.h>
```

## 2. 함수 원형
```c
int pthread_join(pthread_t thread, void **retval);
```

## 3. 반환 값
성공 시 `0` 반환
실패 시 `error number` 반환

## 4. 함수 설명
매개변수는 thread 식별자(id), 해당 스레드가 종료될 때 리턴값 없으면 NULL을 넣으면 된다.
`wait`함수처럼 해당 스레드가 종료할 때까지 기다리며, 두 번째 인자로 리턴값을 받아온다.

## 5. 예시 코드
```c
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>

void *routine(void *param)
{
	int *arr = (int *)param;
	int i = 0;

	for (int i = 0; i < 5; i++)
	{
		printf("%d\n", arr[i]);
		_sleep(300);
	}
	return (void *)arr[0];
}

int main()
{
	pthread_t thread_id;
	int state;

	int main_Arr[5] = { 1, 2, 3 ,4 ,5};
	int thread_Arr[5] = { 6, 7, 8 , 9, 10};
	pthread_create(&thread_id, NULL, routine, (void *)thread_Arr);
	routine((void *) main_Arr);
	pthread_join(thread_id, (void **)&state);
	printf("return value : %d\n", state);
}
```
![return value2](/42Seoul_philosophers/images/join_1.png)

생성된 스레드는 `thread_Arr`의 첫번 째 요소 값을 반환합니다. 반환된 값은 `state`값에 저장됩니다.



# __4. pthread_detach__
## 1. 헤더파일
```c
#include <pthread.h>
```

## 2. 함수 원형
```c
int pthread_detach(pthread_t thread);
```

## 3. 반환 값
성공 시 `0` 반환
실패 시 `error number` 반환

## 4. 함수 설명
매개변수로 입력된 `thread 식별자`를 메인 쓰레드에서 분리시키고 쓰레드의 모든 자원을 free 한다.
`pthread_join` 함수는 쓰레드가 끝나길 기다리고 자원을 반납한다.
이 두 가지의 경우가 아니면 쓰레드가 종료되어도 쓰레드에서 생성한 자원은 메모리에 남아 있게 된다. 그래서 join or detach로 메모리를 해제 해주어야 한다.

## 5. 예시 코드
```c
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>

void *routine(void *param)
{
	int *arr = (int *)param;
	int i = 0;

	for (int i = 0; i < 5; i++)
	{
		printf("%d\n", arr[i]);
		_sleep(300);
	}
}

int main()
{
	pthread_t thread_id;
	int state;

	int main_Arr[5] = { 1, 2, 3 ,4 ,5};
	int thread_Arr[5] = { 6, 7, 8 , 9, 10};
	pthread_create(&thread_id, NULL, routine, (void *)thread_Arr);
	routine((void *) main_Arr);
	pthread_detach(thread_id);
}
```
`join`함수 대신 `detach`를 사용했다. 저렇게 선언 해두면 해당 쓰레드는 종료되면 자동으로 메모리를 반환한다. 주의할 점은 `detach`가 실행되기전 쓰레드 함수가 실행되고 종료될 수 있다. 이러면 메모리 누수로 이어진다.


# __5. pthread_mutex_init__
## 1. 헤더파일
```c
#include <pthread.h>
```

## 2. 함수 원형
```c
int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr);
```

## 3. 반환 값
성공 시 `0` 반환
실패 시 `error number` 반환

## 4. 함수 설명
`뮤텍스(mutext)`는 우리말로 `상호 배제`라는 기술로서, 같은 데이터를 사용하는 여러 개의 쓰레드들이 서로 겹치지 않게 단독으로 실행되게 하는 기술이다.
여러 개의 쓰레드들이 같은 데이터를 사용하는걸 `임계영역(Critical Section)`이라고 한다. 아래의 코드를 보면 문제가 뭔지 알 수 있다.
```c
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>

int shared_var = 1;

void *routine(void *param)
{
	char *str = (char *)param;
	int i = 0;

	for (int i = 0; i < 5; i++)
	{
		printf("%s : %d\n", str, shared_var++);
		_sleep(300);
	}
}

int main()
{
	pthread_t thread_a_id;
	pthread_t thread_b_id;
	int state;

	int main_Arr[5] = { 1, 2, 3 ,4 ,5};
	int thread_Arr[5] = { 6, 7, 8 , 9, 10};
	pthread_create(&thread_a_id, NULL, routine, (void *)"thread A");
	pthread_create(&thread_b_id, NULL, routine, (void *)"thread B");

	pthread_join(thread_a_id, (void *)&state);
	pthread_join(thread_b_id, (void *)&state);
}
```
![return value2](/42Seoul_philosophers/images/mutex_init_1.png)

생성된 쓰레드는 `shared_var` 전역 변수를 출력하고 1을 증가시킵니다. 예제에서는 하나의 변수값만 같이 사용하므로 큰 문제가 없어보이지만 시스템이 커질수록 큰 문제가 발생할 수 있고 최악의 경우는 `데드락`이 발생할 수 있습니다.

## 5. 예시 코드
```c
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>

pthread_mutex_t mutex;
int shared_var = 1;

void *routine(void *param)
{
	char *str = (char *)param;
	int i = 0;

	pthread_mutex_lock(&mutex);
	shared_var = 1;
	for (int i = 0; i < 5; i++)
	{
		printf("%s : %d\n", str, shared_var++);
		//_sleep(100);
	}
	pthread_mutex_unlock(&mutex);
}

int main()
{
	pthread_t thread_a_id;
	pthread_t thread_b_id;
	int state;

	int main_Arr[5] = { 1, 2, 3 ,4 ,5};
	int thread_Arr[5] = { 6, 7, 8 , 9, 10};

	pthread_mutex_init(&mutex, NULL);
	pthread_create(&thread_a_id, NULL, routine, (void *)"thread A");
	pthread_create(&thread_b_id, NULL, routine, (void *)"thread B");

	pthread_join(thread_a_id, NULL);
	pthread_join(thread_b_id, NULL);
}
```
![dup1](/42Seoul_philosophers/images/mutex_init_2.png)

`뮤텍스`를 사용할려면 초기화를 해줘야 합니다.
`pthread_mutex_init(&mutex, NULL)` 초기화를 진행하며 두 번째 매개변수에 NULL은 기본값을 사용한다는 의미입니다.
```c
pthread_mutex_lock(&mutex);
/*
Critical Section
*/
pthread_mutex_unlock(&mutex);
```
`pthread_mutex_lock`, `pthread_mutex_unlock` 함수로 설정된 `임계구역`을 가진 쓰레드들은 서로 겹치지 않고 단독으로 실행됩니다. 하나의 쓰레드가 `lock`를 하면 다른 대기 하는 쓰레드는 `unlock`가 됐는데 항상 검사해야 하며 이를 `busy waiting`라고 합니다.
위 예시는 A가 실행되어 `lock`되었고 모든 루틴을 실행한 후 `unlock`이 됩니다. 곧바로 B 쓰레드가 요청을 하고 루틴을 실행합니다.
