스프링에서는 다양한 의존관계 주입 방법들이 존재한다.

# 다양한 의존관계 주입 방법
의존관계 주입은 크게 4가지 방법이 있다.
- 생성자 주입
- 수정자 주입(setter 주입) 
- 필드 주입
- 일반 메서드 주입


## 생성자 주입

```java
@Autowired
     public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy
 discountPolicy) {
         this.memberRepository = memberRepository;
         this.discountPolicy = discountPolicy;
     }
}

```

생성자 주입은 불변, 필수라는 특징을 갖고 있다.
생성자이기 때문에 스프링 빈을 등록하는 단계에서 의존관계 주입도 진행된다.

또한 스프링 빈을 필수적으로 등록하게 되면서 필수라는 특징도 갖게 된다.


## 수정자 주입

```java

@Autowired(required = false)
public void setDiscountPolicy(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
}

```

setter 주입은 가변, 선택이라는 특징을 갖고 있다.

수정자는 생성자와 달리 
1단계인 스프링 빈 등록
2단계인 의존관계 주입
에서 2단계에서 진행된다.

실제로 생성자 주입과 수정자 주입을 동시에 세팅하고 부트를 가동하면 생성자 주입이 먼저 실행된다.

일반적으로 setter이기 때문에 가변적이라는 특징이 있고 required=false를 통해 주입되는 객체가 없을 때도 무시하고 스프링을 가동할 수 있다.

## 필드 주입

필드 주입은 순수 자바면으로 봤을 때 아무것도 할 수 없다.
DI가 없으면 아무것도 할 수 없기 때문에 더미 데이터로 테스트를 하고 싶어도 할 수가 없다.

```java
@Autowired private MemberRepository memberRepository;
@Autowired private DiscountPolicy discountPolicy;
```

spring이 아니면 필드에 접근할 수가 없기 때문이다.

하지만 코드가 간결하다는 장점 때문에 테스트 코드에서는 잘 쓰이기도 한다.


## 일반 메서드 주입

일반적으로 잘 사용되지 않는다.
