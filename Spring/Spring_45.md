# 트랜잭션 - 개념 이해
데이터를 저장할 때 단순히 파일에 저장해도 되는데, 데이터베이스에 저장하는 이유는 무엇일까?
여러가지 이유가 있지만, 가장 대표적인 이유는 바로 데이터베이스는 트랜잭션이라는 개념을 지원하기 때문이다.
트랜잭션을 이름 그대로 번역하면 거래라는 뜻이다. 이것을 쉽게 풀어서 이야기하면, 데이터베이스에서 트랜잭션은 하 나의 거래를 안전하게 처리하도록 보장해주는 것을 뜻한다. 그런데 하나의 거래를 안전하게 처리하려면 생각보다 고려 해야 할 점이 많다. 예를 들어서 A의 5000원을 B에게 계좌이체한다고 생각해보자. A의 잔고를 5000원 감소하고, B의 잔고를 5000원 증가해야한다.

**5000원 계좌이체**
1. A의 잔고를 5000원 감소
2. B의 잔고를 5000원 증가
계좌이체라는 거래는 이렇게 2가지 작업이 합쳐져서 하나의 작업처럼 동작해야 한다. 만약 1번은 성공했는데 2번에서 시스템에 문제가 발생하면 계좌이체는 실패하고, A의 잔고만 5000원 감소하는 심각한 문제가 발생한다. 데이터베이스가 제공하는 트랜잭션 기능을 사용하면 1,2 둘다 함께 성공해야 저장하고, 중간에 하나라도 실패하면 거래 전의 상태로 돌아갈 수 있다. 만약 1번은 성공했는데 2번에서 시스템에 문제가 발생하면 계좌이체는 실패하고, 거래 전 의 상태로 완전히 돌아갈 수 있다. 결과적으로 A의 잔고가 감소하지 않는다.

모든 작업이 성공해서 데이터베이스에 정상 반영하는 것을 커밋( `Commit` )이라 하고, 작업 중 하나라도 실패해서 거래 이전으로 되돌리는 것을 롤백( `Rollback` )이라 한다.


# 트랜잭션 ACID

트랜잭션은 ACID(http://en.wikipedia.org/wiki/ACID)라 하는 원자성(Atomicity), 일관성(Consistency), 격리성(Isolation), 지속성(Durability)을 보장해야 한다.

- **원자성:** 트랜잭션 내에서 실행한 작업들은 마치 하나의 작업인 것처럼 모두 성공 하거나 모두 실패해야 한다.
- **일관성:** 모든 트랜잭션은 일관성 있는 데이터베이스 상태를 유지해야 한다. 예를 들어 데이터베이스에서 정한 무 결성 제약 조건을 항상 만족해야 한다.
- **격리성:** 동시에 실행되는 트랜잭션들이 서로에게 영향을 미치지 않도록 격리한다. 예를 들어 동시에 같은 데이터 를 수정하지 못하도록 해야 한다. 격리성은 동시성과 관련된 성능 이슈로 인해 트랜잭션 격리 수준(Isolation level)을 선택할 수 있다.
- **지속성:** 트랜잭션을 성공적으로 끝내면 그 결과가 항상 기록되어야 한다. 중간에 시스템에 문제가 발생해도 데이 터베이스 로그 등을 사용해서 성공한 트랜잭션 내용을 복구해야 한다.


트랜잭션은 원자성, 일관성, 지속성을 보장한다. 문제는 격리성인데 트랜잭션 간에 격리성을 완벽히 보장하려면 트랜잭 션을 거의 순서대로 실행해야 한다. 이렇게 하면 동시 처리 성능이 매우 나빠진다. 이런 문제로 인해 ANSI 표준은 트랜 잭션의 격리 수준을 4단계로 나누어 정의했다.

**트랜잭션 격리 수준 - Isolation level**
- READ UNCOMMITED(커밋되지 않은 읽기)
- READ COMMITTED(커밋된 읽기)
- REPEATABLE READ(반복 가능한 읽기) 
- SERIALIZABLE(직렬화 가능)

> 이후에는 일반적으로 많이 사용하는 READ COMMITTED(커밋된 읽기) 트랜잭션 격리 수준을 기준으 로 설명한다.

# 데이터베이스 연결 구조와 DB 세션

트랜잭션을 더 자세히 이해하기 위해 데이터베이스 서버 연결 구조와 DB 세션에 대해 알아보자.

![](https://velog.velcdn.com/images/jckim22/post/5708c33b-522d-4405-83dc-296dc6754e26/image.png)

- 사용자는 웹 애플리케이션 서버(WAS)나 DB 접근 툴 같은 클라이언트를 사용해서 데이터베이스 서버에 접근할 수 있다. 클라이언트는 데이터베이스 서버에 연결을 요청하고 커넥션을 맺게 된다. 이때 데이터베이스 서버는 내 부에 세션이라는 것을 만든다. 그리고 앞으로 해당 커넥션을 통한 모든 요청은 이 세션을 통해서 실행하게 된다.
- 쉽게 이야기해서 개발자가 클라이언트를 통해 SQL을 전달하면 현재 커넥션에 연결된 세션이 SQL을 실행한다. 
- 세션은 트랜잭션을 시작하고, 커밋 또는 롤백을 통해 트랜잭션을 종료한다. 그리고 이후에 새로운 트랜잭션을 다 시 시작할 수 있다.
- 사용자가 커넥션을 닫거나, 또는 DBA(DB 관리자)가 세션을 강제로 종료하면 세션은 종료된다.

![](https://velog.velcdn.com/images/jckim22/post/3c185013-77e2-4740-b82a-85691cda7e3f/image.png)
커넥션 풀이 10개의 커넥션을 생성하면, 세션도 10개 만들어진다.

# 트랜잭션 - DB 예제1 - 개념 이해

**트랜잭션 사용법**
- 데이터 변경 쿼리를 실행하고 데이터베이스에 그 결과를 반영하려면 커밋 명령어인 `commit` 을 호출하고, 결과를 반영하고 싶지 않으면 롤백 명령어인 `rollback` 을 호출하면 된다.
- **커밋을 호출하기 전까지는 임시로 데이터를 저장**하는 것이다. 따라서 해당 트랜잭션을 시작한 세션(사용자)에게만 변경 데이터가 보이고 다른 세션(사용자)에게는 변경 데이터가 보이지 않는다.
- 등록, 수정, 삭제 모두 같은 원리로 동작한다. 앞으로는 등록, 수정, 삭제를 간단히 **변경**이라는 단어로 표현하겠다.

![](https://velog.velcdn.com/images/jckim22/post/1afbe861-4092-427e-95c8-ded77af7827f/image.png)

**세션1 신규 데이터 추가**

![](https://velog.velcdn.com/images/jckim22/post/186aee43-4dcc-4f02-9ec5-2695a6b198b0/image.png)

- 세션1은 트랜잭션을 시작하고 신규 회원1, 신규 회원2를 DB에 추가했다. 아직 커밋은 하지 않은 상태이다. 
- 새로운 데이터는 임시 상태로 저장된다.
- 세션1은 `select` 쿼리를 실행해서 본인이 입력한 신규 회원1, 신규 회원2를 조회할 수 있다.
- 세션2는 `select` 쿼리를 실행해도 신규 회원들을 조회할 수 없다. 왜냐하면 세션1이 아직 커밋을 하지 않았기 때문이다.


**커밋하지 않은 데이터를 다른 곳에서 조회할 수 있으면 어떤 문제가 발생할까?**
- 예를 들어서 커밋하지 않는 데이터가 보인다면, 세션2는 데이터를 조회했을 때 신규 회원1, 2가 보일 것이다. 따 라서 신규 회원1, 신규 회원2가 있다고 가정하고 어떤 로직을 수행할 수 있다. 그런데 세션1이 롤백을 수행하면 신규 회원1, 신규 회원2의 데이터가 사라지게 된다. 따라서 데이터 정합성에 큰 문제가 발생한다.
- 세션2에서 세션1이 아직 커밋하지 않은 변경 데이터가 보이다면, 세션1이 롤백 했을 때 심각한 문제가 발생할 수 있다. 따라서 커밋 전의 데이터는 다른 세션에서 보이지 않는다.



**세션1 신규 데이터 추가 후 commit**

![](https://velog.velcdn.com/images/jckim22/post/8efcd66d-ba54-4983-aedb-a5618a8790c4/image.png)
- 세션1이 신규 데이터를 추가한 후에 `commit` 을 호출했다.
- `commit` 으로 새로운 데이터가 실제 데이터베이스에 반영된다. 데이터의 상태도 임시 완료로 변경되었다.
- 이제 다른 세션에서도 회원 테이블을 조회하면 신규 회원들을 확인할 수 있다.

**세션1 신규 데이터 추가 후 rollback**

![](https://velog.velcdn.com/images/jckim22/post/7573f35f-4586-4a05-a254-80b6fe5cfc35/image.png)
- 세션1이 신규 데이터를 추가한 후에 `commit` 대신에 `rollback` 을 호출했다.
- 세션1이 데이터베이스에 반영한 모든 데이터가 처음 상태로 복구된다.
- 수정하거나 삭제한 데이터도 `rollback` 을 호출하면 모두 트랜잭션을 시작하기 직전의 상태로 복구된다.

# 트랜잭션 - DB 예제2 - 자동 커밋, 수동 커밋


### 자동 커밋
트랜잭션을 사용하려면 먼저 자동 커밋과 수동 커밋을 이해해야 한다.
자동 커밋으로 설정하면 각각의 쿼리 실행 직후에 자동으로 커밋을 호출한다. 따라서 커밋이나 롤백을 직접 호출하지 않아도 되는 편리함이 있다. 하지만 쿼리를 하나하나 실행할 때 마다 자동으로 커밋이 되어버리기 때문에 우리가 원하는 트랜잭션 기능을 제대로 사용할 수 없다.


**자동 커밋 설정** 
```sql
set autocommit true; //자동 커밋 모드 설정
insert into member(member_id, money) values ('data1',10000); //자동 커밋
insert into member(member_id, money) values ('data2',10000); //자동 커밋 
```
따라서 `commit` , `rollback` 을 직접 호출하면서 트랜잭션 기능을 제대로 수행하려면 자동 커밋을 끄고 수동 커밋을 사용해야 한다.


**수동 커밋 설정**
```sql
set autocommit false; //수동 커밋 모드 설정
 insert into member(member_id, money) values ('data3',10000);
 insert into member(member_id, money) values ('data4',10000);
commit; //수동 커밋
```

보통 자동 커밋 모드가 기본으로 설정된 경우가 많기 때문에, **수동 커밋 모드로 설정하는 것을 트랜잭션을 시작**한다고 표현할 수 있다.
수동 커밋 설정을 하면 이후에 꼭 `commit` , `rollback` 을 호출해야 한다.
참고로 수동 커밋 모드나 자동 커밋 모드는 한번 설정하면 해당 세션에서는 계속 유지된다. 중간에 변경하는 것은 가능 하다.
이제 본격적으로 트랜잭션 예제를 실습해보자.

# 트랜잭션 - DB 예제3 - 트랜잭션 실습
먼저 데이터베이스에서 2개의 커넥션을 열어서 세션을 2개를 만든다

![](https://velog.velcdn.com/images/jckim22/post/823377b8-39d1-47a1-928c-77544ca10e1d/image.png)

위처럼 2개의 세션이 생긴다.
A,B 라고 하겠다.

#### A

수동 커밋으로 데이터를 변경한다.
![](https://velog.velcdn.com/images/jckim22/post/6121a46d-0a5c-4399-9b47-39042a853a45/image.png)

![](https://velog.velcdn.com/images/jckim22/post/ac801d9e-385a-4876-99d1-b5158cb34466/image.png)

조회하면 결과가 정상적으로 나온다. (같은 세션이니까)


#### B
![](https://velog.velcdn.com/images/jckim22/post/d9fc96a3-aabe-4758-850a-3f0653d17150/image.png)

하지만 B에서는 적용이 안되었다(커밋이 안되어서)

#### A

커밋을 해보자
![](https://velog.velcdn.com/images/jckim22/post/d823f295-f58e-4a3f-85f8-ebd0a81181fa/image.png)

#### A,B
둘 다 적용이 되었다.
![](https://velog.velcdn.com/images/jckim22/post/462611e4-6f3b-48bc-aedd-200a26f21b1a/image.png)

#### A
다시 해보자
![](https://velog.velcdn.com/images/jckim22/post/762dc956-1a12-4409-8eef-33e4f7c34c45/image.png)

#### B
반영이 안되었다.
![](https://velog.velcdn.com/images/jckim22/post/f1c77d71-afd0-4429-8ab8-c71f28018280/image.png)

#### A
이번엔 rollback을 해보자
![](https://velog.velcdn.com/images/jckim22/post/222cfc54-87a4-49a9-b99b-30f29058c928/image.png)

#### A, B

둘 다 반영이 안되었다.
![](https://velog.velcdn.com/images/jckim22/post/d9fc96a3-aabe-4758-850a-3f0653d17150/image.png)




## 정리

**원자성:** 트랜잭션 내에서 실행한 작업들은 마치 하나의 작업인 것처럼 모두 성공 하거나 모두 실패해야 한다. 트랜잭션의 원자성 덕분에 여러 SQL 명령어를 마치 하나의 작업인 것 처럼 처리할 수 있었다. 성공하면 한번에 반영하 고, 중간에 실패해도 마치 하나의 작업을 되돌리는 것 처럼 간단히 되돌릴 수 있다.
**오토 커밋**
만약 오토 커밋 모드로 동작하는데, 계좌이체 중간에 실패하면 어떻게 될까? 쿼리를 하나 실행할 때 마다 바로바로 커밋 이 되어버리기 때문에 `memberA` 의 돈만 2000원 줄어드는 심각한 문제가 발생한다.
**트랜잭션 시작**
따라서 이런 종류의 작업은 꼭 수동 커밋 모드를 사용해서 수동으로 커밋, 롤백 할 수 있도록 해야 한다. 보통 이렇게 자 동 커밋 모드에서 수동 커밋 모드로 전환 하는 것을 트랜잭션을 시작한다고 표현한다.



# DB 락

세션1이 트랜잭션을 시작하고 데이터를 수정하는 동안 아직 커밋을 수행하지 않았는데, 세션2에서 동시에 같은 데이터 를 수정하게 되면 여러가지 문제가 발생한다. 바로 트랜잭션의 원자성이 깨지는 것이다. 여기에 더해서 세션1이 중간에 롤백을 하게 되면 세션2는 잘못된 데이터를 수정하는 문제가 발생한다.

이런 문제를 방지하려면, 세션이 트랜잭션을 시작하고 데이터를 수정하는 동안에는 커밋이나 롤백 전까지 다른 세션에서 해당 데이터를 수정할 수 없게 막아야 한다.


![](https://velog.velcdn.com/images/jckim22/post/6d9d50a0-f8f9-40a7-a3e8-70dc3e41d946/image.png)

![](https://velog.velcdn.com/images/jckim22/post/56598076-dfe7-4c4a-9864-83d46faa47fb/image.png)
![](https://velog.velcdn.com/images/jckim22/post/7502b869-b300-4901-b2cd-d2da001bd439/image.png)


![](https://velog.velcdn.com/images/jckim22/post/9ef08a73-3742-43ef-8b7d-76a2e7e64c8f/image.png)
![](https://velog.velcdn.com/images/jckim22/post/33d45fcc-f4b2-47f0-8c5b-eeb80e6680df/image.png)


# DB 락 - 변경, 조회

**세션2** 


```sql
 SET LOCK_TIMEOUT 60000;
 set autocommit false;
update member set money=1000 where member_id = 'memberA'; 
```

세션2는 `memberA` 의 데이터를 1000원으로 수정하려 한다.
세션1이 트랜잭션을 커밋하거나 롤백해서 종료하지 않았으므로 아직 세션1이 락을 가지고 있다. 따라서 세션2는 락을 획득하지 못하기 때문에 데이터를 수정할 수 없다. 세션2는 락이 돌아올 때 까지 대기하게 된다.
`SET LOCK_TIMEOUT 60000` : 락 획득 시간을 60초로 설정한다. 60초 안에 락을 얻지 못하면 예외가 발생한 다.
참고로 H2 데이터베이스에서는 딱 60초에 예외가 발생하지는 않고, 시간이 조금 더 걸릴 수 있다.


**세션2 락 타임아웃**

`SET LOCK_TIMEOUT <milliseconds>` : 락 타임아웃 시간을 설정한다.
예) `SET LOCK_TIMEOUT 10000` 10초, 세션2에 설정하면 세션2가 10초 동안 대기해도 락을 얻지 못하면 락 타임아웃 오류가 발생한다.
위 시나리오 중간에 락을 오랜기간 대기하면 어떻게 되는지 알아보자.
10초 정도 기다리면 세션2에서는 다음과 같은 락 타임아웃 오류가 발생한다.
**세션2의 실행 결과** 
```
 Timeout trying to lock table {0}; SQL statement:
 update member set money=10000 - 2000 where member_id = 'memberA' [50200-200]
HYT00/50200
```


# DB 락 - 조회
**일반적인 조회는 락을 사용하지 않는다**

- 데이터베이스마다 다르지만, 보통 데이터를 조회할 때는 락을 획득하지 않고 바로 데이터를 조회할 수 있다. 예를 들어서 세션1이 락을 획득하고 데이터를 변경하고 있어도, 세션2에서 데이터를 조회는 할 수 있다. 물론 세션2에 서 조회가 아니라 데이터를 변경하려면 락이 필요하기 때문에 락이 돌아올 때 까지 대기해야 한다.


**조회와 락**
- 데이터를 조회할 때도 락을 획득하고 싶을 때가 있다. 이럴 때는 `select for update` 구문을 사용하면 된다. 
- 이렇게 하면 세션1이 조회 시점에 락을 가져가버리기 때문에 다른 세션에서 해당 데이터를 변경할 수 없다.
- 물론 이 경우도 트랜잭션을 커밋하면 락을 반납한다.


**조회 시점에 락이 필요한 경우는 언제일까?**
- 트랜잭션 종료 시점까지 해당 데이터를 다른 곳에서 변경하지 못하도록 강제로 막아야 할 때 사용한다.
- 예를 들어서 애플리케이션 로직에서 `memberA` 의 금액을 조회한 다음에 이 금액 정보로 애플리케이션에서 어떤 계산을 수행한다. 그런데 이 계산이 돈과 관련된 매우 중요한 계산이어서 계산을 완료할 때 까지 `memberA` 의 금 액을 다른곳에서 변경하면 안된다. 이럴 때 조회 시점에 락을 획득하면 된다.


**세션1** `

```sql
 set autocommit false;
 select * from member where member_id='memberA' for update;
```

- `select for update` 구문을 사용하면 조회를 하면서 동시에 선택한 로우의 락도 획득한다. 물론 락이 없다면 락을 획득할 때 까지 대기해야 한다.
- 세션1은 트랜잭션을 종료할 때 까지 `memberA` 의 로우의 락을 보유한다.

**세션2** 

```sql
 set autocommit false;
update member set money=500 where member_id = 'memberA'; 
```

- 세션2는 데이터를 변경하고 싶다. 데이터를 변경하려면 락이 필요하다.
- 세션1이 `memberA` 로우의 락을 획득했기 때문에 세션2는 락을 획득할 때 까지 대기한다.
- 이후에 세션1이 커밋을 수행하면 세션2가 락을 획득하고 데이터를 변경한다. 만약 락 타임아웃 시간이 지나면 락 타임아웃 예외가 발생한다.


# 트랜젝션 - 적용1

실제 애플리케이션에서 DB 트랜잭션을 사용해서 계좌이체 같이 원자성이 중요한 비즈니스 로직을 어떻게 구현하는지 알아보자.
먼저 트랜잭션 없이 단순하게 계좌이체 비즈니스 로직만 구현해보자.

#### MemberServiceV1 
```java
public class MemberServiceV1 {

    private final MemberRepositoryV1 memberRepository;

    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        //시작
        Member fromMember = memberRepository.findById(fromId);
        Member toMember = memberRepository.findById(toId);

        memberRepository.update(fromId, fromMember.getMoney() - money);
        validation(toMember);
        memberRepository.update(toId, toMember.getMoney() + money);
        //커밋, 롤백
    }

    private void validation(Member toMember) {
        if (toMember.getMemberId().equals("ex")) {
            throw new IllegalStateException("이체중 예외 발생");
        }
    }
}
```
서비스를 하나 만들자
MemberRepositoryV1 인터페이스를 의존해서 자동 주입 받는다.
업데이트 하는 과정인데 만약 멤버의 이름이 ex이면 예외가 발생하게 설계되어있다.

#### 테스트 MemberServiceV1Test

```java
class MemberServiceV1Test {

    public static final String MEMBER_A = "memberA";
    public static final String MEMBER_B = "memberB";
    public static final String MEMBER_EX = "ex";

    private MemberRepositoryV1 memberRepository;
    private MemberServiceV1 memberService;

    @BeforeEach
    void before() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
        memberRepository = new MemberRepositoryV1(dataSource);
        memberService = new MemberServiceV1(memberRepository);
    }
        @Test
    @DisplayName("정상 이체")
    void accountTransfer() throws SQLException {
        //given
        Member memberA = new Member(MEMBER_A, 10000);
        Member memberB = new Member(MEMBER_B, 10000);
        memberRepository.save(memberA);
        memberRepository.save(memberB);

        //when
        memberService.accountTransfer(memberA.getMemberId(), memberB.getMemberId(), 2000);

        //then
        Member findMemberA = memberRepository.findById(memberA.getMemberId());
        Member findMemberB = memberRepository.findById(memberB.getMemberId());
        assertThat(findMemberA.getMoney()).isEqualTo(8000);
        assertThat(findMemberB.getMoney()).isEqualTo(12000);
    }
```
정상적인 테스트를 해보자
둘 다 만원이고 A가 B에게 2000원을 보내는 상황을 연출했다.
결과는 8000, 12000이다

다음은 예외적인 테스트를 해보자
```java
class MemberServiceV1Test {

    public static final String MEMBER_A = "memberA";
    public static final String MEMBER_B = "memberB";
    public static final String MEMBER_EX = "ex";

    private MemberRepositoryV1 memberRepository;
    private MemberServiceV1 memberService;

    @BeforeEach
    void before() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
        memberRepository = new MemberRepositoryV1(dataSource);
        memberService = new MemberServiceV1(memberRepository);
    }
    @AfterEach
    void after() throws SQLException {
        memberRepository.delete(MEMBER_A);
        memberRepository.delete(MEMBER_B);
        memberRepository.delete(MEMBER_EX);
    }
    @Test
    @DisplayName("이체중 예외 발생")
    void accountTransferEx() throws SQLException {
        //given
        Member memberA = new Member(MEMBER_A, 10000);
        Member memberEx = new Member(MEMBER_EX, 10000);
        memberRepository.save(memberA);
        memberRepository.save(memberEx);

        //when
        assertThatThrownBy(() -> memberService.accountTransfer(memberA.getMemberId(), memberEx.getMemberId(), 2000))
                .isInstanceOf(IllegalStateException.class);

        //then
        Member findMemberA = memberRepository.findById(memberA.getMemberId());
        Member findMemberB = memberRepository.findById(memberEx.getMemberId());
        assertThat(findMemberA.getMoney()).isEqualTo(8000);
        assertThat(findMemberB.getMoney()).isEqualTo(10000);
    }
```

처음에 A에서 2000원이 인출되었는데 ex의 닉네임 때문에 예외가 발생해서 2000원을 받지 못했다.

그래서 결국 8000, 10000이라는 결과를 받게되었고 이것으로 일관성을 어기게 되는 현상이 일어나게 되었다.

트랜젝션이 필요해보인다.



# 트랜잭션 - 적용2
- 이번에는 DB 트랜잭션을 사용해서 앞서 발생한 문제점을 해결해보자.
- 애플리케이션에서 트랜잭션을 어떤 계층에 걸어야 할까? 쉽게 이야기해서 트랜잭션을 어디에서 시작하고, 어디에 서 커밋해야할까?

![](https://velog.velcdn.com/images/jckim22/post/c9c1a132-7e14-4069-a93e-37c2c8b4baf5/image.png)
- 트랜잭션은 비즈니스 로직이 있는 서비스 계층에서 시작해야 한다. 비즈니스 로직이 잘못되면 해당 비즈니스 로직 으로 인해 문제가 되는 부분을 함께 롤백해야 하기 때문이다.
- 그런데 트랜잭션을 시작하려면 커넥션이 필요하다. 결국 서비스 계층에서 커넥션을 만들고, 트랜잭션 커밋 이후에 커넥션을 종료해야 한다.
- 애플리케이션에서 DB 트랜잭션을 사용하려면 **트랜잭션을 사용하는 동안 같은 커넥션을 유지**해야한다. 그래야 같 은 세션을 사용할 수 있다.



**커넥션과 세션**

![업로드중..](blob:https://velog.io/d1b40f78-7d1e-443d-a0d9-ccd6ee9ae2bc)

애플리케이션에서 같은 커넥션을 유지하려면 어떻게 해야할까? 가장 단순한 방법은 커넥션을 파라미터로 전달해서 같 은 커넥션이 사용되도록 유지하는 것이다.


먼저 리포지토리가 파라미터를 통해 같은 커넥션을 유지할 수 있도록 파라미터를 추가하자.


```java
@Slf4j
public class MemberRepositoryV2 {

    private final DataSource dataSource;

    public MemberRepositoryV2(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public Member save(Member member) throws SQLException {
        String sql = "insert into member(member_id, money) values (?, ?)";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, member.getMemberId());
            pstmt.setInt(2, member.getMoney());
            pstmt.executeUpdate();
            return member;
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }

    }

    public Member findById(String memberId) throws SQLException {
        String sql = "select * from member where member_id = ?";

        Connection con = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, memberId);

            rs = pstmt.executeQuery();
            if (rs.next()) {
                Member member = new Member();
                member.setMemberId(rs.getString("member_id"));
                member.setMoney(rs.getInt("money"));
                return member;
            } else {
                throw new NoSuchElementException("member not found memberId=" + memberId);
            }

        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, rs);
        }

    }

    public Member findById(Connection con, String memberId) throws SQLException {
        String sql = "select * from member where member_id = ?";

        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try {
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, memberId);

            rs = pstmt.executeQuery();
            if (rs.next()) {
                Member member = new Member();
                member.setMemberId(rs.getString("member_id"));
                member.setMoney(rs.getInt("money"));
                return member;
            } else {
                throw new NoSuchElementException("member not found memberId=" + memberId);
            }

        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            //connection은 여기서 닫지 않는다.
            JdbcUtils.closeResultSet(rs);
            JdbcUtils.closeStatement(pstmt);
        }

    }

    public void update(String memberId, int money) throws SQLException {
        String sql = "update member set money=? where member_id=?";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setInt(1, money);
            pstmt.setString(2, memberId);
            int resultSize = pstmt.executeUpdate();
            log.info("resultSize={}", resultSize);
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }

    }

    public void update(Connection con, String memberId, int money) throws SQLException {
        String sql = "update member set money=? where member_id=?";

        PreparedStatement pstmt = null;

        try {
            pstmt = con.prepareStatement(sql);
            pstmt.setInt(1, money);
            pstmt.setString(2, memberId);
            int resultSize = pstmt.executeUpdate();
            log.info("resultSize={}", resultSize);
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            //connection은 여기서 닫지 않는다.
            JdbcUtils.closeStatement(pstmt);
        }

    }

    public void delete(String memberId) throws SQLException {
        String sql = "delete from member where member_id=?";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, memberId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }

    }

    private void close(Connection con, Statement stmt, ResultSet rs) {
        JdbcUtils.closeResultSet(rs);
        JdbcUtils.closeStatement(stmt);
        JdbcUtils.closeConnection(con);
    }


    private Connection getConnection() throws SQLException {
        Connection con = dataSource.getConnection();
        log.info("get connection={}, class={}", con, con.getClass());
        return con;
    }


}

```

`MemberRepositoryV2` 는 기존 코드와 같고 커넥션 유지가 필요한 다음 두 메서드가 추가되었다. 참고로 다음 두 메 서드는 계좌이체 서비스 로직에서 호출하는 메서드이다.
`findById(Connection con, String memberId)`
`update(Connection con, String memberId, int money)`

**주의 - 코드에서 다음 부분을 주의해서 보자!**
1. 커넥션 유지가 필요한 두 메서드는 파라미터로 넘어온 커넥션을 사용해야 한다. 따라서 `con =
getConnection()` 코드가 있으면 안된다.
2. 커넥션 유지가 필요한 두 메서드는 리포지토리에서 커넥션을 닫으면 안된다. 커넥션을 전달 받은 리포지토
리 뿐만 아니라 이후에도 커넥션을 계속 이어서 사용하기 때문이다. 이후 서비스 로직이 끝날 때 트랜잭션을 종료하고 닫아야 한다.

이제 트랜젝션이 존재하는 서비스 로직을 작성해보자

#### MemberServiceV2
```java
@Slf4j
@RequiredArgsConstructor
public class MemberServiceV2 {

    private final DataSource dataSource;
    private final MemberRepositoryV2 memberRepository;

    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        Connection con = dataSource.getConnection();
        try {
            con.setAutoCommit(false);//트랜잭션 시작
            //비즈니스 로직
            bizLogic(con, fromId, toId, money);
            con.commit(); //성공시 커밋
        } catch (Exception e) {
            con.rollback(); //실패시 롤백
            throw new IllegalStateException(e);
        } finally {
            release(con);
        }

    }

    private void bizLogic(Connection con, String fromId, String toId, int money) throws SQLException {
        Member fromMember = memberRepository.findById(con, fromId);
        Member toMember = memberRepository.findById(con, toId);

        memberRepository.update(con, fromId, fromMember.getMoney() - money);
        validation(toMember);
        memberRepository.update(con, toId, toMember.getMoney() + money);
    }

    private void validation(Member toMember) {
        if (toMember.getMemberId().equals("ex")) {
            throw new IllegalStateException("이체중 예외 발생");
        }
    }

    private void release(Connection con) {
        if (con != null) {
            try {
                con.setAutoCommit(true); //커넥션 풀 고려
                con.close();
            } catch (Exception e) {
                log.info("error", e);
            }
        }
    }
}

```

1. 커넥션을 연결하며 세션을 획득했다.
2. 수동 커밋모드로 변경하면서 트랜잭션이 시작
3. 비즈니스 로직을 수행
4. 성공하면 commit 예외가 발생하면 rollback
5. 마지막엔 항상 release
>근데 커넥션이 다시 풀에 들어갈 때 설정이 그대로 들어가기 때문에 다시 기본값인 자동커밋 모드로 변경해주는 것이 좋다.


### Test
```java
    @DisplayName("이체중 예외 발생")
    void accountTransferEx() throws SQLException {
        //given
        Member memberA = new Member(MEMBER_A, 10000);
        Member memberEx = new Member(MEMBER_EX, 10000);
        memberRepository.save(memberA);
        memberRepository.save(memberEx);

        //when
        assertThatThrownBy(() -> memberService.accountTransfer(memberA.getMemberId(), memberEx.getMemberId(), 2000))
                .isInstanceOf(IllegalStateException.class);

        //then
        Member findMemberA = memberRepository.findById(memberA.getMemberId());
        Member findMemberB = memberRepository.findById(memberEx.getMemberId());
        assertThat(findMemberA.getMoney()).isEqualTo(10000);
        assertThat(findMemberB.getMoney()).isEqualTo(10000);
    }

}

```

memberEx가 비즈니스 로직 도중 예외가 발생하게 되어서 rollback이 된다.
그래서 memberA의 돈이 8000원에서 다시 1만원으로 돌아오게 된다.
