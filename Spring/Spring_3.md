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


# 회원 객체 -> domain

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

# 회원 저장소 인터페이스 -> repository

```java
public interface MemberRepository {
    Member save(Member member);
    Optional<Member> findById(Long id);
    Optional<Member> findByName(String name);
    List<Member> findAll();

}

```
# 회원 저장소 구현체 -> 메모리 형식

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
-> save할 때마다 sequence를 ++
-> store에 키값과 밸류를 put
3. findById
-> id에 맞는 value를 반환하되 Null 일 수도 있으니 Optional로 감싸서 리턴한다.
4. findByName
-> store의 밸류값들로 stream을 내려보내고 같은 이름이면 리턴한다.
5. findAll
-> 모든 value 값을 ArrayList형식으로 반환
