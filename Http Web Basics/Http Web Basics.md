# HTTP

## Internet Network
### IP
+ `IP` : 인터넷 프로토콜로써, 지정한 IP Address에 **Packet**(통신 단위)로 데이터 전달
  + EX) 100.100.100.1 -> 200.200.200.2
  + Network Layer(3)의 프로토콜
  
### IP의 한계
+ 비연결성 : 대상이 서비스 불능이여도 패킷 전송
+ 비신뢰성 : 패킷 소실, 전달 순서에 문제 발생 가능
+ 프로그램 구분 : 같은 IP 사용 서버에서 통신 애플리케이션이 둘 이상일 경우 구분 불가능
 
### TCP, UDP
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
 
### URI
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

### HTTP 특징
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
> 요청 메시지
1) Start Line : Request-line(HTTP 메서드 + 요청대상 + HTTP Version)</br>
    + `HTTP 메서드`: GET(리소스 조회)/POST(요청 내역 처리)
 
2) HTTP 헤더 : field-name: field-value ex) Host:www.google.com</br>
    + HTTP 전송에 필요한 모든 부가정보가 담겨 있음
  
3) HTTP 메시지 바디 : 실제 전송할 데이터가 담겨 있음</br>

> 응답 메시지
1) Start Line : Status-line(HTTP Version + **Status Code** + 이유 문구)
  + `Status code` : 200(성공)/400(클라이언트 요청 오류)/500(서버 내부 오류)

<img src = "https://user-images.githubusercontent.com/71436576/126471171-8903d3d1-4f4b-4c28-9e16-968ce71c766b.png" width="50%" height="50%">





## HTTP 메서드

### API URI 설계

:seedling: **리소스 식별, URI 계층 구조**를 활용해라!</br>
+ URI는 리소스만 식별, 행위(메서드)는 HTTP 메서드를 이용
  + `리소스` : 회원 조회에서, '회원'이라는 개념 자체
     + ex) 회원 조회/members/{id}, 회원 등록/members/{id}...
  + `행위` : 조회,등록,삭제,변경..  

### HTTP 메서드-GET,POST

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
### 1xx(Informational)

### 2xx(Successful)
  + 클라이언트의 요청을 성공적으로 처리
  + `200 OK` : 요청 성공
  + **201 Created** : 요청 성공하여 새로운 리소스 생성됌
    + `Location 헤더 필드` : 생성된 리소스의 URI 정보가 담겨있음 ex)Location:/members/100
  + **202 Accepted** : 요청 접수되었으나 처리가 완료 X
    + 배치 처리에서 사용
  + **204 No Content** : 서버가 요청을 성공적으로 수행했지만, 응답 페이로드 본문에 보낼 데이터가 X
    + ex) 웹 문서 편집기에서의 Save 버튼 => 결과 내용이 필요 X, 성공 여부만 중요할때!
   
### 3xx(Redirection)
  + 요청을 완료하기 위해, 유저 에이전트(클라이언트 프로그램)의 추가 조치 필요
  
> :question: Redirection
  + 웹 브라우저는 3xx 응답 결과에 Location 헤더가 존재 => Location 위치로 자동 이동(redirect)
  
> Redirection의 종류  
**1) 영구 리다이렉션 : 301,308**</br>
  + 리소스의 URI가 영구적으로 이동하여 원래 URL를 사용X(검색엔진에서 변경 인지)
  + `
