이전 데이터 중심 매핑에서 객체 중심의 매핑을 위해 연관관계 매핑을 알아보자.

# 객체를 테이블에 맞추어 모델링

![](https://velog.velcdn.com/images/jckim22/post/ef8a63fe-d88e-493a-8180-e4b254681613/image.png)

## 객체를 테이블에 맞추어 모델링 (참조 대신에 외래 키를 그대로 사용)
![](https://velog.velcdn.com/images/jckim22/post/434607ed-b3ca-4400-9af3-c30fae13806c/image.png)

## 객체를 테이블에 맞추어 모델링 (외래 키 식별자를 직접 다룸)
![](https://velog.velcdn.com/images/jckim22/post/08204d89-c5d0-41b7-a298-15e4e4afd404/image.png)

식별자로 다시 조회, 객체 지향적인 방법은 아니다.
![](https://velog.velcdn.com/images/jckim22/post/fa792816-47b3-4daa-97c5-89c29527f49c/image.png)

# 객체를 테이블에 맞추어 데이터 중심으로 모델링하면, 협력 관계를 만들 수 없다

- 테이블은 외래 키로 조인을 사용해서 연관된 테이블을 찾는다. 
- 객체는 참조를 사용해서 연관된 객체를 찾는다.
- 테이블과 객체 사이에는 이런 큰 간격이 있다.



# 단방향 연관관계

### 객체 지향 모델링 (객체 연관관계 사용)

![](https://velog.velcdn.com/images/jckim22/post/077f25c0-80d4-458f-899e-3f1e94564011/image.png)
![](https://velog.velcdn.com/images/jckim22/post/48f48ed3-f058-41b0-aeb4-cba765b69a34/image.png)
![](https://velog.velcdn.com/images/jckim22/post/9b7f78a3-2e8e-4241-9a52-a93f877663b0/image.png)

### 연관관계 저장
```java
//팀 저장 
Team team = new Team(); 
team.setName("TeamA");  em.persist(team); 
 
//회원 저장 
Member member = new Member();  member.setName("member1"); 
member.setTeam(team); //단방향 연관관계 설정, 참조 저장  em.persist(member); 
```
### 연관관계 조회
```java
//조회 
Member findMember = em.find(Member.class, member.getId());
//참조를 사용해서 연관관계 조회 
Team findTeam = findMember.getTeam();
```
### 연관관계 수정
```java
// 새로운 팀B 
Team teamB = new Team();  teamB.setName("TeamB");  em.persist(teamB); 
 
// 회원1에 새로운 팀B 설정  member.setTeam(teamB); 
```

# 양방향 매핑

![](https://velog.velcdn.com/images/jckim22/post/1ebb3bfa-14f9-451c-93af-012fb2dd874f/image.png)

테이블은 방향이라는 개념이 없기 때문에 테이블은 단방향일 때와 그대로이다.
그러나 객체 중심일때는 ?

```java
@Entity 
public class Member {
 
private Long id;   
@Column(name = "USERNAME")  private String name;  private int age; 
 
@ManyToOne 
@JoinColumn(name = "TEAM_ID") 
private Team team; 
```

Member 엔티티는 단방향과 당연하게도 동일하다.

![](https://velog.velcdn.com/images/jckim22/post/08026633-12de-4a2d-8c5a-cc47fc694f7b/image.png)

Team 엔티티는 많은 member를 받을 콜렉션을 추가하고 일대다 이므로 @OneToMany를 달아준다.
mappedBy로 주인인 엔티티에서 어떤 변수에 들어갈지 매핑해준다.

>의존관계의 주인이 JoinColumn을 달고 있는데 다 쪽이 주인인게 맞다.

```java
//조회 
Team findTeam = em.find(Team.class, team.getId());
int memberSize = findTeam.getMembers().size();//역방향 조회 
```
# 연관관계의 주인과 mappedBy
- mappedBy = JPA의 멘탈붕괴 난이도
- 객체와 테이블간에 연관관계를 맺는 차이를 이해해야 한다.

## 객체와 테이블이 관계를 맺는 차이

- 객체 연관관계 = 2개
  - 회원 -> 팀 연관관계 1개(단방향)
   - 팀 -> 회원 연관관계 1개(단방향) 
 - 테이블 연관관계 = 1개
   - 회원 <-> 팀의 연관관계 1개(양방향)

![](https://velog.velcdn.com/images/jckim22/post/d2d437b0-bd06-4a61-bfe1-75951d1c1215/image.png)

## 객체의 양방향 관계
- 객체의 양방향 관계는 사실 양방향 관계가 아니라 서로 다른 단뱡향 관계 2개다.
-  객체를 양방향으로 참조하려면 단방향 연관관계를 2개 만들어야 한다.

![](https://velog.velcdn.com/images/jckim22/post/5e763a1e-9aeb-460d-bea3-7663906d2c02/image.png)

## 테이블의 양방향 연관관계
- 테이블은 외래 키 하나로 두 테이블의 연관관계를 관리
- MEMBER.TEAM_ID 외래 키 하나로 양방향 연관관계 가짐  (양쪽으로 조인할 수 있다.)

```sql
SELECT *
FROM MEMBER M
JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
SELECT *
FROM TEAM T
JOIN MEMBER M ON T.TEAM_ID = M.TEAM_ID
```

# 둘 중 하나로 외래 키를 관리해야 한다

![](https://velog.velcdn.com/images/jckim22/post/eff8e071-a697-4bf8-99fb-b76fc3615e51/image.png)

멤버 팀을 하나 업데이트 하고 싶은데 멤버의 팀을 업데이트해야할지 Team의 members를 업데이트 해야할지 고민이다.

# 연관관계의 주인(Owner)

### 양방향 매핑 규칙

![](https://velog.velcdn.com/images/jckim22/post/8e32c4f6-979c-47ba-9540-2574f03c1391/image.png)

### 누구를 주인으로?
- 외래 키가 있는 있는 곳을 주인으로 정해라
- 여기서는 Member.team이 연관관계의 주인

![](https://velog.velcdn.com/images/jckim22/post/31ec35ee-3200-4925-a28f-a58718f81f79/image.png)

## 양방향 매핑시 가장 많이 하는 실수 (연관관계의 주인에 값을 입력하지 않음)
```java
Team team = new Team();  team.setName("TeamA");  em.persist(team); 
Member member = new Member();  member.setName("member1"); 
//역방향(주인이 아닌 방향)만 연관관계 설정
team.getMembers().add(member); 
em.persist(member); 
```
![](https://velog.velcdn.com/images/jckim22/post/10d6cca1-eafb-4b84-8986-500894a8aa7b/image.png)

```java
Team team = new Team();  team.setName("TeamA");  em.persist(team); 
Member member = new Member();  member.setName("member1"); 
team.getMembers().add(member); 
//연관관계의 주인에 값 설정
member.setTeam(team); //** 
em.persist(member); 
```

- 순수 객체 상태를 고려해서 항상 양쪽에 값을 설정하자
- 연관관계 편의 메소드를 생성하자
- 양방향 매핑시에 무한 루프를 조심하자
- 예: toString(), lombok, JSON 생성 라이브러리

## 연관관계 편의 메서드
 ```java
public void setTeam(Team team) {
	this.team = team;
    team.getMembers().add(this);
}
```

순수 객체를 설정하는 코드를 까먹고 실수할 수도 있고 편의를 위해 주인 쪽에서 세팅을 할 때 연관관계인 콜렉션도 같이 세팅한다.


# 양방향 매핑 정리
- 단방향 매핑만으로도 이미 연관관계 매핑은 완료
- 양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추 가된 것 뿐
- JPQL에서 역방향으로 탐색할 일이 많음
- 단방향 매핑을 잘 하고 양방향은 필요할 때 추가해도 됨  (테이블에 영향을 주지 않음)

# 연관관계의 주인을 정하는 기준
- 비즈니스 로직을 기준으로 연관관계의 주인을 선택하면 안됨
- 연관관계의 주인은 외래 키의 위치를 기준으로 정해야함

# 테이블 구조
![](https://velog.velcdn.com/images/jckim22/post/451716a9-24c1-4c22-b57c-b87c94181dd9/image.png)

# 변경된 객체 구조

- 참조를 사용하도록 변경

![](https://velog.velcdn.com/images/jckim22/post/122b263c-4db6-4e1c-b8f2-d7e4e147af79/image.png)



# 일대일 관계

#### 일대일 관계
• 일대일 관계는 그 반대도 일대일
• 주 테이블이나 대상 테이블 중에 외래 키 선택 가능
• 주 테이블에 외래 키
• 대상 테이블에 외래 키
• 외래 키에 데이터베이스 유니크(UNI) 제약조건 추가

![](https://velog.velcdn.com/images/jckim22/post/d9184958-6731-4a5f-92d5-e48d3ac54260/image.png)
• 다대일(@ManyToOne) 단방향 매핑과 유사

### 일대일: 주 테이블에 외래 키 양방향

![](https://velog.velcdn.com/images/jckim22/post/0b50aeaf-b0a4-4563-a30a-161c87579ed9/image.png)

• 다대일 양방향 매핑 처럼 외래 키가 있는 곳이 연관관계의 주인
• 반대편은 mappedBy 적용

 #### 주 테이블에 외래 키
• 주 객체가 대상 객체의 참조를 가지는 것 처럼 주 테이블에 외래 키를 두고 대상 테이블을 찾음
• 객체지향 개발자 선호
• JPA 매핑 편리
• 장점: 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능
• 단점: 값이 없으면 외래 키에 null 허용

####  대상 테이블에 외래 키
• 대상 테이블에 외래 키가 존재
• 전통적인 데이터베이스 개발자 선호
• 장점: 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조 유지
• 단점: 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩됨(프록시는 뒤에서 설명)

### 다대다

- 다대일로 해결할 것

#### @JoinColumn


![](https://velog.velcdn.com/images/jckim22/post/d3393605-71cb-4520-bc92-85c8b1431a87/image.png)

#### @ManyToOne 

![](https://velog.velcdn.com/images/jckim22/post/4dcd5018-acef-4a32-bde9-3ab71f087e9a/image.png)


#### @OneToMany 

![](https://velog.velcdn.com/images/jckim22/post/aaf5a9b2-86ec-4429-aa0a-e5b840c198b0/image.png)
