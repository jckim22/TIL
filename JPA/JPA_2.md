# @Entity
@Entity가 붙은 클래스는 JPA가 관리, 엔티티라 한다. 
• JPA를 사용해서 테이블과 매핑할 클래스는 @Entity 필수

>• 주의
final 클래스, enum, interface, inner 클래스 사용 X
기본 생성자 필수(파라미터가 없는 public 또는 protected 생성자) 저장할 필드에 final 사용 X

## @Entity 속성 정리
속성: name
- JPA에서 사용할 엔티티 이름을 지정한다.
- 기본값: 클래스 이름을 그대로 사용(예: Member)
- 같은 클래스 이름이 없으면 가급적 기본값을 사용한다.

## @Table
@Table은 엔티티와 매핑할 테이블 지정

![](https://velog.velcdn.com/images/jckim22/post/764cd0a8-de03-42f9-af03-18356fa889d1/image.png)


# 데이터베이스 스키마 자동 생성

- DDL을 애플리케이션 실행 시점에 자동 생성
- 테이블 중심 -> 객체 중심
- 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한
DDL 생성
- 이렇게 생성된 DDL은 개발 장비에서만 사용
- 생성된 DDL은 운영서버에서는 사용하지 않거나, 적절히 다듬은 후 사용

# 데이터베이스 스키마 자동 생성 - 속성

![](https://velog.velcdn.com/images/jckim22/post/d417f112-190a-4607-a1a8-112cc2793b08/image.png)

# 데이터베이스 스키마 자동 생성 - 주의

![](https://velog.velcdn.com/images/jckim22/post/19390ae0-14da-4d3a-9d23-35f597df8397/image.png)

## DDL 생성 기능
- 제약조건 추가: 회원 이름은 필수, 10자 초과X
```java
@Column(nullable = false, length = 10)
```
-  유니크 제약조건 추가
```java
@Table(uniqueConstraints = {@UniqueConstraint( name = "NAME_AGE_UNIQUE",
columnNames = {"NAME", "AGE"} )})

```

- DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고
JPA의 실행 로직에는 영향을 주지 않는다.


>운영 서버에서는 DDL로 실행한 SQL을 가져와서 다듬거나 직접 짠다.

# 필드와 컬럼 매핑
#### 요구사항 추가
1. 회원은 일반회원과 관리자로 구분해야한다.
2. 회원 가입일과 수정일이 있어야한다.
3. 회원을 설명할 수 있는 필드가 있어야 한다.이 필드는 길이 제한이 없다.

```java
import javax.persistence.*;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.Date;
@Entity
public class Member {
@Id
private Long id;
@Column(name = "name")
private String username;
private Integer age;
@Enumerated(EnumType.STRING)
private RoleType roleType;
@Temporal(TemporalType.TIMESTAMP)
private Date createdDate;
@Temporal(TemporalType.TIMESTAMP)
private Date lastModifiedDate;
@Lob
private String description;
}
```
## 매핑 어노테이션 정리

![](https://velog.velcdn.com/images/jckim22/post/6226d357-e82a-47b5-9c3e-3e5d92990042/image.png)

### @Column

![](https://velog.velcdn.com/images/jckim22/post/6b16c55e-3565-4b9b-b3d2-2ae8d47cae3d/image.png)


### @Enumerated
자바 enum 타입을 매핑할 때 사용 주의!
ORDINAL 사용X

### @Temporal
날짜 타입(java.util.Date, java.util.Calendar)을 매핑할 때 사용
참고: LocalDate, LocalDateTime을 사용할 때는 생략 가능(최신 하이버네이트 지원)

### @Lob
- 데이터베이스 BLOB, CLOB 타입과 매핑
@Lob에는 지정할 수 있는 속성이 없다.
매핑하는 필드 타입이 문자면 CLOB 매핑, 나머지는 BLOB 매핑

CLOB: String, char[], java.sql.CLOB 
BLOB: byte[], java.sql. BLOB



# 기본 키 매핑

기본 키 매핑 어노테이션
```java
@Id @GeneratedValue
```

![](https://velog.velcdn.com/images/jckim22/post/566fe4c7-69ce-4481-a9b5-49dc606beb9a/image.png)

## 기본 키 매핑 방법
- 직접 할당: @Id만 사용
- 자동 생성(@GeneratedValue)
  - IDENTITY: 데이터베이스에 위임, MYSQL
  - SEQUENCE: 데이터베이스 시퀀스 오브젝트 사용, ORACLE
    - @SequenceGenerator 필요
  - TABLE: 키 생성용 테이블 사용, 모든 DB에서 사용
  - @TableGenerator 필요
  - AUTO: 방언에 따라 자동 지정, 기본값

### IDENTITY 전략 - 특징
• 기본 키 생성을 데이터베이스에 위임
• 주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용  (예: MySQL의 AUTO_ INCREMENT)
• JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL 실행
• AUTO_ INCREMENT는 데이터베이스에 INSERT SQL을 실행
한 이후에 ID 값을 알 수 있음
• IDENTITY 전략은 em.persist() 시점에 즉시 INSERT SQL 실행 하고 DB에서 식별자를 조회

### SEQUENCE 전략 - 특징
- 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트(예: 오라클 시퀀스)
- 오라클, PostgreSQL, DB2, H2 데이터베이스에서 사용

### SEQUENCE 전략 - 매핑
![](https://velog.velcdn.com/images/jckim22/post/b175bafb-b27c-4891-abed-81df321a0741/image.png)


#### SEQUENCE - @SequenceGenerator

![](https://velog.velcdn.com/images/jckim22/post/3f9b6497-614d-4653-bfc8-40a141cde922/image.png)

### TABLE 전략
키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략
장점: 모든 데이터베이스에 적용 가능
단점: 성능

![](https://velog.velcdn.com/images/jckim22/post/f670e5bf-07a0-47a3-8f4d-ab5a3a2cbf9a/image.png)
![](https://velog.velcdn.com/images/jckim22/post/3a5b96f2-a7a4-4547-abc5-2e28f1f77275/image.png)

## 권장하는 식별자 전략
- 기본 키 제약 조건: null 아님, 유일, 변하면 안된다.
- 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다. 대리키(대체키)를 사용하자.
- 예를 들어 주민등록번호도 기본 키로 적절하기 않다.
- 권장: Long형 + 대체키 + 키 생성전략 사용



# 실습 - 테이블 설계

![](https://velog.velcdn.com/images/jckim22/post/56205c27-129a-4ddf-bacf-7c99ba4fe7c1/image.png)
![](https://velog.velcdn.com/images/jckim22/post/c8faf84d-5206-48d6-8471-479071a2b095/image.png)

여기서 Order를 보면 Long 타입이고 find를 하게 되면 Member객체가 아닌 Long을 가지고 온다.
그럼 Member를 가져오려면 또 find를 해야하는데 이건 테이블 중심 설계이기 때문에 객체스럽지가 않다.

## 데이터 중심 설계의 문제점
- 현재 방식은 객체 설계를 테이블 설계에 맞춘 방식 
- 테이블의 외래키를 객체에 그대로 가져옴
- 객체 그래프 탐색이 불가능 
- 참조가 없으므로 UML도 잘못됨

이제 연관관계 매핑을 사용해봐야한다.




