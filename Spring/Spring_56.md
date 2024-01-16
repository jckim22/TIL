![](https://velog.velcdn.com/images/jckim22/post/96b4a156-4a8a-4a57-97d3-594c24f73301/image.png)
![](https://velog.velcdn.com/images/jckim22/post/14419cf1-c8f6-4975-9cce-9de03fd566d3/image.png)


중간에서 `JpaItemRepositoryV2` 가 어댑터 역할을 해준 덕분에 `ItemService` 가 사용하는
`ItemRepository` 인터페이스를 그대로 유지할 수 있고 클라이언트인 `ItemService` 의 코드를 변경하지 않 아도 되는 장점이 있다.


**고민**
- 구조를 맞추기 위해서, 중간에 어댑터가 들어가면서 전체 구조가 너무 복잡해지고 사용하는 클래스도 많아지는 단 점이 생겼다.
- 실제 이 코드를 구현해야하는 개발자 입장에서 보면 중간에 어댑터도 만들고, 실제 코드까지 만들어야 하는 불편 함이 생긴다.
- 유지보수 관점에서 `ItemService` 를 변경하지 않고, `ItemRepository` 의 구현체를 변경할 수 있는 장점이 있다. 그러니까 DI, OCP 원칙을 지킬 수 있다는 좋은 점이 분명히 있다. 하지만 반대로 구조가 복잡해지면서 어 댑터 코드와 실제 코드까지 함께 유지보수 해야 하는 어려움도 발생한다.

**다른 선택**
여기서 완전히 다른 선택을 할 수도 있다.
`ItemService` 코드를 일부 고쳐서 직접 스프링 데이터 JPA를 사용하는 방법이다.
DI, OCP 원칙을 포기하는 대신에, 복잡한 어댑터를 제거하고, 구조를 단순하게 가져갈 수 있는 장점이 있다.

![](https://velog.velcdn.com/images/jckim22/post/82920342-9ea3-45b2-aff8-ce36463b14bd/image.png)


- `ItemService` 에서 스프링 데이터 JPA로 만든 리포지토리를 직접 참조한다. 물론 이 경우 `ItemService` 코드를 변경해야 한다.

**런타임 객체 의존 관계**

**트레이드 오프**
이것이 바로 트레이드 오프다.
- DI, OCP를 지키기 위해 어댑터를 도입하고, 더 많은 코드를 유지한다.
- 어댑터를 제거하고 구조를 단순하게 가져가지만, DI, OCP를 포기하고, `ItemService` 코드를 직접 변경한다.


결국 여기서 발생하는 트레이드 오프는 구조의 안정성 vs 단순한 구조와 개발의 편리성 사이의 선택이다.
이 둘 중에 하나의 정답만 있을까? 그렇지 않다. 어떤 상황에서는 구조의 안정성이 매우 중요하고, 어떤 상황에서는 단 순한 것이 더 나은 선택일 수 있다.


개발을 할 때는 항상 자원이 무한한 것이 아니다. 그리고 어설픈 추상화는 오히려 독이 되는 경우도 많다. 무엇보다 **추상 화도 비용이 든다.** 인터페이스도 비용이 든다. 여기서 말하는 비용은 유지보수 관점에서 비용을 뜻한다. 이 추상화 비용 을 넘어설 만큼 효과가 있을 때 추상화를 도입하는 것이 실용적이다.


이런 선택에서 하나의 정답이 있는 것은 아니지만, 프로젝트의 현재 상황에 맞는 더 적절한 선택지가 있다고 생각한다. 그리고 현재 상황에 맞는 선택을 하는 개발자가 좋은 개발자라 생각한다.(김영한님 의견)


# 실용적인 구조
마지막에 Querydsl을 사용한 리포지토리는 스프링 데이터 JPA를 사용하지 않는 아쉬움이 있었다. 물론 Querydsl을 사용하는 리포지토리가 스프링 데이터 JPA 리포지토리를 사용하도록 해도 된다.


이번에는 스프링 데이터 JPA의 기능은 최대한 살리면서, Querydsl도 편리하게 사용할 수 있는 구조를 만들어보겠다.


**복잡한 쿼리 분리**


![](https://velog.velcdn.com/images/jckim22/post/1ae75942-4fff-43ff-8ae4-1918970e1a0e/image.png)


- `ItemRepositoryV2` 는 스프링 데이터 JPA의 기능을 제공하는 리포지토리이다. 
- `ItemQueryRepositoryV2` 는 Querydsl을 사용해서 복잡한 쿼리 기능을 제공하는 리포지토리이다.


#### ItemQueryRepositoryV2
```java
@Repository
public class ItemQueryRepositoryV2 {

    private final JPAQueryFactory query;

    public ItemQueryRepositoryV2(EntityManager em) {
        this.query = new JPAQueryFactory(em);
    }

    public List<Item> findAll(ItemSearchCond cond) {
        return query.select(item)
                .from(item)
                .where(
                        likeItemName(cond.getItemName()),
                        maxPrice(cond.getMaxPrice())
                )
                .fetch();
    }

    private BooleanExpression likeItemName(String itemName) {
        if (StringUtils.hasText(itemName)) {
            return item.itemName.like("%" + itemName + "%");
        }
        return null;
    }

    private BooleanExpression maxPrice(Integer maxPrice) {
        if (maxPrice != null) {
            return item.price.loe(maxPrice);
        }
        return null;
    }

}

```
#### ItemRepositoryV2
```java
public interface ItemRepositoryV2 extends JpaRepository<Item, Long> {
}

```

#### V2Config

```java
@Configuration
@RequiredArgsConstructor
public class V2Config {

    private final EntityManager em;
    private final ItemRepositoryV2 itemRepositoryV2; //SpringDataJPA

    @Bean
    public ItemService itemService() {
        return new ItemServiceV2(itemRepositoryV2, itemQueryRepositoryV2());
    }

    @Bean
    public ItemQueryRepositoryV2 itemQueryRepositoryV2() {
        return new ItemQueryRepositoryV2(em);
    }

    @Bean
    public ItemRepository itemRepository() {
        return new JpaItemRepositoryV3(em);
    }

}

```

> itemRepositoryV2 Spirng Data JPA인데 이건 Spring에서 자동으로 주입을 해준다.

#### ItemServiceV2

```java
@Service
@RequiredArgsConstructor
@Transactional
public class ItemServiceV2 implements ItemService {

    private final ItemRepositoryV2 itemRepositoryV2;
    private final ItemQueryRepositoryV2 itemQueryRepositoryV2;

    @Override
    public Item save(Item item) {
        return itemRepositoryV2.save(item);
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        Item findItem = itemRepositoryV2.findById(itemId).orElseThrow();
        findItem.setItemName(updateParam.getItemName());
        findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }

    @Override
    public Optional<Item> findById(Long id) {
        return itemRepositoryV2.findById(id);
    }

    @Override
    public List<Item> findItems(ItemSearchCond cond) {
        return itemQueryRepositoryV2.findAll(cond);
    }
}

```
위처럼 구조를 변경하면 좀 더 빠르고 실용적인 구조로 짤 수 있다.
하지만 OCP, DIP에서는 아쉬운 모습이 보인다.

허나 정답은 없다(김영한님 의견)

상황에 맞춰서 실용적인 구조로 가야할지 추상화를 신경 써야할지에 대해 고민을 해보는 것이 좋다.


# 다양한 데이터 접근 기술 조합
어떤 데이터 접근 기술을 선택하는 것이 좋을까?
이 부분은 하나의 정답이 있다기 보다는, 비즈니스 상황과, 현재 프로젝트 구성원의 역량에 따라서 결정하는 것이 맞다 생각한다. `JdbcTemplate` 이나 `MyBatis` 같은 기술들은 SQL을 직접 작성해야 하는 단점은 있지만 기술이 단순하 기 때문에 SQL에 익숙한 개발자라면 금방 적응할 수 있다.
JPA, 스프링 데이터 JPA, Querydsl 같은 기술들은 개발 생산성을 혁신할 수 있지만, 학습 곡선이 높기 때문에, 이런 부분을 감안해야 한다. 그리고 매우 복잡한 통계 쿼리를 주로 작성하는 경우에는 잘 맞지 않는다.

개인적(김영한)으로 추천하는 방향은 JPA, 스프링 데이터 JPA, Querydsl을 기본으로 사용하고, 만약 복잡한 쿼리를 써야 하는 데, 해결이 잘 안되면 해당 부분에는 JdbcTemplate이나 MyBatis를 함께 사용하는 것이다.
실무에서 95% 정도는 JPA, 스프링 데이터 JPA, Querydsl 등으로 해결하고, 나머지 5%는 SQL을 직접 사용해야 하 니 JdbcTemplate이나 MyBatis로 해결한다. 물론 이 비율은 프로젝트 마다 다르다. 아주 복잡한 통계 쿼리를 자주 작 성해야 하면 JdbcTemplate이나 MyBatis의 비중이 높아질 수 있다.

**트랜잭션 매니저 선택**
JPA, 스프링 데이터 JPA, Querydsl은 모두 JPA 기술을 사용하는 것이기 때문에 트랜잭션 매니저로
`JpaTransactionManager` 를 선택하면 된다. 해당 기술을 사용하면 스프링 부트는 자동으로
`JpaTransactionManager` 를 스프링 빈에 등록한다.
그런데 `JdbcTemplate` , `MyBatis` 와 같은 기술들은 내부에서 JDBC를 직접 사용하기 때문에 `DataSourceTransactionManager` 를 사용한다.
따라서 JPA와 JdbcTemplate 두 기술을 함께 사용하면 트랜잭션 매니저가 달라진다. 결국 트랜잭션을 하나로 묶을 수 없는 문제가 발생할 수 있다. 그런데 이 부분은 걱정하지 않아도 된다.

**JpaTransactionManager의 다양한 지원**
`JpaTransactionManager` 는 놀랍게도 `DataSourceTransactionManager` 가 제공하는 기능도 대부분 제공
한다. JPA라는 기술도 결국 내부에서는 DataSource와 JDBC 커넥션을 사용하기 때문이다. 따라서 `JdbcTemplate` , `MyBatis` 와 함께 사용할 수 있다.
결과적으로 `JpaTransactionManager` 를 하나만 스프링 빈에 등록하면, JPA, JdbcTemplate, MyBatis 모두를 하나의 트랜잭션으로 묶어서 사용할 수 있다. 물론 함께 롤백도 할 수 있다.

>
**주의점**
이렇게 JPA와 JdbcTemplate을 함께 사용할 경우 JPA의 플러시 타이밍에 주의해야 한다. JPA는 데이터를 변경하면 변경 사항을 즉시 데이터베이스에 반영하지 않는다. 기본적으로 트랜잭션이 커밋되는 시점에 변경 사항을 데이터베이스 에 반영한다. 그래서 하나의 트랜잭션 안에서 JPA를 통해 데이터를 변경한 다음에 JdbcTemplate을 호출하는 경우 JdbcTemplate에서는 JPA가 변경한 데이터를 읽기 못하는 문제가 발생한다.
이 문제를 해결하려면 JPA 호출이 끝난 시점에 JPA가 제공하는 플러시라는 기능을 사용해서 JPA의 변경 내역을 데이 터베이스에 반영해주어야 한다. 그래야 그 다음에 호출되는 JdbcTemplate에서 JPA가 반영한 데이터를 사용할 수 있다.


>정리
`ItemServiceV2` 는 스프링 데이터 JPA를 제공하는 `ItemRepositoryV2` 도 참조하고, Querydsl과 관련된
`ItemQueryRepositoryV2` 도 직접 참조한다. 덕분에 `ItemRepositoryV2` 를 통해서 스프링 데이터 JPA 기능을 적절히 활용할 수 있고, `ItemQueryRepositoryV2` 를 통해서 복잡한 쿼리를 Querydsl로 해결할 수 있다.
이렇게 하면서 구조의 복잡함 없이 단순하게 개발할 수 있다.
본인이 진행하는 프로젝트의 규모가 작고, 속도가 중요하고, 프로토타입 같은 시작 단계라면 이렇게 단순하면서 라이브 러리의 지원을 최대한 편리하게 받는 구조가 더 나은 선택일 수 있다.
하지만 이 구조는 리포지토리의 구현 기술이 변경되면 수 많은 코드를 변경해야 하는 단점이 있다.
이런 선택에서 하나의 정답은 없다. 이런 트레이드 오프를 알고, 현재 상황에 더 맞는 적절한 선택을 하는 좋은 개발자가 있을 뿐이다.

