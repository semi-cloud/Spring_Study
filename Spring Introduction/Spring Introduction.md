
# MVC
+ Model, View, Controller
+ 서버에서 변경 후 렌더링 된 html 을 클라이언트에게 전달
+ 하나의 view 파일에 컨트롤러까지 처리하지 않고 둘을 분리하는 방식

## View

+ 정적 컨텐츠
  + 파일 그대로 웹 브라우저, 클라이언트에게 전달
  + spring container에서는 controller 우선순위를 주며 부재 시 resources/static 에서 정적 파일 찾아서 반환
  
+ 템플릿 엔진
  + viewResolver에서 찾은 view 파일을 템플릿 엔진을 통해 렌더링 후 웹브라우저에 전달 

## Controller

+ WEB의 첫 진입점으로, 서버/ 비즈니스 로직
```java
@GetMapping("hello-mvc")
    public String helloMvc(@RequestParam(name = "name") String name, Model model){
        model.addAttribute("name", name);
        return "hello-template";
    }
```
  + `@RequestParam` : Get 방식으로 넘어온 URL의 쿼리 스트링 받기 적절한 방법/해당 파라미터가 URL로 전송
  
## API
+ JSON 데이터 구조 포맷으로 데이터를 클라이언트에게 전달(뷰 필요 X)
```java
@GetMapping("hello-string")
    @ResponseBody
    public String helloString(@RequestParam("name") String name) {
        return "hello" + name; //"hello spring"
    }
```
  + `@ResponseBody` : ViewResolver 사용하지 않고 http body에 직접적으로 전달
```java
 @GetMapping("hello-api")
    @ResponseBody
    public Hello helloApi(@RequestParam("name") String name) {
        Hello hello = new Hello();
        hello.setName(name);
        return hello;
    }

    static class Hello {
        private String name;
        
        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
```
  + `객체 반환 + @ResponseBody 이용` : { key : value } 형식의 json 방식으로 데이터를 생성
  + `HttpMessageConverter` : ViewResolver 대신 Json(객체)/String(문자) 데이터 방식에 맞추어 서버에 응답

## Domain
+ 비즈니스 도메인 객체로, 데이터베이스에 저장하고 관리됨

## Repository
+ 데이터베이스에 접근하여 도메인 객체를 DB에 저장하고 관리
+ 같은 기능(메소드)에 다른 종류 repository는 _interface_로 관리!
```java
public interface MemberRepository {
    //Optional<T> : NPE 방지, null이 올 수 있는 값을 감싸는 Wrapper 클래스

출처: https://mangkyu.tistory.com/70 [MangKyu's Diary]

    Member save(Member member);
    Optional<Member> findById(Long id);
    Optional<Member> findByName(String name);
    List<Member> findAll();
}
```

## Service
+ 핵심적 비즈니스 로직 구현, 비즈니스에 의존적으로 설계

#####  💡Controller(외부 요청 받기)-Service(비즈니즈 로직 생성)-Repository(데이터 저장) : 정형화된 Pattern
-------------

# Test

#### 1)회원 레포지토리 테스트
```java
public class MemoryMemberRepositoryTest {

    //테스트의 끝에는 data clear 과정이 필요
    MemoryMemberRepository repository = new MemoryMemberRepository();

    //각 메소드 끝난 후 동작되는 콜백 메소드
    @AfterEach
    public void afterEach(){
        repository.clearStore();
    }

    @Test
    public void save(){
        Member member = new Member();
        member.setName("spring");

        repository.save(member);

        Member result = repository.findById(member.getId()).get();
        //System.out.println("result = " = (result === member));
        //Assertions.assertEquals(member, result);     //같은지 check, 같으면 초록불
        //Assertions.assertEquals(member, null);       //빨간불

        assertThat(member).isEqualTo(result);          //true or false 반환, 테스트 시 사용
    }
```
  + `@AfterEach` : 테스트를 반복적으로 수행 할때, 각 테스트 종료 시마다 동작되는 콜백 메소드<br>
                 : 메모리 DB에 남은 직전 테스트 데이터를 모두 삭제 
  + 각 Test는 서로의 의존 관계가 없이 _독립적으로 실행_ 되어야 한다!

#### 2)회원 서비스 테스트
```java
class MemberServiceTest {

    //shift + f10 : 이전에 실행했던 테스트 다시 실행
    MemberService memberService;
    MemoryMemberRepository memberRepository;

    //각 테스트 실행 전에 수행(같은 MemoryMemberRepository 사용 위해)
    @BeforeEach
    public void beforeEach(){
        memberRepository = new MemoryMemberRepository();
        memberService = new MemberService(memberRepository);
    }
}
```
  + ~~MemberService memberService = new MemberService(memberRepository);~~
  + `@BeforeEach` : 각 테스트 실행 전에 수행, 같은 객체 사용을 위해서(테스트의 독립성) 이 안에서 새로운 객체 생성하며 의존관계를 맺어줌
```java
 @AfterEach
    public void afterEach(){
        memberRepository.clearStore();
    }
    
 //로직을 실행 시 어떠한 exception 이 터져야 test 성공하는 경우
 @Test
    public void 중복_회원_예외(){
        //given
        Member member1 = new Member();
        member1.setName("Spring");

        Member member2 = new Member();
        member2.setName("Spring");

        //when
        memberService.join(member1);
        IllegalStateException e = assertThrows(IllegalStateException.class,
                () -> memberService.join(member2));        //join(member2)를 할때 exception 발생
                
        //then
        Assertions.assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다");
        
         /*
        try{
            memberService.join(member2);
            fail();     
        }catch(IllegalStateException e){        //예외 터짐(정상작동)
            Assertions.assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다");
        }
        */
    }
```
-------------

# 스프링 빈과 의존관계

## 컴포넌트 스캔과 자동 의존관계 설정

```java
@Controller
public class MemberController {

    //private final MemberService memberService = new MemberService(); 기능이 적기 때문에 공용으로 사용 가능
    private final MemberService memberService;

    //생성자에 붙여서 최초 객체 생성 시점에 스프링 빈을 주입
    @Autowired
    public MemberController(MemberService  memberService) {
        this.memberService = memberService;
    }

```
  + 스프링 컨테이너에서 'controller 객체' 와 연관객체 등록 후 관리 가능 == spring 빈(bin)이 관리됌
  + `@Autowired` : DI(Dependency Injection), 즉 외부에서 의존성을 주입하는 방법. Spring이 연관 객체를 찾아서 컨테이너에 등록 시키고 의존성을 연결한다
    > DI의 세가지 방법
      + 1)생성자 주입(초기 조립 시점 이후 변경 불가능해 선호)
      + 2)필드 주입
      + 3)setter 주입(public 으로 열려있어서 변경이 동적)
                                  
  + `@Component` : 컴포넌트 스캔 방식으로, Spring이 스프링 빈으로 자동 등록 시킨다. 이하 속하는 3가지 컴포넌트 모두 해당한다.
    + `@Controller`
    + `@Service`
    + `@Repository`
    
  > 참고 : 스프링 컨테이너에 스프링 빈을 등록할 때, 유일하게 하나만 등록해서 공유(싱글톤). 따라서 같은 스프링 빈이면 모두 같은 인스턴스로 취급!

## 자바 코드를 통한 스프링 빈 등록

```java
@Configuration
public class SpringConfig {

    private EntityManager em;
    @Autowired
    public SpringConfig(EntityManager em){
        this.em = em;
    }

    //객체가 스프링 빈에 등록됌(스프링 컨테이너)
    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository(){
       return new MemoryMemberRepository();
       //return new JpaMemberRepository(em);
    }
}
```
  + `@Bean` : 객체가 스프링 빈으로 스프링 컨테이너에 등록됌
  +  일단 container 에 등록된 객체들에 한해서만, @AutoWired로 연결 가능!
  +  변경이 필요할 때 Config 파일 하나만 변경하고, 나머지 코드는 건드리지 않아도 되는 장점이 있다
---

## 💡 (참고) Get과 Post

>MemberController.java
```java
    @GetMapping("/members/new")      
    public String createForm(){
        return "members/createMemberForm";
    }

    @PostMapping("/members/new")      
    public String create(MemberForm form){
        Member member = new Member();
        member.setName(form.getName());

        memberService.join(member);

        return "redirect:/";
    }
```
  + `@GetMapping()` : 데이터 조회 시 get 방식 사용
  + `@PostMapping()` : 데이터 전달 시 post 방식 사용
  
> createMemberForm.html 중 일부
```HTML
<form action="/members/new" method="post">
    <div class="form-group">
      <label for="name">이름</label>
      <input type="text" id="name" name="name" placeholder="이름을
입력하세요">
    </div>
    <button type="submit">등록</button>
  </form>
```
> MemberForm.java
```java
public class MemberForm {
    private String name;  //html 의 name = "name"

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
---
# 스프링 DB 접근 기술
   + 데이터를 메모리가 아닌, DB에 저장하므로 스프링 서버를 다시 실행해도 데이터가 안전하게 저장됌
## 순수 JDBC
 > JdbcMemberRepository.java 중 일부
```java
//인터페이스 MemberRepository 구현
public class JdbcMemberRepository implements MemberRepository {

    private final DataSource dataSource;
    public JdbcMemberRepository(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Override
    public Member save(Member member) {
        String sql = "insert into member(name) values(?)";
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        try {
            conn = getConnection();
            pstmt = conn.prepareStatement(sql,
                    Statement.RETURN_GENERATED_KEYS);
            pstmt.setString(1, member.getName());
            pstmt.executeUpdate();
            rs = pstmt.getGeneratedKeys();
            if (rs.next()) {
                member.setId(rs.getLong(1));
            } else {
                throw new SQLException("id 조회 실패");
            }
            return member;
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally {
            close(conn, pstmt, rs); }
    }
```
 > 스프링 설정 변경
```java
@Configuration
public class SpringConfig {

     private DataSource dataSource;
     public SpringConfig(DataSource dataSource) {
         this.dataSource = dataSource;
     }
     
    //객체가 스프링 빈에 등록됌(스프링 컨테이너)
    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository);
    }
    
    @Bean
    public MemberRepository memberRepository(){
        // return new MemoryMemberRepository();
        return new JdbcMemberRepository(dataSource);
    }
}
```
  + `DataSource` : 데이터베이스 커넥션을 획득할 때 사용하는 객체임. 스프링 부트는 데이터베이스 커넥션 정보를 바탕으로 DataSource를 생성하고 스프링 빈으로 만들어 DI 받기 가능
  + `OCP (개방-폐쇄 원칙)` 준수 : 스프링의 DI를 사용하여 기존 코드 변경하지 않고 설정 통해 구현 클래스 변경 가능

> 스프링 통합 테스트(+DB)
```java
@SpringBootTest    //스프링 컨테이너와 테스트를 함께 실행
@Transactional     //DB rollback => 테스트 중복 가능
public class MemberServiceIntegrationTest {

    @Autowired MemberService memberService;
    @Autowired MemberRepository memberRepository;

    @Test
    @Commit
    void join() {
        //given
        Member member = new Member();
        member.setName("hello");

        //when
        Long saveId = memberService.join(member);

        //then
        Member findMember = memberService.findOne(saveId).get();
        Assertions.assertThat(member.getName()).isEqualTo(findMember.getName());
    }
```
  + `@Transactional` : 테스트의 독립성을 지키기 위해, 테스트 시작 전 트랜직션을 시작하고 끝나면 DB 롤백(이전 상태로 돌아감)!
  
## 스프링 JDBCTemplate
 > JdbcTemplateMemberRepository.java 중 일부
```java
public class JdbcTemplateMemberRepository implements MemberRepository{
    private final JdbcTemplate jdbcTemplate;

    @Autowired     //생성자 하나면 autowired 생략 가능
    public JdbcTemplateMemberRepository(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    @Override
    public Optional<Member> findById(Long id) {
        List<Member> result = jdbcTemplate.query("select * from member where id = ?",memberRowMapper(), id);
        return result.stream().findAny();
    }
```
## JPA
 + 반복 코드 제거, 기본적인 SQL도 JPA가 직접 만들어서 실행 => 객체 중심 설계로 전환
 
 > JPA 엔티티 매핑
```java
@Entity     //jpa 관리 entity
public class Member {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)     //db가 자동으로 id 생성
    private Long id;   
    private String name;
```
 > JpaMemberRepository.java 중 일부
```java
public class JpaMemberRepository implements MemberRepository {

    //jpa => entityManager 로 동작
    private final EntityManager em;

    public JpaMemberRepository(EntityManager em) {
        this.em = em;
    }
    
    @Override
    public Optional<Member> findById(Long id) {        //pk
        Member member = em.find(Member.class, id);     //조회할 타입, 식별자
        return Optional.ofNullable(member);
    }

    @Override
    public Optional<Member> findByName(String name) {   //pk아닌 대상은 쿼리 작성 필요
        List<Member> result = em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name)
                .getResultList();

        return result.stream().findAny();
    }
```
## 스프링 데이터 JPA
  + 리포지토리에 클래스가 아닌 인터페이스만으로 개발 가능 + JpaRepository에서 기본 CRUD 기능 제공
> SpringDataJpaMemberRepository
```java
public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>, MemberRepository {

    //findByName : 인터페이스에서 공통화 불가능
    //메소드 이름만으로 JPQL select m from Member m where m.name = ? 와 같은 규칙 생성
    @Override
    Optional<Member> findByName(String name);
}
```
  + 스프링 data jpa가 spring bin에 자동으로 리포지토리를 등록 시킴

> 스프링 설정 변경 
```java
@Configuration
public class SpringConfig {

    //스프링 data jpa 가 spring bin 에 자동으로 repository 를 등록시킴!(@Bean 필요 X)
    private final MemberRepository memberRepository;

    @Autowired
    public SpringConfig(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository);
    }
}
```
---
# AOP
 + Aspect Oriented Programming
 + 메소드 호출 시간 측정과 같은 공통 관심 사항과 핵심 관심 사항을 분리한 기법
> 분리되지 않은 예
```java
public List<Member> findMembers() {

    //모든 메소드에 대하여 실행해야 하기 때문에 유지보수가 힘듬
    long start = System.currentTimeMillis();
    try {
        return memberRepository.findAll();
    } finally {
        long finish = System.currentTimeMillis();
        long timeMs = finish - start;
        System.out.println("findMembers " + timeMs + "ms");
    }
}
``` 
> TimeTraceAop.java
```java
@Aspect
//@Component
public class TimeTraceAop {

    @Around("execution(* hello.hellospring..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable{
        long start = System.currentTimeMillis();
        System.out.println("START : " + joinPoint.toString());
        try{
            return joinPoint.proceed();
        }finally{
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END : " + joinPoint.toString() + " " + timeMs + "ms");

        }
    }
}
```
  + 하나의 java 파일로 관리하여 공통 관심 사항 타겟팅
  + spring bin 에 등록할때 proxy, 즉 가짜 스프링 빈을 세워놓아 memberController는 가짜 스프링 빈을 호출 후 나
