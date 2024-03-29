---
layout: default
title: process
parent: OS
---

# Process란 무엇인가?
프로세스의 정의를 찾아보면 `실행중인` 프로그램이라고 정의됩니다. 저는 정의된 내용을 보고 "아 프로세스는 이런거구나!" 라고 이해가 되지 않습니다. 위 말을 제 방식으로 표현하면 프로세스는 `프로그램을 실행하는 단위`로 표현할 수 있습니다  

게임을 예로들어 `쿠키런`이라는 게임을 실행한다고 했을 때, 실행파일 즉 쿠키런 게임에 대한 코드는 저희 `하드 디스크`에 저장이 되어있고 이를 실행하면 `메모리(RAM)`에 프로세스의 형태로 올라가게 됩니다. 그리고 메모리에 쿠키런 프로세스가 올라가면 저희는 쿠키런이 `실행`됐다 라고 말합니다

많은 사람들이 프로그램이 멈출 때 `작업 관리자`에서 특정 프로세스를 종료시킨적이 있을 겁니다. 그리고 그 안에 `무수히 많은 프로세스`들이 있는걸 봤습니다. 이 많은 프로세스들을 어떻게 다 실행할 수 있는걸까요? `운영 체제`는 `시분할`방식을 이용하여 각각의 프로세스에 일정시간을 할당하여 처리합니다. 여러 개의 `프로그램(프로세스)`가 동시에 처리된 것 처럼 보이지만 `CPU`가 프로세스마다 시간을 할당하고 할당한 시간만큼 연산을하여 동시에 실행된 것 처럼 보이는겁니다.

여러개의 프로세스를 CPU가 돌아가며 처리한다고 말씀을 드렸는데 `A프로세스`를 실행하고 시간이 다 끝나면 `B, C, D...`등 다른 프로세스를 실행합니다 이렇게 다른 프로세스로 교체하는걸 `문맥교환(Context Switching)`이라고 합니다. 즉 현재 작업중인 프로세스에 대한 작업을 `완료 or 중단`하고 다른 프로세스를 연산하는걸 의미합니다. 그리고 문맥교환을 하기위해선 프로세스에 대한 `정보`가 필요한데 이 정보를 `PCB(Process Control Block)`라고 합니다. `프로세스가 생성이되면 PCB또한 같이 생성되고 운영체제는 PCB값으로 프로세스의 정보를 확인합니다.` 이 두 개념은 아래에서 자세히 기술하겠습니다.


# PCB(Process Control Blocks)
`PCB`는 프로세스를 실행하는데 중요한 정보를 담고 있고 프로세스가 `생성`될 때 같이 생성됩니다 또한 프로세스가 종료되면 같이 사라집니다.

## PCB 구성
![pcb](/docs/OS/images/pcb.png)  
__(쉽게 배우는 운영체제 pcb image)__

### 1. 포인터
하나의 프로세스가 실행중이고 다른 프로세스는 실행하기를 기다리고 있습니다. 이를 `대기 상태`라고 부르고 이들은 `대기 큐(Queue)`에서 실행을 대기합니다.  
![pcb_queue](/docs/OS/images/pcb_pointer.png)

__(쉽게 배우는 운영체제 pcb image)__  
그림과 같이 `포인터 주소`를 이용하여 큐를 구성합니다. 이름 그대로 PCB의 주소를 의미합니다.  
대기 큐의 PCB순서는 `스케쥴링`이라는 여러 알고리즘에 의하여 순서가 결정됩니다.

### 2. 프로세스 상태(status)
프로세스는 `생성, 준비, 실행, 대기, 보류`등 여러 상태가 있습니다. 현재 어떤 상태인지를 의미하는 값입니다.

### 3. 프로세스 구분자(ID)
프로세스는 각자 `고유`의 ID(정수)값을 가지고 있습니다. `운영체제도 프로세스를 구분`해야하므로 고유한 ID를 블록에 가지고 있습니다

### 4. 프로그램 카운터(PC)
프로세스들이 CPU를 에게 할당받아 사용하는 시간을 `타임 슬라이스`라고 합니다. `A`에서 CPU를 `선점`하여 50%만큼 작업을 하다가 `B`가 CPU를 `선점`했습니다. 일정 시간이 흐르고 난 후 `A가 다시 실행`됐을 때 `어떻게 50%지점부터 시작`할 수 있을까요? `프로그램 카운터(PC)`에 다음 실행할 명령어의 위치를 가리켜 다시 실행되었을 때 이어서 작업할 수 있습니다. 즉 간단히 말해서 `다음에 실행할 명령어`를 가지고 있습니다.

### 5. 프로세스 우선순위
`1.포인터` 그림에서 대기 큐는 한개 였는데 실제로는 여러 개의 큐를 가지고 있습니다. 대기 상태의 큐는 `우선순위에 따라 여러개`가 있으며 `우선순위가 낮은 프로세스`가 먼저 실행됩니다. 우선순위가 낮은 프로세스는 주로 `커널 프로세스`가 해당됩니다.

### 6. PPID, CPID
부모 프로세스, 자식 프로세스에 대한 정보를 가지고 있습니다. 프로세스는 자기 자신을 `복제(fork)`할 수 있으며 복제된 프로세스를 `자식`, 복제를 실행한 프로세스를 `부모` 프로세스라고 합니다.

# 프로세스 상태(Context Switching)
![context_switching](/docs/OS/images/context_switching.png)
__(쉽게 배우는 운영체제 image)__   
그림을 보면 5가지의 상태가 있습니다. 이 상태는 `프로세스가 가지는 상태`이고 `pcb의 블록값`으로도 가지고 있습니다. 각각 어떤 상태인지 알아봅시다.

### 1. 생성 상태
프로그램이 `메모리에 올라오고` 운영체제에 의해 `pcb가 할당받은 상태`를 의미합니다. 즉 프로그램이 방금 막 실행한 상태입니다. 프로세스는 생성되면 바로 실행되는게 아니라 `준비 상태`에서 실행되기를 기다립니다.

### 2. 준비 상태
실행 대기준인 프로세스가 기다리는 상태를 의미합니다. `pcb`가 `준비 큐(ready queue)`에서 대기하며 `스케쥴러`에 의해 관리됩니다.  
프로세스가 준비 상태에서 실행 상태가 되는걸 `dispatch`라고 합니다. 내부적으로 이 명령은 `dispatch(process id)`명령을 통해 실행됩니다.  

### 3. 실행 상태
프로세스가 CPU를 할당받아 실행되는 상태입니다. 실행 상태인 프로세스의 수는 `CPU의 코어 수`입니다. CPU에서 실질적으로 연산을 하는 건 코어이기 때문에 코어 수 만큼 프로세스가 실행 상태가 됩니다  
프로세스가 주어진 `타임 슬라이스`내에 정해진 작업을 완료하지 못하면 `timeout`이 실행되어 `준비 상태`로 돌아가고 다시 실행되기를 기다립니다. 만약 실행 상태에 있는 프로세스가 `입출력`을 요청하면 cpu는 `입출력 관리자`에게 명령하고 해당 프로세스는 `대기 상태`가 됩니다. 이 명령을 `block`라고 합니다. 해당 프로세스는 입출력이 완료될 때 까지 작업을 진행할 수 없으므로 `대기 상태`가 되고 입출력이 완료되면 `준비 상태`가 됩니다. `block`이 수행되면 실행 프로세스 자리가 하나 비었으므로 준비 상태에 있는 프로세스중 새로운 프로세스를 `dispatch`시킵니다. 

### 4. 대기 상태
대기 상태는 `실행 상태`의 프로세스가 `입출력`을 요청하면 `입출력이 끝날 때 까지 기다리는 상태`입니다. 입출력이 완료되면 인터럽트가 발생하여 `준비 상태`가 되는데 이를 `wakeup`이라고 합니다.

### 5. 완료 상태
프로세스가 자신의 일을 끝마친 상태로 `코드와 데이터`를 메모리에서 해제하고 `PCB`또한 해제합니다. 정상적인 종료는 `exit()`명령으로 처리하지만 비정상적인 종료면 종료 직전의 메모리를 옮기는데 이를 `코어 덤프`라고 합니다.

# 문맥 교환(context switching)
문맥 교환의 정의는 CPU를 사용중이던 프로세스는 나가고 새로운 프로세스를 받아 들이는 작업을 의미한다.

### 문맥 교환 절차
![co](/docs/OS/images/context_switching_2.png)  
__(쉽게 배우는 운영체제 image)__  
프로세스 P1과 P2의 문맥 교환 과정이다. p1의 `타임 슬라이스`를 다 사용하고 `timeout`이 발생하고 p2는 `dispatch`가 발생하여 cpu를 선점한다. 또 다시 `타임 슬라이스`가 다하고 나면 p2에서 `timeout`가 발생하고 p1에서는 `dispatch`가 발생하여 CPU를 선점한다.  
그림에서는 두 프로세스로만 나와 있지만 실제 운영체제에서는 많은 프로세스가 위 과정으로 CPU를 선점한다.

  
    
# 프로세스 구조
프로세스는 4개의 영역으로 구성되어 있습니다. 크게 `정적`인 영역과 `동적`인 영역으로 나뉩니다. 이 두 차이는 `코드, 데이터` 영역에 대한 설명을 보고 `스택, 힙` 영역에서 자세히 설명드리겠습니다.  


### 코드 영역
코드 영역은 이름 그대로 `프로그램의 코드`가 저장된 영역입니다. 개발자가 작성한 코드가 이 영역에 탑재되며 이 코드 영역은 `읽기 전용`입니다. 한번 실행된 프로그램이 수정되면 안되기 때문에 일기 전용으로 처리됩니다.  


### 데이터 영역
코드가 실행되면서 사용하는 `변수, 파일`등을 모아놓은 영역입니다. 데이터는 값이 변하기 때문에 이 영역은 `읽기와 쓰기`가 가능합니다. 물론! 상수는 수정이 되지 않습니다.  

```c
main() {
	int a = 1, b = 2;
	printf("%d %d\n", a, b);
	exit(0);
}
```  

이 코드에서 변수 `int a = 1, b = 2`가 데이터 영역에 탑재가 되고 이 코드는 `코드 영역`에 탑재됩니다.  

### 정적, 동적 영역
코드 영역은 크게 두 가지로 나뉘어집니다. 정적 영역(`코드, 데이터`)과 동적 영역(`스택, 힙`)입니다. 무슨 차이가 있을까요??  

`정적 영역`은 이름 그대로 크기가 고정인 영역입니다. 그리고 코드, 데이터 영역은 `프로그램이 프로세스로 메모리에 올라갈 때 크기가 고정된 상태`로 영역이 생성됩니다.  

```c
main() {
	int a = 1, b = 2;
	printf("%d %d\n", a, b);
	exit(0);
}
```  

이 코드가 프로세스로 실행되면 `코드(코드 영역)`, `데이터(int a= 1, b = 2)` 영역으로 고정된 크기로 영역이 할당됩니다.

`동적 영역은 아래 설명할 스택, 힙에서 각각 설명드리겠습니다.`  


### 스택 영역
스택 영역은 프로세스가 실행되는 동안 크기가 가변적으로 변합니다 그래서 `동적 할당 영역으로 분류할 수 있습니다.`

`스택 영역`은 운영체제가 프로세스를 실행하기 위해 데이터를 모아놓은 곳입니다. 필요에 따라 데이터를 `push`하고 `pop`하여 항상 길이가 가변적입니다. 
스택 영역은 어떤 데이터가 저장되는지 알아보겠습니다.

```c
int main() {
	int a = 1, b = 2;
	add(a, b);
}
void add(int c, int d) {
	printNum(c, d);
	printf("%d\n", c * d);
}
void printNum(int num1, int num2) {
	printf("%d %d\n", num1, num2);
}
```  

위 코드는 `main` -> `add`-> `printfNum` 출력 실행 -> `add` 변수 출력 -> `main` 의 순서로 동작됩니다.

`스택 영역` 영역에는 함수를 호출하면 돌아와야할 주소가 저장됩니다. `PrintNum` 함수가 실행되고 있으면 스택은 아래와 같이 되어 있습니다.  

PrintNum() - Top
add()  

`PrintNum`함수가 실행이 종료되면 스택의 `Top`이 `Pop`되어 사라지고 `add`함수의 주소로 돌아갑니다.  

add()  

스택에는 `add`함수만 저장되어 있고 이 함수도 실행하고 종료되면 `pop`되어 사라집니다.  

### 왜 스택 영역에 이런 정보들을 저장하는 걸까요?
함수가 실행되고 돌아가는 순서가 `스택 자료구조`의 동작과 굉장히 비슷하기 때문입니다. 처음에 언어를 설계할 때 이렇게 구조를 동작되도록 만든건지는 모르겠습니다. 간단한 예제를 만들어 봤습니다.  

```c
int main() {
	a()
}
void a() {
	b()
	printf("a ")
}
void b() {
	c()
	printf("b ")
}
void c() {
	printf("c ")
}
```  

이 코드의 결과는 `c b a`입니다. 어찌보면 당연합니다 `a`함수가 실행되고 출력하기전에 `b`함수가 실행됩니다. `b`함수 또한 값이 출력하기 전 `c`함수가 실행됩니다. `c`함수에서 값을 출력하고 종료합니다. 함수가 호출되는 순서는 `a -> b-> c`지만 실행되고 `데이터를 삭제 하는건 반대 방향`입니다. 이런 결과가 나오기 때문에 함수 호출의 주소를 스택에 저장하는 것 같습니다.  

위 설명에서 `함수의 주소`를 스택영역에 저장된다고 말씀 드렸는데 실제론 해당 함수의 `지역 변수`와 `매개 변수`도 같이 저장됩니다.  


### 힙 영역
힙 영역은 `동적 할당`이 되는 데이터가 저장되는 영역입니다.

```c
int main() {
	int *arr;

	arr = (int *)malloc(sizeof(int) * 100);
	free(arr);
	exit(0);
}
```  

`malloc` 함수로 `int * 100`의 크기만큼 동적할당 했습니다. 그러면 이 생성된 메모리는 `힙 영역`에 저장됩니다. 이 메모리는 따로 `free(해제)`하지 않는이상 계속 힙 영역에 남아 있으므로 적절히 사용하고 해제 해줘야 합니다. 안 그러면 힙에 계속 남아 있고 이런 상황을 `메모리 누수`라고 합니다.