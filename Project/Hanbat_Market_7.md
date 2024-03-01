
# 회원가입
#### MemberController

```java
@Controller
@RequiredArgsConstructor
@Slf4j
public class MemberController {

    private final MemberService memberService;

    @GetMapping("/members/new")
    public String createForm(Model model
            , @SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member sessionMember) {
        if (sessionMember != null) {
            return "redirect:/";
        }
        model.addAttribute("memberForm", new MemberForm()); // = @ModelAttribute
        return "members/createMemberForm";
    }

    @PostMapping("/members/new")
    public String create(@Valid @ModelAttribute MemberForm form, BindingResult result
            , @SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member sessionMember) {
        if (sessionMember != null) {
            return "redirect:/";
        }

        if (result.hasErrors()) {
            return "members/createMemberForm";
        }
        Member member = Member.createMember(form.getMail(), form.getPasswd(), form.getMail(), form.getNickname());
        memberService.join(member);
        return "members/completeRegister";
    }
}

```

멤버 컨트롤러에서는 GET요청이 들어오면 준비해놨던 폼을 랜더링 해준다.
폼에서 post 요청을 보내면 MemberForm DTO에 그 내용을 받는다.
그리고 그 DTO로 회원가입을 진행한다.

근데 여기서 @Valid와 BindingResult를 사용해서 예외처리를 진행한다.

#### MemberForm

```java
@Getter @Setter
public class MemberForm{
    @NotEmpty(message = "메일을 입력해주세요.")
    String mail;

    @NotEmpty(message = "비밀번호를 입력해주세요.")
    String passwd;

    @NotEmpty(message = "전화번호를 입력해주세요.")
    String phoneNumber;

    @NotEmpty(message = "닉네임을 입력해주세요.")
    String nickname;
}

```
Dto에 @NotEmpty와 같은 검증 어노테이션을 붙여주고 해당 에러가 생기면 message를 담도록한다.
그래서 멤버 컨트롤러에서 null이 있거나 잘못된 값이 입력이 됐을 경우 에러를 포함해 result를 모델에 담아 보내게 된다.

그 에러로 템플릿에서는 사용자에게 잘못된 값이라는 것을 알려주도록 한다.

# 로그인
#### LoginService
```java
    public Member login(String mail, String passwd) {
        return memberRepository.findByMail(mail)
                .filter(m -> m.getPasswd().equals(passwd))
                .orElse(null);
    }
```

이 애플리케이션에서는 아이디를 메일로 사용하기 때문에 서비스에서 mail로 그에 해당하는 멤버를 찾는다.
그리고 그 멤버가 갖고 있는 패스워드와 받은 패스워드가 일치하는지 확인한다.

#### LoginController

```java
@Controller
@RequiredArgsConstructor
@Slf4j
public class LoginController {

    private final MemberService memberService;

    @GetMapping("/login")
    public String loginForm(@ModelAttribute("loginForm") LoginForm form
            , @SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member sessionMember) {
        if (sessionMember != null) {
            return "redirect:/";
        }
        return "login/loginForm";
    }

    @PostMapping("/login")
    public String login(@Valid @ModelAttribute LoginForm form, BindingResult result
            , HttpServletRequest request, @SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member sessionMember
            , @RequestParam(value = "redirectURL", defaultValue = "/") String redirectURL) {
        if (sessionMember != null) {
            return "redirect:/";
        }
        if (result.hasErrors()) {
            return "login/loginForm";
        }

        Member loginMember = memberService.login(form.getMail(), form.getPasswd());

        if (loginMember == null) {
            result.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
            return "login/loginForm";
        }

        //로그인 성공 처리
        //세션이 있으면 있는 세션 반환, 없으면 신규 세션을 생성
        HttpSession session = request.getSession();

        //세션에 로그인 회원 정보 보관
        session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);

        return "redirect:" + redirectURL;
    }

    @PostMapping("/logout")
    public String logout(HttpServletRequest request) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            session.invalidate();
        }
        return "redirect:/";
    }
}

```
loginForm에서 POST요청이 들어오면 에러 확인하고 로그인을 진행한다. 만약 로그인에 실패하면 공통적인 에러 메시지를 담아 보낸다.

로그인에 성공하면 세션에 로그인 회원 정보를 보관한다.
그리고 다른 컨트롤러에서 세션이 필요하다면 @SessionAttribute로 확인한다.

#### LoginForm
```java
@Getter
@Setter
public class LoginForm {
    @NotEmpty(message = "메일을 입력해주세요.")
    String mail;
    @NotEmpty(message = "비밀번호를 입력해주세요.")
    String passwd;
}

```

LoginFormDTO 역시 @Valid 어노테이션으로 검증한다.



