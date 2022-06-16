---
layout: default
title: 핸들러 의존성 주입
parent: Go-network
---

# 핸들러 의존성 주입
`http.Handler` 인터페이스를 사용하면 요청(req), 응답(res) 객체에 접근할 수 있습니다. 
하지만 이외에도 데이터베이스, 로거, 캐시 등등 추가적으로 다른 객체의 접근이 필요한 경우 어떻게 할까요? 
저희가 여태까지 작성한 코드를 보면서 어떤 문제를 가지고 있는지 살펴보겠습니다.  

```golang
type User struct {
	FirstName string
	LastName  string
}
func HandlePostUser() func(http.ResponseWriter, *http.Request) {
	return func(w http.ResponseWriter, r *http.Request) {
		defer func(r io.ReadCloser) {
			_, _ = io.Copy(ioutil.Discard, r)
			_ = r.Close()
		}(r.Body)

		if r.Method != http.MethodPost {
			http.Error(w, "not Post", http.StatusMethodNotAllowed)
			return
		}
		var user User
		err := json.NewDecoder(r.Body).Decode(&user)
		if err != nil {
			fmt.Println(err)
			http.Error(w, "Decode Failed", http.StatusBadRequest)
			return
		}
		fmt.Println(user)
		w.WriteHeader(http.StatusAccepted)
	}
}
```

이 `핸들러`는 사용자의 이름을 `JSON`으로 받아 출력하는 핸들러입니다. 
그리고 이 핸들러를 등록하는 과정은  

```golang
http.Handle("/readjson", http.HandlerFunc(HandlePostUser()))
```

이런식으로 `Handler` 인터페이스로 랩핑하여 등록합니다.

`handlerPostUser`안에 익명함수는 `ServeHTTP` 메소드를 구현해야 하므로 `(http.ResponseWriter, *http.Request)` 
두 인자만 받습니다. 만약 해당 핸들러 내부에서 데이터베이스에 접근해야 한다면 어떻게 해야할까요? 두 가지 방법을 살펴보겠습니다.

### 클로저를 이용한 의존성 주입
```golang
dbHandler := func(db *sql.DB) http.Handler {
	return http.HandlerFunc(
		func(w http.ResponseWriter, r *http.Request) {
			pingTest := db.Ping()
			//데이터 베이스 객체 접근 가능
        }, 
	)
}
// 핸들러 등록
http.Handle("testDB", dbHandler(db))
```  

이러면 함수 내부 핸들러 스코프 내에서 `db` 객체에 접근할 수 있습니다. 
하지만 이런 프로젝트가 커짐에 따라 이런 객체가 많아지면 이런 방식은 번거롭게 될 수 있습니다. 
이 핸들러를 등록하는 스코프에서 해당 객체를 필요로 하니까요 그래서 이번엔 좀 더 나은 방법을 사용해봅시다.  


### 구조체 메서드를 이용한 의존성 주입
```golang
type Handlers struct {
	db  *sql.DB
	log *log.Logger
}

func (h *Handlers) Handler1() http.Handler {
	return http.HandlerFunc(
		func(w http.ResponseWriter, r *http.Request) {
			err := h.db.Ping()
		})
}
func (h *Handlers) Handler2() http.Handler {
	return http.HandlerFunc(
		func(w http.ResponseWriter, r *http.Request) {
			err := h.db.Ping()
		})
}

//구조체 초기화 후 핸들러 등록
useDB := &Handlers{
    db: db,
    log: log.New(...)
}
http.Handle("/han1", useDB.Hander1())
http.Handle("/han2", useDB.Hander2())
```  

가장 먼저 필요한 `객체(db, logger)`를 가지는 구조체를 정의하고 해당 객체로 초기화를 진행합니다.

그리고 해당 구조체의 `메소드`로 핸들러를 구현합니다. 그러면 핸들러는 구조체의 객체에 접근할 수 있습니다. 
핸들러에서 추가로 다른 객체, 리소스에 접근이 필요한 경우 구조체의 필드로 정의하고 메소드로 구현하여 접근하면 됩니다.
