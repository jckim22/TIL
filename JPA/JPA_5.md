# Member를 조회할 때 Team도 함께 조회해야 할까?

이를 해결하기 위해서 즉시 로딩이 아닌 지연 로딩을 사용해야하는데 이는 프록시를 이해하고 있어야함

# 프록시 기초
• em.find() vs em.getReference()
• em.find(): 데이터베이스를 통해서 실제 엔티티 객체 조회
• em.getReference(): 데이터베이스 조회를 미루는 가짜(프록시)를 가져옴
	-> 사용하는 시점에 쿼리를 날림
    
![](https://velog.velcdn.com/images/jckim22/post/874d978f-96f2-4171-99dc-0935e1022256/image.png)

# 프록시 특징
• 실제 클래스를 상속 받아서 만들어짐
• 실제 클래스와 겉 모양이 같다.
• 사용하는 입장에서는 진짜 객체인지
프록시 객체인지 구분하지 않고
사용하면 됨(이론상)

![](https://velog.velcdn.com/images/jckim22/post/7ab69921-6b36-4bfe-b933-13c146b232a4/image.png)


• 프록시 객체는 실제 객체의 참조(target)를 보관
• 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드 호출

![](https://velog.velcdn.com/images/jckim22/post/73601a06-6633-4cc8-ba7f-4233ba920f71/image.png)

# 프록시 객체의 초기화
```java
Member member = em.getReference(Member.class, “id1”);
member.getName();
```
![](https://velog.velcdn.com/images/jckim22/post/55c3ac98-484f-4023-91fe-6a8ac3261dea/image.png)

# 프록시의 특징
• 프록시 객체는 처음 사용할 때 한 번만 초기화
• 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것은 아님, 초
기화되면 프록시 객체를 통해서 실제 엔티티에 접근 가능
• 프록시 객체는 원본 엔티티를 상속받음, 따라서 타입 체크시 주의해야함 (== 비
교 실패, 대신 instance of 사용)
• 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference()를 호출해
도 실제 엔티티 반환
• 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화하면
문제 발생
(하이버네이트는 org.hibernate.LazyInitializationException 예외를 터트림)


# 즉시 로딩과 지연 로딩
### Member를 조회할 때 Team도 함께 조회해야 할까?

#### 지연 로딩
단순히 member 정보만 사용하는 비즈니스 로직
```java
println(member.getName());
```
![](https://velog.velcdn.com/images/jckim22/post/a5afa078-3c82-430d-876c-065131872f27/image.png)


- member를 사용할 때 member의 실제 엔티티를 불러오고 team은 프록시 객체만 불러온다.
- team을 사용할 때 프록시 객체는 진짜 객체를 호출한다.(쿼리가 나간다)


# 프록시와 즉시로딩 주의
• 가급적 지연 로딩만 사용(특히 실무에서)
• 즉시 로딩을 적용하면 예상하지 못한 SQL이 발생
• 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.
• **@ManyToOne, @OneToOne은 기본이 즉시 로딩**
**-> LAZY로 설정**
• **@OneToMany, @ManyToMany는 기본이 지연 로딩**


# 지연 로딩 활용
• Member와 Team은 자주 함께 사용 -> 즉시 로딩
• Member와 Order는 가끔 사용 -> 지연 로딩
• Order와 Product는 자주 함께 사용 -> 즉시 로딩


![](https://velog.velcdn.com/images/jckim22/post/722952b5-baf1-4ccc-ab02-88524f22fb32/image.png)

![](https://velog.velcdn.com/images/jckim22/post/46b29f1b-3c91-4ca7-aa4a-56619dfbbe65/image.png)
![](https://velog.velcdn.com/images/jckim22/post/d262aa2c-cccc-44f2-a5d6-2f09a2f430a8/image.png)

# 지연 로딩 활용 - 실무
**• 모든 연관관계에 지연 로딩을 사용해라!**
**• 실무에서 즉시 로딩을 사용하지 마라!**
• JPQL fetch 조인이나, 엔티티 그래프 기능을 사용해라!
(뒤에서 설명)
• 즉시 로딩은 상상하지 못한 쿼리가 나간다.

# 영속성 전이: CASCADE

- 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들도 싶을 때
-  예: 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장.
```java
@OneToMany(mappedBy="parent", cascade=CascadeType.PERSIST)
```

# 영속성 전이: CASCADE - 주의!
• 영속성 전이는 연관관계를 매핑하는 것과 아무 관련이 없음
• 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함
을 제공할 뿐

# CASCADE의 종류
• ALL: 모두 적용
• PERSIST: 영속
• REMOVE: 삭제
• MERGE: 병합
• REFRESH: REFRESH
• DETACH: DETACH

# 고아 객체
• 고아 객체 제거: 부모 엔티티와 연관관계가 끊어진 자식 엔티티
를 자동으로 삭제
```java
orphanRemoval = true
Parent parent1 = em.find(Parent.class, id);
```
```java
parent1.getChildren().remove(0);
//자식 엔티티를 컬렉션에서 제거
```
```sql
DELETE FROM CHILD WHERE ID=?
```

# 영속성 주기 + 고아 객체, 생명주기

• CascadeType.ALL + orphanRemoval=true
• 스스로 생명주기를 관리하는 엔티티는 em.persist()로 영속화,
em.remove()로 제거
• 두 옵션을 모두 활성화 하면 부모 엔티티를 통해서 자식의 생명
주기를 관리할 수 있음
• 도메인 주도 설계(DDD)의 Aggregate Root개념을 구현할 때
유용

# 적용

### 글로벌 페치 전략 설정
• 모든 연관관계를 지연 로딩으로
• @ManyToOne, @OneToOne은 기본이 즉시 로딩이므로 지연
로딩으로 변경

### 영속성 전이 설정
• Order -> Delivery를 영속성 전이 ALL 설정
• Order -> OrderItem을 영속성 전이 ALL 설정

