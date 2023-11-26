# 비즈니스 요구사항과 설계
- 회원
   - 회원을 가입하고 조회할 수 있다.
   - 회원은 일반과 VIP 두 가지 등급이 있다.
   - 회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다. (미확정)
-  주문과 할인 정책
   - 회원은 상품을 주문할 수 있다.
   - 회원 등급에 따라 할인 정책을 적용할 수 있다.
   - 할인 정책은 모든 VIP는 1000원을 할인해주는 고정 금액 할인을 적용해달라. (나중에 변경 될 수 있다.) 
   - 할인 정책은 변경 가능성이 높다. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 고민을 미루 고 싶다. 최악의 경우 할인을 적용하지 않을 수 도 있다. (미확정)


> 김영한: 요구사항을 보면 회원 데이터, 할인 정책 같은 부분은 지금 결정하기 어려운 부분이다. 그렇다고 이런 정책이 결정될 때
까지 개발을 무기한 기다릴 수 도 없다. 우리는 앞에서 배운 객체 지향 설계 방법이 있지 않은가! 인터페이스를 만들고 구현체를 언제든지 갈아끼울 수 있도록 설계하면 된다.

# 회원 도메인 설계
회원 도메인 요구사항
회원을 가입하고 조회할 수 있다.
회원은 일반과 VIP 두 가지 등급이 있다.
회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다. (미확정)

**회원 도메인 협력 관계**

![](https://velog.velcdn.com/images/jckim22/post/947729e1-fd24-41e7-955a-c6aafb1b78c2/image.png)
아직 DB 저장소와 외부 시스템 연동 저장소는 정해지지 않았다.



**회원 클래스 다이어그램**
![](https://velog.velcdn.com/images/jckim22/post/11c1b0eb-fec3-444b-a107-d661525a3654/image.png)
실제 구현은 이런 식으로 인터페이스를 구현한 뒤 임플리먼츠를 할 예정이다.


**회원 객체 다이어그램**


![](https://velog.velcdn.com/images/jckim22/post/31956732-efe6-4c69-8044-32fe4f0befc7/image.png)
>회원 서비스: **MemberServiceImpl**

실제 객체는 이런식으로 구현체를 참조한다.

# 주문과 할인 정책

![](https://velog.velcdn.com/images/jckim22/post/e5470eee-e1be-4319-b399-af5a1ed9d718/image.png)
![](https://velog.velcdn.com/images/jckim22/post/a7e801b8-de64-49ba-ae0b-7731fc4c9b65/image.png)
![](https://velog.velcdn.com/images/jckim22/post/a500b788-9a5e-4ad7-8431-a5fd7712f69a/image.png)

**역할과 구현을 분리**해서 자유롭게 구현 객체를 조립할 수 있게 설계했다. 덕분에 회원 저장소는 물론이고, 할인 정책도 유연하게 변경할 수 있다.


![](https://velog.velcdn.com/images/jckim22/post/eec4c349-f808-4978-a8c1-9741c0639b24/image.png)

회원을 메모리에서 조회하고, 정액 할인 정책(고정 금액)을 지원해도 주문 서비스를 변경하지 않아도 된다. 역할들의 협력 관계를 그대로 재사용 할 수 있다.

![](https://velog.velcdn.com/images/jckim22/post/b3be824a-0600-4f0f-a842-412f8d68cc0c/image.png)


회원을 메모리가 아닌 실제 DB에서 조회하고, 정률 할인 정책(주문 금액에 따라 % 할인)을 지원해도 주문 서비스를 변경하지 않아도 된다.
협력 관계를 그대로 재사용 할 수 있다.

# 구현

![](https://velog.velcdn.com/images/jckim22/post/b3a2f859-d018-47c4-9f41-244cac92ee1a/image.png)
실제 완성한 구조이다.

OrderService라는 역할의 구현체만 보겠다.

```java
public class OrderServiceImpl implements OrderService{
    private final MemberRepository memberRepository = new MemoryMemberRepository();
    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
    @Override
    public Order createOrder(Long memberId, String itemName, int itemPrice) {
        Member member =memberRepository.findById(memberId);
        int discountPrice = discountPolicy.discount(member, itemPrice);

        return new Order(memberId, itemName, itemPrice, discountPrice);
    }
}

```
보게 되면 위 구조처럼 OrderService의 구현체는 MemoryMemberRepo와 FixDiscountPolicy를 의존하고 있다.
할인에는 전혀 관여하지 않고 할인은 오직 discountPolicy에게 맡기면서 SOLID의 SRP(단일 책임 원칙)을 적용하는데 성공했다.

하지만 MemoryMemberRepository와 FixDiscountPolicy를 Service에서 직접 의존하게 되면서 문제가 생긴다.
저 둘에 구현체가 바뀌게 되면 Service도 손 봐줘야하기 때문이다.

이렇게 되면 SOLID에서 OCP(개방-폐쇄-원칙)과 DIP(의존관계 역전 원칙)에서 문제가 생긴다.

이는 앞서 입문편에서 배웠던 DI 컨테이너 기술로 해결할 수 있다.
오직 하나의 컴포넌트를 싱글톤으로 인젝션 해주면서 이를 해결할 수 있다.

이에 대해서는 이후에 더 알아보자
