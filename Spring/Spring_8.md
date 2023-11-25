- JPA는 기존의 반복 코드는 물론이고, 기본적인 SQL도 JPA가 직접 만들어서 실행해준다.
- JPA를 사용하면, SQL과 데이터 중심의 설계에서 객체 중심의 설계로 패러다임을 전환을 할 수 있다. 
- JPA를 사용하면 개발 생산성을 크게 높일 수 있다

# 라이브러리 

```
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```


# 설정
```
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=none
```

위 설정으로 JPA가 던지는 쿼리를 볼 수 있게 된다.
또 ddl-auto=none 설정으로 JPA가 자동으로 테이블을 생성하는 기능을 OFF 한다.

# Entity 매핑
기존 도메인에 있는 객체를 Entity로 인식하게 하기 위해 
어노테이션을 작성한다.

```
@Entity
```


# Transaction 매핑

- 스프링은 해당 클래스의 메서드를 실행할 때 트랜잭션을 시작하고, 메서드가 정상 종료되면 트랜잭션을 커밋한다. 만약 런타임 예외가 발생하면 롤백한다.
- JPA를 통한 모든 데이터 변경은 트랜잭션 안에서 실행해야 한다.

이 또한 어노테이션으로 설정한다.

```
@Transactional
```


# JPA 저장소 구현

```java
public class JpaMemberRepository implements MemberRepository {

    private final EntityManager em;

    //DI 됨 (build.gradle)
    public JpaMemberRepository(EntityManager em) {
        this.em = em;
    }

    @Override
    public Member save(Member member) {
        em.persist(member);
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id);
        return Optional.ofNullable(member);
    }

    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name",name)
                .getResultList();
        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }
}

```

훨씬 간단해졌다.

JPA는 EntityManager를 DI 받는다.
EntityManager는 
- persist로 데이터를 인스턴스 그대로 매핑하여 저장하고
- ID와 같은 Key 값으로 조회를 하게 되면 find로 조회하여 바로 인스턴스를 던진다.
- 또 이름 같은 Key가 아닌 데이터로 조회를 할 때는 JPQL이라는 쿼리를 날려야한다.
