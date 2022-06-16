---
layout: default
title: 타임아웃 문제 해결하기
parent: Go-network
---

# 타임아웃
`golang`의 기본 HTTP 클라이언트와 `http.Get, Headd, Post` 함수에서 생성된 요청은 타임아웃이 되지 않습니다.
 타임아웃이 되지 않는건 심각한 문제입니다. 요청이 있으면 `성공, 실패` 둘중 하나를 무조건 응답해야 합니다. 
아무런 동작없이 무한정 블로킹 되는건 서비승에서 굉장히 큰 오류입니다.  

```golang
func BlockIndefinitely() {
	http.HandleFunc("/", func(writer http.ResponseWriter, r *http.Request) {
		fmt.Println("Request Get")
		select {}
	})
	_ = http.ListenAndServe("127.0.0.1:3000", nil)
	fmt.Println("요청이 끝났습니다")
}
```
이 코드는 서버를 생성하고 `/` 인덱스로 요청이 오면 `select`문에서 무한정 대기합니다. 즉 사용자는 계속 응답을 대기합니다.


# 타임아웃 처리하기(Deadline Context)
무한 로딩되는 문제를 해결하기 위해 요청을 보낼 때 `context`를 이용하면 됩니다. 
이전에 `deadline context`을 이용한 적이 있습니다. `임의의 시간`을 가지는 요청을 보내거나 `cancel` 함수로 
요청을 취소하면 계속 대기하는 타임아웃 문제를 해결할 수 있습니다.

```go
func BlockIndefinitelyTimeout() {
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	req, err := http.NewRequestWithContext(ctx, http.MethodGet, "http://localhost/", nil)
	if err != nil {
		fmt.Println(err)
	}
	res, err := http.DefaultClient.Do(req)
	if err != nil {
		if !errors.Is(err, context.DeadlineExceeded) {
			fmt.Println(err)
		}
		return
	}
	res.Body.Close()
}
```
`5`초의 `timeout`을 가지는 `context`를 생성합니다. 그리고 `http.NewRequestWithContext` 함수에 
context, GET, 주소, nil(io.Reader) 을 인자로 주어 요청을 생성합니다. 그리고 요청을 보냅니다.

이 함수를 실행 시 5초가 지나면 자동으로 연결이 종료됩니다.

