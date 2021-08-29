# Singleton
 +  웹 애플리케이션은 요청에 의해 이루어지기 때문에, _클라이언트가 요청을 반복하면 객체가 계속 생성되는 문제가 발생_
 
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

        //참조값이 다른 것을 확인
        //memberService1 != memberService2
        Assertions.assertThat(memberService1).isNotSameAs(memberService2);
    }
} 
 ```
   + `테스트 결과` : 두 객체의 참조 값이 다름(계속해서 객체가 생성되고 있음)
   
## Singleton Pattern

  + :star2: _클래스의 **인스턴스가 1개만 생성**되는것을 보장하는 디자인 패턴_
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
 
   > 🤨 싱글톤 패턴의 문제점
   + `DIP/OCP 위반` : 클라이언트가 구체 클래스에 의존
      + `구체클래스.getInstance()`
   +  테스트가 어렵고 유연성이 떨어짐
     
## Singleton Container
 + 싱글톤 문제점 해결 + 객체 인스턴스를 싱글톤으로 관리 (ex)Spring Bean)
 + `Spring Container` : 싱글톤 컨테이너로, 객체를 하나만 생성해서 관리(싱글톤 레지스트리)함
   + 이미 만들어진 객체를 공유하여 재사용 
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
  + :pencil2: 객체 인스턴스를 공유하기 때문에 _싱글톤 객체의 상태는 **무상태**로 설계해야함_
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
    + :star2:  공유 필드는 항상 조심해야함!

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
     + Spring이 AppConfig을 상속받은 임의 클래스 생성후, 스프링 빈으로 등록하기 때문!
     + `CGLIB` : 바이트코드 조작 라이브러리, 이를 사용하여 다른 클래스 생성
     + 스프링 빈이 없을 때만 새로 등록하고, 이미 존재하면 컨테이너의 빈을 반환하는 코드를 동적으로 생성하여 **싱글톤 보장**
     
  > @Configuration을 붙이지 않으면 싱글톤이 보장되지 않는다!(빈으로 등록은 됌)
