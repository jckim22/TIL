# AOP 적용
AOP: Aspect Oriented Programming
공통 관심 사항(cross-cutting concern) vs 핵심 관심 사항(core concern) 분리

- 회원가입, 회원 조회등 핵심 관심사항과 시간을 측정하는 공통 관심 사항을 분리한다. 시간을 측정하는 로직을 별도의 공통 로직으로 만들었다.
- 핵심 관심 사항을 깔끔하게 유지할 수 있다.
- 변경이 필요하면 이 로직만 변경하면 된다.
- 원하는 적용 대상을 선택할 수 있다.
![](https://velog.velcdn.com/images/jckim22/post/0e4a3710-788c-47af-9e93-134b1ecddff0/image.png)


```java
@Aspect
@Component
public class TimeTraceAop {
    @Around("execution(* hello.hellospring..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        System.out.println("START: " + joinPoint.toString());
        try {
            return joinPoint.proceed();
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END: " + joinPoint.toString() + " " + timeMs + "ms");
        }
    }
}
```

위처럼 새로운 Aop Class르 작성해주면 된다.
>- Aspect : 위에서 설명한 흩어진 관심사를 모듈화 한 것. 주로 부가기능을 모듈화함.
- Target : Aspect를 적용하는 곳 (클래스, 메서드 .. )
- Advice : 실질적으로 어떤 일을 해야할 지에 대한 것, 실질적인 부가기능을 담은 구현체
- JointPoint : Advice가 적용될 위치, 끼어들 수 있는 지점. 메서드 진입 지점, 생성자 호출 시점, 필드에서 값을 꺼내올 때 등 다양한 시점에 적용가능
- PointCut : JointPoint의 상세한 스펙을 정의한 것. 'A란 메서드의 진입 시점에 호출할 것'과 같이 더욱 구체적으로 Advice가 실행될 지점을 정할 수 있

@Component 어노테이션으로 컴포넌트 스캔을 해도 되지만 SpringConfig에서 @Bean으로 스프링 빈으로 등록해놔도 상관 없다.
> 대부분 실무에선 후자가 쓰인다.


# AOP 적용 전

![](https://velog.velcdn.com/images/jckim22/post/d5fe692f-986b-47a8-ba2b-a6872abbaf31/image.png)


# AOP 적용 후

![](https://velog.velcdn.com/images/jckim22/post/b10951c6-d5c2-4403-893f-588414887859/image.png)

AOP는 실제 서버에 코드를 주입하는 형식이 아니고 프록시 기술을 사용해서 가짜 클래스를 불러온다.

그리고 JoinPoint.proceed() 이후에 진짜 클래스를 호출한다.

>DI 구조이기 때문에 위 같은 형식으로 AOP 적용이 가능하다.
