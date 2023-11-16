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


## 스프링 부트 라이브러리
Gradle은 의존관계가 있는 라이브러리를 함께 다운로드 한다.
spring-boot-starter-web spring-boot-starter-tomcat: 톰캣 (웹서버) spring-webmvc: 스프링 웹 MVC
spring-boot-starter-thymeleaf: 타임리프 템플릿 엔진(View) spring-boot-starter(공통): 스프링 부트 + 스프링 코어 + 로깅
spring-boot spring-core
spring-boot-starter-logging logback, slf4j(인터페이스)
## 테스트 라이브러리
spring-boot-starter-test
junit: 테스트 프레임워크(중요)
mockito: 목 라이브러리
assertj: 테스트 코드를 좀 더 편하게 작성하게 도와주는 라이브러리 spring-test: 스프링 통합 테스트 지원

위처럼 라이브러리들이 서로 의존 관계에 있고 Gradle로 인하여 한번에 끌고 와질 수 있다는 것을 알고 있으면 된다.
