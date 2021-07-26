# HTTP

## Internet Network
### :pushpin: IP
+ `IP` : 인터넷 프로토콜로써, 지정한 IP Address에 **Packet**(통신 단위)로 데이터 전달
  + EX) 100.100.100.1 -> 200.200.200.2
  + Network Layer(3)의 프로토콜
  
### :pushpin: IP의 한계
+ 비연결성 : 대상이 서비스 불능이여도 패킷 전송
+ 비신뢰성 : 패킷 소실, 전달 순서에 문제 발생 가능
+ 프로그램 구분 : 같은 IP 사용 서버에서 통신 애플리케이션이 둘 이상일 경우 구분 불가능
 
### :pushpin: TCP, UDP
+ `TCP` : 전송 제어 프로토콜로서, 3-way-handshake **연결과정** 이후에 데이터 전송하므로 순서와 신뢰성 보장
  + `TCP Segment` : 출발지/도착지 PORT 정보, 전송 제어/순서/검증 정보,전송 데이터 등이 담겨있음
  + Transport Layer(4)의 프로토콜
+ `UDP` : 단순 데이터 전달로서, 전송과 순서가 보장되지 않지만, 단순하고 빠른 protocol(연결 과정X)
   
> :bulb: 3-way-handshake 
 + Client > Server : TCP **SYN**
 + Server > Client : TCP **SYN ACK**
 + Client > Server : TCP **ACK**
    + `SYN` : 접속 요청, `ACK` : 요청 수락
  
> :star2: PORT
  + 같은 IP 내에서 프로세스를 식별하는 논리적 단위, http: 80번/ https: 443 port번호 이용
  ``` 
  어떠한 데이터가 송수신을 할 때
  Datalink 계층에서는 호스트의 NIC로 MAC Address를 판별하고 
  Network 계층에서는 IP Address로 목적지를 판별.
  이렇게 MAC Address와 IP Address를 통해 목적지 호스트까지 도달한 후에는
  어떤 Process(프로세스)에서 데이터를 받을 것인지 를 알아야 하는데,이 때 쓰이는 것이 Port Number!
  ```
 
### :pushpin: URI
+ Uniform Resource Identifier로써, 통일된 방식으로 식별 가능한 모든 자원(리소스)을 구분하는데 필요한 정보
  + `URL`: Resource Locator, 자원의 위치를 지정 ex)foo://example.com:8042
  + `URN`: Resource Name, 자원엥 이름을 부여  ex)urn:example:animal:
<img src = "https://user-images.githubusercontent.com/71436576/126471535-8288d3c3-dbe8-4beb-95c8-d95c1eeea92d.png" width="50%" height="50%">

### URL
+ **scheme**://[userinfo@]**host**[:port][/path][?query][#fragment]
  + https://www.google.com:443/search?q=hello&hl=ko
  + `query` : 웹 서버에 제공하는 파라미터, 문자 형태(쿼리 파라미터,쿼리스트링)
    + key = value의 형태로써, ?로 시작함 

## HTTP
  + Hypertext, 즉 모든 종류의 정보(html,이미지,JSON,XML 등)를 주고받을 수 있는 프로토콜

### :pushpin: HTTP 특징
**1. 클라이언트-서버 구조**</br>
  + `클라이언트`가 서버에 요청을 보내고(Request), 응답을 대기 / UI, 사용성에 집중
  + `서버`는 요청에 대한 결과 만들어서 응답(Response) / 데이터, 비즈니스 로직에 집중
  
**2. :star2: 무상태 프로토콜**</br>
> Stateless(무상태) vs Stateful(상태 유지)
  + `Stateless`: 서버가 클라이언트의 **상태(Context)를 보존하지 X**
    + 장점) 무상태는 응답 서버를 쉽게 바꿀 수 있음 => 무한한 서버 증설이 가능(Scale Out)
    + 단점) 클라이언트는 항상 추가적 데이터를 전송해야함
  + `Stateful`: 서버가 클라이언트의 **상태를 유지**, 항상 같은 서버가 유지되어야함 ex)로그인
  
:pencil2: 가능한 최대한 **STATELESS**하게 코드를 짜야 함! </br>

**3. Connectionless**</br>
 + HTTP는 연결을 유지하지 않는 모델
 + 서버는 클라이언트 요청에 대한 응답을 보낸 후, 바로 연결을 끊어 최소한의 자원만 사용하도록 함
    + BUT, 연결과정의 시간과 자원이 낭비됌 => HTTP 지속 연결(Persistent Connection)로 문제 해결! 
  
**4. HTTP 메시지**</br>
> 요청(Request) 메시지
1) Start Line : Request-line(HTTP 메서드 + 요청대상 + HTTP Version)</br>
    + `HTTP 메서드`: GET(리소스 조회)/POST(요청 내역 처리)
 
2) HTTP 헤더 : field-name: field-value ex) Host:www.google.com</br>
    + HTTP 전송에 필요한 모든 부가정보가 담겨 있음
  
3) HTTP 메시지 바디 : 실제 전송할 데이터가 담겨 있음</br>

> 응답(Response) 메시지
1) Start Line : Status-line(HTTP Version + **Status Code** + 이유 문구) </br>
  + `Status code` : 200(성공)/400(클라이언트 요청 오류)/500(서버 내부 오류)
 
2) HTTP 헤더 : field-name: field-value ex) Host:www.google.com</br>
    + HTTP 전송에 필요한 모든 부가정보가 담겨 있음
  
3) HTTP 메시지 바디 : 실제 전송할 데이터가 담겨 있음</br>

<img src = "https://user-images.githubusercontent.com/71436576/126471171-8903d3d1-4f4b-4c28-9e16-968ce71c766b.png" width="50%" height="50%">


## HTTP 메서드

### :pushpin: API URI 설계

:seedling: **리소스 식별, URI 계층 구조**를 활용해라!</br>
+ URI는 리소스만 식별, 행위(메서드)는 HTTP 메서드를 이용
  + `리소스` : 회원 조회에서, '회원'이라는 개념 자체
     + ex) 회원 조회/members/{id}, 회원 등록/members/{id}...
  + `행위` : 조회,등록,삭제,변경..  

### :pushpin: HTTP 메서드-GET,POST

**1)GET** : 리소스 조회</br>
  + 서버에 전달하고 싶은 데이터=> query(파라미터/스트링)를 통해 전달!
  + `안전` => 호출해도 대상 리소스 변경이 X
  + `멱등` => 몇번을 호출하든 최종 결과가 같음
  + `캐시가능(Cacheable)` => 응답 결과 리소스를 캐시해서 사용해도 되는가?

**2)POST** : 요청 데이터 처리(등록)</br>
  + Message Body를 통해 서버로 요청 데이터를 전달
  + 서버는 데이터를 처리
    + 신규 리소스 등록
    + 요청 데이터 처리 : 단순 데이터 생성/변경을 넘어서 **프로세스 처리**해야하는 경우 EX)주문완료->배달시작
    + 다른 메서드로 처리 애매한 경우 
  + `멱등 XX` => 호출에 따라 최종 결과가 달라짐   

> 멱등(Idempotent) 
  + 자동 복구 메커니즘에 활용
  + 서버가 TIMEOUT되어 정상 응답을 못주었을때, 클라이언트가 요청을 반복해도 되는가?의 판단근거
  + BUT, 외부 요인으로 중간에 리소스가 변경되는것을 고려 X

**3)PUT** : 리소스를 **완전 대체**, 해당 리소스가 없으면 생성</br>
  + :star2: 클라이언트가 리소스를 식별해야함
    + `PUT` : PUT /members **/100** HTTP/1.1 
    + `POST`: POST /members HTTP/1.1 
  + `멱등` => 몇번을 호출하든 최종 결과가 같음
 
**4)PATCH** : 리소스 부분 변경</br>

**5)DELETE** : 리소스 삭제</br>
  + `멱등` => 몇번을 호출하든 최종 결과가 같음

### HTTP 메서드 활용
#### :pushpin: 클라이언트에서 서버로 데이터 전송

> 쿼리 파라미터를 통한 데이터 전송
  + GET
  + 주로 정렬 필터(검색어)
> 메시디 바디를 통한 데이터 전송
  + POST, PUT, PATCH
  + 회원 가입, 상품 주문, 리소스 등록, 리소스 변경
  
> 다음 네가지의 상황 예시
1) 정적 데이터 조회</br>
  + 조회는 GET 사용(쿼리 파라미터 없이 리소스 경로로 단순하게 조회)
  + URL 경로를 찾아서, 이미지 리소스를 클라이언트에게 반환
<img src = "https://user-images.githubusercontent.com/71436576/126594283-01bd703d-4154-4357-a070-12d93a170252.png" width=50% height=50%>

2) 동적 데이터 조회</br>
  + 주로 검색,게시판 목록에서 정렬 필터(검색어)
  + 조회는 GET 사용(쿼리 파라미터를 이용해 서버로 데이터 전달)
  + 서버는 쿼리 파라미터를 기반으로 정렬필터해서, 결과를 동적으로 생성
  
 <img src = "https://user-images.githubusercontent.com/71436576/126594288-f7d64cd0-0fff-41ec-a713-cb00c12189d1.png width=50% height=50%>

3) HTML Form을 통한 데이터 전송</br>
  + POST 전송-저장 
    + form의 내용을 메시지 바디를 통해 전송(key=value, 쿼리 파라미터 형식)
    + 전송 데이터를 url encoding 처리
  <img src = "https://user-images.githubusercontent.com/71436576/126594348-f4459c9c-e8d5-452e-97df-8ed95c7dd355.png" width=50% height=50%> 
  
  + GET 전송-저장 : 쿼리 파라미터 형식
 <img src = "https://user-images.githubusercontent.com/71436576/126594376-7212aa06-5420-4828-b66e-39c36f3cb587.png" width=50% height=50%>  

  + GET 전송-조회
 <img src = "https://user-images.githubusercontent.com/71436576/126594399-78564804-33f3-482e-b673-3994efe00cba.png" width=50% height=50%>  

  + multipart/from-data : 파일 업로드 같은 바이너리 데이터 전송시 사용(여러 종류의 파일 가능)
  
  4) HTTP API를 통한 데이터 전송</br>
  + 백엔드 시스템 통신
  + 앱 클라이언트 : 안드로이드/아이폰
  + 웹 클라이언트
    + `AJAX` : hTML의 Form 대신 자바 스크립트를 통한 통신
 <img src = "https://user-images.githubusercontent.com/71436576/126594428-9431f6ab-6327-4963-917c-92e15b3fb10a.png"  width=50% height=50%>
  
#### :pushpin: HTTP API 설계 예시

**1) HTTP API-Collection**</br>
  + **POST 기반 등록** ex)회원 관리 API 제공
  + 신규 자원을 POST 기반으로 등록할시 주의점
    + 클라이언트는 등록될 리소스의 URI을 모름 ex)POST /members
    + :star2: **서버가 새로 등록된 리소스 URI를 생성!** ex)Location:/members/100
  + `Collection` : 서버가 관리하는 리소스 디렉토리(URI 생성 후 관리)  ex)/members
  
**2) HTTP API-Store**</br>
  + **PUT 기반 등록** ex) 정적 컨텐츠 관리, 원격 파일 관리
  + 신규 자원을 PUT 기반으로 등록할시 주의점
    + :star2: **클라이언트는 리소스 URI를 알고 지정해줘야함** ex)PUT /files/star.jpg
  + `Store` : 클라이언트가 관리하는 리소스 저장소(URI 알고 관리)  ex)/files
  
**3) HTML Form 사용**</br>
  + HTML Form은 GET, POST 만 지원
  + `컨트롤 URI(Controller)` : 동사로 된 리소스 경로 ex)POST의 /new,/edit,/delete
    + 문서,컬렉션,스토어로 해결하기 애매한 경우에만 사용!(HTTP API 포함)
  
 <img src = "https://user-images.githubusercontent.com/71436576/126597453-ff1896d8-c247-4091-8715-0b07a8cb7516.png width=50% height=50%>


## HTTP 상태 코드
  + `상태 코드` : 클라이언트가 보낸 요청의 처리 상태를 **응답**에서 알려주는 기능
    + 서버가 인식할 수 없는 상태코드를 반환 => 클라이언트는 상위 상태 코드로 해석!
### :pushpin: 1xx(Informational)

### :pushpin: 2xx(Successful)
  + 클라이언트의 요청을 성공적으로 처리
  + `200 OK` : 요청 성공
  + `201 Created` : 요청 성공하여 새로운 리소스 생성됌
    + `Location 헤더 필드` : 생성된 리소스의 URI 정보가 담겨있음 ex)Location:/members/100
  + `202 Accepted` : 요청 접수되었으나 처리가 완료 X
    + 배치 처리에서 사용
  + `204 No Content` : 서버가 요청을 성공적으로 수행했지만, 응답 페이로드 본문에 보낼 데이터가 X
    + ex) 웹 문서 편집기에서의 Save 버튼 => 결과 내용이 필요 X, 성공 여부만 중요할때!
   
### :pushpin: 3xx(Redirection)
  + 요청을 완료하기 위해, 유저 에이전트(클라이언트 프로그램)의 추가 조치 필요
  
> :question: Redirection
  + 웹 브라우저는 3xx 응답 결과에 Location 헤더가 존재 => Location 위치로 자동 이동(redirect)
<img src = "https://user-images.githubusercontent.com/71436576/126860851-3d777a77-8526-4f37-9d77-feef6dc5ce35.png" width=60% height=60%>

> :question: Redirection의 종류
  
**1) 영구 리다이렉션 : 301,308**</br>
  + 리소스의 URI가 영구적으로 이동하여 원래 URL를 사용X(검색엔진에서 변경 인지)
  + `301 Moved Permanently` : redirect시 요청 메서드가 GET으로 변하고, 본문 제거될수 있음
  + `308 Permanent Redirect` : redirect시 요청 메서드와 본문 유지(POST -> POST) 
  
**2) 일시적 리다이렉션 : 302,307,303**</br>
  + 리소스의 URI가 일시적으로 변경(검색 엔진에서 변경하면 안됌!)
  + `302 Found`: redirect시 요청 메서드가 GET으로 변하고, 본문 제거될수 있음(MAY)
  + `307 Temporary Redirect`: redirect시 요청 메서드와 본문 유지(MUST NOT)
  + `303 See Other`: redierct시 요청 메서드가 GET으로 변경(MUST)

 > 일시적 리다이렉션의 예시
  + POST 주문 후에 웹 브라우저를 새로고침하면 중복주문 되는 문제가 발생!(POST 요청이 2번)
  + `PRG(Post/Redirect/Get)` : 일시적 리다이렉션의 패턴
    + Post 주문 후 결과 화면을 GET 메서드로 redirect
    + 새로고침해도 GET으로 결과 화면만 조회
<img src = "https://user-images.githubusercontent.com/71436576/126861290-583a4b19-b0fa-4acf-a6f5-91bcf98a1ae8.png" width=60% height=60%>
 
**3) 기타 리다이렉션**
 + `304 Not Modified` : 캐시 redirect을 목적으로 사용
  + 클라이언트에게 리소스가 수정되지 않았음을 알려주고, 클라이언트는 로컬PC에 저장된 캐시를 재사용
  + 응답에 메시지 바디가 포함되면 X(로컬 캐시 사용해야함)

### :pushpin: 4xx(Client Error)
  + 오류의 원인이 클라이언트에게 존재!
  + :star2: 클라이언트가 재시도 요청을 해도 이미 잘못된 요청이기 때문에 **복구 불가능** => 요청 수정 필요!
  + `400 Bad Reuest` : 클라이언트가 잘못된 요청을 해 서버가 요청 처리 불가능
    + 요청 파라미터가 잘못되거나, API 스펙이 맞지 않을때
  + `401 Unauthorized` : 클라이언트가 해당 리소스에 대한 인증이 필요
    + `인증(Authentication)` : 본인이 누구인지 확인(로그인)
    + `인가(Authorization)` : 권한 부여(ADMIN 권한, 인증이 있어야 인가가 존재)
  + `403 Forbidden` : 서버가 요청을 이해했지만, 승인을 거부
    + 인증 자격 증명은 있지만, 접근 권한이 불충분한 경우
  + `404 Not Found` : 요청 리소스를 찾을 수 없음
    + 서버에 요청 리소스가 존재X, 클라이언트가 권한이 부족한 리소스에 접근할때

### :pushpin: 5xx(Server Error)
  + 서버 문제로 오류 발생
  + :star2: 재시도 요청시 서버에 문제가 있기 때문에 성공 가능성이 존재(서버 복구 등)
  + `500 Internal Server Error` : 서버 문제로 오류 발생, 애매하면 500 오류
  + `503 Service Unavailable` : 서버가 일시적 과부화/예정된 작업으로 잠시 요청을 처리 X
 
> 비즈니스 로직 상 예외 케이스 말고, 오직 쿼리 / NPE 같은 '서버 문제'에만  5XX 에러를 내야함! 

## HTTP 헤더(1) - 일반 헤더

### :pushpin: HTTP 헤더 개요
 + `Header-field` : field-name ":" OWS field-vlaue OWS( OWS : 띄어쓰기 허용) 
 + HTTP 전송에 필요한 모든 부가정보가 담겨있음 EX) 메시지 바디 크기, 압축, 인증, 캐시...

#### :heavy_check_mark: 헤더 분류
 + `General 헤더` : 메시지 전체에 적용되는 정보
 + `Request 헤더` : 요청 정보 ex) User-Agent:..
 + `Reponse 헤더` : 응답 정보 ex) Server:Apache
 + `Representation(표현) 헤더`: 표현 데이터를 해석할 수 있는 정보 제공
   + 메시지 본문(Payload)을 통해 표현 데이터 전달
    + `표현` : 요청이나 응답에서 전달할 실제 데이터
   + 데이터 유형(html,json),데이터 길이, 압축 정보 등등이 담겨있음

#### :heavy_check_mark: 표현 
  + `Content-Type` : 표현 데이터의 형식
    + 미디어 타입,문자 인코딩 ex) application/json, text/html;charset=utf-8
  + `Content-Encoding` : 표현 데이터의 압축 방식
    + 데이터 읽는 쪽에서 인코딩 헤더의 정보를 가지고 압축 해제
  + `Content-Language` : 표현 데이터의 자연 언어
    + ex) ko, en, en-US
  + `Content-Length` : 표현 데이터의 길이(Byte 단위)
  
### :pushpin: 콘텐츠 협상
 + 요청(Request)시에만 협상 헤더를 사용할 수 있음!
  + `Accept` : 클라이언트가 선호하는 미디어 타입 전달
  + `Accept-Charset` : 클라이언트 선호 문자 인코딩
  + `Accept-Encoding` : 클라이언트 선호 압축 인코딩
  + `Accept-Language` : 클라이언트 선호 자연 언어
  
> 협상과 우선순위(1)
  + Quality Values(q) 값을 사용하여, 클수록 높은 우선순위를 가짐(생략하면 1)
  + ex) Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
    + 1. ko-KR;q=1 (q생략) / 2.ko;q=0.9 ...
    
> 협상과 우선순위(2) 
  + **구체적인 것**이 우선하며, 구체적인 것을 기준으로 미디어 타입을 맞춤
  + ex) Accept: text/* , text/plain, text/plain;format=flowed, */*
    + 1. text/plain;format=flowed / 2. text/plain ..

### :pushpin: 헤더 정보
#### :heavy_check_mark: (1)일반 헤더 정보
  + `From` : 유저 에이전트의 이메일 정보
  + `Referer` : 현재 요청된 페이지(A)의 이전 웹 페이지 주소(B) => B 요청 시 Referer:A 포함해서 요청
    + 유입 경로 데이터 분석 가능
    + 요청에서 사용
  + `User-Agent` : 클라이언트 애플리케이션 정보(웹 브라우저 정보 등)
    + 통계 정보, 어떤 브라우저에서 장애 발생하는지 파악 가능
    + 요청에서 사용
  + `Server` : 요청 처리하는 **ORIGIN 서버**의 소프트웨어 정보
    + 응답에서 사용
  + `Date` : 메시지가 발생하 날짜와 시간
    + 응답에서 사용

#### :heavy_check_mark: (2)특별한 헤더 정보
  + `Host` : 요청한 호스트 정보(도메인)  ex)Host:aaa.com
    + 하나의 IP 주소에 여러 도메인이 적용되어 있을때 유용
    + 요청에서 사용, 필수 정보
  + `Location` 
    + 201(Created): Location 값은 요청에 의해 생성된 리소스 URI
    + 3XX(Redirection) : Location 값은 요청을 자동으로 리디렉션하기 위한 대상 리소스를 가리킴 
  + `Allow` : 허용 가능한 HTTP 메서드
  + `Retry-After` : 유저 에이전트가 다음 요청하기까지 기다려야 하는 시간
     + 503(Service Unavailable) : 서비스가 언제까지 불능인지 알려줌 (ex) Retry-After:120)
  + `Authorization` : 클라이언트 인증 정보를 서버에 전달
    + Authorization: Basic xxx
  + `WWW-Authenticate` : 리소스 접근시 필요한 인증 방법 정의
    + 401 Unauthorized 응답과 함께 사용
    + WWW-Authenticate: Newauth realm="apps", type=1,  title="Login to \"apps\"", Basic realm="simple"
    
### :pushpin: Cookie
 + 무상태 프로토콜이므로, 상태 유지를 위해 모든 요청에 사용자 정보를 넘기게 되면 문제가 발생!
 + 상태 유지를 위해 사용하며, 모든 요청에 쿠키 정보가 자동으로 포함됌
 + 주로 사용자 로그인 세션 관리, 광고 정보 트래킹에 사용 
 + **보안에 민감한 데이터는 저장하면 X** (ex) 주민번호, 신용카드 번호 등)
   + `Set-Cookie`: 서버에서 클라이언트로 쿠키 전달(응답 메시지)
   + `Cookie` : 클라이언트가 서버에서 받은 쿠키를 저장하고, HTTP 요청시 서버로 전달
 + 쿠키 정보는 항상 서버에 전송 => 최소한의 정보만 사용(세션 id, 인증 토큰) 
<img src = "https://user-images.githubusercontent.com/71436576/126864070-693b8aee-6721-4e2f-86bf-984a096cb9ad.png" width=50% height=50%>

#### 쿠키-생명주기
  + `세션 쿠키` : 만료 날짜를 생략하면 브라우저 종료시 까지만 유지
    + `Expires ` : 만료일이 되면 쿠키 삭제 ex)Set-Cookie:expires=Sat,26-Dec-2020
    + `max-age` : 0이나 음수를 지정하면 쿠키 삭제 ex)Set-Cookie:max-age=3600(초)
  + `영속 쿠키` : 만료 날짜를 입력하면 해당 날짜까지 유지

#### 쿠키-도메인
  + 도메인 명시 => 명시한 문서 기준 도메인 + 서브 도메인 포함 ex)domain=example.org
  + 도메인 생략 => 현재 문서 기준 도메인만 적용

#### 쿠키-경로
  + 이 경로를 포함한 하위 경로 페이지만 쿠키 접근, 일반적으로 **path=/루트**로 지정
  + ex)path=/home지정 : /home, /home/level1 다 접근 가능

#### 쿠키-보안
  + `Secure` : https 인 경우에만 저농
  + `HttpOnly` : XSS 공격 방지, HTTP 전송에만 사용
  + `SameSite` : XSRF 공격 방지, 요청 도메인과 쿠키에 설정된 도메인이 같은 경우만 쿠키 전송


## HTTP 헤더(2) - 캐시와 조건부 요청

### :pushpin: 캐시 기본 동작
  + 캐시 존재 X => 데이터가 변경되지 않아도 계속 네트워크 통해 데이터를 다운받아야함(속도 느려짐)
  + 캐시 존재 O => 캐시 가능시간동안, 네트워크 사용하지 X(속도 빨라짐)
    + 캐시의 `유효 시간`이 초과하면, 서버 통해 데이터를 다시 조회하고 캐시를 갱신해야함 => 비효율적!
<img src = "https://user-images.githubusercontent.com/71436576/126864646-be0bfeea-872d-4da3-aa1b-701d58ca6675.png" width=60% height=60%>

### :pushpin: 검증 헤더와 조건부 요청

#### :heavy_check_mark: 검증 헤더와 조건부 요청(1)Last-Modified + If-Modified-Since
  + 캐시 만료 이후에도, 서버에서 기존 데이터를 변경하지 않은 경우 => 저장해두었던 캐시 재활용 가능!
    + 하지만 어떻게 클라이언트와 서버의 데이터가 같다는 것을 알수있을까?
 
> :pencil2: 요청 과정(1)
  1) 서버가 해당 데이터의 최종 수정일(Last-Modifided)을 응답 메시지의 헤더에 추가해서 전송하고, 클라이언트는 응답 결과를 브라우저의 캐시에 저장</br>
  2) 캐시 만료 이후, 서버가 클라이언트가 재요청한 요청 메시지에 담긴 데이터 최종 수정일(if-modified-since)을 보고, 날짜 비교를 통해 변경 여부를 검증</br>
  3) :star2: 데이터 갱신이 없었다면, 서버는 `304 Not Modified(캐시로 redirect)` + `헤더의 메타 정보`만 응답!(HTTP BODY의 실제 데이터는 전송 X)</br>
  4) 클라이언트는 서버가 보낸 응답 헤더 정보로 캐시의 메타 정보를 갱신하고, 저장되있는 데이터 재활용</br>
   
#### :heavy_check_mark: 검증 헤더와 조건부 요청(2)ETag + If-None-Match
  + `ETag(Entitiy Tag)` : 캐시용 데이터에 임의의 고유한 버전 이름을 달아둠
    + 이후, 데이터가 변경되면 이름을 바꾸어서 변경(Hash 다시 생성) ex)ETag:"AAAA" -> ETag:"BBBB"
    + 단순하게 ETag만 서버에 보내서 같으면 유지, 다르면 다시 받기 => **캐시 제어 로직을 서버에서 관리**
    
> :pencil2: 요청 과정(2)
  1) 서버가 해당 데이터의 이름(ETag)을 응답 메시지의 헤더에 추가해서 전송하고, 클라이언트는 ETag값을 브라우저의 캐시에 저장</br>
  2) 캐시 만료 이후, 서버가 클라이언트가 재요청한 요청 메시지에 담긴(If-None-Match) 이름을 보고, 서버 데이터의 이름과 비교해 변경 여부를 검증</br>
  3) :star2: 데이터 갱신이 없었다면, 서버는 `304 Not Modified(캐시로 redirect)` + `헤더의 메타 정보`만 응답!(HTTP BODY의 실제 데이터는 전송 X)</br>
  4) 클라이언트는 서버가 보낸 응답 헤더 정보로 캐시의 메타 정보를 갱신하고, 저장되있는 데이터 재활용</br>

### :pushpin: 캐시와 조건부 요청 헤더 정리

#### 1)캐시 제어 헤더
 + `Cache-Control` : 캐시 지시어(directives)
   + max-age : 캐시 유효 시간, 초단위
   + no-cache : 데이터는 캐시해도 되지만, 항상 Origin 서버에 검증하고 사용
   + no-store : 데이터에 민감한 정보가 있으므로 저장 X(메모리에서 사용 후 최대한 빨리 삭제)
 + `Pragma` : 캐시 제어(하위 호환)
   + no-cache
 + `Expires` : 캐시 유효 기간(하위 호환)
   + 캐시 만료일을 정확한 날짜로 지정(초단위 X), but Max-age가 더 유연하게 사용 가능!

#### 2)검증헤더와 조건부 요청
 + `검증 헤더(Validator)`
    + 캐시 데이터와 서버 데이터가 같은지 검증하는 데이터
    + Last-Modified,ETag
  + `조건부 요청 헤더`
    + 검증 헤더로, 조건에 따른 분기
    + If-Modified-Since(Last-Modified), If-None-Match(ETag)
    + 데이터가 미변경 : 304 Not Modified + 헤더 데이터만 전송(BODY 미포함)
    + 데이터가 변경 : 200 OK + 모든 데이터 전송(BODY 포함)
 
### :pushpin: 캐시 무효화

#### Cache-Control(캐시 지시어-확실한 캐시 무효화)
  + `no-cache` : 데이터는 캐시해도 되지만, 원 서버에 검증하고 사용
    + 원 서버 접근 실패시 => _Error / 200 OK_(프록시 캐시에서 데이터를 보여줌)
  + `no-store` : 데이터에 민감한 정보가 있으므로 저장하면 안됌
  + `must-revalidate` : 캐시 만료후, 최초 조회시 원 서버에 검증 필요
    + 원 서버 접근 실패시 => 항상 _504(Gateway Timeout)오류_ 발생
  + `Pragma` : no-cache




  


   
  
 

