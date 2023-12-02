# 스프링 컨테이너 생성
```java
//스프링 컨테이너 생성
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
```
위와 같이 스프링 컨테이너를 생성한다.
>스프링 컨테이너는 BeanFactory, ApplicationContext로 크게 나눠지는데 여기서는 ApllicationContext가 스프링 컨테이너라고 생각하면 된다.

ApplicationContext는 인터페이스이고 그 구현체 중 하나인 AnnotationConfigApplicationContext를 사용한다.

>XML도 사용하긴하지만 요즘은 다 어노테이션 기반으로 사용한다.

# 스프링 컨테이너의 생성 과정
**1. 스프링 컨테이너 생성**
![](https://velog.velcdn.com/images/jckim22/post/ad6fcbea-fc21-498a-9d00-be5c7f2a1879/image.png)

- `new AnnotationConfigApplicationContext(AppConfig.class)`
- 스프링 컨테이너를 생성할 때는 구성 정보를 지정해주어야 한다.
- 여기서는 `AppConfig.class` 를 구성 정보로 지정했다.


**2. 스프링 빈 등록**

![](https://velog.velcdn.com/images/jckim22/post/e603d8f3-ca38-4c71-b928-548ac30aee8b/image.png)

**3. 스프링 빈 의존관계 설정 - 준비**

![](https://velog.velcdn.com/images/jckim22/post/3c62dee9-3dce-4748-b777-32a8ad337284/image.png)

- 스프링 컨테이너는 파라미터로 넘어온 설정 클래스 정보를 사용해서 스프링 빈을 등록한다. 

**빈 이름**
- 빈 이름은 메서드 이름을 사용한다. 빈 이름을 직접 부여할 수 도 있다.
`@Bean(name="memberService2")`

**4. 스프링 빈 의존관계 설정 - 완료**

![](https://velog.velcdn.com/images/jckim22/post/84498fd0-6eca-4583-9194-38548bb70420/image.png)


스프링 컨테이너는 설정 정보를 참고해서 의존관계를 주입(DI)한다.
단순히 자바 코드를 호출하는 것 같지만, 차이가 있다. 이 차이는 뒤에 싱글톤 컨테이너에서 설명한다.

>스프링은 빈을 생성하고, 의존관계를 주입하는 단계가 나누어져 있다. 그런데 이렇게 자바 코드로 스프링 빈을 등록하면 생성자를 호출하면서 의존관계 주입도 한번에 처리된다. 여기서는 이해를 돕기 위해 개념적으로 나누어 설명했다. 자세 한 내용은 의존관계 자동 주입에서 다시 설명하겠다.


**정리**
스프링 컨테이너를 생성하고, 설정(구성) 정보를 참고해서 스프링 빈도 등록하고, 의존관계도 설정했다. 이제 스프링 컨테이너에서 데이터를 조회해보자.


```java
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("모든 빈 출력하기")
    void findAllBean() {
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            Object bean = ac.getBean(beanDefinitionName);
            System.out.println("name = " + beanDefinitionName + " object = " + bean);
        }
    }

    @Test
    @DisplayName("모든 빈 출력하기")
    void findApplicationBean() {
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);

            if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
                Object bean = ac.getBean(beanDefinitionName);
                System.out.println("name = " + beanDefinitionName + " object = " + bean);
            }
        }
    }
```

#### 모든 빈 출력하기
	
- 실행하면 스프링에 등록된 모든 빈 정보를 출력할 수 있다. 
- `ac.getBeanDefinitionNames()` : 스프링에 등록된 모든 빈 이름을 
- 조회한다. `ac.getBean()` : 빈 이름으로 빈 객체(인스턴스)를 조회한다.

#### 애플리케이션 빈 출력하기
스프링이 내부에서 사용하는 빈은 제외하고, 내가 등록한 빈만 출력해보자.

- 스프링이 내부에서 사용하는 빈은 `getRole()` 로 구분할 수 있다.
- `ROLE_APPLICATION` : 일반적으로 사용자가 정의한 빈 
- `ROLE_INFRASTRUCTURE` : 스프링이 내부에서 사용하는 빈

>출력하게 되면 등록했던 스프링 빈에 대한 정보들이 나온다.


### 빈 조회하기 - 기본

```java
public class ApplicationContextBasicFindTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("빈 이름으로 조회")
    void findBeanByName(){
        MemberService memberService = ac.getBean("memberService", MemberService.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("이름 없이 타입으로만 조회")
    void findBeanByType(){
        MemberService memberService = ac.getBean(MemberService.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("구현 타입으로 조화")
    void findBeanByName2(){
        MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("빈 이름으로 조회X")
    void findBeanByNameX(){
        ac.getBean("xxxxx", MemberService.class);
        assertThrows(NoSuchBeanDefinitionException.class,
                () -> ac.getBean("xxxxx", MemberService.class));
    }
}

```

>이름, 구현 타입으로 조회할 수 있다.

### 빈 조회하기 같은 타입 중복

```java
    @Test
    @DisplayName("타입으로 조회시 같은 타입이 둘 이상 있으면, 중복 오류가 발생한다")
    void findBeanByTypeDuplicate(){
        assertThrows(NoUniqueBeanDefinitionException.class,
                () -> ac.getBean(MemberRepository.class));
    }


    @Test
    @DisplayName("타입으로 조회시 같은 타입이 둘 이상 있으면, 빈 이름을 지정하면 된다")
    void findBeanByName(){
        MemberRepository memberRepository = ac.getBean("memberRepository1", MemberRepository.class);
        assertThat(memberRepository).isInstanceOf(MemberRepository.class);
    }

    @Test
    @DisplayName("특정 타입을 모두 조회하기")
    void findAllBeanByType(){
        Map<String, MemberRepository> beansOfType = ac.getBeansOfType(MemberRepository.class);
        for(String key : beansOfType.keySet()){
            System.out.println("key = " + key + " value = "+ beansOfType.get(key));
        }
        System.out.println("beansOfType = " + beansOfType);
    }

    @Configuration
    static class SameBeanConfig{
        @Bean
        public MemberRepository memberRepository(){
            return new MemoryMemberRepository();
        }

        @Bean
        public MemberRepository memberRepository2(){
            return new MemoryMemberRepository();
        }
    }
```

>같은 타입이 둘 이상이면 타입으로 조회시 중복이 발생하여 NoUniqueBeanDefinitionException이 발생한다.
이를 해결하기 위해 이름으로 조회한다.

### 빈 조회하기 - 부모 클래스 조회

```java
public class ApllicationContextExtendsFindTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("부모 타입으로 조회시, 자식이 둘 이상 있으면, 중복 오류가 발생한다.")
    void findBeanByParentTypeDuplication() {
        assertThrows(NoUniqueBeanDefinitionException.class,
                () -> ac.getBean(DiscountPolicy.class));
    }

    @Test
    @DisplayName("부모 타입으로 조회시, 자식이 둘 이상 있으면, 빈 이름을 지정하면 된다.")
    void findBeanByParentTypeBeanName() {
        DiscountPolicy rateDiscountPolicy = ac.getBean("rateDiscountPolicy", DiscountPolicy.class);
        assertThat(rateDiscountPolicy).isInstanceOf(RateDiscountPolicy.class);
    }

    @Test
    @DisplayName("특정 하위 타입으로 조회")
    void findBeanBySubType() {
        RateDiscountPolicy bean = ac.getBean(RateDiscountPolicy.class);
        assertThat(bean).isInstanceOf(RateDiscountPolicy.class);
    }

    @Test
    @DisplayName("부모 타입으로 모두 조회하기")
    void findAllBeanByParentType() {
        Map<String, DiscountPolicy> beansOfType = ac.getBeansOfType(DiscountPolicy.class);
        assertThat(beansOfType.size()).isEqualTo(2);
        for (String key : beansOfType.keySet()) {
            System.out.println("key = " + key + " value = " + beansOfType.get(key));
        }
    }

    @Test
    @DisplayName("부모 타입으로 모두 조회하기 - Object")
    void findAllBeanByObjectType(){
        Map<String, Object> beansOfType = ac.getBeansOfType(Object.class);
        for(String key : beansOfType.keySet()){
            System.out.println("key = " + key + " value = " + beansOfType.get(key));
        }
    }

    @Configuration
    static class TestConfig {
        @Bean
        public DiscountPolicy rateDiscountPolicy() {
            return new RateDiscountPolicy();
        }

        @Bean
        public DiscountPolicy fixDiscountPolicy() {
            return new FixDiscountPolicy();
        }
    }
 }
```

>부모 타입을 조회하면 그 자식 타입의 빈은 모두 가져와진다.
Object타입은 객체의 조상이기 때문에 Object 타입의 빈을 조회하면 스프링의 모든 빈이 조회된다.

### BeanFactory

빈팩토리는 스프링 컨테이너이지만, 우리가 위에서 사용했던 ApllicationContext는 빈팩토리를 상속 받아서 추가적인 기능을 갖고 있는 이 역시 스프링 컨테이너이다.

![업로드중..](blob:https://velog.io/afb9ef70-3dfc-45f2-963d-6873d386c793)

위처럼 부가적인 기능들을 갖고 있다.





### XML


![업로드중..](blob:https://velog.io/2c55e438-ab84-4968-bcfc-87c8d1b7837a)

위 구조에서 우리는 지금까지 Annotation을 사용했고 현재도 거의 Annotation이 사용되곤 한다.
하지만 GenericXml 기능으로 세팅을 해보는 것도 좋은 경험이 될 수 있다.


```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="memberService" class="hello.core.member.MemberServiceImpl">
        <constructor-arg name="memberRepository" ref="memberRepository" />
    </bean>
    <bean id="memberRepository"
          class="hello.core.member.MemoryMemberRepository" />
    <bean id="orderService" class="hello.core.order.OrderServiceImpl">
        <constructor-arg name="memberRepository" ref="memberRepository" />
        <constructor-arg name="discountPolicy" ref="discountPolicy" />
    </bean>
    <bean id="discountPolicy" class="hello.core.discount.RateDiscountPolicy" />
</beans>
```

```java
    @Test
    void xmlAppContext(){
        ApplicationContext ac = new GenericXmlApplicationContext("XmlAppContext.xml");
        MemberService memberService = ac.getBean("memberService", MemberService.class);
        assertThat(memberService).isInstanceOf(MemberService.class);
    }
```
이처럼 XML로도 스프링 빈을 설정할 수 있는데 위 XML 내용은 AppCOnfig.java와 동일하다.

### BeanDefinition

![업로드중..](blob:https://velog.io/4d2f021f-88bf-48e7-8610-04968b37f603)
![업로드중..](blob:https://velog.io/b28683a8-9f62-4f43-8020-af7440cc48e9)

위처럼 스프링 컨테이너가 BeanDefinition이라는 추상적인 것을 의존하기 때문에 다양한 형태에 설정 정보를 읽어들일 수 있는 것이다.

```java
코드를 입력하세요public class BeanDefinitionTest {
    AnnotationConfigApplicationContext ac = new
            AnnotationConfigApplicationContext(AppConfig.class);

    //    GenericXmlApplicationContext ac = new
//    GenericXmlApplicationContext("appConfig.xml");
    @Test
    @DisplayName("빈 설정 메타정보 확인")
    void findApplicationBean() {
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            BeanDefinition beanDefinition =
                    ac.getBeanDefinition(beanDefinitionName);
            if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
                System.out.println("beanDefinitionName" + beanDefinitionName +
                        " beanDefinition = " + beanDefinition);
            }
        }
    }
}
```
위처럼 Definition을 다 조회할 수 있다.
Object타입은 객체의 조상이기 때문에 Object 타입의 빈을 조회하면 스프링의 모든 빈이 조회된다.
