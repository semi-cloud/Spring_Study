# Spring MVC 

## Spring MVC 전체 구조

### :puspin: 직접 만든 프레임워크 vs 스프링 MVC
 + FrontController -> `DispatcherServlet`
 + handlerMappingMap -> `HandlerMapping`(Interface)
 + MyHandlerAdapter ->  `HandlerAdapter`(Interface)
 + ModelView -> `ModelAndView`
 + viewResolver -> `ViewResolver`(Interface)
 + MyView -> `View` (Interface)

### :pushpin: 스프링 MVC 구조와 동작 순서
<img src="https://user-images.githubusercontent.com/71436576/127876210-185a98f3-1227-4853-80fb-c4dcb3d16674.png" width=50% height=50%>

1) **핸들러 조회**: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회</br>
2) **핸들러 어댑터 조회**: 핸들러를 실행할 수 있는 핸들러 어댑터를 조회</br>
3) **핸들러 어댑터 실행**: 핸들러 어댑터를 실행</br>
4) **핸들러 실행**: 핸들러 어댑터가 실제 핸들러를 실행</br>
5) **ModelAndView 반환**: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환</br>
6) **viewResolver 호출**: 뷰 리졸버를 찾고 실행</br>
7) **View 반환**: 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환</br>
8) **뷰 렌더링**: 뷰를 통해서 뷰를 렌더링</br>

### :pushpin: DispatcherServlet
  + 부모 클래스에서 HttpServlet 상속받아 사용(Servlet으로 동작)
    + DispatcherServlet -> FrameworkServlet -> HttpServletBean -> HttpServlet
  + Spring Boot는 `DispatcherServlet`을 서블릿으로 '자동 등록'하면서, 모든경로에 대해 매핑( urlPatterns="/" )

#### 요청 흐름
  + FrameworkServlet.service() -> DispacherServlet.doDispatch()
  + :star2: `doDispatch()` : 스프링 MVC 동작의 모든 순서를 이 메소드에서 실행

> doDispatch()
```java
    // 1. 핸들러 조회
    mappedHandler = getHandler(processedRequest);
    if (mappedHandler == null) {
        noHandlerFound(processedRequest, response);
    return;
    }
    
    // 2. 핸들러 어댑터 조회 - 핸들러를 처리할 수 있는 어댑터
    HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
    
    // 3. 핸들러 어댑터 실행 -> 4. 핸들러 어댑터를 통해 핸들러 실행 -> 5. ModelAndView 반환
    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
    
    //뷰 렌더링 호출
    //render(mv, request, response);
    processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
}
```
```java
    protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
        View view;
        String viewName = mv.getViewName();
        // 6. 뷰 리졸버를 통해서 뷰 찾기, 7. View 반환
        view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
        // 8. 뷰 렌더링
        view.render(mv.getModelInternal(), request, response);
    }
```

#### :heavy_check_mark: HandlerMapping

 + 컨트롤러의 호출에는 두가지가 필요
   + `HandlerMapping` : 핸들러 매핑에서, 이 컨트롤러를 찾을 수 있어야함  ex)스프링 빈의 이름으로 찾는 핸들러 매핑
   + `HandlerAdapter` : 핸들러 매핑을 통해 찾은 핸들러를 실행 가능한 어댑터가 필요  ex)Controller 인터페이스 실행 가능한 어댑터
   
> HandlerMapping 종류
 + 0 = `RequestMappingHandlerMapping` : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
 + 1 = `BeanNameUrlHandlerMapping` : 스프링 빈의 이름으로 핸들러를 찾음
 
#### :heavy_check_mark: HandlerAdapter

> HandlerAdapter 종류
 + 0 = `RequestMappingHandlerAdapter` : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
 + 1 = `HttpRequestHandlerAdapter` : HttpRequestHandler 처리
 + 2 = `SimpleControllerHandlerAdapter` : Controller 인터페이스(애노테이션X, 과거에 사용) 처리

> EX) OldController
```java
  @Component("/springmvc/old-controller")
  public class OldController implements Controller {
     @Override
     public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
         System.out.println("OldController.handleRequest");
         return null;
     }
  }
```
  + `@Component` : 이 컨트롤러는 /springmvc/old-controller 라는 이름의 스프링 빈으로 등록됌
    + 빈의 이름으로 URL이 매핑된다!
  + HandlerMapping 실행 중 `BeanNameUrlHandlerMapping`가 성공하고, OldController 반환
  + HandlerAdpater의 supports()실행 중 `SimpleControllerHandlerAdapter`가 대상어댑터로 지정되고, OldController를 내부에서 실행


#### :heavy_check_mark: ViewResolver
 + `InternalResourceViewResolver` : 스프링 부트는 이 뷰 리졸버를 자동으로 등록
  + application.properties 에 등록한 `spring.mvc.view.prefix` , `spring.mvc.view.suffix` 설정정보를 사용해서 등록

> ViewResolver 종류
 + 1 = `BeanNameViewResolver` : 빈 이름으로 뷰를 찾아서 반환한다. (예: 엑셀 파일 생성 기능에 사용)
 + 2 = `InternalResourceViewResolver` : JSP를 처리할 수 있는 뷰를 반환

## Spring MVC로 전환

### :pushpin: @RequestMapping
 + 스프링은 @RequestMapping 애노테이션을 사용하는 실용적인 컨트롤러를 만들어냄!

> SpringMemberSaveControllerV1
```java

@Controller
public class SpringMemberSaveControllerV1 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/springmvc/v1/members/save")
    public ModelAndView process(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelAndView mv = new ModelAndView("save-result");
        mv.addObject("member",member);
        return mv;
    }
```
  + `Controller` : @Component + @RequestMapping
    + 스프링이 자동으로 스프링 빈으로 등록(내부에 @Component)
    + 스프링 MVC에선 애노테이션 기반 Controller로 인식
  + `@RequestMapping` : 요청 정보를 매핑, 해당 URL가 호출되면 이 메서드가 호출됌
  + `ModelAndView` : 모델과 뷰 정보를 담아서 반환
  + `mv.addObject()` : 스프링이 제공하는 ModelAndView 를 통해 Model 데이터(뷰 렌더링 시 필요)를 추가할때 사용.
 
### :pushpin: 스프링 MVC - 실용적인 방식

> Controller 통합 버전
```java
@Controller
@RequestMapping("/springmvc/v3/members")
public class SpringMemberControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    //메소드 레벨
    //@RequestMapping(value = "/new-form", method = RequestMethod.GET)
    @GetMapping("/new-form")
    public String newForm(){
        return "new-form";    //view 이름
    }

    //@RequestMapping(value = "/save", method = RequestMethod.POST)
    @PostMapping("/save")
    public String save(
            //request.getParameter("username")
            @RequestParam("username") String username,
            @RequestParam("age") int age,
            Model model) {

        Member member = new Member(username, age);
        memberRepository.save(member);

        model.addAttribute("member", member);
        return "save-result";
    }

    @GetMapping
    public String members(Model model) {
        List<Member> members = memberRepository.findAll();

        model.addAttribute("members", members);
        return "members";
    }
}
```
 + `Model 파라미터` : 모델을 생성하지 않고 인자로 전달받기 
 + `ViewName 직접 반환` : 뷰의 논리 이름을 반환
 + `@RequestParam ` : Http 요청 파라미터를 받으며, request.getParameter("username")과 같게 동작
    + GET 쿼리 파라미터, POST Form 방식을 모두 지원
 + `@GetMapping, @PostMapping` : @RequestMapping은 HTTP Method도 함께 구분 가능
