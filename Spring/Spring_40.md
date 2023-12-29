# API 예외 처리 - 시작
#### API 예외 처리는 어떻게 해야할까?
HTML 페이지의 경우 지금까지 설명했던 것 처럼 4xx, 5xx와 같은 오류 페이지만 있으면 대부분의 문제를 해결할 수 있다.
그런데 API의 경우에는 생각할 내용이 더 많다. 오류 페이지는 단순히 고객에게 오류 화면을 보여주고 끝이지만, API는 각 오류 상황에 맞는 오류 응답 스펙을 정하고, JSON으로 데이터를 내려주어야 한다.


이걸 확인하기 위해 스프링 부트에 기본 탑재된 error url로 가는 기능을 쓰지 않고 아래 WebServerFactoryCustomizer의 구현체를 다시 컴포넌트로 등록한다.
```java
@Component
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

그럼 위를 따라서 에러가 나면 해당 url로 다시 요청이 간다.

우린 지금 아래처럼 컨트롤러를 설정해놓는다.


```java
@Slf4j
@RestController
public class ApiExceptionController {

    @GetMapping("/api/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id) {

        if (id.equals("ex")) {
            throw new RuntimeException("잘못된 사용자");
        }
        return new MemberDto(id, "hello " + id);
    }



    @Data
    @AllArgsConstructor
    static class MemberDto {
        private String memberId;
        private String name;
    }

}

```

그럼 members/ex로 접속했을 때 런타임 에러를 던지게 될 것이다.
런타임 에러를 던지면 에러 설정에 따라 에러 페이지 처리 컨트롤러 갈 것이다.

아래처럼 정상호출 때는 API가 그대로 오게 되지만 전에 작성했던 컨트롤러에 따라서 예외가 발생하면 오류 페이지가 리턴된다.

**정상 호출** `http://localhost:8080/api/members/spring`

```json
 {
     "memberId": "spring",
     "name": "hello spring"
}
```



**예외 발생 호출** `http://localhost:8080/api/members/ex`
```html
 <!DOCTYPE HTML>
 <html>
 <head>
 </head>
<body> 
  ...
</body>
```

API를 요청했는데, 정상의 경우 API로 JSON 형식으로 데이터가 정상 반환된다. 그런데 오류가 발생하면 우리가 미리 만들어둔 오류 페이지 HTML이 반환된다. 이것은 기대하는 바가 아니다. 클라이언트는 정상 요청이든, 오류 요청이든 JSON이 반환되기를 기대한다. 웹 브라우저가 아닌 이상 HTML을 직접 받아서 할 수 있는 것은 별로 없다.

그래서 우리는 오류페이지 컨트롤러를 수정한다.


```java
    @RequestMapping("/error-page/500")
    public String errorPage500(HttpServletRequest request, HttpServletResponse response) {
        log.info("errorPage 500");
        printErrorInfo(request);
        return "error-page/500";
    }

    @RequestMapping(value = "/error-page/500", produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<Map<String, Object>> errorPage500Api(
            HttpServletRequest request, HttpServletResponse response) {

        log.info("API errorPage 500");

        Map<String, Object> result = new HashMap<>();
        Exception ex = (Exception) request.getAttribute(ERROR_EXCEPTION);
        result.put("status", request.getAttribute(ERROR_STATUS_CODE));
        result.put("message", ex.getMessage());

        Integer statusCode = (Integer) request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
        return new ResponseEntity<>(result, HttpStatus.valueOf(statusCode));
    }
```

위처럼 수정하면 MediaType에서 클라이언트의 Accept/JSON을 확인하고 맞다면 우선적으로 아래 컨트롤러를 실행한다.
그럼 우리는 API 스펙에 맞춰 에러를 담아 json으로 보내주면 된다.


# 스프링 부트 기본 오류 처리

스프링 부트 기본 오류처리를 사용할 때는 어떻게 되는지 확인하기 위해 다시 수동으로 설정한 컴포넌트를 주석처리하여 빈에서 제외한다.

```java
//@Component
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

`http://localhost:8080/api/members/ex`
```json
{
"timestamp": "2021-04-28T00:00:00.000+00:00", "status": 500,
"error": "Internal Server Error",
"exception": "java.lang.RuntimeException",
"trace": "java.lang.RuntimeException: 잘못된 사용자\n\tat
   hello.exception.web.api.ApiExceptionController.getMember(ApiExceptionController.
 java:19...,
"message": "잘못된 사용자",
     "path": "/api/members/ex"
 }
```

스프링  부트에서도 자동으로 Accept/json을 확인하고 오류 페이지를 보내지 않고 json 형식으로 에러를 보내준다.



- `errorHtml()` : `produces = MediaType.TEXT_HTML_VALUE` : 클라이언트 요청의 Accept 해더 값이 `text/html` 인 경우에는 `errorHtml()` 을 호출해서 ModelAndView를 제공한다.
- `error()` : 그외 경우에 호출되고 `ResponseEntity` 로 HTTP Body에 JSON 데이터를 반환한다.


스프링 부트는 `BasicErrorController` 가 제공하는 기본 정보들을 활용해서 오류 API를 생성해준다.

다음 옵션들을 설정하면 더 자세한 오류 정보를 추가할 수 있다. 
- server.error.include-binding-errors=always 
- server.error.include-exception=true
- server.error.include-message=always 
- server.error.include-stacktrace=always


물론 오류 메시지는 이렇게 막 추가하면 보안상 위험할 수 있다. 간결한 메시지만 노출하고, 로그를 통해서 확인하자.


# Html 페이지 vs API 오류
`BasicErrorController` 를 확장하면 JSON 메시지도 변경할 수 있다. 그런데 API 오류는 `@ExceptionHandler` 가 제공하는 기능을 사용하는 것이 더 나은 방법이므로 지금은 `BasicErrorController` 를 확장해서 JSON 오류 메시지를 변경할 수 있다 정도로만 이해해두자.

스프링 부트가 제공하는 `BasicErrorController` 는 HTML 페이지를 제공하는 경우에는 매우 편리하다. 4xx, 5xx 등등 모두 잘 처리해준다. 그런데 API 오류 처리는 다른 차원의 이야기이다. API 마다, 각각의 컨트롤러나 예외마 다 서로 다른 응답 결과를 출력해야 할 수도 있다. 예를 들어서 회원과 관련된 API에서 예외가 발생할 때 응답과, 상품과 관련된 API에서 발생하는 예외에 따라 그 결과가 달라질 수 있다. **결과적으로 5xx, 4xx 오류 페이지를 던지는 것보다 매우 세밀하고 복잡하다.** 따라서 이 방법 은 HTML 화면을 처리할 때 사용하고, API 오류 처리는 뒤에서 설명할 `@ExceptionHandler` 를 사용하자.

# API 예외 처리 - HandlerExceptionResolver 시작


**목표**
예외가 발생해서 서블릿을 넘어 WAS까지 예외가 전달되면 HTTP 상태코드가 500으로 처리된다. 발생하는 예외에 따 라서 400, 404 등등 다른 상태코드로 처리하고 싶다.
오류 메시지, 형식등을 API마다 다르게 처리하고 싶다.

**상태코드 변환**
예를 들어서 `IllegalArgumentException` 을 처리하지 못해서 컨트롤러 밖으로 넘어가는 일이 발생하면 HTTP 상태코드를 400으로 처리하고 싶다. 어떻게 해야할까?

```java
        if (id.equals("bad")) {
            throw new IllegalArgumentException("잘못된 입력 값");
        }
```

컨트롤러에 bad url로 들어오면 사용자가 잘못한 것으로 인식하게 하기 위해 IlleagalArgumentException을 던진다.

지금은 아직 예외가 던져지면 WAS에서 서버 문제라고 생각하고 500을 던진다.


**HandlerExceptionResolver**
스프링 MVC는 컨트롤러(핸들러) 밖으로 예외가 던져진 경우 예외를 해결하고, 동작을 새로 정의할 수 있는 방법을 제 공한다. 컨트롤러 밖으로 던져진 예외를 해결하고, 동작 방식을 변경하고 싶으면 `HandlerExceptionResolver` 를 사용하면 된다. 줄여서 `ExceptionResolver` 라 한다.


![](https://velog.velcdn.com/images/jckim22/post/d79abe71-df1c-4940-8643-39d53cdff5a7/image.png)
![](https://velog.velcdn.com/images/jckim22/post/2fb5543c-851d-421b-9dc0-d9859eaca500/image.png)


ExceptionResolver가 WAS로 가고 있는 예외를 잡아 채서 처리한다.
>`ExceptionResolver` 로 예외를 해결해도 `postHandle()` 은 호출되지 않는다

```java
@Slf4j
public class MyHandlerExceptionResolver implements HandlerExceptionResolver {

    @Override
    public ModelAndView resolveException(HttpServletRequest request, 
    HttpServletResponse response, Object handler, Exception ex) {

        log.info("call resolver", ex);

        try {
            if (ex instanceof IllegalArgumentException) {
                log.info("IllegalArgumentException resolver to 400");
                response.sendError(HttpServletResponse.SC_BAD_REQUEST, ex.getMessage());
                return new ModelAndView();
            }

        } catch (IOException e) {
            log.error("resolver ex", e);
        }

        return null;
    }
}

```

`handler` : 핸들러(컨트롤러) 정보
`Exception ex` : 핸들러(컨트롤러)에서 발생한 발생한 예외

위처럼 ExceptionResolver를 직접 구현해보자.

코드를 보면 발생한 예외가 IlliegalArgumentException 일시 예외를 try로 정상척으로 처리하되 sendError로 바꿔치기(?)한다.
sendError는 예외를 터지게 하지는 않지만 WAS로 갔을 때 에러가 있었다고 인식하게 한다.



- `ExceptionResolver` 가 `ModelAndView` 를 반환하는 이유는 마치 try, catch를 하듯이, `Exception` 을 처
   
 리해서 정상 흐름 처럼 변경하는 것이 목적이다. 이름 그대로 `Exception` 을 Resolver(해결)하는 것이 목적이다.
 

 여기서는 `IllegalArgumentException` 이 발생하면 `response.sendError(400)` 를 호출해서 HTTP 상태 코드를 400으로 지정하고, 빈 `ModelAndView` 를 반환한다.
 
 ![](https://velog.velcdn.com/images/jckim22/post/c1c76c11-32fd-4c31-8a45-022147381862/image.png)
![](https://velog.velcdn.com/images/jckim22/post/5ebdae27-4a50-4961-be3b-ef53656309f6/image.png)

```java
    @Override
    public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        resolvers.add(new MyHandlerExceptionResolver());
    }
```

이제 위처럼 우리가 만든 ExceptionResolver를 webConfiguration에서 등록한다.

>`configureHandlerExceptionResolvers(..)` 를 사용하면 스프링이 기본으로 등록하는 `ExceptionResolver` 가 제거되므로 주의, `extendHandlerExceptionResolvers` 를 사용하자.

# API 예외 처리 - HandlerExceptionResolver 활용

**예외를 여기서 마무리하기**
예외가 발생하면 WAS까지 예외가 던져지고, WAS에서 오류 페이지 정보를 찾아서 다시 `/error` 를 호출하는 과정은 생각해보면 너무 복잡하다. `ExceptionResolver` 를 활용하면 예외가 발생했을 때 이런 복잡한 과정 없이 여기에서 문제를 깔끔하게 해결할 수 있다.


```java
@Slf4j
public class    UserHandlerExceptionResolver implements HandlerExceptionResolver {

    private final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {

        try {

            if (ex instanceof UserException) {
                log.info("UserException resolver to 400");
                String acceptHeader = request.getHeader("accept");
                response.setStatus(HttpServletResponse.SC_BAD_REQUEST);

                if ("application/json".equals(acceptHeader)) {
                    Map<String, Object> errorResult = new HashMap<>();
                    errorResult.put("ex", ex.getClass());
                    errorResult.put("message", ex.getMessage());
                    String result = objectMapper.writeValueAsString(errorResult);

                    response.setContentType("application/json");
                    response.setCharacterEncoding("utf-8");
                    response.getWriter().write(result);
                    return new ModelAndView();
                } else {
                    // TEXT/HTML
                    return new ModelAndView("error/500");
                }
            }

        } catch (IOException e) {
            log.error("resolver ex", e);
        }

        return null;
    }
}
```

위에서 UserHandlerExceptionResolver를 만들었다.
UserHandlerExceptionResolver의 내가 정의한 구현체이다.

이전 핸들러에서 UserException이 터졌었고 클라이언트 요청이 json이었으면 에러를 json에 담아 처리한다.
만약 아니라면 error 페이지를 리턴한다.

아래처럼 설정하면 적용이 된다.

```java
@Slf4j
@RestController
public class ApiExceptionController {

    @GetMapping("/api/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id) {

        if (id.equals("ex")) {
            throw new RuntimeException("잘못된 사용자");
        }
        if (id.equals("bad")) {
            throw new IllegalArgumentException("잘못된 입력 값");
        }
        if (id.equals("user-ex")) {
            throw new UserException("사용자 오류");
        }

        return new MemberDto(id, "hello " + id);
    }
```


```java
    @Override
    public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        resolvers.add(new MyHandlerExceptionResolver());
        resolvers.add(new UserHandlerExceptionResolver());
    }
```


**정리**

`ExceptionResolver` 를 사용하면 컨트롤러에서 예외가 발생해도 `ExceptionResolver` 에서 예외를 처리해버린
다.
따라서 예외가 발생해도 서블릿 컨테이너까지 예외가 전달되지 않고, 스프링 MVC에서 예외 처리는 끝이 난다. 결과적으로 WAS 입장에서는 정상 처리가 된 것이다. 이렇게 예외를 이곳에서 모두 처리할 수 있다는 것이 핵심이다.
서블릿 컨테이너까지 예외가 올라가면 복잡하고 지저분하게 추가 프로세스가 실행된다. 반면에 `ExceptionResolver` 를 사용하면 예외처리가 상당히 깔끔해진다.
그런데 직접 `ExceptionResolver` 를 구현하려고 하니 상당히 복잡하다. 지금부터 스프링이 제공하는 `ExceptionResolver` 들을 알아보자.



# API 예외 처리 - 스프링이 제공하는 ExceptionResolver1
스프링 부트가 기본으로 제공하는 `ExceptionResolver` 는 다음과 같다. `HandlerExceptionResolverComposite` 에 다음 순서로 등록
1. `ExceptionHandlerExceptionResolver`
2. `ResponseStatusExceptionResolver`
3. `DefaultHandlerExceptionResolver` 우선순위가가장낮다.

- **ExceptionHandlerExceptionResolver**
`@ExceptionHandler` 을 처리한다. API 예외 처리는 대부분 이 기능으로 해결한다. 조금 뒤에 자세히 설명한다.

- **ResponseStatusExceptionResolver** HTTP 상태 코드를 지정해준다.
예) `@ResponseStatus(value = HttpStatus.NOT_FOUND)`

- **DefaultHandlerExceptionResolver**
스프링 내부 기본 예외를 처리한다.
먼저 가장 쉬운 `ResponseStatusExceptionResolver` 부터 알아보자.


## ResponseStatusExceptionResolver


```java
@ResponseStatus(code = HttpStatus.BAD_REQUEST, reason = "error.bad")
public class BadRequestException extends RuntimeException {
}
```

위 어노테이션 설정만으로도 HTTP 코드를 수정해준다
reason의 텍스트는 바로 텍스트를 넣어도 되지만 messageSource로 읽어오기도 한다.

**messages.properties**
```
error.bad=잘못된 요청 오류입니다. 메시지 사용 
```

스프링 부트에서 제공하는 ResponseStatusExceptionResolver가 어노테이션을 인식하고 그 값들을 통해 우리가 위해서 순수 작업했던 내용들을 진행하면서 HTTP	상태 코드를 sendError로 바꿔치기 한다.


**ResponseStatusException**
`@ResponseStatus` 는 개발자가 직접 변경할 수 없는 예외에는 적용할 수 없다. (애노테이션을 직접 넣어야 하는데,
내가 코드를 수정할 수 없는 라이브러리의 예외 코드 같은 곳에는 적용할 수 없다.)
추가로 애노테이션을 사용하기 때문에 조건에 따라 동적으로 변경하는 것도 어렵다. 이때는
`ResponseStatusException` 예외를 사용하면 된다.


**ApiExceptionController - 추가**

```java
 @GetMapping("/api/response-status-ex2")
 public String responseStatusEx2() {
     throw new ResponseStatusException(HttpStatus.NOT_FOUND, "error.bad", new
 IllegalArgumentException());
}
```

# API 예외 처리 - 스프링이 제공하는 ExceptionResolver2


이번에는 `DefaultHandlerExceptionResolver` 를 살펴보자.

## DefaultHandlerExceptionResolver

```java
    @GetMapping("/api/default-handler-ex")
    public String defaultException(@RequestParam Integer data) {
        return "ok";
    }
```


위처럼 Integer를 갖고오는 컨트롤러에 String으로 넣게 되면 TypeMisMatchException이 터지고 WAS가 500을 던질 것이다.

하지만 스프링에서는 DefaultHandlerExceptionResolver를 제공하기 때문에 이러한 내부적인 예외들을 그에 맞는 상태 코드로 처리해준다.



# ExceptionHandlerExceptionResolver

정리**
지금까지 다음 `ExceptionResolver` 들에 대해 알아보았다.

1. `ExceptionHandlerExceptionResolver`
2. `ResponseStatusExceptionResolver` HTTP응답코드변경 
3. `DefaultHandlerExceptionResolver`스프링내부예외처리

  지금까지 HTTP 상태 코드를 변경하고, 스프링 내부 예외의 상태코드를 변경하는 기능도 알아보았다. 그런데 `HandlerExceptionResolver` 를 직접 사용하기는 복잡하다. API 오류 응답의 경우 `response` 에 직접 데이터를
   넣어야 해서 매우 불편하고 번거롭다. `ModelAndView` 를 반환해야 하는 것도 API에는 잘 맞지 않는다.
스프링은 이 문제를 해결하기 위해 `@ExceptionHandler` 라는 매우 혁신적인 예외 처리 기능을 제공한다. 그것이 아직 소개하지 않은 `ExceptionHandlerExceptionResolver` 이다.

# API 예외 처리 - @ExceptionHandler

**API 예외처리의 어려운 점**
- `HandlerExceptionResolver` 를 떠올려 보면 `ModelAndView` 를 반환해야 했다. 이것은 API 응답에는 필
요하지 않다.
- API 응답을 위해서 `HttpServletResponse` 에 직접 응답 데이터를 넣어주었다. 이것은 매우 불편하다. 스프 링 컨트롤러에 비유하면 마치 과거 서블릿을 사용하던 시절로 돌아간 것 같다.
- 특정 컨트롤러에서만 발생하는 예외를 별도로 처리하기 어렵다. 예를 들어서 회원을 처리하는 컨트롤러에서 발생 하는 `RuntimeException` 예외와 상품을 관리하는 컨트롤러에서 발생하는 동일한 `RuntimeException` 예 외를 서로 다른 방식으로 처리하고 싶다면 어떻게 해야할까?


**@ExceptionHandler**
스프링은 API 예외 처리 문제를 해결하기 위해 `@ExceptionHandler` 라는 애노테이션을 사용하는 매우 편리한 예외 처리 기능을 제공하는데, 이것이 바로 `ExceptionHandlerExceptionResolver` 이다. 스프링은
`ExceptionHandlerExceptionResolver` 를 기본으로 제공하고, 기본으로 제공하는 `ExceptionResolver` 중에 우선순위도 가장 높다. 실무에서 API 예외 처리는 대부분 이 기능을 사용한다.


```java
@Slf4j
@RestController
public class ApiExceptionV2Controller {

    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler
    public ErrorResult illegalExHandler(IllegalArgumentException e){
        log.error("[exceptionHandler] ex", e);
        return new ErrorResult("BAD", e.getMessage());
    }
    
    @ExceptionHandler
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResult exHandler(Exception e){
        log.error("[exceptionHandler] ex", e);
        return new ErrorResult("EX", "내부 오류");
    }

```

스프링은 컨트롤러에서 예외가 터지면 디스패처 서블릿으로 왔다가 ExceptionResolver를 호출한다.
ExceptionResolver는 에러가 터진 핸들러(컨트롤러)에 ExceptionHandelr가 있는지 확인하고 그것에 대한 내용을 처리한다.
이로써 원하는 컨트롤러에 원하는 예외 처리를 자유롭고 더 유연하게 할 수 있게 되었다.

위 코드를 보면 2가지 ExceptionHandler가 있는데 중복 될 때는 범위가 더 작은 IllegalArgumentException이 터지게 된다.

컨트롤러가 @RestController로 등록이 되어있기 때문에 객체를 반환하면 json형태로 리턴되게 된다.

json이 아니더라도 string, HttpEntity, view 등 일반 컨트롤러와 같게 동작이 가능해진다.


# ControllerAdvice

그런데 컨트롤러마다 각기 다르게 처리할 수 있다는 장점이 있지만, 같은 처리를 하고 싶은 컨트롤러가 있다면 중복이 되지 않을까?

그리고 정상코드와 처리 코드가 한 곳에 있는건 마음에 들지 않는다.

그래서 @ControllerAdvice가 존재한다.

```java
@Slf4j
@RestControllerAdvice(basePackages = "hello.exception.api")
public class ExControllerAdvice {

    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExHandler(IllegalArgumentException e) {
        log.error("[exceptionHandler] ex", e);
        return new ErrorResult("BAD", e.getMessage());
    }

    @ExceptionHandler
    public ResponseEntity<ErrorResult> userExHandler(UserException e) {
        log.error("[exceptionHandler] ex", e);
        ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
        return new ResponseEntity(errorResult, HttpStatus.BAD_REQUEST);
    }

    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler
    public ErrorResult exHandler(Exception e) {
        log.error("[exceptionHandler] ex", e);
        return new ErrorResult("EX", "내부 오류");
    }

}
```

이렇게 @RestControllerAdvice로 처리 로직들을 빼놓으면 이러한 공통 관심사인 예외 처리를 모든 컨트롤러에서 사용한다.

```java
// Target all Controllers annotated with @RestController
 @ControllerAdvice(annotations = RestController.class)
 public class ExampleAdvice1 {}
 // Target all Controllers within specific packages
 @ControllerAdvice("org.example.controllers")
 public class ExampleAdvice2 {}
 // Target all Controllers assignable to specific classes
 @ControllerAdvice(assignableTypes = {ControllerInterface.class,
 AbstractController.class})
public class ExampleAdvice3 {}
```

위와 같은 어노테이션, 패키지, 클래스를 지정하는 방법으로 대상 컨트롤러를 지정할 수 있다.
