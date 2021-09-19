# Login

## 패키지 구조
```
hello.login
  domain
    item
    member
    login
  web
    item
    member
    login
```
 + `Domain` : 핵심 비즈니스 업무 영역(화면,UI,기술 제외 영역)
   + :star2: **Domain은 Web에 의존하면 안되며, Web은 Domain을 의존가능 함**
  
## 로그인 처리 : Cookie

### 📌 Cookie Privew
 + 로그인의 **상태 유지**를 가능하게 해줌
 + `서버`에서 로그인에 성공하면, HTTP 응답에 쿠기를 담아 브라우저에 전달
   + `브라우저`는 해당 쿠키를 지속해서 보내줌(모든 요청에 쿠키 포함)  ex)memberId=1
 + 직접 쿠키의 기간을 설정 가능
   + `영속 쿠키` : 만료 날짜를 입력하면 해당 날짜까지 유지
   + `세션 쿠키` : 만료 날짜를 생략하면 브라우저 종료시 까지 유지 
+ _LoginController에서 로그인 성공시, 쿠키 생성_

#### ✔️ LOGIN
> LoginController
```
  Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));
  response.addCookie(idCookie);
```
 + 쿠키 이름: memberId, 값: 회원의 id   `ex)memberId=1`
   + 웹 브라우저는 종료 전까지 회원의 id 를 서버에 계속 보내줌  `ex)memberId=1`
 
 > HomeController : 로그인 성공 시 홈 화면
 ```java
 @GetMapping("/")
    public String homeLogin(@CookieValue(name="memberId", required = false) Long memberId, Model model){
        if(memberId == null){    //쿠키가 없으면
            return "home";
        }

        Member loginMember = memberRepository.findById(memberId);
        if(loginMember == null){     //DB에 정보가 없으면
            return "home";
        }
        
        model.addAttribute("member", loginMember);    //성공 시 뷰로 로그인 정보 전달
        return "loginHome";
    }
 ```
  + `@Cookie` : `HttpServletRequest`를 사용하지 않고 편리하게 쿠키 조회
  + `required = false` : 일반 사용자도 홈에 접근 가능
  
 #### ✔️ LOGOUT
 
 > LoginController
 ```java
  @PostMapping("/logout")
    public String logout(HttpServletResponse response){
        expireCookie(response, "memberId");
        return "redirect:/";
    }

    private void expireCookie(HttpServletResponse response, String cookieName) {
        Cookie cookie = new Cookie(cookieName, null);
        cookie.setMaxAge(0);
        response.addCookie(cookie);
    }
 ```
  + `cookie.setMaxAge(0)` : 해당 쿠키는 즉시 종료
 
###  📌 Cookie와 보안 이슈
 + 쿠키 값은 임의로 변경 가능
   + 실제 웹브라우저 개발자 모드에서 클라이언트가 변경 가능
 + 쿠키에 보관된 정보를 훔쳐갈 수 있음

> 대안
 + 쿠키에 중요한 값이 아닌 **사용자 별로 예측 불가능한 TOKEN**(랜덤 값)을 노출 해야함
   + 서버에서 토큰과 사용자 ID를 매핑해서 인식하고 관리해야함
 + 임의의 값을 넣어도 찾을 수 없도록 예상 불가능해야함
 + 토큰의 만료시간을 짧게 유지해서 시간이 지나면 사용 불가능 하도록 해야함

## 로그인 처리 : Session
+ `Session` : **서버에 중요한 정보를 보관**하고, 연결을 유지하는 방법
  + `Session ID` : **UUID**를 이용해 추정 불가능
  + `Session 저장소` : 서버에 중요한 정보를 보관하는 장소
    + `Cookie: mySessionId=zz0101xx-bab9-4b92-9b32-dadb280f4b61`
+ ❓ 세션의 정확한 정의 :
  + store(세션 저장소) 전체가 세션이라고 볼 수 있음
  + 각각의 키:값 쌍도 세션(세션의 데이터)
  
<img src = "https://user-images.githubusercontent.com/71436576/129439022-09e9ac40-9f35-458e-9c75-07bec091c4ea.png" width=50% height=50%>

+ **서버는 클라이언트에 `세션ID` 만 쿠기에 담아서 전달**         
+ :star2: _회원과 관련된 정보 대신 추정 불가능한 세션 ID만 쿠키를 통해 클라이언트에 전달 해야함!_
  + 이후, 재요청시 전달한 세션 ID 쿠키 정보로 저장소를 조회하여 사용                                                                                                       
+ 즉 Session 저장소에 중요한 정보를 저장하고, 임의의 랜덤한 Token값으로 쿠키를 사용해 통신

### :pushpin: Session 직접 만들기
```java
  public static final String SESSION_COOKIE_NAME = "my_session_id";
  private Map<String, Object> sessionStore = new ConcurrentHashMap<>(); 
```
+ **세션 생성**
  + sessionId 생성 (임의의 추정 불가능한 랜덤 값)
  + 세션 저장소에 sessionId와 보관할 값 저장
sessionId로 응답 쿠키를 생성해서 클라이언트에 전달

```java
 public void createSession(Object value, HttpServletResponse response){
        //세션 id 생성 후 값을 세션에 저장

        String sessionId = UUID.randomUUID().toString();
        sessionStore.put(sessionId, value);

        //쿠키 생성
        Cookie mySessionCookie = new Cookie(SESSION_COOKIE_NAME, sessionId);
        response.addCookie(mySessionCookie);
    }
```
+ **세션 조회**
  + 클라이언트가 요청한 sessionId 쿠키의 값으로, 세션 저장소에 보관한 값 조회
```java
   public Object getSession(HttpServletRequest request){
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
        if(sessionCookie == null){
            return null;
        }
        return sessionStore.get(sessionCookie.getValue());  //UUID 값으로 찾아서 멤버 객체 반환
    }
    
   public Cookie findCookie(HttpServletRequest request, String cookieName){
        if(request.getCookies() == null){
            return null;
        }
        return Arrays.stream(request.getCookies())
                .filter(cookie -> cookie.getName().equals(cookieName))
                .findAny()
                .orElse(null);
    }
```
+ **세션 만료**
  + 클라이언트가 요청한 sessionId 쿠키의 값으로, 세션 저장소에 보관한 sessionId와 값 제거
```java
 public void expire(HttpServletRequest request){
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
        if(sessionCookie != null){
            sessionStore.remove(sessionCookie.getValue());
        }
 }
 ```
 > LoginVersion1
 ```java
  @PostMapping("/login")
    public String loginV2(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletResponse response) {
        ...
        //로그인 성공 처리 TODO
        //세션 관리자를 통해 세션을 생성하고, 회원 데이터 보관
        sessionManager.createSession(loginMember,response);
        return "redirect:/";
    } 
 ```
 
 > LogoutVersion2
 ```java
  @PostMapping("/logout")
    public String logoutV2(HttpServletRequest request){
        sessionManager.expire(request);
        return "redirect:/";
    }
 ```
 > HomeControllerV2
 ```java
 @GetMapping("/")
    public String homeLoginV2(HttpServletRequest request, Model model){
        //세션 관리자에 저장된 회원 정보 조회
        Member member = (Member)sessionManager.getSession(request);

        if(member == null){
            return "home";
        }
        model.addAttribute("member", member);
        return "loginHome";
    }
 ```

## 로그인 처리: Servlet HTTP 세션(1)

### :heavy_check_mark: HTTPSession 소개
 + SessionManager와 같은 방식으로 동작
 + `쿠키 이름` : JSESSIONID
 +  `값` : 추청 불가능 랜덤 값
   + `Cookie: JSESSIONID=5B78E23B513F50164D6FDD8C97B0AD05`
 
> LoginVersion3
```java
       //세션이 있으면 세션 반환, 없으면 신규 세션을 생성
        HttpSession session = request.getSession();

        //세션에 로그인 회원 정보를 보관
       session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);
```
 + `request.getSession()` : 
   + 사용자의 request 정보를 가지고 키를 생성 (이미 존재할 경우 패스)한 후 쿠키에 담아서 응답 
   +  `request.getSession(true)` : 세션이 있으면 기존 반환, 없으면 새로운 세션 생성 후 반환
   +  `request.getSession(false)` : 세션이 있으면 기존 반환, 없으면 새로운 세션 생성 X, null 반환
 +  `session.setAttribute` : 세션에 데이터를 보관(하나에 여러 값 가능)

> LogoutVersion3
```java
    @PostMapping("/logout")
    public String logoutV3(HttpServletRequest request){
        HttpSession session = request.getSession(false);
        if(session != null){
            session.invalidate();   //세션 데이터 날리기
        }
        return "redirect:/";
 ```

> HomeControllerV3
```java
    @GetMapping("/")
    public String homeLoginV3(HttpServletRequest request, Model model){
        HttpSession session = request.getSession(false);
        if(session == null){
            return "home";
        }

        Member loginMember = (Member)session.getAttribute(SessionConst.LOGIN_MEMBER);

        //세션에 회원 데이터가 없으면 home
        if(loginMember == null){
            return "home";
        }

        //세션이 유지되면 홈으로 이동
        model.addAttribute("member", loginMember);
        return "loginHome";
    }
```
 + `request.getSession(false)` : 세션을 찾아서, 사용하는 시점에는 false 옵션으로 새로운 세션 생성을 막아야함
 + `session.getAttribute` : 로그인 시점에 보관한 회원 객체를 찾음
 
## 로그인 처리: Servlet HTTP 세션(2)

### :heavy_check_mark: @SessionAttribute
 + 이미 로그인한 사용자를 찾을 떄는, `@SessionAttribute`사용!
 
 > HomeControllerV4
 ```java
    @GetMapping("/")
    public String homeLoginV3Spring(@SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false
    ) Member loginMember, Model model){

        //세션에 회원 데이터가 없으면 home
        if(loginMember == null){
            return "home";
        }

        //세션이 유지되면 홈으로 이동
        model.addAttribute("member", loginMember);
        return "loginHome";
    }
```
 + `@SessionAttribute(name = "loginMember", required = false) Member loginMember`:
   + 세션을 name으로 찾고, 세션에 들어있는 데이터를 찾아 loginMember에 담아주는 기능을 함

## 세션 정보와 타임아웃 설정
 + 세션은 로그아웃을 호출하는 경우에 삭제됌
   + 사용자가 단순히 웹 브라우저를 종료하는 경우, 세션을 무한정 보관하게 되어 메모리 낭비가 발생

### 세션 타임아웃 설정
 + `세션 종료 시점` : 
   + 세션 생성 시점이 아니라, **사용자가 서버에 최근에 요청한 시간을 기준**으로 유지해야함
   + `session.getLastAccessedTime()` : 최근 세션 접근 시간
   
> 스프링 부트로 글로벌 설정(분 단위)
```
server.servlet.session.timeout=60
```

> 특정 세션 단위로 세션 설정
```java
session.setMaxInactiveInterval(1800); //1800초
```

:star2 : **세션에는 최소한의 데이터만 보관해야함!**</br>
 => 로그인 용 Mebmer객체를 따로 만들어서, 정말 필요한  몇가지 정보만 해서 세션의 객체로 사용
