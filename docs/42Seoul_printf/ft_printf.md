---
layout: default
title: 42Seoul-Printf
has_children: true
permalink: /docs/42Seoul-Printf
---

# 1. Subject
- [Subject](https://github.com/hotkimho/42cursus/blob/master/gnl/reference/get_next_line_subject.pdf)

# 2. 규칙
- `전역변수`는 사용할 수 없습니다
-  필요한 함수는 직접 구현해야합니다.
- `허용하는 함수(system call)`외에 모든 함수의 사용을 금지합니다.

# 3. ft_printf는 어떤 프로젝트 인가요?
`ft_printf`는 c언어의 `printf`함수를 구현하는 프로젝트 입니다.

이 프로젝트의 목적은 `가변 인자`에 대한 학습과 `printf`와 똑같은 동작 및 예외처리를 구현하는 것입니다.

문제에서 제시되지 않은 동작은 [man printf](https://man7.org/linux/man-pages/man3/printf.3.html) 사이트를 참고했습니다.

# 4. 함수 원형
`int ft_printf(const char *, ...);`

`printf`와 똑같이 문자열 포맷을 입력받고, 뒤에 `가변 인자`를 통해 값을 입력받습니다.

반환값은 출력된 `바이트 수`입니다.

# 5. 구현 목록(서식 지정자(옵션))
- `%c` `단일 문자(char)`를 출력합니다.
- `%s` `문자열(string)`를 출력합니다.
- `%p` `void * 형식의 포인터 인자를 16진수`로 출력합니다.
- `%d` `10진수 숫자`를 출력합니다.
- `%i` `10진수 정수`를 출력합니다.
- `%u` `부호 없는 10진수 숫자`를 출력합니다.
- `%x` `소문자 16진수`를 출력합니다.
- `%X` `대문자 16진수`를 출력합니다.
- `%%` `%` 기호를 출력합니다.

# 6. 평가
![img.png](/docs/42Seoul_printf/images/printf_eval_1.png)

![img.png](/docs/42Seoul_printf/images/printf_eval_2.png)