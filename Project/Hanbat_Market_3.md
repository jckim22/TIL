추가적으로 ImageFile과 PreemptionItem, Purchase에 대한 Repository를 만들고 테스트를 진행했다.

### 관계에 대한 고민

Purchase는 이전에 PurchaseHistory였는데 구매에 대한 비즈니스 로직과 구매 내역을 한 엔티티에서 처리하는게 좋을 것 같다는 생각이 들었다.

그리고 이전에는 Purchase와 Item이 일대다 관계였지만 고민을 해보니 인터넷 쇼핑몰처럼 상품 재고가 여러개 있는 것이 아닌 멤버의 게시글 하나당 하나의 상품으로 판단하기 때문에 일대일 관계가 맞았다.

그래서 Entity의 네이밍과 관계를 수정해주었다.

#### Item

```java
    @OneToOne(mappedBy = "item", cascade = CascadeType.ALL)
    private Purchase purchase;	        
```

#### Purchase

```java
    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "item_id")
    private Item item;
        
    private void regisItem(Item item) {
        this.item = item;
        item.setPurchase(this);
    }
```

둘 다 연관관계 메서드를 사용하면 복잡한 일(재귀)들이 벌어질 수 있으니 연관관계 메서드는 자식 메서드에서 진행했다.


### ImageFileRepository

```java
@Repository
@RequiredArgsConstructor
public class JpaImageFileRepository implements ImageFileRepository {

    private final EntityManager em;

    @Override
    public Long save(ImageFile imageFile) {
        em.persist(imageFile);
        return imageFile.getId();
    }

    @Override
    public Optional<ImageFile> findById(Long id) {
        return Optional.ofNullable(em.find(ImageFile.class, id));
    }

    @Override
    public List<ImageFile> findAll() {
        return em.createQuery("select i from ImageFile i")
                .getResultList();
    }
}

```

파일의 경로 등을 저장하는 ImageFile의 레포지토리를 만들었다.
특별한 것은 없다.


### PreemptionItemRepository

```java
@Repository
@RequiredArgsConstructor
public class JpaPreemptionItemRepository implements PreemptionItemRepository{

    private final EntityManager em;

    @Override
    public PreemptionItem save(PreemptionItem preemptionItem) {
        //        if (item.getId() == null){
        //            em.persist(item);
        //        }else {
        //            em.merge(item);
        //        }
        em.persist(preemptionItem);
        return preemptionItem;
    }

    @Override
    public Optional<PreemptionItem> findById(Long id) {
        PreemptionItem preemptionItem = em.find(PreemptionItem.class, id);
        return Optional.ofNullable(preemptionItem);
    }

    @Override
    public List<PreemptionItem> findAllByMember(Member member) {
        Long memberId = member.getId();
        return em.createQuery("select p from PreemptionItem p join p.member m where m.id = :memberId")
                .setParameter("memberId", member.getId())
                .getResultList();
    }

    @Override
    public List<PreemptionItem> findAllByItem(Item item) {
        Long itemId = item.getId();
        return em.createQuery("select p from PreemptionItem p join p.item i where i.id = :itemId")
                .setParameter("itemId", item.getId())
                .getResultList();
    }

    @Override
    public List<PreemptionItem> findAll() {
        return em.createQuery("select p from PreemptionItem p")
                .getResultList();
    }
}

```

찜 목록의 레포지토리를 만들어주었다.
찜은 사람에 따라 상품에 따라 검색할 것이라고 판단했기 때문에 Member와 Item으로 find할 수 있는 메서드를 만들었다.

### PurchaseRepository

```java
@Repository
@RequiredArgsConstructor
public class JpaPurchaserepository implements PurchaseRepository {

    private final EntityManager em;

    @Override
    public Long save(Purchase purchase) {
        em.persist(purchase);
        return purchase.getId();
    }

    @Override
    public Optional<Purchase> findById(Long id) {
        return Optional.ofNullable(em.find(Purchase.class, id));
    }

    //추후에 동적쿼리로 원하는 구매내역만을 볼 수 있는 기능 추가예정
    @Override
    public List<Purchase> findAll() {
        return em.createQuery("select p from Purchase p")
                .getResultList();
    }
}

```

구매에 관한 레포지토리를 만들어주었다.
구매 내역은 날짜나 어떤 종류에 따라서 검색을 할 수도 있기 때문에 추후에 동적 쿼리로 구현하려고 한다.

## 테스트

```java
    @Test
    public void 구매내역_등록() throws Exception {
        //given
        Member newMember = createTestMember1();
        memberService.join(newMember);

        Item testItem = createTestItem(newMember);
        Article article = creteTestArticle(newMember, testItem);

        jpaArticleRepository.save(article);
        //when
        Purchase purchase = Purchase.createPurchase(newMember, testItem);
        jpaPurchaseRepository.save(purchase);

        //then
        Item findItem = jpaItemRepository.findById(testItem.getId()).get();
        Member findMember = memberService.findOne(newMember.getId()).get();

        assertEquals(purchase, findMember.getPurchases().get(0));
        assertEquals(purchase, findItem.getPurchase());
        assertEquals(purchase, jpaPurchaseRepository.findById(purchase.getId()).get());
    }
```


```java
    @Test
    public void 찜_등록() throws Exception {
        //given
        Member newMember = createTestMember1();
        memberService.join(newMember);

        Item testItem = createTestItem(newMember);
        Article article = creteTestArticle(newMember, testItem);

        jpaArticleRepository.save(article);
        //when
        PreemptionItem preemptionItem = PreemptionItem.createPreemptionItem(newMember, testItem);
        jpaPreemptionItemRepository.save(preemptionItem);

        //then
        Item item = jpaItemRepository.findById(testItem.getId()).get();
        assertEquals(preemptionItem, item.getPreemptionItems().get(0));
        assertEquals(preemptionItem, jpaPreemptionItemRepository.
        findById(preemptionItem.getId()).get());
    }
```


```java
    @Test
    public void 아이템_등록() throws Exception {
        //given
        Member newMember = createTestMember1();
        memberService.join(newMember);

        Item testItem = createTestItem(newMember);
        Article article = creteTestArticle(newMember, testItem);
        ImageFile imageFile = ImageFile.createImageFile(article, "jpg", "/");

        //when
        jpaArticleRepository.save(article);

        //then
        List<Article> findArticles = jpaArticleRepository.findAllByMember(newMember);
        assertEquals(imageFile, findArticles.get(0).getImageFiles().get(0));
    }
```

만든 각각의 레포지토리에 대한 테스트를 만들어었다.

![](https://velog.velcdn.com/images/jckim22/post/f52df882-5ca0-462d-9a87-e8bb60059a22/image.png)

이제 지금까지 만들었던 엔티티와 Repository를 잘 사용해서 서비스와 도메인에 비즈니스 로직을 구현해야한다.
