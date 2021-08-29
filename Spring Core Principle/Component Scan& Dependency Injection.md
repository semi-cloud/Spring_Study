# Component Scan

## 컴포넌트 스캔
 + SPRING은 설정 정보(appConfig) 없이도 자동으로 스프링 빈을 등록하고, 의존관계도 자동으로 주입해줌
   + `@ComponentScan` : `@Component` 붙은 모든 클래스 스프링 빈 자동 등록
   + `@Autowired` : 컨테이너가 해당 타입과 맞는 스프링 빈 찾아 의존관계 자동 주입
 
 > @ComponentScan
 ```java
 @Configuration
 @ComponentScan(
         excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
 )
 public class AutoAppConfig {

 }
 ```
 
 > @Autowired
 ```java
 @Component
public class MemberServiceImpl implements MemberService {

    private final MemberRepository memberRepository;

    //자동 의존관계 주입 => MemoryMemberRepository 스프링 빈 주입해줌!
    @Autowired
    public MemberServiceImpl(MemberRepository memberRepository){
        this.memberRepository = memberRepository;
    }
 ``` 
  + 타입으로 조회하기 때문에 `ac.getBean(MemberRepository.class)`와 유사하게 동작
  + 이름 다르지만 같은 타입의 빈이 2개 이상 존재 => `NoUniqueBeanDefntionException` 발생!
  
  <img src="https://user-images.githubusercontent.com/71436576/131255017-4faee79e-0afd-41f4-91a9-1c2ada36e5ed.png" width=60% height=60%>
  
 
 ## 탐색 위치와 기본 스캔 대상
  + 모든 자바 클래스 말고 꼭 필요한 위치부터 탐색하도록, 시작 위치 지정 가능
  
 ### ✔️ 탐색 위치
 ```java
 @ComponentScan(
        basePackages = "hello.core.member",
        ...
)
```
   + `basePackages` : 탐색할 패키지의 시작 위치 지정, 지정 패키지의 하위 패키지 모두 탐색
   + `basePackageClasses` : 지정 클래스의 패키지를 탐색 시작 위치로 지정
   +  둘 다 지정하지 않으면 : @ComponentScan 이 붙은 설정 정보 클래스의 패키지가 시작 위치가 됌, 모두 탐색

#### 권장하는 방법
 +  설정 정보(AppConfig)는 프로젝트 시작 루트 위치에 두고, basePackages 지정은 생략
   + ex) `com.hello`: 프로젝트 시작 루트
 + `com.hello` 를 포함한 하위는 모두 자동으로 컴포넌트 스캔의 대상이 됌

### ✔️ 기본 스캔 대상
   + `@Component` : 컴포넌트 스캔에서 사용
   + `@Controller` : 스프링 MVC 컨트롤러에서 사용
   + `@Service` : 스프링 비즈니스 로직에서 사용
   + `@Repository` : 스프링 데이터 접근 계층에서 사용, 데이터 게층 예외를 스프링 예외로 변환
   + `@Configuration` : 스프링 설정 정보에서 사용, 싱글톤 유지하도록 추가 처리
 
## Filter
 + `includeFilters` : 컴포넌트 스캔 대상을 추가로 지정
 + `excludeFilters` : 컴포넌트 스캔에서 제외할 대상을 지정
 
 > AppConfig Test(1)
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

 > AppConfig Test(2)
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

