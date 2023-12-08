다음 코드를 조금 간단하게 리팩토링 해보자

```java
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
//    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
```
생성자가 딱 1개만 있으면 `@Autowired` 를 생략할 수 있다.

```java
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
//    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();

    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
```

# Lombok
더 줄여보기 전에 lombok이라는 것을 알아보자.

lombok 플러그인을 다운 받고 lombok 라이브러리를 땡겨온다.
그리고 세팅에서 AnnotaionProcessor에서 enable 설정을 한다.

```java
@Getter
@Setter
@ToString
public class HelloLombok {
    private String name;
    private int age;

    public static void main(String[] args){
        HelloLombok hl = new HelloLombok();
        hl.setAge(123);
        hl.setName("sdafasdf");
        System.out.println(hl.getAge());
        System.out.println(hl.toString());
    }
}
```

이제 우리는 위와 같은 기능을 사용할 수 있다.
어노테이션 만으로도 getter, setter, ToString 등을 사용할 수 있다.

# 최신 트렌드

요즘은 생성자를 하나만 써서 @Autowired를 생략하는 추세이다.
이 트렌트를 따라가며 lombok을 사용한다면 더 트렌디한 코드를 짤 수 있다.

```java
@Component
public class OrderServiceImpl implements OrderService{
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
//    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();

    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
```
이제 위 코드를 lombok을 사용해 줄여보자.

```java
@Component
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService{
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
```
끝이다

@RequiredArgsConstrucor 어노테이션을 통해 위 코드와 똑같은 동작을 하는 코드를 작성했다.

# 조회 빈이 2개이상 일 때

## Autowired 필드 명 매칭

RateDiscountPolicy와 FixDiscountPolicy가 둘 다 컴포넌트로 등록이 되어 있을 때 우선순위를 정해야한다.

먼저 스프링은 타입으로판별한다.
둘 다 DiscountPolicy의 구현체이므로 여기서는 동일하다.
그 다음에 필드명으로 우선순위를 정할 수 있다.

```java
@Autowired
DiscountPolicy rateDiscountPolicy
```

위와 같은 방법으로 할 수 있다.

## @Qualify 사용
```java
@Component
@Qualify("mainDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy {
```
위와 같이 등록을 하고

```java
@Autowired
@Qualify("mainDiscountPolicy")
DiscountPolicy rateDiscountPolicy

```
위처럼 우선순위를 찾아갈 수 있다.
하지만 모든 컴포넌트 마다 @Qualify를 붙여야 되고 의존관계를 받는 쪽에서도 @Qualify를 붙여야하는 수고스러움이 붙는다.


## @Primary

```java
@Component
@Primary
public class RateDiscountPolicy implements DiscountPolicy {
```

그래서 위처럼 @Primary를 붙여주게 되면 알아서 이걸 먼저 찾는다.


#### 어노테이션 만들기


```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Qualifier("mainDiscountPolicy")
public @interface MainDiscountPolicy {
}
```

Qualify는 문자이기 때문에 컴퓨터가 인식하지 못한다.
그래서 어노테이션을 만들자
위처럼 어노테이션을 준비한다.


```java
@Component
@MainDiscountPolicy
public class RateDiscountPolicy implements DiscountPolicy {
```
위와 같이 등록을 하고

```java
@Autowired
@MainDiscountPolicy
DiscountPolicy rateDiscountPolicy

```

위처럼 사용할 수도 있다.


# 조회할 빈이 모두 필요할 때 List, Map

사용자가 Rate, Fix를 선택할 수 있을 때는 모두 불러와야한다.

```
Map<String, DiscountPolicy>`
List<DiscountPolicy>` : `DiscountPolicy
```

위처럼 해결할 수 있다.

```java
    @Test
    void findAllBean(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);

        DiscountService discountService = ac.getBean(DiscountService.class);
        Member member = new Member(1L, "userA", Grade.VIP);
        int discountPrice = discountService.discount(member, 10000, "fixDiscountPolicy");
        Assertions.assertThat(discountService).isInstanceOf(DiscountService.class);
        Assertions.assertThat(discountPrice).isEqualTo(1000);
    }

    static class DiscountService {
        private final Map<String, DiscountPolicy> policyMap;
        private final List<DiscountPolicy> policies;

        @Autowired
        public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policies) {
            this.policyMap = policyMap;
            this.policies = policies;
            System.out.println("policyMap = " + policyMap + ", policies = " + policies);
        }

        public int discount(Member member, int price, String discountCode){
            DiscountPolicy discountPolicy = policyMap.get(discountCode);
            return discountPolicy.discount(member, price);
        }
    }
```

위와 같이 모든 빈들을 불러온 뒤
그 키값에 따라 원하는 빈을 불러올 수 있다.


# 자동, 수동의 올바른 실무 운영 기준

요즘은 자동으로 넘어가고 있는 추세이지만 
수동이 필요한 경우도 있다.




애플리케이션은 크게 업무 로직과 기술 지원 로직으로 나눌 수 있다.


- **업무 로직 빈:** 웹을 지원하는 컨트롤러, 핵심 비즈니스 로직이 있는 서비스, 데이터 계층의 로직을 처리하는 리포 지토리등이 모두 업무 로직이다. 보통 비즈니스 요구사항을 개발할 때 추가되거나 변경된다.
- **기술 지원 빈:** 기술적인 문제나 공통 관심사(AOP)를 처리할 때 주로 사용된다. 데이터베이스 연결이나, 공통 로그 처리 처럼 업무 로직을 지원하기 위한 하부 기술이나 공통 기술들이다.

- 업무 로직은 숫자도 매우 많고, 한번 개발해야 하면 컨트롤러, 서비스, 리포지토리 처럼 어느정도 유사한 패턴이 있다. 이런 경우 자동 기능을 적극 사용하는 것이 좋다. 보통 문제가 발생해도 어떤 곳에서 문제가 발생했는지 명 확하게 파악하기 쉽다.
- 기술 지원 로직은 업무 로직과 비교해서 그 수가 매우 적고, 보통 애플리케이션 전반에 걸쳐서 광범위하게 영향을 미친다. 그리고 업무 로직은 문제가 발생했을 때 어디가 문제인지 명확하게 잘 드러나지만, 기술 지원 로직은 적용 이 잘 되고 있는지 아닌지 조차 파악하기 어려운 경우가 많다. 그래서 이런 기술 지원 로직들은 가급적 수동 빈 등 록을 사용해서 명확하게 드러내는 것이 좋다.


>**애플리케이션에 광범위하게 영향을 미치는 기술 지원 객체는 수동 빈으로 등록해서 딱! 설정 정보에 바로 나타나게 하는 것이 유지보수 하기 좋다.**


또 **비즈니스 로직 중에서 다형성을 적극 활용할 때**가 있다.
의존관계 자동 주입 - 조회한 빈이 모두 필요할 때, List, Map을 다시 보자.
`DiscountService` 가 의존관계 자동 주입으로 `Map<String, DiscountPolicy>` 에 주입을 받는 상황을 생각 해보자. 여기에 어떤 빈들이 주입될 지, 각 빈들의 이름은 무엇일지 코드만 보고 한번에 쉽게 파악할 수 있을까? 내가 개 발했으니 크게 관계가 없지만, 만약 이 코드를 다른 개발자가 개발해서 나에게 준 것이라면 어떨까?
자동 등록을 사용하고 있기 때문에 파악하려면 여러 코드를 찾아봐야 한다.
        
이런 경우 수동 빈으로 등록하거나 또는 자동으로하면 **특정 패키지에 같이 묶어**두는게 좋다! 핵심은 딱 보고 이해가 되어야 한다!
