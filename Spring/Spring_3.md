# 일반적인 웹 계층 구조

![](https://velog.velcdn.com/images/jckim22/post/3a5dc98b-a6ad-4260-970c-1bc08ef1c2fd/image.png)
컨트롤러: 웹 MVC의 컨트롤러 역할
서비스: 핵심 비즈니스 로직 구현
리포지토리: 데이터베이스에 접근, 도메인 객체를 DB에 저장하고 관리
도메인: 비즈니스 도메인 객체, 예) 회원, 주문, 쿠폰 등등 주로 데이터베이스에 저장하고 관리됨

# 클래스 의존 관계

![](https://velog.velcdn.com/images/jckim22/post/e114264b-1dab-4165-a66b-362e67a3ce48/image.png)

아직 데이터 저장소가 선정되지 않아서, 우선 인터페이스로 구현 클래스를 변경할 수 있도록 설계 데이터 저장소는 RDB, NoSQL 등등 다양한 저장소를 고민중인 상황으로 가정
개발을 진행하기 위해서 초기 개발 단계에서는 구현체로 가벼운 메모리 기반의 데이터 저장소 사용

> 인터페이스를 만들어 놓으면 DB는 구현만 하면되니 옮기기 쉽다.


## 회원 객체 -> domain

```java
public class Member {
    private int id;
    private String name;

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setId(Long id) {
        this.id = id;
    }
}
```

# 회원 관리 예제(도메인, 레포지토리)

## 회원 저장소 인터페이스 -> repository

```java
public interface MemberRepository {
    Member save(Member member);
    Optional<Member> findById(Long id);
    Optional<Member> findByName(String name);
    List<Member> findAll();

}

```
## 회원 저장소 구현체 -> 메모리 형식

>추후에 다른 DB로 구현할 수 있음

```java
public class MemoryMemberRepository implements MemberRepository {

    private static Map<Long, Member> store = new HashMap<>();
    private static Long sequence = 0L;

    @Override
    public Member save(Member member) {
        member.setId(++sequence);
        store.put(member.getId(), member);
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        return Optional.ofNullable(store.get(id));
    }

    @Override
    public Optional<Member> findByName(String name) {
        return store.values().stream()
                .filter(member -> member.getName().equals(name))
                .findAny();
    }

    @Override
    public List<Member> findAll() {
        return new ArrayList<>(store.values());
    }
}
```

1. ID를 key로한 member가 모여있는 Map 생성
2. save
-> save할 때마다 sequence가 가산적으로 증가
-> store에 키값과 밸류를 put
3. findById
-> id에 맞는 value를 반환하되 Null 일 수도 있으니 Optional로 감싸서 리턴한다.
4. findByName
-> store의 밸류값들로 stream을 내려보내고 같은 이름이면 리턴한다.
5. findAll
-> 모든 value 값을 ArrayList형식으로 반환

# 회원 관리 예제(도메인, 레포지토리) - 테스트


## 회원 저장소 구현체 -> 메모리 형식 -> 테스트

```java
public class MemoryMemberRepositoryTest {

    MemoryMemberRepository repository = new MemoryMemberRepository();

    @AfterEach
    public void afterEach(){
        repository.clearStore();
    }

    @Test
    public void save() {
        Member member = new Member();
        member.setName("spring");

        repository.save(member);

        Member result = repository.findById(member.getId()).get();
//        Assertions.assertEquals(member,result);
        Assertions.assertThat(member).isEqualTo(result);

    }

    @Test
    public void findByName() {
        Member member1 = new Member();
        member1.setName("spring1");
        repository.save(member1);

        Member member2 = new Member();
        member2.setName("spring2");
        repository.save(member2);

        Member result = repository.findByName("spring1").get();

        Assertions.assertThat(result).isEqualTo(member1);
    }

    @Test
    public void findAll() {
        Member member1 = new Member();
        member1.setName("spring1");
        repository.save(member1);

        Member member2 = new Member();
        member2.setName("spring2");
        repository.save(member2);

        List<Member> result = repository.findAll();

        Assertions.assertThat(result.size()).isEqualTo(2);
    }
}
```
메서드 단위, 클래스 단위로 테스트 할 수 있다.

>각 테스트가 끝날 때마다 afterEach로 clear 메소드를 호출하여 데이터를 clear 해주어야 한다. (clear 메소드는 따로 생성했다.)

>이 테스트 코드를 개발 전에 먼저 작성하고 그 틀에 맞추는 형식으로 개발하는 방법이 테스트 주도 개발(TDD) 이다,


# 회원 관리 예제(서비스)


>#### InteliJ 유용한 커맨드
command+option+v: return 형식에 맞는 변수 자동 선언
command+shift+enter: ; 후 줄바꿈
control+T: Extract method와 같은 리팩토링 기능들
command+n: getter, setter, 생성자 등

```java
public class MemberService {

    private final MemberRepository memberRepository = new MemoryMemberRepository();

    /**
     * 회원가입
     *
     * @param member
     * @return
     */
    public Long join(Member member) {
        //같은 이름이 있는 중복 회원X
        validateDuplicateMember(member);
        
        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember(Member member) {
        memberRepository.findByName(member.getName())
                .ifPresent(m -> {
                    throw new IllegalArgumentException("이미 존재하는 회원입니다.");
                });
    }

    /**
     * 전체 회원 조회
     */
    public List<Member> findMembers() {
        return memberRepository.findAll();
    }

    public Optional<Member> findOne(Long memberId) {
        return memberRepository.findById(memberId);
    }
}
```

Optional의 isPresent를 사용해서 중복처리를 했다.
findMembers와 findOne은 지금은 그저 기존에 repository의 메소드를 한번 더 감싸준 형태지만, 네이밍이 서비스에 가까운 만큼 서비스 로직이라고 생각해야한다.

네이밍부터 엄연히 차이가 있다.

>#### ex)
findById는 도메인 네이밍
findOne, findMembers는 서비스에 더 가까운 네이밍


# 회원 관리 예제(서비스) - 테스트

```java
class MemberServiceTest {

    MemberService memberService;
    MemoryMemberRepository memberRepository;

    @BeforeEach
    public void beforeEach() {
        memberRepository = new MemoryMemberRepository();
        memberService = new MemberService(memberRepository);
    }

    @AfterEach
    public void afterEach() {
        memberRepository.clearStore();
    }

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

기존에는 MemberService에서 Repository를 생성했는데 그렇게 되면 테스트할 때 또 레포지토리를 따로 생성해야 했었다.

그래서 외부에서 레포지토리를 받는 형식으로 바꾸었다.
아래 코드를 보면

```java
public class MemberService {

    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
```

생성자로 서비스 입장에서 외부에서 레포지토리를 받는다.
그리고 테스트 코드에서 테스트 시작전마다 레포지토리를 서비스에 넣어준다.

>이렇게 서비스 입장에서 외부에서 넣어주는 것은 **Dependency Injection** 줄여서 DI라고 한다.

또한 테스트 메서드 명은 통상적으로 한글도 허용한다.
예외를 확인하는 것이 더 중요하기 때문에 asserThrows로 예외를 확인한다.

>try-catch로도 예외를 잡아낼 수 있다.
예외 명이 같은지 isEquals로 확인하자
메서드 단위, 클래스 단위로 테스트 할 수 있다.

>각 테스트가 끝날 때마다 afterEach로 clear 메소드를 호출하여 데이터를 clear 해주어야 한다. (clear 메소드는 따로 생성했다.)

>이 테스트 코드를 개발 전에 먼저 작성하고 그 틀에 맞추는 형식으로 개발하는 방법이 테스트 주도 개발(TDD) 이다,

