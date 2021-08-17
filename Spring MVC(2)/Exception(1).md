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

## Servlet Exception: Filter
 + 예외 처리 흐름에서, 필터가 두번 호출될 수 있는 문제가 발생
   + 클라이언트로 부터 발생한 정상 요청인지, 오류 페이지를 출력하기 위한 내부 요청인지 구분 가능해야함
   + Servlet은 문제 해결 위해 `DispathcerType`추가 정보 제공

### ✔️ DispatcherType
 + `DispatcherType` : **고객**이 요청한 것인지, **서버 내부**에서 오류 페이지를 요청하는 것인지 구분
 
> DispatcherType
+ `REQUEST` : 클라이언트 요청
+ `ERROR` : 오류 요청
+ `FORWARD`: MVC에서 배웠던 서블릿에서 다른 서블릿이나 JSP를 호출할 때
  + RequestDispatcher.forward(request, response);
+ `INCLUDE` : 서블릿에서 다른 서블릿이나 JSP의 결과를 포함할 때
  + RequestDispatcher.include(request, response);
+ `ASYNC` : 서블릿 비동기 호출

> WebConfig
```java
filterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.ERROR);
```
 + `setDispatcherTypes` : 클라이언트 요청과 오류 페이지 요청 모두에서 필터가 호출됌
   + 기본값은 `DispatcherType.REQUEST`

## Spring Exception: Interceptor
 + Spring 제공 기능이므로, `DispatcherType`과 무관하게 항상 호출됌
   + 대신, `excludePathPatterns` 사용해서 오류 페이지 경로를 빼기 가능

> WebConfig
```java
  @Override
  public void addInterceptors(InterceptorRegistry registry) {
      registry.addInterceptor(new LogInterceptor())
      .order(1)
      .addPathPatterns("/**")
      .excludePathPatterns(
      "/css/**", "/*.ico"
      , "/error", "/error-page/**"    //오류 페이지 경로
      );
  }
 ```

### 🌺 전체 흐름 정리
 
> /hello 정상 요청
 + `WAS(/hello, dispatchType=REQUEST) -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러 -> View`
 
> /error-ex 오류 요청
 + 필터: DispatchType 으로 중복 호출 제거 ( dispatchType=REQUEST )
 + 인터셉터: 경로 정보로 중복 호출 제거( excludePathPatterns("/error-page/**") )
```
1. WAS(/error-ex, dispatchType=REQUEST) -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러
2. WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
3. WAS 오류 페이지 확인
4. WAS(/error-page/500, dispatchType=ERROR) -> 필터(x) -> 서블릿 -> 인터셉터(x) -> 
컨트롤러(/error-page/500) -> View
```

## Spring boot: 오류 페이지

### 페이지 등록
 + 이전: `WebServerCustomizer`을 만들어서, 예외 종류에 따라 페이지 추가
 + ✔️ Spring boot: ErrorPage 를 자동으로 등록, 이때 `/error` 라는 경로로 기본 오류 페이지를 설정
   + 서블릿 밖으로 예외가 발생하거나, `response.sendError(...)` 가 호출되면 모든 오류는 `/error` 를
호출
### 페이지 Controller
 + 이전 : 예외 처리용 Controller `ErrorPageController`를 만듬
 + ✔️ Spring boot: `BasicErrorController` 라는 스프링 컨트롤러를 자동으로 등록
   + ErrorPage 에서 등록한 `/error` 를 매핑해서 처리하는 Controller

:star2: **개발자는 뷰 선택 우선순위에 따라 오류 페이지만 등록하면 됌!**
> 뷰 선택 우선순위
```
 1.뷰 템플릿
    resources/templates/error/500.html
    resources/templates/error/5xx.html
 2. 정적 리소스( static , public )
    resources/static/error/400.html
    resources/static/error/404.html
    resources/static/error/4xx.html
 3. 적용 대상이 없을 때 뷰 이름( error )
    resources/templates/error.html
```
 + 항상 구체적인 것이 덜 구체적인 것보다 우선순위가 높음

### BasicErrorController 제공 기본 정보
 + `BasicErrorController` : 오류 관련 정보들을 model에 담아서 뷰에 전달하고, 뷰 템플릿은 이 값을
활용해서 출력가능

```
 * timestamp: Fri Feb 05 00:00:00 KST 2021
 * status: 400
 * error: Bad Request
 * exception: org.springframework.validation.BindException
 * trace: 예외 trace
 * message: Validation failed for object='data'. Error count: 1
 * errors: Errors(BindingResult)
 * path: 클라이언트 요청 경로 (`/hello`)
```
 + 하지만 오류 관련 내부 정보들을 고객에게 노출하는 것은 권장 X
