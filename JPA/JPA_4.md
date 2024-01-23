# 상속관계 매핑
- 관계형 데이터베이스는 상속 관계X
- 슈퍼타입 서브타입 관계라는 모델링 기법이 객체 상속과 유사
- 상속관계 매핑: 객체의 상속과 구조와 DB의 슈퍼타입 서브타입 관계를 매핑

![](https://velog.velcdn.com/images/jckim22/post/1ce41372-e4a2-49aa-93d5-3c1793401b92/image.png)



### 슈퍼타입 서브타입 논리 모델을 실제 물리 모델로 구현하는 방법 
-  각각테이블로변환 -> 조인전략
- 통합 테이블로 변환 -> 단일 테이블 전략
- 서브타입 테이블로 변환 -> 구현 클래스마다 테이블 전략

## 주요 어노테이션
- @Inheritance(strategy=InheritanceType.XXX) 
  - JOINED: 조인 전략
  -  SINGLE_TABLE: 단일 테이블 전략
  - TABLE_PER_CLASS: 구현 클래스마다 테이블 전략 
- @DiscriminatorColumn(name=“DTYPE”)
- @DiscriminatorValue(“XXX”)

## 조인 전략

![](https://velog.velcdn.com/images/jckim22/post/dbd9e103-6026-4862-bbb2-d122ccd77a50/image.png)

- 장점
테이블 정규화
외래 키 참조 무결성 제약조건 활용가능 
저장공간 효율화

- 단점
조회 시 조인을 많이 사용, 성능저하
조회 쿼리가 복잡함
데이터 저장시 INSERT SQL 2번 호출

## 단일 테이블 전략

![](https://velog.velcdn.com/images/jckim22/post/fd3b7edd-5d83-4b8b-a87f-346b74f64d54/image.png)
-  장점
  • 조인이 필요 없으므로 일반적으로 조회 성능이 빠름
  • 조회 쿼리가 단순함 
-  단점
• 자식 엔티티가 매핑한 컬럼은 모두 null 허용
• 단일테이블에모든것을저장하므로테이블이커질수있다.상 황에 따라서 조회 성능이 오히려 느려질 수 있다.

```java
@Ingeritance(strategy = InheritanceType.SINGLE_TABLE) //또는 JOINED
@DiscriminatorColum    // default: (name="DTYPE")
```


## 구현 클래스마다 테이블 전략
![](https://velog.velcdn.com/images/jckim22/post/f8d0a69f-107b-41da-bc76-9d3141aff986/image.png)

이 전략은 데이터베이스 설계자와 ORM 전문가 둘 다 추천X
-  여러 자식 테이블을 함께 조회할 때 성능이 느림(UNION SQL 필요)
- 자식 테이블을 통합해서 쿼리하기 어려움


# @MappedSuperclass
- 공통 매핑 정보가 필요할 때 사용(id, name)
![](https://velog.velcdn.com/images/jckim22/post/051628d3-0791-46c5-8f10-bf4e1e8c913a/image.png)

- 상속관계 매핑X
- 엔티티X, 테이블과 매핑X
- 부모 클래스를 상속 받는 자식 클래스에 매핑 정보만 제공
- 조회, 검색 불가(em.find(BaseEntity) 불가)
- 직접 생성해서 사용할 일이 없으므로 추상 클래스 권장


- 테이블과 관계 없고, 단순히 엔티티가 공통으로 사용하는 매핑 정보를 모으는 역할
- 주로 등록일, 수정일, 등록자, 수정자 같은 전체 엔티티에서 공통 으로 적용하는 정보를 모을 때 사용
> @Entity 클래스는 엔티티나 @MappedSuperclass로 지 정한 클래스만 상속 가능



# 적용

![](https://velog.velcdn.com/images/jckim22/post/e3bb1178-ea02-4fde-9c69-ce1f62012a4e/image.png)
![](https://velog.velcdn.com/images/jckim22/post/e72aec55-3c4f-437d-a376-10c645a7d91f/image.png)


BaseEntity를 @MappedSuperclass로 만들어서 일자를 담는다.
그리고 ITEM에 상속시키면 ITEM은 다 갖게된다.
ITEM은 상속관계에 있는 클래스이기 때문에 그 하위에 Movie, Book, Artist 클래스들은 싱글테이블 전략으로 ITEM 테이블에 생성된다.
