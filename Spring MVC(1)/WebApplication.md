# Spring MVC_웹 애플리케이션 이해

## 웹 시스템 구성

### :pushpin: WEB,WAS,DB
  + 정적 리소스는 웹 서버가 처리
  + 애플리케이션 로직같은 동적 처리는 WAS에 요청을 위임
  + `장점` 
    + 효율적인 리소스 관리 가능 (각 WEB 서버/WAS 증설)
    + WAS,DB 장애시 WEB 서버가 오류 화면 제공 가능(웹 서버는 잘 죽지 X)
 
> 웹 서버 VS 웹 애플리케이션 서버(WAS)
  + `웹 서버` : HTTP 기반 동작, **정적 리소스** 제공
  + `웹 애플리케이션 서버(WAS)` : 웹 서버 기능 포함 + **애플리케이션 로직/코드** 수행 ex)톰캣
    + 동적 HTML,서블릿,JSP,스프링 MVC...
  
  
## 서블릿
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
 
### :pushpin: 서블릿 컨테이너
  + 톰캣처럼, 서블릿을 지원하는 **WAS**이며 서블릿 객체를 생성/초기화/호출/종료하는 생명주기 관리
  + 서블릿 객체는 **싱글톤**으로 관리됌
    + 최초 로딩 시점에 서블릿 객체 하나만 생성해두고 재활용 => 고객 요청은 동일한 서블릿 객체 인스턴스에 접근
    + 따라서 **공유 변수 사용에 주의**
  + 동시 요청을 위한 멀티스레드 지원

## 동시요청-멀티 쓰레드

### :pushpin: 쓰레드란?
  + 애플리케이션 코드 하나하나를 순차적으로 실행(한번에 하나의 코드라인만)
    + 동시 처리가 필요하면 **요청 시 마다 신규 쓰레드를 생성**해야함
    
  > 😥 요청 마다 쓰레드 생성의 단점
  + 쓰레드 생성 비용은 매우 비쌈 => 응답 속도가 늦어짐
  + 컨텍스트 스위칭 비용이 발생
  + 쓰레드 생성에 제한이 X=> CPU와 메모리 임계점을 넘어 서버가 죽을 수 있음
    
### :pushpin: 쓰레드 풀
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

## HTML,HTTP API,CSR,SSR
  + `정적 리소스` : 고정된 HTML 파일,CSS,JS,이미지,영상 등 브라우저에게 제공
  + `HTML 페이지` : WAS가 동적으로 필요한 HTML 파일 생성해서 브라우저에게 전달
  + `HTTP API`: HTML이 아니라, 데이터 전달(주로 JSON 형식 {KEY : VALUE})
    + UI 클라이언트 접점
      + 앱 클라이언트/웹 브라우저에서 자바 스크립트 통한 HTTP API호출/React,Vue.js
    + 서버 TO 서버
      + 주문 서버 -> 결제 서버/ 기업간 데이터 통신
  
### ✔️ SSR
  + `서버 사이드 렌더링(SSR)` : HTML 최종 결과를 서버에서 만들어서, 웹 브라우저에 전달 
  + 주로 정적인 화면에 사용 EX) JSP, 타임리프(백엔드 개발자)
  
### ✔️ CSR
  + `클라이언트 사이드 렌더링(CSR)` : HTML 결과를 자바스크립트 이용해 웹브라우저에서 동적으로 생성해 적용
  + 주로 복잡하고 동적인 화면에 사용 EX)React, Vue.js(프론트엔드 개발자)
  
<img src = "https://user-images.githubusercontent.com/71436576/126956781-aaf037b8-69cc-4b73-8bda-0c2de8c81761.png" width=50% height=50%>
