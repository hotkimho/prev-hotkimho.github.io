---
layout: default
title: Mutex
parent: Philosophers

---

# __mutex__
`mutex`는 `mut`ual + `ex`clusion의 약자를 따서 만든 단어입니다.

공통된 자원을 사용하고, 서로 다른 두 `프로세스` 또는 `스레드`로 동시에 접근해서는 안되는 영역을 `임계구역(Critical Section)`이라고 합니다.
`mutex`는 임계구역에서 `처리 단위`(프로세스, 스레드)가 동시에 접근하지 못하도록 막는 기법입니다.

## __mutex Lock, mutex unlock__
mutex는 2가지의 명령상요할 수 있습니다.

- lock : 임계구역에 들어갈 권한을 얻는다. 다른 `프로세스, 스레드`가 권한을 얻는 상태면 종료할 때까지 대기한다.
- unlock : 임계구역의 권한을 반납한다. 대기중인 다른 `프로세스, 스레드`가 실행된다.

쉽게 생각하면 열쇠를 얻는다고 생각하면 됩니다.

```c
while (true) {
	mutex_lock
		critical section
	mutex_unlock
		remainder section
}
```
위 코드에서 보면 lock하는 순간 권한을 얻고 다른 `스레드`가 권한을 먼저 얻었으면 종료할 때까지 대기합니다. 그리고 권한을 얻는 순간 `critical section`을 수행합니다.

## __busy waiting__
mutex는 lock, unlock의 명령을 통해 권한을 얻으면 실행하고 얻지 못하면 얻을 때까지 대기한다고 했습니다.
이 때 권한을 얻지 못하고 계속 무한루프를 돌며 권한을 얻을 수 있는지 체크하는 걸 `busy waiting`이라고 합니다. 이 때 `context switch`는 일어나지 않습니다.

# __semaphore__
