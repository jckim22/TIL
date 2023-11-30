# 변경된 요구사항
비즈니스로 갑자기 고정값 할인해서 고정률 할인으로 바뀌어야 하는 상황이 왔다.
하지만 객체 지향적 설계를 해놓았기 때문에 걱정이 없다.

```java
public class RateDiscountPolicy implements DiscountPolicy {

    private int discountPercent = 10;

    @Override
    public int discount(Member member, int price) {
        if (member.getGrade() == Grade.VIP) {
            return price * discountPercent / 100;
        } else {
            return 0;
        }
    }
}

```
위처럼 할인이라는 역할(인터페이스)의 새로운 구현체인 RateDiscountPolicy를 만들었다.

```java
    @Test
    @DisplayName("VIP는 10% 할인이 적용되어야 한다.")
    void vip_o() {
        //given
        Member member = new Member(1L, "memberVIP", Grade.VIP);
        //when
        int discount = discountPolicy.discount(member, 10000);
        //then
        assertThat(discount).isEqualTo(1000);
    }

    @Test
    @DisplayName("VIP가 아니면 할인이 적용되지 않아야 한다")
    void vip_x(){
        //given
        Member member = new Member(2L, "memberBASIC", Grade.BASIC);
        //when
        int discount = discountPolicy.discount(member, 10000);
        //then
        assertThat(discount).isEqualTo(0);
    }
```
위와 같은 테스트도 통과했다.

>assertions는 static import가 좋다


# 문제점 
하지만 이를 적용하기 위해서 나는 service로직의 코드를 직접 건들여야했다.
아래처럼 말이다.

```java
    private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
//    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
```

이런식으로 다형성을 이용하긴 했지만 서비스에서 직접 구현체를 의존하게 되기 때문에 서비스 로직도 손봐줘야했다.

- 우리는 역할과 구현을 충실하게 분리했다. OK
- 다형성도 활용하고, 인터페이스와 구현 객체를 분리했다. OK OCP, DIP 같은 객체지향 설계 원칙을 충실히 준수했다
   - 그렇게 보이지만 사실은 아니다.
- DIP: 주문서비스 클라이언트( `OrderServiceImpl` )는 `DiscountPolicy` 인터페이스에 의존하면서 DIP를 지킨 것 같은데?
   - 클래스 의존관계를 분석해 보자.
   - 추상(인터페이스) 뿐만 아니라 **구체(구현) 클래스에도 의존**하고 있다.
   - 추상(인터페이스) 의존: `DiscountPolicy`
   - 구체(구현) 클래스: `FixDiscountPolicy` , `RateDiscountPolicy`
- OCP: 변경하지 않고 확장할 수 있다고 했는데!
> **지금 코드는 기능을 확장해서 변경하면, 클라이언트 코드에 영향을 준다!** 따라서 **OCP를 위반**한다.

# 실제 의존 관계

![](https://velog.velcdn.com/images/jckim22/post/53eaa055-f062-411c-bb86-076771b5442e/image.png)

이런식으로 추상화에 의존해야한다는 DIP 원칙을 어긴게 되어 버린다.
이것을 어겼기 때문에 결국 서비스 코드도 함께 변경해야하기 때문에 자동으로 OCP도 위반하게 된다.

지금 이 상황은 기름차에서 전기차로 바꿨을 때 운전면허증도 그에 맞게 새로 발급받은 상황과 비슷한 것이다.

기름차에서 전기차로 바꾸더라도 나는 그냥 차만 갈아타면 되야한다.

```java
public class OrderServiceImpl implements OrderService{
    private final MemberRepository memberRepository = new MemoryMemberRepository();
    private DiscountPolicy discountPolicy;
```

위처럼 하면 될 거 같다.

근데 당연하게도 NULL Pointer Exception이 발생할 것이다.
어떻게 하면 구현체 없이 돌아가게 할 수 있을까 ?

>누군가가 구현체를 밖에서 대신 주입을 해줘야한다. (Dependency Injection)


# 해결



## 관심사의 분리
- 애플리케이션을 하나의 공연이라 생각해보자.
- 각각의 인터페이스를 배역(배우 역할)이라 생각하자.
- 그런데! 실제 배역 맞는 배우를 선택하는 것은 누가 하는가?
   - 로미오와 줄리엣 공연을 하면 로미오 역할을 누가 할지 줄리엣 역할을 누가 할지는 배우들이 정하는게 아니다. 
   - 이 전 코드는 마치 로미오 역할(인터페이스)을 하는 레오나르도 디카프리오(구현체, 배우)가 줄리엣 역할(인터페이스)을 하는 여자 주인공(구현체, 배우)을 직접 초빙하는 것과 같다. 
   - 디카프리오는 공연도 해야하고 동시에 여자 주 인공도 공연에 직접 초빙해야 하는 **다양한 책임**을 가지고 있다.
   
**관심사를 분리하자**
- 배우는 본인의 역할인 배역을 수행하는 것에만 집중해야 한다.
- 디카프리오는 어떤 여자 주인공이 선택되더라도 똑같이 공연을 할 수 있어야 한다.
- 공연을 구성하고, 담당 배우를 섭외하고, 역할에 맞는 배우를 지정하는 책임을 담당하는 별도의 **공연 기획자**가 나 올시점이다.
- 공연 기획자를 만들고, 배우와 공연 기획자의 책임을 확실히 분리하자.

```java
public class AppConfig {

    public MemberService memberService() {
        return new MemberServiceImpl(new MemoryMemberRepository());
    }

    public OrderService orderService() {
        return new OrderServiceImpl(new MemoryMemberRepository(), new RateDiscountPolicy());
    }
}
```

기획자 역할을 하는 AppConfig를 만들어서 기존에 인터페이스 구현체에게 사용하고픈 객체를 넣어주었다.

이로서 디카프리오는 연기에만 집중하면 되게 되었다.

### OrderServiceImpl

```java
public class OrderServiceImpl implements OrderService{
    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;
//    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();

    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }

```


- 설계 변경으로 `OrderServiceImpl` 은 `FixDiscountPolicy` 를 의존하지 않는다! 단지 `DiscountPolicy` 인터페이스만 의존한다.
- `OrderServiceImpl` 입장에서 생성자를 통해 어떤 구현 객체가 들어올지(주입될지)는 알 수 없다.
- `OrderServiceImpl` 의 생성자를 통해서 어떤 구현 객체을 주입할지는 오직 외부( `AppConfig` )에서 결정한다.
- `OrderServiceImpl` 은 이제부터 실행에만 집중하면 된다.
- `OrderServiceImpl` 에는 `MemoryMemberRepository`,` FixDiscountPolicy` 객체의 의존관계가 주 입된다.


```java
class OrderServiceTest {

    MemberService memberService;
    OrderService orderService;

    @BeforeEach
    public void beforeEach() {
        AppConfig appConfig = new AppConfig();
        memberService = appConfig.memberService();
        orderService = appConfig.orderService();
    }
```

위처럼 구현체의존하지 않는 테스트 코드 또한 만들 수 있다.

# **정리**
- AppConfig를 통해서 관심사를 확실하게 분리했다.
- 배역, 배우를 생각해보자.
- AppConfig는 공연 기획자다.
- AppConfig는 구체 클래스를 선택한다.
- 배역에 맞는 담당 배우를 선택한다. 애플리케이션이 어떻게 동작해야 할 지 전체 구성을 책임진다.
- 이제 각 배우들은 담당 기능을 실행하는 책임만 지면 된다.
- `OrderServiceImpl` 은 기능을 실행하는 책임만 지면 된다.

## 리팩토링

그리고 아래와 같이 리팩토링을 진행함으로서 
```java
public class AppConfig {

    public MemberService memberService(){
        return new MemberServiceImpl(memberRepository());
    }

    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    public DiscountPolicy discountPolicy(){
        return new RateDiscountPolicy();
    }
}
```
아래와 같은 확실한 구조가 생겼다.
뭔가 수정할 때 우리는 구성 영역(기획자)만 건드리면 된다.

![업로드중..](blob:https://velog.io/a37968fb-d4c8-40cb-8d51-5ce47f108f82)

## 전체 흐름


- 지금까지의 흐름을 정리해보자.
- 새로운 할인 정책 개발
- 새로운 할인 정책 적용과 문제점 관심사의 분리
- AppConfig 리팩터링
- 새로운 구조와 할인 정책 적용
**새로운 할인 정책 개발**
>다형성 덕분에 새로운 정률 할인 정책 코드를 추가로 개발하는 것 자체는 아무 문제가 없음

**새로운 할인 정책 적용과 문제점**
>새로 개발한 정률 할인 정책을 적용하려고 하니 **클라이언트 코드**인 주문 서비스 구현체도 함께 변경해야함
주문 서비스 클라이언트가 인터페이스인 `DiscountPolicy` 뿐만 아니라, 구체 클래스인 `FixDiscountPolicy` 도 함께 의존 **DIP 위반**
	
**관심사의 분리**
- 애플리케이션을 하나의 공연으로 생각
- 기존에는 클라이언트가 의존하는 서버 구현 객체를 직접 생성하고, 실행함
비유를 하면 기존에는 남자 주인공 배우가 공연도 하고, 동시에 여자 주인공도 직접 초빙하는 다양한 책임을 가지 고 있음
- 공연을 구성하고, 담당 배우를 섭외하고, 지정하는 책임을 담당하는 별도의 **공연 기획자**가 나올 시점
- 공연 기획자인 AppConfig가 등장
- AppConfig는 애플리케이션의 전체 동작 방식을 구성(config)하기 위해, **구현 객체를 생성**하고, **연결**하는 책임 이제부터 클라이언트 객체는 자신의 역할을 실행하는 것만 집중, 권한이 줄어듬(책임이 명확해짐)

**AppConfig 리팩터링**
- 구성 정보에서 역할과 구현을 명확하게 분리 역할이 잘 드러남
- 중복 제거

**새로운 구조와 할인 정책 적용**
- 정액 할인 정책 정률% 할인 정책으로 변경
- AppConfig의 등장으로 애플리케이션이 크게 **사용 영역**과, 객체를 생성하고 **구성(Configuration)하는 영역**으 로 분리
- 할인 정책을 변경해도 AppConfig가 있는 구성 영역만 변경하면 됨, 사용 영역은 변경할 필요가 없음. 물론 클라 이언트 코드인 주문 서비스 코드도 변경하지 않음


