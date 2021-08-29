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
   + 컨테이너의 스프링 빈 저장소에 **빈 이름 + 빈 객체**로 저장
     + `빈 이름` : memberService
     + `빈 객체` : MemberServiceImpl@x01
   + :pencil2: 빈 이름은 중복이 되면 안됌!
 
 > 스프링 빈 의존관계 설정
   + 스프링 컨테이너는 설정 정보 참고해 의존관계를 주입
   
## 스프링 빈 조회

### :heavy_check_mark: 1)모든 빈 조회
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
  
  ### :heavy_check_mark: 2)기본
  
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
  
  ### :heavy_check_mark: 3)동일 타입이 둘 이상
  
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
  
  ### :heavy_check_mark: 4)상속 관계

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
 + xml, java 등 **다양한 빈 설정 형식을 가능하게 지원**
   + ex) `GenericXmlApplicationContext` 는 `xmlBeanDefinitionReader`사용하여 `appConfig.xml`설정정보를 읽고 `beanDefinition` 생성
 + 자바의 config : `factoryBean`을 통해 등록하는 방법
 
