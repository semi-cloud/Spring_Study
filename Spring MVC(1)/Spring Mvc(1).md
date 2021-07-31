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

