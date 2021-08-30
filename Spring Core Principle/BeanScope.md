# 빈 스코프
 + 빈이 존재할 수 있는 범위, 스프링 빈은 기본적으로 싱글톤 스코프로 생성
 
> Component scan 자동 등록
```java
  @Scope("prototype")
  @Component
  public class HelloBean {}
```

> 수동 등록
```java
  @Scope("prototype")
  @Bean
  PrototypeBean HelloBean() {
     return new HelloBean();
  }
```
## 빈 스코프와 종류

### :pushpin: 스코프의 종류
+ `싱글톤` : 기본 스코프로써, 스프링 컨테이너와 같은 생명주기를 가지는 스코프(시작-종료)
+ `프로토타입` : 스프링 컨테이너가 빈의 생성과 의존관계 주입까지만 관여(초기화 메소드까지는 불러줌)하는 매우 짧은 범위의 스코프
+ `웹 관련 스코프`
  + `request` : 웹 요청이 들어오고 나갈때까지 유지되는 스코프
  + `session` : 웹 세션이 생성되고 종료될 때까지 유지되는 스코프
  + `application` : 웹의 서블릿 컨텍스와 같은 범위로 유지되는 스코프  

## 프로토타입 스코프
 + 조회 시 스프링 컨테이너는 항상 새로운 인스턴스를 생성해서 반환
 > ☑️ 싱글톤 스코프 : 컨테이너가 같은 인스턴스(주소가 같음)의 스프링 빈 반환
 
### :pushpin: 빈 요청 과정

1. 프로토타입 스코프의 빈을 스프링 컨테이너에 요청</br>
2. :star2: 요청 시 컨테이너가 프로토타입 빈 생성, 의존관계 주입(DI), 초기화 까지 처리
  + 따라서, **@PreDestory** 와 같은 **종료 메소드 호출 불가능**
  + 빈을 조회한 클라이언트가 프로토타입 빈을 관리해야함
3. 생성한 빈을 클라이언트에 반환하고 관리하지 X, 이후 같은 요청이 올때마다 새로운 프로토타입 빈 생성해서 반환

<img src="https://user-images.githubusercontent.com/71436576/131356036-8fcdcd8e-9b77-440e-860d-f90d5dd4163f.png" width=50% height=50%>


> 프로토타입 스코프의 빈 테스트
  ```java
  public class PrototypeTest {

      @Test
      void prototypeBeanFind(){
          AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);
          System.out.println("find prototypeBean1");
          PrototypeBean bean1 = ac.getBean(PrototypeBean.class);
          System.out.println("find prototypeBean2");
          PrototypeBean bean2 = ac.getBean(PrototypeBean.class);

          System.out.println("bean1 = " + bean1);
          System.out.println("bean2 = " + bean2);
          assertThat(bean1).isNotSameAs(bean2);

          //bean1.close();
          ac.close();
      }

      @Scope("prototype")
      static class PrototypeBean{

          @PostConstruct
          public void init(){
              System.out.println("PrototypeBean.init");
          }

          @PreDestroy
          public void destroy(){
              System.out.println("PrototypeBean.destroy");
          }
      }
  }
  ```
 > 실행 결과
  ```
  find prototypeBean1
  PrototypeBean.init
  find prototypeBean2
  PrototypeBean.init
  bean1 = hello.core.scope.PrototypeTest$PrototypeBean@4671115f   //서로 다른 스프링 빈 생성 확인(초기화도 2번)
  bean2 = hello.core.scope.PrototypeTest$PrototypeBean@36cda2c2
  ```
  + 스프링 컨테이너에서 **빈을 조회 할때 생성**되고, 초기화 메서드 실행(컨테이너 생성 시점에 초기화 X)
  + 종료 메소드가 실행되지 않는 것을 볼 수 있다!
    
 ### :pushpin: 싱글톤 빈과 함께 사용시 문제점
 
 + 스프링의 기본은 싱글톤 빈이므로, 프로토타입을 사용하는 경우가 생김
 + 다만 싱글톤 빈은 생성 시점에만 의존관계 주입을 받기 때문에, **프로토타입이 요청 시마다 새로 생성되는 것이 아니라 주입 시에만 한번 생성되어 같은 빈이 유지됌**
 ```java
    @Scope("singleton")    //싱글톤
    static class ClientBean{
        private final PrototypeBean prototypeBean;    //생성 시점에 이미 주입(계속 같은 것)

        //요청하는 시점에 prototypeBean 생성해서 주입
        @Autowired
        public ClientBean(PrototypeBean prototypeBean) {
            this.prototypeBean = prototypeBean;
        }

        public int logic(){
            prototypeBean.addCount();
            return prototypeBean.getCount();
        }
    }
    
    @Scope("prototype")
    static class PrototypeBean {
         private int count = 0;
         
         public void addCount() {
             count++;
         }
         
         public int getCount() {
             return count;
         }
         
         @PostConstruct
         public void init() {
             System.out.println("PrototypeBean.init " + this);
         }
         
         @PreDestroy
         public void destroy() {
             System.out.println("PrototypeBean.destroy");
         }
     }
 ```
  + `Client Bean` : 싱글톤이기 때문에 주입 된 `PrototypeBean`은 이미 과거(생성 시점)에 주입이 끝난 빈
    + 서로 다른 클라이언트가 요청해도, 같은 프로토타입 빈이 반환됌
  + :bulb: 어떻게 사용할 때 마다 항상 새로운 프로토타입 빈을 생성할 수 있을까?

### :pushpin: Solution: Provider
 
#### ✔️ 스프링 컨테이너에 요청
 ```java
 @Autowired
  private ApplicationContext ac;
  
  public int logic() {
     PrototypeBean prototypeBean = ac.getBean(PrototypeBean.class);  //항상 새로운 프로토타입 빈 생성
     prototypeBean.addCount();
     int count = prototypeBean.getCount();
     return count;
 }
 ``` 
  + `DL / 의존관계 조회` : 외부에서 주입 받는 것이 아니라 직접 필요한 의존관계를 찾는 것
    + 지정한 빈을 컨테이너에서 찾기
  + 하지만, 스프링의 애플리케이션 컨텍스트 전체를 주입받게 되면 스프링 컨테이너에 종속적인 코드가 되고, 단위 테스트도 어려워짐
  
#### ✔️ ObjectProvider
 + `ObjectProvider` : 스프링 컨테이너에서 필요한 빈을 대신 조회하는 DL 서비스를 제공, 스프링에 의존적
 
 ```java
   @Autowired
   private ObjectProvider<PrototypeBean> prototypeBeanProvider;

   public int logic() {
      PrototypeBean prototypeBean = prototypeBeanProvider.getObject();  //컨테이너에서 해당 빈 찾아서 반환
      prototypeBean.addCount();
      int count = prototypeBean.getCount();
      return count;
   }
 ```
  + `getObject()`: 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환한다(DL)
 
#### ✔️ JSR303 Provider
 ```java 
  @Scope("singleton")
  static class ClientBean{
   
       // implementation 'javax.inject:javax.inject:1' gradle 에 추가 필수
        @Autowired
        private Provider<PrototypeBean> prototypeBeanProvider;   //항상 새로운 프로토타입 빈 생성

        public int logic(){
            PrototypeBean prototypeBean = prototypeBeanProvider.get(); 
            prototypeBean.addCount();
            return prototypeBean.getCount();
        }
    }
 ```
  + `get()` : 스프링 컨테이너 통해 해당 빈 찾아서 반환(DL)

> 스프링 VS 자바 표준(JAVAX) : 둘의 기능이 겹칠 때는 그냥 스프링 제공 기능 사용하면 됌!
 
## 웹 스코프
  + 웹 환경에서만 동작하며, 스프링이 스코프의 종료시점까지 관리(종료 메소드 호출 가능)
    + `request` : HTTP 웹 요청 하나가 들어오고 나갈때까지 유지되는 스코프, 각각의 요청마다 별도의 빈 인스턴스 생성되고 관리
    + `session` : HTTP Session과 동일 생명주기 가지는 스코프!

    + `application` : 서블릿 컨텍스와 같은 범위로 유지되는 스코프 
    + `websocket` : 웹 소켓과 동일한 생명주기 가지는 스코프

<img src="https://user-images.githubusercontent.com/71436576/131358236-4fe7bfde-dab1-4489-80e4-c5fdd3304952.png" width=50% height=50%>

### :pushpin: 웹 환경 구축
 ```java
 //web 라이브러리 추가
 implementation 'org.springframework.boot:spring-boot-starter-web'
 ```
  + `spring-boot-starter-web` : 스프링 부트는 내장 톰켓 서버 활용하여 웹서버와 스프링 함께 실행
  + 웹 라이브러리 추가 시`AnnotationConfigServletWebServerApplicationContext`를 기반으로 애플리케이션을 구동함

### :pushpin: Request 스코프
 + 동시에 여러 HTTP 요청이 올때, 정확히 어떤 요청이 남긴 로그인지 구분 가능
 + 기대하는 공통 포멧: [UUID][requestURL] {message}
   + `UUID` : HTTP 요청을 구분
   + `requestURL` : 어떤 URL을 요청해서 남은 로그인지 확인

> 로그 출력 위한 클래스
 ```java
 @Component
 @Scope(value = "request")     //http 요청 당 하나씩 생성, 요청 끝나는 시점에 소멸
 public class MyLogger {

     private String uuid;
     private String requestURL;

     //빈 생성 시점에 알 수 없어서 외부에서 setter 로 입력
     public void setRequestURL(String requestURL) {
         this.requestURL = requestURL;
     }

     public void log(String message){
         System.out.println("[" + uuid + "]" + "[" + requestURL + "]" + message);

     }

     @PostConstruct
     public void init(){
         uuid = UUID.randomUUID().toString();
         System.out.println("[" + uuid + "] request scope bean create:" + this);
     }

     @PreDestroy
     public void close(){
         System.out.println("");
         System.out.println("[" + uuid + "] request scope bean close:" + this);
     }
 }
 ```
   + 빈 생성 시점에, 초기화 메소드 사용하여 자동으로 uuid 생성해 저장 => 다른 HTTP 요청과 구분 가능
   
> 로거 작동 확인 테스트용 컨트롤러
 ```java
 @Controller
 @RequiredArgsConstructor
 public class LogDemoController {

     private final LogDemoService logDemoService;
     private final MyLogger myLogger;

     @RequestMapping("log-demo")
     @ResponseBody
     public String logDemo(HttpServletRequest request){
         String requestURL = request.getRequestURI().toString();  //받은 URL값을 myLogger에 저장
         myLogger.setRequestURL(requestURL);     //다른 http 요청과 구분 가능(각각의 요청)

         myLogger.log("controller test");
         logDemoService.logic("testId");
         return "ok";
     }
 }
 ```
 
 > 비즈니스 로직
 ```java
 @Service
 @RequiredArgsConstructor
 public class LogDemoService {

     //웹 관련 정보가 서비스 계층까지 넘어오면 안됌
     private final MyLogger myLogger;

     public void logic(String testId) {
         myLogger.log("service id = " + testId);
     }
 }
 ```
  + `MyLogger` : request scope이기 때문에, requestURL과 같은 웹 관련 정보를 서비스 계층에서 파라미터로 넘기지 않고 MyLogeer 멤버 변수에 저장하여 코드 유지 가능
  + :fire: 웹과 관련된 부분은 컨트롤러까지만 사용, **서비스 계층은 웹 기술에 종속되면 안됌!**

> 컨테이너에게 빈을 요청하는 단계는 의존관계 주입 단계가 아니라 뒤로 지연시켜야 오류 발생 X

### :pushpin: Request scope와 Provider
 + `ObjectProvider`: ObjectProvider.getObject() 를 호출하는 시점까지 request scope 빈의 생성을 지연 가능
 > LogDemoController 중 일부
 ```java
   @RequestMapping("log-demo")
   @ResponseBody
   public String logDemo(HttpServletRequest request){
        String requestURL = request.getRequestURI().toString();
        MyLogger myLogger = myLoggerProvider.getObject();  //필요한 시점에 처음 생성되고 init()실행됌
        myLogger.setRequestURL(requestURL);
        ...
    }
 ```
 > LogDemoService
 ```java
 @Service
 @RequiredArgsConstructor
 public class LogDemoService {

     private final ObjectProvider<MyLogger> myLoggerProvider;

     public void logic(String testId) {
         MyLogger myLogger = myLoggerProvider.getObject();
         myLogger.log("service id = " + testId);
     }
 }
 ```
### :pushpin: 스코프와 프록시

 ```java
 @Component
 @Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS) 
 public class MyLogger {

 }
 ```
   + `proxyMode` : 스프링 컨테이너는 CGLIB로 MyLogger의 **가짜 프록시 객체**를 생성 후 등록, http 요청과 관게 없이 다른 빈에 미리 주입 가능
   + 요청이 오면 가짜 프록시 객체가 실제 myLogger를 호출하고, 싱글톤 처럼 동작
   > :star2: 진짜 객체 조회를 꼭 필요한 시점 까지 지연처리가 가능! </br>
   > 다형성과 DI 컨테이너가 가진 큰 장점 => 단지 애노테이션 설정 변경만으로 원본 객체를 프록시로 대체 가능
 
