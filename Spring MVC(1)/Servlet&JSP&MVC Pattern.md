# 서블릿,JSP,MVC 패턴

## 서블릿
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

## JSP
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
  
## MVC 패턴

### :pushpin: 서블릿과 JSP의 한계
+ 서블릿과 JSP만으로 **비즈니스 로직 + 뷰 렌더링**을 모두 처리하면 유지보수가 힘들어지는 문제 발생
+ :star2: **변경의 Life Cycle**이 다른 문제 발생
  + UI수정과 비즈니스 로직을 수정하는 일은 서로에게 영향이 X

### :pushpin:  Model,View,Controller
+ 하나의 서블릿이나, JSP로 처리하던 것을 컨트롤러(Controller)와 뷰(View)라는 영역으로 서로 역할을 나눈 것
+ 웹 애플리케이션은 대부분 MVC 패턴 사용

#### :heavy_check_mark: 1)Controller
+ HTTP 요청을 받아서 파라미터를 검증하고, 비즈니스 로직을 실행
+ 뷰에 전달할 결과 데이터를 조회해서 모델에 담음
+ `비즈니스 로직` : 컨트롤러가 아닌, **Service**라는 계층에서 별도로 처리

#### :heavy_check_mark: 2)Model
+ 뷰에 출력할, 필요한 데이터를 모두 담아둠 => 뷰는 오직 화면 렌더링 역할에만 집중 가능!

#### :heavy_check_mark: 3)View
+ 모델에 담겨있는 데이터를 사용해서 화면을 그림(HTML 생성 역할)

<img src="https://user-images.githubusercontent.com/71436576/127742670-00e8c1c6-fb36-41cd-82ff-7a03f3edbb86.png" width=50% height=50%>

### :pushpin: MVC 패턴의 적용
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

### :pushpin: MVC 컨트롤러의 단점
 + **공통 처리**가 힘듬  ex)forward 중복, viewPath 중복 등..
 + 따라서, 컨트롤러 호출 전에 먼저 공통 기능 처리
   + :star2: **프론트 컨트롤러 패턴**을 도입하여 해결!(스프링 MVC의 핵심)
