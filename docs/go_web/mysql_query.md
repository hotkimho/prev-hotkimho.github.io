---
layout: default
title: mysql Query
parent: Go-web
---

# Mysql
## 패키지 설치
mysql 연결에 필요한 패키지들을 설치해야합니다.
```golang
go get github.com/go-sql-driver/mysql
```
실제로 mysql을 사용할 땐 golang에 기본 패키지인 `database/sql` 패키지를 사용할거지만 이 패키지도 설치를 해주어야 합니다.
그 이유는 `database/sql` 패키지에서 드라이버 패키지를 사용하기 때문입니다.

`import _ "github.com/go-sql-driver/mysql`

그래서 `_`로 alias를 주어 직접 사용하지 않게합니다.

## sql 연결
### sql.Oepn 함수
```go
sql.Oepn(driverName, dataSourceName)
```
`Oepn` 함수는 첫 번째 인수로 사용할 데이터 베이스 종류, 두 번째 인수로 `데이터 베이스 주소`를 입력받습니다.

```
root:password@(주소:포트)/data이름
root:password@(127.0.0.1:3306)/go_user
```

위가 dataSourceName의 양식이고 아래 코드가 제가 직접 테스트에 사용한 주소입니다. 물론 비밀번호는 실제 local 데이터베이스 비밀번호입니다.

## query 실행
### 테이블 구조
테스트에 사용한 `users` 테이블입니다.

<div markdown="1">

| id | username  | password | created_at |
|:-- |:----------|:-------- |:---------- |
| 1 | kimho | password | 2022-06-01 12:00:00

</div>

```go
CREATE TABLE users (
    id INT AUTO_INCREMENT,
    username TEXT NOT NULL,
    password TEXT NOT NULL,
    created_at DATETIME,
    PRIMARY KEY (id)
);
```
위 쿼리로 생성할 수 있습니다.

### Insert
`DML Operation`(DELETE, UPDATE, INSERT) 문을 사용할 때 `db.Exec` 함수를 사용합니다.
 사용한 코드의 예제를 보겠습니다.

```go
db, err := sql.Open("mysql", databaseURL)
	if err != nil {
		fmt.Println(err)
	}
	username := "kimho"
	password := "rlagh33"
	createAt := time.Now()

	result, err := db.Exec(`INSERT INTO users (username, password, created_at) VALUES (?, ?, ?)`, username, password, createAt)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(result.LastInsertId())
```
먼저 연결한 데이터 베이스 객체를 `db` 변수에 저장하고, 
각각 `username`, `password`, `createAt` 값을 정의했습니다.

`db.Exec` 메소드를 이용해 ``INSERT INTO users (username, password, created_at) VALUES (?, ?, ?)`의 쿼리를 입력했습니다.
`?` 값이 오면 `db.Exec`함수의 두 번쨰 인자부터 값을 넣을 수 있습니다. `Exec(query string, args ...any)` 두 번째 인자부터
 가변인자로 받기 때문에 몇 개를 넣어도 상관없습니다.

삽입이 정상적으로 실행되고 `LastInsertId` 메소드를 이용해 몇 번째 ID인지 출력했습니다.

![mysql_insert.png](/docs/go_web/images/mysql_insert.png)

코드를 실행하고 테이블을 조회하면 잘 들어가 있습니다.

### SELECT
