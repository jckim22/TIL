# HTTP 요청 데이터

**주로 다음 3가지 방법을 사용한다.** 

**GET - 쿼리 파라미터**
- /url**?username=hello&age=20**
- 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달 
- 예) 검색, 필터, 페이징등에서 많이 사용하는 방식

**POST - HTML Form**
- content-type: application/x-www-form-urlencoded 
- 메시지 바디에 쿼리 파리미터 형식으로 전달 username=hello&age=20
- 예) 회원 가입, 상품 주문, HTML Form 사용 

**HTTP message body에 데이터를 직접 담아서 요청**
- HTTP API에서 주로 사용, JSON, XML, TEXT
- 데이터 형식은 주로 JSON 사용 
- POST, PUT, PATCH


# GET - 쿼리 파라미터

메시지 바디 없이, URL의 **쿼리 파라미터**를 사용해서 데이터를 전달하자. 
>예) 검색, 필터, 페이징등에서 많이 사용하는 방식

쿼리파라미터는 URL에 다음과같이 `?` 를 시작으로 보낼 수 있다.추가 파라미터는  `&`  로 구분하면 된다. 
```
http://localhost:8080/request-param?username=hello&age=20
```
서버에서는 `HttpServletRequest` 가 제공하는 다음 메서드를 통해 쿼리 파라미터를 편리하게 조회할 수 있다.

쿼리 파라미터 조회 메서드

```java
String username = request.getParameter("username"); //단일 파라미터 조회
Enumeration<String> parameterNames = request.getParameterNames(); //파라미터 이름들
모두 조회
Map<String, String[]> parameterMap = request.getParameterMap(); //파라미터를 Map으로
조회
String[] usernames = request.getParameterValues("username"); //복수 파라미터 조회
```

예시

```java
/**
* 1. 파라미터 전송 기능
* http://localhost:8080/request-param?username=hello&age=20
* <p>
* 2. 동일한 파라미터 전송 가능
* http://localhost:8080/request-param?username=hello&username=kim&age=20 */
 @WebServlet(name = "requestParamServlet", urlPatterns = "/request-param")
 public class RequestParamServlet extends HttpServlet {
@Override
     protected void service(HttpServletRequest request, HttpServletResponse
 response) throws ServletException, IOException {
   
 System.out.println("[전체 파라미터 조회] - start");
         /*
         Enumeration<String> parameterNames = request.getParameterNames();
         while (parameterNames.hasMoreElements()) {
             String paramName = parameterNames.nextElement();
             System.out.println(paramName + "=" +
 request.getParameter(paramName));
} */
         request.getParameterNames().asIterator()
                 .forEachRemaining(paramName -> System.out.println(paramName +
"=" + request.getParameter(paramName))); System.out.println("[전체 파라미터 조회] - end"); System.out.println();
System.out.println("[단일 파라미터 조회]");
String username = request.getParameter("username"); System.out.println("request.getParameter(username) = " + username);
         String age = request.getParameter("age");
         System.out.println("request.getParameter(age) = " + age);
         System.out.println();
System.out.println("[이름이 같은 복수 파라미터 조회]"); System.out.println("request.getParameterValues(username)"); String[] usernames = request.getParameterValues("username"); for (String name : usernames) {
             System.out.println("username=" + name);
         }
         response.getWriter().write("ok");
     }
}

```

결과

직접 url에 접속해서 실행해보면 결과는 다음과 같다.
```
http://localhost:8080/request-param?username=hello&age=20
```

![](https://velog.velcdn.com/images/jckim22/post/c0b09a6a-11b5-48f8-8536-fa5e4471a3ff/image.png)

# GET, POST - HTML Form


**요청 URL**: http://localhost:8080/request-param **content-type**: `application/x-www-form-urlencoded` 
**message body**: `username=hello&age=20`
       
 `application/x-www-form-urlencoded` 형식은 앞서 GET에서 살펴본 쿼리 파라미터 형식과 같다. 따라서 **쿼 리 파라미터 조회 메서드를 그대로 사용**하면 된다.
클라이언트(웹 브라우저) 입장에서는 두 방식에 차이가 있지만, 서버 입장에서는 둘의 형식이 동일하므로, `request.getParameter()` 로 편리하게 구분없이 조회할 수 있다.


>content-type은 HTTP 메시지 바디의 데이터 형식을 지정한다.
**GET URL 쿼리 파라미터 형식**으로 클라이언트에서 서버로 데이터를 전달할 때는 HTTP 메시지 바디를 사용하 지 않기 때문에 content-type이 없다.
**POST HTML Form 형식**으로 데이터를 전달하면 HTTP 메시지 바디에 해당 데이터를 포함해서 보내기 때문에 바디에 포함된 데이터가 어떤 형식인지 content-type을 꼭 지정해야 한다. 이렇게 폼으로 데이터를 전송하는 형 식을 `application/x-www-form-urlencoded` 라 한다.


###	Postman을 사용한 테스트
이런 간단한 테스트에 HTML form을 만들기는 귀찮다. 이때는 Postman을 사용하면 된다.
**Postman** 테스트 주의사항 POST 전송시
Body `x-www-form-urlencoded` 선택
Headers에서 content-type: `application/x-www-form-urlencoded` 로 지정된 부분 꼭 확인

### 결과

html form 방식으로 post요청을 보내게 되면 위에 쿼리 파라미터와 똑같은 방식으로 response body에 담아 데이터를 보내기 때문에 위 자바코드를 재활용해도 된다

![](https://velog.velcdn.com/images/jckim22/post/f8c8ee14-b128-45f5-86d7-5d87503b7085/image.png)

위처럼 request-param에 html form으로 post요청을 보내게 되면 아래와 같은 결과가 나온다.
![](https://velog.velcdn.com/images/jckim22/post/5088b692-b189-4883-be7b-59c87ff77af3/image.png)

# HTTP 요청 데이터 - API 방식


- **HTTP message body**에 데이터를 직접 담아서 요청
   - HTTP API에서 주로 사용, JSON, XML, TEXT
   - 데이터 형식은 주로 JSON 사용
   - POST, PUT, PATCH
   - 웹 브라우저에서 사용하는 것이 아님
   
   
### 단순 텍스트
      
- 먼저 가장 단순한 텍스트 메시지를 HTTP 메시지 바디에 담아서 전송하고, 읽어보자.
- HTTP 메시지 바디의 데이터를 InputStream을 사용해서 직접 읽을 수 있다.

postman으로 request 바디에 텍스트를 써서 보낸다.

```java
@WebServlet(name = "requestBodyStringServlet", urlPatterns = "/request-body-string")
public class RequestBodyStringServlet extends HttpServlet {
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //메시지 바디의 내용을 바이트 코드로 바로 얻을 수 있음
        ServletInputStream inputStream = request.getInputStream();
        //String으로 변환
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        System.out.println("messageBody = " + messageBody);

        response.getWriter().write("ok");;
    }

}
```
위처럼 InputStream으로 얻은 바이트 코드를 String으로 변환해서 Body를 출력하며 아래와 같은 결과가 나온다.

![](https://velog.velcdn.com/images/jckim22/post/4d8d48a0-02d9-4536-98d9-1e919e541230/image.png)


### json 형식으로

**JSON 형식 전송**
POST http://localhost:8080/request-body-json
content-type: **application/json**
message body: `{"username": "hello", "age": 20}`
결과: `messageBody = {"username": "hello", "age": 20}`


```java
@Getter @Setter
public class HelloData {
    private String username;
    private int age;
}
```
위처럼 데이터를 설정한다.
getter, setter는 lombok을 이용한다.

```java
@WebServlet(name = "requestBodyJsonServlet", urlPatterns = "/request-body-json")
public class RequestBodyJsonServlet extends HttpServlet {

    private ObjectMapper objectMapper = new ObjectMapper();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        System.out.println("messageBody = " + messageBody);

        //objectMapper로 json 데이터를 꺼내봄
        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);

        System.out.println("helloData name = " + helloData.getUsername());
        System.out.println("helloData age = " + helloData.getAge());

        response.getWriter().write("ok");
    }
}

```

위 코드처럼 String으로 받은 json을 Jackson 라이브러리의 ObjectMapper를 사용해서 파싱한다.

그렇게 해서 json데이터의 키로 객체와 매핑하여 데이터를 담아 사용한다.

![](https://velog.velcdn.com/images/jckim22/post/dc96fbd4-2bfe-41eb-8c10-13ed7e3ae4ef/image.png)
![](https://velog.velcdn.com/images/jckim22/post/5bc93886-16c1-4be4-b3f7-3a8bf1c1e0d3/image.png)
