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

## `@Controller` vs `@RestController`
* `@Controller` : 반환 값이 `String`이면 view 이름으로 인식된다. 그래서 **뷰를 찾고 뷰가 랜더링**된다.
* `@RestController` (=`@Controller` + `@ResponseBody`) : 반환 값을 **HTTP 메시지 바디에 바로 입력**한다. 따라서 실행 결과로 ok 메세지를 받을 수 있다.

---

## HTTP 요청

### 기본, 헤더 조회

```java
@RequestMapping("/headers")
public String headers(HttpServletRequest request,
                        HttpServletResponse response,
                        HttpMethod httpMethod,
                        Locale locale,
                        @RequestHeader MultiValueMap<String, String>
                        headerMap,
                        @RequestHeader("host") String host,
                        @CookieValue(value = "myCookie", required = false) String cookie
) {
    log.info("request={}", request);
    log.info("response={}", response);
    log.info("httpMethod={}", httpMethod);
    log.info("locale={}", locale);
    log.info("headerMap={}", headerMap);
    log.info("header host={}", host);
    log.info("myCookie={}", cookie);
    
    return "ok";
}
```

* `HttpMethod` : HTTP 메서드 조회
  * `org.springframework.http.HttpMethod`

* `Locale` : Locale 정보 조회

* `@RequestHeader MultiValueMap<String, String> headerMap` : 모든 HTTP 헤더를 MultiValueMap 형식으로 조회
  * `MultiValueMap` : 하나의 키에 여러 값을 받을 수 있는 Map
  * HTTP header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용
  * **keyA=value1&keyA=value2**

```
MultiValueMap<String, String> map = new LinkedMultiValueMap();
map.add("keyA", "value1");
map.add("keyA", "value2");

//[value1,value2]
List<String> values = map.get("keyA");
```

* `@RequestHeader("host") String host` : 특정 HTTP 헤더 조회
  * 속성
    * 필수 값 여부: `required`
    * 기본 값 속성: `defaultValue`

* `@CookieValue(value = "myCookie", required = false) String cookie` : 특정 쿠키 조회
  * 속성
    * 필수 값 여부: `required`
    * 기본 값: `defaultValue`
  
## HTTP 요청 파라미터 - `@RequestParam`

### `@RequestParam`
파라미터 이름으로 바인딩
```
// ?username=kim
@RequestParam("username") String name
```

HTTP 파라미터 이름이 변수 이름과 같으면 `@RequestParam(name="xx")` 생략 가능
```
// ?username=kim
@RequestParam String username
```

`String` , `int` , `Integer` 등의 단순 타입이면 `@RequestParam` 생략 가능
* `@RequestParam` 애노테이션을 생략하면 스프링 MVC는 내부에서 `required=false`를 적용한다.
```
// ?username=kim
String username
```

> 파라미터 필수 여부
> 
> `@RequestParam(required = true)` : 기본값, 파라미터가 없으면 400 오류가 발생한다.
> 
> `@RequestParam(required = false)` : 없어도 오류나지 않는다.

### 주의!
```java
@ResponseBody
@RequestMapping("/request-param-required")
public String requestParamRequired(
                          @RequestParam(required = true) String username,
                          @RequestParam(required = false) Integer age
) {
    log.info("username={}, age={}", username, age);
    return "ok";
}
```

`?username=kim`으로 요청이 들어왔을 때
* age는 null이 된다.
* 하지만 `int` 타입은 null을 담을 수 없다.
* 그래서 `Integer`로 변경하거나 `defaultValue`를 사용해야 한다.

`?username=&age=20`으로 요청이 들어왔을 때
* username이 빈 문자열로 들어오기 때문에 오류가 발생하지 않는다.
* null과 ""은 다르다!

> `defaultValue`
> 
> @RequestParam(required = true, defaultValue = "guest")
> 
> `defaultValue`를 사용하면 이미 기본값이 설정되어 있기 때문에 `required`는 의미가 없다.
> 
> 빈 문자에도 기본값 적용이 가능하다. (= 빈 문자열 처리 가능!) 

파라미터를 Map, MultiValueMap으로 조회할 수 있다.

`@RequestParam Map` -> `Map(key=value)`

`@RequestParam MultiValueMap` -> `MultiValueMap(key=[value1, value2, ...]`

ex) (key=userIds, value=[id1, id2])

파라미터의 값이 1개가 확실하다면 `Map`을 사용해도 되지만, 그렇지 않다면 `MultiValueMap`을 사용하자.

## HTTP 요청 파라미터 - `@ModelAttribute`
요청 파라미터를 받아서 필요한 객체를 만들고 그 객체에 값을 넣어주는 과정을 자동화해 준다.

```java
// 요청 파라미터를 바인딩 할 객체
@Data
public class HelloData {
  private String username;
  private int age;
}
```
> `@Data` : `@Getter` , `@Setter` , `@ToString` , `@EqualsAndHashCode` , `@RequiredArgsConstructor` 자동 적용

```
/* RequestParam 사용 시 */

// @RequestParam String username;
// @RequestParam int age;

HelloData data = new HelloData();
data.setUsername(username);
data.setAge(age);
```

```
/* ModelAttribute 사용 시 */

@ModelAttribute HelloData helloData
```

위처럼 바인딩 할 객체를 넣어주기만 하면 자동으로 바인딩이 된다!

동작 순서는 다음과 같다.
1. `HelloData` 객체를 생성한다.
2. 요청 파라미터의 이름으로 `HelloData` 객체의 프로퍼티를 찾는다.
3. 그리고 해당 프로퍼티의 setter를 호출해서 파라미터의 값을 입력(바인딩)한다.

ex) 파라미터 이름이 `username` 이면 `setUsername()` 메서드를 찾아서 호출하고 값을 입력한다.

> 프로퍼티
> 
> 객체에 `getUsername()` , `setUsername()` 메서드가 있으면, 이 객체는 `username`이라는 프로퍼티를 가지고 있다.
> 
> `username` 프로퍼티의 값을 변경하면 `setUsername()` 호출, 조회하면 `getUsername()`호출

## HTTP 요청 메세지 - 단순 텍스트
요청 파라미터와 다르게, HTTP 메시지 바디를 통해 데이터가 직접 넘어오는 경우는 `@RequestParam`, `@ModelAttribute`를 사용할 수 없다. (물론 HTML Form 형식으로 전달되는 경우는 요청 파라미터로 인정된다.)

### `@RequestBody`
`@RequestBody`를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다.

> 주의! **@RequestBody는 생략 불가능**
> 
> 생략하면  `@ModelAttribute`가 적용됨

참고로 헤더 정보가 필요하다면 `HttpEntity` 를 사용하거나 `@RequestHeader` 를 사용하면 된다.

## `@ResponseBody`
`@ResponseBody`를 사용하면 응답 결과를 HTTP 메시지 바디에 직접 담아서 전달할 수 있다.

## HTTP 요청 메세지 - JSON
`HttpEntity`, `@RequestBody`를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해 준다.

> 주의! HTTP 요청시에 content-type이 application/json으로 되어있어야 한다.
> 
> 그래야 JSON을 처리할 수 있는 HTTP 메시지 컨버터가 실행된다.

## HTTP 응답

### 정적 리소스
웹 브라우저에 정적인 HTML, css, js 제공

디렉토리
* `/static`
* `/public`
* `/resources`
* `/META-INF/resources`

### 뷰 템플릿
웹 브라우저에 동적인 HTML 제공

기본 경로 : `src/main/resources/templates`

## HTTP 메시지 컨버터
스프링 MVC는 다음과 같은 경우에 HTTP 메시지 컨버터를 적용한다.

HTTP 요청 : `@RequestBody`, `HttpEntity` or `RequestEntity`

HTTP 응답 : `@ResponseBody`, `HttpEntity` or `ResponseEntity`

### `HttpMessageConverter`
`canRead()`/`canWrite()` : 메시지 컨버터가 해당 클래스, 미디어타입을 지원하는지 체크

`read()`/`write()` : 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능

스프링 부트는 다양한 메시지 컨버터를 제공하는데, 대상 클래스 타입과 미디어 타입 둘을 체크해서 사용여부를 결정한다. 만약 만족하지 않으면 다음 메시지 컨버터로 우선순위가 넘어간다.

몇 가지 주요한 메세지 컨버터는 다음과 같다.

`ByteArrayHttpMessageConverter` : `byte[]` 데이터를 처리한다.
* 클래스 타입: `byte[]`
* 미디어타입: `*/*`
* 예시
  * 요청 : `@RequestBody byte[] data`
  * 응답 : `@ResponseBody return byte[]` 쓰기 미디어타입 `application/octet-stream`

`StringHttpMessageConverter` : `String` 문자로 데이터를 처리한다.
* 클래스 타입: `String`
* 미디어타입: `*/*`
* 예시
  * 요청 : `@RequestBody String data`
  * 응답 : `@ResponseBody return "ok"` 쓰기 미디어타입 `text/plain`

`MappingJackson2HttpMessageConverter` : application/json
* 클래스 타입: 객체 또는 `HashMap`
* 미디어타입 `application/json` 관련
  * 예시
    * 요청 : `@RequestBody HelloData data`
    * 응답 : `@ResponseBody return helloData` 쓰기 미디어타입 `application/json` 관련

### 동작 과정 - 요청
1. HTTP 요청이 오고 컨트롤러에서 `@RequestBody`, `HttpEntity` 파라미터를 사용한다.
2. 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 `canRead()`를 호출한다.
   * 대상 클래스 타입을 지원하는가?
     * ex) `@RequestBody`의 대상 클래스 (`byte[]` , `String` , `HelloData` )
   * HTTP 요청의 Content-Type 미디어 타입을 지원하는가?
     * ex) `text/plain` , `application/json` , `*/*`
3. `canRead()` 조건을 만족하면 `read()`를 호출해서 객체 생성하고 반환한다.

### 동작 과정 - 응답
1. 컨트롤러에서 `@ResponseBody`, `HttpEntity`로 값이 반환된다.
2. 메시지 컨버터가 메시지를 쓸 수 있는지 확인하기 위해 `canWrite()`를 호출한다.
   * 대상 클래스 타입을 지원하는가?
     * ex) return의 대상 클래스 (`byte[]` , `String` , `HelloData` )
   * HTTP 요청의 Accept 미디어 타입을 지원하는가? (더 정확히는 `@RequestMapping`의 `produces`)
     * ex) `text/plain`, `application/json`, `*/*`
3. `canWrite()` 조건을 만족하면 `write()`를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성한다.

### 메세지 컨버터의 위치
핸들러 어댑터인 `RequestMappingHandlerAdapter` (요청 매핑 헨들러 어뎁터)에 있다.
![img.png](src/main/resources/static/img.png)

### `HandlerMethodArgumentResolver`
줄여서 `ArgumentResolver`

어노테이션 기반 컨트롤러를 처리하는 `RequestMappingHandlerAdapter`는 `ArgumentResolver`를 호출해서 컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)을 생성한다.

그리고 이렇게 파리미터의 값이 모두 준비되면 컨트롤러를 호출하면서 값을 넘겨준다.

> `ArgumentResolver` 덕분에 우리는 어노테이션 기반 컨트롤러에서 다양한 파라미터를 사용할 수 있는 것이다.

동작 방식
1. `ArgumentResolver`의 `supportsParameter()` 호출
2. 해당 파라미터를 지원하는지 체크
3. 지원하면 `resolveArgument()`를 호출해서 실제 객체를 생성
4. 컨트롤러 호출 시 생성된 객체 반환

### `HandlerMethodReturnValueHandler`
줄여서 `ReturnValueHandler`

`ArgumentResolver`와 비슷한데, 이것은 응답 값을 변환하고 처리한다.

컨트롤러에서 String으로 뷰 이름을 반환해도, 동작하는 이유가 바로 ReturnValueHandler 덕분이다.