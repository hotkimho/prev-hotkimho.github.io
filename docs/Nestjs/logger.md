---
layout: default
title: Logger
parent: Nestjs_board

---
```
nestjs에 대한 한글 자료가 많이 없어 공식문서(영어), 인강등을 참고해서 다시볼 때 알아보기 쉽게 정리한 글입니다. 잘못된 내용이 있을 수 있습니다.
```

# Logger

로그는 서비스가 동작하는 과정을 남기고 추적하는 문구입니다.  
이슈가 발생할 경우, 잘못된 동작만으로 문제를 찾고 해결하는게 어렵기 때문에 로그를 남겨 추적할 경우 원이 파악에 도움이되고 개발하는데 들이는 시간을 줄일 수 있습니다.  

`nestjs`의 기본 내장 `Logger` 클래스는 `@nest/common` 패키지로 제공이 됩니다.  
로깅 옵션을 통해 다음과 같은 동작을 할 수 있습니다.  
1. 로깅 비활성화
2. 로그 레벨 지정 - (log, error, warn, debug, verbose)
3. 로거의 타임스탬프 재정의. ex) 날짜를 ISO8601 형식으로 변경
4. 기본 로거를 재정의(오버라이딩)
5. 기본 로거를 확장해서 커스텀 로거를 작성
6. 의존성 주입을 통해 손쉽게 로거를 주입하거나 테스트 모듈로 제공    
  


# 내장 로거
`@nest/common` 패키지 안에 `Logger`클래스가 정의되어 있습니다.  
이를 사용할려면 Logger 인스턴스를 생성하여 사용합니다.
```typescript
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class AppService {
  private readonly logger = new Logger('hoService');

  getHello(): string {
    this.logger.error('level: error 빨간색');
    this.logger.warn('level: warn 주황색');
    this.logger.log('level: log 초록색');
    this.logger.verbose('level: verbose 퍼렁색');
    this.logger.debug('level: debug 보라색');
    return 'Hello World!';
  }
}
```  
`private readonly logger = new Logger('hoService')`  
생성자의 입력값으로 로그에 `context`값을 주어 어디서 로그가 출력되는지 알 수 있습니다. `getHello()` 함수가 실행되면 로그는 아래와 같이 나옵니다.  
![image1](/docs/nestjs/images/logger_1.png)  

## 1. 로그 비활성화
```typescript
const app = await NestFactory.create(AppModule, {
  logger: false,
});
```
`main.ts` 파일안 app을 생성하는 과정에서 옵션 값으로 `logger: false`를 주면 내장 로거가 비활성화 됩니다.  

## 2. 로그 레벨 지정
직접 빌드, 배포를 해본적은 없지만... ㅠㅠ  실제 사용되는 서비스에선 debug 로그가 남지 않도록 해야 합니다. 이 문제는 개발, 프로덕션등 실행환경마다 `로그 레벨`을 지정하여 해결할 수 있습니다.  

```typescript
const app = await NestFactory.create(AppModule, {
    logger: process.env.NODE_ENV === 'production' 
    ? ['error', 'warn', 'log']
    : ['error', 'warn', 'log', 'verbose', 'debug']
});
```  
`main.ts`에서 app을 생성할 때 옵션값을 위 코드와 같이 설정할 수 있습니다.  
`.env`파일안에 `실행 환경`을 정의해두고 사용하면 쉽게 관리할 수 있습니다.  

만약 `log level`을 1개만 설정한다면 해당 로그보다 레벨이 `낮은`로그들도 모두 출력이 됩니다.  
예를 들어 `verbose`로그 레벨로 1개만 설정이 되면
`'error', 'warn', 'log', 'verbose'` 4개의 로그가 모두 출력됩니다.  
로그 레벨 값은 nestjs `is-log-level-enabled.util.ts` 파일안에 정의 되어 있습니다. 
```typescript
const LOG_LEVEL_VALUES: Record<LogLevel, number> = {
  debug: 0,
  verbose: 1,
  log: 2,
  warn: 3,
  error: 4,
};
```
