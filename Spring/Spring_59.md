Entity 생성

Repository 생성


h2 Database 설정

`main/resources/application.yml` 
```yaml
spring:
  datasource:
#
  url: jdbc:h2:tcp://localhost/~/jpashop
  username: sa
  password:
  driver-class-name: org.h2.Driver
jpa:
  hibernate:
    ddl-auto: create
  properties:
    hibernate:
       show_sql: true
      format_sql: true
logging.level:
  org.hibernate.SQL: debug
# org.hibernate.type: trace #스프링 부트 2.x, hibernate5
# org.hibernate.orm.jdbc.bind: trace #스프링 부트 3.x, hibernate6
```

`spring.jpa.hibernate.ddl-auto: create`
이 옵션은 애플리케이션 실행 시점에 테이블을 drop 하고, 다시 생성한다.

>참고: 모든 로그 출력은 가급적 로거를 통해 남겨야 한다.
`show_sql` : 옵션은 `System.out` 에 하이버네이트 실행 SQL을 남긴다. `org.hibernate.SQL` : 옵션은 logger를 통해 하이버네이트 실행 SQL을 남긴다.

**주의!**
`application.yml` 같은 `yml` 파일은 띄어쓰기(스페이스) 2칸으로 계층을 만듭니다. 따라서 띄어쓰기 2칸을 필수로 적어주어야 합니다.
예를 들어서 아래의 `datasource` 는 `spring:` 하위에 있고 앞에 띄어쓰기 2칸이 있으므로
`spring.datasource` 가 됩니다. 다음 코드에 주석으로 띄어쓰기를 적어두었습니다

#### 쿼리 파라미터 로그 남기기 - 스프링 부트 3.0
스프링 부트 3.0 이상을 사용하면 라이브러리 버전을 1.9.0 이상을 사용해야 한다.
```
 implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.9.0'
```


# 도메인 분석 설계

- 회원 기능
회원 등록
회원 조회 
- 상품 기능
상품 등록 상품 수정 상품 조회
- 주문 기능
상품 주문
주문 내역 조회
주문 취소 
- 기타 요구사항
상품은 재고 관리가 필요하다.
상품의 종류는 도서, 음반, 영화가 있다. 상품을 카테고리로 구분할 수 있다.
상품 주문시 배송 정보를 입력할 수 있다.

![](https://velog.velcdn.com/images/jckim22/post/242be1a5-6c84-4a32-8687-180a638c8230/image.png)

**회원, 주문, 상품의 관계:** 회원은 여러 상품을 주문할 수 있다. 그리고 한 번 주문할 때 여러 상품을 선택할 수 있으므로 주문과 상품은 다대다 관계다. 하지만 이런 다대다 관계는 관계형 데이터베이스는 물론이고 엔티티에서도 거의 사용하 지 않는다. 따라서 그림처럼 주문상품이라는 엔티티를 추가해서 다대다 관계를 일대다, 다대일 관계로 풀어냈다.
**상품 분류:** 상품은 도서, 음반, 영화로 구분되는데 상품이라는 공통 속성을 사용하므로 상속 구조로 표현했다.


![](https://velog.velcdn.com/images/jckim22/post/d4a0dbd2-06ab-44d4-8f8b-158c3923105a/image.png)

**회원(Member):** 이름과 임베디드 타입인 주소( `Address` ), 그리고 주문( `orders` ) 리스트를 가진다.
**주문(Order):** 한 번 주문시 여러 상품을 주문할 수 있으므로 주문과 주문상품( `OrderItem` )은 일대다 관계다. 주문은 상품을 주문한 회원과 배송 정보, 주문 날짜, 주문 상태( `status` )를 가지고 있다. 주문 상태는 열거형을 사용했는데 주 문( `ORDER` ), 취소( `CANCEL` )을 표현할 수 있다.

**주문상품(OrderItem):** 주문한 상품 정보와 주문 금액( `orderPrice` ), 주문 수량( `count` ) 정보를 가지고 있다. (보 통 `OrderLine` , `LineItem` 으로 많이 표현한다.)
**상품(Item)**: 이름, 가격, 재고수량( `stockQuantity` )을 가지고 있다. 상품을 주문하면 재고수량이 줄어든다. 상품의 종류로는 도서, 음반, 영화가 있는데 각각은 사용하는 속성이 조금씩 다르다.
**배송(Delivery)**: 주문시 하나의 배송 정보를 생성한다. 주문과 배송은 일대일 관계다.
**카테고리(Category):** 상품과 다대다 관계를 맺는다. `parent` , `child` 로 부모, 자식 카테고리를 연결한다. 
**주소(Address)**: 값 타입(임베디드 타입)이다. 회원과 배송(Delivery)에서 사용한다.


>참고: 회원이 주문을 하기 때문에, 회원이 주문리스트를 가지는 것은 얼핏 보면 잘 설계한 것 같지만, 객체 세상은 실제 세계와는 다르다. 실무에서는 회원이 주문을 참조하지 않고, 주문이 회원을 참조하는 것으로 충분하다. 여기 서는 일대다, 다대일의 양방향 연관관계를 설명하기 위해서 추가했다.


![](https://velog.velcdn.com/images/jckim22/post/23f36e7f-7652-4ea0-ae16-bd5a54a7e665/image.png)

**MEMBER**: 회원 엔티티의 `Address` 임베디드 타입 정보가 회원 테이블에 그대로 들어갔다. 이것은 `DELIVERY` 테 이블도 마찬가지다.
**ITEM**: 앨범, 도서, 영화 타입을 통합해서 하나의 테이블로 만들었다. `DTYPE` 컬럼으로 타입을 구분한다.


>참고: 테이블명이 `ORDER` 가 아니라 `ORDERS` 인 것은 데이터베이스가 `order by` 때문에 예약어로 잡고 있는 경우가 많다. 그래서 관례상 `ORDERS` 를 많이 사용한다.
**참고: 실제 코드에서는 DB에 소문자 + _(언더스코어) 스타일을 사용하겠다.**
데이터베이스 테이블명, 컬럼명에 대한 관례는 회사마다 다르다. 보통은 대문자 + _(언더스코어)나 소문자 + _(언 더스코어) 방식 중에 하나를 지정해서 일관성 있게 사용한다. 강의에서 설명할 때는 객체와 차이를 나타내기 위해 데이터베이스 테이블, 컬럼명은 대문자를 사용했지만, **실제 코드에서는 소문자 + _(언더스코어) 스타일을 사용하 겠다.**

연관관계 매핑 분석
**회원과 주문:** 일대다 , 다대일의 양방향 관계다. 따라서 연관관계의 주인을 정해야 하는데, 외래 키가 있는 주문을 연관 관계의 주인으로 정하는 것이 좋다. 그러므로 `Order.member` 를 `ORDERS.MEMBER_ID` 외래 키와 매핑한다.
**주문상품과 주문:** 다대일 양방향 관계다. 외래 키가 주문상품에 있으므로 주문상품이 연관관계의 주인이다. 그러므로 `OrderItem.order` 를 `ORDER_ITEM.ORDER_ID` 외래 키와 매핑한다.
**주문상품과 상품:** 다대일 단방향 관계다. `OrderItem.item` 을 `ORDER_ITEM.ITEM_ID` 외래 키와 매핑한다. **주문과 배송:** 일대일 양방향 관계다. `Order.delivery` 를 `ORDERS.DELIVERY_ID` 외래 키와 매핑한다.
**카테고리와 상품:** `@ManyToMany` 를 사용해서 매핑한다.(실무에서 @ManyToMany는 사용하지 말자. 여기서는 다대 다 관계를 예제로 보여주기 위해 추가했을 뿐이다)


**참고: 외래 키가 있는 곳을 연관관계의 주인으로 정해라.**
연관관계의 주인은 단순히 외래 키를 누가 관리하냐의 문제이지 비즈니스상 우위에 있다고 주인으로 정하면 안된 다.. 예를 들어서 자동차와 바퀴가 있으면, 일대다 관계에서 항상 다쪽에 외래 키가 있으므로 외래 키가 있는 바퀴 를 연관관계의 주인으로 정하면 된다. 물론 자동차를 연관관계의 주인으로 정하는 것이 불가능 한 것은 아니지만, 자동차를 연관관계의 주인으로 정하면 자동차가 관리하지 않는 바퀴 테이블의 외래 키 값이 업데이트 되므로 관리 와 유지보수가 어렵고, 추가적으로 별도의 업데이트 쿼리가 발생하는 성능 문제도 있다. 자세한 내용은 JPA 기본 편을 참고하자.



# 엔티티 클래스 개발
예제에서는 설명을 쉽게하기 위해 엔티티 클래스에 Getter, Setter를 모두 열고, 최대한 단순하게 설계 실무에서는 가급적 Getter는 열어두고, Setter는 꼭 필요한 경우에만 사용하는 것을 추천

>참고: 이론적으로 Getter, Setter 모두 제공하지 않고, 꼭 필요한 별도의 메서드를 제공하는게 가장 이상적이다. 하지만 실무에서 엔티티의 데이터는 조회할 일이 너무 많으므로, Getter의 경우 모두 열어두는 것이 편리하다. Getter는 아무리 호출해도 호출 하는 것 만으로 어떤 일이 발생하지는 않는다. 하지만 Setter는 문제가 다르다. Setter를 호출하면 데이터가 변한다. Setter를 막 열어두면 가까운 미래에 엔티티가 도대체 왜 변경되는지 추적 하기 점점 힘들어진다. 그래서 엔티티를 변경할 때는 Setter 대신에 변경 지점이 명확하도록 변경을 위한 비즈니 스 메서드를 별도로 제공해야 한다.


# 엔티티 설계시 주의점
### 엔티티에는 가급적 Setter를 사용하지 말자
Setter가 모두 열려있다. 변경 포인트가 너무 많아서, 유지보수가 어렵다. 나중에 리펙토링으로 Setter 제거
### 모든 연관관계는 지연로딩으로 설정!
- 즉시로딩( `EAGER` )은 예측이 어렵고, 어떤 SQL이 실행될지 추적하기 어렵다. 특히 JPQL을 실행할 때 N+1 문제
가 자주 발생한다.
- 실무에서 모든 연관관계는 지연로딩( `LAZY` )으로 설정해야 한다.
- 연관된 엔티티를 함께 DB에서 조회해야 하면, fetch join 또는 엔티티 그래프 기능을 사용한다. 
- @XToOne(OneToOne, ManyToOne) 관계는 기본이 즉시로딩이므로 직접 지연로딩으로 설정해야 한다.
### 컬렉션은 필드에서 초기화 하자.
컬렉션은 필드에서 바로 초기화 하는 것이 안전하다.
- `null` 문제에서 안전하다.
- 하이버네이트는 엔티티를 영속화 할 때, 컬랙션을 감싸서 하이버네이트가 제공하는 내장 컬렉션으로 변경한다. 만 약 `getOrders()` 처럼 임의의 메서드에서 컬력션을 잘못 생성하면 하이버네이트 내부 메커니즘에 문제가 발생 할 수 있다. 따라서 필드레벨에서 생성하는 것이 가장 안전하고, 코드도 간결하다.


```
 Member member = new Member();
 System.out.println(member.getOrders().getClass());
 em.persist(member);
 System.out.println(member.getOrders().getClass());
//출력 결과
 class java.util.ArrayList
 class org.hibernate.collection.internal.PersistentBag
```


### 테이블, 컬럼명 생성 전략
스프링 부트에서 하이버네이트 기본 매핑 전략을 변경해서 실제 테이블 필드명은 다름
https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#howto-
configure-hibernate-naming-strategy http://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/


# 회원 레포지토리 개발
```java
package jpabook.jpashop.repository;

import jakarta.persistence.EntityManager;
import jakarta.persistence.PersistenceContext;
import jpabook.jpashop.domain.Member;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
@RequiredArgsConstructor
public class MemberRepository {

    private final EntityManager em;

    public void save(Member member){
        em.persist(member);
    }

    public Member findOne(Long id) {
        return em.find(Member.class, id);
    }

    public List<Member> findAll(){
        return em.createQuery("select m from Member m",Member.class)
                .getResultList();
    }

    public List<Member> findByName(String name){
         return em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name",name)
                 .getResultList();


    }
}

```

# 회원 서비스 개발

```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;

    /**
     * 회원 가입
     */
    @Transactional
    public Long join(Member member){
        validateDuplicateMember(member);
        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember(Member member){
        List<Member> findMembers = memberRepository.findByName((member.getName()));
        if(!findMembers.isEmpty()){
            throw new IllegalArgumentException("이미 존재하는 회원입니다.");
        }
    }

    //회원 전체 조회

    public List<Member> findMembers(){
        return memberRepository.findAll();
    }

    public Member findOne(Long memberId){
        return memberRepository.findOne(memberId);
    }

}

```

# 회원 테스트

test에 새로운 properties를 만들어서 인메모리로 테스트

```java
@SpringBootTest
@Transactional
class MemberServiceTest {

    @Autowired MemberService memberService;
    @Autowired MemberRepository memberRepository;

    @Test
    public void 회원가입() throws Exception {
        //given
        Member member = new Member();
        member.setName("Kim");;
        //when
        Long saveId = memberService.join(member);
        //then
        assertEquals(member, memberService.findOne(saveId));
    }
    
    @Test
    public void 중복_회원_예외() throws Exception {
        //given
        Member member1 = new Member();
        member1.setName("kim");

        Member member2 = new Member();
        member2.setName("kim");
        //when
        //then
        Assertions.assertThrows(IllegalArgumentException.class, () -> {
            memberService.join(member1);
            memberService.join(member2);
        });
    }
}
```

# 상품 도메인 수정

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "dtype")
@Getter
public abstract class Item {

    @Id
    @GeneratedValue
    @Column(name = "item_id")
    private Long id;

    @OneToMany(mappedBy = "item")
    private List<CategoryItem> categorys = new ArrayList<>();

    private String name;
    private int price;
    private int stockQuantity;

    //==비즈니스로직==//
    public void addStock(int quantity) {
        this.stockQuantity += quantity;
    }

    /**
     * stock 감소
     */

    public void removeStock(int quantity) {
        int restStock = this.stockQuantity - quantity;
        if (restStock < 0){
            throw new NotEnoughStockException("need more stock");
        }
        this.stockQuantity = restStock;
    }
}

```

상품 도메인에 재고를 증가시키고 감소시키는 비즈니스 로직을 추가한다.
감소시키는 과정에서 검증을 한다.

그에 대한 예외는 새로 만든다.


```java
public class NotEnoughStockException extends RuntimeException{
    public NotEnoughStockException() {
        super();
    }

    public NotEnoughStockException(String message) {
        super(message);
    }

    public NotEnoughStockException(String message, Throwable cause) {
        super(message, cause);
    }

    public NotEnoughStockException(Throwable cause) {
        super(cause);
    }

    protected NotEnoughStockException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}

```

RuntimeException을 오버라이딩 한다.

# 상품 레포지토리

```java
@Repository
@RequiredArgsConstructor
public class ItemRepository {

    private final EntityManager em;

    public void save(Item item){
        if (item.getId() == null){
            em.persist(item);
        }else {
            em.merge(item);
        }
    }

    public Item findOne(Long id){
        return em.find(Item.class,id);
    }

    public List<Item> findAll(){
        return em.createQuery("select i from Item i", Item.class)
                .getResultList();
    }
}

```

상품 서비스는 이를 위임한다.

## 상품 서비스
>이렇게 위임만 하는 서비스는 꼭 필요한가? 에 대해서 의문을 가져보자

```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class ItemService {

    private final ItemRepository itemRepository;

    @Transactional
    public void saveItem(Item item){
        itemRepository.save(item);
    }

    public List<Item> findItems(){
        return itemRepository.findAll();
    }

    public Item findOne(Long itemId){
        return itemRepository.findOne(itemId);
    }
}

```



# 주문 엔티티 로직
```java
ntity
@Table(name = "orders")
@Getter @Setter
public class Order {

    @Id @GeneratedValue
    @Column(name = "order_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<>();

    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;

    private LocalDateTime orderDate; //주문시간

    @Enumerated(EnumType.STRING)
    private OrderStatus status; //주문상태 [ORDER, CANCEL]

    //연관관계 메서드
    public void setMember(Member member){
        this.member = member;
        member.getOrders().add(this);
    }

    public void addOrderItem(OrderItem orderItem){
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }

    public void setDelivery(Delivery delivery){
        this.delivery = delivery;
        delivery.setOrder(this);
    }

    //==생성 메서드==//
    public static Order createOrder(Member member, Delivery delivery, OrderItem... orderItems ){
        Order order = new Order();
        order.setMember(member);
        order.setDelivery(delivery);
        for (OrderItem orderItem: orderItems){
            order.addOrderItem(orderItem);
        }
        order.setStatus(OrderStatus.ORDER);
        order.setOrderDate(LocalDateTime.now());
        return order;
    }

    //==비즈니스 로직==//
    /**
     * 주문 취소
     */
    public void cancel(){
        if(delivery.getStatus() == DeliveryStatus.COMP){
            throw new IllegalArgumentException("이미 배송완료된 상품은 취소가 불가능합니다.");
        }

        this.setStatus(OrderStatus.CANCEL);
        for(OrderItem orderItem : orderItems){
            orderItem.cancel();
        }
    }

    //==조회 로직==//
    public int getTotalPrice(){
        return orderItems.stream()
                .mapToInt(OrderItem::getTotalPrice)
                .sum();

    }
}

```
# 주문 상품 엔티티 로직
```java
@Entity
@Getter @Setter
public class OrderItem {

    @Id @GeneratedValue
    @Column(name = "order_item_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "item_id")
    private Item item;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;

    private int orderPrice;
    private int count;
    //==생성 메서드==//
    public static OrderItem createOrderItem(Item item, int orderPrice, int count){
        OrderItem orderItem = new OrderItem();
        orderItem.setItem(item);
        orderItem.setOrderPrice(orderPrice);
        orderItem.setCount(count);

        item.removeStock(count);
        return orderItem;
    }

    //==비즈니스 로직==//
    public void cancel(){
        getItem().addStock(count);
    }

    public int getTotalPrice() {
        return getOrderPrice() * getCount();
    }
}

```


# 주문 레포지토리 개발

일단 이름과 주문 상태가 모두 있다고 가정할 때 쿼리 로직은 아래와 같다.
```java
@Repository
@RequiredArgsConstructor
public class OrderRepository {

    private final EntityManager em;

    public void save(Order order){
        em.persist(order);
    }

    public Order findOne(Long id){
        return em.find(Order.class, id);
    }

    public List<Order> findAll(OrderSearch orderSearch){
        return em.createQuery("select o from Order o join o.member m" +
                        " where o.status = :status " +
                        " and m.name like :name", Order.class)
                .setParameter("status", orderSearch.getOrderStatus())
                .setParameter("name", orderSearch.getMemberName())
                .setMaxResults(1000) //최대 1000건
                .getResultList();

    }


}

```

# jpql 동적 쿼리

매우 복잡하다
```java
 public List<Order> findAllByString(OrderSearch orderSearch) {
     //language=JPAQL
     String jpql = "select o From Order o join o.member m";
     boolean isFirstCondition = true;
//주문 상태 검색
if (orderSearch.getOrderStatus() != null) {
         if (isFirstCondition) {
             jpql += " where";
             isFirstCondition = false;
         } else {
             jpql += " and";
}
         jpql += " o.status = :status";
     }
//회원 이름 검색
if (StringUtils.hasText(orderSearch.getMemberName())) {
         if (isFirstCondition) {
             jpql += " where";
             isFirstCondition = false;
         } else {
             jpql += " and";
}
         jpql += " m.name like :name";
     }
TypedQuery<Order> query = em.createQuery(jpql, Order.class) .setMaxResults(1000); //최대 1000건
     if (orderSearch.getOrderStatus() != null) {
         query = query.setParameter("status", orderSearch.getOrderStatus());
     }
     if (StringUtils.hasText(orderSearch.getMemberName())) {
              query = query.setParameter("name", orderSearch.getMemberName());
     }
     return query.getResultList();
 }
```

## JPA Criteria로 처리
```java
 public List<Order> findAllByCriteria(OrderSearch orderSearch) {
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Order> cq = cb.createQuery(Order.class);
Root<Order> o = cq.from(Order.class);
Join<Order, Member> m = o.join("member", JoinType.INNER); //회원과 조인
     List<Predicate> criteria = new ArrayList<>();
//주문 상태 검색
if (orderSearch.getOrderStatus() != null) {
         Predicate status = cb.equal(o.get("status"),
 orderSearch.getOrderStatus());
         criteria.add(status);
     }
//회원 이름 검색
if (StringUtils.hasText(orderSearch.getMemberName())) {
         Predicate name =
                 cb.like(m.<String>get("name"), "%" + orderSearch.getMemberName()
 + "%");
         criteria.add(name);
}
     cq.where(cb.and(criteria.toArray(new Predicate[criteria.size()])));
TypedQuery<Order> query = em.createQuery(cq).setMaxResults(1000); //최대 1000 건
     return query.getResultList();
 }
```

이 역시도 너무 복잡하다.

# QueryDsl로 해결

```java
public List<Order> findAll(OrderSearch orderSearch) {
	QOrder order = QOder.order;
    QMember member = QMember.member;
    
    return query
    		.select(order)
            .from(order)
            .join(order.member, member)
            .where(statusEq(orderSearch.getOrderStatus()),
            		nameLike(orderSearch.getMemberName()))
            .limit(1000)
            .fetch();
}

private BooleanExpression statusEq(OrderStatus statusCond) {
	if (statusCond == null) {
    	return null;
    }
    return order.status.eq(statusCond);
}

private BooleanExpression nameLike(String nameCond) {
	if(!StringUtils.hasText(nameCond)) {
    	return null;
    }
    return member.name.like(nameCond);
```

훨씬 좋은 가독성을 갖게 되었다.
