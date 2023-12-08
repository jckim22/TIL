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


## Option

```java
    @Test
    void AutowiredOption(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestBean.class);
    }

    static class TestBean {

        //의존관계가 없으니 호출이 안됨
        @Autowired(required = false)
        public void setNoBean1(Member noBean1){
            System.out.println("noBean1 = " + noBean1);;
        }

        //Null이 들어옴
        @Autowired
        public void setNoBean2(@Nullable Member noBean2){
            System.out.println("noBean1 = "+noBean2);
        }

        //Optional.empty가 들어옴
        @Autowired(required = false)
        public void setNoBean3(Optional<Member> noBean3){
            System.out.println("noBean3 = " + noBean3);
        }
    }
```

위와 같이 required = false로 호출을 막거나 @Nullable로 Null처리하거나 Optional<>로 Optioanl로 처리할 수 있다.

### 생성자를 써야하는 이유

순수한 자바 언어의 특성을 살리자 -> 테스트를 위해

#### 1. 불변

한번 설정해놓으면 변경할 수 없다
-> 연극도 첫 배역을 공연 중간에 수정하지는 않는다.

#### 2. 누락

세터를 사용하면 깜빡하고 세터를 설정하지 않거나 호출하지 않아서 누락이 날 수 있다.
생성자를 사용하면 컴파일 단계에서 오류가 발생해 깜빡할 수가 없다.

또한 final 키워드를 사용해서 생성자에서 초기화 되지 않았을 때 컴파일 오류를 낼 수 있다.
