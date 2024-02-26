### MemberService

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class MemberService {

    private final MemberRepository jpaMemberRepository;

    /**
     * 회원가입
     */
    @Transactional
    public Long join(Member member) {
        validateDuplicateMember(member);
        jpaMemberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember(Member member) {
        Optional<Member> findMember = jpaMemberRepository.findByNickName((member.getNickname()));
        if (findMember.isPresent()) {
            throw new IllegalArgumentException("이미 존재하는 닉네임입니다.");
        }

        findMember = jpaMemberRepository.findByMail((member.getMail()));
        if (findMember.isPresent())
        {
            throw new IllegalArgumentException("이미 존재하는 메일입니다.");
        }

        findMember = jpaMemberRepository.findByPhoneNumber((member.getPhoneNumber()));
        if (findMember.isPresent()) {
            throw new IllegalArgumentException("이미 존재하는 전화번호입니다.");
        }
    }

    /**
     * 회원 조회
     */

    public List<Member> findMembers() {
        return jpaMemberRepository.findAll();
    }

    public Optional<Member> findOne(Long memberId) {
        return jpaMemberRepository.findById(memberId);
    }

    public Optional<Member> findOne(String nickName) {
        return jpaMemberRepository.findByNickName(nickName);
    }
}

```

회원의 관한 서비스를 만들어 주었다.
일단 회원에서는 특별한 비즈니스 로직이 존재하지는 않고 대부분 위임이다.

그래도 서비스를 만들었던 이유는 예외 검증에 있다.
회원가입을 하는 과정에서 중복이 있는지 거를 수 있는 검증 메서드를 만들었다.


## 테스트

그래서 테스트 또한 검증에 관한 테스트만 진행했다.
Repository에서 이미 CR에 대한 테스트를 했기 때문이다.

```java
@SpringBootTest
@Transactional
class MemberServiceTest {

    @Autowired
    MemberService memberService;
    @Autowired
    MemberRepository jpaMemberRepository;

    @Test
    public void 중복회원_예외_닉네임() {
        //given
        Member member1 = createTestMember("hello",
                "jckim229@gmail.com", "01012341234");

        Member member2 = createTestMember("hello",
                "jckim229@gmail.com", "01043214231");

        //when, then
        memberService.join(member1);

        IllegalArgumentException e = assertThrows(IllegalArgumentException.class, ()
                -> memberService.join(member2));
        assertEquals("이미 존재하는 닉네임입니다.", e.getMessage());

        //변경 감지로 인한 update
        member1.changeMail("123@gmail.com");
        Member findMember1 = jpaMemberRepository.findById(member1.getId()).get();

        assertEquals(1, jpaMemberRepository.findAll().size());
        assertEquals("123@gmail.com", findMember1.getMail());
    }

    @Test
    public void 중복회원_예외_메일() {
        //given
        Member member1 = createTestMember("hello",
                "jckim229@gmail.com", "01012341234");

        Member member2 = createTestMember("hello123",
                "jckim229@gmail.com", "01043214231");

        //when, then
        memberService.join(member1);

        IllegalArgumentException e = assertThrows(IllegalArgumentException.class, ()
                -> memberService.join(member2));
        assertEquals("이미 존재하는 메일입니다.", e.getMessage());
    }

    @Test
    public void 중복회원_예외_전화번호() {
        //given
        Member member1 = createTestMember("hello",
                "jckim229@gmail.com", "01012341234");

        Member member2 = createTestMember("hello123",
                "zxcv@gmail.com", "01012341234");

        //when, then
        memberService.join(member1);

        IllegalArgumentException e = assertThrows(IllegalArgumentException.class, ()
                -> memberService.join(member2));
        assertEquals("이미 존재하는 전화번호입니다.", e.getMessage());
    }
```

위처럼 테스트를 진행했는데 변경 감지에 대한 테스트도 진행했다.

```java
        //변경 감지로 인한 update
        member1.changeMail("123@gmail.com");
        Member findMember1 = jpaMemberRepository.findById(member1.getId()).get();

        assertEquals(1, jpaMemberRepository.findAll().size());
        assertEquals("123@gmail.com", findMember1.getMail());
                
```
위처럼 진행했을 때 따로 persist하지 않아도 이미 영속성 객체이기 때문에 setting이 되면 바로 변경이 감지되어서 데이터베이스에 저장이 된다.

## 정리

다음으로는 게시글 관련 로직들을 구현해야한다.
이후에 아이템 관련 구매 로직전까지는 복잡하지는 않을 것 같다.
