# 로깅 간단히 알아보기

앞으로 로그를 사용할 것이기 때문에, 이번시간에는 로그에 대해서 간단히 알아보자.
운영 시스템에서는 `System.out.println()` 같은 시스템 콘솔을 사용해서 필요한 정보를 출력하지 않고, 별도의 로 깅 라이브러리를 사용해서 로그를 출력한다.
참고로 로그 관련 라이브러리도 많고, 깊게 들어가면 끝이 없기 때문에, 여기서는 최소한의 사용 방법만 알아본다.

**로깅 라이브러리**
스프링 부트 라이브러리를 사용하면 스프링 부트 로깅 라이브러리( `spring-boot-starter-logging` )가 함께 포 함된다.
   
스프링 부트 로깅 라이브러리는 기본으로 다음 로깅 라이브러리를 사용한다.


- SLF4J - http://www.slf4j.org
- Logback - http://logback.qos.ch

로그 라이브러리는 Logback, Log4J, Log4J2 등등 수 많은 라이브러리가 있는데, 그것을 통합해서 인터페이스로 제 공하는 것이 바로 SLF4J 라이브러리다.
쉽게 이야기해서 SLF4J는 인터페이스이고, 그 구현체로 Logback 같은 로그 라이브러리를 선택하면 된다. 실무에서는 스프링 부트가 기본으로 제공하는 Logback을 대부분 사용한다.

**로그 선언**
`private Logger log = LoggerFactory.getLogger(getClass());`
`private static final Logger log = LoggerFactory.getLogger(Xxx.class)` `@Slf4j` : 롬복 사용 가능
**로그 호출** `log.info("hello")`
`System.out.println("hello")`

```java
@RestController
@Slf4j
public class LogTestController {
//    private final Logger log = LoggerFactory.getLogger(getClass());

    @RequestMapping("/log-test")
    public String logTest() {
        String name = "Spring";

        System.out.println("name = " + name);
        //log.info(" info log={}"+ name); 안됨(쓸모없는 리소스를 사용)

        log.trace("trace log={}", name);
        log.debug("debug log={}", name);
        log.info(" info log={}", name);
        log.warn(" warn log={}", name);
        log.error("error log={}", name);

        return "ok";
    }
}

```

**매핑 정보** 
`@RestController`
- `@Controller` 는 반환 값이 `String` 이면 뷰 이름으로 인식된다. 그래서 **뷰를 찾고 뷰가 랜더링** 된다.
- `@RestController` 는 반환 값으로 뷰를 찾는 것이 아니라, **HTTP 메시지 바디에 바로 입력**한다. 따라서 실행 결과로 ok 메세지를 받을 수 있다. `@ResponseBody` 와 관련이 있는데, 뒤에서 더 자세히 설명한다.

**테스트**
- 로그가 출력되는 포멧 확인
   - 시간, 로그 레벨, 프로세스 ID, 쓰레드 명, 클래스명, 로그 메시지
- 로그 레벨 설정을 변경해서 출력 결과를 보자.
   - LEVEL: `TRACE > DEBUG > INFO > WARN > ERROR` 
   - 개발 서버는 debug 출력
   - 운영 서버는 info 출력
- `@Slf4j` 로 변경


**로그 레벨 설정** 
`application.properties`
```
#전체 로그 레벨 설정(기본 info) logging.level.root=info
         
 #hello.springmvc 패키지와 그 하위 로그 레벨 설정 logging.level.hello.springmvc=debug
```
**올바른 로그 사용법** 
- `log.debug("data="+data)`
로그 출력 레벨을 info로 설정해도 해당 코드에 있는 "data="+data가 실제 실행이 되어 버린다. 결과적으
로 문자 더하기 연산이 발생한다. 
- `log.debug("data={}", data)`
로그 출력 레벨을 info로 설정하면 아무일도 발생하지 않는다. 따라서 앞과 같은 의미없는 연산이 발생하지 않는다.


**로그 사용시 장점**
- 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있고, 출력 모양을 조정할 수 있다.
- 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력하고, 운영서버에서는 출력하지 않는 등 로그를 상황에 맞게 조절할 수 있다.
- 시스템 아웃 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등, 로그를 별도의 위치에 남길 수 있다. 특히 파 일로 남길 때는 일별, 특정 용량에 따라 로그를 분할하는 것도 가능하다.
- 성능도 일반 System.out보다 좋다. (내부 버퍼링, 멀티 쓰레드 등등) 그래서 실무에서는 꼭 로그를 사용해야 한 다.
