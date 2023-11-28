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
