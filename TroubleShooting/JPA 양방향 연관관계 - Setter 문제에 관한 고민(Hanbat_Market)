# 연관관계 매핑

[양방향 연관관계](https://velog.io/@jckim22/JPA-%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84-%EB%A7%A4%ED%95%91)
위 링크에 기록된 것처럼 이전에 한번 공부했던 내용이다.

막상 내 프로젝트에 적용하려고 하니 막막하더라

그래서 단방향만 구현해서 개발을 해볼까 생각해봤지만, 공부를 하는 입장이기에 절대 용납되지 않아서 양방향 연관관계를 도입했다.

## 연관관계의 주인

항상 연관관계의 주인이 누군지에 초점을 맞추니 매우 쉬운 일이 되었다.
- 외래키가 있는 쪽이 연관관계의 주인
- 연관관계의 주인이 있는 쪽에 연관관계 편의 메서드도 작성한다.


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

Member는 중심이 되는 객체이기에 외래키가 없어서 연관관계의 주인이되는 컬럼이 없다.
그래서 다 oneToMany로 양방향을 맞춰주었다.
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
Article은 Member와 Item과의 관계에서 주인이 된다.
따라서 그에 다른 편의 메서드도 작성해주었다.
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

### Setter에 관한 고민

개발을 진행하다가 문득 고민이 생겼다.
연관관계 편의 메서드를 작성하다보면 setter가 필요한 경우가 분명히 있을텐데

위 상황 같은 경우에는 Item과 Article이 1:1 관계였는데 양방향 세팅을 하려면 콜렉션에 add를 해주는 것도 아니라서 setter를 사용하는 방법밖에는 떠오르지 않았다.

또한 createMember를 하는 것 자체가 편의 메서드를 호출하는 것인데 이것 역시 세팅에 자유롭지 못한 것 아닌가? 라는 생각이 들었다.

고민이 되어서 현업에 계시는 분께 여쭤봤는데 아래와 같은 답변을 받을 수 있었다.

>양쪽 세터에서
```java
this.delivery = delivery;
if(!delivery.getOrder().equals(this)) {
 delivery.setOrder(this);
}
```
이런식의 패턴을 쓰면 그냥 setter 여는게아니라 괜찮습니다
일단 세팅 편의 메서드를 구상하시는데 결국 이게 세터나 다름 없어서 고민하시는 것 같아 무조건 쓰지마라가 경우에 따라 이유있는 세팅 메서드라면 사용하셔도 된다는 뜻으로 말씀드렸네요

위처럼 조건을 넣어서 진행하면 일반적인 세팅 메서드가 아니라 더 안전한 세팅 메서드가 되기 때문에 연관관계 편의 메서드를 작성할 때 참고하라는 답변을 받았다.

이처럼 세터를 작성하더라도 더 안전한 개발을 할 수 있는 방법을 따라간다면 문제들을 해결해나갈 것이라고 생각한다.

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

## 정리

공통적으로 연관관계 로딩은 무조건 지연로딩으로 설정했다.
지연로딩을 한다고해서 꼭 N+1 문제를 해결할 수 있는 것이 아니기에 후에 쿼리문으로 N+1 문제를 완벽하게 해결하려고 한다.

연관관계에서 주인이 아닌 쪽은 Cascade 영속성 전이 설정을 해두었다.
이것이 중복 persist를 일으켜서 문제가 될 때는 빼야겠지만, 현 단계에서는 문제가 발견되는 않았다.


