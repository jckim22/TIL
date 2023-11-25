# Home 화면

```java
    @GetMapping("/")
    public String home() {
        return "home";
    }
```

home.html이라는 템플릿 파일을 보여준다.
가입과 목록 조회 버튼이 있다.

# MemberForm

```java
public class MemberForm {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

템플릿 파일에 form에서 name이라는 value에 값을 넣고 post요청하면 spring에서 setName으로 매개변수에 memberForm.name을 세팅해준다.

# MemberController

```java
    @GetMapping("/members/new")
    public String createForm() {
        return "members/createMemberForm";
    }

    @PostMapping("/members/new")
    public String create(MemberForm form) {
        Member member = new Member();
        member.setName(form.getName());

        memberService.join(member);

        return "redirect:/";
    }

    @GetMapping("/members")
    public String list(Model model) {
        List<Member> members = memberService.findMembers();
        model.addAttribute("members", members);
        return "members/memberList";
    }
```

from에 세팅된 name으로 memberService를 통해 member를 등록하고 홈으로 리다이렉션한다.

템플릿 파일에 each로 members를 조회한다.
