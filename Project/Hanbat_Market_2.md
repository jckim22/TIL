이번에는 Repository를 개발해본다.

ImageFileRepository도 필요하지만, 일단은 Member, Item, Article에 대한 Repository만 만들어주었다.

#### 추상화에 의존한 이유

솔직한 간단할 수 있는 개인프로젝트인데 굳이 추상화에 의존하기 위해 인터페이스를 설계할 필요가 있나? 라는 생각을 했다.

하지만 이 프로젝트를 진행하는 것에 큰 이유중 하나인 '성장'을 이룰려면 최대한으로 학습할 수 있도록 프로젝트를 진행해야한다고 생각했다.

그런 의미로 나는 JPA를 먼저 사용한 뒤 DTO를 설계하고 REST API를 제대로 설계해서 API 버전도 진행하려고 했다.
그 뒤 기존에 사용했던 JPA는 그대로 둔 상태로 MyBatis로 프로젝트의 기술을 스위칭하려고 했다.

그걸 쉽게 이루기 위해서, 또 언제든 확장에 열려있기 위해서 SOLID 5원칙 . OCP, DIP를 지키기 위해 추상화에 의존하는 프로젝트를 제작하고자 했다.

그래서 일단 인터페이스를 설계했고 아래와 같은 구조로 만들었다.


![](https://velog.velcdn.com/images/jckim22/post/8d9ce3ca-110a-4148-91f3-20a302ff86a4/image.png)



# MembmerRepository
먼저 MemberRepository이다.
#### MemberRepository
```java
public interface MemberRepository {
    Member save(Member member);

    Optional<Member> findById(Long id);

    Optional<Member> findByNickName(String name);

    List<Member> findAll();
}
```
#### JpaMemberRepository
```java
@Repository
@RequiredArgsConstructor
public class JpaMemberRepository implements MemberRepository {

    private final EntityManager em;

    @Override
    public Member save(Member member) {
        em.persist(member);
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id);
        return Optional.ofNullable(member);
    }

    @Override
    public Optional<Member> findByNickName(String nickname) {
        List<Member> result = em.createQuery("select m from Member m where m.nickname = :nickname", Member.class)
                .setParameter("nickname", nickname)
                .getResultList();
        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }
}
```

Member는 Repository의 기본적인 동작만 있으면 된다.
굳이 동적쿼리가 필요하지 않을 것 같다고 생각한다. (클라이언트가 직접 꺼내는 일은 없을 것이라고 생각하여)

# ItemRepository
#### ItemRepository
```java
public interface ItemRepository {
    Item save(Item item);

    Optional<Item> findById(Long id);

    List<Item> findAllByMember(Member member);

    List<Item> findAll();
}

```
#### JpaItemRepository
```java
@Repository
@RequiredArgsConstructor
public class JpaItemRepository implements ItemRepository {

    private final EntityManager em;


    @Override
    public Item save(Item item) {
        //        if (item.getId() == null){
        //            em.persist(item);
        //        }else {
        //            em.merge(item);
        //        }
        em.persist(item);
        return item;
    }

    @Override
    public Optional<Item> findById(Long id) {
        Item item = em.find(Item.class, id);
        return Optional.ofNullable(item);
    }

    @Override
    public List<Item> findAllByMember(Member member) {
        Long memberId = member.getId();
        return em.createQuery("select i from Item i join i.member m where m.id = :memberId")
                .setParameter("memberId", member.getId())
                .getResultList();
    }

    @Override
    public List<Item> findAll() {
        return em.createQuery("select i from Item i")
                .getResultList();
    }
}

```

Item도 마찬가지로 클라이언트에 의해서 쿼리가 바뀔 필요가 없기 때문에 동적 쿼리는 필요하지 않다.

# ArticleRepository

동적 쿼리가 필요한 부분은 Article이다.
객체지향 관점으로 보면 사용자가 조건에 따라 검색하는데 그건 Item의 관한 것이고 Artice이 갖고 있는 Item을 꺼내어 조건에 맞는지 확인하고 맞다면 해당 Article 리턴한다.


#### ArticleRepository

```java
public interface ArticleRepository {

    Article save(Article article);

    Optional<Article> findById(Long id);

    List<Article> findAll();

    List<Article> findAllByMember(Member member);

    List<Article> findAllBySearch(ArticleSearchDto articleSearchDto);
}

```

검색에 관한 DTO를 만들어주었다.
검색은 상품 이름과, 상품 판매여부로 검색할 수 있도록 했다.

#### ArticleSearchDto
```java
@Getter
@Setter
public class ArticleSearchDto {
    ItemStatus itemStatus;

    String itemName;
}

```


#### JpaArticleRepository
```java
@Repository
@RequiredArgsConstructor
public class JpaArticleRepository implements ArticleRepository {

    private final EntityManager em;

    @Override
    public Article save(Article article) {
        em.persist(article);
        return article;
    }

    @Override
    public Optional<Article> findById(Long id) {
        Article article = em.find(Article.class, id);
        return Optional.ofNullable(article);
    }

    @Override
    public List<Article> findAll() {
        List<Article> articles = em.createQuery("select a from Article a", Article.class)
                .getResultList();
        return articles;
    }

    @Override
    public List<Article> findAllByMember(Member member) {
        Long memberId = member.getId();
        return em.createQuery("select a from Article a join a.member m where m.id = :memberId")
                .setParameter("memberId", member.getId())
                .getResultList();
    }

    @Override
    public List<Article> findAllBySearch(ArticleSearchDto articleSearchDto) {
        //language=JPAQL
        String jpql = "select a From Article a join a.item i";
        boolean isFirstCondition = true;
        //아이템 상태 검색
        if (articleSearchDto.getItemStatus() != null) {
            if (isFirstCondition) {
                jpql += " where";
                isFirstCondition = false;
            } else {
                jpql += " and";
            }
            jpql += " i.itemStatus = :itemStatus";
        }
        //아이템 이름 검색
        if (StringUtils.hasText(articleSearchDto.getItemName())) {
            if (isFirstCondition) {
                jpql += " where";
                isFirstCondition = false;
            } else {
                jpql += " and";
            }
            jpql += " i.itemName like :itemName";
        }
        TypedQuery<Article> query = em.createQuery(jpql, Article.class)
                .setMaxResults(1000); //최대 1000건
        if (articleSearchDto.getItemStatus() != null) {
            query = query
                    .setParameter("itemStatus", articleSearchDto.getItemStatus());
        }
        if (StringUtils.hasText(articleSearchDto.getItemName())) {
            query = query
                    .setParameter("itemName", articleSearchDto.getItemName());
        }
        return query.getResultList();
    }
}

```

동적쿼리를 QuertDsl를 사용해도 되었지만 개인적으로 일단 순수 String으로 JPQL을 작성하여 해결해보고 싶었다.

나중에 QueryDsl로 리팩토링할 것이다.

# 테스트

>테스트를 진행하다가 영속성 전이에 대한 고민이 생겨 트러블 슈팅을 정리해놓았다.
[영속성 전이가 꼭 필요한 상황인가](https://velog.io/@jckim22/TruobleShooting-%EC%98%81%EC%86%8D%EC%84%B1-%EC%A0%84%EC%9D%B4%EA%B0%80-%EA%BC%AD-%ED%95%84%EC%9A%94%ED%95%9C-%EC%83%81%ED%99%A9%EC%9D%B8%EA%B0%80)

테스트는 크게 등록, 연관관계 테스트, 전체 조회테스트 이런 구조로 진행하였다.
더미 데이터는 일단 직접 작성해주었다.

>추후에 profile을 사용하여 테스트에만 더미 데이터를 넣을 예정이다.

#### JpaMemberRespository

```java
@SpringBootTest
@Transactional
class JpaMemberRepositoryTest {
    @Autowired
    MemberRepository jpaMemberRepository;

    @Test
    public void 회원가입() throws Exception {
        //given
        Member member = createTestMember();
        //when
        Member saveMember = jpaMemberRepository.save(member);
        //then
        assertEquals(saveMember, jpaMemberRepository.findById(saveMember.getId()).get());
        assertEquals(saveMember, jpaMemberRepository.findByNickName(saveMember.getNickname()).get());
    }

    @Test
    public void 전체회원조회() throws Exception {
        //given
        Member member1 = createTestMember();
        Member member2 = Member.createMember("wncks0303@naver.com", "321", "010-321-321", "토마스");
        Member member3 = Member.createMember("wncks0303@sadf.com", "213123", "010-4244-321", "케빈");

        jpaMemberRepository.save(member1);
        jpaMemberRepository.save(member2);
        jpaMemberRepository.save(member3);
        //when
        List<Member> members = jpaMemberRepository.findAll();
        //then
        assertEquals(3, members.size());
    }

    public Member createTestMember() {
        String mail = "jckim229@gmail.com";
        String passwd = "1234";
        String phoneNumber = "010-1234-1234";
        String nickname = "김주찬";

        return Member.createMember(mail, passwd, phoneNumber, nickname);
    }
}
```


#### JpaItemRepositoryTest

```java
@SpringBootTest
@Transactional
class JpaItemRepositoryTest {

    @Autowired
    ItemRepository jpaItemRepository;
    @Autowired
    MemberRepository jpaMemberRepository;

    @Test
    public void 아이템_등록() throws Exception {
        //given
        Member member = createTestMember();

        Item item = createTestItem(member);
        //when
        jpaItemRepository.save(item);
        //then
        assertEquals(item, jpaItemRepository.findById(item.getId()).get());
    }

    @Test
    public void 아이템_멤버_연관성테스트() throws Exception {
        //given
        Member member = createTestMember();
        jpaMemberRepository.save(member);

        Item item = createTestItem(member);
        jpaItemRepository.save(item);

        //when
        Item findItem = jpaItemRepository.findById(item.getId()).get();
        Member findMember = jpaMemberRepository.findById(findItem.getMember().getId()).get();

        //then
        assertEquals(findItem, jpaItemRepository.findById(item.getId()).get());
        assertEquals(findMember, jpaMemberRepository.findById(member.getId()).get());

        assertEquals(item, findItem);
        assertEquals(member, findMember);
    }

    /**
     * 모든 아이템을 조회하는 것을 테스트 합니다.
     * 멤버와 이이템의 연관성을 테스트 합니다.
     */

    @Test
    public void 아이템_전체조회() throws Exception {
        //given
        Member member = createTestMember();
        jpaMemberRepository.save(member);

        Item item = createTestItem(member);
        Item item2 = createTestItem(member);
        Item item3 = createTestItem(member);
        jpaItemRepository.save(item);
        jpaItemRepository.save(item2);
        jpaItemRepository.save(item3);

        Member findMember = jpaMemberRepository.findById(member.getId()).get();

        //when
        List<Item> items = jpaItemRepository.findAll();
        List<Item> memberItems = jpaItemRepository.findAllByMember(findMember);

        //then
        assertEquals(3, items.size());
        assertEquals(findMember.getItems().size(), items.size());
        assertEquals(findMember.getItems().size(), memberItems.size());
    }

    public Member createTestMember() {
        String mail = "jckim229@gmail.com";
        String passwd = "1234";
        String phoneNumber = "010-1234-1234";
        String nickname = "김주찬";

        return Member.createMember(mail, passwd, phoneNumber, nickname);
    }

    public Item createTestItem(Member member) {
        String ItemName = "PS5";
        Long price = 10000L;

        return Item.createItem(ItemName, price, member);
    }
}
```

#### JpaArticleRepositoryTest

```java
@SpringBootTest
@Transactional
class JpaArticleRepositoryTest {
    @Autowired
    ItemRepository jpaItemRepository;
    @Autowired
    MemberRepository jpaMemberRepository;
    @Autowired
    ArticleRepository jpaArticleRepository;

    @Test
    public void 게시글_등록() throws Exception {
        //given
        Member member = createTestMember();
        Item item = createTestItem(member, "PS5");
        Article article = creteTestArticle(member, item);

        //when
        jpaArticleRepository.save(article);

        //then
        assertEquals(article, jpaArticleRepository.findById(article.getId()).get());
    }

    @Test
    public void 게시글_아이템_멤버_연관성테스트() throws Exception {
        //given
        Member member = createTestMember();
        jpaMemberRepository.save(member);

        Item item = createTestItem(member, "PS5");
        Article article = creteTestArticle(member, item);

        jpaArticleRepository.save(article);

        //when
        Article findArticle = jpaArticleRepository.findById(article.getId()).get();
        Item findItem = jpaItemRepository.findById(findArticle.getItem().getId()).get();
        Member findMember = jpaMemberRepository.findById(findArticle.getMember().getId()).get();

        //then
        assertEquals(article, jpaArticleRepository.findById(article.getId()).get());
        assertEquals(findItem, jpaItemRepository.findById(item.getId()).get());
        assertEquals(findMember, jpaMemberRepository.findById(member.getId()).get());

        assertEquals(article, findArticle);
        assertEquals(item, findItem);
        assertEquals(member, findMember);
    }

    /**
     * 모든 게시글을 조회하는 것을 테스트 합니다.
     * 멤버와 아이템과 게시글의 연관성을 테스트합니다.
     */

    @Test
    public void 게시글_전체조회() throws Exception {
        //given
        Member member = createTestMember();
        Member member2 = createTestMember2();

        jpaMemberRepository.save(member);
        jpaMemberRepository.save(member2);

        Item item = createTestItem(member, "PS3");
        Item item2 = createTestItem(member, "PS4");
        Item item3 = createTestItem(member2, "PS5");

        Article article = creteTestArticle(member, item);
        Article article2 = creteTestArticle(member, item2);
        Article article3 = creteTestArticle(member2, item3);

        jpaArticleRepository.save(article);
        jpaArticleRepository.save(article2);
        jpaArticleRepository.save(article3);

        Member findMember = jpaMemberRepository.findById(member.getId()).get();

        //when
        List<Article> articles = jpaArticleRepository.findAll();
        List<Article> memberArticles = jpaArticleRepository.findAllByMember(findMember);

        //then
        assertEquals(3, articles.size());
        assertEquals(2, findMember.getItems().size());
        assertEquals(2, memberArticles.size());
    }

    @Test
    public void 게시글_검색() throws Exception {
        //given
        Member member = createTestMember();
        Member member2 = createTestMember2();

        jpaMemberRepository.save(member);
        jpaMemberRepository.save(member2);

        Item item = createTestItem(member, "PS3");
        Item item2 = createTestItem(member, "PS4");
        Item item3 = createTestItem(member2, "PS5");

        Article article = creteTestArticle(member, item);
        Article article2 = creteTestArticle(member, item2);
        Article article3 = creteTestArticle(member2, item3);

        jpaArticleRepository.save(article);
        jpaArticleRepository.save(article2);
        jpaArticleRepository.save(article3);

        ArticleSearchDto articleSearchDto = new ArticleSearchDto();

        //when
        List<Article> searchAll = jpaArticleRepository.findAllBySearch(articleSearchDto);

        articleSearchDto.setItemName("PS5");
        List<Article> searchByItemName = jpaArticleRepository.findAllBySearch(articleSearchDto);

        articleSearchDto.setItemName(null);
        articleSearchDto.setItemStatus(ItemStatus.COMP);
        item.changeItemStatus();

        List<Article> searchByItemStatus = jpaArticleRepository.findAllBySearch(articleSearchDto);

        //then
        assertEquals(searchAll.size(), 3);
        assertEquals(searchByItemName.size(), 1);
        assertEquals(searchByItemStatus.size(), 1);
    }

    public Member createTestMember() {
        String mail = "jckim229@gmail.com";
        String passwd = "1234";
        String phoneNumber = "010-1234-1234";
        String nickname = "김주찬";

        return Member.createMember(mail, passwd, phoneNumber, nickname);
    }

    public Member createTestMember2() {
        String mail = "wncks0303@naver.com";
        String passwd = "1234";
        String phoneNumber = "010-4321-4321";
        String nickname = "토마스";

        return Member.createMember(mail, passwd, phoneNumber, nickname);
    }

    public Item createTestItem(Member member, String name) {
        String itemName = name;
        Long price = 10000L;
        String description = "hello";
        String tradingPlace = "성수역 7번 출구";

        return Item.createItem(itemName, price, member);
    }


    public Article creteTestArticle(Member member, Item item) {
        String title = "증고책 팝니다";
        String description = "hello";
        String tradingPlace = "성수역 7번 출구";

        return Article.createArticle(title, description, tradingPlace, member, item);
    }

}
```


# 정리

![](https://velog.velcdn.com/images/jckim22/post/96cacf91-90a5-4359-b07d-e326f8a2f614/image.png)

모든 테스트를 통과할 수 있었고 현재까지는 안정적이라는 느낌을 많이 받게 되었다.
