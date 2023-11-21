앞서 말했던 DI에 연장선이다.
# 컴포넌트

![](https://velog.velcdn.com/images/jckim22/post/721cc806-dcd3-4ee1-ba8c-3facfc838a50/image.png)

컴포넌트(Component)란 프로그래밍에 있어 재사용이 가능한 각각의 독립된 모듈을 뜻한다.

그림에서 확인 할 수 있듯이 컴포넌트 기반 프로그래밍을 하면 마치 레고 블록처럼 이미 만들어진 컴포넌들을 조합하여 화면을 구성할 수 있다.

웹 컴포넌트는 이러한 컴포넌트 기반 프로그래밍을 웹에서도 적용할 수 있도록 W3C에서 새로 정한 규격이다. 웹 표준을 기반으로 구축되었으며, 최신 부라우저 및 모든 JavaScript 라이브러리, 프레임워크에서도 사용할 수 있다.

따라서 웹 컴포넌트를 이용하여 코드를 작성하면 Vue.js 나 React.js 와 같은 라이브러리, 프레임워크에 의존하지 않고 상호 운용이 가능하게끔 작성할 수 있다.

# 컴포넌트 스캔
스프링 컨테이너에서 동일하거나 하위 프로젝트에 있는 어노테이션으로 등록되어 있는 컴포넌트를 스캔해서 스프링 빈으로 등록한다.

# Dependency Injection(의존 관계)
의존 관계란 앞 게시글에 봤듯이 의존되는 객체를 외부에서 주입해주는 것을 뜻한다.

# 예제
```java
@Controller
public class MemberController {
    private final MemberService memberServic;

    @Autowired //MemberService를 가져옴(객체로) (Dependency Injection)
    public MemberController(MemberService memberService) {
        this.memberServic = memberService;
    }
}
```


```java
@Service
public class MemberService {

    private final MemberRepository memberRepository;

    @Autowired
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

```

```java
@Repository
public class MemoryMemberRepository implements MemberRepository {

    private static Map<Long, Member> store = new HashMap<>();
    private static Long sequence = 0L;
```

생성자에 @Autowired 가 있으면 스프링이 연관된 객체를 스프링 컨테이너에서 찾아서 넣어준다. 이렇게 객체 의존관계를 외부에서 넣어주는 것을 DI (Dependency Injection), 의존성 주입이라 한다.
이전 테스트에서는 개발자가 직접 주입했고, 여기서는 @Autowired에 의해 스프링이 주입해준다.

### 컴포넌트 스캔 원리
@Component 애노테이션이 있으면 스프링 빈으로 자동 등록된다.
@Controller 컨트롤러가 스프링 빈으로 자동 등록된 이유도 컴포넌트 스캔 때문이다.
@Component 를 포함하는 다음 애노테이션도 스프링 빈으로 자동 등록된다. - 
 - @Controller
 - @Service
 - @Repository
 
![](https://velog.velcdn.com/images/jckim22/post/2984c21c-9970-4a97-aac4-9679129908f6/image.png)

memberService 와 memberRepository 가 스프링 컨테이너에 스프링 빈으로 등록되었다.
> 스프링은 스프링 컨테이너에 스프링 빈을 등록할 때, 기본으로 싱글톤으로 등록한다(유일하게 하나만 등록해서 공유한다) 따라서 같은 스프링 빈이면 모두 같은 인스턴스다. 설정으로 싱글톤이 아니게 설정할 수 있지만, 특별한 경우를 제외하면 대부분 싱글톤을 사용한다.

>결론적으로 같은 객체를 재사용하고 하나의 객체로만 사용하기 위해 (싱글톤) Denpendency Injection을 사용한다.
