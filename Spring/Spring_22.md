
**웹 스코프의 특징**
- 웹 스코프는 웹 환경에서만 동작한다.
- 웹 스코프는 프로토타입과 다르게 스프링이 해당 스코프의 종료시점까지 관리한다. 따라서 종료 메서드가 호출된 다.

**웹 스코프 종류**
- **request:** HTTP 요청 하나가 들어오고 나갈 때 까지 유지되는 스코프, 각각의 HTTP 요청마다 별도의 빈 인스턴 스가 생성되고, 관리된다.
- **session:** HTTP Session과 동일한 생명주기를 가지는 스코프
- **application:** 서블릿 컨텍스트( `ServletContext` )와 동일한 생명주기를 가지는 스코프
- **websocket:** 웹 소켓과 동일한 생명주기를 가지는 스코프
사실 세션이나, 서블릿 컨텍스트, 웹 소켓 같은 용어를 잘 모르는 분들도 있을 것이다. 여기서는 request 스코프를 예제 로 설명하겠다. 나머지도 범위만 다르지 동작 방식은 비슷하다.

![](https://velog.velcdn.com/images/jckim22/post/5fa260e7-c32e-4be8-a993-81697aa255ab/image.png)

위처럼 요청에 따라 빈을 생성하고 그 요청이 지속되는 한은 그 빈을 재활용하고 끝나면 destroy하는 스코프를 가진 것이 웹 스코프이다.


# 예제

```java
@Component
@Scope(value = "request")
public class MyLogger {

    private String uuid;
    private String requestURL;

    public void setRequestURL(String requestURL) {
        this.requestURL = requestURL;
    }

    public void log(String message) {
        System.out.println("[" + uuid + "]" + "[" + requestURL + "]" + message);
    }

    @PostConstruct
    public void init(){
        String string = UUID.randomUUID().toString();
        System.out.println("[" + uuid + "] request scope bean create:" + this);
    }

    @PreDestroy
    public void close(){
        System.out.println("[" + uuid + "] request scope bean close:" + this);
    }

}

```


```java
@Controller
@RequiredArgsConstructor
public class LogDemoController {
    private final LogDemoService logDemoService;
    private final MyLogger myLogger;

    @RequestMapping("log-demo")
    @ResponseBody
    public String logDemo(HttpServletRequest request){
        String requestURL = request.getRequestURL().toString();
        myLogger.setRequestURL(requestURL);

        myLogger.log("controller test");
        logDemoService.logic("testId");
        return "OK";
    }
}

```



```java
@Service
@RequiredArgsConstructor
public class LogDemoService {
    private final MyLogger myLogger;

    public void logic(String id){
        myLogger.log("service id = " + id);
    }

}
```

>- 비즈니스 로직이 있는 서비스 계층에서도 로그를 출력해보자.
- 여기서 중요한점이 있다. request scope를 사용하지 않고 파라미터로 이 모든 정보를 서비스 계층에 넘긴다면, 파라미터가 많아서 지저분해진다. 더 문제는 requestURL 같은 웹과 관련된 정보가 웹과 관련없는 서비스 계층까 지 넘어가게 된다. 웹과 관련된 부분은 컨트롤러까지만 사용해야 한다. 서비스 계층은 웹 기술에 종속되지 않고, 가급적 순수하게 유지하는 것이 유지보수 관점에서 좋다.
- request scope의 MyLogger 덕분에 이런 부분을 파라미터로 넘기지 않고, MyLogger의 멤버변수에 저장해서 코드와 계층을 깔끔하게 유지할 수 있다.

위 세 클래스를 살펴보자

MyLogger request scope를 가진 컴포넌트이다.
Spring-Web에서 Controller를 찾아가고 컨트롤러에서는 서비스와 로거를 주입받는다. (LOMBOK이 생성자 주입을 해결해줌)

컨트롤러라는 싱글톤 빈이 생성되었지만 아직 요청이 없었기에 requset scope 빈은 아직 생성되지 않았다.
하지만 위 코드에서는 컨트롤러 생성 시 바로 주입을 요청하기 때문에 스프링 애플리케이션을 실행 시키면 오류가 발생한다.

## provider로 해결

```java
@Controller
@RequiredArgsConstructor
public class LogDemoController {
    private final LogDemoService logDemoService;
    private final ObjectProvider<MyLogger> myLoggerProvider;

    @RequestMapping("log-demo")
    @ResponseBody
    public String logDemo(HttpServletRequest request) {
        String requestURL = request.getRequestURL().toString();
        MyLogger myLogger = myLoggerProvider.getObject();
        myLogger.setRequestURL(requestURL);
        myLogger.log("controller test");
        logDemoService.logic("testId");
        return "OK";
    }
}

```

위처럼 코드를 수정해보자
Provider를 사용함으로써 컨트롤러 생서시 DI가 아닌 DL이 진행된다.
객체가 아닌 객체를 계속 생성해줄 수 있는 Provider를 받고
log-demo로 요청이 들어왔을 때 getObject로 이 때 오브젝트를 생성한다.
request 스코프이기 때문에 만족하게 되고 생성된다.
그리고 세팅을 한다.

이전에 배웠듯이 객체의 생성과 초기화를 분리하였기에 나올 수 있는 결과이다.

이제 매 요청마다 다른 Logger가 생성이 될 거고 동시에 여러 요청이 와도 각 요청이 끝나기 전까지는 각각의 Logger가 destroy되지 않는다.

4개의 Logger가 생성된 뒤 각각의 클라이언트가 원하는 서비스를 진행할 때마다 그에 맞는 UUID로 로그가 찍히고 클라이언트가 종료할 때 destroy 된다.


# 스코프와 프록시

이를 더 간단하게 할 수는 없을까?


```java
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
```

위처럼 설정하고 원래 코드로 바꾸면 똑같이 작동한다.

![](https://velog.velcdn.com/images/jckim22/post/44d28f81-7705-4967-9b59-1017c6a75253/image.png)

가짜 프록시 객체로 컴파일 오류를 막고 진짜 필요할 때 진짜 빈을 요청하면서 지연 처리를 하게 된다.


**가짜 프록시 객체는 요청이 오면 그때 내부에서 진짜 빈을 요청하는 위임 로직이 들어있다.**
- 가짜 프록시 객체는 내부에 진짜 `myLogger` 를 찾는 방법을 알고 있다.
- 클라이언트가 `myLogger.log()` 을 호출하면 사실은 가짜 프록시 객체의 메서드를 호출한 것이다.
- 가짜 프록시 객체는 request 스코프의 진짜 `myLogger.log()` 를 호출한다.
- 가짜 프록시 객체는 원본 클래스를 상속 받아서 만들어졌기 때문에 이 객체를 사용하는 클라이언트 입장에서는 사 실 원본인지 아닌지도 모르게, 동일하게 사용할 수 있다(다형성)

**동작 정리**
- CGLIB라는 라이브러리로 내 클래스를 상속 받은 가짜 프록시 객체를 만들어서 주입한다.
- 이 가짜 프록시 객체는 실제 요청이 오면 그때 내부에서 실제 빈을 요청하는 위임 로직이 들어있다.
- 가짜 프록시 객체는 실제 request scope와는 관계가 없다. 그냥 가짜이고, 내부에 단순한 위임 로직만 있고, 싱 글톤 처럼 동작한다.

**특징 정리**
- 프록시 객체 덕분에 클라이언트는 마치 싱글톤 빈을 사용하듯이 편리하게 request scope를 사용할 수 있다. 사실 Provider를 사용하든, 프록시를 사용하든 핵심 아이디어는 진짜 객체 조회를 꼭 필요한 시점까지 지연처리 한다는 점이다.
     
- 단지 애노테이션 설정 변경만으로 원본 객체를 프록시 객체로 대체할 수 있다. 이것이 바로 다형성과 DI 컨테이너 가 가진 큰 강점이다.
꼭 웹 스코프가 아니어도 프록시는 사용할 수 있다.

**주의점**
- 마치 싱글톤을 사용하는 것 같지만 다르게 동작하기 때문에 결국 주의해서 사용해야 한다.
- 이런 특별한 scope는 꼭 필요한 곳에만 최소화해서 사용하자, 무분별하게 사용하면 유지보수하기 어려워진다.
