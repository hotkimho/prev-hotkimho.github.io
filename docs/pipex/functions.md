---
layout: default
title: Functions
parent: Pipex

---


# __1. fork__
## 1. 헤더파일
```c
#include <unistd.h>
```

## 2. 함수 원형
```c
pid_t fork(void);
```

## 3. 반환 값
성공 시 부모 프로세스는 `자식 프로세스의 프로세스 ID` 리턴  
자식 프로세스는 `0` 리턴  
실패 시 `EOF` 리턴  

## 4. 함수 설명
`fork`함수는 프로세스를 생성하는 함수입니다.  
새롭게 생성된 프로세스를 `자식 프로세스`라고 하고 자식 프로세스를 만든 프로세스를 `부모 프로세스`라고 합니다.  
그리고 생성된 프로세스는 `프로세스 ID`라는 번호를 부여받아 운영체제에서 관리 하게 됩니다.  
우리 입장에선 프로세스 ID를 확인해도 이게 자식인지, 부모인지 구분할 수 없습니다.  
두 프로세스를 구분하는 유일한 방법은 fork함수의 `return value`을 확인하는 것입니다.  
fork함수는 프로세스 ID(이하 pid)를 반환합니다. 부모는 `자식 프로세스의 pid`를 반환하고, 자식은 `0`을 반환합니다.  
즉 fork함수의 반환값이 0이면 자식 프로세스, 아니면 부모 프로세스를 의미합니다.  
fork가 정상적으로 실행되지 않으면 `-1`을 반환하기 때문에 이 경우는 예외처리를 별도로 해주어야 합니다.  

## 5. 예시 코드
```c
#include <stdio.h>
#include <unistd.h>

int main(void)
{
	//프로세스 ID를 저장할 변수
	int pid;

	if ((pid = fork()) == -1)
		printf("프로세스 생성 실패\n");
	
	if (pid == 0)
		printf("(자식 프로세스) fork 리턴 값 : %d\n", pid);
	else
		printf("(부모 프로세스) fork 리턴 값 : %d\n", pid);
	printf("PID(%d) 종료\n", pid);
}
```
![1.return value](/docs/pipex/images/1.return_value.png)

코드를 보면 fork함수의 반환값을 `pid`변수에 담고, pid값이 0이면 자식 프로세스의 `pid 반환값`  
, pid값이 0이 아니면 부모 프로세스의 `pid 반환값`을 출력한다.  
결과를 보면 부모 pid 반환값은 20573, 자식의 pid 반환값은 0을 의미한다는 걸 알 수 있다.  
그리고 실행 순서가 부모 실행 -> 부모 종료 -> 자식 실행 -> 자식 종료 순서로 결과가 나왔지만 항상 이 순서를 보장하지 않는다. 즉 `순차적으로 실행`되는게 아님을 주의 해야 한다.  


# __2. wait__
## 1. 헤더파일
```c
#include <sys/wait.h>
#include <sys/types.h>
```

## 2. 함수 원형
```c
pid_t wait(int *wstatus);
```

## 3. 반환 값
성공 시 `종료된 자식 프로세스의 프로세스 ID 반환`
실패 시 `-1`  

## 4. 함수 설명
함수의 이름 그대로 기다리는 함수이다.   
어떤걸 기다리냐면 `fork`를 사용해 `자식 프로세스`를 생성했을 때, `부모 프로세스`에서 `자식 프로세스`가 종료될 때 까지 기다린다.   
하지만 자식 프로세스가 종료 되지 않는다면 부모 프로세스에서 `무한 대기`상태가 될 수 있다. 
```
그럼 왜 자식 프로세스가 먼저 실행되고 종료되길 기다리는가??
fork 함수가 실행되면 부모와 자식 프로세스의 실행 순서를 보장할 수 없다고 했다.   
부모 프로세스가 먼저 종료되면 자섹 프로세스는 고아 프로세스(PPID(1))가 된다.  
이 문제를 방지하기 위해 wait을 사용한다.
```
## 5. 예시 코드
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main(void)
{
	//프로세스 ID를 저장할 변수
	int pid;
	int wstatus;

	if ((pid = fork()) == -1)
	{
		printf("프로세스 생성 실패\n");
		exit(0);
	}
	
	if (pid == 0)
	{
		printf("자식 프로세스 시작\n");
		for (int i = 0; i < 3; i++) 
        {
            printf("Child : %d\n", i);
            sleep(2);
        }
		printf("자식 프로세스 종료\n");
        exit(3);
	}
	else
	{
		printf("부모 프로세스 시작\n");
		printf("부모 PID: %d\n",getpid());
		printf("자식 PID: %d\n",pid);
		int child_pid = wait(&wstatus);
		printf("wait 인자값: %d\n", wstatus);
		printf("wait 반환값: %d\n", child_pid);
		printf("부모 프로세스 종료\n");
		exit(0);
	}
}
```
![return value2](/docs/pipex/images/2.return_value.png)
위 결과를 보면 `부모 프로세스 -> 자식 프로세스` 순서로 실행됐지만 부모 프로세스에서 자식 프로세서가 종료되기를 기다린다.   
그리고 `wait` 함수는 `종료된 자식 프로세스의 PID`를 반환한다.


# __3. waitpid__
## 1. 헤더파일
```c
#include <sys/wait.h>
```

## 2. 함수 원형
```c
pid_t waitpid(pid_t pid, int *status, int options);
```

## 3. 반환 값
성공 시 `종료된 자식 프로세스의 프로세스 ID 반환`
실패 시 `-1`  

## 4. 함수 설명
`waitpid` 함수는 `wait` 함수와 같이 `자식 프로세스`의 상태를 획득하고 자원을 수거 및 반환하는 함수이다.   
인자는 3개(pid_t pid, int *status, int options)를 받으며, `반환값과 2번 째 인자`는 `wait함수와 동일`하다. 그래서 1, 3번만 설명을 하겠다.   
그 전에 waitpid 와 wait의 차이를 알아보자   
wait 함수는 fork를 통해 생성한 `자식`이 `1개가 아니라 여러 개`인 경우, wait 함수를 사용한 `부모 프로세스`는 종료된 임의의 `자식 프로세스`를 받는다. 즉 특정한 자식 프로세스를 지목할 수 없다. `waitpid` 함수는 이런 문제를 `pid`인자를 받음으로써 해결할 수 있게 한다.

__첫 번째 인자__
<div markdown="1">

| 첫 번째 인자 | 내용 |
|:-----------	 |:-|
| pid가 -1인 경우  | 임의의 자식 프로세스를 기다린다 |
| pid가 양수인 경우 | 프로세스 ID가 pid인 자식 프로세스를 기다린다 |
| pid가 -1 보다 작은 경우 | 프로세스 그룹 ID가 pid의 절대값과 같은 자식을 기다린다 |
| pid가 0인 경우 | waitpid 호출한 프로세스(부모)ㅇ의 프로세스 그룹 PID와 같은 프로세스 그룹 ID를 가진 프로세스를 기다린다 |

</div>

__세 번째 인자__
<div markdown="1">

| 세 번째 인자 | 내용 |
|:-----------	 |:-|
| WNOHANG  | 자식 프로세스의 종료를 기다리지 않고 상태를 반환한다 |
| WUNTRACED | 중단된 자식 프로세스의 상태를 반환한다 |
| WCONTINUED | 중단되었다가 다시 실행된 자식 프로세스의 상태를 반환한다. |

</div>

# __4. pipe__
## 1. 헤더파일
```c
#include <unistd.h>
```

## 2. 함수 원형
```c
int pipe(int fd[2]);
```

## 3. 반환 값
성공 시 `0`    
실패 시 `-1`

## 4. 함수 설명
`pipe`는 `프로세스간 통신`을 할 때 사용되는 기법이다. pipe는 `IPC`의 기법중 하나로 모든 유닉스 시스템이 제공한다.
<span class="fs-2">
[IPC](https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4_%EA%B0%84_%ED%86%B5%EC%8B%A0){: .btn}
</span>   
`pipe`는 원소를 2개 가지는 `int`형 배열을 인자로 받는다.   
`pipe`함수가 정상적으로 실행되면 커널영역에 파이프가 생성된다.

![pipe1](/docs/pipex/images/4.pipe_1.png)
파이프는 커널영역에 생성되고, 생성한 프로세스는 생성한 파이프에 대한 `파일 디스크럽터`를 갖는다. 파일디스크립터 fd[0]은 `읽기`, fd[1]은 `쓰기`용 파이프입니다. 그래서 fd[1]에 데이터를 쓰게 되면 fd[0]을 통해 데이터를 읽을 수 있습니다.   

파이프를 이용해 `다른 프로세스(부모자식)간 통신`을 하려면 어떻게 해야할까?   
우선 `fork`를 통해 자식 프로세스를 생성한다. __여기서 중요한 것은 자식 프로세스를 생성할 때 부모 프로세스의 파일 디스크립터도 복제된다.__ 그래서 `부모 프로세스`에서 fd[1]에 값을 쓰면 `자식 프로세스`의 fd[0]을 통해 읽을 수 있다.

![pipe2](/docs/pipex/images/4.pipe_2.png)
그림을 보면 `parent`에서 `fork`를 하고 파일 디스크립터 `fd[1]`을 이용해 파이프에 데이터를 `write`하면, `child`에서는 같은 파일디스크립터를 공유하고 있으므로 `fd[0]`을 이용해 데이터를 `read`하면 된다.

## 5. 예시 코드
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main(void)
{
	int fd[2];
	pid_t pid;
	char buf[1024];

	if ((pipe(fd)) == -1)
	{
		printf("파이프 생성 실패\n");
		exit(1);
	}
	if ((pid = fork()) == -1)
	{
		printf("프로세스 생성 실패\n");
		exit(1);
	}
	
	if (pid == 0) //자식 프로세스
	{
		close(fd[1]);
		read(fd[0], buf, 1024);
		printf("자식에서 읽은 값: %s\n", buf);
	}
	else //부모 프로세스
	{
		close(fd[0]);
		strcpy(buf, "부모 프로세스에서 write");
		write(fd[1], buf, strlen(buf));
	}
	exit(0);
}
```
![pipe3](/docs/pipex/images/4.pipe_3.png)
`자식, 프로세스`를 보면 각각 사용하지 않는 파일 디스크립터는 `close`를 했습니다. `사용하지 않는 파일디스크립터`를 정리하기 위함입니다.   
부모 프로세스에서 `부모 프로세서에서 write`문자열을 `fd[1]`에 write했고, 자식 프로세스에서 `fd[0]`을 통해 데이터를 read해서 출력했습니다.

# __5. dup__
## 1. 헤더파일
```c
#include <unistd.h>
```

## 2. 함수 원형
```c
int dup(int fd);
```

## 3. 반환 값
성공 시 `파일 디스크립터를 복사하여 반환`   
실패 시 `-1`

## 4. 함수 설명
`dup` 함수는 이름 그대로 `파일 디스크립터`를 인자로 받고 복사하여 반환하는 함수입니다. 하지만 `복제`를 하는것이지 원본과 똑같은 파일디스크립터가 아닙니다.   
복제할 때, 복제되는 `파일 디스크립터`의 값은 현재 사용하고 있지 않는 가장 작은 파일 디스크립터값을 반환 합니다.   

## 5. 예시 코드
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main(void)
{
	int fd[2];
	int dup_fd;
	char buf[1024];

	pipe(fd);
	dup_fd = dup(fd[0]);
	
	write(fd[1], "hi", 3);
	read(dup_fd, buf, 1024);
	printf("dup_fd을 통해 읽어온 값: %s\n", buf);
	printf("fd[0]: %d\n", fd[0]);
	printf("dup_fd: %d\n", dup_fd);
}
```
![dup1](/docs/pipex/images/5.dup_1.png)

위 코드에서 `dup_fd`에 `fd[0]`를 복제 했습니다. fd[0]의 `파일 디스크립터 테이블`의값이 `3`인 경우 fd[0]과 dup_fd를 출력했을 때, 같은 값을 가지고 있지만 출력되는 `파일 디스크립터`의 값이 다릅니다.   
그 이유는 복제된 `dup_fd`는 `파일 디스크립터 테이블` 새롭게 추가되고 `fd[0]`가 가르키는 `파일 테이블`을 같이 가르킵니다. 즉 함수 이름 dup에 맞게 새롭게 복제되어 같은 값을 가르키는 파일 디스크립터를 반환합니다.   
그래서 `dup`함수를 사용했으면 복제된 `파일 디스크립터`변수도 close해줘야 합니다.


# __6. dup2__
## 1. 헤더파일
```c
#include <unistd.h>
```

## 2. 함수 원형
```c
pid_t dup2(int fd, int fd2);
```

## 3. 반환 값
성공 시 `두 번째 매개변수 파일 디스크립터 반환`    
실패 시 `-1`

## 4. 함수 설명
`dup`함수는 매개 변수에 2개의 `파일 디스크립터`를 입력 받습니다. 그리고 `첫 번째 인자`를 `두 번째 인자`에 복제합니다. 그리고 복제된 파일 디스크립터를 반환 합니다.   
만약 `두 번째 인자`가 `open`상태라면 `close`를 진행하고 복제를 진행합니다.

## 5. 예시 코드
```c
#include <stdio.h>
#include <unistd.h>

int main(void)
{
	int fd[2];
	int fd2[2];
	int temp_fd;

	char buf[1024], buf1[1024], buf2[1024];

	pipe(fd);
	pipe(fd2);
	temp_fd = dup2(fd[1], fd2[1]);

	write(fd[1], "hello", 6);
	read(fd[0], buf, 1024);
	printf("fd[1]에 쓴 데이터 : %s\n", buf);

	write(fd2[1], "kimho", 6);
	read(fd[0], buf1, 1024);
	printf("fd2[1]에 쓴 데이터 : %s\n", buf1);

	write(temp_fd, "12345", 6);
	read(fd[0], buf2, 1024);
	printf("temp_fd에 쓴 데이터 : %s\n", buf2);
}
```
![dup2_1](/docs/pipex/images/6.dup2_1.png)
`temp_fd = dup2(fd[1], fd2[1])` 코드를 실행함으로써 `fd[1]`을 `fd2[1]`에 복제하고 반환합니다.   
그리고 각각 `fd[1], fd2[1], temp_fd`에 문자열을 `write`하고 `fd[0]`을 통해 `read`하면 저장한 문자열이 모두 출력된 것을 볼 수 있습니다.


# __7. access__
## 1. 헤더파일
```c
#include <unistd.h>
```

## 2. 함수 원형
```c
int access(const char *path, int mode);
```

## 3. 반환 값
성공 시 `0`    
실패 시 `-1`

## 4. 함수 설명
파일 명대로 파일을 확인하는 함수입니다. 첫 번째 인자에 해당하는 `file`을 두 번째 인자값인 `mode`값으로 확인을 합니다.   
mode값으로 파일을 검사할 때, `mode` 에 해당하는 내용이 가능하면 `1`, 실패하면 `-1`을 반환합니다.   
__Mode 값__
<div markdown="1">

| mode | 내용 |
|:-----------	 |:-|
| F_OK | 파일 존재 확인 |
| R_OK | 읽기 확인 |
| W_OK | 쓰기 확인 |
| X_OK | 실행 확인 |

</div>

# __8. execve__
## 1. 헤더파일
```c
#include <unistd.h>
```

## 2. 함수 원형
```c
int execve(const char *pathname, char *const argv[], char *const envp[]);
```

## 3. 반환 값
성공 시 `별도의 프로세스가 생성되고 정상적으로 실행되면 0 반환`    
실패 시 `-1`

## 4. 함수 설명
`execve` 함수는 `exec` 계열의 함수입니다. `exec` 계열 함수는 보통 매개변수로 입력된 파일을 실행하는 함수입니다.   
`execve` 함수는 3개의 인자를 받습니다. 첫 번째 인자로 받은 `pathname`의 파일을 실행하고, `argv`, `envp` 두, 세 번째 매개변수 값을 전달합니다.   
첫 번째 인자에 대해 `실행 권한` 확인이 필요하며 이는 `access`함수를 통해 검사가 가능합니다.
`argv`는 첫 번째 인자로 실행되는 파일에 대한 옵션입니다. 예를 들어 첫 번째 파일이 `ls`명령을 실행하면 `argv`안에 `argv[]={"ls", "-l"}` 와 같이 옵션값이 들어가며 가장 `첫 번째`인덱스에는 명령어의 이름이 들어가야 합니다.   
`evnp`는 환경 설정 파일이다.
`argv, envp`모두 char *[] 배열 형태이기에 마지막 인덱스에 `NULL`값을 넣어줘야 한다.   

## 5. 예시 코드
```c
#include <stdio.h>
#include <unistd.h>

int main(int argc, char **argv, char **envp)
{
	execve("/bin/ls", argv, NULL);
}
```
![execve_1](/docs/pipex/images/8.evecve_1.png)
코드를 보면 첫 번째 인자로 `/bin/ls` 파일을 지정했고 실핼할 때 `-l`옵션을 주었다.   
그래서 `ls -l`명령어가 실행된다.