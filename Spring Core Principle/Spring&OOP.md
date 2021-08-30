# Spring
 + 필수 : `스프링 프레임워크`, 스프링 부트
 + 선택 : 스프링 데이터(CRUD), 세션, 시큐리티,Rest Docs(Api), 배치, 클라우드,,etc
 + 핵심 : 순수한 객체 지향 언어의 강점을 살리는 프레임워크, 나머지는 객체 지향을 위한 도구들
 
## 스프링 프레임워크
+ 핵심 기술 : 스프링 DI 컨테이너, AOP, 이벤트,,etc
+ DI, DI 컨테이너 기술로 다형성 + OCP,DIP를 지원 =>클라이언트 코드 변경 없이 기능 확장 가능

 > :pencil2: 프레임워크 vs 라이브러리</br>
 + `프레임워크(FrameWork)` : 프로그램 개발을 위한 여러 요소와 메뉴얼(규약)을 제공하는 프로그램이며, 코드를 제어하고 대신 실행함(ex) JUnit)</br>   
 + `라이브러리(Library)` : 개발 시 선택적으로 사용 가능한 기능을 제공하는 도구들이며, 작성 코드가 직접 제어의 흐름을 담당</br>
 
## 스프링 부트
 + 기본적으로 spring framework 사용, 스프링을 편리하게 사용 가능하도록 지원
 + Tomcat 같은 웹 서버를 스프링에서 내장하여 쉽게 애플리케이션 생성
 + 선쉬운 빌드 구성을 위한 starter 종속성 제공 등등..

## OOP(객체 지향 프로그래밍)

 + `다형성` : 객체 설계 시 역할(인터페이스)과 구현(역할 수행 객체)으로 분리
   + 클라이언트는 대상의 인터페이스만 알고, 내부 구조와 구현 대상의 변경에 영향 X
   + Overriding
   ```java
   public class MemberService {
      //private MemberRepository memberRepository = new MemoryMemberRepository();
      private MemberRepository memberRepository = new JdbcMemberRepository();
   }
   ```
   + 클라이언트 변경하지 않고, 서버의 구현 기능을 유연하게 변경 가능 => 무한 확장
   + 스프링의 IoC, DI는 다형성을 쉽게 사용하도록 한다 
 + 객체 클라이언트와 객체 서버의 협력 관계로 이루어진다
 

### :pushpin: SOLID 원칙
 + `SRP(단일 책임 원칙)` : 한 클래스는 하나의 책임(변경 이유)만을 가져야함
 + `OCP(개방-폐쇄 원칙)` : 확장에는 열려있으나, 기존 코드의 변경에는 닫혀 있어야함
   + 다형성을 활용하여, 인터페이스를 구현한 새로운 클래스 만들어 확장
   ```java
   public class MemberService {
      //private MemberRepository memberRepository = new MemoryMemberRepository();
      private MemberRepository memberRepository = new JdbcMemberRepository();
   }
   ``` 
     + :thumbsdown: 클라이언트 구현 클래스 직접 선택함으로써 OCP와 DIP가 지켜지지 X
     + `Spring Container` : 객체 생서의 연관관계를 맺어주는 별도의 조립, 설정자
 + `LSP(리스코프 치환 원칙)` : 객체는 언제나 하위 타입의 인스터스로 변경 가능해야함
 + `ISP(인터페이스 분리 원칙)` : 인터페이스의 단일 책임, 특정 클라이언트 위한 여러개의 인터페이스로 분리
 + `DIP(의존관계 역전 원칙)` : 클라이언트 코드는 구체화가 아닌 역할인 인터페이스에만 의존해야함 => 변경 용이
   + MemberRepository m = new JdbcMemberRepository() : 인터페이스와 구현 클래스 동시 의존(DIP 위반)
   
> 다형성 만으로는 OCP와 DIP를 지킬 수가 없다!
> EX) OrderServiceImpl.java 중 일부
```java
public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository = new MemoryMemberRepository();
    //private final DiscountPolicy discountPolicy = new FixDiscountPolicy(); 
    private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
}
```
 + 위 코드는 인터페이스와 구현 클래스 모두에 의존하고 있음(DIP 위반)
 + 의존하고 있기 때문에 기능 확장을 하게 되면 클라이언트 코드인 OrderServiceImpl에 영향을 줌(OCP 위반)
 + 해결 : 관심사의 분리를 통해!

## 관심사의 분리
 + 객체 생성과 연결하는 역할(구성)+ 실행하는 역할(사용)
 
 > OrderServiceImpla.java중 일부
```java
public class OrderServiceImpl implements  OrderService{

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;     //인터페이스(추상화)에만 의존

    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
```
 > AppConfig.java
 ```java
 public class AppConfig {

    //생성자 주입, 리팩토링
    public MemberService memberService(){
        return new MemberServiceImpl(memberRepository());
    }

    private MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    public OrderService orderService(){
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    public DiscountPolicy discountPolicy(){
        return new FixDiscountPolicy();
    }
}
 ```
  + `AppConfig` : new의 사용 대신 구현 객체를 생성하고, 인스턴스의 참조를 생성자 통해서 주입(DI)
    +  OrderServiceImpl의 생성자 통해서 어떤 구현 객체가 주입되는지는 외부에서 결정
    +  의존관계에 대한 고민은 외부인 AppConfig에 맡기고 역할 실행에만 집중!
    +  이후 어떠한 변경 사유에 대해서도 AppConfig파일인 구성영역만 변경하면 됌
    ```java
    public class AppConfig {
       ...
       public DiscountPolicy discountPolicy(){
           //return new FixDiscountPolicy();
           return new RateDiscountPolicy();
       }
   }
   ``
> OrderServiceTest.java중 일부
 ```java
public class OrderServiceTest {
    MemberService memberService;
    OrderService orderService;

    @BeforeEach
    public void beforeEach(){
        AppConfig appConfig = new AppConfig();
        memberService = appConfig.memberService();
        orderService = appConfig.orderService();
    }
}
```

## IoC
 + `Inversion of Control` : 프로그램의 제어 흐름을 직접이 아닌 외부에서 관리하는것, 즉 **제어가 역전**되는것
   + 기존의 프로그램은 클라이언트 구현 객체가 스스로 필요한 서버 구현 객체를 생성,연결,실행
   + **Appconfig와 같이 외부에서 제어 흐름을 결정하고**, 구현 객체는 실행의 역할만 담당하게 됌
   + EX) JUnit과 같은 `FrameWork` 도 IoC 

## DI
 + `Dependency Injection` : **외부에서 객체들의 의존 관계를 주입하는 것**
 + `의존관계` : 정적 클래스 의존관계 + 실행 시점에 결정되는 동적 객체 의존 관계
 
 ### :pushpin: 정적인 클래스 의존관계
   + 실행하지 않아도 코드 분석(import)를 통해 클래스 간 의존관계를 분석 가능, 클래스 다이어그램
   + 하지만 어떤 객체가 주입될지는 분석 불가능

 ### :pushpin: 동적인 객체 인스턴스 의존 관계
   + 실행 시점(런타임)에 실제 생성된 객체 인스턴스의 참조가 연결된 의존 관계, 객체 다이어그램
   + `의존관계 주입`을 통해 클라이언트의 호출 대상의 인스턴스 변경 가능
   + `의존관계 주입`을 통해 정적 클래스 의존관계의 변경 없이 동적 객체 의존관계만 변경 가능

## DI(IoC) Container
 + `DI 컨테이너` : AppConfig와 같이 객체를 생성하고 관리 + 의존관계 연결(주입)해주는 것
 +  `Spring Container` : 빈 관리와 조회 담당의 'BeanFactory' 최상위 인터페이스 + 부가 인터페이스(기능)들 상속
 ```java
 @Configuration
public class AppConfig {
    @Bean
    public MemberService memberService(){
        return new MemberServiceImpl(memberRepository());
    }
    ...
} 
 ```
  + @Bean이 붙은 메소드 명 = 스프링 빈의 이름
 ```java
 public class MemberApp {
    public static void main(String[] args) {

        //AppConfig appConfig = new AppConfig();
        //MemberService memberService = appConfig.memberService();
        //MemberService memberService = new MemberServiceImpl();
        
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService = applicationContext.getBean("memberService", MemberService.class); 
 ```
   + `ApplicationContext` : 스프링 컨테이너로써, AppConfig의 설정(구성)정보를 가지고 @Bean이 적힌 메소드들을 호출에 반환된 객체를 컨테이너에 등록(스프링 빈)
   + 컨테이너에서 직접 필요한 스프링 빈을 찾아서 사용하도록 변경
 
