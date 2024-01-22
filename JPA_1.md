
![](https://velog.velcdn.com/images/jckim22/post/6c0c3d28-e386-4d30-b067-18cb23212d86/image.png)

>주의
• 엔티티 매니저 팩토리는 하나만 생성해서 애플리케이션 전체에
서 공유
• 엔티티 매니저는 쓰레드간에 공유X (사용하고 버려야 한다).
• JPA의 모든 데이터 변경은 트랜잭션 안에서 실행

### JPQL 소개
• 가장 단순한 조회 방법
• EntityManager.find()
• 객체 그래프 탐색(a.getB().getC())
• 나이가 18살 이상인 회원을 모두 검색하고 싶다면?

### JPQL 
• JPA를 사용하면 엔티티 객체를 중심으로 개발
• 문제는 검색 쿼리
• 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색
• 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능
• 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요

• JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공
• SQL과 문법 유사, SELECT, FROM, WHERE, GROUP BY,
HAVING, JOIN 지원
• JPQL은 엔티티 객체를 대상으로 쿼리
• SQL은 데이터베이스 테이블을 대상으로 쿼리

• 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리
• SQL을 추상화해서 특정 데이터베이스 SQL에 의존X
• JPQL을 한마디로 정의하면 객체 지향 SQL


```java
List<Member> result = em.createQuert("select m from Member as m", Member.class)
			.setFirstResult(5)
            .setMaxResults(8)
            .getResultList();
```

# 영속성 컨텍스트

### 엔티티 매니저 팩토리와 엔티티 매니저

![](https://velog.velcdn.com/images/jckim22/post/c8e558ca-f324-4c88-9a1c-07343f084bbf/image.png)

### 영속성 컨텍스트

- JPA를 이해하는데 가장 중요한 용어
- “엔티티를 영구 저장하는 환경”이라는 뜻
- EntityManager.persist(entity); -> DB가 아니라 컨텍스트에 저장 -> Commit할 때 DB에 저장

### 엔티티 매니저? 영속성 컨텍스트?

- 영속성 컨텍스트는 논리적인 개념
- 눈에 보이지 않는다.
- 엔티티 매니저를 통해서 영속성 컨텍스트에 접근


#### J2SE 환경
엔티티 매니저와 영속성 컨텍스트가 1:1

![](https://velog.velcdn.com/images/jckim22/post/ac72d1e4-f63e-4b37-af52-48639e9ece8f/image.png)

#### J2EE, 스프링 프레임워크 같은 컨테이너 환경 
엔티티 매니저와 영속성 컨텍스트가 N:1
![](https://velog.velcdn.com/images/jckim22/post/c2f45482-69bc-40ec-b2c1-3b4e5e8a3a31/image.png)



# 엔티티의 생명주기

- 비영속 (new/transient)
영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
- 영속 (managed)
영속성 컨텍스트에 관리되는 상태
- 준영속 (detached)
영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제 (removed)
삭제된 상태

![](https://velog.velcdn.com/images/jckim22/post/c2bb722e-5976-41e2-bbfe-6e632e6c071e/image.png)

## 비영속

![](https://velog.velcdn.com/images/jckim22/post/b3f85451-9159-42e3-8b7c-e303f7f5532a/image.png)


```java
//객체를 생성한 상태(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");
```

## 영속


![](https://velog.velcdn.com/images/jckim22/post/5e599b3e-8bdc-4d8a-9c23-bc4381b3cbd7/image.png)

```java
//객체를 생성한 상태(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername(“회원1”);
EntityManager em = emf.createEntityManager();
em.getTransaction().begin();
//객체를 저장한 상태(영속)
em.persist(member);
```

## 준영속
```java
//회원 엔티티를 영속성 컨텍스트에서 분리, 준영속 상태
em.detach(member);
```

## 삭제
```java
//객체를 삭제한 상태(삭제)
em.remove(member);
```

# 영속성 컨텍스트의 이점

- 1차 캐시
- 동일성(identity) 보장
- 트랜잭션을 지원하는 쓰기 지연 (transactional write-behind)
- 변경 감지(Dirty Checking)
- 지연 로딩(Lazy Loading)

## 엔티티 조회, 1차 캐시

![](https://velog.velcdn.com/images/jckim22/post/7ea0c8b4-0b42-4311-95a0-9a9d75f558c7/image.png)

1차 캐시에서 조회
```java
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");
//1차 캐시에 저장됨
em.persist(member);
//1차 캐시에서 조회
Member findMember = em.find(Member.class, "member1");
```
![](https://velog.velcdn.com/images/jckim22/post/20760d31-2715-4773-b3fa-42255bce5d0e/image.png)



데이터베이스에서 조회
```java
Member findMember2 = em.find(Member.class, "member2");
```

![](https://velog.velcdn.com/images/jckim22/post/188298f6-cb59-438e-8840-f184987bd26d/image.png)


영속 엔티티의 동일성 보장
```java
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");
System.out.println(a == b); //동일성 비교 true
```

1차 캐시로 반복 가능한 읽기(REPEATABLE READ) 등급의 트랜잭 션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공

## 엔티티 등록 트랜잭션을 지원하는 쓰기 지연

```java
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();
//엔티티 매니저는 데이터 변경시 트랜잭션을 시작해야 한다.
transaction.begin(); // [트랜잭션] 시작
em.persist(memberA);
em.persist(memberB);
//여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.
//커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.
transaction.commit(); // [트랜잭션] 커밋
```

![](https://velog.velcdn.com/images/jckim22/post/16761254-ef94-43a2-a770-d1331db7079b/image.png)
![](https://velog.velcdn.com/images/jckim22/post/0cad9e45-826b-4b16-8770-6d0600d3881d/image.png)
![](https://velog.velcdn.com/images/jckim22/post/67412ca8-e152-4ea2-8979-54d7aa8f227e/image.png)

## 엔티티 수정 - 변경 감지

```java
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();
transaction.begin(); // [트랜잭션] 시작
// 영속 엔티티 조회
Member memberA = em.find(Member.class, "memberA");
// 영속 엔티티 데이터 수정
memberA.setUsername("hi");
memberA.setAge(10);
//em.update(member) 이런 코드가 있어야 하지 않을까?
transaction.commit(); // [트랜잭션] 커밋
```

![](https://velog.velcdn.com/images/jckim22/post/8302b7f6-7580-48c7-86ac-85ab339041b1/image.png)

## 엔티티 삭제
```java
//삭제 대상 엔티티 조회
Member memberA = em.find(Member.class, “memberA");
em.remove(memberA);//엔티티 삭제
```

## 플러시
영속성 컨텍스트의 변경내용을 데이터베이스에 반영

### 플러시 발생
- 변경 감지
- 수정된 엔티티 쓰기 지연 SQL 저장소에 등록
- 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송  (등록, 수정, 삭제 쿼리)


### 영속성 컨텍스트를  플러시하는 방법

- em.flush() - 직접 호출
- 트랜잭션 커밋 - 플러시 자동 호출
- JPQL 쿼리 실행 - 플러시 자동 호출

### JPQL 쿼리 실행시 플러시가 자동 으로 호출되는 이유

```java
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);
//중간에 JPQL 실행
query = em.createQuery("select m from Member m", Member.class);
List<Member> members= query.getResultList();
```

### 플러시 모드 옵션
```java
em.setFlushMode(FlushModeType.COMMIT)
```
- FlushModeType.AUTO
커밋이나 쿼리를 실행할 때 플러시 (기본값)
- FlushModeType.COMMIT
커밋할 때만 플러시

### 플러시는!
- 영속성 컨텍스트를 비우지 않음
- 영속성 컨텍스트의 변경내용을 데이터베이스에 동기화
- 트랜잭션이라는 작업 단위가 중요 -> 커밋 직전에만 동기화 하면 됨

# 준영속 상태

- 영속 -> 준영속
- 영속 상태의 엔티티가 영속성 컨텍스트에서 분리(detached)
- 영속성 컨텍스트가 제공하는 기능을 사용 못함

## 준영속 상태로 만드는 방법
- em.detach(entity)
특정 엔티티만 준영속 상태로 전환

- em.clear() 
영속성 컨텍스트를 완전히 초기화

- em.close() 
영속성 컨텍스트를 종료

