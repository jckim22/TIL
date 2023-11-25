# 통합 테스트
애플리케이션의 여러 컴포넌트 혹은 모듈간의 상호작용과 조정을 검증하는데 중점을 둔 소프트웨어 테스팅 유형이다.
유닛 테스트에서 테스트된 유닛들이 결합되었을 때 올바르게 작동되는지를 확인한다.
데이터 통신, 데이터 공유, 컴포넌트간 전반적 제어 흐름에 관련된 문제를 식별하는데 도움이 된다.
분산 시스템, Micro Service Architecture (MSA), 타사 API 등에 의존하는 서비스 애플리케이션을 테스트할 때 중요하다.


개발 프로세스에서 보통 단위 테스트 후 시스템 테스트 전에 수행된다.
자동화 가능한 테스트 영역이다.
출처: https://jake-seo-dev.tistory.com/579 [제이크서 위키 블로그:티스토리]

# 구현

Transactional 어노테이션으로 인에 insertCommit selectCommit까지는 되어도 롤백이 되어  DB에 영향을 주지 않게 된다.

>이런 테스트 레벨에서는 굳이 DI를 생성자로 하지 않아도 된다.

```java
@SpringBootTest
@Transactional
class MemberServiceIntegrationTest {

    @Autowired MemberService memberService;
    @Autowired MemberRepository memberRepository;

    @Test
    void 회원가입() {
        //given
        Member member = new Member();
        member.setName("spring");

        //when
        Long saveId = memberService.join(member);

        //then
        Member findMember = memberService.findOne(saveId).get();
        org.assertj.core.api.Assertions.assertThat(member.getName()).isEqualTo(findMember.getName());

    }

    @Test
    public void 중복_회원_예외() {
        //given
        Member member1 = new Member();
        member1.setName("spring");

        Member member2 = new Member();
        member2.setName("spring");

        //when
        memberService.join(member1);
        IllegalArgumentException e = assertThrows(IllegalArgumentException.class, ()
                -> memberService.join(member2));

        org.assertj.core.api.Assertions.assertThat(
                e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");


//        try {
//            memberService.join(member2);
//            fail();
//        } catch (IllegalArgumentException e) {
//            org.assertj.core.api.Assertions.assertThat(
//                    e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
//        }

        //then


    }

    @Test
    void findMembers() {
    }

    @Test
    void findOne() {
    }
}
```


## 단점

기존에 JVM으로만 진행했던 단위테스트와 비교했을 때 통합 테스트는 컨테이너도 올리고 여러가지로 시간과 메모리를 많이 잡아 먹게 된다.

통합테스트가 필요할 수도 있지만, 단위 테스트보다 좋지 않은 테스트일 확률이 높다.
