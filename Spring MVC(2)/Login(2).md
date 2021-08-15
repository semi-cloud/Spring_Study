# Login(2) : Filter, Interceptor
 + 웹과 관련된 **공통 관심사**를 처리할때, HTTP Header나 URL 정보들이 필요
   + `Servlet Filter` , `Spring Interceptor` :  HttpServletRequest를 제공
## Filter 개요

### Filter 흐름
 + `HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러`
   + :pencil2: **필터가 호출된 다음에 서블릿이 호출됌** (Spring : Dispatcher Servlet)
   + 로그인이 안되어 있다면, 필터에서 걸러져서 서블릿이 호출되지 X
   
### Filter 인터페이스
```java
public interface Filter {
   public default void init(FilterConfig filterConfig) throws ServletException {}
   public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException;
   public default void destroy() {}
}
```
 + 서블릿 컨테이너가 필터를 싱글톤 객체로 생성하고, 관리
 + `init()` : 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출
 + :star2: `doFilter()`: 고객의 요청이 올 때 마다 해당 메서드가 호출(필터 로직 구현)
   + **다음 필터가 있으면 필터를 호출하고, 필터가 없으면 서블릿을 호출**
   + 만약 이 로직을 호출하지 않으면 다음 단계로 진행되지 않음!
 + `destroy()`: 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출

> WebConfig - 필터 설정
```java
@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean logFilter(){
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(new LogFilter());
        filterRegistrationBean.setOrder(1);  
        filterRegistrationBean.addUrlPatterns("/*");

        return filterRegistrationBean;
    }
}
```
 + Spring Boot :  FilterRegistrationBean 을 사용해서 등록하면 됌
 + `setOrder(1)` : 필터는 체인으로 동작하기 때문에 순서가 필요, 낮을 수록 먼저 동작
 + `addUrlPatterns("/*")` : 필터를 적용할 URL 패턴을 지정
 
## Filter 구현
 + 모든 컨트롤러에 로그인 인증 여부 코드를 작성하는 것은 비효율적
   + 로그인 하지 않은 사용자는, 상품 관리에 접근 불가능! 
   
> LoginCheckFilter - 인증 체크 필터
```java
@Slf4j
public class LoginCheckFilter implements Filter {

    private static final String[] whitelist = {"/", "/members/add", "login", "/logout", "/css/*"};

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String requestURI = httpRequest.getRequestURI();

        HttpServletResponse httpResponse = (HttpServletResponse) response;

        try{
            log.info("인증 체크 필터 시작{}", requestURI);

            if(isLoginCheckPath(requestURI)){
                log.info("인증 체크 로직 실행{}", requestURI);
                HttpSession session = httpRequest.getSession(false);
                if(session == null || session.getAttribute(SessionConst.LOGIN_MEMBER) == null{
                    log.info("미인증 사용자 요청 {}", requestURI );
                    //로그인으로 redirect
                    httpResponse.sendRedirect("/login?redirectURL=" + requestURI);
                    return;      //여기서 끝내야함(중요)
                }
            }
            chain.doFilter(request,response);
        }catch(Exception e){
            throw e;  
        }finally{
            log.info("인증 체크 필터 종료 {}", requestURI);
        }
    }

    //화이트 리스트의 경우 인증 체크 X
    private boolean isLoginCheckPath(String requestURI){
        return !PatternMatchUtils.simpleMatch(whitelist, requestURI);
    }
}
```
 + `whitelist` : 인증과 무관하게 항상 허용하는 경로들
   + whitelist 제외한 나머지 모든 경로에 인증 체크 로직 적용 필요
 + `httpResponse.sendRedirect("/login?redirectURL=" + requestURI)` : 현재 요청한 경로인 requestURI 를 /login 에 쿼리 파라미터로 함께 전달
 + `return` : 이후 필터는 물론 서블릿, 컨트롤러의 호출을 막음
   + redirect로 응답까지만 해주고, 요청이 끝나 이후 비즈니스 로직 동작에는 영향을 주지 않도록 해야함

> LoginControllerV4
```java
@PostMapping("/login")
public String loginV4(...@RequestParam(defaultValue = "/") String redirectURL..){

   //로그인 성공 시
   return "redirect:" + redirectURL;   //없으면 홈(/)으로, 있으면 기존 화면으로
```

## Spring 인터셉터
 + `Interceptor` : **스프링 MVC 구조에 특화된 필터 기능**을 제공

### Spring 인터셉터 호출 흐름
```
 HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러
 ex)
 HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터(적절하지 않은 요청이라 판단, 컨트롤러 호출
 X) // 비 로그인 사용자
```
 + 서블릿 :  `DispatcherServlet`
 + 인터셉터는 디스패처 서블릿에서 시작해, 컨트롤러 호출 직전에 호출됌
 + 스프링 인터셉터는 chain 형식

### Spring 인터셉터 Interface
```java
public interface HandlerInterceptor {
  //1.preHandle
  default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {}

  //2.postHandle
  default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {}

  //3.afterCompletion
  default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) t  hrowsException {}
}

```
<img src="https://user-images.githubusercontent.com/71436576/129467415-7ac52d92-c2f1-42fe-9f02-a8e29a7cce2f.png" width=50% height=50%>

#### ✔️ 정상 흐름
 + `preHandle` : **컨트롤러 호출 전**에 호출됌 (핸들러 어댑터 호출 전)
   +  응답값이 `true` 이면 다음으로 진행하고, `false` 이면 더이상 진행하지 않아 나머지 인터셉터나 핸들러 어댑터도 호출 X 
 + `postHandle` : **컨트롤러 호출 후**에 호출됌(핸들러 어댑터 호출 후)
 + `afterCompletion` : **뷰가 렌더링 된 이후**에 호출됌

#### ✔️ 예외 흐름
 + `preHandle` : 컨트롤러 호출 전에 호출
 + `postHandle` : 컨트롤러에서 예외가 발생하면,  postHandle 은 호출되지 X
 + `afterCompletion` : :star2: afterCompletion 은 예외와 무관하게 항상 호출됌 => 포함된 예외 정보를 로그로 출력 가능
   + 예외와 무관하게 공통 처리를 해야할시, `afterCompletion` 사용
   
## 인터셉터 구현

> LogInterceptor.java
```java
@Slf4j
public class LogInterceptor implements HandlerInterceptor {
    public static final String LOG_ID = "logId";
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();
        String uuid = UUID.randomUUID().toString();   //요청 로그 구분

        request.setAttribute(LOG_ID, uuid);  

        if(handler instanceof HandlerMethod){
            HandlerMethod hm = (HandlerMethod) handler;  //호출할 컨트롤러의 모든 정보
        }
        log.info("REQUEST [{}][{}][{}]", uuid, requestURI, handler); 
        return true;  //다음 인터셉터, 컨트롤러 호출됌
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle [{}]", modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        String requestURI = request.getRequestURI();
        Object logId = request.getAttribute(LOG_ID);
        log.info("RESPONSE [{}][{}][{}]", logId, requestURI, handler);
        if (ex != null) {
            log.error("afterCompletion error", ex);
        }
    }
}
```
 + ` request.setAttribute(LOG_ID, uuid)` : 인터셉터는 호출 시점이 완전히 분리되기 때문에, request에 담아두고 사용
   + `서블릿 필터`: 지역변수로 해결이 가능
 + `HandlerMethod`: `@Controller` `@RequestMapping` 을 활용한 핸들러 매핑의 경우 넘어오는 핸들러 정보(컨트롤러)
 + `ResourceHttpRequestHandler` : /resources/static 와 같은 정적 리소스가 호출 되는 경우 넘어오는 정보
 
 > WEBCONFIG - 인터셉터 등록
 ```java
 @Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**")  //하위 전부 다
                .excludePathPatterns("/css/**", "/*.ico", "/error");
    }
 }
 ```
  + `addPathPatterns("/**")` : 인터셉터를 적용할 URL 패턴을 지정

> LoginCheckInterceptor.java
```java
@Slf4j
public class LoginCheckInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();
        log.info("인증 체크 인터셉터 실행 {}", requestURI);

        HttpSession session = request.getSession();

        if(session == null || session.getAttribute(SessionConst.LOGIN_MEMBER) == null){
            log.info("미인 증 사용자 요청");
            //로그인으로 redirect
            response.sendRedirect("/login?redirectURL=" + requestURI);
        }
        return true;
    }
}
```
 + 인증은, 컨트롤러 호출 전에만 필요하므로 `preHandle`만 구현하면 됌
 
> WebConfig.java - 로그인 인증 인터셉터 추가
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

        registry.addInterceptor(new LoginCheckInterceptor())
                .order(2)
                .addPathPatterns("/**")   //세밀하게 적용 가능
                .excludePathPatterns("/", "/members/add", "/login", "/logout",
                        "/css/**", "/*.ico", "/error");
}
```
 + interceptor에서 직접 제외할 경로를 정하는 것이 아닌, Interceptor를 '설정'할때 설정 정보로 넘겨주어 편리
   + `excludePathPatterns` : 인터셉터를 사용 하지 않을 부분
  
## ArgumentResolver 활용
 + 로그인 회원을 더 간편하게 찾을 수 있게 도와주는 기능
 
> Annotation 등록
```java
  @Target(ElementType.PARAMETER)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface Login {

  }
```

> LoginMemberArgumentResolver.java
```java
 @Slf4j
 public class LoginMemberArgumentResolver implements HandlerMethodArgumentResolver {
     @Override
     public boolean supportsParameter(MethodParameter parameter) {

         boolean hasLoginAnnotation = parameter.hasParameterAnnotation(Login.class);
         boolean hasMemberType = Member.class.isAssignableFrom(parameter.getParameterType());

         return hasLoginAnnotation && hasMemberType;  //true면 아래 메소드 실행
     }
     
     @Override
     public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {

         HttpServletRequest request = (HttpServletRequest) webRequest.getNativeRequest();
         HttpSession session = request.getSession(false);
         if(session == null){
             return null;
         }
         return session.getAttribute(SessionConst.LOGIN_MEMBER); //세션 존재 시 멤버 반환

     }
 }
```
