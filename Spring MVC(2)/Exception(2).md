# Exception(2): API ์์ธ ์ฒ๋ฆฌ
 + ๐ผ๊ฐ ์์ธ ์ํฉ์ ๋ง๋ ์ค๋ฅ ์๋ต ์คํ์ ์ ํ๊ณ , `JSON`์ผ๋ก ๋ฐ์ดํฐ๋ฅผ ๋ด๋ ค์ฃผ์ด์ผ ํจ

## Servlet ์ค๋ฅ ํ์ด์ง

> ApiExceptionController
```java
@Slf4j
@RestController
public class ApiExceptionController {

    //header ์ Accept : application/json ์ธ์ง check ํด์ผํด
    @GetMapping("/api/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id){
        if(id.equals("ex")){
            throw new RuntimeException("์๋ชป๋ ์ฌ์ฉ์");    //์ค๋ฅ ๋ฐ์
        }

        return new MemberDto(id, "hello " + id);    //์ ์ ๋์
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
    //API ์์ธ ์ฒ๋ฆฌ
    @RequestMapping(value = "/error-page/500", produces = MediaType.APPLICATION_JSON_VALUE)  //์ฐ์ ์์๋ฅผ ๊ฐ์ง
    public ResponseEntity<Map<String, Object>> errorPage500Api(
            HttpServletRequest request, HttpServletResponse response){

        log.info("API errorPage 500");

        Map<String, Object> result = new HashMap<>();  //์์ ๋ณด์ฅ X
        Exception ex = (Exception) request.getAttribute(ERROR_EXCEPTION);
        result.put("status", request.getAttribute(ERROR_STATUS_CODE));    //500
        result.put("message", ex.getMessage());             //์๋ชป๋ ์ฌ์ฉ์
        Integer statusCode = (Integer) request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);

        return new ResponseEntity<>(result, HttpStatus.valueOf(statusCode));
    }
```
 + ์ค๋ฅ ํ์ด์ง Controller๊ฐ JSON ํ์์ผ๋ก ์๋ต
 + `produces = MediaType.APPLICATION_JSON_VALUE` :
   + ํด๋ผ์ด์ธํธ ์์ฒญ HTTP Header์ `Accept` ์ ๊ฐ์ด `application/json` ์ผ ๋ ํด๋น ๋ฉ์๋๊ฐ ํธ์ถ๋
 ```
 {
 "message": "์๋ชป๋ ์ฌ์ฉ์",
 "status": 500
}
```

## Spring boot ์์ธ ์ฒ๋ฆฌ
 + ๊ธฐ๋ณธ ์ค์ :  ์ค๋ฅ ๋ฐ์์ /error ๋ฅผ ์ค๋ฅ ํ์ด์ง๋ก ์์ฒญ
   + `BasicErrorController` ๋ ์ด ๊ฒฝ๋ก๋ฅผ ๊ธฐ๋ณธ์ผ๋ก ๋งคํ
   
> BasicErrorController 
```java
  @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
  public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {}
  
  @RequestMapping
  public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {}
```
 + `/error` ๋์ผํ ๊ฒฝ๋ก๋ฅผ ์ฒ๋ฆฌํ๋ `errorHtml()` , `error()` ๋ ๋ฉ์๋
   + `errorHtml()` : ํด๋ผ์ด์ธํธ ์์ฒญ์ Accept ํด๋ ๊ฐ์ด `text/html` ์ธ ๊ฒฝ์ฐ์ ํธ์ถ๋๋ฉฐ,  view๋ฅผ ์ ๊ณต
   + `error()` : ๊ทธ์ธ ๊ฒฝ์ฐ์ ํธ์ถ๋๋ฉฐ, **ResponseEntity** ๋ก HTTP Body์ **JSON ๋ฐ์ดํฐ**๋ฅผ ๋ฐํ

## HandlerExceptionResolver
 + WAS๋, ๋ฐ์ํ ์์ธ๋ฅผ HTTP ์ํ์ฝ๋ 500์ผ๋ก ์ฒ๋ฆฌ(throw error..)
   + :question: ๋ฐ์ํ๋ ์์ธ์ ๋ฐ๋ผ์, ์ํ์ฝ๋๋ฅผ ๋ค๋ฅด๊ฒ ์ฒ๋ฆฌํ๋ ค๋ฉด?
```java
@GetMapping("/api/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id){
        if(id.equals("ex")){
            throw new RuntimeException("์๋ชป๋ ์ฌ์ฉ์");
        }

        if(id.equals("bad")){    //์๋์ผ๋ก ์ํ ์ฝ๋ 500
            throw new IllegalArgumentException("์๋ชป๋ ์๋ ฅ ๊ฐ");
        }
        return new MemberDto(id, "hello " + id);
    }
```
 + `IllegalArgumentException` : ํด๋ผ์ด์ธํธ ์ชฝ์์ ์๋ชปํ `400 ์ฝ๋`๊ฐ ์๋, ์๋ `500 ์ฝ๋` ์ฒ๋ฆฌ
 
### :heavy_check_mark: HandlerExceptonResolver ๊ฐ์
 + `ExceptionResolver` : ์ปจํธ๋กค๋ฌ ๋ฐ์ผ๋ก ๋์ ธ์ง ์์ธ๋ฅผ ํด๊ฒฐํ๊ณ , ๋์์ ์๋ก ์ ์
   + ์์ธ๋ฅผ ์์ ํ ์ฒ๋ฆฌํ ๊ฒฝ์ฐ, ๋ค์ ์ปจํ์ด๋์์ ์์ฒญ์ ํ์ง X
 
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
                return new ModelAndView();    //ํ๋ฆ return, ์์ธ ๋จน๊ณ  ์๋ก์ด 400 ์์ธ ์ ๋ฌ
            }
        }catch(IOException e){
            e.printStackTrace();
        }
        return null;
    }
}
```
 + `ModelAndView` ๋ฐํ : ๊ธฐ์กด ์์ธ๋ฅผ ์ฒ๋ฆฌ(ํด๊ฒฐ) ํด์, **์ ์ ํ๋ฆ์ผ๋ก ๋ณ๊ฒฝ**ํ๋ ์ญํ 
   + ๊ธฐ์กด์ `500 ์ค๋ฅ`๋ ๋จน๊ณ , WAS์์๋ ์๋กญ๊ฒ ์ง์ ๋ `400 ์ฝ๋`๋ฅผ ์ธ์
   
> WebConfig: Resolver ์ถ๊ฐ
```java
  @Override
  public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
   resolvers.add(new MyHandlerExceptionResolver());
  }
```
> :pushpin: Resolver์ ๋ฐํ ๊ฐ์ ๋ฐ๋ฅธ DispatcherServlet ๋์ ๋ฐฉ์
 + `๋น ModelAndView`: ๋ทฐ๋ฅผ ๋ ๋๋ง ํ์ง ์๊ณ , ์ ์ ํ๋ฆ์ผ๋ก ์๋ธ๋ฆฟ์ด ๋ฆฌํด
 + `ModelAndView ์ง์ `: View , Model ๋ฑ์ ์ ๋ณด๋ฅผ ์ง์ ํด์ ๋ฐํํ๋ฉด ๋ทฐ๋ฅผ ๋ ๋๋ง
 + `null` : ๋ค์ ExceptionResolver ๋ฅผ ์ฐพ์์ ์คํ
   + ์ฒ๋ฆฌ ๊ฐ๋ฅํ ExceptionResolver ๊ฐ ์์ผ๋ฉด ์์ธ ์ฒ๋ฆฌ๊ฐ X, ๊ธฐ์กด์ ๋ฐ์ํ ์์ธ๋ฅผ ์๋ธ๋ฆฟ ๋ฐ์ผ๋ก ๋์ง

> :pushpin: ExceptionResolver ํ์ฉ
 + **์์ธ ์ํ ์ฝ๋ ๋ณํ**
   + ์์ธ๋ฅผ `response.sendError(xxx)` ํธ์ถ๋ก ๋ณ๊ฒฝํด์ ์๋ธ๋ฆฟ์์ ์ํ ์ฝ๋์ ๋ฐ๋ฅธ ์ค๋ฅ๋ฅผ ์ฒ๋ฆฌํ๋๋ก ์์
 + **๋ทฐ ํํ๋ฆฟ ์ฒ๋ฆฌ**
   + ModelAndView ์ ๊ฐ์ ์ฑ์์ ์์ธ์ ๋ฐ๋ฅธ ์๋ก์ด ์ค๋ฅ ํ๋ฉด ๋ทฐ ๋ ๋๋ง ํด์ ๊ณ ๊ฐ์๊ฒ ์ ๊ณต
 + **API ์๋ต ์ฒ๋ฆฌ**
   + `response.getWriter().println("hello")`์ฒ๋ผ HTTP ์๋ต ๋ฐ๋์ ์ง์  ๋ฐ์ดํฐ๋ฅผ ๋ฃ์ฃผ๊ธฐ ๊ฐ๋ฅ
   + `JSON` => API ์๋ต ์ฒ๋ฆฌ ๊ฐ๋ฅ

### :heavy_check_mark: HandlerExceptionResolver ํ์ฉ
 + _WAS_ ๊น์ง ์์ธ๊ฐ ๋์ ธ์ง๊ณ ,  ์ค๋ฅ ํ์ด์ง ์ ๋ณด๋ฅผ ์ฐพ์์ ๋ค์ `/error` ๋ฅผ ํธ์ถํ๋ ๊ณผ์ ์ ๋๋ฌด ๋ณต์ก
 + `ExceptionResolver` ๋ฅผ ํ์ฉํ๋ฉด _์คํ๋ง MVC์์ ์์ธ์ฒ๋ฆฌ๋ฅผ ์์ ํ ๋๋ผ ์ ์์_
 
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

                    return new ModelAndView();   //์์ธ ๋จน๊ณ  ์ ์ ์ ๋ฌ(๋ค์ ์์ฒญ X)
                }else{
                    // TEXT/HTML
                    return new ModelAndView("error/500");   //์ ์ ๋ทฐ ์ ๋ฌ(๋ค์ ์์ฒญ X)
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
       throw new UserException("์ฌ์ฉ์ ์ค๋ฅ");
   }
```
 + `UserException` : RuntimeException ๊ตฌํ ํด๋์ค

## Spring ์ ๊ณต ExceptionResolver
 + ์คํ๋ง ๋ถํธ๊ฐ ๊ธฐ๋ณธ์ผ๋ก ์ ๊ณตํ๋ ExceptionResolver 
   + ExceptionHandlerExceptionResolver
   + ResponseStatusExceptionResolver
   + DefaultHandlerExceptionResolver(์ฐ์  ์์ ๊ฐ์ฅ ๋ฎ์)


### :heavy_check_mark: ResponseStatusExceptionResolver
 + ์์ธ์ ๋ฐ๋ผ์, HTTP ์ํ ์ฝ๋๋ฅผ ์ง์ ํด์ค
   + `@ResponseStatus` ๊ฐ ๋ฌ๋ ค์๋ ์์ธ
   + `ResponseStatusException` ์์ธ
   
#### 1)@ResponseStatus
> BadRequestException
```java
@ResponseStatus(code = HttpStatus.BAD_REQUEST, reason="error.bad")  //response.sendError()
public class BadRequestException extends RuntimeException {
}
```
  + ์ปจํธ๋กค๋ฌ ๋ฐ์ผ๋ก ์์ธ๊ฐ ๋์ ธ์ง๋ฉด, `ResponseStatusExceptionResolver` ์์ธ๊ฐ ํด๋น ์ ๋ธํ์ด์์ ํ์ธ
    + ์ค๋ฅ ์ฝ๋๋ฅผ ๋ณ๊ฒฝํ๊ณ (ex)_HttpStatus.BAD_REQUEST_), ๋ฉ์์ง๋ ๋ด์
  + `error.bad` : reason ์ MessageSource ์์ ์ฐพ๋ ๊ธฐ๋ฅ ์ ๊ณต
    + `messages.properties`์ ์ถ๊ฐ
  + :bulb: ๋ด๋ถ์์ `response.sendError` ํธ์ถ => WAS์์ ์ค๋ฅ ํ์ด์ง(`/error`) ๋ด๋ถ ์์ฒญ
    
#### 2)ResponseStatusException
 + `ResponseStatusException`: ๊ฐ๋ฐ์๊ฐ ์ง์  ๋ณ๊ฒฝํ์ง ๋ชปํ๋ ์์ธ์ฒ๋ผ, ์กฐ๊ฑด์ ๋ฐ๋ผ ๋์ ์ผ๋ก ๋ณ๊ฒฝ ๊ฐ๋ฅ
> ApiExceptionController - ์ถ๊ฐ
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
 + ์คํ๋ง ๋ด๋ถ์์ ๋ฐ์ํ๋ ์คํ๋ง ์์ธ๋ฅผ ํด๊ฒฐ
   + ex) ํ๋ผ๋ฏธํฐ ๋ฐ์ธ๋ฉ ์์ ์, ํ์ ์ค๋ฅ => `TypeMismatchException`
   + 500 ์ค๋ฅ๊ฐ ๋ฐ์ํ์ง๋ง, ํด๋ผ์ด์ธํธ ์๋ชป์ด๋ฏ๋ก HTTP ์ํ ์ฝ๋ 400์ด ๋ง์
> defaultException
```java
    @GetMapping("/api-default-handler-ex")
    public String defaultException(@RequestParam Integer Data){
        return "ok";
    }
```
 + :bulb: ๋ด๋ถ์์ `response.sendError` ํธ์ถ => WAS์์ ์ค๋ฅ ํ์ด์ง(`/error`)๋ฅผ ๋ด๋ถ ์์ฒญ

๐คจ _ํ์ง๋ง, ๋ ๋ฐฉ์ ๋ชจ๋ API ์ค๋ฅ ์๋ต์ ๊ฒฝ์ฐ ModelAndView๋ฅผ ๋ฐํํด, ์ง์  ๋ฐ์ดํฐ๋ฅผ ๋ณํํด ๋ฃ์ด์ผํ๋ ๋ฌธ์  ์กด์ฌ!_

### :heavy_check_mark: ExceptionHandlerExceptionResolver
 + `@ExceptionHandler` : API ์์ธ ์ฒ๋ฆฌ ๋ฌธ์ ์ ์ ํด๊ฒฐ์ฑ์ด๋ฉฐ, ์ค๋ฌด์์ ์ฌ์ฉํ๋ ๋ฐฉ์

> ์์ธ ๋ฐ์์ API ์๋ต์ผ๋ก ์ฌ์ฉํ๋ ๊ฐ์ฒด
```java
  @Data
  @AllArgsConstructor
  public class ErrorResult {
      private String code;
      private String message;
  }
```

> ApiExceptionV2Controller(1)
```java
    //ExceptionHandlerExceptionResolver ๊ฐ ์ฒ๋ฆฌ
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExHandle(IllegalArgumentException e){
        return new ErrorResult("BAD", e.getMessage());  //์ ์ ํ๋ฆ(200)
    }
```
#### ์คํ ํ๋ฆ: IllegalArgumentException
1) `IllegalArgumentException` ์์ธ๊ฐ ์ปจํธ๋กค๋ฌ ๋ฐ์ผ๋ก ์ ๋ฌ</br>
2) `ExceptionResolver`๊ฐ ์๋ํ๊ณ  ์ฐ์ ์์๊ฐ ๊ฐ์ฅ ๋์ `ExceptionHandlerExceptionResolver`๊ฐ ์คํ๋๋ฉฐ, ์ปจํธ๋กค๋ฌ์ ํด๋น ์์ธ๋ฅผ ์ฒ๋ฆฌ ๊ฐ๋ฅํ `@ExceptionHandler`๊ฐ ์๋์ง ํ์ธ</br>
3) `@Responsebody`๊ฐ ์ ์ฉ๋์ด,  HTTP Converter๋ฅผ ์ด์ฉํด ์๋ต์ด JSON์ผ๋ก ๋ฐํ๋</br>
4) `@ResponseStatus` : HTTP ์ํ ์ฝ๋ ์ง์ , ์ฌ๊ธฐ์๋ 400์ผ๋ก ์๋ต</br>

:star2: _์ํ ์ฝ๋๋ฅผ ๋ฐ๋ก ์ง์ ์ํด์ฃผ๋ฉด, ์ ์ ํ๋ฆ ์ฒ๋ฆฌ๋์ด 200 OK๊ฐ ์ฝ๋๋ก ์ง์ ๋_:star2:

> ApiExceptionV2Controller(2)
```java
    @ExceptionHandler 
    public ResponseEntity<ErrorResult> userHandler(UserException e){   //์๋ต ๊ฐ๋ฅ
        ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
        return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
    }
 ```
 +  `ResponseEntity` : HTTP ์ปจ๋ฒํฐ ์ฌ์ฉํด HTTP ๋ฉ์์ง ๋ฐ๋์ ์ง์  ์๋ต
 
 > ApiExceptionV2Controller(3)
```java
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)    //์ํ ์ฝ๋ 500
    @ExceptionHandler
    public ErrorResult exHandler(Exception e){  //์์์ ์ฒ๋ฆฌ ๋ชปํ๋ ์๋ฌ๋ค(๊ณตํต ์ฒ๋ฆฌ ์๋ฌ)
        return new ErrorResult("EX", "๋ด๋ถ ์ค๋ฅ");
    }
```
 + ` @ExceptionHandler`: ํด๋น ์ปจํธ๋กค๋ฌ์์ ์ฒ๋ฆฌํ๊ณ  ์ถ์ ์์ธ๋ฅผ ์ง์ (์์ธ์ ์์ ํด๋์ค ํฌํจ)
   + `Exception` : ์์ธ์ ๋ถ๋ชจ ํด๋์ค, ๊ตฌ์ฒด์  ์์ ํด๋์ค์์ ์ฒ๋ฆฌ ๋ชปํ๋ ์์ธ๋ค์ด ๋์ด์ด
 + `RuntimeException`์ด ์์ธ๋ก ๋์ ธ์ง๋ฉด, ์ด ๋ฉ์๋ ํธ์ถ 

## @ControllerAdvice
 + `@ControllerAdvice` : ํ๋์ Controller์ ๋ชจ์ฌ์๋ ์ ์ ์ฝ๋์, ์์ธ ์ฒ๋ฆฌ ์ฝ๋๋ฅผ ๋ถ๋ฆฌ ๊ฐ๋ฅ

> ExControllerAdvice
```java
@RestControllerAdvice
public class ExControllerAdvice {

    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExHandler(IllegalArgumentException e){
        return new ErrorResult("BAD", e.getMessage());  //์ ์ ํ๋ฆ(200)
    }

    @ExceptionHandler
    public ResponseEntity<ErrorResult> userHandler(UserException e){
        ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
        return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
    }

    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler
    public ErrorResult exHandler(Exception e){  //์์์ ์ฒ๋ฆฌ ๋ชปํ๋ ์๋ฌ๋ค(๊ณตํต ์ฒ๋ฆฌ ์๋ฌ)
        return new ErrorResult("EX", "๋ด๋ถ ์ค๋ฅ");

    }
}
```
 + `@RestControllerAdvice` : `@RestController` + `@ControlerAdvice`
 + ๋์์ ์ง์ ํ์ง ์์ผ๋ฉด, ๋ชจ๋  ์ปจํธ๋กค๋ฌ์ ์ ์ฉ๋
   + `@ControllerAdvice(annotations = RestController.class)`
   + `@ControllerAdvice("org.example.controllers")`
   
> ApiExceptionV2Controller
```java
@RestController
public class ApiExceptionV2Controller {
    //์์ธ ์ฒ๋ฆฌ ๋ถ๋ถ ๋ถ๋ฆฌ
    @GetMapping("/api2/members/{id}")
    public ApiExceptionController.MemberDto getMember(@PathVariable("id") String id){
        if(id.equals("ex")){
            throw new RuntimeException("์๋ชป๋ ์ฌ์ฉ์");
        }

        if(id.equals("bad")){  
            throw new IllegalArgumentException("์๋ชป๋ ์๋ ฅ ๊ฐ");
        }

        if(id.equals("uer-ex")){
            throw new UserException("์ฌ์ฉ์ ์ค๋ฅ");
        }
        return new ApiExceptionController.MemberDto(id, "hello " + id);
    }

    @Data
    @AllArgsConstructor
    static class MemberDto{
        private  String memberId;
        private String name;
    }
}
```
