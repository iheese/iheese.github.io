---
layout: post
title: '네트워크 Interview 대비'
subtitle: 'HTTP, HTTPS, SSL, Handshake, REST, DNS, TCP, UDP, OSI7, TCP/IP'
date: 2022-11-09 12:00:00 +0900
categories: 'ETC'
background: '/img/posts/etc/git.png'
---



#### HTTP 프로토콜이란 무엇인가요?
- HTTP(Hyper Text Transfer Protocol)이란 데이터를 주고 받기 위한 프로콜이며 서버,클라이너트 모델을 따릅니다. 상태정보를 저장하지 않는 Stateless, 클라이언트의 요청에 맞는 응답을 보낸 후 연결을 끊는 Connectionless의 특징을 가지고 있습니다.
- 장점
> - 연결 상태 처리, 상태 정보를 관리할 필요가 없어 서버 디자인이 간단하다.
> - 각각의 HTTP 요청에 독립적으로 응답한다.
- 단점
> - 이전 통신의 정보를 모르기 때문에 매번 인증이 필요하고 이를 해결하기 위해 세션과 쿠키를 이용한다. 

<BR>

#### HTTP1 vS HTTP2 차이는 어떻게 되나요?
- HTTP1은 기본적으로 연결당 하나의 요청과 응답을 처리하기 때문에 동시 전송 문제와 다수의 리소스를 처리하기에 속도와 성능 이슈를 가지고 있습니다. 
> - HOL(Head Of Line) Blocking - 특정 응답 지연
> - RTT(Round Trip Time) 증가
> - 헤비한 Header 구조
- HTTP2는 성능 뿐만 아니라 속도면에서 월등합니다.
> - Multiplexed Streams : 한 커넥션에 여러 개의 메세지를 동시에 주고 받을 수 있습니다.
> - Stream Prioritization : 요청 리소스 간의 의존 관계를 설정
> - Server Push : HTML 문서 상에 필요한 리소스를 클라이언트 요청없이 보내줄 수 있습니다.
> - Header Compression : Header 정보를 HPACK 압충 방식을 이용해 압축 전송합니다.

<BR> 

#### HTTP와 HTTPS의 차이점에 대해서 설명해보세요.
- HTTP는 평문 데이터를 전송하는 프로토콜이므로 중간에 정보를 가로챌 수 있습니다. 이를 보완하기 위해 나온 것이 HTTPS입니다. 
- HTTPS는 SSL(Secure Socket Layer, 인터넷을 통해 전달되는 정보를 보호하기 위해 개발한 통신 규약)으로 암호화한 HTTP를 의미합니다.
> - TCP를 기반으로 만들어진 패킷이 HTTP이다. 

<br>

#### HTTPS에 대해서 설명하고 SSL Handshake에 대해서 설명해보세요.
- HTTP에 SSL 보안 계층을 추가한 것을 의미한다. HTTPS는 인증, 공개키 암호화, 비밀키 암호화를 사용한다. 
- 공개키로 암호화되면 개인키로 복호화 가능, 개인키로 암호화되면 공개키로 복호화 가능하다. 
- 대칭키(비밀키) 하나의 키로 암호화, 복호화 모두 처리한다. 
- SSL Handshake 단계
> - 클라이언트는 TCP 3way handshake를 수행하고 Client Hello를 전송한다, 암호화 알고리즘 목록 전달 `->` 서버 
> - 클라이언트 `<-` Server Hello, 암호화 알고리즘 선택, 인증서 전달
> > - 인증서에는 서버가 발행한 공개키가 들어 있고, CA(Certificate Authority, 인증기관)의 개인키로 암호화되어 있다. 
> > - 클라이언트는 CA의 공개키로 인증서 검증을 한다.
> - 검증 후 클라이언트 대칭키(비밀키) 생성하고 SSL 인증서 내부에 있던 공개키로 암호화하여 대칭키를 전달 `->` 서버
> - 이 후 대칭키를 통해 암호화하여 통신한다.  

<br>

#### GET과 POST의 차이점에 대해서 설명해보세요.
- GET 요청은 서버에 존재하는 정보를 요청합니다. 일반적으로 Body에 값을 입력하지 않고 헤더에 입력하는 것이 일반적이며, URL에 데이터가 노출되므로 보안이 중요한 데이터는 GET으로 요청하면 안된다.
> - 캐싱 가능, 멱등성 보장
- POTS 요청은 서버에 정보를 생성하는 것을 요청합니다. 일반적으로 Body에 데이터를 추가하여 전송합니다. URL에 데이터가 노출되지 않아 비교적 안전합니다. 
> - 캐싱 불가능, 멱등성 보장X


<br>

#### HTTP 메서드와 이것이 하는 역할에 대해서 설명해보세요.
- GET : 데이터 조회
- POST : 데이터 등록
- PUT : 데이터 변경, 존재하지 않으면 생성
- PATCH : 일부 데이터만 변경
- DELETE : 데이터 삭제

<BR>
#### 세션과 쿠키란 무엇인가요?
- 쿠키 : 클라이언트에 저장되는 작은 기록 정보 파일이다. HTTP에서 클라이언트의 상태 정보를 저장했다가 필요시 참조하거나 재사용할 수 있다.
> - text로 저장, 쿠키 저장시 만료 안되면 삭제 안됨, 세션보다 빠름, 보안에 취약
- 세션 : 일정 시간 동안 같은 사용자로부터 들어오는 일련의 요구를 하나로 보고, 그 상태를 유지시키는 것이다.
> - Object로 저장, 브라우저 종료시 삭제, 쿠키보다 느림, 보안에 비교적 좋음

<br>

#### RESTful이란 무엇이며, 이것에 대해서 아는대로 설명해보세요.
- HTTP URI를 통해 자원을 표시하고, HTTP Method를 통해 자원에 대한 처리를 표현하며, 자원의 상태(정보) 전달한다.(JSON or XML)
- REST(Representational State Transfer) 아키텍처 스타일을 모두 고수하는 것은 매우 어려운 일이다. (무상태, 계층화 시스템, 캐시 가능성, 온디맨드 코드), 분산 환경에 적합하지 않다, 표준이 존재하지 않는다.

<br>

#### 웹 통신의 큰 흐름: https://www.google.com/ 을 접속할 때 일어나는 일
- 사용자가 브라우저에 URL 입력
- DNS 서버에서 도메인 네임으로 서버의 진짜 주소 찾음
> - DNS(Domain Name System) : 도메인 이름을 실제 네트워크 상에서 사용하는 IP 주소로 바꿔주는 시스템, 계층 구조를 가지는 분산 데이터베이스 구조를 가진다.
- IP 주소로 웹서버에서 TCP 3 handshake로 연결 수립
- 클라이언트는 웹 서버로 HTTP 요청 메시지를 보냄
- 웹 서버는 HTTP 응답 메세지를 보냄
- 도착한 HTTP 응답 메세지는 웹 페이지 데이터로 변환되고 웹 브라우저에 의해 출력

<br>

### TCP와 UDP의 차이점에 대해서 설명해보세요.
- TCP(Transmission Control Protocol), 가상 회선을 이용하는 연결형 서비스로 3 way handshaking 과정을 통해 연결을 설정합니다. 높은 신뢰성을 보장하지만 속도가 비교적 느리다는 단점이 있다.
> - 순서 보장, 1:1 통신, 수신 여부 확인,  신뢰성 높음, 속도 느림
> - 흐름 제어, 혼잡 제어, 오류 제어 
> > - 파일 교환, 문자 등
- UDP(User Datagram Protocol), 데이터그램 방식을 사용하는 비연결형 서비스로 스트리밍, RTP(Real-time Transport Protocol)와 같이 실시간 연속성이 더 중요한 서비스에 사용된다. 비교적 속도가 빠르다.
> - 순서 보장X, 1:1 or 1:N or N:N 통신, 수신 여부 확인X, 신뢰성 낮음, 속도 빠름


<br>

#### TCP 3, 4 way handshake에 대해서 설명해보세요.
- TCP 3 way handshake는 가상회선을 수립하는 단계입니다. 클라이언트는 서버에 요청을 전송할 수 있는지, 서버는 클라이언트에게 응답을 전송할 수 있는지 확인하는 과정입니다. 
> - 접속 요청 : 클라이언트 SYN `->` 서버  
> - 요청 수락 : 클라이언트 `<-` SYN + ACK 서버
> - 연결 : 클라이언트 ACK `->` 서버 
> > - SYN : synchronize sequence numbers의 약자
> > - ACK : acknowledgment의 약자
- TCP 4 way handshake는 TCP연결을 해제하는 단계입니다. 클라이언트는 서버에게 연결해제를 통지하고 서버가 이를 확인하고 클라이언트에게 이를 받았음을 전송하고 최종적으로 연결이 해제됩니다. 
> - 연결 종료 요청 : 클라이언트 FIN플래그 `->` 서버
> - 요청 확인 : 클라이언트 `<-` ACK 서버
> - 연결 종료 : 클라이언트 `<-` FIN플래그 서버
> - 연결 종료 확인 : 클라이언트 ACK `->` 서버

<br>

#### OSI7계층과 그 존재 이유, TCP/IP 4계층에 대해 설명해보세요.
- OSI7 계층은 네트워크 통신을 구성하는 요소들을 7개의 계층으로 표준화한 것입니다. 통신이 일어나는 과정을 단계별로 파악할 수 있다는 장점이 있습니다.
> - 7 Application Layer : 사용자에게 통신하기 위한 서비스 제공, 인터페이스 역할 (ex : HTTP)
> - 6 Presentation Layer : 데이터의 형식을 정의하는 계층, 문자 코드, 압축, 암호화 등 데이터 변환 담당
> - 5 Session Layer : 세션 체결, 통신 방식 등을 결정(ex : FTP)
> - 4 Transport Layer : 최종 수신 프로세스로 데이터의 전송을 담당, 데이터 손실 확인, 포트로 식별을 담당 (단위 : Segment, ex : TCP, UDP)
> - 3 Network Layer : 다른 네트워크와 통신을 위한 경로 설정, 논리 주소 결정 (단위 : Packet, ex : Router)
> - 2 DataLink Layer : 네트워크 기기간 데이터 송수신, 물리 주소 결정 (단위 : Frame, ex: 이더넷)
> - 1 Physical Layer : 물리적 연결, 데이터를 전기 신호로 바꿔주는 계층 (단위 : bit, 장비 : 케이블, 리피터, 허브)

- `아A 파P 서S 탈T 났N 다D 픽PHY`
- 7 `->` 1 캡슐화(하나씩 쌓임), 1 `->` 7 역캡슐화 
- 우리가 실제로 사용하는 네트워크는 TCP/IP입니다.
> - 4 Application Layer : 세션, 프레젠테이션, 어플리케이션 계층에 해당한다.(5, 6, 7) (프로토콜 : HTTP, FTP, Telnet, DNS, SMTP)
> - 3 Transport Layer : 전송 계층(4), 네트워크 양단의 송수신 호스트 사이에서 신뢰성 있는 전송 기능을 제공한다. (프로토콜 : TCP, UDP)
> - 2 Internet Layer : 네트워크 계층(3), 상위 트랜스포트 계층으로부터 받은 데이터에 IP패킷 헤더를 붙여 IP 패킷을 만들어 전송하는 역할을 한다.(프로토콜 : IPv4, IPv6)
> - 1 Network Layer : 물리, 데이터링크(1, 2) 계층에 해당한다. 하드웨어적인 요소, (프로토콜 : Ehternet(이더넷), Token Ring, PPP)

<br>
#### CORS란 무엇이며 이것에 대해서 설명해보세요.
- 대게 프론트엔드 개발시 로컬에서 API 서버에 요청을 보낼 때 흔하게 발생합니다.
- 서로 다른 도메인 간에 자원을 공유하는 것을 뜻한다. 대부분 브라우저에서 이를 차단하며, 서버 측에서 헤더를 통해서 사용 가능한 자원을 알려줍니다.
- Preflight request는 실제 요청을 보내도 안전한지 판단하기 위해 사전에 보내는 요청입니다. OPTIONS 메소드로 요청하며 COSR를 허용하는지 확인합니다. CORS가 허용된 웹서버라면 사용 가능한 리소스를 헤더에 담아 응답합니다.

<br>

#### 웹 서버 소프트웨어(Apache, Nginx)는 OSI 7계층 중 어디서 작동하는지 설명해보세요. (설명 부족)
- 웹 서버 소프트웨어는 7계층인 Application Layer에서 작동합니다. 웹 서버는 HTTP 프로토콜을 이용하여 서버와 정보를 주고 받습니다. 

<br>
#### 웹 서버 소프트웨어(Apache, Nginx)의 서버 간 라우팅 기능은 OSI 7계층 중 어디서 작동하는지 설명해보세요.
- 서버 간 라우팅 기능은 3계층은 Network Layer에서 작동합니다. Packet을 통해 데이터를 전송합니다.


<br>
#### 추가  학습할 것

 HTTP1 vs HTTP2의 차이를 아시나요?
 [HTTP1.1 vs 2](https://seo-tory.tistory.com/82)

<br>

Reference:
- [Backend-Interview-Question_ ksundong](https://github.com/ksundong/backend-interview-question)

- [신입 개발자 기술면접 질문 정리-네트워크_슬기로운 개발 생활](https://dev-coco.tistory.com/161?category=1056309)
- [[ 네트워크 쉽게 이해하기 22편 ] TCP 3 Way-Handshake & 4 Way-Handshake_Mind Net](https://mindnet.tistory.com/entry/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-22%ED%8E%B8-TCP-3-WayHandshake-4-WayHandshake)
