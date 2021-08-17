# Exception(2): API 예외 처리
 + 🌼각 예외 상황에 맞는 오류 응답 스펙을 정하고, `JSON`으로 데이터를 내려주어야 함

## Servlet 오류 페이지

> ApiExceptionController
```java
@Slf4j
@RestController
public class ApiExceptionController {

    //header 의 Accept : application/json 인지 check 해야해
    @GetMapping("/api/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id){
        if(id.equals("ex")){
            throw new RuntimeException("잘못된 사용자");    //오류 발생
        }

        return new MemberDto(id, "hello " + id);    //정상 동작
    }

    @Data
    @AllArgsConstructor
    static class MemberDto{
        private  String memberId;
        private String name;
    }
}
```
 
> ErrorPageController
```java
    //API 예외 처리
    @RequestMapping(value = "/error-page/500", produces = MediaType.APPLICATION_JSON_VALUE)  //우선순위를 가짐
    public ResponseEntity<Map<String, Object>> errorPage500Api(
            HttpServletRequest request, HttpServletResponse response){

        log.info("API errorPage 500");

        Map<String, Object> result = new HashMap<>();  //순서 보장 X
        Exception ex = (Exception) request.getAttribute(ERROR_EXCEPTION);
        result.put("status", request.getAttribute(ERROR_STATUS_CODE));    //500
        result.put("message", ex.getMessage());             //잘못된 사용자
        Integer statusCode = (Integer) request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);

        return new ResponseEntity<>(result, HttpStatus.valueOf(statusCode));
    }
```
 + 오류 페이지 Controller가 JSON 형식으로 응답
 + `produces = MediaType.APPLICATION_JSON_VALUE` :
   + 클라이언트 요청 HTTP Header의 `Accept` 의 값이 `application/json` 일 때 해당 메서드가 호출됌
 ```
 {
 "message": "잘못된 사용자",
 "status": 500
}
```

## Spring boot 예외 처리
 + 기본 설정:  오류 발생시 /error 를 오류 페이지로 요청
   + `BasicErrorController` 는 이 경로를 기본으로 매핑
   
> BasicErrorController 
```java
  @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
  public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {}
  
  @RequestMapping
  public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {}
```
 + `/error` 동일한 경로를 처리하는 `errorHtml()` , `error()` 두 메서드
   + `errorHtml()` : 클라이언트 요청의 Accept 해더 값이 `text/html` 인 경우에 호출되며,  view를 제공
   + `error()` : 그외 경우에 호출되며, **ResponseEntity** 로 HTTP Body에 **JSON 데이터**를 반환

## HandlerExceptionResolver
 + WAS는, 발생한 예외를 HTTP 상태코드 500으로 처리(throw error..)
   + :question: 발생하는 예외에 따라서, 상태코드를 다르게 처리하려면?
```java
@GetMapping("/api/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id){
        if(id.equals("ex")){
            throw new RuntimeException("잘못된 사용자");
        }

        if(id.equals("bad")){    //자동으로 상태 코드 500
            throw new IllegalArgumentException("잘못된 입력 값");
        }
        return new MemberDto(id, "hello " + id);
    }
```
 + `IllegalArgumentException` : 클라이언트 쪽에서 잘못한 `400 코드`가 아닌, 자동 `500 코드` 처리
 
### :heavy_check_mark: HandlerExceptonResolver 개요
 + `ExceptionResolver` : 컨트롤러 밖으로 던져진 예외를 해결하고, 동작을 새로 정의
   + 예외를 완전히 처리한 경우, 다시 컨테이너에서 요청을 하지 X
 
<img src="https://user-images.githubusercontent.com/71436576/129685803-82d4fa46-839c-4ccc-939b-63f09f8fb7de.png" width=50% height=50%>

> HandlerExceptionResolver
```java
@Slf4j
public class MyHandlerExceptionResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {

        try{
            if( ex instanceof IllegalArgumentException){
                log.info("IllegalArgumentException resolver to 400");
                response.sendError(HttpServletResponse.SC_BAD_REQUEST, ex.getMessage());
                return new ModelAndView();    //흐름 return, 예외 먹고 새로운 400 예외 전달
            }
        }catch(IOException e){
            e.printStackTrace();
        }
        return null;
    }
}
```
 + `ModelAndView` 반환 : 기존 예외를 처리(해결) 해서, **정상 흐름으로 변경**하는 역할
   + 기존의 `500 오류`는 먹고, WAS에서는 새롭게 지정된 `400 코드`를 인식
   
> WebConfig: Resolver 추가
```java
  @Override
  public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
   resolvers.add(new MyHandlerExceptionResolver());
  }
```
> :pushpin: Resolver의 반환 값에 따른 DispatcherServlet 동작 방식
 + `빈 ModelAndView`: 뷰를 렌더링 하지 않고, 정상 흐름으로 서블릿이 리턴
 + `ModelAndView 지정`: View , Model 등의 정보를 지정해서 반환하면 뷰를 렌더링
 + `null` : 다음 ExceptionResolver 를 찾아서 실행
   + 처리 가능한 ExceptionResolver 가 없으면 예외 처리가 X, 기존에 발생한 예외를 서블릿 밖으로 던짐

> :pushpin: ExceptionResolver 활용
 + **예외 상태 코드 변환**
   + 예외를 `response.sendError(xxx)` 호출로 변경해서 서블릿에서 상태 코드에 따른 오류를 처리하도록 위임
 + **뷰 템플릿 처리**
   + ModelAndView 에 값을 채워서 예외에 따른 새로운 오류 화면 뷰 렌더링 해서 고객에게 제공
 + **API 응답 처리**
   + `response.getWriter().println("hello")`처럼 HTTP 응답 바디에 직접 데이터를 넣주기 가능
   + `JSON` => API 응답 처리 가능

### :heavy_check_mark: HandlerExceptionResolver 활용
 + WAS까지 예외가 던져지고,  오류 페이지 정보를 찾아서 다시 `/error` 를 호출하는 과정은 너무 복잡
 + `ExceptionResolver` 를 활용하면 _스프링 MVC에서 예외처리를 완전히 끝낼 수 있음_
 
> UserHandlerExceptionResolver
```java
public class UserHandlerExceptionResolver implements HandlerExceptionResolver {

    private final ObjectMapper objectMapper = new ObjectMapper();
    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        try{
            if(ex instanceof UserException){
                log.info("UserException resolver to 400");
                String acceptHeader = request.getHeader("accept");
                response.setStatus(HttpServletResponse.SC_BAD_REQUEST);

                if("application/json".equals(acceptHeader)) {
                    Map<String, Object> errorResult = new HashMap<>();
                    errorResult.put("ex", ex.getClass());
                    errorResult.put("message", ex.getMessage());
                    String result = objectMapper.writeValueAsString(errorResult);

                    response.setContentType("application/json");
                    response.setCharacterEncoding("utf-8");
                    response.getWriter().write(result);

                    return new ModelAndView();   //예외 먹고 정상 전달(다시 요청 X)
                }else{
                    // TEXT/HTML
                    return new ModelAndView("error/500");   //정상 뷰 전달(다시 요청 X)
                }
            }
        }catch(IOException e){
            log.error("resolver ex", e);
        }
        return null;
    }
}
```

> ApiExceptionController
```java
  if (id.equals("user-ex")) {
       throw new UserException("사용자 오류");
   }
```
 + `UserException` : RuntimeException 구현 클래스

## Spring 제공 ExceptionResolver
 + 스프링 부트가 기본으로 제공하는 ExceptionResolver 
   + ExceptionHandlerExceptionResolver
   + ResponseStatusExceptionResolver
   + DefaultHandlerExceptionResolver(우선 순위 가장 낮음)


### :heavy_check_mark: ResponseStatusExceptionResolver
 + 예외에 따라서, HTTP 상태 코드를 지정해줌
   + `@ResponseStatus` 가 달려있는 예외
   + `ResponseStatusException` 예외
   
#### 1)@ResponseStatus
> BadRequestException
```java
@ResponseStatus(code = HttpStatus.BAD_REQUEST, reason="error.bad")  //response.sendError()
public class BadRequestException extends RuntimeException {
}
```
  + 컨트롤러 밖으로 예외가 던져지면, `ResponseStatusExceptionResolver` 예외가 해당 애노테이션을 확인
    + 오류 코드를 변경하고(ex)_HttpStatus.BAD_REQUEST_), 메시지도 담음
  + :pencil2: 내부에서 `response.sendError` 호출 => WAS에서 오류 페이지(`/error`) 내부 요청
  + `error.bad` : reason 을 MessageSource 에서 찾는 기능 제공
    + `messages.properties`에 추가
    
#### 2)ResponseStatusException
 + `ResponseStatusException`: 개발자가 직접 변경하지 못하는 예외처럼, 조건에 따라 동적으로 변경 가능
> ApiExceptionController - 추가
```java
    @GetMapping("/api/response-status-ex1")
    public String responseStatusEx1(){
        throw new BadRequestException();
    }

    @GetMapping("/api/response-status-ex2")
    public String responseStatusEx2(){
        throw new ResponseStatusException(HttpStatus.NOT_FOUND, "error-bad", new IllegalArgumentException());
    }
```
### :heavy_check_mark: DefaultHandlerExceptionResolver
 + 스프링 내부에서 발생하는 스프링 예외를 해결
   + ex) 파라미터 바인딩 시점에, 타입 오류 => `TypeMismatchException`
   + 500 오류가 발생하지만, 클라이언트 잘못이므로 HTTP 상태 코드 400이 맞음
> defaultException
```java
    @GetMapping("/api-default-handler-ex")
    public String defaultException(@RequestParam Integer Data){
        return "ok";
    }
```
 + :pencil2: 내부에서 `response.sendError` 호출 => WAS에서 오류 페이지(`/error`)를 내부 요청

🤨 _하지만, 두 방식 모두 API 오류 응답의 경우 ModelAndView를 반환해, 직접 데이터를 변환해 넣어야하는 문제 존재!_

### :heavy_check_mark: ExceptionHandlerExceptionResolver
