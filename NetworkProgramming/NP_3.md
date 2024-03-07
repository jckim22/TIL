### 이더넷 패킷 구성
- 이더넷 헤더 - 6바이트
- 이더넷 데이터 - ~~~
- 뒤에 오는 것에 대한 정보 - 2바이트

### IP 패킷
- IP header length -> IP는 20/4로 보내서 5로 보내지만 받는 쪽에서는 x4를 해주는 것으로 약속이 되어 있다.
	-> 이런 약속을 프로토콜이라고 한다.

- Tos -> 현대에서는 라우터 성능이 좋아서 서비스 품질(우선순위)와 관련된 Tos(Type of Service)는 항상 0이다.
	
- Total-Length -> 16비트 할당(전체 IP 패킷이 6만 정도의 크기까지 가능할 것이라고 가정)

- Fragment identifier 필드 (16bit -> 분열이 발생한 경우, 조각을 다시 결합하기 원래의 데이터를 식별하기 위해서 사용한다.

- Fragmentation Flags 필드 (3bit) -> 처음 1bit는은 항상 0으로 설정, 나머지 2비트의 용도는 다음과 같다.

- May Fragment -> IP 라우터에 의해 분열되는 여부를 나타낸다. 플래그 0 - 분열 가능 1 - 분열 방지
- More Fragments -> 원래 데이터의 분열된 조각이 더 있는지 여부 판단. 
	 				플래그 0 - 마지막 조각, 기본값 1- 조각이 더 있음

- Fragmentation Offset 필드 (13bit) -> 8바이트 오프셋으로 조각에 저장된 원래 데이터의 바이트 범위를 나타낸다.
	하지만 요즘 시대에서 TCP에서 분열되어서 내려오기 때문에 IP 자체에서 Fragment를 진행할 일이 없다.

- Time-to-live 필드(8bit) -> 데이터을 전달할 수 없는 것으로 판단되어 소멸되기 이전에 데이터가 이동할 수 있는 단계의 수를 나타낸다.

- Time-to-Live 필드는 1에서 255사이의 값을 지정하며 라우터들은 패킷을 전달 할 때마다 이 값을 하나씩 감소시킨다.
     			        TTL은 홉이 진행될 때마다 1씩 줄어든다.
				    ->라우팅이 잘못 될 경우 사이클이 발생 -> TTL로 방지

- Protocol Identifier 필드(8bit) -> 상위 계층 프로토콜(뒤에 뭐가 있는지)
				1 - ICMP, 2 - IGMP, 6 - TCP, 17 - UDP

- Header Checksum 필드(16bit) -> IP 헤더의 체크섬을 저장, 라우터를 지나갈때 마다 재 계산을 하기 때문에 속도가 떨어진다.
				-> 헤더의 데이터에 오류가 있는지 확인하기 위함
				-> 1의 보수를 취하는 이유 -> 받는 쪽에서 체크섬 포함해서 다 계산했을 때 1111이 나오면 ok 이므로 계산이 편해진다.
				-> 에러가 났는데도 체크섬을 통과할 수도 있다.
				-> 그런데도 사용하는 이유 -> 에러가 잘 안나고 체크섬 외에 에러를 잡는 시스템이 잘 되어 있다. ( ex: 이더넷의 CRC 덕분에 에러가 안 남)

- Source IP Address 필드(32bit) -> 출발지 IP 주소

- Destiantion IP Address 필드(32bit) -> 목적지 IP 주소

- Options(선택적) 필드(가변적) -> Type-of-Service 플래그 처럼 특별한 처리 옵션을 추가로 정의 할 수 있다.


### 3계층 -> (네트워크, 인터넷, IP) 계층

### 4계층 -> TransPort(전송) 계층
- TCP
		순서대로 보내고 받는 것이 목적(Sequence number)
		Sequence number
		-> 1 + 100byte
		-> 101 +250byte
		-> 351 + ~~byte
	tcp는 오류를 검증하는 기능만 있지 오류를 고치지는 못한다.
	다시 요청하게 된다.
	UDP는 이런 Sequence number가 붙지 않기 때문에 빠르고 오버헤드가 적다
	UDP도 오류 검출 가능.(CheckSum 존재)

>TCP -> High-Overhead -> reliable protocol(무결성) -> 신뢰 있는 프로토콜
	UDP -> low-Overhead -> unreliable protocol -> 신뢰성이 떨어지는 프로토콜(하지만 우리 생각보다 훨씬 유용하다)

### 5계층 -> Application(응용) Layer
- HTTP -> for World Wide Web
- SMTP, POP, IMAP -> for email
- FTP, FSP, TFTP -> for file transfer
- NFS -> for file access
	
### IP Addresses and Domain Names
	
- Domain Name
	www.naver.com <-> ~~~.~~~.~~~.~~~
	DHCP 서버를 먼저 찾고 IP를 알려준다.
	DHCP 서버를 찾는 방법
	-> 인터넷 접속을 하면
	-> DHCP가 있는지 방송을 한다.
	-> DHCP 서버가 IP를 줌(공유기)
	
### Ipv6
- 0이 너무 많아서 앞에 오는 0을 :으로 대체
### localhost
- 7.0.01
### broadcast -> DHCP 서버에서 누가 있냐고 물어볼 때
- 255.255.255.255

### Port -> 4계층 패킷 안에
>4계층과 5계층 안에 필요함 -> Application을 구분함
	src와 dst의 포트 번호가 달라도 됨(각자 4,5계층의 소통에서 필요하기 때문)
	FTP -> 21
	SSH -> 22
	Telnet -> 23
	smtp -> 25
	HTTP -> 80
	HTTPS -> 443

### IPv4가 지금까지 쓰일 수 있는 이유

>앞의 네트워크를 나타는 것이 perfix
	뒤에 네트워크 안에 노드가 sufix
	그리고 NAT라는 기능(공유기)
	공유기는 진짜 IP 주소 하나로 가짜 IP 주소 여럿을 사용한다.
	
### RFC -> 표준 문서(IP, DHCP 등등)

### TCP에서 seq를 받는 방법	
>1. TCP에서 ACK 넘버로 다음에 받을 번호만 계속 요청한다.
2. ACK를 못 받았을 때 재전송
	

	

