---
layout: default
title: 데이터베이스 연결(mysql)
parent: Nestjs_board

---

# __데이터베이스__
데이터베이스는 `mysql`(amazon RDS)과 `typeorm`을 사용합니다.

# __데이터베이스 연결__
## 패키지 설치
1. `mysql`, `typerom` 패키지를 설치합니다.

`npm install --save @nestjs/typeorm typeorm mysql2`  

2. 데이터베이스 정보를 `dotenv` 패키지를 사용해 관리하기 때문에 같이 설치해줍니다.

`npm i --save @nestjs/config`

## dotenv 사용
`dotenv`는 민감한 정보를 소스파일에 노출되지 않게 하는 패키지로, `.env`파일 안에 `키=값`의 형태로 데이터를 저장하여 사용합니다.  
`.env` 파일안에 데이터베이스 정보를 저장하고 `process.env.키` 방식으로 사용합니다.  
먼저 `nestjs`에서 `dotenv`를 사용하도록 설정해줍니다.
```typescript
//app.module
imports: [
    ConfigModule.forRoot({
      isGlobal: true,
    })
]
...
```
이렇게 설정하면 앞으로 `process.env`를 `전역변수`처럼 사용할 수 있습니다.  

## typeorm 연결
```typescript
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
    }),
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: process.env.MYSQLDB,
      port: parseInt(process.env.MYSQLPORT),
      username: process.env.MYSQLADMIN,
      password: process.env.MYSQLPASSWORD,
      database: 'board',
      entities: [],
      synchronize: true,
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```  
amazon RDS의 값을 `.env`안에 넣어서 사용합니다.  
`host` - RDS 엔드 포인트  
`port` - RDS PORT  
`username` - RDS ADMIN  
`password` - RDS ADMIN PASSWORD  
`entities` - 불러올 DB entity

![db image](/docs/nestjs_board/images/connect_DB_1.png)
연결이 잘 된걸 볼 수 있습니다.