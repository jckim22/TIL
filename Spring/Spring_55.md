## QUERY의 문제점
•QUERY는 문자, Type-check 불가능
•실행하기 전까지 작동여부 확인 불가

## 에러는 크게 2가지
•컴파일 에러 (좋은 에러)
•런타임 에러 (나쁜 에러)

## SQL, JPQL
•만약 SQL이 클래스처럼 타입이 있고 자바 코드로 작성 할 수 있다면? 
•type-safe

## Type-safe
•컴파일시 에러 체크 가능 
•Code-assistant x 100!!! 
•CTRL + SPACE + . (DOT)


# QueryDSL
•쿼리를 Java로 type-safe하게 개발할 수 있게 지원하는 프레임워크
•주로 JPA 쿼리(JPQL)에 사용

### 질문 : 사람을 찾아보자
•20~40살
•성 = 김씨 
•나이 많은 순서
•3명을 출력하라.


## JPA에서 QUERY 방법은 크게 3가지
•1. JPQL(HQL)
•2. Criteria API
•3. MetaModel Criteria API(type-safe)

### 1. JPQL(HQL)
```

@Test
public void jpql() {
String query =
"select m from Member m " +
"where m.age between 20 and 40 " + " and m.name like '김%' " +
"order by m.age desc";
List<Member> resultList = entityManager.createQuery(query, Member.class)
}
```


•장점 : SQL QUERY와 비슷해서 금방 익숙해짐
•단점 : type-safe 아님 동적쿼리 생성이 어려움


### 2. Criteria API
```java
@Test
public void jpaCriteriaQuery() {
CriteriaBuilder cb = entityManager.getCriteriaBuilder(); CriteriaQuery<Member> cq = cb.createQuery(Member.class); Root<Member> root = cq.from(Member.class);
Path<Integer> age = root.get("age"); Predicate between = cb.between(age, 20,40);
Path<String> path = root.get("name"); Predicate like = cb.like(path, "김%");
CriteriaQuery<Member> query = cq.where( cb.and(between, like) ); query.orderBy(cb.desc(age));
List<Member> resultList = entityManager.createQuery(query).setMaxResults(3).getResultList();
}
```



•장점 : 동적쿼리 생성이 쉬움??
•단점
•1. type-safe 아님
•2. 너무 너무 너무 복잡함
•3. 알아야 할게 너무 많음

### 3. MetaModel Criteria API (type-safe)
•root.get("age") -> root.get(Member_.age) •Criteria API + MetaModel
•Criteria API와 거의 동일
•type-safe
•복잡하긴 마찬가지


## QueryDSL

QueryDSL 분석
•Domain(도메인) •Specific(특화) •Language(언어)

### DSL
•도메인 + 특화 + 언어
•특정한 도메인에 초점을 맞춘 제한적인 표현력을 가진 컴퓨터 프로그래밍 언어 
•특징 : 단순, 간결, 유창

### QueryDSL
•쿼리 + 도메인 + 특화 + 언어
•쿼리에 특화된 프로그래밍 언어 
•단순, 간결, 유창
•다양한 저장소 쿼리 기능 통합

### 데이터 쿼리 기능 추상화

![](https://velog.velcdn.com/images/jckim22/post/3bb3740e-70a3-4d77-ba8a-728d5e199a9c/image.png)


•JPA, MongoDB, SQL 같은 기술들을 위해 type-safe SQL을 만드는 프레임워크

### Type-safe Query Type 생성

![](https://velog.velcdn.com/images/jckim22/post/fe8e88d8-506b-42cf-9049-4845d776daae/image.png)


### 코드생성기
•APT : Annotation Processing Tool 
•@Entity


## QueryDSL-JPA

•Querydsl은 JPA 쿼리(JPQL)을 typesafe 하게 작성하는데 많이 사용됨 
•너를 위해 만들었다!


```java

JPAQueryFactory query = new JPAQueryFactory (entityManager); QMember m = QMember.member;
List<Member> list = query .select(m)
.from(m) 
.where(
m.age.between(20, 40).and(m.name.like("김%")) )
.orderBy(m.age.desc()) .limit(3)
.fetch(m);
```

### 생성된 쿼리
```java
select id, age, name
from MEMBER
where age between 20 and 40 and name like '김%'
order by age desc
limit 3
```

### 작동 방식
![](https://velog.velcdn.com/images/jckim22/post/db5506de-945b-40bf-81b2-825ad3a82b84/image.png)

- •장점
  - •type-safe 
•단순함
•쉬움

- •단점
  - •Q코드 생성을 위한 APT를 설정해야함
  
##  SpringDataJPA + Querydsl
•SpringData프로젝트의 약점은 조회
•Querydsl로 복잡한 조회 기능 보완
•복잡한 쿼리
•동적 쿼리
•단순한 경우 : SpringDataJPA
•복잡한 경우 : Querydsl직접 사용



# Querydsl 세팅
```
//Querydsl 추가
	implementation 'com.querydsl:querydsl-jpa'
	annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jpa"
	annotationProcessor "jakarta.annotation:jakarta.annotation-api"
	annotationProcessor "jakarta.persistence:jakarta.persistence-api"
```


```
//Querydsl 추가, 자동 생성된 Q클래스 gradle clean으로 제거
clean {
delete file('src/main/generated')
}
```

Qitem이라고 하는 객체가 필요하다.

![](https://velog.velcdn.com/images/jckim22/post/7aca3f58-2270-4eb0-85bb-0cb08908b07b/image.png)

여기에 가면 크게 2가지 옵션을 선택할 수 있다. 참고로 옵션은 둘다 같게 맞추어 두자.
1. **Gradle**: Gradle을 통해서 빌드한다.
2. **IntelliJ IDEA**: IntelliJ가 직접 자바를 실행해서 빌드한다.

### 옵션 선택1 - Gradle - Q타입 생성 확인 방법
**Gradle IntelliJ 사용법**
- `Gradle -> Tasks -> build -> clean` 
- `Gradle -> Tasks -> other -> compileJava`
**Gradle 콘솔 사용법**
- `./gradlew clean compileJava`
Q 타입 생성 확인
- `build -> generated -> sources -> annotationProcessor -> java/main` 하위에
  - `hello.itemservice.domain.QItem` 이 생성되어 있어야 한다.
  
  
  >Q타입은 컴파일 시점에 자동 생성되므로 버전관리(GIT)에 포함하지 않는 것이 좋다.
gradle 옵션을 선택하면 Q타입은 `gradle build` 폴더 아래에 생성되기 때문에 여기를 포함하지 않아야 한다. 대부분 `gradle build` 폴더를 git에 포함하지 않기 때문에 이 부분은 자연스럽게 해결된다.

**Q타입 삭제**
`gradle clean` 을 수행하면 `build` 폴더 자체가 삭제된다. 따라서 별도의 설정은 없어도 된다.


### 옵션 선택2 - IntelliJ IDEA - Q타입 생성 확인 방법
`Build -> Build Project` 또는
`Build -> Rebuild` 또는
`main()` , 또는 테스트를 실행하면 된다.

`src/main/generated` 하위에 `hello.itemservice.domain.QItem` 이 생성되어 있어야 한다.
> Q타입은 컴파일 시점에 자동 생성되므로 버전관리(GIT)에 포함하지 않는 것이 좋다.
IntelliJ IDEA 옵션을 선택하면 Q타입은 `src/main/generated` 폴더 아래에 생성되기 때문에 여기를 포함하 지 않는 것이 좋다.

**Q타입 삭제** 
```groovy
//Querydsl 추가, 자동 생성된 Q클래스 gradle clean으로 제거
clean {
     delete file('src/main/generated')
}
```

- `IntelliJ IDEA` 옵션을 선택하면 `src/main/generated` 에 파일이 생성되고, 필요한 경우 Q파일을 직접 삭제 해야 한다.
- `gradle` 에 해당 스크립트를 추가하면 `gradle clean` 명령어를 실행할 때 `src/main/generated` 의 파일도 함 께 삭제해준다.
 
 
 >Querydsl은 이렇게 설정하는 부분이 사용하면서 조금 귀찮은 부분인데, IntelliJ가 버전업 하거나 Querydsl의 Gradle 설정이 버전업 하면서 적용 방법이 조금씩 달라지기도 한다. 그리고 본인의 환경에 따라서 잘 동작하지 않기도
한다. 공식 메뉴얼에 소개 되어 있는 부분이 아니기 때문에, 설정에 수고로움이 있지만 `querydsl gradle` 로 검색하 면 본인 환경에 맞는 대안을 금방 찾을 수 있을 것이다.


 # 적용
 
 **JpaItemRepositoryV3**
 
 ```java
@Repository
@Transactional
public class JpaItemRepositoryV3 implements ItemRepository {

    private final EntityManager em;
    private final JPAQueryFactory query;

    public JpaItemRepositoryV3(EntityManager em) {
        this.em = em;
        this.query = new JPAQueryFactory(em);
    }

    @Override
    public Item save(Item item) {
        em.persist(item);
        return item;
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        Item findItem = em.find(Item.class, updateParam);
        findItem.setItemName(updateParam.getItemName());
        findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }

    @Override
    public Optional<Item> findById(Long id) {
        Item item = em.find(Item.class, id);
        return Optional.ofNullable(item);
    }

    public List<Item> findAllOld(ItemSearchCond cond) {

        String itemName = cond.getItemName();
        Integer maxPrice = cond.getMaxPrice();

        QItem item = QItem.item;
        BooleanBuilder builder = new BooleanBuilder();
        if (StringUtils.hasText(itemName)) {
            builder.and(item.itemName.like("%" + itemName + "%"));
        }
        if (maxPrice != null) {
            builder.and(item.price.loe(maxPrice));
        }

        List<Item> result = query
                .select(item)
                .from(item)
                .where(builder)
                .fetch();

        return result;
    }

    @Override
    public List<Item> findAll(ItemSearchCond cond) {

        String itemName = cond.getItemName();
        Integer maxPrice = cond.getMaxPrice();

        return query
                .select(item)
                .from(item)
                .where(likeItemName(itemName), maxPrice(maxPrice))
                .fetch();
    }

    private BooleanExpression likeItemName(String itemName) {
        if (StringUtils.hasText(itemName)) {
            return item.itemName.like("%" + itemName + "%");
        }
        return null;
    }

    private BooleanExpression maxPrice(Integer maxPrice) {
        if (maxPrice != null) {
            return item.price.loe(maxPrice);
        }
        return null;
    }
}
```
 JPAQueryFactory로 쿼리를 작성한다.
 QItem으로 객체를 쿼리에서 사용한다.
 builder로 하는 방법이 있지만 메서드를 나눠서 동적 쿼리를 작성하는 방법이 더 좋다.
 
 **공통**
Querydsl을 사용하려면 `JPAQueryFactory` 가 필요하다. `JPAQueryFactory` 는 JPA 쿼리인 JPQL을 만들 기 때문에 `EntityManager` 가 필요하다.
설정 방식은 `JdbcTemplate` 을 설정하는 것과 유사하다.
참고로 `JPAQueryFactory` 를 스프링 빈으로 등록해서 사용해도 된다.


**save(), update(), findById()**
기본 기능들은 JPA가 제공하는 기본 기능을 사용한다.
**findAllOld**
Querydsl을 사용해서 동적 쿼리 문제를 해결한다.
`BooleanBuilder` 를 사용해서 원하는 `where` 조건들을 넣어주면 된다.
이 모든 것을 자바 코드로 작성하기 때문에 동적 쿼리를 매우 편리하게 작성할 수 있다.

**findAll**
앞서 `findAllOld` 에서 작성한 코드를 깔끔하게 리팩토링 했다. 다음 코드는 누가 봐도 쉽게 이해할 수 있을 것이다.
```java
 List<Item> result = query
         .select(item)
              .from(item)
     .where(likeItemName(itemName), maxPrice(maxPrice))
     .fetch();
```
- Querydsl에서 `where(A,B)` 에 다양한 조건들을 직접 넣을 수 있는데, 이렇게 넣으면 AND 조건으로 처리된다. 참고로 `where()` 에 `null` 을 입력하면 해당 조건은 무시한다.
- 이 코드의 또 다른 장점은 `likeItemName()` , `maxPrice()` 를 다른 쿼리를 작성할 때 재사용 할 수 있다는 점 이다. 쉽게 이야기해서 쿼리 조건을 부분적으로 모듈화 할 수 있다. 자바 코드로 개발하기 때문에 얻을 수 있는 큰 장점이다.
이제 설정하고 실행해보자.


## 정리
**Querydsl 장점**
Querydsl 덕분에 동적 쿼리를 매우 깔끔하게 사용할 수 있다.
```java
 List<Item> result = query
         .select(item)
       ```
     .from(item)
     .where(likeItemName(itemName), maxPrice(maxPrice))
     .fetch();
```

- 쿼리 문장에 오타가 있어도 컴파일 시점에 오류를 막을 수 있다.
- 메서드 추출을 통해서 코드를 재사용할 수 있다. 예를 들어서 여기서 만든 `likeItemName(itemName)` ,`maxPrice(maxPrice)` 메서드를 다른 쿼리에서도 함께 사용할 수 있다.

Querydsl을 사용해서 자바 코드로 쿼리를 작성하는 장점을 느껴보았을 것이다. 그리고 동적 쿼리 문제도 깔끔하게 해
결해보았다. Querydsl은 이 외에도 수 많은 편리한 기능을 제공한다. 예를 들어서 최적의 쿼리 결과를 만들기 위해서 DTO로 편리하게 조회하는 기능은 실무에서 자주 사용하는 기능이다. JPA를 사용한다면 스프링 데이터 JPA와 Querydsl은 실무의 다양한 문제를 편리하게 해결하기 위해 선택하는 기본 기술이라 생각한다.
Querydsl에 대한 자세한 내용은 **실전! Querydsl** 강의를 참고하자.
