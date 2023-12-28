
# 로그인 요구사항
- 홈 화면 - 로그인 전 회원 가입
   - 로그인
- 홈 화면 - 로그인 후
   - 본인 이름(누구님 환영합니다.) 상품 관리
   - 로그 아웃
- 보안 요구사항
   - 로그인 사용자만 상품에 접근하고, 관리할 수 있음
   - **로그인 하지 않은 사용자가 상품 관리에 접근하면 로그인 화면으로 이동**
- 회원 가입, 상품 관리

## 패키지 구조

![](https://velog.velcdn.com/images/jckim22/post/8f88d6ea-7382-401e-bbb6-44a6d95bcb5e/image.png)


**도메인이 가장 중요하다.**
도메인 = 화면, UI, 기술 인프라 등등의 영역은 제외한 시스템이 구현해야 하는 핵심 비즈니스 업무 영역을 말함

향후 web을 다른 기술로 바꾸어도 도메인은 그대로 유지할 수 있어야 한다.
이렇게 하려면 web은 domain을 알고있지만 domain은 web을 모르도록 설계해야 한다. 이것을 web은 domain을 의존하지만, domain은 web을 의존하지 않는다고 표현한다. 예를 들어 web 패키지를 모두 삭제해도 domain에는 전 혀 영향이 없도록 의존관계를 설계하는 것이 중요하다. 반대로 이야기하면 domain은 web을 참조하면 안된다.

   
# 회원 가입
먼저 Member class를 정의한다.
어노테이션 기반 검증을 위한 설정도 완료한다.

여기서는 회원정보 수정 기능을 만들지 않을 것이기 때문에 DTO를 만들지 않고 바로 객체에 데이터를 바인딩한다.

```java
@Data
public class Member {

    private Long id;

    @NotEmpty
    private String loginId; //로그인 ID
    @NotEmpty
    private String name; //사용자 이름
    @NotEmpty
    private String password;
}
```
MemberRepository를 만든다.
인터페이스를 정의 한뒤 거기에서 구현하면 좋겠지만, 메모리만 사용할 것이지 않기 때문에 바로 클래스를 구현한다.

특별한 점은 없다.
스트림 람다로 가독성을 높이고 Optional을 리턴하여 null처리를 한다.

```java
@Slf4j
@Repository
public class MemberRepository {

    private static Map<Long, Member> store = new HashMap<>(); //static 사용
    private static long sequence = 0L;//static 사용

    public Member save(Member member) {
        member.setId(++sequence);
        log.info("save: member={}", member);
        store.put(member.getId(), member);
        return member;
    }

    public Member findById(Long id) {
        return store.get(id);
    }

    public Optional<Member> findByLoginId(String loginId) {
        return findAll().stream()
                .filter(m -> m.getLoginId().equals(loginId))
                .findFirst();
    }

    public List<Member> findAll() {
        return new ArrayList<>(store.values());
    }

    public void clearStore() {
        store.clear();
    }
}

```
Lombok의 @RequiredArgsConsructor 어노테이션으로 생성자 주입을 받는다.
GetMapping을 했을 때도 ModelAttribute를 통해 데이터를 바인딩 한다.

Post 요청시 member에 데이터를 바인딩 하고 Repository를 이용하여 save한다.
```java
@Controller
@RequiredArgsConstructor
@RequestMapping("/members")
public class MemberController {

    private final MemberRepository memberRepository;

    @GetMapping("/add")
    public String addForm(@ModelAttribute("member") Member member) {
        return "members/addMemberForm";
    }

    @PostMapping("/add")
    public String save(@Valid @ModelAttribute Member member, BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            return "members/addMemberForm";
        }

        memberRepository.save(member);
        return "redirect:/";
    }
}

```

# Login 

Login은 Repository가 필요 없고 서비스만 필요하다.
그래서 Item이나 Member처럼 Class를 정의할 것이 아니라 WEB에서 바로 DTO를 만들어 데이터를 운반하면 된다.

먼저 서비스 로직을 만든다.
null을 리턴 받으면 로그인 실패로 할 것이기 때문에 로그인 실패면 null을 리턴한다.

```java
@Service
@RequiredArgsConstructor
public class LoginService {

    private final MemberRepository memberRepository;

    /**
     * @return null 로그인 실패
     */
    public Member login(String loginId, String password) {
        return memberRepository.findByLoginId(loginId)
                .filter(m -> m.getPassword().equals(password))
                .orElse(null);
    }
}

```

이제 DTO를 만들어 주고 어노테이션으로 필드 검증까지 한다.

```java
@Data
public class LoginForm {

    @NotEmpty
    private String loginId;

    @NotEmpty
    private String password;
}
```

그리고 컨트롤러를 구현한다.
post 요청으로 아이디와 비밀번호의 대한 정보가 왔을 때 필드 에러가 있다면 로그인 폼을 다시 보여주고
로그인 DTO로 전달받은 데이터로 주입받은 서비스 객체의 메서드로 로그인을 했을 때 null이 반환되면 필드의 문제는 아니고 객체 자체의 문제인 에러기 때문에 직접 코드 처리를 한다.


```java
    private final LoginService loginService;

    @GetMapping("/login")
    public String loginForm(@ModelAttribute("loginForm") LoginForm form) {
        return "login/loginForm";
    }

//    @PostMapping("/login")
    public String login(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletResponse response) {
        if (bindingResult.hasErrors()) {
            return "login/loginForm";
        }

        Member loginMember = loginService.login(form.getLoginId(), form.getPassword());

        if (loginMember == null) {
            bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
            return "login/loginForm";
        }


```

![](https://velog.velcdn.com/images/jckim22/post/4c292df0-410b-4d17-abad-bdc984517a9c/image.png)
![](https://velog.velcdn.com/images/jckim22/post/69589225-1ab3-4ca2-82e3-341867796ba2/image.png)
![](https://velog.velcdn.com/images/jckim22/post/a7f83c34-9156-4e5f-ba38-1cf44405b464/image.png)




**쿠키에는 영속 쿠키와 세션 쿠키가 있다.**
영속 쿠키: 만료 날짜를 입력하면 해당 날짜까지 유지
세션 쿠키: 만료 날짜를 생략하면 브라우저 종료시 까지만 유지
브라우저 종료시 로그아웃이 되길 기대하므로, 우리에게 필요한 것은 세션 쿠키이다.


이제 http 응답으로 쿠키를 남기자

```java
//    @PostMapping("/login")
    public String login(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletResponse response) {
        if (bindingResult.hasErrors()) {
            return "login/loginForm";
        }

        Member loginMember = loginService.login(form.getLoginId(), form.getPassword());

        if (loginMember == null) {
            bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
            return "login/loginForm";
        }

        //로그인 성공 처리

        //쿠키에 시간 정보를 주지 않으면 세션 쿠기(브라우저 종료시 모두 종료)
        Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));
        response.addCookie(idCookie);
        return "redirect:/";

    }
```

위처럼 쿠키를 생성하고 리스폰스 헤더에 추가한다.
>쿠키에게 시간 정보를 주지 않으면 디폴트는 세션 쿠키이다.

다음으로 쿠키를 사용해 홈 화면에서 로그인 상태를 유지하자.

```java
//    @GetMapping("/")
    public String homeLogin(@CookieValue(name = "memberId", required = false) Long memberId, Model model) {

        if (memberId == null) {
            return "home";
        }

        //로그인
        Member loginMember = memberRepository.findById(memberId);
        if (loginMember == null) {
            return "home";
        }

        model.addAttribute("member", loginMember);
        return "loginHome";
    }
```
위 코드를 보면 @CookieValue로 쿠키를 가져온다 required=false 설정으로 없어도 괜찮은 것으로 간주한다.
만약 로그인이 안되어있으면 home으로 리턴한다.
쿠키값은 있지만 데이터베이스에서도 찾을 수 없으면 home으로 간다.
그리고 로그인한 멤버의 정보를 model 담아 랜더링한다.

이후 템플릿에는 login을 한 홈화면을 또 따로 만들어서 관리한다.
쿠키 유무를 조건식으로 판단해서 하는 방법이 아니다.





```
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
          href="../css/bootstrap.min.css" rel="stylesheet">
</head>
<body>

<div class="container" style="max-width: 600px">
    <div class="py-5 text-center">
        <h2>홈 화면</h2>
    </div>

    <h4 class="mb-3" th:text="|로그인: ${member.name}|">로그인 사용자 이름</h4>

    <hr class="my-4">

    <div class="row">
        <div class="col">
            <button class="w-100 btn btn-secondary btn-lg" type="button"
                    th:onclick="|location.href='@{/items}'|">
                상품 관리
            </button>
        </div>
        <div class="col">
            <form th:action="@{/logout}" method="post">
                <button class="w-100 btn btn-dark btn-lg" onclick="location.href='items.html'" type="submit">
                    로그아웃
                </button>
            </form>
        </div>
    </div>

    <hr class="my-4">

</div> <!-- /container -->

</body>
</html>
```

그리고 로그아웃 기능도 만들어본다.
쿠키를 expire하는 로직을 수행하고 다시 홈으로 리다이렉트 한다.

```java
    public String logout(HttpServletResponse response) {
        expireCookie(response, "memberId");
        return "redirect:/";
    }
```


```java
    private void expireCookie(HttpServletResponse response, String cookieName) {
        Cookie cookie = new Cookie(cookieName, null);
        cookie.setMaxAge(0);
        response.addCookie(cookie);
    }
```


# 세션 동작 방식

앞서 쿠키에 중요한 정보를 보관하는 방법은 여러가지 보안 이슈가 있었다. 이 문제를 해결하려면 결국 중요한 정보를 모두 서버에 저장해야 한다. 그리고 클라이언트와 서버는 추정 불가능한 임의의 식별자 값으로 연결해야 한다.

이렇게 서버에 중요한 정보를 보관하고 연결을 유지하는 방법을 세션이라 한다.

### 세션 동작 방식
세션을 어떻게 개발할지 먼저 개념을 이해해보자.

![](https://velog.velcdn.com/images/jckim22/post/d1caa097-5a18-4f9b-b036-cfdbd0cf0e64/image.png)
![](https://velog.velcdn.com/images/jckim22/post/f5ba02ea-6f31-4a02-a519-2158b3a7be4d/image.png)
![](https://velog.velcdn.com/images/jckim22/post/8f52e895-fd94-426d-8256-a174c76534c7/image.png)
![](https://velog.velcdn.com/images/jckim22/post/ed7be56b-c4f9-4365-b848-2ed2980769eb/image.png)

1. 로그인 비밀번호를 확인하여 로그인에 성공
2. 서버는 그에 맞는 세션 아이디를 생성하고 저장
3. 클라이언트에게 세션 아이디를 쿠키로 세팅
4. 클라이언트는 쿠키에 저장된 세션 아이디로 인증을 받음


# 세션 직접 만들기

```java
@Component
public class SessionManager {

    public static final String SESSION_COOKIE_NAME = "mySessionId";
    private Map<String, Object> sessionStore = new ConcurrentHashMap<>();

    /**
     * 세션 생성
     */
    public void createSession(Object value, HttpServletResponse response) {

        //세션 id를 생성하고, 값을 세션에 저장
        String sessionId = UUID.randomUUID().toString();
        sessionStore.put(sessionId, value);

        //쿠키 생성
        Cookie mySessionCookie = new Cookie(SESSION_COOKIE_NAME, sessionId);
        response.addCookie(mySessionCookie);
    }

    /**
     * 세션 조회
     */
    public Object getSession(HttpServletRequest request) {
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
        if (sessionCookie == null) {
            return null;
        }
        return sessionStore.get(sessionCookie.getValue());
    }

    /**
     * 세션 만료
     */
    public void expire(HttpServletRequest request) {
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
        if (sessionCookie != null) {
            sessionStore.remove(sessionCookie.getValue());
        }
    }


    public Cookie findCookie(HttpServletRequest request, String cookieName) {
        if (request.getCookies() == null) {
            return null;
        }
        return Arrays.stream(request.getCookies())
                .filter(cookie -> cookie.getName().equals(cookieName))
                .findAny()
                .orElse(null);
    }

}

```

세션을 직접 만들면 위와 같은 sessionManager를 정의할 수 있다.
1. sessionManger를 자동 주입 받기 위해 스프링 빈으로 등록한다.
2. sessionStore를 만든다. (동시성 문제를 해결하기 위해 ConcurrentHashMap을 사용)
3. UUID로 고유의 세션 아이디를 생성하고 쿠키를 만들어 리스폰스 헤더에 담는다.
3. findCookie를 구현한다.
4. getSession, expire를 만든다.

```java
    void sessionTest() {

        //세션 생성
        MockHttpServletResponse response = new MockHttpServletResponse();
        Member member = new Member();
        sessionManager.createSession(member, response);

        //요청에 응답 쿠키 저장
        MockHttpServletRequest request = new MockHttpServletRequest();
        request.setCookies(response.getCookies());

        //세션 조회
        Object result = sessionManager.getSession(request);
        assertThat(result).isEqualTo(member);

        //세션 만료
        sessionManager.expire(request);
        Object expired = sessionManager.getSession(request);
        assertThat(expired).isNull();

    }
```

1. 세션을 생성한다. (MockHttpServletResponse사용)
2. 요청 쿠키에 세션아이디를 저장한다.
3. 세션을 조회한다. (member를 get)
4. 세션을 expire 한다.

# 적용

```java
    public String loginV2(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletResponse response) {
        if (bindingResult.hasErrors()) {
            return "login/loginForm";
        }

        Member loginMember = loginService.login(form.getLoginId(), form.getPassword());

        if (loginMember == null) {
            bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
            return "login/loginForm";
        }

        //로그인 성공 처리

        //세션 관리자를 통해 세션을 생성하고, 회원 데이터 보관
        sessionManager.createSession(loginMember, response);

        return "redirect:/";

    }
```

위처럼 로그인을 성공 했을 때 세션 매니저로 세션을 생성하고 스토어에 저장한다.(인증)

```java
    public String homeLoginV2(HttpServletRequest request, Model model) {

        //세션 관리자에 저장된 회원 정보 조회
        Member member = (Member)sessionManager.getSession(request);

        //로그인
        if (member == null) {
            return "home";
        }

        model.addAttribute("member", member);
        return "loginHome";
    }
```

위에서는 홈 화면에서 로그인이 된 화면인지 구별할 때 getSession으로 member를 받고 null인지 여부로 판단한다.

# 서블릿 HTTP 세션

세션이라는 개념은 대부분의 웹 애플리케이션에 필요한 것이다. 어쩌면 웹이 등장하면서 부터 나온 문제이다. 서블릿은 세션을 위해 `HttpSession` 이라는 기능을 제공하는데, 지금까지 나온 문제들을 해결해준다. 우리가 직접 구현한 세션의 개념이 이미 구현되어 있고, 더 잘 구현되어 있다.

먼저 로그인을 수정하자

```java
    public String loginV3(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletRequest request) {
        if (bindingResult.hasErrors()) {
            return "login/loginForm";
        }

        Member loginMember = loginService.login(form.getLoginId(), form.getPassword());

        if (loginMember == null) {
            bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
            return "login/loginForm";
        }

        //로그인 성공 처리
        //세션이 있으면 있는 세션 반환, 없으면 신규 세션을 생성
        HttpSession session = request.getSession();
        //세션에 로그인 회원 정보 보관
        session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);

        return "redirect:/";

    }

```
위 코드를 보면 로그인이 성공한다면 getSession(true)로 반환한다.
true일 때는 세션이 브라우저에 존재하지 않을 때 새로 세션을 생성한다.
만약 존재하면 그대로 반환한다.
그리고 정해진 상수와 멤버 객체를 session에 추가한다.


다음은 로그아웃이다.
```java
    @PostMapping("/logout")
    public String logoutV3(HttpServletRequest request) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            session.invalidate();
        }
        return "redirect:/";
    }
```
request헤더에서 세션을 false로 얻는다.
없으면 그대로 null을 주겠지만 있다면(null이 아니면) session을 invalidate함으로써 처리한다.


마지막으로 홈 화면에서 세션 인가다.

```java
    public String homeLoginV3(HttpServletRequest request, Model model) {

        HttpSession session = request.getSession(false);
        if (session == null) {
            return "home";
        }

        Member loginMember = (Member)session.getAttribute(SessionConst.LOGIN_MEMBER);

        //세션에 회원 데이터가 없으면 home
        if (loginMember == null) {
            return "home";
        }

        //세션이 유지되면 로그인으로 이동
        model.addAttribute("member", loginMember);
        return "loginHome";
    }
```
세션을 가져오고 로그인 안한 사용자가 세션을 생성할 일 없도록 (false)로 설정한다.
상수로 세션을 가져온다.
세션을 가져왔는데 member가 null이라면 서버 스토리지에 맞는 사용자가 없는 것이므로 홈으로 가고 세션이 인가되어서 유지되면 로그인홈으로 이동하게 된다.
