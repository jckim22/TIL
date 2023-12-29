# Exception(예외)
**자바 직접 실행**
자바의 메인 메서드를 직접 실행하는 경우 `main` 이라는 이름의 쓰레드가 실행된다.
실행 도중에 예외를 잡지 못하고 처음 실행한 `main()` 메서드를 넘어서 예외가 던져지면, 예외 정보를 남기고 해당 쓰 레드는 종료된다.

**웹 애플리케이션**
웹 애플리케이션은 사용자 요청별로 별도의 쓰레드가 할당되고, 서블릿 컨테이너 안에서 실행된다.
애플리케이션에서 예외가 발생했는데, 어디선가 try ~ catch로 예외를 잡아서 처리하면 아무런 문제가 없다. 그런데 만 약에 애플리케이션에서 예외를 잡지 못하고, 서블릿 밖으로 까지 예외가 전달되면 어떻게 동작할까?


WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생) 

## response.sendError(HTTP 상태 코드, 오류 메시지)

**예외 발생 흐름**

```
WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생) 
```

**sendError 흐름** 
```
WAS(sendError 호출 기록 확인) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러
 (response.sendError())
```
sendError는 바로 예외가 터지는게 아니라 WAS에 도달했을 때 Error가 났었다는걸 알릴 수 있다.
http 상태코드와 오류 메시지를 보낼 수 있다.

```java
@Slf4j
@Controller
public class ServletExController {

    @GetMapping("/error-ex")
    public void errorEx() {
        throw new RuntimeException("예외 발생!");
    }

    @GetMapping("/error-404")
    public void error404(HttpServletResponse response) throws IOException {
        response.sendError(404, "404 오류!");
    }

    @GetMapping("/error-400")
    public void error400(HttpServletResponse response) throws IOException {
        response.sendError(400, "400 오류!");
    }

    @GetMapping("/error-500")
    public void error500(HttpServletResponse response) throws IOException {
        response.sendError(500);
    }
}
```

위처럼 특정 페이지로 요청이 들어왔는데 RuntimeException이나 sendError로 에러가 났다면 

```java
public class WebServerCustomizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {
    @Override
    public void customize(ConfigurableWebServerFactory factory) {

        ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error-page/404");
        ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");

        ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/error-page/500");

        factory.addErrorPages(errorPage404, errorPage500, errorPageEx);
    }
}
```

WebServerFactoryCustomizer<ConfigurableWebServerFactory\> 다음을 구현한 클래스에 의해 해당 예러에 맞는 에러 처리 페이지로 요청이 보내진다.

여기서 중요한 점은 이때 단순히 리다이렉트가 아니라 오류 페이지 경로로 필터, 서블릿, 인터셉터, 컨트롤러가 모두 다시 호출된다.
그렇다는 말은 컨트롤러가 따로 필요하다는 것이다.

```java
@Slf4j
@Controller
public class ErrorPageController {

    //RequestDispatcher 상수로 정의되어 있음
    public static final String ERROR_EXCEPTION = "javax.servlet.error.exception";
    public static final String ERROR_EXCEPTION_TYPE = "javax.servlet.error.exception_type";
    public static final String ERROR_MESSAGE = "javax.servlet.error.message";
    public static final String ERROR_REQUEST_URI = "javax.servlet.error.request_uri";
    public static final String ERROR_SERVLET_NAME = "javax.servlet.error.servlet_name";
    public static final String ERROR_STATUS_CODE = "javax.servlet.error.status_code";

    @RequestMapping("/error-page/404")
    public String errorPage404(HttpServletRequest request, HttpServletResponse response) {
        log.info("errorPage 404");
        printErrorInfo(request);
        return "error-page/404";
    }

    @RequestMapping("/error-page/500")
    public String errorPage500(HttpServletRequest request, HttpServletResponse response) {
        log.info("errorPage 500");
        printErrorInfo(request);
        return "error-page/500";
    }


    private void printErrorInfo(HttpServletRequest request) {
        log.info("ERROR_EXCEPTION: {}", request.getAttribute(ERROR_EXCEPTION));
        log.info("ERROR_EXCEPTION_TYPE: {}", request.getAttribute(ERROR_EXCEPTION_TYPE));
        log.info("ERROR_MESSAGE: {}", request.getAttribute(ERROR_MESSAGE));
        log.info("ERROR_REQUEST_URI: {}", request.getAttribute(ERROR_REQUEST_URI));
        log.info("ERROR_SERVLET_NAME: {}", request.getAttribute(ERROR_SERVLET_NAME));
        log.info("ERROR_STATUS_CODE: {}", request.getAttribute(ERROR_STATUS_CODE));
        log.info("dispatchType={}", request.getDispatcherType());
    }
}

```

위처럼 특정 URL로 매핑되어 요청이 서버 내부적으로 다시 들어오게 되고 오류 페이지를 리턴한다.
근데 이때 그냥 요청하는 것이 아니라 서버 내부적으로 에러에 대한 내용도 담겨져서 리턴이 된다.
>DispactherType에 대해 집중해보자



오류가 발생하면 오류 페이지를 출력하기 위해 WAS 내부에서 다시 한번 호출이 발생한다. 이때 필터, 서블릿, 인터셉터 도 모두 다시 호출된다. 그런데 로그인 인증 체크 같은 경우를 생각해보면, 이미 한번 필터나, 인터셉터에서 로그인 체크 를 완료했다. 따라서 서버 내부에서 오류 페이지를 호출한다고 해서 해당 필터나 인터셉트가 한번 더 호출되는 것은 매 우 비효율적이다.
결국 클라이언트로 부터 발생한 정상 요청인지, 아니면 오류 페이지를 출력하기 위한 내부 요청인지 구분할 수 있어야 한다. 서블릿은 이런 문제를 해결하기 위해 `DispatcherType` 이라는 추가 정보를 제공한다.

# DispatcherType
필터는 이런 경우를 위해서 `dispatcherTypes` 라는 옵션을 제공한다. 
고객이 처음 요청하면 `dispatcherType=REQUEST` 이다.
이렇듯 서블릿 스펙은 실제 고객이 요청한 것인지, 서버가 내부에서 오류 페이지를 요청하는 것인지
`DispatcherType` 으로 구분할 수 있는 방법을 제공한다.

```java
 public enum DispatcherType {
     FORWARD,
     INCLUDE,
     REQUEST,
     ASYNC,
     ERROR
}
```

**DispatcherType**
- `REQUEST` : 클라이언트 요청
- `ERROR` : 오류 요청
-`FORWARD` : MVC에서 배웠던 서블릿에서 다른 서블릿이나 JSP를 호출할 때 `RequestDispatcher.forward(request, response);`
- `INCLUDE` : 서블릿에서 다른 서블릿이나 JSP의 결과를 포함할 때 `RequestDispatcher.include(request, response);`
- `ASYNC` : 서블릿 비동기 호출


```java
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(new LogFilter());
        filterRegistrationBean.setOrder(1);
        filterRegistrationBean.addUrlPatterns("/*");
        filterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.ERROR);
        return filterRegistrationBean;
    }
```

필터를 DispacterType이 REQUEST일 때만 호출하게 하고 싶으면 위처럼 setDispatcherTypes를 설정하면 되지만 필터는 디폴트 값이 REQUEST 이다.

# 서블릿 예외 처리 - 인터셉터

#### 인터셉터 중복 호출 제거

앞서 필터의 경우에는 필터를 등록할 때 어떤 `DispatcherType` 인 경우에 필터를 적용할 지 선택할 수 있었다. 그런 데 인터셉터는 서블릿이 제공하는 기능이 아니라 스프링이 제공하는 기능이다. 따라서 `DispatcherType` 과 무관하게 항상 호출된다.

대신에 인터셉터는 다음과 같이 요청 경로에 따라서 추가하거나 제외하기 쉽게 되어 있기 때문에, 이러한 설정을 사용해 서 오류 페이지 경로를 `excludePathPatterns` 를 사용해서 빼주면 된다.

```java
.excludePathPatterns("/css/**", "*.ico", "/error", "/error-page/**");//오류 페이지 경로
```

`/error-ex` 오류 요청
필터는 `DispatchType` 으로 중복 호출 제거 ( `dispatchType=REQUEST` )
인터셉터는 경로 정보로 중복 호출 제거( `excludePathPatterns("/error-page/**")` )

# 스프링 부트 - 오류 페이지

![](https://velog.velcdn.com/images/jckim22/post/df7849ff-58ce-408d-9668-f68d26218c7f/image.png)


![](https://velog.velcdn.com/images/jckim22/post/01e82c65-22d5-4fcb-abfc-894582835977/image.png)


해당 경로 위치에 HTTP 상태 코드 이름의 뷰 파일을 넣어두면 된다.
뷰 템플릿이 정적 리소스보다 우선순위가 높고, 404, 500처럼 구체적인 것이 5xx처럼 덜 구체적인 것 보다 우선순위가 높다.
5xx, 4xx 라고 하면 500대, 400대 오류를 처리해준다.

# 스프링 부트 - 오류 페이지2 

#### BasicErrorController가 제공하는 기본 정보들

`BasicErrorController` 컨트롤러는 다음 정보를 model에 담아서 뷰에 전달한다. 뷰 템플릿은 이 값을 활용해서 출력할 수 있다.


```java
<div class="container" style="max-width: 600px">
     <div class="py-5 text-center">
<h2>500 오류 화면 스프링 부트 제공</h2> </div>
<div>
<p>오류 화면 입니다.</p>
</div> <ul>
<li>오류 정보</li> <ul>
             <li th:text="|timestamp: ${timestamp}|"></li>
             <li th:text="|path: ${path}|"></li>
             <li th:text="|status: ${status}|"></li>
             <li th:text="|message: ${message}|"></li>
             <li th:text="|error: ${error}|"></li>
             <li th:text="|exception: ${exception}|"></li>
             <li th:text="|errors: ${errors}|"></li>
             <li th:text="|trace: ${trace}|"></li>
</ul>
</li> </ul>
     <hr class="my-4">
 </div> <!-- /container -->
 </body>
 </html>

```


오류 관련 내부 정보들을 고객에게 노출하는 것은 좋지 않다. 고객이 해당 정보를 읽어도 혼란만 더해지고, 보안상 문제 가 될 수도 있다.
그래서 `BasicErrorController` 오류 컨트롤러에서 다음 오류 정보를 `model` 에 포함할지 여부 선택할 수 있다.


application.properties`
- `server.error.include-exception=false` : `exception` 포함 여부( `true` , `false` ) 
- `server.error.include-message=never` : `message` 포함 여부
-   `server.error.include-stacktrace=never` : `trace` 포함 여부 
- `server.error.include-binding-errors=never` : `errors ` 포함 여부

> 실무에서는 이것들을 노출하면 안된다! 사용자에게는 이쁜 오류 화면과 고객이 이해할 수 있는 간단한 오류 메시지를 보
여주고 오류는 서버에 로그로 남겨서 로그로 확인해야 한다.
