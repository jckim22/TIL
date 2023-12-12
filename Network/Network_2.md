# HTTP

 ## HyperText Transfer Protocol
 
 ### HTTP 메시지에 모든 것을 전송
 - 이미지, 음성, 파일,HTML, TXT, JSON 등
 - 서버 간에 데이터를 주고받을 때도 HTTP를 사용
 
 - TCP: HTTP/1.1, HTTP/2
 - UDP: HTTP/3
 - 현재 HTTP/1.1 주로 사용
 - HTTP/2, HTTP/3 도 점점 증가
 
 # HTTP 특징
 - 클라이언트 서버 구조
 - 무상태 프로토콜(Stateless), 비연결성
 - HTTP 메시지
 - 단순함, 확장 가능
 
 
 
  ##  클라이언트 서버 구조
  
  - Request Response 구조
  - 클라이언트는 서버에 요청을 보내고, 응답을 대기
  - 서버가 요청에 대한 결과를 만들어서 응답
  
 ##  무상태 프로토콜(Stateless), 비연결성
 ####  Stateful
 
고객이 결제를 위해 점원과 대화하는데 점원은 그대로다.

> 고객: 노트북 얼마인가요?
점원: 100만원입니다.
고객: 2개 주세요
점원: 200만원입니다. (노트북이라는 상태 유지)

- 상태 유지: 중간에 다른 점원으로 바뀌면 안된다.
   - 항상 같은 서버를 유지해야한다. (중간에 서버가 장애가 나면?)

#### Stateless

고객이 결제를 위해 점원과 대화하는데 점원이 바뀐다.

> 고객: 노트북 얼마인가요?
점원: 100만원입니다.
고객: 2개 주세요
점원: 무엇을 2개 구입하십니까?
고객: 노트북 2개주세요
점원: 200만원입니다.

- 중간에 다른 점원으로 바뀌어도 된다.
   - 갑자기 고객이 증가해도 점원을 대거 투입할 수 있다.
   - 갑자기 클라이언트 요청이 증가해도 서버를 대거 투입할 수 있다.
- 무상태는 응답 서버를 쉽게 바꿀 수 있다. -> **무한한 서버 증설 가능**
   - 아무 서버나 호출해도 된다. (중간에 서버가 장애나도 OK)
- 실무 한계가 있다.
   - 모든 것을 무상태로 설계 할 수 있는 경우도 있고 없는 경우도 있다.
   - 무상태
      - 예) 로그인이 필요 없는 단순한 서비스 소개 화면
   - 상태 유지
      - 예) 로그인
   - 로그인한 사용자의 경우 로그인 상태를 서버에 유지
   - 브라우저 쿠키와 서버 세션 등을 사용해서 상태 유지
   - 상태 유지는 최소한만 사용

> 단점으로 -> 데이터를 너무 많이 사용한다.


#### 연결을 유지하는 모델(Connectionful)
- 예) TCP는 3way handshake로 연결을 하고 시작하고 유지한다.

> 서버 자원을 많이 소모


#### 연결을 유지하지 않는 모델(Connectionless)

- 예) IP, HTTP

> 요청 응답이 끝나면 연결을 끊어버림

- HTTP는 비연결형 모델
- 일반적으로 초 단위의 이하의 빠른 속도로 응답
- 1시간 동안 수천명이 서비스를 사용해도 서버에서 동시에 처리하는 요청은 수십개 이하로 매우 작음
	- 예) 웹 브라우저에서 계속 연속해서 검색 버튼을 누르지는 않음
- 서버 자원을 매우 효율적으로 사용할 수 있음
 
 ###### 한계
 
 - TCP/IP 연결을 새로 맺어야함 - 3 way handshake 시간 추가
 - 웹 브라우저로 사이트를 요청하면 HTML 뿐만 아니라 자바스크립트, css, 추가 이미지 등 수 많은 자원이 함계 다운로드
 - 지금은 HTTP 지속 연결(Persistent Connections)로 문제 해결
    - 요청 후 자원을 쭉 응답 받고 연결을 종료함
 - HTTP/2, HTTP/2에서 더 많은 최적화
 - 정말 같은 시간에 딱 맞추어 발생하는 대용량 트래픽(티켓팅, 예약, 수강신청 등)
 
 
 
  ## HTTP 메시지
  
  # 시작 라인
  - 요청 메시지
    - start-line = request-line / status-line
    - request-line = method SP(공백) request-target(절대 경로) SP HTTP-version CRLF(엔터)
  
  
  
  # HTTP 헤더
  
  - header-field = field-name ":" OWS field-value OWS(띄어쓰기 허용)
  
  - 용도
     - 메시지 바디으 내용, 크기, 압축, 인증, 요청 클라이언트 정보, 서버 애플리케이션 정보, 캐시 관리 정보
    
 # HTTP 바디   
 
 HTML, JSON, TEXT ....
  
      
  
  
 ## 단순함, 확장 가능
 
 - HTTP는 단순하다, 스펙도 읽어볼만하다
 - HTTP 메시지도 매우 단순
 - 크게 성공하는 표준 기술은 단순하지만 확장 가능한 기술
 
 
 
 # HTTP 메서드
 
 ### HTTP API
 
 회원 정보관리 API를 만들라는 요구사항이 들어왔다.
 
 - 요구사항 확인
    - 회원 목록 조회
    - 회원 조회
    - 회원 등록
    - 회원 수정
    - 회원 삭제
    
 - API URI(Uniform Resource Identifier) 설계(x)
    - 회원 목록 조회 /read-member-list
    - 회원 조회 /read-member-by-id
    - 회원 등록 /create-member
    - 회원 수정 /update-member
    - 회원 삭제 /delete-member
   
   > 이것이 좋은 URI 설계일까 ??
   
   가장 중요한 것 **리소스 식별**
   
 - API URI 고민
    - 리소스의 의미는 뭘까?
       - 회원을 등록하고 수정하고 조회하는게 리소스가 아니다 !
       - 예) 미네랄을 캐라 -> 미네랄이 리소스
       - 회원이라는 개념 자체가 바로 리소스다.
	- 리소스를 어떻게 식별하는게 좋을까?
       - 회원을 등록하고 수정하고 조회하는 것을 모두 배제
       - 회원이라는 리소르만 식별하면 된다. -> 회원 리소스를 URI에 매핑
       
 - API URI(Uniform Resource Identifier) 설계(O)
    - 회원 목록 조회 /members
    - 회원 조회 /members/{id}
    - 회원 등록 /members/{id}
    - 회원 수정 /members/{id}
    - 회원 삭제 /members/{id}
    
    -> 어떻게 구분하지?
    
    > 계층 구조상 상위를 컬렉션으로 보고 복수단어 사용 권장(member -> members)
    
    
 - 리스소와 행위를 분리
    - URI는 리소스만 식별 !
    - 리소스: 회원
    - 행위: 조회, 등록, 삭제, 변경
    - 행위(메서드)는 어떻게 구분??? -> HTTP 메서드
        
        
### HTTP 메서드 종류
- GET: 리소스 조회
- POST: 요청 데이터 처리, 주로 등록에 사용
- PUT: 리소스를 대체, 해당 리소스가 없으면 생성(디렉토리에 파일을 넣는 것이랑 같음)
- PATCH: 리소스 부분 변경
- DELETE: 리소스 삭제

- HEAD: GET과 동일하지만 메시지 부분 제외 상태 줄과 헤더와 반환
- OPTIONS: 대상 리소스에 대한 통신 가능 옵션(메서들)을 설명(주로 CORS에서 사용)
- CONNECT: 대상 자원으로 식별되는 서버에 대한 터널을 설정
- TRACE: 대상 리소스에 대한 경로를 따라 메시지 루프백 테스트를 수행(요청 헤더를 다시 받음)
 
 ### GET, POST
 
 ###### GET
 
 - 리소스 조회
 - 서버에 전달하고 싶은 데이터는 query를 통해서 전달
 - 메시지 바디를 사용해서 데이터를 전달할 수 있지만, 지원하지 않는 곳이 많아서 권장X
 
 
 ###### POST
 
  - 요청 데이터 처리
 - 메시지 바디를 통해 서버로 요청 데이터 전달
 - 서버는 요청 데이터를 처리
    - 메시지 바디를 통해 들어온 데이터를 처리하는 모든 기능을 수행한다.
    - 주문 -> 결제완료 -> 배달중 -> 배달완료처럼 단순히 값 변경을 넘어 프로세스의 상태가 변경되는 경우에도 사용
- 주로 전달된 데이터로 신규 리소스 등록, 프로세스 처리에 사용

- 다른 메서드로 처리하기 애매한 경우
   - 애매하면 POST

 ### PUT, PATCH, DELETE
 
  ###### PUT
  
  - 리소스를 대체
  - 리소스를 생성
  - 쉽게 말해서 덮어버림
  
  - 클라이언트가 리소스 위치를 알고 URI를 지정한다는 점에서 POST와 큰 차이점이 있다.
  POST: POST /members
  PUT:  PUT /members/100 -> 100번임을 알고 있음
 
   ###### PATCH
   
   - 리소스 부분 변경
   
   ###### DELETE
   
   - 리소스 제거
 
 
 ### HTTP 메서드의 속성
 
 - 안전
    - 호출해도 리소스를 변경하지 않는다.  
    
 - 멱등
    - 한 번 호출하든 두 번 호출하든 100번 호출하든 결과가 똑같다
    - GET: 조회는 몇번을 하든 
    - PUT: 결과를 대체하기 떄문
    - DELETE: 결과를 삭제하기 때문
    
    -> 자동 복구 메커니즘 (실패해도 다시 요청해도 문제 없음)
    
    - POST: **멱등이 아님** 데이터는 계속 추가될 것임
    - PATCH: 설계에 따라 다름 호출 때마다 +10이 되는 설계라면 멱등성 위반
    
    
 - 캐시가능
 
    - 응답 결과 리소스를 캐시해서 사용해도 되는가?
    - GET, HEAD, POST, PATCH 캐시 가능
    - 실제로는 GET, HEAD 정도만 캐시로 사용
       - POST, PATCH는 본문 내용까지 캐시 키로 구분해야하는데 쉽지 않음
       
       
# HTTP 메서드 활용

### 클라이언트에서 서버로 데이터 전송

- 쿼리 파라미터를 통한 데이터 전송
   - GET
   - 주로 정렬 필터(검색어)
- 메시지 바디를 통한 데이터 전송
   - POST, PUT, PATCH
   - 회원 가입, 상품 주문, 리소스 등록, 리소스 변경
   

#### 정적 데이터 조회

쿼리 파라미터 미사용

#### 동적 데이터 조회

쿼리 파라미터 사용 (?key=value)
   
 
### HTML Form 데이터 전송

- HTML Form 전송은 GET, POST만 지원
- COntent-Type: application/x-www-form-urlfencoded
- COntent-Type: application/form-data

### HTYP API 데이터 전송

```
POST/ members HTTP/1.1
Content-Type: application/json
{
	"username": "young",
    "age": 20
}
```

- 서버 to 서버
   - 백엔드 시스템 통신
- 앱 클라이언트
   - 아이폰, 안드로이드
- 웹 클라이언트
   - HTML에서 Form 전송 대신 자바 스크립트를 통한 통신에 사용(AJAX)
   - 예) React, VueJS 같은 웹 클라이언트와 API 통신
- POST, PUT, PATCH: 메시지 바디를 통해 데이터 전송
- GET: 조회, 쿼리 파라미터로 데이터 전달
- COntent-Type: application/json 주로 사용


# HTTP API 설계 예시

- HTTP API - 컬렉션
   - POST 기반 등록
   - 예) 회원 관리 API 제공
- HTTP API - 스토어
   - PUT 기반 등록
   - 예) 정적 컨텐츠 관리, 원격 파일 관리
- HTML FORM 사용
   - 웹 페이지 회원 관리
   - GET, POST만 지원


# 회원 관리 시스템

### API 설계 
#### POST 기반 등록

- 회원 목록 /members -> GET
- 회원 목록 /members -> POST
- 회원 목록 /members{id} -> GET
- 회원 목록 /members{id} -> PATCH, PUT, Post
- 회원 목록 /members{id} -> DELETE

#### 신규 자원 등록 특징

- 클라이언트는 등록될 리소스의 URI를 모른다.
- 서버가 새로 등록된 리소스 URI를 생성해준다.
- 컬렉션
   - 서버가 관리하는 리소스 디렉토리
   - 서버가 리소스의 URI를  생성하고 관리
   - 여기서 컬렉션은 /members


### 파일 관리 시스템 
#### API 설계 - PUT기반 등록 (디렉토리에 파일 넣고 덮어쓰는 걸 생각)

- 파일 목록 /files -> GET
- 파일 조회 /files{filename} -> GET
- 파일 등록 /files{filename} -> PUT 
- 파일 삭제 /files{filename} -> DELETE
- 파일 대량 등록 /files -> POST

#### 신규 자원 등록 특징

- 클라이언트는 등록될 리소스의 URI를 안다.
- 클라이언트가 직접 리소스의 URI를 지정한다.
- 스토어
   - 클라이언트가 관리하는 리소스 저장소
   - 클라이언트가 리소스의 URI를 알고 관리
   - 여기서 스토어는 /files
   
### HTML FORM 사용

- HTML FORM은 GET, POST만 지원
- AJAX 같은 기술을 사용해서 해결 가능 -> 회원 API 참고
- 여기서는 순수 HTML, HTML FORM 이야기
- GET, POST만 지원하므로 제약이 있음

#### 설계

- 회원 목록 /members -> GET
- 회원 등록 폼 /members/new -> GET
- 회원 등록 /members/new, /members -> POST
- 회원 조회 /members{id} -> GET
- 회원 수정 폼 /members{id}/edit -> GET
- 회원 수정 /members{id}/edit, members/{id} -> POST
- 회원 삭제 /members{id}/delete -> POST


# 정리

- HTTP API - 컬렉션
   - POST 기반 등록
   - 서버가 리소스 URI 결정
- HTTP API - 스토어
   - PUT 기반 등록
   - 클라이언트가 리소스 URI 결정
- HTML FORM 사용
   - 순수 HTML + HTML form 사용
   - GET, POST만 지원
   
   
# 참고하면 좋은 URI 설계 개념

- 문서(document) 
   - 단일 개념(파일 하나, 객체 인스턴스, 데이터베이스 row)
   - 예) /members/100, /files/star.jpg
- 컬렉션(collection) 
   - 서버가 관리하는 리소스 디렉터리
   - 서버가 리소스의 URI를 생성하고 관리
   - 예) /members
- 스토어(store) 
   - 클라이언트가 관리하는 자원 저장소
   - 클라이언트가 리소스의 URI를 알고 관리
   - 예) /files
- 컨트롤러(controller), 컨트롤 URI 
   - 문서, 컬렉션, 스토어로 해결하기 어려운 추가 프로세스 실행
   - 동사를 직접 사용
   - 예) /members/{id}/delete


# HTTP 상태 코드

- 1xx (Informational): 요청이 수신되어 처리중
- 2xx (Successful): 요청 정상 처리
- 3xx (Redirection): 요청을 완료하려면 추가 행동이 필요
- 4xx (Client Error): 클라이언트 오류, 잘못된 문법등으로 서버가 요청을 수행할 수 없음
- 5xx (Server Error): 서버 오류, 서버가 정상 요청을 처리하지 못함


## 1xx
거의 사용되지 않음

## 2xx - 성공
- 200 OK
   - 요청 성공
- 201 Created
   - 요청 성공해서 새로운 리소스가 생성됨 (members/100)
   > 웹 브라우저는 3XX 응답의 결과에 Location 헤더가 있으면, Location 위치로 자동 이동(리다이렉트)
   
- 202 Accepted
   - 요청 접수되었으나 처리가 완료되지 않았음
- 204 No Content
   - 서버가 요청을 서공적으로 수행했지만, 응답 페이로드 본문에 보낼 데이터가 없음(편집 후 저장을 했을 때 아무 내용이 없어도 된다.)

## 3xx - 리다이렉션

> 웹 브라우저는 3XX 응답의 결과에 Location 헤더가 있으면, Location 위치로 자동 이동(리다이렉트)

- 영구 리다이렉션 - 특정 리소스의 URI가 영구적으로 이동
   -  예) /members -> /users
   -  예) /event -> /new-event
- 일시 리다이렉션 - 일시적인 변경
   -  주문 완료 후 주문 내역 화면으로 이동
   -  PRG: Post/Redirect/Get
- 특수 리다이렉션
   -  결과 대신 캐시를 사용

- 300 Multiple Choices
- 301 Moved Permanently
- 302 Found
- 303 See Other
- 304 Not Modified
- 307 Temporary Redirect
- 308 Permanent Redirect


#### 영구 리다이렉션
301, 308
• 리소스의 URI가 영구적으로 이동
• 원래의 URL를 사용X, 검색 엔진 등에서도 변경 인지

• 301 Moved Permanently 
• 리다이렉트시 요청 메서드가 GET으로 변하고, 본문이 제거될 수 있음(MAY) 

• 308 Permanent Redirect 
• 301과 기능은 같음
• 리다이렉트시 요청 메서드와 본문 유지(처음 POST를 보내면 리다이렉트도 POST 유지)


#### 일시적인 리다이렉션
302, 307, 303
• 리소스의 URI가 일시적으로 변경
• 따라서 검색 엔진 등에서 URL을 변경하면 안됨

• 302 Found 
• 리다이렉트시 요청 메서드가 GET으로 변하고, 본문이 제거될 수 있음(MAY) 

• 307 Temporary Redirect 
• 302와 기능은 같음
• 리다이렉트시 요청 메서드와 본문 유지(요청 메서드를 변경하면 안된다. MUST NOT) 

• 303 See Other 
• 302와 기능은 같음
• 리다이렉트시 요청 메서드가 GET으로 변경


##### PRF: Post/Redirect/Get
일시적인 리다이렉션 - 예시

- POST로 주문 후에 웹 브라우저를 새로고침하면?
- 새로고침은 다시 요청
- 중복 주문이 될 수 있다.

주문하고 ok가 되어서 데이터베이스에 주문 데이터가 저장 되고 새로고침을 누른다면 또 POST 요청이 가게 된다.
그래서 POST로 주문 후에 결과 화면을 GET으로 리다이렉트 한다.

#### 기타 리다이렉션
300 304

• 300 Multiple Choices: 안쓴다.

• 304 Not Modified
• 캐시를 목적으로 사용
• 클라이언트에게 리소스가 수정되지 않았음을 알려준다. 따라서 클라이언트는 로컬PC에
저장된 캐시를 재사용한다. (캐시로 리다이렉트 한다.)
• 304 응답은 응답에 메시지 바디를 포함하면 안된다. (로컬 캐시를 사용해야 하므로)
• 조건부 GET, HEAD 요청시 사용


## 4xx - 클라이언트 오류

클라이언트 오류
• 클라이언트의 요청에 잘못된 문법등으로 서버가 요청을 수행할 수 없음
• 오류의 원인이 클라이언트에 있음
• 중요! 클라이언트가 이미 잘못된 요청, 데이터를 보내고 있기 때문에, 똑같은 재시도가 실
패함


#### 400 - Bad Request
요청 구문, 메시지 등등 오류
클라이언트는 요청 내용을 검토하고 다시 요청해야함

#### 401 Unauthorized
클라이언트가 해당 리소스에 대한 인증이 필요함

> 인증(Authentication): 본인이 누구인지 확인(로그인)
인가(Authorization): 권한부여 (ADMIN 권한처럼 특정 리소스에 접근할 수 있는 권한, 인증이 있어야 인가가 있음)

#### 403 Forbidden
- 인증 자격은 있지만, 권한이 불충분함
- ADMIN 권한은 아님

### 404 Not Found
- 요청 리소스를 찾을 수 없음

## 5xx - 서버 오류

- 서버 문제로 오류 발생
- 서버에 문제가 있기 때문에 재시도 하면 성공할 수도 있음

#### 500 Internal Server Error

- 서버 문제로 오류 발생, 애매하면 500 오류

#### 503 Service Unacailable

- 서비스 일시적인 이용불가(과부화, 예정된 작업으로 잠시 요청을 처리할 수 없음)

- 서버 문제로 오류 발생
• 서버에 문제가 있기 때문에 재시도 하면 성공할 수도 있음(복구가 되거나 등등)


# HTTP 헤더
## 일반 헤더

#### HTTP 바디

• 메시지 본문(message body)을 통해 표현 데이터 전달
• 메시지 본문 = 페이로드(payload)
• 표현은 요청이나 응답에서 전달할 실제 데이터
• 표현 헤더는 표현 데이터를 해석할 수 있는 정보 제공
• 데이터 유형(html, json), 데이터 길이, 압축 정보 등등
• 참고: 표현 헤더는 표현 메타데이터와, 페이로드 메시지를 구분해야 하지만, 여기서는 생략


#### 표현

• Content-Type: 표현 데이터의 형식
• Content-Encoding: 표현 데이터의 압축 방식
• Content-Language: 표현 데이터의 자연 언어
• Content-Length: 표현 데이터의 길이
• 표현 헤더는 전송, 응답 둘다 사용


#### 협상(콘텐츠 네고시에이션)
클라이언트가 선호하는 표현 요청

• Accept: 클라이언트가 선호하는 미디어 타입 전달
• Accept-Charset: 클라이언트가 선호하는 문자 인코딩
• Accept-Encoding: 클라이언트가 선호하는 압축 인코딩
• Accept-Language: 클라이언트가 선호하는 자연 언어
• 협상 헤더는 요청시에만 사용

클라이언트의 요청에 따라 서버가 우선순위를 잡고 응답을 보내줌

한국어 브라우저가 영어 서버에 보냈는데 Accept-Language가 없으면 한국어 지원이 되어도 영어를 보내줄 것이다.


#### 협상과 우선순위1 - Quality Values(q)

• Quality Values(q) 값 사용
• 0~1, 클수록 높은 우선순위
• 생략하면 1
• Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
• 1. ko-KR;q=1 (q생략)
• 2. ko;q=0.9
• 3. en-US;q=0.8
• 4. en;q=0.7


#### 협상과 우선순위2 - Quality Values(q)

• 구체적인 것이 우선한다.
• Accept: text/*, text/plain, text/plain;format=flowed, */*
1. text/plain;format=flowed
2. text/plain
3. text/*
4. */*


> 협상 헤더는 요청시에만 사용한다 !

# 전송 방식

- Transfer-Encoding
- Range, Content-Range

## 전송 방식 설명

- 단순 전송
Content-Length 만큼 전송
- 압축 전송
Content-Encoding으로 뭐로 압축했는지 보내고
Length만큼 전송
- 분할 전송
Transfer-Encoding: chunked로 덩어리로 쪼개서 전송
- 범위 전송
Range, Content-Range
범위만큼 요청받고 범위만큼 전송한다.


## 일반 정보

- From: 유저 에이전트의 이메일 정보
- Referer: 이전 웹페이지 주소
- User-Agent: 유저 에이전트 애플리케이션 정보
- Server: 요청을 처리하는 오리진 서버의 소프트웨어 정보
- Date: 메시지가 생성된 날짜


## 특별한 정보

- Host: 요청한 호스트 정보(도메인)
요청에서 사용
필수
하나의 서버가 여러 도메인을 처리해야 할 때
하나의 IP 주소에 여러 도메인이 적용되어 있을 때


- Location: 페이지 리다이렉션
- Allow: 허용 가능한 HTTP 메서드
- Retry-After: 유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간
- 쿠키
   - HTTP는 무상태(Stateless) 프로토콜이다
   - 그래서 모든 요청에 사용자 정보를 포함한다.
   - 사용자 정보가 포함되도록 개발을 해야함
   - 쿠키는 항상 서버에 전송되어서 네트워크 트래픽 추가 유발한다.
   - 그래서 최소한의 정보만 사용한다(세션 id, 인증 토큰)
   - 서버에 전송하지 않고, 웹 브라우저 내부에 데이터를 저장하고 싶으면 웹 스토리지를 사용한다.
   
   
# 캐시와 조건부 요청
cache-control: max-age=60
-> 60초 동안 유효한 캐시를 저장해서 네트워크를 사용하지 않아도 된다.
-> 로딩 속도가 빨라진다.

## 캐시 시간 초과
- 캐시 유효 시간이 초과하면 다음 2가지 상황이 일어남

1. 서버에서 기존 데이터를 변경함
2. 서버에서 기존 데이터를 변경하지 않음

### 데이터 변경 x
다시 쓰면 된다.

##### 검증 헤더 추가
Last-Modified: 2020-11-10
-> 60초 지나고 If-modified-since(조건부 요청)를 다시 보냄
근데 서버에서 데이터 최종 수정일을 검사했을 때 같다면 데이터가 아직 수정되지 않은 것임
서버에서 304 Not Modified를 보냄 -> 데이터 재사용


• 캐시 유효 시간이 초과해도, 서버의 데이터가 갱신되지 않으면
• 304 Not Modified + 헤더 메타 정보만 응답(바디X)
• 클라이언트는 서버가 보낸 응답 헤더 정보로 캐시의 메타 정보를 갱신
• 클라이언트는 캐시에 저장되어 있는 데이터 재활용
• 결과적으로 네트워크 다운로드가 발생하지만 용량이 적은 헤더 정보만 다운로드
• 매우 실용적인 해결책



## 검증 헤더
 검증 헤더 (Validator) 
• ETag: "v1.0", ETag: "asid93jkrh2l"
• Last-Modified: Thu, 04 Jun 2020 07:19:24 GMT

## 조건부 요청 헤더

조건부 요청 헤더
• If-Match, If-None-Match: ETag 값 사용
• If-Modified-Since, If-Unmodified-Since: Last-Modified 값 사용

#### If-Modified-Since: 이후에 데이터가 수정되었으면?
• 데이터 미변경 예시
• 캐시: 2020년 11월 10일 10:00:00 vs 서버: 2020년 11월 10일 10:00:00
• 304 Not Modified, 헤더 데이터만 전송(BODY 미포함)
• 전송 용량 0.1M (헤더 0.1M)

• 데이터 변경 예시
• 캐시: 2020년 11월 10일 10:00:00 vs 서버: 2020년 11월 10일 11:00:00
• 200 OK, 모든 데이터 전송(BODY 포함)
• 전송 용량 1.1M (헤더 0.1M, 바디 1.0M)

#### Last-Modified, If-Modified-Since 단점
- 1초 미만 단위로 캐시 조정이 불가능
- 날짜 기반의 로직 사용
- 데이터를 수정해서 날짜가 다르지만, 같은 데이터를 수정해서 데이터 결과가 똑같은 경우
- 서버에서 별도의 캐시 로직을 관리하고 싶은 경우
   - 예) 스페이스나 주석처럼 크게 영향이 없는 변경에서 캐시를 유지하고 싶은 경우

ETag(Entity Tag)
• 캐시용 데이터에 임의의 고유한 버전 이름을 달아둠
• 예) ETag: "v1.0", ETag: "a2jiodwjekjl3"
• 데이터가 변경되면 이 이름을 바꾸어서 변경함(Hash를 다시 생성)
• 예) ETag: "aaaaa" -> ETag: "bbbbb"

• 진짜 단순하게 ETag만 서버에 보내서 같으면 유지, 다르면 다시 받기!
• 캐시 제어 로직을 서버에서 완전히 관리
• 클라이언트는 단순히 이 값을 서버에 제공(클라이언트는 캐시 메커니즘을 모름)
• 예)
• 서버는 배타 오픈 기간인 3일 동안 파일이 변경되어도 ETag를 동일하게 유지
• 애플리케이션 배포 주기에 맞추어 ETag 모두 갱신


# 캐시 제어 헤더

### Cache-Control
max-age
- 캐시 유효 시간, 초 단위
no-cache
- 데이터는 캐시해도 되지만, 항상 origin 서버에 검증하고 사용
no-store
- 데이터에 민감한 정보가 있으므로 저장하면 안됨


### Pragma
no-cache

### Expires
캐시 만료일 지정 -> max-age 하위호환


# 프록시 캐시

### origin 서버 직접 접근
먼 나라의 사람들이 500ms라는 긴 시간을 기다려야함

### 프록시 캐시 도입

프록시 캐시 서버에서 400ms 걸려서 받고
먼 나라의 사람은 100ms 걸려서 받는다.

예) 사람들이 자주 보는 유트브를 보면 빠른데, 잘 안보는 유튜브 보면 느리다(프록시 서버 때문)

# 캐시 무효화

Cache-Control: no-cache,  no-store, must-revalidate
Pragma: no-cache (하위 호환)
