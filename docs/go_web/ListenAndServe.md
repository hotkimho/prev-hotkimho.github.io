---
layout: default
title: ListenAndServe 분석
parent: Go-web
---

# HTTP 연결 대기
서버는 항상 요청을 대기하고 요청이 오면 응답을 해주어야 합니다. 그래서 가장 먼저 수신을 대기하는 함수를 확인해봅시다.

```golang
package main

import (
	"net/http"
)

func main() {
	http.ListenAndServe("127.0.0.1:80", nil)
}
```
`ListenAndServe`함수는 인자로 2개의 인자를 받습니다.

1. 대기할 서버 주소:포트
2. `Handler` 인터페이스를 가지는 변수

1번의 경우 저희가 접속할 서버의 주소라는걸 알 수 있지만 두 번째 인자는 무엇이며 왜 코드에는 `nil`이 들어가 있는걸까요?

먼저 `Handler` 인터페이스를 보겠습니다
```golang
//server.go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```
`server.go`안에 정의되어 있고 핸들러 인터페이스는 `ServeHTTP` 메소드를 가지고 있습니다. 그리고 `ServeHTTP` 메소드는 요청을 받아 응답하는 역할을 해줍니다.

`Handler` 인터페이스를 알았으니  `ListenAndServe` 함수의 구현 부분을 보겠습니다.

```golang
func ListenAndServe(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
}
```
저희가 입력한 `주소와 핸들러`를 `Server` 구조체 초기값으로 넣어넣어줬습니다. 쭉쭉 파고드렁 `Server`의 구조체가 어떻게 정의 되어있는지 보겠습니다.

```golang
type Server struct {
	Addr string
	Handler Handler // handler to invoke, http.DefaultServeMux 
	TLSConfig *tls.Config
	ReadTimeout time.Duration
	ReadHeaderTimeout time.Duration
	WriteTimeout time.Duration
	IdleTimeout time.Duration
	MaxHeaderBytes int
	TLSNextProto map[string]func(*Server, *tls.Conn, Handler)
	ConnState func(net.Conn, ConnState)
	ErrorLog *log.Logger
	BaseContext func(net.Listener) context.Context
	ConnContext func(ctx context.Context, c net.Conn) context.Context
	inShutdown atomicBool // true when server is in shutdown
	disableKeepAlives int32     // accessed atomically.
	nextProtoOnce     sync.Once // guards setupHTTP2_* init
	nextProtoErr      error 
	mu         sync.Mutex
	listeners  map[*net.Listener]struct{}
	activeConn map[*conn]struct{}
	doneChan   chan struct{}
	onShutdown []func()
}
```
뭔가 엄청 많습니다. 아마 저희가 생성할 서버의 `옵션`값들을 가지고 있는 구조체 인거 같습니다. 이 구조체를 만들고 `server.ListenAndServe` 메소드를 실행했습니다. 이 함수에서 서버가 생성되는것 같습니다.

```golang
func (srv *Server) ListenAndServe() error {
	if srv.shuttingDown() {
		return ErrServerClosed
	}
	addr := srv.Addr
	if addr == "" {
		addr = ":http"
	}
	ln, err := net.Listen("tcp", addr)
	if err != nil {
		return err
	}
	return srv.Serve(ln)
}
```
`server` 구조체의 `ListenAndServe` 메소드의 구현입니다. 미숙해서 모든 코드의 내용을 완벽히 이해할 순 없지만 조금씩 찾아가 보겠습니다.

`ln, err := net.Listen("tcp", addr)`

여기서 `net.Listen` 함수를 실행했습니다!! 서버의 `리스너`를 생성하는 부분을 찾았습니다.  
(리스너에 대해 공부한 내용은 [리스너](https://hotkimho.github.io/docs/go_network/tcp/) 이 링크를 참조하시길 바랍니다.

간단히 설명하면 인자로 입력된 `addr(127.0.0.1:80)`의 주소로 수신 연결이 가능한 `tcp` 서버를 생성합니다 수신 가능한 서버를 생성했지만 요청에 대한 내용이 없습니다. 아마 `srv.Serve`함수안에 있을겁니다.

```golang
func (srv *Server) Serve(l net.Listener) error {
	if fn := testHookServerServe; fn != nil {
		fn(srv, l) // call hook with unwrapped listener
	}

	origListener := l
	l = &onceCloseListener{Listener: l}
	defer l.Close()

	if err := srv.setupHTTP2_Serve(); err != nil {
		return err
	}

	if !srv.trackListener(&l, true) {
		return ErrServerClosed
	}
	defer srv.trackListener(&l, false)

	baseCtx := context.Background()
	if srv.BaseContext != nil {
		baseCtx = srv.BaseContext(origListener)
		if baseCtx == nil {
			panic("BaseContext returned a nil context")
		}
	}

	var tempDelay time.Duration // how long to sleep on accept failure

	ctx := context.WithValue(baseCtx, ServerContextKey, srv)
	for {
		rw, err := l.Accept()
		if err != nil {
			select {
			case <-srv.getDoneChan():
				return ErrServerClosed
			default:
			}
			if ne, ok := err.(net.Error); ok && ne.Temporary() {
				if tempDelay == 0 {
					tempDelay = 5 * time.Millisecond
				} else {
					tempDelay *= 2
				}
				if max := 1 * time.Second; tempDelay > max {
					tempDelay = max
				}
				srv.logf("http: Accept error: %v; retrying in %v", err, tempDelay)
				time.Sleep(tempDelay)
				continue
			}
			return err
		}
		connCtx := ctx
		if cc := srv.ConnContext; cc != nil {
			connCtx = cc(connCtx, rw)
			if connCtx == nil {
				panic("ConnContext returned nil")
			}
		}
		tempDelay = 0
		c := srv.newConn(rw)
		c.setState(c.rwc, StateNew, runHooks) // before Serve can return
		go c.serve(connCtx)
	}
}
```
코드가 굉장히 많지만 연결을 하는 부분을 찾았습니다. 코드가 매우 길어 핵심부분만 가져오겠습니다

```golang
rw, err := l.Accept()
		if err != nil {
			select {
			case <-srv.getDoneChan():
				return ErrServerClosed
			default:
			}
			if ne, ok := err.(net.Error); ok && ne.Temporary() {
				if tempDelay == 0 {
					tempDelay = 5 * time.Millisecond
				} else {
					tempDelay *= 2
				}
				if max := 1 * time.Second; tempDelay > max {
					tempDelay = max
				}
				srv.logf("http: Accept error: %v; retrying in %v", err, tempDelay)
				time.Sleep(tempDelay)
				continue
			}
			return err
		}
```
`l` 객체는 저희가 `Listen` 함수로 생성한 리스너 객체입니다. 리스너 객체의 `Accept` 메소드는 연결 요청이 오면 __서버, 클라이언트가 서로 `3 way handshake`를 진행합니다. 연결이 성공적이면 `Conn` 연결 객체를 반환합니다.

위 코드는 연결이 `성공`하면 `rw`에 연결 객체가 반환되고 실패하면 `에러`를 반환하여 if문을 수행합니다.

# mux
깜박한 내용이 있습니다.

`http.ListenAndServe("127.0.0.1:80", nil)`

의 코드에서 `handler`부분을 `nil`을 사용했습니다. 이 의미는 이 핸들러를 `DefaultServeMux`로 사용하겠다는 의미입니다.

`mux`는 사용자가 URL로 요청을 할 때, 각각의 URL에 맞게 핸들러를 실행시켜주는 역할을 합니다. 다른 말로 `router`라고 하며 쉽게 말해 여러 개 입력 중 요청에 맞는 핸들러를 실행시켜줍니다.

`Handler Handler // handler to invoke, http.DefaultServeMux if nil`

아까 살펴본 `server` 구조체에 정의되어있고 `nil`로 들어올 시 `http.DefaultServeMux`를 사용합니다. 즉 기본 라우터를 사용하게 된다는 의미입니다. 

# 요약
1. `ListenAndServe` 함수는 생성할 주소, `mux(라우터)(nil입력 시 디폴트mux)`를 입력받아 대기하는 서버를 생성합니다.
2. 내부적으로 `net.Listen`함수를 실행하여 입력된 주소로 `tcp` 리스너를 생성합니다.
3. 리스너가 생성이 되면 `Accept`함수가 연결 요청을 응답하고 `3 way handshake`를 진행해 연결을 진행합니다.
4. 연결이 되면 요청한 값에 따라 맞는 핸들러를 실행하여 응답합니다.

함수의 이름 그대로 `Listen`을 진행하고 `Serve`하는 방식입니다.

# 후기
다른 언어를 공부할 땐 직접 해당언어의 코드를 살펴보는건 생각도 못했습니다. 하지만 golang은 `내부 코드`가 알기쉽게 되어있어서 이렇게 파볼 수 있는거 같습니다.
