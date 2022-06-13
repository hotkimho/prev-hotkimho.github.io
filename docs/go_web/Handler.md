---
layout: default
title: Handler 함수 분석
parent: Go-web
---

# Handler
이전 `ListenAndServe` 분석에서 `Handler`는 `ServeHTTP` 메소드를 가지는 인터페이스인걸 확인 했습니다. 그리고 `ListenAndServe`의 두 번째 인자로 `Handler` 인터페이스를 받고, nil의 값이 오면 `DefaultMux`로 초기화 된다고 했습니다.

그럼 직접 `ServeHTTP` 인터페이스를 구현해서 인자로 주면 어떻게 동작할까요?
```golang
package main

import (
	"fmt"
	"net/http"
)

type testStruct struct{}

func (t testStruct) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "test~~~~")
}

func main() {
	var test testStruct
	http.ListenAndServe("127.0.0.1:80", test)
}
```

`testStruct` 구조체는 `ServeHTTP` 메소드를 가지고 있으니 `Handler` 인터페이스의 인자로 값을 줄 수 있습니다. 그리고 실제로 동작을 하고 `localhost:80`으로 접속 하면 아래와 같은 결과가 나옵니다.

![serveHTTP](/docs/go_web//images/serveHTTP.png)

`Handler` 인터페이스를 구현하면 이렇게 사용자가 직접 정의하며 핸들러의값을 줄 수 있습니다.


# HandleFunc
위의 방식은 `Handler` 인터페이스만 구현되면 동작이 되는걸 보여드린겁니다. 실제론 사용자 요청에 따른 각기 다른 핸들러를 실행시켜야 합니다.

```golang
package main

import (
	"fmt"
	"net/http"
)

func helloFunc(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "hello Function\n")
}

func testFunc(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "test Function\n")
}

func main() {

	http.HandleFunc("/hello", helloFunc)
	http.Handle("/test", http.HandlerFunc(testFunc))
	http.ListenAndServe("127.0.0.1:80", nil)
}
```
먼저 `hello Function` 값을 출력하는 `helloFunc` 함수와 `test Function` 값을 출력하는 `testFunc`함수를 정의했습니다.

먼저

`http.ListenAndServe("127.0.0.1:80", nil)`

두 번째 인자가 `nil` 이기 때문에 등록하는 핸들러들은 모두 `DefaultServeMux` 기본 `mux(라우터)`에 등록됩니다.

```golang
http.HandleFunc("/hello", helloFunc)
http.Handle("/test", http.HandlerFunc(testFunc))
```
둘 모두 각각의 함수를 `DefaultMux`에 등록하는 과정입니다.

`127.0.0.1/hello` 입력시 `helloFunc` 함수가 실행되고 `127.0.0.1/test` 입력시 `testFunc` 함수가 실행됩니다.

근데 왜 서로 등록하는 방식이 다를까요?

`HandleFunc, Handle` 함수 모두 첫 번째 인자로 `pattern(요청)`을 받지만 두 번째 인자가 다릅니다.

### HandlerFunc
```golang
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}
```

두 번째 인자로 `func(ResponseWriter, *Request)` 함수를 받습니다. 이 의미는 (ResponseWrite, *Request) 두 인자를 받는 함수를 두 번째 인자로 받는다는 의미입니다.

```golang
//첫 번째 방법
func helloFunc(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "hello Function\n")
}
http.HandleFunc("/hello", helloFunc)

//두 번째 방법(같은 동작입니다)
http.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "hello Function\n")
	})
```

저희가 정의한 `helloFunc` 함수는 `ResponseWriter, *Request` 두 인자를 받는 함수라 `HandleFunc` 함수의 두 번째 인자로 넘겨줄 수 있습니다.

`DefaultServeMux.HandleFunc(pattern, handler)`

그리고 입력된 함수를 `DefaultServeMux` 즉 기본 `mux(라우터)`에 등록합니다.

### Handle
```golang
func Handle(pattern string, handler Handler) {
	DefaultServeMux.Handle(pattern, handler)
	}
```

`Handle` 함수는 두 번째 인자로 `Handler` 인터페이스를 받아 `DefaultServeMux`에 함수를 등록합니다.

`http.Handle("/test", http.HandlerFunc(testFunc))`

그래서 핸들러에 등록할 때 `http.HandlerFunc`타입으로 변환하여 넘겨줍니다.

`http.HadnlerFunc` 타입은 `server.go` 파일안에 아래와 같이 정의되어 있습니다.

`type HandlerFunc func(ResponseWriter, *Request)`

그래서 매개변수에 넘겨줄 때 `http.HandlerFunc(testFunc)`로 넘겨줍니다.


# mux
`ListenAndServe` 함수의 두 번째 인자로 `nil`을 주면 디폴트 mux로 핸들러가 등록됩니다. 하지만 저희가 직접 `mux`를 생성할 수 있습니다.

`mux := http.NewServeMux()`

이렇게 `http.newServeMux` 함수로 생성합니다. 이 함수는 어떻게 구현되어 있을까요?

```golang
func NewServeMux() *ServeMux {
	return new(ServeMux)
}
```
`ServeMux` 인스턴스를 생성하여 바로 반환합니다.

```golang
type ServeMux struct {
	mu    sync.RWMutex
	m     map[string]muxEntry
	es    []muxEntry // slice of entries sorted from longest to shortest.
	hosts bool       // whether any patterns contain hostnames
}
```
`ServeMux` 구조체는 이렇게 정의 되어 있습니다. 뭔가 어떤게 어떤 역할을 하는지 아직은 모르겠네요


```golang
package main

import (
	"fmt"
	"net/http"
)

func testFunc(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "test Function\n")
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "hello Function\n")
	})
	mux.Handle("/test", http.HandlerFunc(testFunc))
	http.ListenAndServe("127.0.0.1:80", mux)
}
```
`DefaultServeMux`를 사용하지 않고 직접 `mux`를 생성하여 핸들러를 등록한 코드입니다. 디폴트 mux를 사용한 코드와 비교가 되시나요???

```golang
mux := http.NewServeMux()
```
직접 `mux`를 생성하고 핸들러를 등록했습니다. 위의 예제랑 동일한 동작을 합니다.


