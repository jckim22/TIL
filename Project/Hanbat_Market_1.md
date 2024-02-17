추후에 MyBatis로도 개발할 것이고 REST API로 개발하여 React와 협업도 할 것이지만, 일단은 JPA와 Thmyeleaf를 사용해서 개발해보기로 했다.

공부를 위해 일단은 Spring Data JPA와 QueryDsl은 사용하지 않기로 했다.

이번 프로젝트에서 가장 먼전 진행한 것은 domain에 Entity들을 구현하는 것이다.
일전에 김영한님의 강의에서 들었던 것처럼 RDBMS를 객체지향에 입혀 그 패러다임을 맞추는 것에 집중했다.

원리들을 하나씩 다시 복습해가며 개발을 진행했다.

# 문서화

먼저 아래처럼 컨벤션과 같은 규칙들을 문서화했다.
나와 같은 서버 개발 지망생인 친구와 같이 진행하는 프로젝트이지만, 거의 초중반에는 개인 프로젝트르 진행되기 때문에 이런 컨벤션같은 것들을 꼭 정할 필요가 없을지도 모르지만, 항상 10명 이상이 같이 보는 코드라고 생각하면서 개발하기를 원했다.

![](https://velog.velcdn.com/images/jckim22/post/aaa56335-c8f8-4208-a52d-3e27a0ec711f/image.png)
![](https://velog.velcdn.com/images/jckim22/post/6c5f4c9f-eccd-417d-a62f-3ac44dad4a02/image.png)



### 요구사항 정리
문서에 요구사항도 정리했다.

> 한밭 마켓은 거래 사이트이다.

- 한밭 마켓에는 회원이 존재한다.
    - 회원은 회원 정보를 가진다.
        - 회원 정보 아래와 같이 구성된다.
            - 이메일
            - 비밀번호
            - 전화번호
            
    - 회원은 회원 가입을 할 수 있다.
        - 회원은 회원정보를 전부 입력해야 회원 가입할 수 있다.
        
    - 회원은 로그인할 수 있다.
        - 회원은 아이디와 비밀번호를 입력해 로그인할 수 있다.
        
    - 회원은 상품을 구매할 수 있다.
    - 회원은 상품을 판매할 수 있다.
    - 회원은 거래 내역을 볼 수 있다.
        
        
    - 회원은 게시글을 작성할 수 있다.
    - 회원은 게시글을 검색할 수 있다.

- 한밭 마켓에는 게시판이 존재한다.
    - 게시판에는 게시글을 작성할 수 있다.
    - 게시판의 게시글을 조회할 수 있다.
        - 게시판의 게시글을 검색해서 조회할 수 있다.
            - 검색 기준은 판매중, 판매완료가 있다.
            - 검색 기준은 상품 이름이 있다.
        - 게시판의 게시글을 특정 기준에 의해 정렬될 수 있다.
            - 가격 높은순
            - 가격 낮은순
            - 최근 등록일
    - 게시판의 게시글을 수정할 수 있다.
    - 게시판의 게시글을 삭제할 수 있다.

- 한밭 마켓에는 게시글이 존재한다.
    - 게시글에는 상품을 등록할 수 있다.
    - 게시글을 아래와 같이 구성된다.
        - 등록 일자
        - 수정 일자

- 한밭 마켓에는 상품이 존재한다.
    - 상품의 상품 정보은 아래와 같이 구성된다.
        - 사진
            - 사진 파일은 다음과 같이 구성된다
                - id
                - 파일 타입
                - 파일 이름
                - 생성 일자
                - 수정 일자
        - 가격
        - 상품 설명
        - 거래 장소
        - 찜한 회원의 수
        - 거래 상태
    - 회원은 상품을 찜할 수 있다.
    
### ERD

요구사항을 기반으로 ERD도 작성했다.

![](https://velog.velcdn.com/images/jckim22/post/1eb0063b-a7ae-4e6e-ba88-bc6d814248c3/image.png)
>관계선이 버그로 안보이는 것 같다..

# 고민

첫 스프링 프로젝트여서 구조는 최대한 간단하게 가져갔다.
또한 여기서 고민했던 것이 꼭 양방향 연관관계로 엔티티를 구현해 할지에 대한 고민이었다.

그렇게 구현하게 되면 처음인 나에게 양방향 연관관계에 대한 고민, 편의 메서드 등 좀 난이도가 올라갈 것이라는 우려 때문이었다.

하지만 공부하고 성장하기를 원했기에 바로 적용해보았다.
이에 대한 고민은 아래 링크에 적어 놓았다.
[JPA 양방향 연관관계](https://velog.io/@jckim22/TroubleShooting-JPA-%EC%96%91%EB%B0%A9%ED%96%A5-%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84-HanbatMarket)

# 개발

### Member
```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED) // 생성자를 통해서 값 변경 목적으로 접근하는 메시지들 차단
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "user_id")
    private Long id; //동시성 문제해결을 위해 추후에 AtomicLong 사용

    @Column(nullable = false, unique = true)
    private String mail;

    @Column(nullable = false)
    private String passwd;

    @Column(nullable = false, unique = true)
    private String phoneNumber;

    @Column(nullable = false, unique = true)
    private String nickname;

    private Member(String mail, String passwd, String phoneNumber, String nickname) {
        this.mail = mail;
        this.passwd = passwd;
        this.phoneNumber = phoneNumber;
        this.nickname = nickname;
    }

    @OneToMany(mappedBy = "member", cascade = CascadeType.ALL)
    private List<Article> articles = new ArrayList<>();

    @OneToMany(mappedBy = "member", cascade = CascadeType.ALL)
    private List<Item> items = new ArrayList<>();

    @OneToMany(mappedBy = "member", cascade = CascadeType.ALL)
    private List<PreemptionItem> preemptionItems = new ArrayList<>();

    @OneToMany(mappedBy = "member", cascade = CascadeType.ALL)
    private List<PurchaseHistory> purchaseHistories = new ArrayList<>();

    public static Member createMember(String mail, String passwd, String phoneNumber, String nickname){
        return new Member(mail, passwd, phoneNumber, nickname);
    }

}
```
Member Entity이다.
>id는 동시성 문제를 해결할 수 있는 AtomicLong을 적용해보고자 했지만, 아직 더 집중해야할 것들에 집중하자고 생각해서 Long을 사용했다.

생성 메서드를 만들고 생성자를 실행하게 했는데 이 부분에 대해서 조금 더 고민이 필요해보인다.
Builder를 사용하는 방법에 대해서도 공부해보고 리팩토링을 진행보려고 한다.

### Article
```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED) // 생성자를 통해서 값 변경 목적으로 접근하는 메시지들 차단
@EntityListeners(AuditingEntityListener.class)
public class Article {

    @Id
    @GeneratedValue
    @Column(name = "article_id")
    private Long id;

    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "item_id")
    private Item item;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private Member member;

    @OneToMany(mappedBy = "article", cascade = CascadeType.ALL)
    private List<ImageFile> imageFiles = new ArrayList<>();

    @CreatedDate
    @Convert(converter = Jsr310JpaConverters.LocalDateTimeConverter.class)
    private LocalDateTime createdAt;

    private Article(Member member, Item item) {
        this.createdAt = LocalDateTime.now();
    }

    /**
     * 연관관계 편의 메서드
     */
    private void regisMember(Member member) {
        this.member = member;
        member.getArticles().add(this);
    }

    private void regisItem(Item item) {
        this.item = item;
        item.setArticle(this);
    }

    /**
     * @param member
     * @param item   생성 메서드
     */
    public static Article createArticle(Member member, Item item) {
        Article article = new Article(member, item);
        article.regisMember(member);
        article.regisItem(item);
        return article;
    }

}

```

게시글 엔티티이다.
처음에 생성시간과 수정시간을 컬럼으로 넣었는데 수정시간은 일단 빼놓았다.

#### LocalDateTime은 Byte로 저장되는 것으로 알고있는데?

LocalDateTime은 바이트로 값을 받아오기 때문에 TimeStamp로 저장되지 않는 문제를 해결하기 위해 구글링했더니 아래와 같은 컨버터를 추가하면 TimeStamp로 변환할 수 있었다.
```java
  @Convert(converter = Jsr310JpaConverters.LocalDateTimeConverter.class)
```


#### NoArgsConstructor의 접근 제어자가 Protected인 이유

구글링하며 공부하다가 Entity마다 @NoArgsConstructor가 붙어있는 것을 많이 볼 수 있었다.
이건 기본 생성자에 관한 어노테이션인데 의문이 들었다.

>왜 AccesLevel을 Protected로 한 것이지?

바로 기본 생성자로 인한 무분별한 객체 생성을 막기 위함이라고 한다.
예를 들어 아래 같은 상황을 보자

```java
Member member = new Member()
meber.set(passwd)
```
>id는 설정이 되지 않았다.

하지만 기본 생성자를 막아주게 되면
이런 잘못된 무분별한 생성을 막아준다는 것에 큰 의미가 있다.

>근데 왜 private가 아니라 protected일까?

그 비밀은 JPA의 동작 방식에 있다.
JPA에서 로딩을 할 때 직접 객체를 건드는 것이 아닌 진짜 객체를 상속받은 프록시 객체를 생성하여 간접적으로 지연로딩을 진행한다. 
프록시 객체는 super를 사용해서 간접적으로 원래 객체를 호출한다.
근데 만약 여기서 private으로 설정해놓으면 호출할 수가 없다.

이런 이유들로 아래와 같은 어노테이션을 붙여주게 된다.

```java
@NoArgsConstructor(access = AccessLevel.PROTECTED)
```



### Item

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED) // 생성자를 통해서 값 변경 목적으로 접근하는 메시지들 차단
public class Item {

    @Id
    @GeneratedValue
    @Column(name = "item_id")
    private Long id;

    @Column(nullable = false)
    private Long price;

    private String description;

    private String tradingPlace;

    private Long preemptionCount;

    @Enumerated(EnumType.STRING)
    private ItemStatus itemStatus;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private Member member;

    @OneToOne(mappedBy = "item", cascade = CascadeType.ALL)
    private Article article;

    @OneToMany(mappedBy = "item", cascade = CascadeType.ALL)
    private List<PreemptionItem> PreemptionItems = new ArrayList<>();

    @OneToMany(mappedBy = "item", cascade = CascadeType.ALL)
    private List<PurchaseHistory> purchaseHistories = new ArrayList<>();

    private Item(Long price, String description, String tradingPlace, Member member) {
        this.price = price;
        this.description = description;
        this.tradingPlace = tradingPlace;
        this.preemptionCount = 0L;
        this.itemStatus = ItemStatus.SALE;
    }

    //일대일 연관관계 메서드를 위해 세터 생성
    protected void setArticle(Article article) {
        this.article = article;
    }

    /**
     * 연관관계 편의 메서드
     */
    private void regisMember(Member member) {
        this.member = member;
        member.getItems().add(this);
    }

    /**
     * 생성 메서드
     */
    public static Item createItem(Long price, String description, String tradingPlace, Member member) {
        Item item = new Item(price, description, tradingPlace, member);
        item.regisMember(member);
        return item;
    }
}
```
Item이다.
Setter를 최대한 지양하고자 했지만 Setter가 있다.
이러한 고민은 위에 달아놨었던 [JPA 양방향 연관관계](https://velog.io/@jckim22/TroubleShooting-JPA-%EC%96%91%EB%B0%A9%ED%96%A5-%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84-HanbatMarket) 이 링크에서 확인할 수 있다.

### ImageFile

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class ImageFile {

    @Id
    @GeneratedValue
    @Column(name = "file_id")
    private Long id;

    private String type;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "article_id")
    private Article article;

    private ImageFile(String type, String name) {
        this.type = type;
        this.name = name;
    }

    /**
     * 연관관계 메서드
     */
    private void regisArticle(Article article) {
        this.article = article;
        article.getImageFiles().add(this);
    }

    /**
     * 생성 메서드
     */
    public static ImageFile createImageFile(Article article, String type, String name) {
        ImageFile imageFile = new ImageFile(type, name);
        imageFile.regisArticle(article);
        return imageFile;
    }
}

```

이미지 파일 클래스이다.
실제 파일이름이랑 저장될 때 파일이름 2가지가 있어야하는데 일단은 진행하고 나중에 리팩토링 하려고 한다.
### PreemptionItem

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class PreemptionItem {

    @Id
    @GeneratedValue
    @Column(name = "preemption_item_id")
    private Long id;
    ;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private Member member;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "item_id")
    private Item item;

    /**
     * 연관관계 편의 메서드
     */
    private void regisMember(Member member) {
        this.member = member;
        member.getPreemptionItems().add(this);
    }

    void regisItem(Item item) {
        this.item = item;
        item.getPreemptionItems().add(this);
    }

    public static PreemptionItem createPreemptionItem(Member member, Item item) {
        PreemptionItem preemptionItem = new PreemptionItem();
        preemptionItem.regisItem(item);
        preemptionItem.regisMember(member);
        return preemptionItem;
    }
}

```

찜 목록이다 연관관계는 [JPA 양방향 연관관계](https://velog.io/@jckim22/TroubleShooting-JPA-%EC%96%91%EB%B0%A9%ED%96%A5-%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84-HanbatMarket)여기서 설명할 것이기 때문에 특별한 것은 없다.

### PurchaseHistory

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class PurchaseHistory {

    @Id
    @GeneratedValue
    @Column(name = "purchase_history_id")
    private Long id;
    ;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private Member member;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "item_id")
    private Item item;

    /**
     * 연관관계 편의 메서드
     */
    private void regisMember(Member member) {
        this.member = member;
        member.getPurchaseHistories().add(this);
    }

    private void regisItem(Item item) {
        this.item = item;
        item.getPurchaseHistories().add(this);
    }

    public static PurchaseHistory createPurchaseHistory(Member member, Item item) {
        PurchaseHistory purchaseHistory = new PurchaseHistory();
        purchaseHistory.regisItem(item);
        purchaseHistory.regisMember(member);
        return purchaseHistory;
    }
}

```

구매내역 또한 특별한 것이다 없다.

## 정리

이렇게 개발을 완료하고 ddl-auto 기능으로 애플리케이션을 시작하고 받은 ddl 쿼리들을 보며 수정할 부분을 또 찾고 있다.

간단한 Entity들을 개발하더라도 원리들을 하나 하나 다 이해하고 싶은 마음에 시간이 조금 더 걸리는 것 같다.

추후에 여기에 비즈니스 로직과 수정시간, AtomicLong, 파일 이름 등 많은 리팩토링이 진행될 것이다.
