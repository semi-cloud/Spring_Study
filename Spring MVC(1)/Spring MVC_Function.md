# Spring MVC-기본 기능

## Logging
 + 운영 시스템에서는 System.out.println() 같은 시스템 콘솔 대신, 로깅 라이브러리를 사용해 로그를 출력
  + `SLF4J ` : 스프링 부트 기본 로깅 라이브러리
 
> LogTestController
```java
//@Slf4j             //로그 출력 라이브러리
@RestController      //RestAPI
public class LogTestController {
    private final Logger log = LoggerFactory.getLogger(getClass());

    @RequestMapping("/log-test")
    public String LogTest() {
        String name = "Spring";

        //순서대로(레벨 낮 => 높은 순서대로)
        //log.trace("trace log =" + name);  //불필요한 연산이 trace 출력 전에 일어남
        log.trace("trace log={}", name);
        log.debug("trace log={}", name);    //개발 서버
        log.info("trace log={}", name);     //운영 서버(기본은 info 부터)
        log.warn("trace log={}", name);
        log.error("trace log={}", name);

        return "ok";    //view 이름이 아니라 문자 그대로 반환
    }
}
```
  + `RestConroller` : 반한 값이 String 인 경우, 뷰를 찾는 것이 아닌 HTTP 메시지 바디에 직접 입력
  
### :pushpin: 로그 레벨과 사용법
+ `LEVEL`: **TRACE > DEBUG > INFO > WARN > ERROR**
 + 개발 서버는 debug 출력
 + 운영 서버는 info 출력
```
    #전체 로그 레벨 설정(기본 info)
    logging.level.root=info
    
    #hello.springmvc 패키지와 그 하위 로그 레벨 설정
    logging.level.hello.springmvc=debug
```
+ 사용법 : `log.debug("data={}", data)` 
  + ~~log.debug("data="+data) : 로그 출력 레벨과 무관하게 불필요한 연산이 발생~~

### :pushpin: 로그 사용시 장점
+ 쓰레드 정보, 클래스 이름 같은 부가 정보도 볼 수 있음
+ 로그 레벨에 따라 로그를 서버의 종류에 맞게 조절 가능 (운영 서버, 개발 서버..)
+ 콘솔 뿐만 아니라, 파일이나 네트워크와 같은 별도의 위치에 남길 수 있음
+ 성능도 일반 System.out보다 좋음(내부 버퍼링, 멀티 쓰레드 등등) 
  + :star2: 실무에서는 무조건 Log를 사용해야함!
  
 
## HTTP 요청 매핑

> 기본
```java
  @GetMapping("/hello-basic")
    public String helloBasic(){
        log.info("helloBasic");
        return "ok";
    }
```
> PathVariable(경로 변수) 사용
```java
    //URL 경로 변수(쿼리 파라미터 방식 아님)

    @GetMapping("/mapping/{userId}")
    public String mappingPath(@PathVariable("userId") String data){
        log.info("mappingPath userId={}", data);
        return "ok";
    }
```
  + 최근 HTTP API는 다음과 같이 **리소스 경로에 식별자를 넣는 스타일**을 선호!
  + `@PathVariable`: @RequestMapping이 템플릿화 한 URL 경로와 매칭된 부분을 조회
    + @PathVariable 의 이름과 파라미터 이름이 같으면 생략 가능

> 미디어 타입 조건 매핑 - HTTP 요청 Content-Type, consume
```java
    @PostMapping(value = "/mapping-consume", consumes = MediaType.APPLICATION_JSON_VALUE)
    public String mappingConsumes() {
        log.info("mappingConsumes");
        return "ok";
    }
```
 + `consumes` : **HTTP 요청의 Content-Type 헤더**를 기반으로 미디어 타입으로 매핑
   + 맞지 않으면, HTTP 415 상태코드(Unsupported Media Type)을 반환

> 미디어 타입 조건 매핑 - HTTP 요청 Accept, produce
```java
    @PostMapping(value = "/mapping-produce", produces = "text/html")
    public String mappingProduces() {
        log.info("mappingProduces");
        return "ok";
    }
```
  + `produces` : **HTTP 요청의 Accept 헤더**를 기반으로 미디어 타입으로 매핑
    + 맞지 않으면, HTTP 406 상태코드(Not Acceptable)을 반환
 
## HTTP 요청 데이터 조회

### :pushpin: HTTP 요청- 기본, 헤더 조회

#### 회원 관리 API(매핑)
+ 회원 목록 조회: GET /users
+ 회원 등록: POST /users
+ 회원 조회: GET /users/{userId}
+ 회원 수정: PATCH /users/{userId}
+ 회원 삭제: DELETE /users/{userId}

#### HTTP 요청 정보 조회
> HTTP 요청 정보들 조회하는 Controller
```java
@Slf4j
@RestController
public class RequestHeaderController {

    @RequestMapping("/headers")
    public String headers(HttpServletRequest request,
                          HttpServletResponse response,
                          HttpMethod httpMethod,
                          Locale locale,
                          @RequestHeader MultiValueMap<String, String> headerMap,
                          @RequestHeader("host") String host,
                          @CookieValue(value = "myCookie", required = false) String cookie

                          ){
        log.info("request={}", request);
        log.info("response={}", response);
        log.info("httpMethod={}", httpMethod);
        log.info("locale={}", locale);
        log.info("headerMap={}", headerMap);
        log.info("header host={}", host);
        log.info("myCookie={}", cookie);
        return "ok";
    }
}
```
+ `@RequestHeader MultiValueMap<String, String>` : 모든 HTTP 헤더를 MultiValueMap 형식으로 조회
  + :question: MultiValueMap : 하나의 키에 여러값을 받을 수 있음  
+ `@RequestHeader("host") String host` : 특정 HTTP 헤더 조회


### :pushpin: HTTP 요청 파라미터-쿼리,HTML FORM

> 잠깐! 먼저 기억하고 가야 할 것이 있음

🌱 **클라이언트에서 서버로 요청 데이터를 전달할 때**</br>
 **1) GET- 쿼리 파라미터**</br>
  + /url?username=hello&age=20
  + 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달 
  
 **2) POST- HTML Form**</br>
  + content-type: application/x-www-form-urlencoded
  + 메시지 바디에 쿼리 파리미터 형식으로 전달 username=hello&age=20
  
 **3) HTTP message body**</br>
  + HTTP API에서 주로 사용, JSON, XML, TEXT (주로 json 형식)
  + POST, PUT, PATCH

:star2:**결국 GET 이던, POST던 둘다 요청 파라미터(Request Parameter) 조회**
 
> requestParamV1
```java
@Slf4j
@Controller
public class RequestParamController {

    @RequestMapping("/request-param-v1")
    public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));
        log.info("username={}, age={}", username, age);

        response.getWriter().write("ok");

    }
}
```
#### :heavy_check_mark: HTTP 요청 파라미터 - @RequestParam
+ 스프링에서 제공하는 `@RequestParam`을 통해 편리하게 사용 가능
+ String, int, Integer와 같은 단순 타입에 사용!

> requestParamV2
```java
    //v2
    @ResponseBody 
    @RequestMapping("/request-param-v2")
    public String requestParamV2(
            @RequestParam("username") String memberName,
            @RequestParam("age") int memberAge
    ){
        log.info("username={}, age={}", memberName, memberAge);
        return "ok";
    }
```
 + `@RequestParam` : 파라미터 이름으로 바인딩
   + request.getParameter("username")와 똑같이 동작!
   + HTTP 파라미터 이름이 변수 이름과 같으면 생략 가능 ex)String username, int age
 + `@ResponseBody` : View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력


>  requestParamReuquired-파라미터 필수 여부
```java
    //int = null X 객체타입만 null 가능해서 Integer 사용해야해!
    @ResponseBody  
    @RequestMapping("/request-param-required")
    public String requestParamReuquired(
            @RequestParam(required = true) String memberName,
            @RequestParam(required = false) Integer memberAge) {
        log.info("username={}, age={}", memberName, memberAge);
        return "ok";
    }
```
 + `@RequestParam(required = true)` : 파라미터 필수 여부(기본값)
 + 주의 1 : '/request-param?username=' 형태로 들어오면 빈 문자로 에러가 나지 않음
 + 주의 2 : `(required = false)`인 경우, null 값을 받을 수 있는 Integer 변경 혹은 defaultValue 사용 해야함
   + @RequestParam(required = true, **defaultValue = "guest"**) String username
   + @RequestParam(required = false, **defaultValue = "-1"**) int age


> requestParamMap-파라미터 Map으로 조회
```java
    @ResponseBody  //==restController
    @RequestMapping("/request-param-default")
    public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
        log.info("username={}, age={}", paramMap.get("username"), paramMap.get("age"));
        return "ok";
    }
```

#### :heavy_check_mark: HTTP 요청 파라미터 - @ModelAttribute

+ 요청 파라미터 받아서 필요한 객체 만들고, 값을 객체에 넣어주는 과정을 자동화해줌
+ String, int, Integer와 같은 단순 타입 외의 나머지 타입에 사용!

> HelloData
```java
@Data
public class HelloData {
    private String username;
    private int age;

}
```
 + 롬복 `@Data` : `@Getter` , `@Setter` , `@ToString` , `@EqualsAndHashCode` , `@RequiredArgsConstructor` 를 자동으로 적용해줌
 
> ModelAttributeV1
```java
    //ModelAttribute 생략 가능
    @ResponseBody
    @RequestMapping("/model-attribute-v1")
    public String modelAttributeV1(@ModelAttribute HelloData helloData){
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
        return "ok";
    }
```
 + `@ModelAttribute ` : HelloData 객체의 생성 + 객체에 값을 넣어줌
   + 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾은 후, 해당 프로퍼티의 setter를
호출해서 파라미터의 값을 입력(바인딩)

### :pushpin: HTTP 요청 파라미터-HTTP API
 + HTTP 메시지 바디를 통해 데이터가 직접 데이터가 넘어오는 경우는 @RequestParam , @ModelAttribute 를 사용 불가능! 

#### :heavy_check_mark: 단순 텍스트인 경우

> requestBodyStringV1,V2
```java
@Slf4j
@Controller
public class RequestBodyStringController {

    @PostMapping("/request-body-string-v1")
    public void requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException {
        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("messageBody={}", messageBody);

        response.getWriter().write("ok");
    }
    
     @PostMapping("/request-body-string-v2")
    public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {

        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
        log.info("messageBody={}", messageBody);
        responseWriter.write("ok");
    }
}
```
 + `InputStream(Reader)` : HTTP 요청 메시지 바디의 내용을 직접 조회
 + `OutputStream(Writer)` : HTTP 응답 메시지의 바디에 직접 결과 출력
 
> requestBodyStringV3 - **HttpEntity** 
```java
    //message body 에 있는 객체 자동 String 변환 <String>
    @PostMapping("/request-body-string-v3")
    public HttpEntity requestBodyStringV3(HttpEntity<String> httpEntity) throws IOException {

        String messageBody = httpEntity.getBody();
        log.info("messageBody={}", messageBody);

        return new HttpEntity<>("ok");
    }
```
 + `HttpEntity`: **HTTP header, body 정보**를 편리하게 조회 가능
   + **메시지 바디 정보를 직접 조회 하며, 요청 파라미터 조회 기능과는 관계 X**
 + 요청, 응답에 모두 사용 가능
   + 메시지 바디 정보 직접 반환, 헤더 정보 포함 가능
 + `RequestEntity` , `ResponseEntity` : Http Entity 상속 받은 객체들
   + HttpMethod, url 정보가 추가 기능(request) / HTTP 상태 코드 설정 가능(response)
   + `return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED)`


> requestBodyStringV4 - **@RequestBody** 
```java
    @ResponseBody
    @PostMapping("/request-body-string-v4")
    public String requestBodyStringV4(@RequestBody String messageBody) throws IOException {

        log.info("messageBody={}", messageBody);
        return "ok";
    }
}
```
 + `@RequestBody` : HTTP 메시지 바디 정보 편리하게 조회 가능
   + 헤더 정보 조회 : HttpEntity 를 사용하거나 @RequestHeader 를 사용!
 
 
#### :heavy_check_mark: JSON 형식인 경우

> requestBodyJsonV1-Servlet + ObjectMapper 이용
```java
@Slf4j
@Controller
public class RequestBodyJsonController {

    private ObjectMapper objectMapper = new ObjectMapper();

    @PostMapping("/request-body-json-v1")
    public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        HelloData data = objectMapper.readValue(messageBody, HelloData.class);
        
        response.getWriter().write("ok");
    }
}
```
 + @RequestBody 이용해 HTTP 메시지에서 데이터 꺼내서 messageBody에 저장하고, objectMapper를 통해서 JSON -> 자바 객체로 변환하는 과정이 너무 불편

> requestBodyJsonV2-**@RequestBody 객체 파라미터**
```java
    //객체를 파라미터로 넘김 => 자동으로 <json -> 원하는 문자나 객체>로 변환
    @ResponseBody
    @PostMapping("/request-body-json-v2")
    public String requestBodyJsonV2(@RequestBody HelloData data) throws IOException {

        log.info("username={}, age={}", data.getUsername(), data.getAge());
        return "ok";
    }
```
 + `@RequestBody 객체 파라미터` : HTTP 메시지 컨버터가 문자/객체(JSON 형식인 경우)로 자동 변환해줌
 +  애노테이션 생략 시, @ModelAttribute(요청 파라미터 시 사용) 가 적용되어 **생략은 불가능**
 
> requestBodyJsonV3- **@ResponseBody 객체 반환**
```java
    @ResponseBody
    @PostMapping("/request-body-json-v5")
    public HelloData requestBodyJsonV5(@RequestBody HelloData data) throws IOException {

        log.info("username={}, age={}", data.getUsername(), data.getAge());
        return data;
    }
}
```
 + `@RequestBody 요청` : JSON 요청 HTTP 메시지 컨버터 객체
 + `@ResponseBody 응답` : 객체 HTTP 메시지 컨버터 JSON 응답

## HTTP 응답 데이터 조회

### :pushpin: HTTP 응답-정적 리소스,템플릿

> 정적 리소스
 + 정적 리소스 경로 : src/main/resources/static 
 + 스프링 부트는 다음의 디렉토리에 있는 정적 리소스를 제공
   + `/static` , `/public`, `/resources` , `/META-INF/resources` 

> 뷰 템플릿(동적 HTML)
 + 뷰 템플릿 경로 : src/main/resources/templates
 
```java
 @RequestMapping("/response-view-v2")
 public String responseViewV2(Model model) {
     model.addAttribute("data", "hello!!");
     return "response/hello";
 }
```
 + 뷰의 논리 이름을 반환하면 `templates/response/hello.html` 뷰 템플릿이 렌더링

### :pushpin: HTTP 응답-HTTP API(메시지 바디)

 + HTTP API를 제공하는 경우에는 HTML이 아니라 데이터 전달 => Message Body에 실어 보냄!
   + 정적 리소스나 뷰 템플릿을 거치지 않고, 직접 HTTP 응답 메시지를 전달하는 경우
 + 서블릿을 이용한 초기버전 : `HttpServletResponse`의 response.getWriter().write("ok")로 데이터 전달

> Version2 - **HttpEntity, ResponseEntity(Http Status 추가)**
```java
 @GetMapping("/response-body-string-v2")
 public ResponseEntity<String> responseBodyV2() {
     return new ResponseEntity<>("ok", HttpStatus.OK);
 }
 
 @ResponseBody
 @GetMapping("/response-body-string-v3")
 public String responseBodyV3() {
     return "ok";
 }
 
 @GetMapping("/response-body-json-v1")
 public ResponseEntity<HelloData> responseBodyJsonV1() {
     HelloData helloData = new HelloData();
     helloData.setUsername("userA");
     helloData.setAge(20);
     return new ResponseEntity<>(helloData, HttpStatus.OK);
 }
```
 + `@ResponseBody`가 있어야 문자값으로 반환, 없으면 HttpEntity/ResponseEntitiy 객체를 통해서 문자(데이터) 전달해야함
   + `ResponseEntitiy` : HTTP 응답 코드 설정 가능
   + `@ResponseStatus(HttpStatus.OK)`는 @ResponseBody와 함께 쓸 수 있음!(응답 코드 설정)
 + `@ResponseBody` 와 `ResponseEntity` 는 **HTTP 메시지 컨버터** 통해서 HTTP 메시지 직접 입력
 + :star2: `@RestController` : 해당 컨트롤러에 모두 @ResponseBody 가 적용되는 효과

## HTTP 메시지 컨버터

+ HTTP API처럼 JSON 데이터를 HTTP 메시지 바디에서 직접 읽거나 쓰는것을 편리하게 도와줌
  + ViewResolver 대신에 HttpMessageConverter가 동작

#### :heavy_check_mark: HTTP 메시지 컨버터 동작 과정
 1) `canRead() , canWrite()` : 메시지 컨버터가 **해당 클래스, 미디어타입을 지원하는지 체크**</br>
 2) `read() , write()` : 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능</br>

#### 다음의 경우에 HTTP 메시지 컨버터 적용
 + HTTP 요청: @RequestBody , HttpEntity(RequestEntity) 
 + HTTP 응답: @ResponseBody , HttpEntity(ResponseEntity)

> 스프링 부트 기본 메시지 컨버터
```
0 = ByteArrayHttpMessageConverter
1 = StringHttpMessageConverter 
2 = MappingJackson2HttpMessageConverter
```
 + `ByteArrayHttpMessageConverter` : byte[] 데이터를 처리
   + `지원 클래스 타입`: byte[], `지원 미디어타입`: */* ,
   + 요청 예) @RequestBody byte[] data
   + 응답 예) @ResponseBody return byte[] 쓰기, 미디어타입 application/octet-stream
 + `StringHttpMessageConverter` : String 문자로 데이터를 처리
   + `지원 클래스 타입`: String , `지원 미디어타입`: */*
   + 요청 예) @RequestBody String data
   + 응답 예) @ResponseBody return "ok" 쓰기 미디어타입 text/plain
 + `MappingJackson2HttpMessageConverter` : application/json
   + `지원 클래스 타입`: 객체 또는 HashMap , `지원 미디어타입`: application/json 관련
   + 요청 예) @RequestBody HelloData data
   + 응답 예) @ResponseBody return helloData 쓰기 미디어타입 application/json 관련
  
### :pushpin: 요청 매핑 핸들러 어댑터 구조

<img src="https://user-images.githubusercontent.com/71436576/128367963-66cb74af-3dd0-4675-8dda-2144984b2778.png"
     width=50% height=50%>

#### 요청
 + 애노테이션 기반의 컨트롤러를 처리하는 `RequestMappingHandlerAdaptor` 가 ArgumentResolver 를 호출
   + `ArgumentResolver` : HTTP Message Converter 사용해서, 컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)을 생성
   + 다양한 종류의 ArgumentResolver들 존재(ex)@RequestBody 처리, HttpEntitiy 처리..)
   
#### 응답
 + `ReturnValueHandler` :  HTTP 메시지 컨버터를 호출해서 응답 결과를 만듬 
   + @ResponseBody 와 HttpEntity 를 처리하는 ReturnValueHandler가 존재
