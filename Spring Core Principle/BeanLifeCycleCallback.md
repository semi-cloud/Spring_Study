# 빈 생명주기 콜백

## 빈 생명주기 콜백
 + 애플리케이션 시작 시점에 필요한 커넥션(DB 커넥션 풀, 네트워크 소켓)을 미리 생성하거나, 종료 시점에 커넥션을 종료하는 작업을 진행해야함
   + 객체의 초기화와 종료 작업이 필요!

### :bulb: 스프링 빈의 이벤트 라이프사이클
스프링 컨테이너 생성-> 스프링 빈 생성-> 의존관계 주입-> 초기화 콜백-> 사용-> 소멸 전 콜백-> 스프링 종료
 + `초기화 콜백` : 빈이 생성 + 의존관계 주입 완료 후 호출
 + `소멸 전 콜백` : 빈이 소멸되기 직전에 호출
 
 > 객체의 생성과 초기화를 분리하지 않은 예
 ```java
 public class NetworkClient {
     private String url; 
     
     public NetworkClient() {
        System.out.println("생성자 호출, url = " + url);   
        connect();             //초기화 과정 => 객체 생성과 분리 필요!
        call("초기화 연결 메시지");
     }
 }
 ```
   + `객체 생성자` : 필수 정보(파라미터) 받기, 메모리 할당하여 객체 생성, 간단하게 내부 값들 변경 동작
   + `객체 초기화` : 생성값 활용하여 외부 커넥션을 연결, 동작
   + :star: **객체의 생성과 초기화를 분리하자!**

## 빈 생명주기 콜백 방법

### :pushpin: 인터페이스 InitializingBean, Disposable Bean
  + 현재 거의 사용하지 않는 방법.
  ```java
  public class NetworkClient implements InitializingBean, DisposableBean {
    ... 
    //의존관계 주입이 끝나면
    @Override
    public void afterPropertiesSet() throws Exception {
        connect();
        call("초기화 메시지");
    }

    //빈 소멸 전에
    @Override
    public void destroy() throws Exception {
        disconnect();
    }
}
 ```
### :pushpin: 빈 등록 초기화,소멸 메소드
+ 설정 정보에 @Bean(initMethod = "init", destroyMethod = "close") 처럼 초기화, 소멸 메서드를 지정
 ```java
 public class NetworkClient{
  ...
  //의존관계 주입이 끝나면
  public void init() {
        System.out.println("NetworkClient.init");
        connect();
        call("초기화 메시지");
    }

    //빈 소멸 전에
    public void close() {
        System.out.println("NetworkClient.close");
        disconnect();
    } 
 ```
 >BeanLifeCycleTest.java중 일부
 ```java
 @Configuration
    static class LifeCycleConfig {

        @Bean(initMethod = "init", destroyMethod = "close")
        public NetworkClient networkClient(){
            NetworkClient networkClient = new NetworkClient();  //url : null 값
            networkClient.setUrl("http://hello-spring.dev"); 
            return networkClient;
        }
    }
 ```
  + 코드를 고칠 수 없는 외부 라이브러리에도 초기화, 종료 메서드를 적용 가능
  + `destoryMethod` : 추론 기능이 있어서 외부 라이브러리의 종료 메소드인 close, shutdown 자동 호출

### :pushpin: @PostConstruct, @PreDestory
 ```java
  @PostConstruct
  public class NetworkClient{
  
    @PostConstruct
    public void init() throws Exception {
        System.out.println("NetworkClient.init");
        connect();
        call("초기화 메시지");
    }

    @PreDestroy
    public void close() throws Exception {
        System.out.println("NetworkClient.close");
        disconnect();
    }
 }
 ```
  + 스프링에 종속적이지 않아서(javax) 다른 컨테이너에서도 동작하며, 컴포넌트 스캔과 잘 어울림
  + 외부 라이브러리에는 적용 불가능(코드를 변경해야하기 때문에)
