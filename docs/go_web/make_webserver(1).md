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

## 수정 사항
로그인 세션을 `Database`에 저장하는 식으로 구현하려고 했습니다.

하지만 계속 데이터베이스를 조회하는 성능 문제가 있어 `Redis`를 사용하는 방식으로 수정했습니다.

기존에 있는 내용은 다 지우고 Redis를 사용하는 방식으로 다시 올릴예정입니다.