# Servlet

## 프로젝트 생성
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

## HttpServletRequest 

### 부가 제공 기능(Request)

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

#### ✔️ 1) 임시 저장소 기능
  + 해당 HTTP 요청이 시작부터 끝날 때 까지 유지되는 임시 저장소 기능
    + 저장: `request.setAttribute(name, value)`
    + 조회: `request.getAttribute(name)`
    
#### ✔️ 2) 세션 관리 기능
  + `request.getSession(create: true)`

## HTTP 요청 데이터

### :pushpin: GET 쿼리 파라미터
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

### :pushpin: POST HTML Form
 + POST 형식으로 보낼때, HTTP 메시지에는 `content-type` 이 지정됌(메시지 바디에 데이터가 존재)
   + `content-type` : GET 에서 살펴본 쿼리 파라미터 형식과 같음
      + message body: username=hello&age=20
 + 따라서, 마찬가지로 `request.getParameter()` 이용해 Form을 통해 전송한 데이터 가져오기 가능!

### :pushpin: API 메시지 바디-단순 텍스트
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

### :pushpin: API 메시지 바디-JSON
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
  
## HTTPServletResponse

### :pushpin: 기본 사용법
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

### :pushpin: HTTP 응답 데이터-HTML/JSON
> HTML 형식
  + response.setContentType("text/html")
> JSON 형식
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
