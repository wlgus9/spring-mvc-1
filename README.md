## 스프링 MVC

### 프로젝트 생성 시 주의점
https://start.spring.io/ -> Packaging : **Jar 선택**

Jar
* 항상 내장 서버(톰캣 등)를 사용
* `webapp` 경로는 사용하지 않음
* `/resources/static/` 위치에 `index.html` 파일을 두면 Welcome 페이지로 처리

War
* 내장 서버도 사용 가능하지만, 주로 외부 서버에 배포하는 목적으로 사용

---

## 로깅(Logging)
`SLF4J` : 로그 라이브러리는 `Logback(스프링부트 기본 제공)`, `Log4J`, `Log4J2` 등 많은 라이브러리들을 통합해서 인터페이스로 제공해 주는 라이브러리

쉽게 말해 `SLF4J`는 인터페이스이고 그 구현체로 `Logback` 같은 로그 라이브러리를 선택하면 된다.

### 로그 선언
```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

private Logger log = LoggerFactory.getLogger(getClass());

OR

private static final Logger log = LoggerFactory.getLogger(Xxx.class)
```
### lombok 사용 시
`@Slf4j`


### 로그 호출
`log.info("hello")`
`System.out.println("hello")`

### 로그 출력 포멧
* 시간, 로그 레벨, 프로세스 ID, 스레드명, 클래스명, 로그 메시지

### 로그 레벨
* `TRACE > DEBUG > INFO > WARN > ERROR`
* 일반적으로 개발 서버는 `debug`, 운영 서버는 `info` 출력

### 로그 레벨 설정
`application.properties`
```
# 전체 로그 레벨 설정(기본 info)
logging.level.root=info

# hello.springmvc 패키지와 그 하위 로그 레벨 설정
logging.level.hello.springmvc=debug
```

### 로그 사용 시 주의점 
ex) 로그 출력 레벨을 `info`로 설정했을 때

`log.debug("data = " + data)` : `"data="+data`가 실제 실행되면서 문자 더하기 연산이 발생

`log.debug("data={}", data)` : 파라미터를 넘기기 때문에 의미없는 연산이 발생하지 않음, 따라서 **이 방법이 올바른 로그 사용 방법이다.**

---

## 요청 매핑

### `@Controller` vs `@RestController`
* `@Controller` : 반환 값이 `String`이면 view 이름으로 인식된다. 그래서 **뷰를 찾고 뷰가 랜더링**된다.
* `@RestController` (=`@Controller` + `@ResponseBody`) : 반환 값을 **HTTP 메시지 바디에 바로 입력**한다. 따라서 실행 결과로 ok 메세지를 받을 수 있다.
