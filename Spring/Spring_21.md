
# 빈 스코프란?
지금까지 우리는 스프링 빈이 스프링 컨테이너의 시작과 함께 생성되어서 스프링 컨테이너가 종료될 때 까지 유지된다 고 학습했다. 이것은 스프링 빈이 기본적으로 싱글톤 스코프로 생성되기 때문이다. 스코프는 번역 그대로 빈이 존재할 수 있는 범위를 뜻한다.

**스프링은 다음과 같은 다양한 스코프를 지원한다.**

- **싱글톤**: 기본 스코프, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프이다.
- **프로토타입**: 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프이다.
- **웹 관련 스코프**
   - **request**: 웹 요청이 들어오고 나갈때 까지 유지되는 스코프이다.    
   - **session**: 웹 세션이 생성되고 종료될 때 까지 유지되는 스코프이다. 
   -   **application**: 웹의 서블릿 컨텍스트와 같은 범위로 유지되는 스코프이다.
   
빈 스코프는 다음과 같이 지정할 수 있다. 

**컴포넌트 스캔 자동 등록**
```java
 @Scope("prototype")
  
  @Component
 public class HelloBean {}
```
**수동 등록** 
```java
 @Scope("prototype")
 @Bean
 PrototypeBean HelloBean() {
     return new HelloBean();
 }
```
지금까지 싱글톤 스코프를 계속 사용해보았으니, 프로토타입 스코프부터 확인해보자.



# 프로토타입 빈

### 싱글톤
![](https://velog.velcdn.com/images/jckim22/post/ca5a9430-34ef-4c49-8aeb-90a55d6f202c/image.png)

스프링 빈은 초기화 단계까지 끝나고도 계속 컨테이너에서 관리하고 호출할 때마다 같은 빈을 반환한다.

### 프로토타입 

![](https://velog.velcdn.com/images/jckim22/post/1f4bab16-3130-4050-b0bd-3d813db009f2/image.png)

하지만 프로토타입에서는 초기화까지만 마치고 이후에는 관여하지 않는다.
이후 이 객체에 대한 책임은 클라이언트에게 있다.


```java
public class SingletonTest {
    @Test
    void singletonBeanFind(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SingletonBean.class);

        SingletonBean bean1 = ac.getBean(SingletonBean.class);
        SingletonBean bean2 = ac.getBean(SingletonBean.class);
        System.out.println("bean1 = " + bean1);
        System.out.println("bean2 = " + bean2);

        Assertions.assertThat(bean1).isSameAs(bean2);

        ac.close();
    }

    @Scope("singleton")
    @Configuration
    static class SingletonBean{
        @PostConstruct
        public void init(){
            System.out.println("SingletonBean.init");
        }

        @PreDestroy
        public void destroy(){
            System.out.println("SingletonBean.destroy");
        }
    }
}

```

위에 싱글톤 테스트를 진행하고 실행결과를 살펴보자
![](https://velog.velcdn.com/images/jckim22/post/112da8d5-0dda-42be-be46-a7d26858495c/image.png)

위처럼 같은 객체임을 확인할 수 있고 컨테이너를 close할 때 컨테이너가 끝까지 관리하기 때문에 destroy가 실행된 것을 볼 수 있다.


```java
public class PrototypeTest {

    @Test
    void prototypeBeanFind(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);

        PrototypeBean bean1 = ac.getBean(PrototypeBean.class);
        PrototypeBean bean2 = ac.getBean(PrototypeBean.class);
        System.out.println("bean1 = " + bean1);
        System.out.println("bean2 = " + bean2);

        Assertions.assertThat(bean1).isNotSameAs(bean2);

        ac.close();
    }
    @Scope("prototype")
    @Configuration
    static class PrototypeBean{
        @PostConstruct
        public void init(){
            System.out.println("PrototypeBean.init");
        }

        @PreDestroy
        public void destroy(){
            System.out.println("PrototypeBean.destroy");
        }
    }
}
```

![](https://velog.velcdn.com/images/jckim22/post/b29a849a-e96a-44bf-847a-8d72c8e5b395/image.png)

프로토타입에서는 초기화까지는 되지만 서로 다른 객체임을 확인할 수 있고 두 객체이기 때문에 초기화도 두번 진행된다.
그리고 초기화 이후 컨테이너에서 관리하지 않기 때문에 close를 해도 destroy가 실행되지 않는다.

# 문제점

![](https://velog.velcdn.com/images/jckim22/post/7a20227b-518e-48b8-a73f-9e7bf82c647b/image.png)

- 클라이언트 B는 `clientBean` 을 스프링 컨테이너에 요청해서 받는다.싱글톤이므로 항상 같은 `clientBean` 이 반환된다.
- **여기서 중요한 점이 있는데, clientBean이 내부에 가지고 있는 프로토타입 빈은 이미 과거에 주입이 끝난 빈이 다. 주입 시점에 스프링 컨테이너에 요청해서 프로토타입 빈이 새로 생성이 된 것이지, 사용 할 때마다 새로 생성되는 것이 아니다!**


위 그림처럼 clientBean이라는 싱글톤이 프로토타입 빈을 주입받고 있다면 싱글톤이 생성되고 주입받는 과정에서 끝이 나기 때문에 이 싱글톤 빈을 호출한다고 해서 매번 프로토타입이 생성되는게 아니다.
이럴거면 프로토타입도 싱글톤을 쓰는게 나을 정도이다.

# 문제 해결

## 스프링 컨테이너에 요청
가장 간단한 방법은 싱글톤 빈이 프로토타입을 사용할 때 마다 스프링 컨테이너에 새로 요청하는 것이다.

```java
public class PrototypeProviderTest {
    @Test
    void providerTest() {
        AnnotationConfigApplicationContext ac = new
                AnnotationConfigApplicationContext(ClientBean.class, PrototypeBean.class);
        ClientBean clientBean1 = ac.getBean(ClientBean.class);
        int count1 = clientBean1.logic();
        Assertions.assertThat(count1).isEqualTo(1);
        ClientBean clientBean2 = ac.getBean(ClientBean.class);
        int count2 = clientBean2.logic();
        Assertions.assertThat(count2).isEqualTo(1);
    }

    static class ClientBean {

        @Autowired
        private ApplicationContext ac;

        public int logic() {
            PrototypeBean prototypeBean = ac.getBean(PrototypeBean.class);
            prototypeBean.addCount();
            int count = prototypeBean.getCount();
            return count;
        }
    }

    @Scope("prototype")
    static class PrototypeBean {
        private int count = 0;

        public void addCount() {
            count++;
        }

        public int getCount() {
            return count;
        }

        @PostConstruct
        public void init() {
            System.out.println("PrototypeBean.init " + this);
        }

        @PreDestroy
        public void destroy() {
            System.out.println("PrototypeBean.destroy");
        }
    }
}

```

위처럼 빈 자체에서 프로토타입 빈을 요청한다.

- 실행해보면 `ac.getBean()` 을 통해서 항상 새로운 프로토타입 빈이 생성되는 것을 확인할 수 있다.
- 의존관계를 외부에서 주입(DI) 받는게 아니라 이렇게 직접 필요한 의존관계를 찾는 것을 Dependency Lookup (DL) 의존관계 조회(탐색) 이라한다.
- 그런데 이렇게 스프링의 애플리케이션 컨텍스트 전체를 주입받게 되면, 스프링 컨테이너에 종속적인 코드가 되고, 단위 테스트도 어려워진다.
- 지금 필요한 기능은 지정한 프로토타입 빈을 컨테이너에서 대신 찾아주는 딱! **DL** 정도의 기능만 제공하는 무언가 가 있으면 된다.



## ObjectFactory, ObjectProvider
지정한 빈을 컨테이너에서 대신 찾아주는 DL 서비스를 제공하는 것이 바로 `ObjectProvider` 이다. 참고로 과거에는 `ObjectFactory` 가 있었는데, 여기에 편의 기능을 추가해서 `ObjectProvider` 가 만들어졌다.




```java
 @Autowired
 private ObjectProvider<PrototypeBean> prototypeBeanProvider;
 public int logic() {
     PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
     prototypeBean.addCount();
     int count = prototypeBean.getCount();
     return count;
}

```

**특징**
- ObjectFactory: 기능이 단순, 별도의 라이브러리 필요 없음, 스프링에 의존
- ObjectProvider: ObjectFactory 상속, 옵션, 스트림 처리등 편의 기능이 많고, 별도의 라이브러리 필요 없음, 스프링에 의존


## JSR-330 Provider

```java
 @Autowired
 private Provider<PrototypeBean> provider;
 public int logic() {
     PrototypeBean prototypeBean = provider.get();
     prototypeBean.addCount();
     int count = prototypeBean.getCount();
     return count;
}
```

위처럼 사용하면 되고 자바 표준이다.


**특징**
- `get()` 메서드 하나로 기능이 매우 단순하다.
- 별도의 라이브러리가 필요하다.
- 자바 표준이므로 스프링이 아닌 다른 컨테이너에서도 사용할 수 있다.


**정리**
- 그러면 프로토타입 빈을 언제 사용할까? 매번 사용할 때 마다 의존관계 주입이 완료된 새로운 객체가 필요하면 사 용하면 된다. 그런데 실무에서 웹 애플리케이션을 개발해보면, 싱글톤 빈으로 대부분의 문제를 해결할 수 있기 때 문에 프로토타입 빈을 직접적으로 사용하는 일은 매우 드물다.
- `ObjectProvider` , `JSR330 Provider` 등은 프로토타입 뿐만 아니라 DL이 필요한 경우는 언제든지 사용할 수 있다.


>실무에서 자바 표준인 JSR-330 Provider를 사용할 것인지, 아니면 스프링이 제공하는 ObjectProvider 를 사용할 것인지 고민이 될 것이다. ObjectProvider는 DL을 위한 편의 기능을 많이 제공해주고 스프링 외에 별 도의 의존관계 추가가 필요 없기 때문에 편리하다. 만약(정말 그럴일은 거의 없겠지만) 코드를 스프링이 아닌 다른 컨테이너에서도 사용할 수 있어야 한다면 JSR-330 Provider를 사용해야한다.
스프링을 사용하다 보면 이 기능 뿐만 아니라 다른 기능들도 자바 표준과 스프링이 제공하는 기능이 겹칠때가 많 이 있다. 대부분 스프링이 더 다양하고 편리한 기능을 제공해주기 때문에, 특별히 다른 컨테이너를 사용할 일이 없 다면, 스프링이 제공하는 기능을 사용하면 된다.


# 결론

자바 표준과 스프링 표준을 선택하는 것에서 더 고민해보자
