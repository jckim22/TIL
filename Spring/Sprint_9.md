이 JPA를 인터페이스만으로 간단하게 해결하는 방법이 있다.
아래를 보자

```java
 public interface SpringDataJpaMemberRepository extends JpaRepository<Member,
    Long>, MemberRepository {
        Optional<Member> findByName(String name);
    }
```

이렇게 Interface로 JpaRepository와 MemberRepository만 상속 받고 key가 아닌 Name으로 조회했던 findByName메서드만 override하면 끝이다.

이로써 지금까지 했던 모든 기능들이 구현되었다.

```java
@Configuration
  public class SpringConfig {
  private final MemberRepository memberRepository;
  
  @Autowrierd
  public SpringConfig(MemberRepository memberRepository) {
          this.memberRepository = memberRepository;
}
      @Bean
      public MemberService memberService() {
          return new MemberService(memberRepository);
      }
}
```

>config에서도 위처럼 설정해주면 스프링 데이터 JPA가 SpringDataJpaMemberRepository 를 스프링 빈으로 자동 등록해준다.
사실 정확히는 구현되지 않고 인터페이스만 존재하는 Repository를 찾는다.
또한 EntityManager를 DI 받았던 것과 달리 MemberRepository를 상속 받았음으로서 자동으로 DI 된다.

# 제공 기능

![](https://velog.velcdn.com/images/jckim22/post/0bbaca50-5226-4219-ab40-926f16677127/image.png)

![](https://velog.velcdn.com/images/jckim22/post/f5b9f5ba-e9ad-4109-8395-ef9d372a40da/image.png)

위처럼 다양한 기능들을 제공하기 때문에 내가 굳이 일반적인 기능은 구현할 필요가 없다.

>하지만 findByName과 같이 공통화 할 수 없는 기능들은 직접 구현한다.

>findByName을 @Override 어노테이션을 붙이면 이름을 보고 기능을 구현해준다.
인터페이스 이름만으로 기능 구현을 끝낼 수 있는 것이다.


>실무에서는 JPA와 스프링 데이터 JPA를 기본으로 사용하고, 복잡한 동적 쿼리는 Querydsl이라는 라이브러리를 사용하면 된다. Querydsl을 사용하면 쿼리도 자바 코드로 안전하게 작성할 수 있고, 동적 쿼리도 편리하게 작성할 수 있다. 이 조합으로 해결하기 어려운 쿼리는 JPA가 제공하는 네이티브 쿼리를 사용하거나, 앞서 학습한 스프링 JdbcTemplate를 사용하면 된다.
