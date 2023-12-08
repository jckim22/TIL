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
