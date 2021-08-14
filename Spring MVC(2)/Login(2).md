# Login(2) : Filter, Intercepter
 + 웹과 관련된 공통 관심사를 처리할때, HTTP Header나 URL 정보들이 필요
   + `Servlet Filter` , `Spring Intercepter` :  HttpServletRequest를 제공
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
