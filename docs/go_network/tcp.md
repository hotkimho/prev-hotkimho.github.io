---
layout: default
title: Tcp
parent: Go-network
---

# 들어가며
작성한 함수들은 모두 테스트용으로 만들었습니다. 그래서 함수에서 `error`를 반환하지 않고 출력만 했습니다.

# TCP 수신 연결 응답 및 요청
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
`net.Listen` 함수는 `네트워크의 종류(tcp)`와 `ip:port`를 매개변수로 받습니다. `net.Listen` 함수는 `net.Listener` 인터페이스와 `error` 인터페이스를 반환합니다.

함수가 성공하면 인자로 입력한 `ip:port`주소로 바인딩됩니다. 이 뜻은 운영체제에 해당 주소를 가진 리스너를 할당하였다는 의미입니다.

`defer`키워드로 함수자 종료되면 자동으로 실행되는 익명 함수를 만들어 생성된 리스너가 `우아(listener.Close())`하게 종료되게 합니다.


그럼 `net.Listener` 인터페이스는 어떻게 정의 되어 있을까요? 실제 `golang` 코드를 보겠습니다.
### Listener 인터페이스
```golang
//net.go
type Listener interface {
	Accept() (Conn, error)
	Close() error
	Addr() Addr
}
```
`net.go`에 정의된 리스터 인터페이스입니다. 3개의 메소드를 가지고 있습니다.
- Accept() - 클라이언트 연결되면 연결된 `Conn(커넥션)`을 반환합니다.
- Close() - TCP 연결을 종료합니다
- Addr() - 연결된 네트워크 타입과 주소가 저장된 `Addr` 인터페이스를 반환합니다.


### net.Listen 
```golang
//dial.go
func Listen(network, address string) (Listener, error) {
	var lc ListenConfig
	return lc.Listen(context.Background(), network, address)
}
```
`ListenConfig` 구조체를 선언하며 `ListenConfig`의 `Listen`메소드를 실행하여 입력한 `network`, `address`에 맞게 설정해줍니다.

`Listen` 메소드는
```golang
func (lc *ListenConfig) Listen(ctx context.Context, network, address string) (Listener, error) {
	addrs, err := DefaultResolver.resolveAddrList(ctx, "listen", network, address, nil)
	if err != nil {
		return nil, &OpError{Op: "listen", Net: network, Source: nil, Addr: nil, Err: err}
	}
	sl := &sysListener{
		ListenConfig: *lc,
		network:      network,
		address:      address,
	}
	var l Listener
	la := addrs.first(isIPv4)
	switch la := la.(type) {
	case *TCPAddr:
		l, err = sl.listenTCP(ctx, la)
	case *UnixAddr:
		l, err = sl.listenUnix(ctx, la)
	default:
		return nil, &OpError{Op: "listen", Net: sl.network, Source: nil, Addr: la, Err: &AddrError{Err: "unexpected address type", Addr: address}}
	}
	if err != nil {
		return nil, &OpError{Op: "listen", Net: sl.network, Source: nil, Addr: la, Err: err} // l is non-nil interface containing nil pointer
	}
	return l, nil
}
```
`dial.go`안에 정의되어 있으며, 내부적으로 `Listener` 인터페이스를 만들어 저희가 입력한 설정에 맞게 설정한 후 반환해줍니다.

리스너가 요청을 수락하는 과정을 알아봅시다.

```golang
for {
		//수신 연결, 실패 시 err 반환
		conn, err := listener.Accept()
		if err != nil {
			fmt.Println(err)
			return
		}
		go func(c net.Conn) {
			defer c.Close()
			fmt.Println("goroutine : ", c.LocalAddr())
		}(conn)
	}
```
for 루프를 돌며 서버에서 수신 요청을 대기합니다. 수신 요청이 오면 `listener.Accept`함수가 요청을 수락하고 `고루틴`에 해당 연결에 대한 일을 처리합니다. 위 코드는 연결이 있으면 주소만 출력하고 종료합니다.