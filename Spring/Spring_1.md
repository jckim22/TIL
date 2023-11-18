# 프로젝트 생성
https://start.spring.io/
## 프로젝트 선택
* Project: Gradle Project -> 요즘에는 Gradle, Gradle-Groovy Project
* group -> 보통 기업명, 단체
* artifactld -> project-name
* Dependencies -> Spring Web, Thymeleaf(template)

#### Tip 
> 최근 IntelliJ 버전은 Gradle을 통해서 실행하는 것이 기본 설정이다.
자바로 바로 실행하는 설정을 해준다.

# 라이브러리
## 의존 관계
Spring-stater를 import하면 의존 관계에 의해서 계속 땡겨오게 되고 Spring-core까지 땡겨오게 되는 것이다.

## 스프링 부트 라이브러리
Gradle은 의존관계가 있는 라이브러리를 함께 다운로드 한다.
spring-boot-starter-web spring-boot-starter-tomcat: 톰캣 (웹서버) spring-webmvc: 스프링 웹 MVC
spring-boot-starter-thymeleaf: 타임리프 템플릿 엔진(View) spring-boot-starter(공통): 스프링 부트 + 스프링 코어 + 로깅
spring-boot spring-core
spring-boot-starter-logging logback, slf4j(로그에 대한인터페이스) -> logback(slf4j의 구현체, 로그)

## 테스트 라이브러리
spring-boot-starter-test
junit: 테스트 프레임워크(중요)
mockito: 목 라이브러리
assertj: 테스트 코드를 좀 더 편하게 작성하게 도와주는 라이브러리spring-test: 스프링 통합 테스트 지원

위처럼 라이브러리들이 서로 의존 관계에 있고 Gradle로 인하여 한번에 끌고 와질 수 있다는 것을 알고 있으면 된다.

# View 실행

![](https://velog.velcdn.com/images/jckim22/post/c89eff7f-42ba-498d-80ce-cb2116fd059f/image.png)

브라우저가 hello라는 URL로 접속하면 

```java
@Controller
public class HelloController {

    @GetMapping("hello")
    public String Hello(Model model) {
        model.addAttribute("data", "hello");
        return "hello";
    }
}
```
위처럼 컨트롤러에서 hello를 매핑해서 메서드를 찾는다.
controller의 메서드는 Model로 부터 데이터를 받고 model에 data라는 키 값에 hello라는 value를 추가해서 hello.html 템플릿 파일(View)에 넘겨준다.

>- 컨트롤러에서 리턴 값으로 문자를 반환하면 뷰 리졸버( viewResolver )가 화면을 찾아서 처리한다.
  - 스프링 부트 템플릿엔진 기본 viewName 매핑
   - resources:templates/ +{ViewName}+ .html

# build 및 실행

지금까지는 intellij에서 실행한 것이고 이제 배포를 위한 .jar 파일을 만들어서 실행을 해본다.

```
gradle build
```
위 명령어를 통해 build가 되고 jar파일이 생성된다.

배포할 때 해당 서버에서 이 jar파일 하나만 실행해주면 된다.
