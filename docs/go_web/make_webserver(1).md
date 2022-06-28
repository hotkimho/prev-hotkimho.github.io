---
layout: default
title: go 웹서버 만들기(1) 회원가입 및 로그인
parent: Go-web
---
# 시작하기 전
`golang`을 이용해 `회원가입` 및 `로그인`을 구현을 정리하며, 구현중 생기는
문제도 같이 기록합니다.

[https://github.com/hotkimho/go-Server](https://github.com/hotkimho/go-Server) 깃허브 링크입니다.


## 폴더 구조
![tree](/docs/go_web/images/dir_tree.png)

파일의 구조는 위 그림과 같습니다.
- auth 인증과 관련된 폴더
  - auth/users 유저 인증에 대한 함수들를 가지고 있습니다.
- config 설정파일 또는 전역 변수에 대한 설정을 가지고 있습니다.
- model 데이터의 구조체, 타입, 상수를 정의합니다.
  - model/user user에 대한 구조체, 타입, 상수를 가지고 있습니다.
- userAuthhandler 유저 인증에 대한 핸들러들을 가지고 있습니다.

## ERD
![erd](/docs/go_web/images/erd_1.png)  
# user 테이블
- uuid 유저를 식별하기 위한 아이디입니다.
- username 로그인할 때 사용하는 아이디입니다.
- password 로그인할 때 사용하는 비밀번호입니다.
- create_at 유저가 생성된 시간을 가지고 있습니다.

# user_session 테이블
- session_id 로그인된 유저를 식별하기 위한 id값입니다 클라이언트 쿠키에 같은값이 저장됩니다.
- user_id `user`테이블의 uuid입니다. `user`테이블에 대한 외래키값입니다.
- create_at session은 15분이 지나면 자동으로 만료됩니다. 그걸위한 생성 시간을 가지고 있습니다.


# 회원가입
```golang
package main

import (
  "encoding/json"
  "fmt"
  users "github.com/go-Server/auth/users"
  global "github.com/go-Server/config"
  "github.com/go-Server/model"
  "github.com/gofrs/uuid"
  "net/http"
  "time"
)

func SighUp() http.Handler {
	return http.HandlerFunc(
		func(w http.ResponseWriter, r *http.Request) {
			//Post 요청이 아니면 바로 종료
			if r.Method != http.MethodPost {
				global.Logger.Error("not Post Request")
				http.Error(w, "잘못된 요청입니다", http.StatusBadRequest)
				return
			}

			//json으로 데이터를 가져온다
			var user model.SignupRequestUser
			err := json.NewDecoder(r.Body).Decode(&user)
			if err != nil {
				global.Logger.Error(err.Error())
				http.Error(w, "잘못된 요청입니다", http.StatusBadRequest)
				return
			}
			
			//아이디, 비밀번호값 검증
			if users.ValidateRequestUser(user) == false {
				global.Logger.Error("Bad request to Signup")
				http.Error(w, "", http.StatusBadRequest)
				return
			}

			//데이터 베이스에 데이터 삽입
			err = users.InsertUser(user)
			if err != nil {
				global.Logger.Error(err.Error())
				http.Error(w, "사용자가 이미 존재합니다", http.StatusConflict)
				return
			}
			http.Error(w, "회원가입이 성공했습니다", http.StatusCreated)
			global.Logger.Info("생성한 유저 : " + user.Username)
		})
}
```

전체 코드를 먼저 올리고 간단하게 작은 블록씩 묶어서 설명을 하겠습니다.
주석이 있으므로 설명은 간단하게 적습니다.

### JSON 파싱
```go
package main

type SignupRequestUser struct {
Username string `validate:"required"`
Password string `validate:"required"`
}

var user model.SignupRequestUser
    err := json.NewDecoder(r.Body).Decode(&user)
	if err != nil {
		global.Logger.Error(err.Error())
		http.Error(w, "잘못된 요청입니다", http.StatusBadRequest)
		return
	}
```
가장 먼저 Body로 입력된 `username, password`값을 `JSON`의 구조로 가져옵니다.

`SignupRequestUser` 구조체의 `validate:"required"` 옵션은 값이 무조건 들어 있어야 하는 옵션이며
값이 없을 시 에러를 반환합니다.

### 입력값 검증
```go
if users.ValidateRequestUser(user) == false {
	global.Logger.Error("Bad request to Signup")
	http.Error(w, "아아디 또는 비밀번호를 다시 입력해주세요", http.StatusBadRequest)
	return
}

    //사용한 함수들
    //ValidateRequestUser 입력된 정보가 유효한지 판단하고 문제가 있으면 에러를 반환합니다.
func ValidateRequestUser(user model.SignupRequestUser) bool {
    validate := validator.New()

    err := validate.Struct(user)
    if err != nil {
    return false
    }
    return true
}
```
validator 패키지를 이용해 구조체에 옵션값을 지정할 수 있습니다. 위에서 `validate:"required"`을 구조체에 정의했으며
`valiadte.Struct` 함수를 이용해 입력된 값을 저장한 구조체에 옵션을 적용합니다.
만약 옵션에 위배되는 값이 있을경우 `validate.Struct` 함수는 에러를 반환합니다.

### 유저 저장
```go
//유저 정보를 데이터베이스에 넣습니다. 
err = users.InsertUser(user)
    if err != nil {
		global.Logger.Error(err.Error())
		http.Error(w, "사용자가 이미 존재합니다", http.StatusConflict)
		return
	}
	http.Error(w, "회원가입이 성공했습니다", http.StatusCreated)
    global.Logger.Info("생성한 유저 : " + user.Username)

    //사용한 함수들
    //InsertUser()
func InsertUser(user model.SignupRequestUser) error {
    //유저 중복검사, 중복된 아이디가 있으면 에러를 반환합니다.
    err := DuplicateCheckUser(user.Username)
    if err != nil {
		return err
    }
    
    //비밀번호 해싱, 비밀번호를 암화하합니다.
    hashedPassword, err := GeneratePassword(user.Password)
    if err != nil {
		return err
    }
    
    //uuid 생성, 유저를 구분하는 uuid를 생성합니다.
    newUuid, err := uuid.NewV4()
    if err != nil {
        return err
    }
    
	//uuid, 암호화한 비밀번호를 데이터베이스에 삽입합니다.
    _, err = global.Db.Exec(model.InsertUserQuery,
    newUuid.String(), user.Username, hashedPassword)
    if err != nil {
        return err
    }
        return nil
    }
```
입력값 검증을 끝마치면, 중복검사 -> 패스워드 암호화 -> uuid 생성 -> 데이터베이스 삽입 순서로 진행됩니다.
  
중복 검사를 하는 이유는 `동일한 사용자`가 있을 수 있으므로 검사하는것이며, `패스워드는 항상 암호화하여 보관`해야합니다.


# 로그인
