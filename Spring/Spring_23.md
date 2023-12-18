# 만약에

- html 폼으로 post 요청을 했는데 웹 애플리케이션 서버(WAS)를 내가 직접 구현해 한다면 ?
   - 서버 TCP/OP 연결 대기, 소켓 연결
   - HTTP 요청 메시지를 파싱해서 읽기
   - POST 방식, /save URL 인지
   - Content-Type 확인
   HTTP 메시지 바디 내용 파싱
      - username, age 데이터를 사용할 수 있게 파싱
   - 저장 프로세스 실행
   - 비즈니스 로직 실행
      - 데이터베이스에 저장 요청
   - HTTP 응답 메시지 생성 시작
      - HTTP 시작 라인 생성
      - Header 생성
      - 메시지 바디에 HTML 생성에서 입력
   - TCP/IP에 응답 전달, 소켓 종료

# 서블릿
## 특징

- urlPatterns(/hello)의 URL이 호출되면 서블릿 코드가 실행
- HTTP 요청 정보를 편리하게 사용할 수 있는 HttpServletRequest
- HTTP 응답 정보를 편리하게 제공할 수 있는 HttpServletResponse
- 개발자는 HTTP 스펙을 매우 편리하게 사용

## 서블릿 컨테이너

![](https://velog.velcdn.com/images/jckim22/post/4c26e3ca-0b83-4b18-a147-f2a756c0fea5/image.png)


- 톰캣처럼 서블릿을 지원하는 WAS를 서블릿 컨테이너라고 함
- 서블릿 컨테이너는 서블릿 객체를 생성, 초기화, 호출, 종료하는 생명주기 관리 
- 서블릿 객체는 싱글톤으로 관리
   - 고객의 요청이 올 때 마다 계속 객체를 생성하는 것은 비효율 
   - 최초 로딩 시점에 서블릿 객체를 미리 만들어두고 재활용
   - 모든 고객 요청은 동일한 서블릿 객체 인스턴스에 접근
   - 공유 변수 사용 주의
   - 서블릿 컨테이너 종료시 함께 종료
- JSP도 서블릿으로 변환 되어서 사용
- 동시 요청을 위한 멀티 쓰레드 처리 지원


# 동시 요청 - 멀티 쓰레드

![](https://velog.velcdn.com/images/jckim22/post/8127d1d3-ac54-40ad-9243-53414bd54b4b/image.png)

쓰레드가 호출한다.

## 쓰레드

- 애플리케이션 코드를 하나하나 순차적으로 실행하는 것은 쓰레드 
- 자바 메인 메서드를 처음 실행하면 main이라는 이름의 쓰레드가 실행 
- 쓰레드가 없다면 자바 애플리케이션 실행이 불가능
- 쓰레드는 한번에 하나의 코드 라인만 수행
- 동시 처리가 필요하면 쓰레드를 추가로 생성

![](https://velog.velcdn.com/images/jckim22/post/d8fdf743-b593-46bd-beef-e59f76cc07da/image.png)


그래서 요청 때마다 쓰레드를 생성한다.

![](https://velog.velcdn.com/images/jckim22/post/68b5a53f-f4f6-4ddd-87c6-dd66c5dac251/image.png)

요청 마다 쓰레드 생성 장단점

장점
- 동시 요청을 처리할 수 있다.
- 리소스(CPU, 메모리)가 허용할 때 까지 처리가능
- 하나의 쓰레드가 지연 되어도, 나머지 쓰레드는 정상 동작한다.

단점
- 쓰레드는 생성 비용은 매우 비싸다.
- 고객의 요청이 올 때 마다 쓰레드를 생성하면, 응답 속도가 늦어진다.
- 쓰레드는 컨텍스트 스위칭 비용이 발생한다.
> 코어가 1개안데 2쓰레드면 코어 하나가 왔다갔다 하면서 문맥 교환하는 비용이 발생한다.
- 쓰레드 생성에 제한이 없다. (고객 요청이 너무 많기 때문에)
- 고객 요청이 너무 많이 오면, CPU, 메모리 임계점을 넘어서 서버가 죽을 수 있다. (트래픽 과부하)

# 쓰레드 풀

![](https://velog.velcdn.com/images/jckim22/post/ed606dd1-5a39-44da-8c88-bb8232711df9/image.png)

쓰레드 풀에서 미리 쓰레드 만들고 쓰고 반납하는 식으로 한다.
그렇게 해서 쓰레드 풀에 더 이상 쓰레드를 할당할 수 없으면 대기, 거절 등을 통해 서버가 죽는 것을 방어할 수 있다.


#### 요청 마다 쓰레드 생성의 단점 보완

특징
- 필요한 쓰레드를 쓰레드 풀에 보관하고 관리한다.
- 쓰레드 풀에 생성 가능한 쓰레드의 최대치를 관리한다. 톰캣은 최대 200개 기본 설정 (변경 가능) 사용

사용
- 쓰레드가 필요하면, 이미 생성되어 있는 쓰레드를 쓰레드 풀에서 꺼내서 사용한다.
- 사용을 종료하면 쓰레드 풀에 해당 쓰레드를 반납한다.
- 최대 쓰레드가 모두 사용중이어서 쓰레드 풀에 쓰레드가 없으면?
>기다리는 요청은 거절하거나 특정 숫자만큼만 대기하도록 설정할 수 있다.

장점
- 쓰레드가 미리 생성되어 있으므로, 쓰레드를 생성하고 종료하는 비용(CPU)이 절약되고, 응답 시간이 빠르다.
- 생성 가능한 쓰레드의 최대치가 있으므로 너무 많은 요청이 들어와도 기존 요청은 안전하게 처리할 수 있다.

#### 실무 팁

- WAS의 주요 튜닝 포인트는 최대 쓰레드(max thread) 수이다.
- 이 값을 너무 낮게 설정하면?
동시 요청이 많으면, 서버 리소스는 여유롭지만, 클라이언트는 금방 응답 지연 
- 이 값을 너무 높게 설정하면?
동시 요청이 많으면, CPU, 메모리 리소스 임계점 초과로 서버 다운
- 장애 발생시?
클라우드면 일단 서버부터 늘리고, 이후에 튜닝
클라우드가 아니면 열심히 튜닝


#### 쓰레드 풀의 적정 숫자
- 적정 숫자는 어떻게 찾나요?
애플리케이션 로직의 복잡도, CPU, 메모리, IO 리소스 상황에 따라 모두 다름 - 성능 테스트
최대한 실제 서비스와 유사하게 성능 테스트 시도 
툴: 아파치 ab, 제이미터, nGrinder


# WAS의 멀티 쓰레드 지원 
#### 핵심

- 멀티 쓰레드에 대한 부분은 WAS가 처리
- 개발자가 멀티 쓰레드 관련 코드를 신경쓰지 않아도 됨
- 개발자는 마치 싱글 쓰레드 프로그래밍을 하듯이 편리하게 소스 코드를 개발
- 멀티 쓰레드 환경이므로 싱글톤 객체(서블릿, 스프링 빈)는 주의해서 사용

# 정적 리소스 
- 정적 파일 
![](https://velog.velcdn.com/images/jckim22/post/d2acdcc2-141d-49c7-8f7c-45c936a6d829/image.png)


# HTML 페이지 
- 동적 생성

![](https://velog.velcdn.com/images/jckim22/post/af0b29a0-6108-4e44-93f2-c83d8e8feb49/image.png)


# HTTP API

- 데이터를 전달
- 주로 JSON 형식 사용
- 다양한 시스템에서 호출

![](https://velog.velcdn.com/images/jckim22/post/0010df5f-ff58-46a8-95c2-47c2b17855ae/image.png)

웹 브라우저에서 호출하지 않고 클라이언트 별도 처리로 UI 화면을 구현한다.
또 API에서 API 서버로 호출이 가능하다.

![](https://velog.velcdn.com/images/jckim22/post/efa8708d-5822-447e-a25a-21a04b6d76e6/image.png)

- UI 클라이언트 접점
   - 앱 클라이언트(아이폰, 안드로이드, PC 앱)
   - 웹 브라우저에서 자바스크립트를 통한 HTTP API 호출
   - React, Vue.js 같은 웹 클라이언트
- 서버 to 서버
   - 주문 서버 -> 결제 서버
   - 기업간 데이터 통신
   
# 서버사이드 렌더링, 클라이언트 사이드 렌더링

![](https://velog.velcdn.com/images/jckim22/post/471dab17-db58-4ff4-b63c-2454bad00025/image.png)

   
# SSR - 서버 사이드 렌더링   

![](https://velog.velcdn.com/images/jckim22/post/09ad9a26-43bd-4170-aa83-74d8d8897293/image.png)


# CSR - 클라이언트 사이드 렌더링

![](https://velog.velcdn.com/images/jckim22/post/9bf942b0-4904-42e8-9861-55eb05257bde/image.png)

