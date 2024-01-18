>**실행하기 전에 트랜잭션 관련 로그를 확인할 수 있도록 다음을 꼭! 추가하자.** 
```properties
 logging.level.org.springframework.transaction.interceptor=TRACE
 logging.level.org.springframework.jdbc.datasource.DataSourceTransactionManager=D
EBUG
#JPA log
 logging.level.org.springframework.orm.jpa.JpaTransactionManager=DEBUG
 logging.level.org.hibernate.resource.transaction=DEBUG
#JPA SQL
logging.level.org.hibernate.SQL=DEBUG
```

# 커밋, 롤백
```java
    @TestConfiguration
    static class Config {
@Bean
        public PlatformTransactionManager transactionManager(DataSource
dataSource) {
            return new DataSourceTransactionManager(dataSource);
        }
}
    @Test
    void commit() {
		log.info("트랜잭션 시작");
        TransactionStatus status = txManager.getTransaction(new
		DefaultTransactionAttribute());
		log.info("트랜잭션 커밋 시작"); txManager.commit(status); log.info("트랜잭션 커밋 완료");
}
    @Test
    void rollback() {
		log.info("트랜잭션 시작");
        TransactionStatus status = txManager.getTransaction(new
		DefaultTransactionAttribute());
		log.info("트랜잭션 롤백 시작"); txManager.rollback(status); log.info("트랜잭션 롤백 완료");
}
}
```


>`@TestConfiguration` : 해당 테스트에서 필요한 스프링 설정을 추가로 할 수 있다. `DataSourceTransactionManager` 를 스프링 빈으로 등록했다. 이후 트랜잭션 매니저인 `PlatformTransactionManager` 를 주입 받으면 방금 등록한 `DataSourceTransactionManager` 가 주입된다.



# 트랜잭션 두 번 사용

**double_commit() - BasicTxTest 추가**
```java
 @Test
 void double_commit() {
log.info("트랜잭션1 시작");
     TransactionStatus tx1 = txManager.getTransaction(new
 DefaultTransactionAttribute());
log.info("트랜잭션1 커밋"); txManager.commit(tx1);
log.info("트랜잭션2 시작");
TransactionStatus tx2 = txManager.getTransaction(new
 
 DefaultTransactionAttribute()); log.info("트랜잭션2 커밋"); txManager.commit(tx2);
}
```


>**주의!**
로그를 보면 트랜잭션1과 트랜잭션2가 같은 `conn0` 커넥션을 사용중이다. 이것은 중간에 커넥션 풀 때문에 그런 것이 다. 트랜잭션1은 `conn0` 커넥션을 모두 사용하고 커넥션 풀에 반납까지 완료했다. 이후에 트랜잭션2가 `conn0` 를 커 넥션 풀에서 획득한 것이다. 따라서 둘은 완전히 다른 커넥션으로 인지하는 것이 맞다.
트랜잭션1: `Acquired Connection [HikariProxyConnection@1000000 wrapping conn0]`
트랜잭션2: `Acquired Connection [HikariProxyConnection@2000000 wrapping conn0]`
히카리 커넥션풀이 반환해주는 커넥션을 다루는 프록시 객체의 주소가 트랜잭션1은 `HikariProxyConnection@1000000` 이고, 트랜잭션2는 `HikariProxyConnection@2000000` 으로 서로 다
른 것을 확인할 수 있다.
결과적으로 `conn0` 을 통해 커넥션이 재사용 된 것을 확인할 수 있고, `HikariProxyConnection@1000000` ,
`HikariProxyConnection@2000000` 을 통해 각각 커넥션 풀에서 커넥션을 조회한 것을 확인할 수 있다.

### 둘 다 커밋하는 경우
![](https://velog.velcdn.com/images/jckim22/post/aa9bb6ab-5a0e-4bd4-ad03-6f8cd53d9509/image.png)

### 둘 다 롤백하는 경우
![](https://velog.velcdn.com/images/jckim22/post/36264e1e-298b-44a8-b5ae-9e5d759aa343/image.png)

위처럼 트랜잭션 2개를 각각 사용하면 따로 진행이 된다.(물리적으로는 같은 커넥션이지만, 아예 다른 커넥션으로 인지하는 것이 맞기 때문)


#  전파 

트랜잭션을 각각 사용하는 것이 아니라, 트랜잭션이 이미 진행중인데, 여기에 추가로 트랜잭션을 수행하면 어떻게 될까?

기존 트랜잭션과 별도의 트랜잭션을 진행해야 할까? 아니면 기존 트랜잭션을 그대로 이어 받아서 트랜잭션을 수행해야 할까?

이런 경우 어떻게 동작할지 결정하는 것을 트랜잭션 전파(propagation)라 한다. 
스프링은 다양한 트랜잭션 전파 옵션을 제공한다.


>지금부터 설명하는 내용은 트랜잭션 전파의 기본 옵션인 `REQUIRED` 를 기준으로 설명한다.



**물리 트랜잭션, 논리 트랜잭션**

![](https://velog.velcdn.com/images/jckim22/post/a4c65806-d1ff-4ec2-ab20-212a7ca68206/image.png)

- 스프링은 물리적인 트랜잭션과 논리적인 트랜잭션이라는 개념으로 트랜잭션을 나눈다.
>물론 트랜잭션이 진행준인데 추가적으로 트랜잭션이 수행될 때만 해당된다.

이렇게 하면 이런 복잡한 상황에서 다음 간단한 원칙으로 해결할 수 있게 된다.

**원칙**
- **모든 논리 트랜잭션이 커밋되어야 물리 트랜잭션이 커밋된다.** 
- **하나의 논리 트랜잭션이라도 롤백되면 물리 트랜잭션은 롤백된다.**


**inner_commit() - BasicTxTest 추가** 
```java
 @Test
 void inner_commit() {
	log.info("외부 트랜잭션 시작");
     TransactionStatus outer = txManager.getTransaction(new
	 DefaultTransactionAttribute());
     log.info("outer.isNewTransaction()={}", outer.isNewTransaction());
	log.info("내부 트랜잭션 시작");
     TransactionStatus inner = txManager.getTransaction(new
	 DefaultTransactionAttribute());
	log.info("inner.isNewTransaction()={}", inner.isNewTransaction()); log.info("내부 트랜잭션 커밋");
	txManager.commit(inner);
	log.info("외부 트랜잭션 커밋");
     txManager.commit(outer);
}
```

위 그림과 같은 상황에 코드를 작성했다.

**트랜잭션 참여**
- 지금까지 설명한 상황처럼 그냥 내부 트랜잭션(논리)이 외부 트랜잭션에 참여해서 하나의 큰 트랜잭션(물리)가 되는 것이다.

내부 트랜잭션은 이미 진행중인 외부 트랜잭션에 참여한다. 이 경우 신규 트랜잭션이 아니다

그런데 코드를 잘 보면 커밋을 두 번 호출했다. 트랜잭션을 생각해보면 하나의 커넥션에 커밋은 한번만 호출할 수 있다. 커밋이나 롤백을 하면 해당 트랜잭션은 끝나버린다.

**스프링의 대처**

로그를 보면 내부 트랜잭션 커밋이후 아무 일도 발생하지 않고 외부 트랜잭션 커밋이 진행된다.
**외부 트랜잭션만 물리 트랜잭션을 시작하고 커밋한다.**

스프링은 처음 트랜잭션이 실제 물리 트랜잭션을 관리하도록 하고 이후는 참여하도록 하게 한다.
이를 통해 중복 문제를 해결한다.

## 외부 커밋

### 요청 흐름

![](https://velog.velcdn.com/images/jckim22/post/f164152e-5e4d-463e-a4b8-51beca79ce8b/image.png)

위 그림처럼 내부 트랜잭션 매니저는 트랜잭션 동기화 매니저를 통해 기존 트랜잭션이 존재하는지 확인하고 존재한다면 참여한다. (아무것도 하지 않는다.)
-> 이후 로직들은 자연스럽게 트랜잭션 동기화 매니저에 보관된 기존 커넥션을 사용하게 된다.

### 응답 흐름

![](https://velog.velcdn.com/images/jckim22/post/471b6082-6e84-4295-834a-440dea41cc0f/image.png)

내부 커밋은 실제 커밋을 호출하지 않고 외부 커밋일 때 실제 커밋을 호출한다.
실제 커넥션에 커밋하는 것이기 때문에 이를 물리 커밋이라고 할 수 있다.


**핵심 정리**
- 여기서 핵심은 트랜잭션 매니저에 커밋을 호출한다고해서 항상 실제 커넥션에 물리 커밋이 발생하지는 않는다는 점이다.
- 신규 트랜잭션인 경우에만 실제 커넥션을 사용해서 물리 커밋과 롤백을 수행한다. 신규 트랜잭션이 아니면 실제 물리 커넥션을 사용하지 않는다.
- 이렇게 트랜잭션이 내부에서 추가로 사용되면 트랜잭션 매니저에 커밋하는 것이 항상 물리 커밋으로 이어지지 않 는다. 그래서 이 경우 논리 트랜잭션과 물리 트랜잭션을 나누게 된다. 또는 외부 트랜잭션과 내부 트랜잭션으로 나누어 설명하기도 한다.
- 트랜잭션이 내부에서 추가로 사용되면, 트랜잭션 매니저를 통해 논리 트랜잭션을 관리하고, 모든 논리 트랜잭션이 커밋되면 물리 트랜잭션이 커밋된다고 이해하면 된다.


## 외부 롤백

![](https://velog.velcdn.com/images/jckim22/post/d00bcb24-0f7c-40c3-8dcb-8898d4800ea4/image.png)

논리 트랜잭션이 하나라도 롤백되면 전체 물리 트랜잭션은 롤백된다.
따라서 이 경우 내부 트랜잭션이 커밋했어도, 내부 트랜잭션 안에서 저장한 데이터도 모두 함께 롤백된다.

외부 커밋인 상황과 비슷하다.

## 내부 롤백

![](https://velog.velcdn.com/images/jckim22/post/783c8953-649b-4b4d-b12f-5f98d77a8695/image.png)

 내부 트랜잭션이 롤백을 했지만, 내부 트랜잭션은 물 리 트랜잭션에 영향을 주지 않는다. 그런데 외부 트랜잭션은 커밋을 해버린다. 지금까지 학습한 내용을 돌아보면 외부 트랜잭션만 물리 트랜잭션에 영향을 주기 때문에 물리 트랜잭션이 커밋될 것 같다.
 
 **실행 결과 - inner_rollback()** 
 ```
외부 트랜잭션 시작
 Creating new transaction with name [null]:
 PROPAGATION_REQUIRED,ISOLATION_DEFAULT
 Acquired Connection [HikariProxyConnection@220038608 wrapping conn0] for JDBC
 transaction
 Switching JDBC Connection [HikariProxyConnection@220038608 wrapping conn0] to
 manual commit
내부 트랜잭션 시작
 Participating in existing transaction
내부 트랜잭션 롤백
 Participating transaction failed - marking existing transaction as rollback-only
 Setting JDBC transaction [HikariProxyConnection@220038608 wrapping conn0]
 rollback-only
외부 트랜잭션 커밋
 Global transaction is marked as rollback-only but transactional code requested
 commit
 Initiating transaction rollback
 Rolling back JDBC transaction on Connection [HikariProxyConnection@220038608
 wrapping conn0]
 Releasing JDBC Connection [HikariProxyConnection@220038608 wrapping conn0] after
transaction
 ```
 
 내부 롤백을 하게 되면 위같은 로그가 발생한다.
 
- 외부 트랜잭션 시작
물리 트랜잭션을 시작한다.
- 내부 트랜잭션 시작
`Participating in existing transaction`
기존 트랜잭션에 참여한다.
- 내부 트랜잭션 롤백
`Participating transaction failed - marking existing transaction as rollback-only`
내부 트랜잭션을 롤백하면 실제 물리 트랜잭션은 롤백하지 않는다. 대신에 기존 트랜잭션을 롤백 전용으로 표시한다.
- 외부 트랜잭션 커밋
외부 트랜잭션을 커밋한다.
`Global transaction is marked as rollback-only`
커밋을 호출했지만, 전체 트랜잭션이 롤백 전용으로 표시되어 있다. 따라서 물리 트랜잭션을 롤백한다.
 
 ![](https://velog.velcdn.com/images/jckim22/post/88dc9988-8b19-44ca-a6c4-70ce0d47600a/image.png)

#### 요청흐름
롤백을 요청했고 트랜잭션 매니저에서 신규 트랜잭션이 아닌 것을 확인했다.
하지만 여기서 물리적인 롤백을 해버리면 로직이 끝나기 때문에 실제 롤백을 수행하지는 않는다.

대신 트랜잭션 동기화 매니저에게 rollbackOnly라는 설정을 하게 한다.

#### 응답 흐름

외부 트랜잭션은 커밋을 하려고 했으나 rollbackOnly라는 설정으로 인해 rollback을 하게 된다.
실제 데이터베이스에 rollback이 반영되고 이는 물리적인 rollback이 된다.

- 트랜잭션 매니저에 커밋을 호출한 개발자 입장에서는 분명 커밋을 기대했는데 롤백 전용 표시로 인해 실제
로는 롤백이 되어버렸다.
이것은 조용히 넘어갈 수 있는 문제가 아니다. 시스템 입장에서는 커밋을 호출했지만 롤백이 되었다는 것은 분명하게 알려주어야 한다.
예를 들어서 고객은 주문이 성공했다고 생각했는데, 실제로는 롤백이 되어서 주문이 생성되지 않은 것이다. 스프링은 이 경우 `UnexpectedRollbackException` 런타임 예외를 던진다. 그래서 커밋을 시도했지 만, 기대하지 않은 롤백이 발생했다는 것을 명확하게 알려준다.



**정리**
- 논리 트랜잭션이 하나라도 롤백되면 물리 트랜잭션은 롤백된다.
- 내부 논리 트랜잭션이 롤백되면 롤백 전용 마크를 표시한다.
- 외부 트랜잭션을 커밋할 때 롤백 전용 마크를 확인한다. 롤백 전용 마크가 표시되어 있으면 물리 트랜잭션을 롤백 하고, `UnexpectedRollbackException` 예외를 던진다.


# REQUIRES_NEW

외부 트랜잭션과 내부 트랜잭션을 완전히 분리해서 각각 별도의 물리 트랜잭션을 사용하는 방법이다.
그래서 커밋과 롤백도 각각 별도로 이루어지게 된다.

이 방법은 내부 트랜잭션에 문제가 발생해서 롤백해도, 외부 트랜잭션에는 영향을 주지 않는다. 반대로 외부 트랜잭션에 문제가 발생해도 내부 트랜잭션에 영향을 주지 않는다. 


![](https://velog.velcdn.com/images/jckim22/post/de01fd21-317f-47aa-bd02-f8bdc3c6e57e/image.png)


```java
    @Test
    void inner_rollback_requires_new() {
        log.info("외부 트랜잭션 시작");
        TransactionStatus outer = txManager.getTransaction(new DefaultTransactionAttribute());
        log.info("outer.isNewTransaction()={}", outer.isNewTransaction()); //true

        log.info("내부 트랜잭션 시작");
        DefaultTransactionAttribute definition = new DefaultTransactionAttribute();
        definition.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);
        TransactionStatus inner = txManager.getTransaction(definition);
        log.info("inner.isNewTransaction()={}", inner.isNewTransaction()); //true

        log.info("내부 트랜잭션 롤백");
        txManager.rollback(inner); //롤백

        log.info("외부 트랜잭션 커밋");
        txManager.commit(outer); //커밋
    }
```

- 내부 트랜잭션을 시작할 때 전파 옵션인 `propagationBehavior` 에 `PROPAGATION_REQUIRES_NEW` 옵션 을 주었다.
- 이 전파 옵션을 사용하면 내부 트랜잭션을 시작할 때 기존 트랜잭션에 참여하는 것이 아니라 새로운 물리 트랜잭 션을 만들어서 시작하게 된다.


**외부 트랜잭션 시작**
외부 트랜잭션을 시작하면서 `conn0` 를 획득하고 `manual commit` 으로 변경해서 물리 트랜잭션을 시작한다. 외부 트랜잭션은 신규 트랜잭션이다.( `outer.isNewTransaction()=true` )

**내부 트랜잭션 시작**
내부 트랜잭션을 시작하면서 `conn1` 를 획득하고 `manual commit` 으로 변경해서 물리 트랜잭션을 시작한다. 내부 트랜잭션은 외부 트랜잭션에 참여하는 것이 아니라, `PROPAGATION_REQUIRES_NEW` 옵션을 사용했기 때 문에 완전히 새로운 신규 트랜잭션으로 생성된다.( `inner.isNewTransaction()=true` )

**내부 트랜잭션 롤백**
내부 트랜잭션을 롤백한다.
내부 트랜잭션은 신규 트랜잭션이기 때문에 실제 물리 트랜잭션을 롤백한다. 내부 트랜잭션은 `conn1` 을 사용하므로 `conn1` 에 물리 롤백을 수행한다.

**외부 트랜잭션 커밋**
외부 트랜잭션을 커밋한다.
외부 트랜잭션은 신규 트랜잭션이기 때문에 실제 물리 트랜잭션을 커밋한다. 외부 트랜잭션은 `conn0` 를 사용하므로 `conn0` 에 물리 커밋을 수행한다.

#### 요청 흐름
![업로드중..](blob:https://velog.io/5ce04c00-fcc2-4b06-a023-c93d680cc75d)

**외부**
트랜잭션 매니저는 트랜잭션을 생성한 결과를 `TransactionStatus` 에 담아서 반환하는데, 여기에 신규
트랜잭션의 여부가 담겨 있다. `isNewTransaction` 를 통해 신규 트랜잭션 여부를 확인할 수 있다. 트랜
잭션을 처음 시작했으므로 신규 트랜잭션이다.( `true` )

**내부**
**REQUIRES_NEW 옵션**과 함께 `txManager.getTransaction()` 를 호출해서 내부 트랜잭션을 시작
한다.
트랜잭션 매니저는 `REQUIRES_NEW` 옵션을 확인하고, 기존 트랜잭션에 참여하는 것이 아니라 새로운 트 랜잭션을 시작한다.
트랜잭션 매니저는 데이터소스를 통해 커넥션을 생성한다.
생성한 커넥션을 수동 커밋 모드( `setAutoCommit(false)` )로 설정한다. - **물리 트랜잭션 시작**

트랜잭션 매니저는 트랜잭션 동기화 매니저에 커넥션을 보관한다.
이때 `con1` 은 잠시 보류되고, 지금부터는 `con2` 가 사용된다. (내부 트랜잭션을 완료할 때 까지 `con2` 가 사용된다.)
트랜잭션 매니저는 신규 트랜잭션의 생성한 결과를 반환한다. `isNewTransaction == true`
로직2가 사용되고, 커넥션이 필요한 경우 트랜잭션 동기화 매니저에 있는 `con2` 커넥션을 획득해서 사용한
다.



#### 응답 흐름 
**내부**
1. 로직2가 끝나고 트랜잭션 매니저를 통해 내부 트랜잭션을 롤백한다. (로직2에 문제가 있어서 롤백한다고 가
정한다.)
2. 트랜잭션 매니저는 롤백 시점에 신규 트랜잭션 여부에 따라 다르게 동작한다. 현재 내부 트랜잭션은 신규 트
랜잭션이다. 따라서 실제 롤백을 호출한다.
3. 내부 트랜잭션이 `con2` 물리 트랜잭션을 롤백한다.
트랜잭션이 종료되고, `con2` 는 종료되거나, 커넥션 풀에 반납된다. 이후에 `con1` 의 보류가 끝나고, 다시 `con1` 을 사용한다.

**외부**
4. 외부 트랜잭션에 커밋을 요청한다.
5. 외부 트랜잭션은 신규 트랜잭션이기 때문에 물리 트랜잭션을 커밋한다.
6. 이때 `rollbackOnly` 설정을 체크한다. `rollbackOnly` 설정이 없으므로 커밋한다.
7. 본인이 만든 `con1` 커넥션을 통해 물리 트랜잭션을 커밋한다.
트랜잭션이 종료되고, `con1` 은 종료되거나, 커넥션 풀에 반납된다.

**정리**
`REQUIRES_NEW` 옵션을 사용하면 물리 트랜잭션이 명확하게 분리된다.
`REQUIRES_NEW` 를 사용하면 데이터베이스 커넥션이 동시에 2개 사용된다는 점을 주의해야 한다.

# 다양한 전파 옵션
스프링은 다양한 트랜잭션 전파 옵션을 제공한다. 전파 옵션에 별도의 설정을 하지 않으면 `REQUIRED` 가 기본으로 사 용된다.
참고로 실무에서는 대부분 `REQUIRED` 옵션을 사용한다. 그리고 아주 가끔 `REQUIRES_NEW` 을 사용하고, 나머지는 거 의 사용하지 않는다. 그래서 나머지 옵션은 이런 것이 있다는 정도로만 알아두고 필요할 때 찾아보자.

**REQUIRED**
가장 많이 사용하는 기본 설정이다. 기존 트랜잭션이 없으면 생성하고, 있으면 참여한다. 트랜잭션이 필수라는 의미로 이해하면 된다. (필수이기 때문에 없으면 만들고, 있으면 참여한다.)
기존 트랜잭션 없음: 새로운 트랜잭션을 생성한다. 기존 트랜잭션 있음: 기존 트랜잭션에 참여한다.

**REQUIRES_NEW**
항상 새로운 트랜잭션을 생성한다.
기존 트랜잭션 없음: 새로운 트랜잭션을 생성한다. 기존 트랜잭션 있음: 새로운 트랜잭션을 생성한다.

**SUPPORT**
트랜잭션을 지원한다는 뜻이다. 기존 트랜잭션이 없으면, 없는대로 진행하고, 있으면 참여한다.
기존 트랜잭션 없음: 트랜잭션 없이 진행한다. 기존 트랜잭션 있음: 기존 트랜잭션에 참여한다.
       
**NOT_SUPPORT**
트랜잭션을 지원하지 않는다는 의미이다.
기존 트랜잭션 없음: 트랜잭션 없이 진행한다.
기존 트랜잭션 있음: 트랜잭션 없이 진행한다. (기존 트랜잭션은 보류한다)

**MANDATORY**
의무사항이다. 트랜잭션이 반드시 있어야 한다. 기존 트랜잭션이 없으면 예외가 발생한다.
기존 트랜잭션 없음: `IllegalTransactionStateException` 예외 발생 기존 트랜잭션 있음: 기존 트랜잭션에 참여한다.

**NEVER**
트랜잭션을 사용하지 않는다는 의미이다. 기존 트랜잭션이 있으면 예외가 발생한다. 기존 트랜잭션도 허용하지 않는 강 한 부정의 의미로 이해하면 된다.
기존 트랜잭션 없음: 트랜잭션 없이 진행한다.
기존 트랜잭션 있음: `IllegalTransactionStateException` 예외 발생

**NESTED**
기존 트랜잭션 없음: 새로운 트랜잭션을 생성한다.
기존 트랜잭션 있음: 중첩 트랜잭션을 만든다.
중첩 트랜잭션은 외부 트랜잭션의 영향을 받지만, 중첩 트랜잭션은 외부에 영향을 주지 않는다. 중첩 트랜잭션이 롤백 되어도 외부 트랜잭션은 커밋할 수 있다.
외부 트랜잭션이 롤백 되면 중첩 트랜잭션도 함께 롤백된다.
참고
JDBC savepoint 기능을 사용한다. DB 드라이버에서 해당 기능을 지원하는지 확인이 필요하다. 중첩 트랜잭션은 JPA에서는 사용할 수 없다.

**트랜잭션 전파와 옵션**
`isolation` , `timeout` , `readOnly` 는 트랜잭션이 처음 시작될 때만 적용된다. 트랜잭션에 참여하는 경우에는 적용
되지 않는다.
예를 들어서 `REQUIRED` 를 통한 트랜잭션 시작, `REQUIRES_NEW` 를 통한 트랜잭션 시작 시점에만 적용된다.
