# Exception
 + 웹 애플리케이션 :  사용자 **요청별로 별도의 쓰레드가 할당**되고, 서블릿 컨테이너 안에서 실행됌
 + 예외 발생시 애플리케이션에서 잡지 못해, 서블릿 밖에까지 예외가 전달될 경우 흐름
   + `WAS(ex)Tomcat ) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)`

## Servlet Exception

### ✔️ Servlet Container
 + `HTTP 상태 코드 500` 반환
   + Exception 의 경우, 서버 내부에서 처리할 수 없는 오류가 발생한 것으로 생각!
 ```java
 public void errorEx() {
     throw new RuntimeException("예외 발생!");  //500 status
 }
 ```

### ✔️ response.sendError()
 + HttpServletResponse에서 제공하며, **Servlet Container에게 오류 발생 정보 전달**(당장 예외 터진게 X)
   + `response.sendError(HTTP 상태 코드)`
   + `response.sendError(HTTP 상태 코드, 오류 메시지)`
 + Servlet Container는 고객에게 응답 전에, response에 메소드가 호출되었는지 확인후 설정된 오류 페이지를 제공
 
```
WAS(sendError 호출 기록 확인) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러
```
 + `WAS` : 해당 예외를 처리하는 오류 페이지 정보를 확인
## Servlet Exception: 예외페이지

> Servlet 오류 페이지 등록: Spring Boot
```java
@Component
public class WebServerCustomizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {

    @Override
    public void customize(ConfigurableWebServerFactory factory) {

        //오류가 WAS 까지 간후, 다시 반대로 와서 컨트롤러를 호출
        ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error-page/404");
        ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");

        ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/error-page/500");  

        factory.addErrorPages(errorPage404, errorPage500, errorPageEx);
    }
}
```
 + 해당 예외와 자식 타입의 오류까지 함께 처리
 + 해당 예외 발생시, **오류를 처리 가능한 컨트롤러를 호출**
   + `RuntimeException` -> `/error-page/500` 이 호출됌
 
> ErrorPageController: 오류 처리 컨트롤러
```java

@Controller
public class ErrorPageController {

    @RequestMapping("/error-page/404")
    public String errorPage404(HttpServletRequest request, HttpServletResponse response){
        return "error-page/404";
    }

    @RequestMapping("/error-page/500")
    public String errorPage500(HttpServletRequest request, HttpServletResponse response){
        return "error-page/500";
    }
}
```

### ✔️ 오류 페이지 작동 원리
```
  1) WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
  2) WAS `/error-page/500` 다시 요청 -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러(/errorpage/500) -> View
```
 + 예외가 발생하면 WAS는 오류 페이지 경로를 찾아서, 내부 오류 페이지를 호출
   +  이때 오류 페이지 경로로 `Filter`, `Servlet`, `Interceptor`,`Controller` 모두 다시 호출됌!
 + :pencil2: **오직 서버 내부에서의 추가적인 호출일 뿐, 클라이언트는 알지 X**
 
### ✔️ 오류 정보 추가
 + WAS는 오류 페이지 요청뿐만 아니라, 오류 정보를 `request`의 `attribute`에 추가해서 넘겨줌
   + 필요하면, 전달된 오류 정보를 사용 가능
```
//RequestDispatcher 상수로 정의되어 있음
 public static final String ERROR_EXCEPTION ="javax.servlet.error.exception";
 log.info("ERROR_EXCEPTION: ex=", request.getAttribute(ERROR_EXCEPTION))
```
