# HTTP

## Internet Network
#### IP
+ `IP` : 인터넷 프로토콜로써, 지정한 IP Address에 **Packet**(통신 단위)로 데이터 전달
  + EX) 100.100.100.1 -> 200.200.200.2
  + Network Layer(3)의 프로토콜
  
#### IP의 한계
+ 비연결성 : 대상이 서비스 불능이여도 패킷 전송
+ 비신뢰성 : 패킷 소실, 전달 순서에 문제 발생 가능
+ 프로그램 구분 : 같은 IP 사용 서버에서 통신 애플리케이션이 둘 이상일 경우 구분 불가능
 
#### TCP, UDP
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
 
#### URI
+ Uniform Resource Identifier로써, 통일된 방식으로 식별 가능한 모든 자원(리소스)을 구분하는데 필요한 정보
  + `URL`: Resource Locator, 자원의 위치를 지정 ex)foo://example.com:8042
  + `URN`: Resource Name, 자원엥 이름을 부여  ex)urn:example:animal:
  
#### URL
+ **scheme**://[userinfo@]**host**[:port][/path][?query][#fragment]
  + https://www.google.com:443/search?q=hello&hl=ko
  + `query` : 웹 서버에 제공하는 파라미터, 문자 형태(쿼리 파라미터,쿼리스트링)
    + key = value의 형태로써, ?로 시작함 

## HTTP
  + Hypertext, 즉 모든 종류의 정보(html,이미지,JSON,XML 등)를 주고받을 수 있는 프로토콜

#### HTTP 특징
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
  
**4.HTTP 메시지**</br>
> 요청 메시지
1) Start Line : Request-line(HTTP 메서드 + 요청대상 + HTTP Version)</br>
  + `HTTP 메서드`: GET(리소스 조회)/POST(요청 내역 처리)
 
2) HTTP 헤더 : field-name: field-value ex) Host:www.google.com</br>
  + HTTP 전송에 필요한 모든 부가정보가 담겨 있음
  
3) HTTP 메시지 바디 : 실제 전송할 데이터가 담겨 있음</br>

> 응답 메시지
1) Start Line : Status-line(HTTP Version + **Status Code** + 이유 문구)
  + `Status code` : 200(성공)/400(클라이언트 요청 오류)/500(서버 내부 오류)


## HTTP 메서드

#### API URI 설계

:seedling: **리소스 식별, URI 계층 구조**를 활용해라!</br>
+ URI는 리소스만 식별, 행위(메서드)는 HTTP 메서드를 이용
  + `리소스` : 회원 조회에서, '회원'이라는 개념 자체
     + ex) 회원 조회/members/{id}, 회원 등록/members/{id}...
  + `행위` : 조회,등록,삭제,변경..  

#### HTTP 메서드-GET,POST

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

