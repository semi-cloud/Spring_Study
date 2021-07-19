# Spring
 + 필수 : `스프링 프레임워크`, 스프링 부트
 + 선택 : 스프링 데이터(CRUD), 세션, 시큐리티,Rest Docs(Api), 배치, 클라우드,,etc
 + 핵심 : 순수한 객체 지향 언어의 강점을 살리는 프레임워크, 나머지는 객체 지향을 위한 도구들
 
## 스프링 프레임워크
 > :star2: 프레임워크 vs 라이브러리
  > + 1)`프레임워크(FrameWork)` : 프로그램 개발을 위한 여러 요소와 메뉴얼(규약)을 제공하는 프로그램이며, 코드를 제어하고 대신 실행함(ex) JUnit)
      
  > + 2)`라이브러리(Library)` : 개발 시 선택적으로 사용 가능한 기능을 제공하는 도구들이며, 작성 코드가 직접 제어의 흐름을 담당 
 + 핵심 기술 : 스프링 DI 컨테이너, AOP, 이벤트,,etc
 + DI, DI 컨테이너 기술로 다형성 + OCP,DIP를 지원 =>클라이언트 코드 변경 없이 기능 확장 가능

## 스프링 부트
 + 기본적으로 spring framework 사용, 스프링을 편리하게 사용 가능하도록 지원
 + Tomcat 같은 웹 서버를 스프링에서 내장하여 쉽게 애플리케이션 생성
 + 선쉬운 빌드 구성을 위한 starter 종속성 제공 등등..

> :bulb: 객체 지향 프로그래밍
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
 

> :bulb: SOLID 원칙
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
 + Inversion of Control : 프로그램의 제어 흐름을 직접이 아닌 외부에서 관리하는것, 즉 제어가 역전되는것
   + 기존의 프로그램은 클라이언트 구현 객체가 스스로 필요한 서버 구현 객체를 생성,연결,실행
   + Appconfig와 같이 외부에서 제어 흐름을 결정하고, 구현 객체는 실행의 역할만 담당하게 됌
   + EX) JUnit과 같은 `FrameWork` 도 IoC 

## DI
 + Dependency Injection : 외부에서 객체들의 의존 관계를 주입하는 것
 + `의존관계` : 정적 클래스 의존관계 + 실행 시점에 결정되는 동적 객체 의존 관계
 > :bulb: 정적인 클래스 의존관계
   + 실행하지 않아도 코드 분석(import)를 통해 클래스 간 의존관계를 분석 가능, 클래스 다이어그램
   + 하지만 어떤 객체가 주입될지는 분석 불가능

 > :bulb: 동적인 객체 인스턴스 의존 관계
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
  
  
 
# 스프링 컨테이너와 스프링 빈

## 스프링 컨테이너 생성

 > 스프링 컨테이너 생성
 ```java
   //ApplicationContext : 인터페이스
   ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
```
 + 스프링 컨테이너를 생성 후 AppConfig.class를 구성정보로 지정
 
 > 스프링 빈 등록
 ```java
 public class AppConfig {
 
    //@Bean(name = memberService2")로 이름 직접 부여 가능
    @Bean
    public MemberService memberService(){
        return new MemberServiceImpl(memberRepository());
    }
 ```
   + 컨테이너의 스프링 빈 저장소에 <빈 이름(memberService) + 빈 객체(MemberServiceImpl@x01)>로 저장
   + :pencil2: 빈 이름은 중복이 되면 안됌!
 
 > 스프링 빈 의존관계 설정
   + 스프링 컨테이너는 설정 정보 참고해 의존관계를 주입
   
## 스프링 빈 조회

  #### 1)모든 빈 조회
  ```java
  public class ApplicationContextInfoTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    
    @Test
    @DisplayName("모든 빈 출력하기")
    void findAllBean(){
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();

        for (String beanDefinitionName : beanDefinitionNames) {
            Object bean = ac.getBean(beanDefinitionName);
            System.out.println("name = " + beanDefinitionName + "object=" + bean);

        }
    }

    @Test
    @DisplayName("애플리케이션 빈 출력하기")
    void findApplicationBean(){
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();   
        
        for (String beanDefinitionName : beanDefinitionNames) {
            BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);
            
            if(beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION){
                Object bean = ac.getBean(beanDefinitionName);
                System.out.println("name = " + beanDefinitionName + "object=" + bean);

            }
        }
    }
} 
  ```
  + `getBeanDefinitonNames()` : 모든 빈 이름 조회, `getBean` : 빈 이름으로 빈 객체 조회
  + `ROLE_APPLICATION` : 직접 등록한 애플리케이션 빈, `ROLE_INFRASTRUCTURE` : 스프링이 내부에서 사용하는 빈
  
  #### 2)기본
  
  ```java
  public class ApplicationContextBasicFindTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("빈 이름으로 조회")
    void findBeanByName(){
        //인터페이스로 조회
        MemberService memberService = ac.getBean("memService", MemberService.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("타입으로 조회")
    void findBeanByType(){
        MemberService memberService = ac.getBean(MemberService.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("구체 타입으로 조회")
    void findBeanByName2(){
        //인터페이스 타입이 아닌 인스턴스 타입으로 조회
        //구현에 의존하여 좋지 않은 코드(유연성이 떨어짐)
        MemberService memberService = ac.getBean("memService", MemberServiceImpl.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("빈 이름으로 조회X")
    void findBeanByNameX(){
        //ac.getBean("xxxx", MemberService.class); => 에러
        //예외가 나야 성공
        assertThrows(NoSuchBeanDefinitionException.class, () -> {
            ac.getBean("xxxx", MemberService.class);
        });
        MemberService memberService = ac.getBean("memService", MemberServiceImpl.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }
}
  ```
  
  #### 3)동일 타입이 둘 이상
  
  ```java
  public class ApplicationContextSameBeanFindTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);

    @Test
    @DisplayName("타입으로 조회 시 같은 타입 둘 이상 존재하면, 중복 오류가 발생")
    void findBeanByTypeDuplicate(){
        MemberRepository bean = ac.getBean(MemberRepository.class);
        assertThrows(NoUniqueBeanDefinitionException.class, () -> {
            ac.getBean(MemberRepository.class);
        });
    }

    @Test
    @DisplayName("타입으로 조회 시 같은 타입 둘 이상 존재하면, 빈 이름 지정하면 됌")
    void findBeanByName(){
        MemberRepository memberRepository = ac.getBean("memberRepository1", MemberRepository.class);
        assertThat(memberRepository).isInstanceOf(MemberRepository.class);
    }

    @Test
    @DisplayName("특정 타입을 모두 조회")
    void findAllBeanByType(){
        Map<String, MemberRepository> beansOfType = ac.getBeansOfType(MemberRepository.class);
        for (String key : beansOfType.keySet()) {
            System.out.println("key = " + key + "value = " + beansOfType.get(key));
        }
        assertThat(beansOfType.size()).isEqualTo(2);
    }

    @Configuration
    static class SameBeanConfig {

        //반환 객체가 같은 경우
        @Bean
        public MemberRepository memberRepository1() {
            return new MemoryMemberRepository();
        }

        @Bean
        public MemberRepository memberRepository2() {
            return new MemoryMemberRepository();
        }
    }
   ```
  
  #### 4)상속 관계

   + 부모 타입으로 조회 시, 모든 자식 타입들도 함께 조회됌
     +  `Object` 타입으로 조회(객체 최상의 부모)할시 모든 스프링 빈이 조회됌
   ```java
   public class ApplicationContextExtendsFindTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

    @Test
    @DisplayName("부모 타입으로 조회 시, 자식이 둘이상 있으면 중복 오류 발생")
    void findBeanByTypeDuplicate(){
        DiscountPolicy bean = ac.getBean(DiscountPolicy.class);
        assertThrows(NoUniqueBeanDefinitionException.class,
                () -> ac.getBean(DiscountPolicy.class));
    }

    @Test
    @DisplayName("부모 타입으로 조회 시, 자식이 둘이상 있으면, 빈 이름 지정하면 됌")
    void findBeanByTypeBeanName(){
        DiscountPolicy rateDiscountPolicy = ac.getBean("rateDiscountPolicy", DiscountPolicy.class);
        assertThat(rateDiscountPolicy).isInstanceOf(RateDiscountPolicy.class);
    }

    @Test
    @DisplayName("특정 하위 타입으로 조회")
    void findBeanBySubType(){
        DiscountPolicy bean = ac.getBean(RateDiscountPolicy.class);
        assertThat(bean).isInstanceOf(RateDiscountPolicy.class);
    }

    @Test
    @DisplayName("부모 타입으로 모두 조회")
    void findAllBeanByParentType(){
        Map<String, DiscountPolicy> beansOfType = ac.getBeansOfType(DiscountPolicy.class);
        assertThat(beansOfType.size()).isEqualTo(2);
    }

    @Test
    @DisplayName("부모 타입으로 모두 조회 - Object")
    void findAllBeanByObjectType(){
        Map<String, Object> beansOfType = ac.getBeansOfType(Object.class);
    }ㅇ

    @Configuration
    static class TestConfig{
        @Bean
        public DiscountPolicy rateDiscountPolicy(){
            return new RateDiscountPolicy();
        }

        @Bean
        public DiscountPolicy fixDiscountPolicy(){
            return new FixDiscountPolicy();
        }
    }
}   
   ```

## BeanDefinition
 + 스프링 빈 설정 메타 정보를 담은 인터페이스로, 추상화에 의존하도록 설계됌
 + xml, java 등 다양한 빈 설정 형식을 가능하게 지원
 + ex) GenericXmlApplicationContext 는 'xmlBeanDefinitionReader'사용하여 'appConfig.xml'설정정보를 읽고 `beanDefinition` 생성
 + 자바의 config : `factoryBean`을 통해 등록하는 방법
 
 
# Singleton
 +  웹 애플리케이션은 요청에 의해 이루어지기 때문에, 클라이언트가 요청을 반복하면 객체가 계속 생성되는 문제가 발생
 > SPRING 없는 순수한 DI 컨테이너
 ```java
 public class SingletonTest {

    @Test
    @DisplayName("스프링 없는 순수한 DI 컨테이너")
    void pureContainer(){
        AppConfig appConfig = new AppConfig();
        //1.조회 : 호출할때 마다 객체를 생성
        MemberService memberService1 = appConfig.memberService();

        //2.조회 : 호출할때 마다 객체를 생성
        MemberService memberService2 = appConfig.mb emberService();

        //참조값이 다른 것을 확인(계속해서 객체가 생성됌);
        //memberService1 != memberService2
        Assertions.assertThat(memberService1).isNotSameAs(memberService2);
    }
} 
 ```
## Singleton Pattern

  + 클래스의 인스턴스가 1개만 생성되는것을 보장하는 디자인 패턴
  + 이미 만들어진 객체를 공유하여 효율적으로 사용 가능
  > SingletonService.java
  ```java
  public class SingletonService {

    //클래스 레벨에 올라가 단 하나만 존재
    private static final SingletonService instance = new SingletonService();

    //조회
    public static SingletonService getInstance(){
        return instance;
    }
    
    //new 키워드 막기
    private SingletonService(){
    }

    public void logic(){
        System.out.println("싱글톤 객체 로직 호출");
    }
  }
  ```
   + `static` : 미리 하나의 객체만 클래스 레벨에 올려둠
   +  후에 객체 인스턴스를 요청할때는 getInstance() 를 통해서만 조회 가능 => _항상 같은 객체를 반환_
   +  생성자를 private으로 막아서 new로 인한 객체 인스턴스 생성 차단
 
   > 싱글톤 패턴의 문제점
     + `DIP/OCP 위반` : 클라이언트가 구체 클래스에 의존(구체클래스.getInstance())
     +  테스트가 어렵고 유연성이 떨어짐
     
## Singleton Container
 + 싱글톤 문제점 해결 + 객체 인스턴스를 싱글톤으로 관리 (ex)Spring Bean)
 + `Spring Container` : 싱글톤 컨테이너로, 객체를 하나만 생성해서 관리(싱글톤 레지스트리)함. 이미 만들어진 객체를 공유하여 재사용 
 ```java
   @Test
    @DisplayName("스프링 컨테이너와 싱글톤")
    void springContainer(){
     
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService1 = ac.getBean("memberService", MemberService.class);
        MemberService memberService2 = ac.getBean("memberService", MemberService.class);

        //테스트 성공
        assertThat(memberService1).isSameAs(memberService2);
    }
 ```
## Singleton 주의점
  + 객체 인스턴스를 공유하기 때문에 싱글톤 객체의 상태는 _무상태_ 로 설계해야함
    + 특정 클라이언트에 의존적이거나 값 변경 가능한 필드 사용 불가(공유 변수도 X)
    ```java
    public class StatefulService {

      //private int price; //상태 유지 필드 10000 -> 20000

      public int order(String name, int price){
          System.out.println("name = " + name + "price = " + price);
          return price;
          //this.price = price;
      }
      /*
      public int getPrice(){
          return price;
      }
      */
    }
    ```
     >  :star2:  공유 필드는 항상 조심해야한다!!
    ```java
    @Test
    void statefulServiceSingleton(){
        //Service 1,2는 같은 객체(싱글톤 컨테이너)
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

        //ThreadA
        int aPrice = statefulService1.order("userA", 10000);
        //ThreadB
        int bPrice = statefulService2.order("userB", 20000);

        //ThreadA 가 주문 금액 조회
        //int price = statefulService1.getPrice();   //20000
        //assertThat(statefulService1.getPrice()).isEqualTo(20000);
        System.out.println("price = " + aPrice);  //10000
    }

    static class TestConfig{
        @Bean
        public StatefulService statefulService(){
            return new StatefulService();
        }
    }
    ```
 ## Configuration 과 바이트코드 조작
  + AppConfig.java에서 MemberRepository가 3번 반복되서 호출되지 않는 이유?
     + SPIRING이 AppConfig을 상속받은 임의 클래스 생성후, 스프링 빈으로 등록하기 때문!
     + `CGLIB` : 바이트코드 조작 라이브러리, 이를 사용하여 다른 클래스 생성
     + 스프링 빈이 없을 때만 새로 등록하고, 이미 존재하면 컨테이너의 빈을 반환하는 코드를 동적으로 생성하여 **싱글톤 보장**
     
  > @Configuration을 붙이지 않으면 싱글톤이 보장되지 않는다!(빈으로 등록은 됌)


# 컴포넌트 스캔

## 컴포넌트 스캔과 의존관계 자동 주입
 + SPRING은 설정 정보(appConfig) 없이도 자동으로 스프링 빈을 등록하고, 의존관계도 자동으로 주입해줌
 
 > `@ComponentScan` : '@Component` 붙은 모든 클래스 스프링 빈 자동 등록
 ```java
  @Configuration
 //@붙은 클래스들을 스캔 후 자동 스프링 빈 등록
 @ComponentScan(
         excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
 )
 public class AutoAppConfig {

 }
 ```
 
 > `@Autowired` : 컨테이너가 해당 타입과 맞는 스프링 빈 찾아 의존관계 자동 주입
 ```java
 @Component
public class MemberServiceImpl implements MemberService {

    private final MemberRepository memberRepository;

    //자동 의존관계 주입 => MemoryMemberRepository 스프링 빈 주입해줌!
    //ac.getBean(MemberRepository.class)
    @Autowired
    public MemberServiceImpl(MemberRepository memberRepository){
        this.memberRepository = memberRepository;
    }
 ``` 
  + 타입으로 조회하기 때문에 ac.getBean(MemberRepository.class)와 유사하게 동작
  + 이름 다르지만 같은 타입의 빈이 2개 이상 존재 => NoUniqueBeanDefntionException 발생!
 
 ## 탐색 위치와 기본 스캔 대상
 
 + 1.탐색 위치
 ```java
 @ComponentScan(
        basePackages = "hello.core.member",
        ...
)
```
   + `basePackages` : 탐색할 패키지의 시작 위치 지정, 지정 패키지의 하위 패키지 모두 탐색
   + `basePackageClasses` : 지정 클래스의 패키지를 탐색 시작 위치로 지정
   +  둘 다 지정하지 않으면 : @ComponentScan 이 붙은 설정 정보 클래스의 패키지가 시작 위치가 됌, 모두 탐색
 
   > 설정 정보는 프로젝트 시작 루트 위치에 둬야 함!(ex) com.hello)

 + 2.기본 스캔 대상
   + `@Component` : 컴포넌트 스캔에서 사용
   + `@Controller` : 스프링 MVC 컨트롤러에서 사용
   + `@Service` : 스프링 비즈니스 로직에서 사용
   + `@Repository` : 스프링 데이터 접근 계층에서 사용, 예외를 스프링 예외로 변환
   + `@Configuration` : 스프링 설정 정보에서 사용, 싱글톤 유지하도록 추가 처리
 
## Filter
 
 ```java
 public class ComponentFilterAppConfigTest {

    @Test
    void filterScan(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(ComponentFilterAppConfig.class);
        BeanA beanA = ac.getBean("beanA", BeanA.class);
        Assertions.assertThat(beanA).isNotNull();

        assertThrows(
                NoSuchBeanDefinitionException.class,
                () -> ac.getBean("beanB", BeanB.class));

    }
 }
 ```
 ```java
    @Configuration
    @ComponentScan(
            includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class ),
            excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)

    )
    static class ComponentFilterAppConfig{

    }
```
  + includeFilters에 @MyIncludeCompenet을 추가하여 BeanA만 스프링 빈에 등록됌
    
## 빈의 중복 등록과 충돌 
+ 1. 자동 빈 등록 vs 자동 빈 등록 : `ConflictingBeanDifintionException` 발생
+ 2. 수동 빈 등록 vs 자동 빈 등록 : 수동 등록 빈이 우선순위를 가짐(자동 빈을 오버라이딩)
  + But Spring Boot에서는 오류가 남(잡기 어려운 버그가 생성되기 때문에)



# 의존관계 자동 주입

## 의존관계 주입 방법
 
 1) 생성자 주입
 
  + 생성자 통해서 의존관계 주입, 빈 등록과 관계 주입이 동시에 일어남
  + 생성자 호출 시점에만 1번 호출되기 때문에, **불변, 필수** 의존관계에 사용
  
  > 생성자가 1개 일시, @Autowired 생략 가능
  ```java
  @Component
public class OrderServiceImpl implements  OrderService{

    //final => 생성자 주입에서만 붙일 수 있음
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;   
  
    // @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
  ```
  + 순수한 자바 코드 테스트(Spring X)가 가능하며, 오직 생성자 주입에서만 final 키워드 사용 가능
  
 2) setter 주입
  + 필드 값 변경하는 수정자 메서드 통해 관계 주입
  + **선택, 변경** 가능성 있는 의존관계에 사용
 ```java
 @Component
 public class OrderServiceImpl implements  OrderService {

    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;

    //@Autowired(required = false) : 주입할 대상이 없어도 동작
    @Autowired
    public void setMemberRepository(MemberRepository memberRepository) {
    this.memberRepository = memberRepository;
    }

    @Autowired
    public void setDiscountPolicy(DiscountPolicy discountPolicy) {
    this.discountPolicy = discountPolicy;
    }
 }
 ```
 
 3) 필드 주입
  + 외부에서 변경 불가능하여 테스트 하기 어려운 단점이 있음
  + DI 프레임워크 없이 순수 자바 코드로 불가능
    + 애플리케이션 실제 코드와 관계 없는 테스트 코드에서는 사용 가능
  ```java
  @Component
  public class OrderServiceImpl implements OrderService {
     @Autowired private MemberRepository memberRepository;
     @Autowired private DiscountPolicy discountPolicy;
  }
  ``` 
  
 4) 일반 메서드 주입(잘 사용 X)
 ```java
  @Component
  public class OrderServiceImpl implements OrderService {
      private MemberRepository memberRepository;
      private DiscountPolicy discountPolicy;

      @Autowired
      public void init(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
         this.memberRepository = memberRepository;
         this.discountPolicy = discountPolicy;
      }
  }
 ```
 
## 옵션 처리
 + 자동 주입 대상을 옵션으로 처리( 주입할 스프링 빈 없어도 동작 가능하도록)
 ```java
  static class TestBean{

        //Member 는 빈으로 등록되어 있지 X
        @Autowired(required = false)
        public void setNoBean1(Member noBean1){
            System.out.println("noBean1 = " + noBean1);  //X
        }

        @Autowired
        public void setNoBean2(@Nullable Member noBean2){
            System.out.println("noBean2 = " + noBean2);  //null
        }

        @Autowired
        public void setNoBean3(Optional<Member> noBean3){
            System.out.println("noBean3 = " + noBean3);  //Optional.empty
        }
    }
 ```
  + `@Autowired(required = false)` : 메소드 호출 자체가 안됌
  + `@Nullable` : 자동 주입 대상 없으면 null이 입력
  + `Optional<>` : 자동 주입 대상 없으면 'Optional.empty' 입력

## Bean이 여러개인 경우
 + `Autowired` : 타입(Interface)으로 조회하기 때문에 ac.getBean(DiscountPolicy.class)와 유사하게 동작
  + 이름 다르지만 같은 타입의 빈이 2개 이상 존재시 => NoUniqueBeanDefntionException 발생!

 #### :bulb: Solution
 1. **@Autowired 필드 명 매칭**
   + 타입 매칭의 결과가 2개 이상 : 필드명 또는 파라미터 명으로 빈 이름 매칭
   ```java
    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy rateDiscountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = rateDiscountPolicy;
    }  
   ```
 2.**@Qualifier 사용**
   + 추가 구분자를 붙여주는 일종의 옵션, 빈 이름 변경하는 것이 아님!
   ```java
   @Component
   @Qualifier("mainDiscountPolicy")
   public class RateDiscountPolicy implements DiscountPolicy {}   
   ```
   
   ```java
   @Component
   @Qualifier("fixDiscountPolicy")
   public class FixDiscountPolicy implements DiscountPolicy {}
   ```  
   > 생성자 자동 주입
   ```java
   @Autowired
   public OrderServiceImpl(MemberRepository memberRepository, @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
      this.memberRepository = memberRepository;
      this.discountPolicy = discountPolicy;
   }   
   ```
   + @Qualifier("mainDiscountPolicy") 못찾을시, 같은 이름의 스프링 빈을 찾고 없으면 예외 발생
 3.**@Primary 사용**
  + `@Primary` : 이 Annotation이 붙은 빈이 우선순위를 가짐
    + 주로 메인 데이터베이스의 커넥션을 획득하는 스프링 빈에서 사용하며, 서브 데이터 베이스는 `@Qualifier`를 지정해 명시적으로 획득하도록 사용
  ```java 
  @Component
  @Primary
  public class RateDiscountPolicy implements DiscountPolicy {}  //rate에 우선권을 준다.
  @Component
  public class FixDiscountPolicy implements DiscountPolicy {}
  ```
  
 4.**직접 Annotation 생성**
 ```java 
  //잘못 입력하는 등의 컴파일 오류 잡기 위해 직접 annotation 생성
  @Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
  @Retention(RetentionPolicy.RUNTIME)
  @Inherited
  @Documented
  @Qualifier("mainDiscountPolicy")
  public @interface MainDiscountPolicy {
  }
 ```
  + 사용하려는 빈과 OrderServiceImpl 생성자 파라미터 앞에 @MainDiscountPolicy 붙이면 됌 
  
 ## 모든 빈 조회
 
 > 클라이언트가 할인의 종류를 선택할 수 있다고 가정하면, 해당 타입의 스프링 빈이 모두 필요
 ```java
 public class AllBeanTest {

    @Test
    void findAllBean() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);
        DiscountService discountService = ac.getBean(DiscountService.class);
        Member member = new Member(1L, "userA", Grade.VIP);
        int discountPrice = discountService.discount(member, 10000, "fixDiscountPolicy");

        assertThat(discountService).isInstanceOf(DiscountService.class);
        assertThat(discountPrice).isEqualTo(1000);

        int rateDiscountPrice = discountService.discount(member, 10000, "fixDiscountPolicy");
        assertThat(rateDiscountPrice).isEqualTo(1000);

    }

    //AnnotationConfigApplicationContext(DiscountService.class) : 자동으로 스프링 빈으로 등록해줌
    static class DiscountService {
        private final Map<String, DiscountPolicy> policyMap;
        private final List<DiscountPolicy> policies;

        @Autowired
        public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policies) {
            this.policyMap = policyMap;
            this.policies = policies;
            System.out.println("policyMap = " + policyMap);  //{ name1 : object1, name2 : object2 .. }
            System.out.println("policies = " + policies); 
        }

        public int discount(Member member, int price, String discountCode) {
            DiscountPolicy discountPolicy = policyMap.get(discountCode);  //rate or fix discountpolicy 넘어옴
            return discountPolicy.discount(member, price);
        }
    }
}
```
  + `Map<>` : map의 키에 스프링 빈의 이름을 넣어주고, 그 값으로 해당 타입으로 조회한 모든 스프링 빈을 담아줌
  
 #### :pencil2: 자동과 수동의 올바른 실무 운영 기준
 
  + `업무 로직 빈` : 웹 지원 컨트롤러, 서비스, 리포지토리 등 비즈니스 요구사항을 개발할때 추가/수정 되는 로직. 문제의 원인 파악이 쉬워 **자동 빈 등록 기능 사용**
    + > 업무 로직에서 다형성 활용 케이스
    + Map<String, DiscountPolicy>에 주입될 빈들에 대한 정보는, 1)별도의 설정정보 만들어 수동 등록 하거나, 2)자동으로 할 시 같은 패키지로 묶어놔야함

  + `기술 지원 빈` : 업무 로직 지원하기 위한 하부 기술, 공통 관심사(AOP)처리 시 주로 사용. 애플리케이션 전반에 광범위한 영향을 미치기 때문에 **수동 빈으로 등록**하여 설정 정보에서 명확하게 드러내야함




# 빈 생명주기 콜백

## 빈 생명주기 콜백
 + 애플리케이션 시작 시점에 필요한 커넥션(DB 커넥션 풀, 네트워크 소켓)을 미리 생성하거나, 종료 시점에 커넥션을 종료하는 작업을 진행해야함
   + 객체의 초기화와 종료 작업이 필요!

**:bulb:스프링 빈의 이벤트 라이프사이클**
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

1.**인터페이스 InitializingBean, Disposable Bean**
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
2.**빈 등록 초기화,소멸 메소드**
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

3.**@PostConstruct, @PreDestory**
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



# 빈 스코프
 + 빈이 존재할 수 있는 범위, 스프링 빈은 기본적으로 싱글톤 스코프로 생성

## 빈 스코프와 종류

#### 스코프의 종류
+ `싱글톤` : 기본 스코프로써, 스프링 컨테이너와 같은 생명주기를 가지는 스코프(시작-종료)
+ `프로토타입` : 스프링 컨테이너가 빈의 생성과 의존관게 주입까지만 관여(초기화 메소드까지는 불러줌)하는 매우 짧은 범위의 스코프
+ `웹 관련 스코프`
  + `request` : 웹 요청이 들어오고 나갈때까지 유지되는 스코프
  + `session` : 웹 세션이 생성되고 종료될 때까지 유지되는 스코프
  + `application` : 웹의 서블릿 컨텍스와 같은 범위로 유지되는 스코프  

## 프로토타입 스코프
 + 조회 시 스프링 컨테이너는 항상 새로운 인스턴스를 생성해서 반환
 > ☑️ 싱글톤 스코프 : 컨테이너가 같은 인스턴스(주소가 같음)의 스프링 빈 반환
 
#### 빈 요청 과정
1. 프로토타입 스코프의 빈을 스프링 컨테이너에 요청</br>
2. :star2: 요청 시 컨테이너가 프로토타입 빈 생성, 의존관계 주입(DI), 초기화 까지 처리
  + 따라서, **@PreDestory** 와 같은 **종료 메소드 호출 불가능**
  + 빈을 조회한 클라이언트가 프로토타입 빈을 관리해야함
4. 생성한 빈을 클라이언트에 반환하고 관리하지 X, 이후 같은 요청이 올때마다 새로운 프로토타입 빈 생성해서 반환

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
    
 #### 싱글톤 빈과 함께 사용시 문제점
 
 + 스프링의 기본은 싱글톤 빈이므로, 프로토타입을 사용하는 경우가 생김
 + 다만 싱글톤 빈은 생성 시점에만 의존관계 주입을 받기 때문에, **프로토타입이 요청 시마다 새로 생성되는 것이 아니라 주입 시에만 한번 생성되어 같은 빈이 유지됌**
 ```java
   @Scope("singleton")
    static class ClientBean{
        private final PrototypeBean prototypeBean;   //생성 시점에 이미 주입(계속 같은 것)

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
 ```
 
 > :bulb: Solution?
 
 **1.스프링 컨테이너에 요청**
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

**2.ObjectProvider**
 + 스프링 컨테이너에서 필요한 빈을 대신 조회하는 DL 서비스를 제공, 스프링에 의존적
 
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
 
 **3.JSR303 Provider**
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
    + `session` : HTTP Session과 동일 생명주기 가지는 스코프
    + `application` : 서블릿 컨텍스와 같은 범위로 유지되는 스코프 
    + `websocket` : 웹 소켓과 동일한 생명주기 가지는 스코프

#### 웹 환경 구축
 ```java
 //web 라이브러리 추가
 implementation 'org.springframework.boot:spring-boot-starter-web'
 ```
  + `spring-boot-starter-web` : 스프링 부트는 내장 톰켓 서버 활용하여 웹서버와 스프링 함께 실행
  + 웹 라이브러리 추가 시`AnnotationConfigServletWebServerApplicationContext`를 기반으로 애플리케이션을 구동함

#### Request 스코프
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
  +`MyLogger` : request scope이기 때문에, requestURL과 같은 웹 관련 정보를 서비스 계층에서 파라미터로 넘기지 않고 MyLogeer 멤버 변수에 저장하여 코드 유지 가능
  + :fire: 웹과 관련된 부분은 컨트롤러까지만 사용, **서비스 계층은 웹 기술에 종속되면 안됌!**

> 컨테이너에게 빈을 요청하는 단계는 의존관계 주입 단계가 아니라 뒤로 지연시켜야 오류 발생 X

#### Request scope와 Provider
 +`ObjectProvider`: ObjectProvider.getObject() 를 호출하는 시점까지 request scope 빈의 생성을 지연 가능
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
#### 스코프와 프록시

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
 
