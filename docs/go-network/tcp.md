---
layout: default
title: Tcp
parent: Go-network
---

# 리스너
`golang` 표준 라이브러리의 `net`패키지를 이용해 `TCP` 연결을 수립해보겠습니다. `net`패키지의 `net.Listen`함수를 사용하면 수신 연결 요청 처리가 가능한 `TCP`서버를 만들 수 있습니다. 이런 서버를 `리스너(Listener)`라고 합니다.  
```golang
func Listener() {
	//리스너에 IP와 포트 번호가 바인딩됨
	listener, err := net.Listen("tcp", "127.0.0.1:3000")
	if err != nil {
		fmt.Println("listener(): Error net.Listen func")
		return
	}
	defer func() { _ = listener.Close() }()
	fmt.Printf("bound to %s", listener.Addr())
}
```