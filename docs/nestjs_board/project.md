---
layout: default
title: 프로젝트 명세서
parent: Nestjs_board

---

## 1. 개요
- 프로젝트 : 게시판
- 개발 인원 : 혼자
- 개발 기간 : 2022.04.29 ~ 2022.5.13 
- 개발 목적 : Nestjs 공부겸 2주동안 게시판을 구현해본다.
- 주요 기능 : CRUD 기능을 제공하는 게시판, 사용자 회원가입, 로그인(카카오), 댓글
- 개발 환경 : Nestjs 8.0, React, 추가될 시 명시
- 데이터베이스 : mysql(amazon RDS)
- 개발 방법 : 백엔드를 배우는게 목적이기에 백엔드 완료 후 간단히 프론트를 작업한다. 따로 하는 이유는 서로 독립된 개발로 보고 실제에서도 백과 프론트 작업하는 인원이 다르다.
- 
## 2. 요구사항
### 1. 회원가입 페이지
  - ID(username), 닉네임, 비밀번호, 이메일을 입력 받음
  - 유효성 검사는 프론트, 백 둘다 검증해야 된다고 생각함(실수가 있을 수 있고 실수가 생기면 치명적인 문제) 하지만 이 프로젝트는 백엔드 위주이기에 백엔드에서 검사를 진행
  - ID 유효성 검사
    - 2 ~ 10글자
    - 알파벳 대소문자(a-z, A-Z), 숫자(0-9)로 구성
    - 공백, 빈칸을 허용하지 않음
    - 데이터베이스에서 중복 검사를 진행
  - 닉네임 유효성 검사
    - 2 ~ 10글자
    - 알파벳 대소문자(a-z, A-Z), 숫자(0-9), 한글(1글자당 2)로 구성
    - 공백, 빈칸을 허용하지 않음
    - 데이터베이스에서 중복 검사를 진행
  - 비밀번호 유효성 검사
    - 최소 4글자 이상
    - 특수문자, 알파벳(a-z, A-Z), 숫자(0-9)가 각각 1개 이상 들어가야 함
    - 암호화되어 데이터베이스에 저장
  - 이메일 유효성 검사
    - 이메일 형식 검사
    - 데이터베이스에서 중복 검사를 진행
  
### 2. 로그인 페이지
- ID(username), password 입력을 받아 로그인 진행
- ID, password중 하나라도 일치 하지 않을 시, "아이디 또는 비밀번호를 다시 입력해주세요" 문구 보여줌
- 공백, ID, password의 유효성 검증이 하나라도 통과가 안되면 경고 문구 보여줌
- 카카오 로그인
  - 카카오 로그인 동작을 확인 후 서술!!
  
### 3. 마이 페이지
- 로그인이 검증되어야 접속 가능
- 닉네임, 비밀번호 수정 가능
- 공백, 유효성 검사에 어긋나는 입력 시 경고 문구 보여줌
  
### 4. 게시판 페이지
- 게시글 작성, 수정, 삭제 기능
- 게시글 작성
  - 제목, 내용, 첨부파일로 구성됨
  - 제목은 공백이 올 수 없음
  - 본인의 게시글만 수정, 삭제 버튼이 노출됨
- 댓글
  - 게시글 하단에 댓글 입력창이 노출됨
  - 댓글은 ID, 생성된 날짜, 내용으로 구성됨
  - 댓글은 공백이 올 수 없음
  - 대댓글은...... 좀더 고민!
- 게시글이 삭제되면 게시글에 작성된 댓글이 삭제되엉야 함(중요)

## 3. DB 관계도
![db](/docs/nestjs_board/images/board_ERD.png)

__Post, Comemnt 관계도는 1:N의 관계__
__Comment 테이블 `commentId` 추가__
### User
<div markdown="1">

| 컬럼명 | 자료형 | 옵션 | 설명 |
|:-----|:------|:----|:----|
| id  | BIGINT | pk  | 고유 id |
| username | VARCHAR | unique, not null| 아이디 | 
| password  | VARCHAR | not null| 비밀번호 |
| nickname  | VARCHAR | unique, not null | 닉네임 |
| email  | VARCHAR | unique, not null | 이메일 |
| created_date  | VARCHAR | not null | 생성날자 |

</div>

### Post
<div markdown="1">

| 컬럼명 | 자료형 | 옵션 | 설명 |
|:-----|:------|:----|:----|
| postId  | BIGINT | pk  | 고유 id |
| userId | BIGINT | fk, not null| user id | 
| title  | VARCHAR | not null| 제목 |
| content  | TEXT | not null | 내용 |
| writer  | VARCHAR | not null | 작성자 |
| created_date  | VARCHAR | not null | 생성날자 |
| modified_date  | VARCHAR | not null | 수정날자 |
| view  | BIGINT | not null | 조회수 |

</div>

### Comment
<div markdown="1">

| 컬럼명 | 자료형 | 옵션 | 설명 |
|:-----|:------|:----|:----|
| commentId  | BIGINT | pk  | 고유 id |
| userId  | BIGINT | fk, not null  | user id |
| postId | BIGINT | fk, not null | 게시글 id | 
| title | VARCHAR | not null | 제목 | 
| writer  | VARCHAR | not null | 작성자 |
| content  | TEXT | not null | 내용 |
| created_date  | VARCHAR | not null | 생성날자 |
| modified_date  | VARCHAR | not null | 수정날자 |

</div>

```
API문서는 추가되면 내용을 추가하고, 스웨거를 사용해 이미지도 첨부
```

### 사용자 API
<div markdown="1">

| 이름 | Method | URL | 설명 |
|:-----|:------|:----|:----|
| 로그인 | GET | /auth/login  | 게시글 전체 목록을 가져온다 |
| 로그아웃  | GET | /auth/logout  | 특정 게시글을 가져온다 |
| 회원가입 | POST | /auth/SignUp |  | 특정 페이지에서 게시글 목록을 가져온다. 
| 회원가입 중복 검사| GET | /auth/SignUp/:중복체크할 내용 | 제목 | 입력한 값에 맞는 게시글을 가져온다.

</div>

### 게시글 API
<div markdown="1">

| 이름 | Method | URL | 설명 |
|:-----|:------|:----|:----|
| 게시글 전체 조회  | GET | /post  | 게시글 전체 목록을 가져온다 |
| 특정 게시글 조회  | GET | /post/:id  | 특정 게시글을 가져온다 |
| 게시글 페이지 조회 | GET | 모르겠음 | 게시글 id | 특정 페이지에서 게시글 목록을 가져온다. 
| 게시글 검색| VARCHAR | 모르겠음 | 제목 | 입력한 값에 맞는 게시글을 가져온다.

</div>

### 댓글 API
<div markdown="1">

| 이름 | Method | URL | 설명 |
|:-----|:------|:----|:----|
| 게시글에 등록된 댓글 조회  | GET | /post/:id/comment  | 게시글에 등로된 댓글 목록을 가져온다 |
| 댓글 등록  | POST | /post/:id/comment  | 게시글에 댓글을 등록한다 |
| 댓글 수정 | PUT | /post/:id/comment/:id | 게시글 id | 게시글에 등록된 댓글의 내용을 수정한다
| 댓글 삭제| DELETE | /post/:id/comment/:id | 제목 | 게시글에 등록된 댓글을 삭제한다.

</div>