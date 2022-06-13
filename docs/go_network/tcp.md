---
layout: default
title: socket(tcp)
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
for 루프를 돌며 서버에서 수신 요청을 대기합니다. 수신 요청이 오면 `listener.Accept`메소드가 요청을 수락하고 `고루틴`에 해당 연결에 대한 일을 처리합니다. 위 코드는 연결이 있으면 주소만 출력하고 고루틴이 종료하고 다시 요청을 대기합니다.

중요한 점은 수신 요청이 오면 `listener.Accept`메소드가 클라이언트와 서버간 `TCP 핸드셰이크` 절차를 진행하고 `끝날 때까지 블로킹됩니다.` 에러(핸드셰이크가 실패하거나, 리스너가 닫힌 경우)가 있으면 에러값이 반환되고 성공되면 `net.Conn` 인스턴스의 주소가 반환됩니다.

# Dial을 사용하여 종료
```golang
func ListenerAndDial() {
	//포트가 없으면 랜덤으로!!
	listener, err := net.Listen("tc", "127.0.0.1:")
	if err != nil {
		fmt.Println(err)
		return
	}
	done := make(chan struct{})
	go func() {
		defer func() { done <- struct{}{} }()
		for {
			conn, err := listener.Accept()
			if err != nil {
				fmt.Println(err)
				return
			}
			go func(c net.Conn) {
				defer func() {
					c.Close()
					done <- struct{}{}
				}()

				buf := make([]byte, 1024)
				for {
					n, err := c.Read(buf)
					if err != nil {
						if err != io.EOF {
							fmt.Println(err)
						}
						return
					}
					fmt.Printf("received: %q\n", buf[:n])
				}
			}(conn)
		}
	}()
	conn, err := net.Dial("tcp", listener.Addr().String())
	if err != nil {
		fmt.Println(err)
	}
	conn.Close()
	<-done
	listener.Close()
	<-done
}
```


# 타임아웃
타임아웃이 발생할 때 어떻게 처리를 해야 할까요? `타임아웃 발생 시, 일시적으로 발생한 에러인지, 치명적인 에러인지 판단할 기준이 있어야 합니다.`

```golang
//builtin.go
type error interface {
	Error() string
}
```
하지만 저희가 자주 사용하는 `error` 인터페이스에는 그런 기준에 대한 내용이 포함되어 있지 않습니다.

다행이 `net` 패키지를 사용하면 에러에 대한 많은 정보를 얻을 수 있습니다. `net` 패키지안에 함수와 메서드으이 반환되는 에러들은 대부분 `net.Error` 인터페이스로 구현되어 있습니다

```golang
//net.go
type Error interface {
	error
	Timeout() bool // Is the error a timeout?

	// Deprecated: Temporary errors are not well-defined.
	// Most "temporary" errors are timeouts, and the few exceptions are surprising.
	// Do not use this method.
	Temporary() bool
}
```
`net.Error` 인터페이스에는 `Timeout`, `Temporary`메서드가 있고, 운영체제에서 리소스를 사용할수 없거나 함수 호출 차단, 연결시간이 초과된 경우 `Timeout` 메서드는 `true`를 반환합니다. 또한 Timeout 메서드가 true를 반환하거나 운영체제의 리소스 제한을 초과한 경우 `Temporary` 메서드가 `true`를 반환합니다. `net` 패키지의 함수와 메서드에서 `범용적인 에러 인터페이스(error)`를 반환하는 경우 `err.(net.Error)`와 같이 `타입 어션셜`을 사용하여야 합니다.

__Temporary 함수의 주석에 사용하지 말라고 되어 있습니다. 책의 을 공부하며 최대한 사용을 자제해보겠습니다__

# context를 취소해서 연결 중단시키기
`context`는 작업을 지시할 때 작업 가능 시간, 작업 취소 등 지시할 수 있는 기능을 말합니다. 고루틴으로 작업할 때 일정시간만 작업하거나 고루틴을 취소할 때 사용합니다. 취소는 `signal`을 보내서 취소를 합니다.

작업을 취소하기 위해서는 초기회 시에 `context` 객체와 같이 반환되는 `cancel` 함수를 사용합니다. `cancel` 함수를 사용하면 데드라인, 타임아웃이 끝나기 전에 임의로 고루틴을 종료할 수 있습니다.

```golang
func DialContextCancel() {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	sync := make(chan struct{})

	go func() {
		defer func() {
			fmt.Println("고루틴 종료")
			sync <- struct{}{}
		}()
		var d net.Dialer
		d.Control = func(_, _ string, _ syscall.RawConn) error {
			time.Sleep(time.Second * 1)
			return nil
		}
		fmt.Println("start DialContext")
		conn, err := d.DialContext(ctx, "tcp", "127.0.0.1:3000")
		if err != nil {
			fmt.Println(err)
			return
		}
		conn.Close()
		fmt.Println("connection did not time out")
	}()
	fmt.Println("start cancel")
	cancel()
	<-sync
	if ctx.Err() != context.Canceled {
		fmt.Printf("expected canceld context; actual: %q", ctx.Err())
	}
}
```
`context.WithCancel` 함수를 통해 `context(ctx)`와 `cancel`함수를 초기화 했습니다.


`net.Dialer` 구조체는 `Dial`, `DialContext` 메서드를 통해 `리스너`에 연결을 요청합니다. `Dialer.Control` 메서드는 `Dial`연결이 시작된 후 실행됩니다.

`conn, err := d.DialContext(ctx, "tcp", "127.0.0.1:3000")`
`ctx` context를 가지고 `tcp`, `127.0.0.1:3000`의 서버로 연결을 요청합니다. 연결이 성공하면 `conn`객체를 반환하고 실패하면 err를 반환합니다. 

해당 코드에서는 연결을 요청하는 `고루틴`을 실행하고 바로 `context`의 `cancel`함수를 호출합니다. 취소 함수가 호출되면 현재 `ctx` context를 가진 연결이 종료됩니다. 현재 `Dial.Control` 메서드에서 연결을 하고 `Sleep` 함수를 통해 대기합니다. 대기중 `cancel`함수가 호출됬으므로 바로 종료하며 에러를 반환합니다.

![contextCancel](/docs/go_network/images/contextCancelResult.png)

콘솔 로그를 보면 `start cancel`이 메인루틴, `start DialContext`이 고루틴에서 출력되고 바로 해당 고루틴은 종료됩니다.

# 다중 다이얼러 연결 취소하기!
다중 다이얼러 취소를 왜 해야할까요?? 여러 개의 서버가 리소스를 요청할 때 각각 데이터를 받는게 아니라 한 개의 서버만 데이터를 받아도 되는 경우, 각각의 서버는 리소스 요청을 비동기적으로 하고 하나의 서버가 데이터를 받으면 다른 요청은 필요가 없으니 연결 시도를 중단해야합니다. 이런 경우 각 서버에 동일한 `context`를 각 `dialer(연결 요청)`에 전달합니다. 하나의 서버가 데이터를 받으면 나머지 요청들은 `context`를 취소하여 연결을 취소할 수 있습니다.

```golang
func DialContextCancelFanOut() {
	ctx, cancel := context.WithDeadline(
		context.Background(),
		time.Now().Add(10*time.Second),
	)

	listener, err := net.Listen("tcp", "127.0.0.1:3000")
	if err != nil {
		fmt.Println("net.Listen Error")
		return
	}
	defer listener.Close()

	go func() {
		fmt.Println("connect listener")
		conn, err := listener.Accept()
		if err == nil {
			conn.Close()
			fmt.Println("close listener")
		}
	}()
	dial := func(ctx context.Context, address string, response chan int,
		id int, wg *sync.WaitGroup) {
		defer func() {
			wg.Done()
			fmt.Println(id, " end dial")
		}()
		fmt.Println(id, " start dial")
		var d net.Dialer
		c, err := d.DialContext(ctx, "tcp", address)
		if err != nil {
			return
		}
		c.Close()
		select {
		case <-ctx.Done():
		case response <- id:
		}
	}
	res := make(chan int)
	var wg sync.WaitGroup
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go dial(ctx, listener.Addr().String(), res, i+1, &wg)
	}
	response := <-res
	cancel()
	wg.Wait()
	close(res)

	if ctx.Err() != context.Canceled {
		fmt.Printf("expected canceld context; actual:%s", ctx.Err())
	}
	fmt.Printf("Dialer %d retrieved thr resource\n", response)
}
```
간단히 요약하면 `동일한 콘텍스트를 여러 개의 다이얼러에게 전달하고, 첫 응답을 받으면 컨텍스트를 취소하여 다른 다이얼러들의 연결을 중단합니다.`

```golang
go func() {
	conn, err := listener.Accept()
	if err == nil {
		conn.Close()
	}
}()
```
`리스너`는 요청이 수락되는 즉시 연결을 종료합니다.

```golang
dial := func(ctx context.Context, address string, response chan int, id int, wg *sync.WaitGroup) {
		defer func() {
			wg.Done()
		}()
		var d net.Dialer
		c, err := d.DialContext(ctx, "tcp", address)
		if err != nil {
			return
		}
		c.Close()
		select {
		case <-ctx.Done():
		case response <- id:
		}
	}
	res := make(chan int)
	var wg sync.WaitGroup
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go dial(ctx, listener.Addr().String(), res, i+1, &wg)
	}
	response := <-res
	cancel()
	wg.Wait()
	close(res)
```
이 익명함수는 입력된 `address`주소로 연결을 시도합니다. 연결이 성공하면 연결을 종료하고 `response` 채널에 ID를 전송합니다. `response` 채널을 받음으로써 데이터를 받았다고 인식하고 `cancel`함수를 실행하여 `context`를 취소합니다. 다른 다이얼러들을 취소할 때 `wg.Wait`함수로 인해 고루틴이 종료될 때까지 블로킹 됩니다.
