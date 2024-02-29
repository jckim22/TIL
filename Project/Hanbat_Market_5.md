이번 이슈는 Article 관련 CRUD를 하는 비즈니스 로직을 구현하고 테스트를 작성하는 것이었다.

사실 Item을 Article에 등록하고 수정하기 때문에 Item을 없애고 Article로 상품을 다 처리할 수도 있었다.

그러나 Item은 따로 관리하고 싶었다.
나중에 한 게시글에 여러 Item을 담을 수 있도록 업데이트가 될 수 있기 때문에 후에 확장성을 고려한 것이다.

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

    private String title;

    private String description;

    private String tradingPlace;

    private ArticleStatus articleStatus;

    @OneToOne(mappedBy = "article", cascade = CascadeType.ALL)
    private Item item;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private Member member;

    @OneToMany(mappedBy = "article", cascade = CascadeType.ALL)
    private List<ImageFile> imageFiles = new ArrayList<>();

    @CreatedDate
    @Convert(converter = Jsr310JpaConverters.LocalDateTimeConverter.class)
    private LocalDateTime createdAt;

    private Article(String title, String description, String tradingPlace) {
        this.title = title;
        this.description = description;
        this.tradingPlace = tradingPlace;
        this.createdAt = LocalDateTime.now();
        this.articleStatus = ArticleStatus.OPEN;
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
     * 생성 메서드
     */
    public static Article create(ArticleCreateDto articleCreateDto) {
        Article article = new Article(articleCreateDto.getTitle(),
                articleCreateDto.getDescription(), articleCreateDto.getTradingPlace());

        article.regisMember(articleCreateDto.getMember());
        article.regisItem(articleCreateDto.getItem());

        return article;
    }

    public void update(ArticleUpdateDto articleUpdateDto) {
        this.title = articleUpdateDto.getTitle();
        this.description = articleUpdateDto.getDescription();
        this.tradingPlace = articleUpdateDto.getDescription();
        this.item.updateItem(articleUpdateDto.getItemUpdateDto());
    }

    public void delete() {
        this.articleStatus = ArticleStatus.HIDE;
        this.item.deleteItem();
    }
}

```

Article domain에 CUD에 대한 비즈니스 로직을 작성해주었다.


#### HardDelete와 SoftDelete

Delete를 처리할 때 데이터베이스에 데이터를 직접 바로 삭제할 수도 있지만, 개발자의 실수를 방지하기 위해 삭제된 데이터를 일단 따로 모아놓고 나중에 일괄 삭제하는 것이 더 좋은 방법이라고 할 수 있다.

앞서 말한 직접 삭제하는 것을 **HardDelete** 방식이라고 하고 모아놓고 따로 처리하는 방식을 **SoftDelete** 방식이라고 한다.

#### SofteDelete

SoftDelte는 상태를 관리하며 삭제 처리를 하거나 아예 다른 삭제 테이블에 모아놓는 방법이 있다.
본 프로젝트에서는 편의를 위해 HIDE라는 게시글의 상태를 만들어 삭제를 처리하도록 했다.

deleteArticle의 로직을 보면 ArticleStatus를 OPEN에서 HIDE로 바꾸어 주는 것을 볼 수 있다.
```java
    public void deleteItem(){
        this.itemStatus = ItemStatus.HIDE;
    }
```
위처럼 Item도 마찬가지로 게시글이 삭제되었으니 HIDE로 상태를 변경해준다.

### 수정 시 Setter를 사용하지 않는 방법
수정 시에는 Setter를 사용하면 편하겠지만, Setter로 인해서 의도하지 않는 값 변경이 이루어지지 않도록 하고 싶었다.
그래서 DTO를 잘 사용하여 업데이트를 처리하도록 했다.
그에 관한 내용은 아래 링크에 정리해놓았다.
[Update시 Setter를 사용하지 않는 전략](https://velog.io/@jckim22/TroubleShooting-Update%EC%8B%9C-Setter%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EC%95%8A%EB%8A%94-%EC%A0%84%EB%9E%B5-HanbatMarket)

### ArticleService

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class ArticleService {

    private final ArticleRepository articleRepository;
    private final MemberRepository memberRepository;

    //영속성 전이로 ItemRepository에 따로 persist하지 않아도 됨
    @Transactional
    public Long regisArticle(Long memberId, ArticleCreateDto articleCreateDto, ItemCreateDto itemCreateDto) {
        Member newMember = memberRepository.findById(memberId).get();
        Item newItem = Item.createItem(itemCreateDto, newMember);
        articleCreateDto.setItem(newItem);
        articleCreateDto.setMember(newMember);
        return articleRepository.save(Article.create(articleCreateDto)).getId();
    }

    @Transactional
    public Long regisArticle(Long memberId, ArticleCreateDto articleCreateDto, ItemCreateDto itemCreateDto, List<ImageFileDto> imageFilesDto) {
        Member newMember = memberRepository.findById(memberId).get();
        Item newItem = Item.createItem(itemCreateDto, newMember);
        articleCreateDto.setItem(newItem);
        articleCreateDto.setMember(newMember);

        Article newArticle = Article.create(articleCreateDto);

        imageFilesDto.stream()
                .forEach(imageFileDto -> ImageFile
                        .createImageFile(newArticle, imageFileDto));

        return articleRepository.save(newArticle).getId();
    }

    //검색
    public List<Article> findArticles(ArticleSearchDto articleSearchDto) {
        return articleRepository.findAllBySearch(articleSearchDto);
    }

    public List<Article> findArticles() {
        return articleRepository.findAll();
    }

    public List<Article> findArticles(Member member) {
        return articleRepository.findAllByMember(member);
    }

    public Article findArticle(Long articleId) {
        return articleRepository.findById(articleId).get();
    }

    @Transactional
    public void deleteArticle(Long articleId) {
        Article article = articleRepository.findById(articleId).get();
        article.delete();
    }

    @Transactional
    public Long updateArticle(Long articleId, ArticleUpdateDto articleUpdateDto) {
        Article article = articleRepository.findById(articleId).get();
        article.update(articleUpdateDto);
        return articleId;
    }
}

```

이렇게해서 ArticleService가 완성되었다.


## 테스트

```java
    @Test
    public void 게시글_등록() throws Exception {
        //given
        Member member = Member.createMember("jckim229@gmail.com", "0303", "01028564221", "김주찬");
        memberService.join(member);
        ArticleCreateDto articleCreateDto = createArticleCreateDto("PS5 팝니다.", "싸게 팝니다.", "대전");
        ItemCreateDto itemCreateDto = createItemCreateDto("PS5", 170000L);
        List<ImageFileDto> imageFilesDto = createTestImageFilesDto();
        //when
        articleService.regisArticle(member.getId(), articleCreateDto, itemCreateDto, imageFilesDto);
        articleService.regisArticle(member.getId(), articleCreateDto, itemCreateDto);
        //then (등록된 게시글이 2개)
        assertEquals(2, articleService.findArticles(member).size());
        assertEquals(2, articleService.findArticles().size());

        //등록된 이미지 파일이 없음
        assertEquals(0, articleService.findArticles().get(1).getImageFiles().size());
    }

    @Test
    public void 게시글_등록_검색() throws Exception {
        //given
        Member member = Member.createMember("jckim229@gmail.com", "0303", "01028564221", "김주찬");
        memberService.join(member);

        ArticleCreateDto articleCreateDto = createArticleCreateDto("PS5 팝니다.", "싸게 팝니다.", "대전");
        ArticleCreateDto articleCreateDto2 = createArticleCreateDto("PS3 팝니다.", "싸게 팝니다.", "대전");
        ItemCreateDto itemCreateDto = createItemCreateDto("PS5", 170000L);
        ItemCreateDto itemCreateDto2 = createItemCreateDto("PS3", 130000L);
        List<ImageFileDto> imageFilesDto = createTestImageFilesDto();

        ArticleSearchDto articleSearchDto = new ArticleSearchDto();
        articleSearchDto.setTitle("PS3");
        //when
        articleService.regisArticle(member.getId(), articleCreateDto, itemCreateDto, imageFilesDto);
        articleService.regisArticle(member.getId(), articleCreateDto, itemCreateDto);
        articleService.regisArticle(member.getId(), articleCreateDto2, itemCreateDto2);
        //then (등록된 게시글이 3개)
        assertEquals(3, articleService.findArticles(member).size());
        assertEquals(3, articleService.findArticles().size());

        //then (PS3로 검색한 게시글이 1개)
        assertEquals(1, articleService.findArticles(articleSearchDto).size());
        articleSearchDto.setItemName("PS");
        articleSearchDto.setTitle("");

        //then (PS로 검색한 아이템 이름에 해당하는 게시글이 3개
        assertEquals(3, articleService.findArticles(articleSearchDto).size());

        //등록된 이미지 파일이 없음
        assertEquals(0, articleService.findArticles().get(1).getImageFiles().size());
    }

    @Test
    public void 게시글_수정() throws Exception {
        //given
        Member member = Member.createMember("jckim229@gmail.com", "0303", "01028564221", "김주찬");
        memberService.join(member);

        ArticleCreateDto articleCreateDto = createArticleCreateDto("PS5 팝니다.", "싸게 팝니다.", "대전");
        ItemCreateDto itemCreateDto = createItemCreateDto("PS5", 170000L);
        List<ImageFileDto> imageFilesDto = createTestImageFilesDto();

        Long articleId = articleService.regisArticle(member.getId(), articleCreateDto, itemCreateDto);
        Article article = articleService.findArticle(articleId);

        //when
        ArticleUpdateDto articleUpdateDto = new ArticleUpdateDto();
        articleUpdateDto.setTitle("수정한 게시글 제목");
        articleUpdateDto.setDescription("qwer");
        articleUpdateDto.setItemUpdateDto(createItemUpdateDto());
        articleUpdateDto.setTradingPlace("sdf");
        imageFilesDto.stream()
                .forEach(imageFileDto -> ImageFile.createImageFile(article, imageFileDto));

        articleService.updateArticle(articleId, articleUpdateDto);

        //then (수정된 게시글 제목으로 검색)
        ArticleSearchDto articleSearchDto = new ArticleSearchDto();
        articleSearchDto.setTitle("수정한");
        assertEquals(1, articleService.findArticles(articleSearchDto).size());

        //then (등록한 이미지 파일 2개)
        assertEquals(2, articleService.findArticles().get(0).getImageFiles().size());

    }

    @Test
    public void 게시글_삭제() throws Exception {
        //given
        Member member = Member.createMember("jckim229@gmail.com", "0303", "01028564221", "김주찬");
        memberService.join(member);

        ArticleCreateDto articleCreateDto = createArticleCreateDto("PS5 팝니다.", "싸게 팝니다.", "대전");
        ItemCreateDto itemCreateDto = createItemCreateDto("PS5", 170000L);

        Long articleId = articleService.regisArticle(member.getId(), articleCreateDto, itemCreateDto);

        //when (상태를 변경하는 것으로 softDelete 진행
        articleService.deleteArticle(articleId);

        //then
        assertEquals(ArticleStatus.HIDE, articleService.findArticles(member).get(0).getArticleStatus());
    }

}
```

이렇게 해서 CRUD에 대한 테스트까지 진행했다
READ는 검색을 통한 조회도 테스트하였다.

![](https://velog.velcdn.com/images/jckim22/post/16328541-caa8-4938-9dda-97170beb4bd1/image.png)
