---
layout: default
title: 데이터 송수신
parent: Go-network
---

# 들어가며
작성한 함수들은 모두 테스트용으로 만들었습니다. 그래서 함수에서 `error`를 반환하지 않고 출력만 했습니다.

# 고정된 버퍼로 데이터 읽기
```go
func ReadIntoBuffer() {
	//20개의 Byte크기의 슬라이르 생성
	payload := make([]byte, 20)
	_, err := rand.Read(payload) //랜덤한 값으로 채웁니다
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("payload:", payload)
	//리스너 생성(localhost:3000)
	listener, err := net.Listen("tcp", "127.0.0.1:3000")
	if err != nil {
		fmt.Println(err)
		return
	}
	//고루틴에서 리스너가 요쳥을 수락합니다.
	go func() {
		conn, err := listener.Accept()
		if err != nil {
			fmt.Println(err)
			return
		}
		defer conn.Close()
		fmt.Println("연결이 성공했습니다", listener.Addr().String())
		_, err = conn.Write(payload)
		if err != nil {
			fmt.Println(err)
			return
		}
		fmt.Println("write payload")
	}()

	conn, err := net.Dial("tcp", listener.Addr().String())
	if err != nil {
		fmt.Println(err)
		return
	}
	buf := make([]byte, 15)
	for {
		n, err := conn.Read(buf)
		if err != nil {
			if err != io.EOF {
				fmt.Println(err)
			}
			break
		}
		fmt.Println("읽은 값:", buf)
		fmt.Printf("read %d bytes\n", n)
	}
	conn.Close()
}
```
먼저 `20Byte` 크기의 랜덤한값을 가지는 `payload` 슬라이스를 만들었습니다.

고루틴으로 `net.Listen` 함수로 리스너(127.0.0.1:3000)을 생성했습니다. 그리고 리스너에 요청이 오면 
랜덤한 값을 가지는 `payload`를 `Write`합니다.  

`net.Dial` 함수를 사용해서 리스너로 연결을 시도합니다. 연결이 성공하면 `15Byte`크깅의 버퍼를 만들어 
데이터를 `Read`합니다. 서버의 데이터는 20Byte이고 클라이언틍의 버퍼는 15Byte입니다. 버퍼보다 데이터의 양이 많기 때문에 
for문으로 루프를 돌아 여러번 데이터를 Read합니다.  

![read_write1](/docs/go_network/images/read_write_1.png)  

결과를 보면 `20Byte`의 랜덤한 값을 출력합니다. 그리고 리스너에서 연결요청이 오면
 데이터를 Write합니다.

서버에서 데이터가 `Write`되면 클라이언트에서 데이터를 `Read`해서 출력합니다. 
데이터가 계속 `15Byte`가 고정되지만 두 번째 Read에서 앞의 5자리는 정상적으로 읽혔습니다.


