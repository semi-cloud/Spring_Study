# HTTP

## Internet Network
#### IP
+ `IP` : 인터넷 프로토콜로써, 지정한 IP Address에 Packet(통신 단위)로 데이터 전달
  + EX) 100.100.100.1 -> 200.200.200.2
  + Network Layer(3)의 프로토콜
  
#### IP의 한계
+ 비연결성 : 대상이 서비스 불능이여도 패킷 전송
+ 비신뢰성 : 패킷 소실, 전달 순서에 문제 발생 가능
+ 프로그램 구분 : 같은 IP 사용 서버에서 통신 애플리케이션이 둘 이상일 경우 구분 불가능
 
#### TCP, UDP
+ `TCP` : 전송 제어 프로토콜로서, 3-way-handshake 연결과정 이후에 데이터 전송하므로 순서와 신뢰성 보장
  + `TCP Segment` : 출발지/도착지 PORT 정보, 전송 제어/순서/검증 정보,전송 데이터 등이 담겨있음
  + Transport Layer(4)의 프로토콜
   
> :bulb: 3-way-handshake 
 + Client > Server : TCP **SYN**
 + Server > Client : TCP **SYN ACK**
 + Client > Server : TCP **ACK**
  + `SYN` : 접속 요청, `ACK` : 요청 수락
  
+ `UDP` : 단순 데이터 전달로서, 전송과 순서가 보장되지 않지만, 단순하고 빠른 protocol(연결 과정X)

> :star2: PORT
  + 같은 IP 내에서 프로세스를 식별하는 논리적 단위, http: 80번 port 이용!
  ``` 
  어떠한 데이터가 송수신을 할 때
  Datalink 계층에서는 호스트의 NIC로 MAC Address를 판별하고 
  Network 계층에서는 IP Address로 목적지를 판별.
  이렇게 MAC Address와 IP Address를 통해 목적지 호스트까지 도달한 후에는
  어떤 Process(프로세스)에서 데이터를 받을 것인지 를 알아야 하는데 이 때 쓰이는 것이
  Port Number(포트 번호!)
  ```
 





