지금까지의 코드를 보면 100번의 요청이 들어오면 100다 instance를 생성해주었다.
메모리 낭비가 매우 심해보인다.

그래서 싱글톤을 적용한다.

아래를 보자


```java
    @Test
    @DisplayName("스프링 없는 순수한 DI 컨테이너")
    void pureContainer(){
        AppConfig appConfig = new AppConfig();
        //1. 조회: 호출할 때 마다 객체를 생성
        MemberService memberService1 = appConfig.memberService();

        //2. 조회: 호출할 때 마다 객체를 생성
        MemberService memberService2 = appConfig.memberService();

        //참조값이 다른 것을 확인
        System.out.println("memberService1 = "+ memberService1);
        System.out.println("memberService2 = "+ memberService2);

        assertThat(memberService1).isNotSameAs(memberService2);
```

위와 같은 테스트를 하게 되면 당연히 통과한다.
두 memberService는 다른 인스턴스일테니까
>issameAs -> 주소값비교 (깊은비교)
isEqualto -> 내용비교 (얕은비교)
isInstanceOf -> 그 클래스의 하위 인스턴스인지 확인

그럼 아래처럼 싱글톤 패턴을 구현해보자
```java
public class SingletonService {

    private static final SingletonService instance = new SingletonService();

    public static SingletonService getInstance() {
        return instance;
    }

    private SingletonService(){

    }

    public void logic(){

    }
}
```

위처럼 구현하고 아래와 같은 테스트를 진행하면


```java
    @Test
    @DisplayName("싱글톤 패턴을 적용한 객체 사용")
    void singletonServiceTest(){
        SingletonService singletonService1 = SingletonService.getInstance();
        SingletonService singletonService2 = SingletonService.getInstance();

        System.out.println("singletonService1 = "+singletonService1);
        System.out.println("singletonService2 = "+singletonService2);

        assertThat(singletonService1).isSameAs(singletonService2);
    }
```

getInstance로 같은 인스턴스를 불러오기 때문에 테스트가 통과될 것이다.

하지만 이러한 싱글톤에는 많은 문제가 있다.

**싱글톤 패턴 문제점**
- 싱글톤 패턴을 구현하는 코드 자체가 많이 들어간다.
- 의존관계상 클라이언트가 구체 클래스에 의존한다. DIP를 위반한다. 
- 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다. 테스트하기 어렵다.
- 내부 속성을 변경하거나 초기화 하기 어렵다.
- private 생성자로 자식 클래스를 만들기 어렵다.
- 결론적으로 유연성이 떨어진다.
- 안티패턴으로 불리기도 한다.


# 싱글톤 컨테이너
스프링 컨테이너는 싱글톤 패턴의 문제점을 해결하면서, 객체 인스턴스를 싱글톤(1개만 생성)으로 관리한다. 지금까지 우리가 학습한 스프링 빈이 바로 싱글톤으로 관리되는 빈이다.

```java
    void springContainer(){
//        AppConfig appConfig = new AppConfig();
        ApplicationContext ac =new AnnotationConfigApplicationContext(AppConfig.class);

        //1. 조회: 호출할 때 마다 객체를 생성
        MemberService memberService1 = ac.getBean("memberService", MemberService.class);

        //2. 조회: 호출할 때 마다 객체를 생성
        MemberService memberService2 = ac.getBean("memberService", MemberService.class);

        //참조값이 다른 것을 확인
        System.out.println("memberService1 = "+ memberService1);
        System.out.println("memberService2 = "+ memberService2);

        assertThat(memberService1).isSameAs(memberService2);

    }
```

위 테스트는 통과이다.
스프링 빈은 같은 인스턴스를 사용하는 싱글톤 컨테이너에 속해있으니까

이로써 싱글톤의 많은 문제들이 해결이 되었다.

- 스프링 컨테이너는 싱글턴 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리한다.
   - 이전에 설명한 컨테이너 생성 과정을 자세히 보자. 컨테이너는 객체를 하나만 생성해서 관리한다.
- 스프링 컨테이너는 싱글톤 컨테이너 역할을 한다. 이렇게 싱글톤 객체를 생성하고 관리하는 기능을 싱글톤 레지스트리라 한다.
- 스프링 컨테이너의 이런 기능 덕분에 싱글턴 패턴의 모든 단점을 해결하면서 객체를 싱글톤으로 유지할 수 있다.
   - 싱글톤 패턴을 위한 지저분한 코드가 들어가지 않아도 된다.
   - DIP, OCP, 테스트, private 생성자로 부터 자유롭게 싱글톤을 사용할 수 있다.
   
   
**싱글톤 컨테이너 적용 후**

![](https://velog.velcdn.com/images/jckim22/post/2ce0ecd3-8c80-4a3b-8ec2-33e89ca441f9/image.png)


- 스프링 컨테이너 덕분에 고객의 요청이 올 때 마다 객체를 생성하는 것이 아니라, 이미 만들어진 객체를 공유해서 효율적으로 재사용할 수 있다.

# 싱글톤 방식의 주의점

- 싱글톤 패턴이든, 스프링 같은 싱글톤 컨테이너를 사용하든, 객체 인스턴스를 하나만 생성해서 공유하는 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에 싱글톤 객체는 상태를 유지(stateful)하 게 설계하면 안된다.
- 무상태(stateless)로 설계해야 한다!
   - 특정 클라이언트에 의존적인 필드가 있으면 안된다.
   - 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다!
   - 가급적 읽기만 가능해야 한다.
   - 필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다.
- 스프링 빈의 필드에 공유 값을 설정하면 정말 큰 장애가 발생할 수 있다!!!

```java
public class StatefulService {

    private int price; // 상태를 유지하는 필드

    public void order(String name, int price){
        System.out.println("name = "+ name + " price = " + price);
        this.price = price;
    }

    public int getPrice(){
        return price;
    }
}
```

위 서비스는 price를 상태 정보로 계속 갖고 있다.

```java
    @Test
    void statefulServiceSingleton(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

        //ThreadA: A사용자 10000원 주문
        statefulService1.order("userA", 10000);;
        //ThreadA: B사용자 20000원 주문
        statefulService2.order("userB", 20000);;

        //Thread: 사용자 A 주문 금액조회
        int price = statefulService1.getPrice();
        System.out.println("price = "+ price);

        Assertions.assertThat(statefulService1.getPrice()).isEqualTo(20000);
    }
```

그래서 위와 같은 상황에서 문제가 생길 수 있다.
사용자 A가 만원을 주문했는데 B가 2만원을 주문함으로서 상태성이 유지되는 변수의 값이 변해버린 것이다.


```java
public class StatefulService {

    public int order(String name, int price){
        System.out.println("name = "+ name + " price = " + price);
        return price;
    }

}
```

위와 같이 수정하여 무상태성으로 코드를 작성하면 이를 해결할 수 있다.

# @Configuration

도대체 어떻게 스프링은 싱글톤을 유지하는걸까 ?

```java
@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService(){
        System.out.println("call memberService");
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        System.out.println("call memberRepository");
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        System.out.println("call orderService");
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy(){
        return new RateDiscountPolicy();
    }
}
```

위 AppConfig의 자바 코드만 보면 계속해서 객체를 생성해야할 것만 같다.

비밀은 @Configuration에 있다.


![](https://velog.velcdn.com/images/jckim22/post/7f54c8db-7007-4b6a-a2d2-1b1e01441091/image.png)

위와 같이 스프링에서는 AppConfig@CGLIB라는 또 다른 자식 클래스를 생성한다.

>
그 임의의 다른 클래스가 바로 싱글톤이 보장되도록 해준다. 아마도 다음과 같이 바이트 코드를 조작해서 작성되어 있을 것이다.(실제로는 CGLIB의 내부 기술을 사용하는데 매우 복잡하다.)

### CGLIB 예상 코드

```java
 @Bean
 public MemberRepository memberRepository() {
if (memoryMemberRepository가 이미 스프링 컨테이너에 등록되어 있으면?) { return 스프링 컨테이너에서 찾아서 반환;
} else { //스프링 컨테이너에 없으면
기존 로직을 호출해서 MemoryMemberRepository를 생성하고 스프링 컨테이너에 등록 return 반환
}
}
```
훨씬 복잡하겠지만, 간단하게 원리를 예상해보면 위와 같은 로직으로 짜여져 있을 것이고 이것이 싱글톤을 지켜준다.
