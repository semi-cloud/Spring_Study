# Spring MVC

## 웹 애플리케이션 이해

### :pushpin: 웹 시스템 구성
:star2: **WEB,WAS,DB**</br>
  + 정적 리소스는 웹 서버가 처리
  + 애플리케이션 로직같은 동적 처리는 WAS에 요청을 위임
  + `장점` 
    + 효율적인 리소스 관리 가능 (각 WEB 서버/WAS 증설)
    + WAS,DB 장애시 WEB 서버가 오류 화면 제공 가능(웹 서버는 잘 죽지 X)
 
> 웹 서버 VS 웹 애플리케이션 서버(WAS)
  + `웹 서버` : HTTP 기반 동작, **정적 리소스** 제공
  + `웹 애플리케이션 서버(WAS)` : 웹 서버 기능 포함 + **애플리케이션 로직/코드** 수행 ex)톰캣
    + 동적 HTML,서블릿,JSP,스프링 MVC...
  
  
### :pushpin: 서블릿
  + 웹 기반의 요청에 대한 동적인 처리가 가능한 Server Side에서 돌아가는 Java Program
    + 웹 애플리케이션 서버를 직접 구현할때 필요한 모든 기능들을 제공
    + ex)TCP/IP 연결대기, HTTP 요청 메시지 파싱, HTTP 응답 메시지 생성 등
    
 ```java
 @WebServlet(name = " helloServlet", urlPatterns = "/hello")
 public class HelloServlet extends HttpServlet {
    
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response){
      //애플리케이션 로직
    }
 }
 ```
  + urlPatterns = "/hello"의 URL 이 호출되면, 서블릿 코드가 실행
  + `HttpServletRequest`: HTTP 요청 정보 사용 가능
  + `HttpServletResponse`: HTTP 응답 정보 제공 가능
    + WAS 는 Response 객체에 담긴 내용으로 HTTP 응답 정보 생성
 
<img src = "https://user-images.githubusercontent.com/71436576/126949533-e0e0e1b7-bc67-4b74-be7a-434659a79c43.png" width=50% height=50%>
  + HTTP 요청시, WAS는 Request, Response 객체를 새로 만들어서 서블릿 객체 호출
 
#### ✔️서블릿 컨테이너
  + 톰캣처럼, 서블릿을 지원하는 **WAS**이며 서블릿 객체를 생성/초기화/호출/종료하는 생명주기 관리
  + 서블릿 객체는 **싱글톤**으로 관리됌
    + 최초 로딩 시점에 서블릿 객체 하나만 생성해두고 재활용 => 고객 요청은 동일한 서블릿 객체 인스턴스에 접근
    + 따라서 **공유 변수 사용에 주의**
  + 동시 요청을 위한 멀티스레드 지원

### :pushpin: 동시요청-멀티 쓰레드

#### ✔️ 쓰레드란?
  + 애플리케이션 코드 하나하나를 순차적으로 실행(한번에 하나의 코드라인만)
    + 동시 처리가 필요하면 **요청 시 마다 신규 쓰레드를 생성**해야함
    
  > 😥 요청 마다 쓰레드 생성의 단점
  + 쓰레드 생성 비용은 매우 비쌈 => 응답 속도가 늦어짐
  + 컨텍스트 스위칭 비용이 발생
  + 쓰레드 생성에 제한이 X=> CPU와 메모리 임계점을 넘어 서버가 죽을 수 있음
    
#### ✔️ 쓰레드 풀
  + `쓰레드 풀` : 필요한 쓰레드를 쓰레드 풀에 보관하고 관리하며, 생성가능한 쓰레드의 최대치 설정 가능(톰캣 => MAX 200)
    + 쓰레드가 필요 : 풀에서 꺼내서 사용하고 종료시 풀에 다시 반납
    + 최대 쓰레드가 모두 사용중인 경우 : 요청 거절 OR 특정 숫자만큼 대기 설정
  + `장점` : 쓰레드 생성/종료하는 비용(CPU)이 절약, 응답 시간이 빨라짐
 
  > WAS 의 주요 튜닝 포인트
  + `최대 쓰레드(MAX THREAD)` 
    + 너무 낮게 설정 : 동시 요청이 많으면, 서버 리소소는 여유롭지만 클라이언트 응답 지연 
    + 너무 높게 설정 : CPU, 메모리 리소스 임계점 초과로 서버 다운
  + `쓰레드 풀의 적정 숫자`
    + 애플리케이션 로직의 복잡도,CPU,메모리,IO 리소스 상황에 따라 모두 다름
    + 성능 테스트가 필요!
  + BUT, WAS 가 멀티 쓰레드 부분을 처리 => 개발자는 멀티 쓰레드 관련 신경 X

### ✔️ HTML,HTTP API,CSR,SSR
  + `정적 리소스` : 고정된 HTML 파일,CSS,JS,이미지,영상 등 브라우저에게 제공
  + `HTML 페이지` : WAS가 동적으로 필요한 HTML 파일 생성해서 브라우저에게 전달
  + `HTTP API`: HTML이 아니라, 데이터 전달(주로 JSON 형식 {KEY : VALUE})
    + UI 클라이언트 접점
      + 앱 클라이언트/웹 브라우저에서 자바 스크립트 통한 HTTP API호출/React,Vue.js
    + 서버 TO 서버
      + 주문 서버 -> 결제 서버/ 기업간 데이터 통신
  
 1)**SSR**</br>
  + `서버 사이드 렌더링(SSR)` : HTML 최종 결과를 서버에서 만들어서, 웹 브라우저에 전달 
  + 주로 정적인 화면에 사용 EX) JSP, 타임리프(백엔드 개발자)
  
 2)**CSR**</br>
  + `클라이언트 사이드 렌더링(CSR)` : HTML 결과를 자바스크립트 이용해 웹브라우저에서 동적으로 생성해 적용
  + 주로 복잡하고 동적인 화면에 사용 EX)React, Vue.js(프론트엔드 개발자)
  
<img src = "https://user-images.githubusercontent.com/71436576/126956781-aaf037b8-69cc-4b73-8bda-0c2de8c81761.png" width=50% height=50%>

## 서블릿

### :pushpin: 프로젝트 생성
  ```java
  @WebServlet(name = "helloServlet", urlPatterns = "/hello")
  public class HelloServlet extends HttpServlet {

      @Override
      protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

          System.out.println("HelloServlet.service");

          String username = request.getParameter("username");
          System.out.println("username = " + username);

          response.setContentType("text/plain");      //헤더 정보(content type)
          response.setCharacterEncoding("utf-8");     //헤더 정보(content type)
          response.getWriter().write("hello " + username);    //http body
      }
  }
  ```
  + 스트링 부트의 내장 톰캣 서버 안에 서블릿 컨테이너가 존재 => helloServlet 객체 생성 
  + `@WebServlet`: 서블릿 애노테이션
    + `name`: 서블릿 이름
    + `urlPatterns`: URL 매핑

### :pushpin: HttpServletRequest 

#### 부가 제공 기능(Request)

 > HTTP 요청 메시지 
  ```
  POST /save HTTP/1.1           //start line
  Host: localhost:8080          //header
  Content-Type: application/x-www-form-urlencoded   //header
  username=kim&age=20        //body
  ```
  + `START LINE` : HTTP 메소드,URL,쿼리 스트링,스키마/프로토콜
  + `HEADER` : 헤더 조회
  + `BODY` :
    + form 파라미터 형식 조회
    + message body 데이터 직접 조회

1) 임시 저장소 기능</br>
  + 해당 HTTP 요청이 시작부터 끝날 때 까지 유지되는 임시 저장소 기능
    + 저장: `request.setAttribute(name, value)`
    + 조회: `request.getAttribute(name)`
    
2) 세션 관리 기능</br>
  + `request.getSession(create: true)`

### :pushpin: HTTP 요청 데이터

#### ✔️ GET 쿼리 파라미터
  + `request.getParameter` : 쿼리 파라미터 조회
```java
@WebServlet(name = "requestParamServlet", urlPatterns = "/request-param")
public class RequestParamServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //모든 파라미터 조회
        request.getParameterNames().asIterator()
            .forEachRemaining(paramName -> System.out.println(paramName +  "=" + request.getParameter(paramName)));

        //단일 파라미터 조회
        String username = request.getParameter("username");
        String age = request.getParameter("age");
        System.out.println("username = " + username);
        System.out.println("age = " + age);

        //이름이 같은 복수 파라미터 조회
        String[] usernames = request.getParameterValues("username"); //여러개 있을때 사용
        for (String name : usernames) {
            System.out.println("name = " + name);
        }
    }
}
```

#### ✔️ POST HTML Form
 + POST 형식으로 보낼때, HTTP 메시지에는 `content-type` 이 지정됌(메시지 바디에 데이터가 존재)
   + `content-type` : GET 에서 살펴본 쿼리 파라미터 형식과 같음
      + message body: username=hello&age=20
 + 따라서, 마찬가지로 `request.getParameter()` 이용해 Form을 통해 전송한 데이터 가져오기 가능!

#### ✔️ API 메시지 바디-단순 텍스트
 + 'HTTP message body'에 직접 데이터를 담아서 요청(주로 JSON)
   + POST, PUT, PATCH 에서 사용
 
 ```java
 @WebServlet(name = "requestBodyStringServlet", urlPatterns = "'/request-body-string")
public class RequestBodyStringServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ServletInputStream inputStream = request.getInputStream();   //바이트 코드
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);//문자열로 변환

        System.out.println("messageBody = " + messageBody);

        response.getWriter().write("ok");
    }
}
```
  + `inputStream` : byte 코드를 반환하기 때문에 문자로 변환(Charset 지정 필요)

#### ✔️ API 메시지 바디-JSON
> 요청 메시지
+ content-type: application/json (Body raw, 가장 오른쪽에서 JSON 선택)
+ message body: {"username": "hello", "age": 20}

```java
@WebServlet(name = "requestBodyJsonServlet", urlPatterns = "/request-body-json")
public class RequestBodyJsonServlet extends HttpServlet {

    private com.fasterxml.jackson.databind.ObjectMapper objectMapper = new ObjectMapper();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ServletInputStream inputStream = request.getInputStream();   //바이트 코드
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);//문자열로 변환

        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);

        response.getWriter().write("ok");
    }
}
```
  + `ObjectMapper` : JSON 변환 라이브러리
  
### :pushpin: HTTPServletResponse

#### :heavy_check_mark:기본 사용법
```@WebServlet(name = "responseHeaderServlet", urlPatterns = "/response-header")
public class ResponseHeaderServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //[status-line]
        response.setStatus(HttpServletResponse.SC_OK);

        //[response-headers]
        response.setHeader("Content-Type", "text/plain;charset=utf-8");
        response.setHeader("Cache-Control", "no-cache ,no-store, must-revalidate" );
        response.setHeader("Pragma", "no-cache");
        response.setHeader("my-header", "hello");

        //[header 편의 메서드]
        content(response);
        cookie(response);
        redirect(response);

        //[message body]
        PrintWriter writer = response.getWriter();
        writer.println("ok");
    }
}
```
> Content 편의 메서드
```java
 private void content(HttpServletResponse response) {
        //Content-Type: text/plain;charset=utf-8
        //Content-Length: 2
        //response.setHeader("Content-Type", "text/plain;charset=utf-8");
        response.setContentType("text/plain");
        response.setCharacterEncoding("utf-8");
        //response.setContentLength(2); //(생략시 자동 생성)
    }
```
> 쿠키 편의 메서드
```java
  private void cookie(HttpServletResponse response) {
        //response.setHeader("Set-Cookie", "myCookie=good; Max-Age=600");
        Cookie cookie = new Cookie("myCookie", "good");
        cookie.setMaxAge(600); //600초
        response.addCookie(cookie);
    }
```
> redirect 편의 메서드
```java
   private void redirect(HttpServletResponse response) throws IOException {
        //response.setStatus(HttpServletResponse.SC_FOUND); //302
        //response.setHeader("Location", "/basic/hello-form.html");
        response.sendRedirect("/basic/hello-form.html");
    }
```

#### :heavy_check_mark: HTTP 응답 데이터-html/json
> html 형식
  + response.setContentType("text/html")
> json 형식
 + response.setContentType("application/json")
 +  objectMapper 이용해서 객체를 JSON 문자로 변환
```java
@WebServlet(name="responseJsonServlet", urlPatterns = "/response-json")
public class ResponseJsonServlet extends HttpServlet {

    private ObjectMapper objectMapper = new ObjectMapper();
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("application/json");
        response.setCharacterEncoding("utf-8");

        HelloData helloData = new HelloData();
        helloData.setUsername("kim");
        helloData.setAge(20);

        String result = objectMapper.writeValueAsString(helloData);
        response.getWriter().write(result);
    }
}
```

## 서블릿,JSP,MVC 패턴

### :pushpin: 서블릿
 + 뷰(View)화면을 위한 HTML을 만드는 작업이 자바 코드에 섞여서, 지저분하고 복잡해지는 문제 발생

> Member 저장 서블릿
```java
@WebServlet(name="memberSaveServlet", urlPatterns = "/servlet/members/save")
public class MemberSaveServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        //비즈니스 로직
        Member member = new Member(username, age);
        memberRepository.save(member);

        //응답(동적 html)
        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");
        PrintWriter writer = response.getWriter();

        writer.write("<html>\n" +
                "<head>\n" +
                " <meta charset=\"UTF-8\">\n" +
                "</head>\n" +
                "<body>\n" +
                "성공\n" +
                "<ul>\n" +
                " <li>id="+member.getId()+"</li>\n" +
                " <li>username="+member.getUsername()+"</li>\n" +
                " <li>age="+member.getAge()+"</li>\n" +
                "</ul>\n" +
                "<a href=\"/index.html\">메인</a>\n" +
                "</body>\n" +
                "</html>");
    }
}
```
  + 멤버 객체를 사용해서 결과 하면용 HTML을 동적으로 만들어서 응답
  + BUT 자바 코드로 HTML을 만드는 것은 매우 비효율적
    + `템플릿 엔진` : HTML 문서에 동적으로 변경해야 하는 부분만 자바 코드로 변경 가능! ex)JSP,Thymeleaf..

### :pushpin: JSP
 + 뷰를 생성하는 HTML 작업에만 집중하고, 중간중간 동적으로 변경이 필요한 부분에만 자바 코드를 적용

> save.jsp
```java
<%@ page import="hello.servlet.domain.member.MemberRepository" %>
<%@ page import="hello.servlet.domain.member.Member" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    //request,response 사용 가능
    MemberRepository memberRepository = MemberRepository.getInstance();

    String username = request.getParameter("username");
    int age = Integer.parseInt(request.getParameter("age"));

    //비즈니스 로직
    Member member = new Member(username, age);
    memberRepository.save(member);

%>
<html>
<head>
    <title>Title</title>
</head>
<body>
성공
<ul>
    <li>id=<%=member.getId()%></li>
    <li>username=<%=member.getUsername()%></li>
    <li>age=<%=member.getAge()%></li>
</ul>
</body>
</html>
```
  + `<% ~~ %>` : 자바 코드 입력 가능한 부분

### :pushpin: MVC 패턴

#### 서블릿과 JSP의 한계
+ 서블릿과 JSP만으로 **비즈니스 로직 + 뷰 렌더링**을 모두 처리하면 유지보수가 힘들어지는 문제 발생
+ :star2: **변경의 Life Cycle**이 다른 문제 발생
  + UI수정과 비즈니스 로직을 수정하는 일은 서로에게 영향이 X

#### :heavy_check_mark: Model,View,Controller
+ 하나의 서블릿이나, JSP로 처리하던 것을 컨트롤러(Controller)와 뷰(View)라는 영역으로 서로 역할을 나눈 것
+ 웹 애플리케이션은 대부분 MVC 패턴 사용

**1)Controller**</br>
+ HTTP 요청을 받아서 파라미터를 검증하고, 비즈니스 로직을 실행
+ 뷰에 전달할 결과 데이터를 조회해서 모델에 담음
+ `비즈니스 로직` : 컨트롤러가 아닌, **Service**라는 계층에서 별도로 처리

**2)Model**</br>
+ 뷰에 출력할, 필요한 데이터를 모두 담아둠 => 뷰는 오직 화면 렌더링 역할에만 집중 가능!

**3)View**</br>
+ 모델에 담겨있는 데이터를 사용해서 화면을 그림(HTML 생성 역할)

<img src="https://user-images.githubusercontent.com/71436576/127742670-00e8c1c6-fb36-41cd-82ff-7a03f3edbb86.png" width=50% height=50%>

#### MVC 패턴의 적용
+ Controller : 서블릿
+ View : JSP
+ Model : HttpServletRequest 객체 사용
  + `request.setAttribute()` : 데이터 보관
  + `request.getAttribute()` : 데이터 조회

> 회원 등록 폼-컨트롤러
```java
//WEB-INF : 외부에서 직접적으로 new-form.jsp 파일을 경로로 접근 불가능, 항상 controller(servlet)통해서 가능함
@WebServlet(name="mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/new-form")
public class MvcMemberFormServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);      //다른 서블릿,JSP 로 이동

    }
}
```
 + `dispatcher.forward` : 다른 서블릿이나 JSP로 이동, **서버 내부에서 다시 호출이 일어남**
   + `redirect` : 실제 클라이언트에게 응답이 나갔다가 다시 서버로 돌아옴 => 클라이언트 인지 가능, URL 경로 변경
 + `/WEB-INF` : 외부에서 직접적으로 JSP 파일을 경로로 접근 불가능, 항상 controller(servlet)통해서 호출 가능

> 회원 등록 폼 JSP
```jsp
<!-- 상대경로 사용, [현재 URL이 속한 계층 경로 + /save] -->
<form action="save" method="post">
    username: <input type="text" name="username" />
    age: <input type="text" name="age" />
    <button type="submit">전송</button>
</form>
```
  + 절대 경로가 아닌, 상대경로
    + 폼 전송시 ` /servlet-mvc/members/` => `/servlet-mvc/members/save` 으로 자동 변경

> 회원 저장 컨트롤러
```java
@WebServlet(name = "mvcMemberSaveServlet", urlPatterns = "/servlet/members/save")
public class MvcMemberSaveServlet extends HttpServlet {

    MemberRepository memberRepository = MemberRepository.getInstance();
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        //비즈니스 로직
        Member member = new Member(username, age);
        memberRepository.save(member);

        //Model에 데이터를 보관
        request.setAttribute("member", member);

        String viewPath = "/WEB-INF/views/save-result.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request,response);

    }
}
```
  + `setAttribute()` 로 request 객체에 데이터 보관해서 뷰에 전달
  + `request.getAttribute()` 로 뷰에서 데이터 꺼내기 => **${member.id}** 형태로 변경 가능

#### MVC 컨트롤러의 단점
 + **공통 처리**가 힘듬  ex)forward 중복, viewPath 중복 등..
 + 따라서, 컨트롤러 호출 전에 먼저 공통 기능 처리
   + :star2: **프론트 컨트롤러 패턴**을 도입하여 해결!(스프링 MVC의 핵심)

## MVC 프레임워크 생성

### Front Controller
  + 프론트 컨트롤러(Servlet) 하나로 클라이언트의 요청을 받고, 프론트는 요청에 맞는 컨트롤러를 찾아서 호출
    + 프론트를 제외한 나머지 컨트롤러 => 서블릿 사용 필요성 X
 
<img src="https://user-images.githubusercontent.com/71436576/127743885-20e4f0e1-6bc2-4465-a92e-f5b2285f16bc.png" width=50% height=50%>

#### :heavy_check_mark: 프론트 컨트롤러 도입-V1
**1)기존 구조를 최대한 유지하면서, 프론트 컨트롤러 도입**</br>
 
> Controller 인터페이스
```java
public interface ControllerV1 {
    void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```

> 인터페이스 구현한 각각 컨트롤러 클래스
```java
public class MemberFormControllerV1 implements ControllerV1 {
    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);      //다른 서블릿,JSP 로 이동
    }
}
```

> Front Controller 클래스
```java
@WebServlet(name="frontControllerServletV1", urlPatterns = "/front-controller/v1/*" )
public class FrontControllerServletV1 extends HttpServlet {

    private Map<String, ControllerV1> controllerMap = new HashMap<>();

    public FrontControllerServletV1() {
        controllerMap.put("/front-controller/v1/members/new-from", new MemberFormControllerV1());
        controllerMap.put("/front-controller/v1/members/save", new MemberSaveControllerV1());
        controllerMap.put("/front-controller/v1/members", new MemberListControllerV1());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        //front-controller/v1/members
        String requestURI = request.getRequestURI();

        //**다형성 => 부모(인터페이스)에 자식이 담길 수 있음!
        ControllerV1 controller = controllerMap.get(requestURI);
        if(controller == null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        //**다형성 => override 된 process 호출됌
        controller.process(request,response);
    }
}
```
  + `/front-controller/v1/*` : /front-controller/v1 를 포함한 하위 모든 요청은 이 서블릿에서 받아들임
  + `ControllerMap`
    + `key` : 매핑 URL
    + `value` : 호출될 컨트롤러
  + `service()` : requestURI를 조회해서 실제 호출할 컨트롤러를 MAP에서 찾은 후, process()를 통해 컨트롤러 실행


#### :heavy_check_mark: View 분리-V2
**1)단순 반복 되는 뷰 로직 분리**</br>
  + 모든 컨트롤러에서 뷰로 이동하는 부분에 중복이 존재 => **별도로 뷰 처리하는 객체 생성**

<img src="https://user-images.githubusercontent.com/71436576/127867213-357edbbe-57f7-44fc-9c68-b1e60f6380fe.png" width=50% height=50%>

> 뷰 처리 객체 MyView
```java
public class MyView {
    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    //JSP 렌더링
    public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request,response);
    }
}
```

> Controller 인터페이스
```java
public interface ControllerV2 {
    //MyView(뷰 객체)를 반환
    MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```

> 인터페이스 구현 Controller 클래스
```java
public class MemberSaveControllerV2 implements ControllerV2 {
    MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        //비즈니스 로직
        Member member = new Member(username, age);
        memberRepository.save(member);

        //Model에 데이터를 보관
        request.setAttribute("member", member);

        //경로 넣어서 MyView 반환
        return new MyView("/WEB-INF/views/save-result.jsp");
    }
}
```
> Front Controller 중 일부
```java
@Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ...
        MyView view = controller.process(request, response);
        view.render(request,response);     //forward 로직 수행
    }
```
#### :heavy_check_mark: Model 추가-V3

<img src="https://user-images.githubusercontent.com/71436576/127869644-d4b88c4d-3df0-4378-a7e2-9de4907f14be.png" width=50% height=50%>

**1) Servlet 종속성 제거 : ModelView 추가**</br>
 + 컨트롤러에서 HttpServletRequest를 사용, request.setAttribute()를 통해 데이터 저장하고 뷰에 전달(종속적)
 + :star2: `ModelView` : 위 두 기능을 대체할 객체, 뷰이름과 뷰에 전달할 Model 데이터 포함

> ModelView
```java
public class ModelView {
  
    private String viewName;      //view 이름  

    private Map<String,Object> model = new HashMap<>();  //뷰 렌더링 시 필요한 model 객체

    public ModelView(String viewName) {
        this.viewName = viewName;
    }

    public String getViewName() {
        return viewName;
    }

    public void setViewName(String viewName) {
        this.viewName = viewName;
    }

    public Map<String, Object> getModel() {
        return model;
    }

    public void setModel(Map<String, Object> model) {
        this.model = model;
    }
}
```
  + `model` : 뷰를 렌더링 할때 필요한 데이터를 (key,value)형태로 저장
  
> Controller 인터페이스
```java
public interface ControllerV3 {
    //서블릿에 종속적 X
    ModelView process(Map<String, String> paramMap);
}
```
  + HttpServletRequest가 제공하는 파라미터 => Front Controller에서 paramMap에 담아서 호출
  
> 인터페이스 구현 controller
```java
public class MemberSaveControllerV3 implements ControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();
    @Override
    public ModelView process(Map<String, String> paramMap) { 
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);
        
        ModelView mv = new ModelView("save-result");
        mv.getModel().put("member", member);
        return mv;
    }
}
``` 
  + `paramMap` : 요청 파라미터의 정보가 담겨 있음
  + `save-result` : view의 논리적 이름으로 지정, 물리적 이름은 Front Controller에서 처리
  
> Front Controller 중 일부
```java
@Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        ...
        //paramMap을 넘겨주기(모든 파라미터 정보)
        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);

        String viewName = mv.getViewName();  //논리 이름 가져오기
        MyView view = viewResolver(viewName);

        view.render(mv.getModel(),request,response);  //모델(렌더링시 필요한 데이터 존재)도 함께 넘겨주기
    }

    //뷰의 논리이름으로 물리 경로를 생성해서 반환
    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }

    //모든 파라미터 정보 가져와서 Map에 담는 함수
    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                    .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
```
  + 맨 앞의 Front 컨트롤러에서만 servlet에 종속적이기 때문에, 여기서 모든 파라미터 정보를 Map에 담아 넘겨줘야함
  + 나머지 컨트롤러에서는 <뷰이름 + 렌더링 시 필요한 데이터>가 담긴 ModelView를 반환하는 역할만 함
 
**2) 뷰 이름 중복 제거 : viewResolver 추가**</br>
  + 컨트롤러가 반한환 논리 뷰 이름을 변환 후, 실제 물리 뷰 경로가 있는 MyView 객체 반환
    + `MyView`: HTML 화면을 렌더링
  + `view.render(mv.getModel(),request,response)`: 이후 렌더링 시 필요한 데이터가 담긴 model도 함께 넘겨줘야함
   
> MyView 중 일부
```java
public class MyView {
    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }    
    ...
    public void render(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        modelToRequestAttribute(model, request);
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request,response);
    }

    private void modelToRequestAttribute(Map<String, Object> model, HttpServletRequest request) {
        model.forEach((key, value) -> request.setAttribute(key,value));
    }
}
```

#### :heavy_check_mark: 단순,실용적 컨트롤러-V4
+ 구현 입장에서 ModelView를 직접 생성해서 반환하지 않도록 편리한 인터페이스 제공
+ **컨트롤러가 ModelView 를 반환하지 않고, ViewName 만 반환!**

> Controller Interface
```java
public interface ControllerV4 {
    String process(Map<String,String> paramMap, Map<String, Object> model);
}
```
  + model 객체가 외부에서 전달되기 때문에 그냥 사용하면 되고, 뷰 이름만 반환 

> Interface 구현 Controller
```java
public class MemberSaveControllerV4 implements ControllerV4 {

    private MemberRepository memberRepository = MemberRepository.getInstance();
    @Override
    public String process(Map<String, String> paramMap, Map<String, Object> model) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);
        
        //변경 부분(모델 객체 생성 X)
        model.put("member", member);
        return "save-result";       //뷰 이름만 반환
    } 
```
> Front Controller 중 일부
```java
 @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ...
        Map<String, String> paramMap = createParamMap(request);
        Map<String, Object> model = new HashMap<>();   //추가

        String viewName = controller.process(paramMap, model);   //파라미터와 함께 model(빈 객체)도 함께 전달

        MyView view = viewResolver(viewName);      //뷰 논리 이름을 직접 반환하기 때문에 바로 사용
        view.render(model,request,response);
    }
```

#### :heavy_check_mark: 유연한 컨트롤러-V5
+ Adapter 도입 => 프레임워크를 유연하고 확장성 있게 설계
+ Front Controller가 한가지 방식이 아닌, 다양한 방식의 컨트롤러를 처리 가능하도록 변경!(ControllerV3,V4..)
<img src="https://user-images.githubusercontent.com/71436576/127876210-185a98f3-1227-4853-80fb-c4dcb3d16674.png" width=50% height=50%>

**1) Handler Adapter**</br>

> Handler Adapter 인터페이스
``` java
public interface MyHandlerAdapter {

    boolean supports(Object handler);  //handler == controller

    ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;
}
```
  + `supports()` : Adapter가 해당 컨트롤러를 처리 할 수 있는지 판단하는 메서드
  + `handle()` :  Front Controller가 아닌 **Adapter가 실제 컨트롤러를 호출**하고, 결과로 ModelView 반환

> ControllerV3HandlerAdapter.java
```java
public class ControllerV3HandlerAdapter implements MyHandlerAdapter {
    @Override
    public boolean supports(Object handler) {         //ControllerV3 인터페이스를 구현했는지 체크
        return (handler instanceof ControllerV3);
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        ControllerV3 controller = (ControllerV3) handler;     //캐스팅

        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);         //Adapter에서 실제 컨트롤러 호출

        return mv;          //ModelView 반환
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }  
}
```

**2) Hadler**</br>
 + 컨트롤러를 직접 매핑해서 사용하지 않고, Adpater가 지원하는 한에서 어떤 것이라도 URL에 매핑해서 사용 가능(더 넓은 범위)

> Front Controller
```java
@WebServlet(name = "frontControllerServletV5", urlPatterns = "/front-controller/v5/*")
public class FrontControllerServletV5 extends HttpServlet {

    //모든 컨트롤러 지원(ControllerV4 -> Object)
    private final Map<String, Object> handlerMappingMap = new HashMap<>();      //핸들러 매핑(컨트롤러 매핑) 저장
    private final List<MyHandlerAdapter> handlerAdapters = new ArrayList<>();   //핸들러 어댑터 리스트

    public FrontControllerServletV5(){
        initHandlerMappingMap();     //핸들러 매핑 초기화
        initHandlerAdapters();       //어댑터 초기화
    }

    private void initHandlerMappingMap() {
        handlerMappingMap.put("/front-controller/v5/v3/members/new-from", new MemberFormControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members/save", new MemberSaveControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members", new MemberListControllerV3());

        handlerMappingMap.put("/front-controller/v5/v4/members/new-from", new MemberFormControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members/save", new MemberSaveControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members", new MemberListControllerV4());
    }
    
    private void initHandlerAdapters() {
        handlerAdapters.add(new ControllerV3HandlerAdapter());
        handlerAdapters.add(new ControllerV4HandlerAdapter());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Object handler = getHandler(request);              //1.핸들러 매핑

        if(handler == null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        MyHandlerAdapter adapter = getHandlerAdapter(handler);   //2.핸들러 처리 가능한 어댑터 조회
        ModelView mv = adapter.handle(request, response, handler);   //3.실제 어댑터 호출(controller.process()과정)

        String viewName = mv.getViewName();       
        MyView view = viewResolver(viewName);      //4.논리 -> 물리 주소 변환

        view.render(mv.getModel(),request,response);  //5.뷰 렌더링(모델뷰 같이 전달)
    }

    private Object getHandler(HttpServletRequest request) {   
        String requestURI = request.getRequestURI();
        return handlerMappingMap.get(requestURI);
    }
    
    private MyHandlerAdapter getHandlerAdapter(Object handler) { 
        MyHandlerAdapter a;
        for (MyHandlerAdapter adapter : handlerAdapters) {
            if(adapter.supports(handler)){
                return adapter;
            }
        }
        throw new IllegalArgumentException("handler adapter 를 찾을 수 없습니다. handlers" + handler);
    }
    
    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }
}
```
  + `getHandler()` : URL에 매핑된 핸들러 객체 찾아서 반환
  + `getHandlerAdapter()` : 핸들러 처리 가능한 어댑터 찾기

> ControllerV4HandlerAdapter 중 일부
```java
    ModelView mv = new ModelView(viewName);

    mv.setModel(model);
    return mv;
```
 + :star2: Controllerv4는 모델뷰가 아닌 뷰의 '이름'을 반환해야함 => `Adpater` 덕분에 형식을 맞추어 반환 가능!

## Spring MVC

### :pushpin: Spring MVC 전체 구조

#### 직접 만든 프레임워크 vs 스프링 MVC
 + FrontController -> `DispatcherServlet`
 + handlerMappingMap -> `HandlerMapping`(Interface)
 + MyHandlerAdapter ->  `HandlerAdapter`(Interface)
 + ModelView -> `ModelAndView`
 + viewResolver -> `ViewResolver`(Interface)
 + MyView -> `View` (Interface)

#### :heavy_check_mark: 스프링 MVC 구조와 동작 순서
<img src="https://user-images.githubusercontent.com/71436576/127876210-185a98f3-1227-4853-80fb-c4dcb3d16674.png" width=50% height=50%>

1) **핸들러 조회**: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회</br>
2) **핸들러 어댑터 조회**: 핸들러를 실행할 수 있는 핸들러 어댑터를 조회</br>
3) **핸들러 어댑터 실행**: 핸들러 어댑터를 실행</br>
4) **핸들러 실행**: 핸들러 어댑터가 실제 핸들러를 실행</br>
5) **ModelAndView 반환**: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환</br>
6) **viewResolver 호출**: 뷰 리졸버를 찾고 실행</br>
7) **View 반환**: 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환</br>
8) **뷰 렌더링**: 뷰를 통해서 뷰를 렌더링</br>

#### :heavy_check_mark: DispatcherServlet
  + 부모 클래스에서 HttpServlet 상속받아 사용(Servlet으로 동작)
    + DispatcherServlet -> FrameworkServlet -> HttpServletBean -> HttpServlet
  + Spring Boot는 `DispatcherServlet`을 서블릿으로 '자동 등록'하면서, 모든경로에 대해 매핑( urlPatterns="/" )

**요청 흐름**
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

### :pushpin: Spring MVC로 전환

#### :heavy_check_mark: @RequestMapping
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
 
#### :heavy_check_mark: 스프링 MVC - 실용적인 방식

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
 
## Spring MVC-기본 기능

### :pushpin: Logging
 + 운영 시스템에서는 System.out.println() 같은 시스템 콘솔 대신, 로깅 라이브러리를 사용해 로그를 출력
  + `SLF4J ` : 스프링 부트 기본 로깅 라이브러리
 
> LogTestController
```java
//@Slf4j             //로그 출력 라이브러리
@RestController      //RestAPI
public class LogTestController {
    private final Logger log = LoggerFactory.getLogger(getClass());

    @RequestMapping("/log-test")
    public String LogTest() {
        String name = "Spring";

        //순서대로(레벨 낮 => 높은 순서대로)
        //log.trace("trace log =" + name);  //불필요한 연산이 trace 출력 전에 일어남
        log.trace("trace log={}", name);
        log.debug("trace log={}", name);    //개발 서버
        log.info("trace log={}", name);     //운영 서버(기본은 info 부터)
        log.warn("trace log={}", name);
        log.error("trace log={}", name);

        return "ok";    //view 이름이 아니라 문자 그대로 반환
    }
}
```
  + `RestConroller` : 반한 값이 String 인 경우, 뷰를 찾는 것이 아닌 HTTP 메시지 바디에 직접 입력
  
#### :heavy_check_mark: 로그 레벨과 사용법
+ `LEVEL`: **TRACE > DEBUG > INFO > WARN > ERROR**
 + 개발 서버는 debug 출력
 + 운영 서버는 info 출력
```
    #전체 로그 레벨 설정(기본 info)
    logging.level.root=info
    
    #hello.springmvc 패키지와 그 하위 로그 레벨 설정
    logging.level.hello.springmvc=debug
```
+ 사용법 : `log.debug("data={}", data)` 
  + ~~log.debug("data="+data) : 로그 출력 레벨과 무관하게 불필요한 연산이 발생~~

#### :heavy_check_mark: 로그 사용시 장점
+ 쓰레드 정보, 클래스 이름 같은 부가 정보도 볼 수 있음
+ 로그 레벨에 따라 로그를 서버의 종류에 맞게 조절 가능 (운영 서버, 개발 서버..)
+ 콘솔 뿐만 아니라, 파일이나 네트워크와 같은 별도의 위치에 남길 수 있음
+ 성능도 일반 System.out보다 좋음(내부 버퍼링, 멀티 쓰레드 등등) 
  + :star2: 실무에서는 무조건 Log를 사용해야함!
  
 
### :pushpin: HTTP 요청 매핑

> 기본
```java
  @GetMapping("/hello-basic")
    public String helloBasic(){
        log.info("helloBasic");
        return "ok";
    }
```
> PathVariable(경로 변수) 사용
```java
    //URL 경로 변수(쿼리 파라미터 방식 아님)

    @GetMapping("/mapping/{userId}")
    public String mappingPath(@PathVariable("userId") String data){
        log.info("mappingPath userId={}", data);
        return "ok";
    }
```
  + 최근 HTTP API는 다음과 같이 **리소스 경로에 식별자를 넣는 스타일**을 선호!
  + `@PathVariable`: @RequestMapping이 템플릿화 한 URL 경로와 매칭된 부분을 조회
    + @PathVariable 의 이름과 파라미터 이름이 같으면 생략 가능

> 미디어 타입 조건 매핑 - HTTP 요청 Content-Type, consume
```java
    @PostMapping(value = "/mapping-consume", consumes = MediaType.APPLICATION_JSON_VALUE)
    public String mappingConsumes() {
        log.info("mappingConsumes");
        return "ok";
    }
```
 + `consumes` : **HTTP 요청의 Content-Type 헤더**를 기반으로 미디어 타입으로 매핑
   + 맞지 않으면, HTTP 415 상태코드(Unsupported Media Type)을 반환

> 미디어 타입 조건 매핑 - HTTP 요청 Accept, produce
```java
    @PostMapping(value = "/mapping-produce", produces = "text/html")
    public String mappingProduces() {
        log.info("mappingProduces");
        return "ok";
    }
```
  + `produces` : **HTTP 요청의 Accept 헤더**를 기반으로 미디어 타입으로 매핑
    + 맞지 않으면, HTTP 406 상태코드(Not Acceptable)을 반환
 
## HTTP 요청/응답 데이터 조회

### :pushpin: HTTP 요청- 기본, 헤더 조회

#### 회원 관리 API(매핑)
+ 회원 목록 조회: GET /users
+ 회원 등록: POST /users
+ 회원 조회: GET /users/{userId}
+ 회원 수정: PATCH /users/{userId}
+ 회원 삭제: DELETE /users/{userId}

#### HTTP 요청 정보 조회
> HTTP 요청 정보들 조회하는 Controller
```java
@Slf4j
@RestController
public class RequestHeaderController {

    @RequestMapping("/headers")
    public String headers(HttpServletRequest request,
                          HttpServletResponse response,
                          HttpMethod httpMethod,
                          Locale locale,
                          @RequestHeader MultiValueMap<String, String> headerMap,
                          @RequestHeader("host") String host,
                          @CookieValue(value = "myCookie", required = false) String cookie

                          ){
        log.info("request={}", request);
        log.info("response={}", response);
        log.info("httpMethod={}", httpMethod);
        log.info("locale={}", locale);
        log.info("headerMap={}", headerMap);
        log.info("header host={}", host);
        log.info("myCookie={}", cookie);
        return "ok";
    }
}
```
+ `@RequestHeader MultiValueMap<String, String>` : 모든 HTTP 헤더를 MultiValueMap 형식으로 조회
  + :question: MultiValueMap : 하나의 키에 여러값을 받을 수 있음  
+ `@RequestHeader("host") String host` : 특정 HTTP 헤더 조회


### :pushpin: HTTP 요청 파라미터-쿼리,HTML FORM

> 잠깐! 먼저 기억하고 가야 할 것이 있음

🌱 **클라이언트에서 서버로 요청 데이터를 전달할 때**
 **1) GET- 쿼리 파라미터**</br>
  + /url?username=hello&age=20
  + 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달 
  
 **2) POST- HTML Form**</br>
  + content-type: application/x-www-form-urlencoded
  + 메시지 바디에 쿼리 파리미터 형식으로 전달 username=hello&age=20
  
 **3) HTTP message body**</br>
  + HTTP API에서 주로 사용, JSON, XML, TEXT (주로 json 형식)
  + POST, PUT, PATCH

:star2:**결국 GET 이던, POST던 둘다 요청 파라미터(Request Parameter) 조회**
 
> requestParamV1
```java
@Slf4j
@Controller
public class RequestParamController {

    @RequestMapping("/request-param-v1")
    public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));
        log.info("username={}, age={}", username, age);

        response.getWriter().write("ok");

    }
}
```
#### :heavy_check_mark: HTTP 요청 파라미터 - @RequestParam
+ 스프링에서 제공하는 `@RequestParam`을 통해 편리하게 사용 가능
+ String, int, Integer와 같은 단순 타입에 사용!

> requestParamV2
```java
    //v2
    @ResponseBody 
    @RequestMapping("/request-param-v2")
    public String requestParamV2(
            @RequestParam("username") String memberName,
            @RequestParam("age") int memberAge
    ){
        log.info("username={}, age={}", memberName, memberAge);
        return "ok";
    }
```
 + `@RequestParam` : 파라미터 이름으로 바인딩
   + request.getParameter("username")와 똑같이 동작!
   + HTTP 파라미터 이름이 변수 이름과 같으면 생략 가능 ex)String username, int age
 + `@ResponseBody` : View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력


>  requestParamReuquired-파라미터 필수 여부
```java
    //int = null X 객체타입만 null 가능해서 Integer 사용해야해!
    @ResponseBody  
    @RequestMapping("/request-param-required")
    public String requestParamReuquired(
            @RequestParam(required = true) String memberName,
            @RequestParam(required = false) Integer memberAge) {
        log.info("username={}, age={}", memberName, memberAge);
        return "ok";
    }
```
 + `@RequestParam(required = true)` : 파라미터 필수 여부(기본값)
 + 주의 1 : '/request-param?username=' 형태로 들어오면 빈 문자로 에러가 나지 않음
 + 주의 2 : `(required = false)`인 경우, null 값을 받을 수 있는 Integer 변경 혹은 defaultValue 사용 해야함
   + @RequestParam(required = true, **defaultValue = "guest"**) String username
   + @RequestParam(required = false, **defaultValue = "-1"**) int age


> requestParamMap-파라미터 Map으로 조회
```java
    @ResponseBody  //==restController
    @RequestMapping("/request-param-default")
    public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
        log.info("username={}, age={}", paramMap.get("username"), paramMap.get("age"));
        return "ok";
    }
```

#### :heavy_check_mark: HTTP 요청 파라미터 - @ModelAttribute

+ 요청 파라미터 받아서 필요한 객체 만들고, 값을 객체에 넣어주는 과정을 자동화해줌
+ String, int, Integer와 같은 단순 타입 외의 나머지 타입에 사용!

> HelloData
```java
@Data
public class HelloData {
    private String username;
    private int age;

}
```
 + 롬복 `@Data` : `@Getter` , `@Setter` , `@ToString` , `@EqualsAndHashCode` , `@RequiredArgsConstructor` 를 자동으로 적용해줌
 
> ModelAttributeV1
```java
    //ModelAttribute 생략 가능
    @ResponseBody
    @RequestMapping("/model-attribute-v1")
    public String modelAttributeV1(@ModelAttribute HelloData helloData){
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
        return "ok";
    }
```
 + `@ModelAttribute ` : HelloData 객체의 생성 + 객체에 값을 넣어줌
   + 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾은 후, 해당 프로퍼티의 setter를
호출해서 파라미터의 값을 입력(바인딩)

### :pushpin: HTTP 요청 파라미터-HTTP API
 + HTTP 메시지 바디를 통해 데이터가 직접 데이터가 넘어오는 경우는 @RequestParam , @ModelAttribute 를 사용 불가능! 

#### :heavy_check_mark: 단순 텍스트인 경우
> 단순 텍스트 
```java
@Slf4j
@Controller
public class RequestBodyStringController {

    @PostMapping("/request-body-string-v1")
    public void requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException {
        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("messageBody={}", messageBody);

        response.getWriter().write("ok");
    }

    @PostMapping("/request-body-string-v2")
    public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {

        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
        log.info("messageBody={}", messageBody);
        responseWriter.write("ok");
    }

    //message body 에 있는 객체 자동 String 변환 <String>
    //헤더 정보도 조회 가능
    @PostMapping("/request-body-string-v3")
    public HttpEntity requestBodyStringV3(HttpEntity<String> httpEntity) throws IOException {

        String messageBody = httpEntity.getBody();
        log.info("messageBody={}", messageBody);

        return new HttpEntity<>("ok");
    }

    @ResponseBody
    @PostMapping("/request-body-string-v4")
    public String requestBodyStringV4(@RequestBody String messageBody) throws IOException {

        log.info("messageBody={}", messageBody);
        return "ok";
    }




}
```

#### :heavy_check_mark: JSON 형식인 경우


### :pushpin: HTTP 응답-정적 리소스,템플릿


### :pushpin: HTTP 응답-HTTP API

### :pushpin: HTTP 메시지 컨버터

### :pushpin: 요청 매핑 핸들러 어댑터 구조

## 




