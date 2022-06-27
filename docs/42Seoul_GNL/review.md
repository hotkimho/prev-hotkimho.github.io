---
layout: default
title: 내용 정리 및 후기
parent: 42Seoul-GNL

---

## gnl의 목표

```
파일 디스크럽터에서 EOF까지 파일을 읽으면서 newline을 만나면 한 줄로 인식하고 읽은 한 줄은 line 변수에 담는다. 그리고 정상적인 동작 or 비 정상적인 동작을 정수형태로 반환한다.

파일을 읽을 때 허용되는 함수는 read함수 이며 읽을 BUFFER_SIZE는 컴파일 시 gcc -Wall -Wextra -Werror -D BUFFER_SIZE=32의 형태로 설정한다.

### get_next_line이 호출될 때마다 파일을 읽고, 읽는 도중 newline을 만나면 읽은 현재 라인을 반환하며, 전체 파일을 읽어서 한줄씩 처리하면 안된다(중요!)

함수 원형
int get_next_line(int fd, char **line)

매개변수
1. 프로그램이 읽을 파일 디스크럽터
2. 읽어왔던 값(읽은 라인을 저장해 반환할 주소)

리턴 값
- 정상적인 동작 1
- EOF에 도달했을 때 0
- 에러가 발생한 경우 -1

허용되는 함수
read, malloc, free

static 변수 허용 가능

*그외에 사용할 문자열 함수는 직접 작성해야 한다.
```

## 주의 사항
- heap에 할당된 메모리는 사용이 완료되면 해제되어야 한다.(메모리 누수는 있으면 안됨)
- 한번에 모든 파일을 읽어서 처리하면 안된다. 컴파일 시 지정된 `BUFFER_SIZE`만큼 씩 읽어야 한다.



## 사용한 변수
- char buf[BUFFER_SIZE + 1]
    - `BUFFER_SIZE` 수 만큼 읽어오고 널문자까지 고려해 + 1개 할당
- static char *str
    - buf에 저장된 문자열들은 함수가 종료되면 사라진다. 문자열들을 저장하기 위한 static 변수

## 사용한 함수
- strlen
- strchr
    - 문자열을 처음부터 끝까지 순회하며, `newline`을 찾으면 찾은 index를 반환, 못 찾으면 음수값(-1)을 반환
- strjoin
    - buf에 저장된 값을 `static` 변수에 저장하기 위해 사용
    - `static`변수는 선언 시 0을 가지고 있으므로, 처음에 합치는 경우를 위해 `static`변수가 NULL이면 ""이란 빈 문자열을 할당
- strdup
    - 빈 문자열을 만들거나, `static`의 값을 새로 나누기 위해 사용
- strncpy
    - strjoin내부에서 사용

## 처음 생각한 방법
1. buf(buffer)에 `BUFFER_SIZE`만큼 값을 읽어온다.
2. buf를 처음부터 끝까지 순회하며 `newline`을 찾는다.
3. `newline`을 찾으면, 찾은곳 까지 `line`변수에 동적할당된 주소를 할당하고 나머지를 `str(static)`변수에 저장(문자열을 합침)하고 `1`을 반환한다.
4. `newline`(buf에 없을 경우)이 없으면 `str`을 탐색하며 `newline`을 찾는다. 여기서 `newline`을 찾을 경우 3번과 같이 찾은 부분을 line변수에 동적할당된 주소를 할당한다.
5. `str`에서도 `newline`을 못 찾으면 1번부터 다시 파일을 읽어와서 탐색한다.
6. 1 ~ 5번의 과정을 반복하다보면 결국 파일을 다 읽고 읽은 문자열들은 `str`에 저장될 것이다. 파일을 다 읽은 순간부터는 4번의 과정을 반복한다. 만약 `str`에서 `newline`을 찾지 못하면 읽은 파일의 마지막 내용으로 판단하고 `str`을 line변수에 할당하고 0을 반환한다.

위의 방법으로 진행 시, `buf`검사 -> `str`검사 이런식으로 이어지기에 과정이 복잡하고 코드로 구현시에도 불필요한 부분이 많았다. 2 ~ 3번의 과정을 삭제하고 `buf`가 값을 읽어오는 즉시 `str`변수에 문자열을 합치고 4번 과정으로 바로 진행을 했다.

## 구현한 방법
1. `buf`에 `BUFFER_SIZE`만큼 값을 읽어온다.
2. `buf`를 `str`변수에 `strjoin`을 사용해 합친다.
3. `str`을 순회하며 `newline`을 찾으면 찾은곳 까지 `line`변수에 동적할당하여 주소를 할당하고 `1`을 리턴한다.
4. `newline`을 못 찾으면 1번부터 다시 시작하여 문자열을 읽어오고 다시 `newline`을 찾는다.
5. 파일 내용을 모두 읽으면 읽는 과정을 생략하고, 3번 과정부터 시작한다. 만약 `newline`을 찾지 못하면 마지막내용으로 생각하고 남은 문자열을 line에 할당하고 0을 반환한다.

`str`에서 `newline`을 찾고 새로운 공간을 동적할당 하고 사용된 메모리는 해제하는 자세한 작업은 위 방법에 작성하지 않았다(귀찮..).
## 예외 처리
- `static`변수가 NULL의 값을 가지고 있을 때 `빈 파일`인지 `첫 선언`인지 어떻게 구분할까?
    - 파일을 읽어 `buf`에 값을 저장한 후 `strjoin`함수를 이용해 `str`에 문자열을 합치고 `strjoin`함수안에 `str`이 NULL 이면 `strdup`함수를 이용해 `""`으로 초기화 한다. `첫 선언`이면 이 과정을 통해 자연스럽게 할당이 될것이고 이런 과정을 거치지 않고 `str`을 사용할 때 NULL값을 가지고 있다면 빈 파일로 간주한다.
- `buf`에 값을 읽어올 때 `read`함수를 사용하는데 이 함수의 반환값은 (읽어온 값의 크기, 0(다 읽었을 경우), -1(제대로 읽지 못한 경우)을 가진다. -1이 반환된 경우 잘못된 행동으로 생각하고 예외처리를 진행한다.

## 문제를 해결하며 궁금한 점
- `static` 변수는 프로그램이 종료되면 자동으로 메모리가 해제된다. 그러면 `static` 포인터 변수가 `heap`에 메모리를 가르키고 있는 경우 프로그램이 종료되면 `static` 변수가 해제될 때 `heap`의 메모리가 해제될까?
    - 검색해도 정확한 정보를 얻을 수 없어서 나는 프로그램이 종료되면 `static`변수는 해제되지만 `heap`에  할당된 영역은 해제가 안 될거라고 생각하고 코드를 작성했다. 이 내용은 평가를 진행하며 나는 이렇게 생각해서 이렇게 코드를 작성했다 라고 설명했고 한 `귀인`을 만나 내가 생각한게 정답이라는걸 알았다.