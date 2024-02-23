MVC 패턴에 의거한 템플릿으로 VIEW를 해결하여 어느정도 구실이 갖춰진 프로젝트를 REST API로 전환하는 과정에 맞닥뜨린 트러블을 이야기하고자 한다.

전환하는 과정에서 내가 우선적으로 하려고 했었던 것은 일단은 DTO를 다 만들어서 컨트롤러를 API로 전환하는 것이었고 REST API로 인증, 인가를 좀 더 안전하게 진행하기 위해 Spring Security JWT를 사용해서 인증, 인가를 해결하고자 했다.

또한 현재 @Valid로 검증이 되어있는 입력값들을 API 상에서 예외처리 하고자 했다. 

가장 우선인 것은 일단은 DTO를 다 개발하고 컨트롤러들을 리팩토링하는 과정이라고 생각이 들었다.
# 어떤 계층에서 DTO 변환을 진행해야할까?

그래서 나는 DTO를 만들고 Controller Layer에서 DTO -> Entity -> DTO 과정을 진행하려고 했다.
근데 엔티티가 복잡하면 복잡할 수록 컨트롤러가 Service나 Repository, Domain에 과도하게 의존하게 되고 뭔가 복잡한 로직이 노출되는 느낌을 받았다.

#### HomeControllerApi
```java
@GetMapping("/api")
public HomeResult<HomeResponseDto> home(@RequestBody ArticleSearchDto articleSearchDto) {
    // 트랜잭션 내에서 Member 엔티티를 로드
    Member loginMember = memberService.findOneWithArticles("jckim2").orElseThrow(() -> new RuntimeException("Member not found"));

    // 나머지 코드는 그대로 유지
    List<Article> articles = articleService.findArticles(articleSearchDto);
    List<PreemptionItem> preemptionItemByMember = preemptionItemService.findPreemptionItemByMember(loginMember);

    List<HomeArticlesDto> collect = articles.stream()
            .map(a -> new HomeArticlesDto(a.getId(), a.getTitle(), a.getDescription(), a.getTradingPlace(), a.getArticleStatus(),
                    a.getItem().getItemName(), a.getItem().getPrice(), a.getMember().getNickname(),
                    a.getImageFiles().get(0).getStoreFileName(), a.getCreatedAt()))
            .collect(Collectors.toList());

    HomeResponseDto homeResponseDto = new HomeResponseDto(collect, preemptionItemByMember.size());

    return new HomeResult<>(homeResponseDto);
```

또한 Transactional 안에서 일어나는 로직이 아니기도 하기 떄문에 LazyInitializationException이 계속 발생하게 되는 구조가 되었다.

# 해결

이렇게 되면 너무 코드가 복잡해지고 유지보수도 어려워지겠다 생각하여 [이 글](https://velog.io/@jsb100800/Spring-boot-DTO-Entity-%EA%B0%84-%EB%B3%80%ED%99%98-%EC%96%B4%EB%8A%90-Layer%EC%97%90%EC%84%9C-%ED%95%98%EB%8A%94%EA%B2%8C-%EC%A2%8B%EC%9D%84%EA%B9%8C)을 보고 이 API의 Entity <-> DTO 변환은 Service Layer에서 진행하는 것이 알맞겠다고 생각했다.

그렇게 하여 Service Layer로 코드를 옮겼고 아래처럼 컨트롤러는 매우 깔끔해졌다.


```java
    @GetMapping("/api")
    public HomeResponseDto home(@RequestBody ArticleSearchDto articleSearchDto) {
        /**
         * 토큰으로 멤버 인증하고 넣어주는 과정 필요함
         */
        Member loginMember = memberService.findOne("jckim2").get(); //시큐리티 구현전 임시 더미 데이터 멤버

        List<Article> articles = articleService.findArticles(articleSearchDto);
        List<HomeArticlesDto> homeArticlesDtos = articleService.findArticlesToDto(articles);
        List<PreemptionItem> preemptionItemByMember = preemptionItemService.findPreemptionItemByMember(loginMember);

        return new HomeResponseDto(preemptionItemByMember.size(), homeArticlesDtos.size(), homeArticlesDtos);
    }
```
아래처럼 Service에 보낼 때 DTO로 보냈고 또 서비스에서 DTO로 받을 수 있었다.
```java
        List<Article> articles = articleService.findArticles(articleSearchDto);
        List<HomeArticlesDto> homeArticlesDtos = articleService.findArticlesToDto(articles);
```

# LazyInitializationException과 성능 최적화

#### findArticlesToDto
```java
public List<HomeArticlesDto> findArticlesToDto(List<Article> articles) {
    List<HomeArticlesDto> collect = articles.stream()
            .map(a -> {
                List<ImageFile> imageFiles = a.getImageFiles();
                String storeFileName = (imageFiles != null && !imageFiles.isEmpty()) ? imageFiles.get(0).getStoreFileName() : null;

                return new HomeArticlesDto(
                        a.getId(),
                        a.getTitle(),
                        a.getDescription(),
                        a.getTradingPlace(),
                        a.getArticleStatus(),
                        a.getItem().getItemName(),
                        a.getItem().getPrice(),
                        a.getMember().getNickname(),
                        storeFileName,
                        a.getCreatedAt()
                );
            })
            .collect(Collectors.toList());
    return collect;
}

```
이제 해결이 되었겠다 싶었다.
Service는 Transaction이니 LazyInitializationException도 피해갈 수 있을 것이라 생각했다.
하지만 실패했다.

이유는 아무리 트랜잭션 안에서 진행되는 것이라고 할지라도 지연로딩이 진행되고 있기 때문에 영속성 컨텍스트에 Article만 담겨있고 연관된 다른 것들은 담겨있지 않은데 그것들을 건드리려고 하는 시점에서 이미 세션이 종료되었기 때문에 꺼내올 수가 없는 것이다.

그래서 한번에 가져와야겠다는 생각을 했다.
하지만 EAGER로 즉시 로딩으로 가져오게 되면 3개의 연관관계가 붙어있는 Article Entity는 N+N+N+1 문제를 일으켜버리고 말 것이다.

# 해결

그래서 쿼리를 한번만 날리기 위해 fetch Join을 사용하기로 했다.
3개중 컬렉션이 연관관계인 ImageFiles는 따로 Repository를 수정하여 트랜잭션 안에서 가져오도록 하겠다.

#### findAllBySearch

```java
    @Override
    public List<Article> findAllBySearch(ArticleSearchDto articleSearchDto) {
        //language=JPAQL
        String jpql = "select a From Article a join fetch a.item i join fetch a.member m";
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
        //게시글 제목 검색
        if (StringUtils.hasText(articleSearchDto.getTitle())) {
            if (isFirstCondition) {
                jpql += " where";
                isFirstCondition = false;
            } else {
                jpql += " and";
            }
            jpql += "  a.title like concat('%', :title, '%')";
        }
        //아이템 이름 검색
        if (StringUtils.hasText(articleSearchDto.getItemName())) {
            if (isFirstCondition) {
                jpql += " where";
                isFirstCondition = false;
            } else {
                jpql += " and";
            }
            jpql += " i.itemName like concat('%', :itemName, '%')";
        }
        //삭제된 게시글 제외
        if (isFirstCondition) {
            jpql += " where";
            isFirstCondition = false;
        } else {
            jpql += " and";
        }
        jpql += " a.articleStatus != :articleStatusHide and i.itemStatus != :itemStatusHide";

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
        if (StringUtils.hasText(articleSearchDto.getTitle())) {
            query = query
                    .setParameter("title", articleSearchDto.getTitle());
        }
        query = query
                .setParameter("articleStatusHide", ArticleStatus.HIDE)
                .setParameter("itemStatusHide", ItemStatus.HIDE);
        return query.getResultList();
    }
```
아직 QueryDsl을 적용하지 않았기 때문에 직접 String으로 Jpql을 수정했다.
위처럼 join fetch 문으로 아이템과 멤버를 한 쿼리에 가져왔다.
#### findByArticle
```Java
    @Override
    public List<ImageFile> findByArticle(Article article) {
        return em.createQuery("select i from ImageFile i where i.article.id = :articleId")
                .setParameter("articleId", article.getId())
                .getResultList();
    }
```

그리고 위처럼 article로 그에 맞는 ImageFile을 반환하는 트랜잭션을 새로 작성했다.
Transaction 안에서 진행되기 때문에 지연로딩으로 인한 에러는 발생하지 않을 것이다.

#### findArticlesToDto
```java
    public List<HomeArticlesDto> findArticlesToDto(List<Article> articles) {
        List<HomeArticlesDto> homeArticlesDtos = articles.stream()
                .map(a -> {
                    List<ImageFile> imageFiles = imageFileRepository.findByArticle(a);
                    String storeFileName = null;
                    if (imageFiles != null && !imageFiles.isEmpty()) {
                        storeFileName = imageFiles.get(0).getStoreFileName();
                    }

                    return new HomeArticlesDto(
                            a.getId(),
                            a.getTitle(),
                            a.getDescription(),
                            a.getTradingPlace(),
                            a.getArticleStatus(),
                            a.getItem().getItemName(),
                            a.getItem().getPrice(),
                            a.getMember().getNickname(),
                            storeFileName,
                            a.getCreatedAt()
                    );
                })
                .collect(Collectors.toList());
        return homeArticlesDtos;
    }
```
최종적으로 서비스 코드를 위처럼 수정했고 

#### HomeControllerApi
```java
    @GetMapping("/api")
    public HomeResponseDto home(@RequestBody ArticleSearchDto articleSearchDto) {
        /**
         * 토큰으로 멤버 인증하고 넣어주는 과정 필요함
         */
        Member loginMember = memberService.findOne("jckim2").get(); //시큐리티 구현전 임시 더미 데이터 멤버

        List<Article> articles = articleService.findArticles(articleSearchDto);
        List<HomeArticlesDto> homeArticlesDtos = articleService.findArticlesToDto(articles);
        List<PreemptionItem> preemptionItemByMember = preemptionItemService.findPreemptionItemByMember(loginMember);

        return new HomeResponseDto(preemptionItemByMember.size(), homeArticlesDtos.size(), homeArticlesDtos);
    }
```

 
#### RequsetBody
```json
{
    "itemStatus": null,
    "title": null,
    "itemName": null
}
```
api에 요청을 보냈더니
#### ResponseBody
```
{
    "memberPreemptionSize": 0,
    "articlesCount": 5,
    "articles": [
        {
            "id": 1,
            "title": "플스4 팝니다.",
            "description": "동해물과 백두산이 마르고 닳도록 하느님이 보우하사 우리나라 만세 무궁화 삼천리 화려강산 대한사람 대한으로 길이 보전하세",
            "tradingPlace": "대전 서구 월평동",
            "articleStatus": "OPEN",
            "itemName": "PlayStation4",
            "price": 170000,
            "memberNickname": "jckim2",
            "fileName": null,
            "createdAt": "2024-02-21T23:16:11.833"
        },
        {
            "id": 2,
            "title": "닌텐도 스위치 팝니다.",
            "description": "동해물과 백두산이 마르고 닳도록 하느님이 보우하사 우리나라 만세 무궁화 삼천리 화려강산 대한사람 대한으로 길이 보전하세",
            "tradingPlace": "대전 서구 월평동",
            "articleStatus": "OPEN",
            "itemName": "Nintendo Switch",
            "price": 100000,
            "memberNickname": "jckim2",
            "fileName": null,
            "createdAt": "2024-02-21T23:16:11.841"
        },
        {
            "id": 3,
            "title": "야구방망이 팝니다.",
            "description": "동해물과 백두산이 마르고 닳도록 하느님이 보우하사 우리나라 만세 무궁화 삼천리 화려강산 대한사람 대한으로 길이 보전하세",
            "tradingPlace": "대전 서구 월평동",
            "articleStatus": "OPEN",
            "itemName": "야구방망이",
            "price": 50000,
            "memberNickname": "jckim2",
            "fileName": null,
            "createdAt": "2024-02-21T23:16:11.842"
        },
        {
            "id": 4,
            "title": "행정학 전공서적 팝니다.",
            "description": "동해물과 백두산이 마르고 닳도록 하느님이 보우하사 우리나라 만세 무궁화 삼천리 화려강산 대한사람 대한으로 길이 보전하세",
            "tradingPlace": "대전 서구 월평동",
            "articleStatus": "OPEN",
            "itemName": "행정학입문",
            "price": 34000,
            "memberNickname": "wncks0303",
            "fileName": null,
            "createdAt": "2024-02-21T23:16:11.843"
        },
        {
            "id": 5,
            "title": "컴퓨터 개론 팝니다.",
            "description": "동해물과 백두산이 마르고 닳도록 하느님이 보우하사 우리나라 만세 무궁화 삼천리 화려강산 대한사람 대한으로 길이 보전하세",
            "tradingPlace": "대전 서구 월평동",
            "articleStatus": "OPEN",
            "itemName": "컴퓨터 개론",
            "price": 25000,
            "memberNickname": "wncks0303",
            "fileName": null,
            "createdAt": "2024-02-21T23:16:11.843"
        }
    ]
}
```
위처럼 데이터를 잘 받을 수 있었다.

Fetch Join으로 성능 최적화는 물론 LazyInitializationException도 해결할 수 있었다.
