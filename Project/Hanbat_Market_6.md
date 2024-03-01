# Purchase -> Trade 네이밍 변경

막상 로직을 짜려고 하니 구매자와 소비자가 어떻게 구매를 결정할지에 대한 고민을 하게 되었다.
쇼핑몰처럼 재고를 클라이언트가 요청에서 주문하는 방식이 아닌 채팅을 통해 합의를 보고 구매자가 채택을 하는 방식으로 구매를 진행해야했다.

#### Status 추가
그래서 추가한 것이 Status를 추가한 것이다.
Status는 RESERVATION과 COMPLETE 그리고 RESERVATION 상태에서 해당 거래 예약을 취소한 상태인 CANCEL이 있다.

여기서 이런 상태들이 추가되니 점점 Purchase라는 단어를 사용하기에 애매하다는 생각이 들었다.
구매라는 동작을 하는데 예약, 취소 등이 어색했다.

그래서 '구매'보다는 '거래'가 적절할 것 같다고 생각하여 Purchase -> Trade로 명칭을 변경했다.
```java
public enum TradeStatus {
    RESERVATION, COMP, CANCEL
}
```

# 동적 쿼리

이전에 개발한 게시글 쿼리에 이슈가 있었다.
HIDE가 되어 삭제된 게시글까지 갖고 오는 쿼리가 문제였다.

그래서 아래처럼 수정해주었다.

#### findAllBySearch
```java
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

어떤 쿼리든 간에 마지막에 게시글과 Item의 Status가 HIDE가 아닌 row만을 가져오도록 했다.
>추후 QueryDsl로 리팩토링 예정




# 구매 내역 로직

#### TradeRepository

```java
@Repository
@RequiredArgsConstructor
public class JpaTradeRepository implements TradeRepository {

    private final EntityManager em;

    @Override
    public Long save(Trade trade) {
        em.persist(trade);
        return trade.getId();
    }

    @Override
    public Optional<Trade> findById(Long id) {
        return Optional.ofNullable(em.find(Trade.class, id));
    }

    //추후에 동적쿼리로 원하는 구매내역만을 볼 수 있는 기능 추가예정
    @Override
    public List<Trade> findAll() {
        return em.createQuery("select p from Trade p", Trade.class)
                .getResultList();
    }

    @Override
    public List<Trade> findCompleteByMember(Member member) {
        return em.createQuery("select p from Trade p join p.member m where " +
                        "p.tradeStatus = :tradeStatus", Trade.class)
                .setParameter("tradeStatus", TradeStatus.COMP)
                .getResultList();
    }

    @Override
    public List<Trade> findReservationByMember(Member member) {
        return em.createQuery("select p from Trade p join p.member m where " +
                        "p.tradeStatus = :tradeStatus", Trade.class)
                .setParameter("tradeStatus", TradeStatus.RESERVATION)
                .getResultList();
    }
}
```

예약중인 거래만을 검색하거나 거래완료된 거래를 검색하는 메서드를 따로 추가했다.

# 비즈니스 로직

#### Trade
```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@EntityListeners(AuditingEntityListener.class)
public class Trade {

    @Id
    @GeneratedValue
    @Column(name = "trade_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private Member member;

    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "item_id")
    private Item item;

    @CreatedDate
    @Convert(converter = Jsr310JpaConverters.LocalDateTimeConverter.class)
    private LocalDateTime tradeDate;

    private TradeStatus tradeStatus;


    /**
     * 연관관계 편의 메서드
     */
    private void regisMember(Member member) {
        this.member = member;
        member.getTrades().add(this);
    }

    private void regisItem(Item item) {
        this.item = item;
        item.setTrade(this);
    }

    /**
     * 비즈니스 로직
     */
    public static Trade reservation(Member member, Item item) {
        Trade trade = new Trade();
        trade.regisItem(item);
        trade.regisMember(member);
        trade.tradeStatus = TradeStatus.RESERVATION;
        return trade;
    }

    public Trade complete() {
        this.tradeStatus = TradeStatus.COMP;
        return this;
    }

    public void cancel(){
        this.tradeStatus = TradeStatus.CANCEL;
    }
}
```

객체를 객체답게 사용하기 위해 Trade 클래스 안에 비즈니스 로직들을 구현했다.
예약을 하면서 거래를 생성하도록 했고 예약을 취소하면 그 거래에 대한 SoftDelete를 진행한다.


#### TradeService
```java
public class TradeService {

    private final TradeRepository tradeRepository;
    private final MemberRepository memberRepository;
    private final ArticleRepository articleRepository;
    private final ItemRepository itemRepository;

    //판매자의 구매자가 약속을 체결하면 예약
    @Transactional
    public Long reservation(Long memberId, Long itemId) {
        Member member = memberRepository.findById(memberId).get();
        Item item = itemRepository.findById(itemId).get();
        Trade trade = Trade.reservation(member, item);

        tradeRepository.save(trade);

        return trade.getId();
    }

    //판매자가 판매를 완료하거나 구매자를 확실히 결정하면 구매완료
    @Transactional
    public Long tradeComplete(Long tradeId) {
        Trade trade = tradeRepository.findById(tradeId).get();
        trade.complete();
        return tradeId;
    }

    //구매내역 조회
    public List<Trade> findCompletedByMember(Member member) {
        return tradeRepository.findCompleteByMember(member);
    }

    //예약중인 거래 조회
    public List<Trade> findReservedByMember(Member member) {
        return tradeRepository.findReservationByMember(member);
    }

    //예약취소(판매자가 다룰 수 있음)
    @Transactional
    public void cancelTrade(Long tradeId) {
        Trade trade = tradeRepository.findById(tradeId).get();
        if (trade.getTradeStatus() == TradeStatus.COMP) {
            throw new IllegalArgumentException("이미 완료된 거래는 취소할 수 없습니다.");
        }
        trade.cancel();
    }
}
```
거래에 관한 서비스 로직을 작성했다.
대부분 위임에 가까운 로직이라 특별한 것은 없다.

# 테스트

#### TradeServiceTest

```java
    @Test
    public void 구매_예약() throws Exception {
        //given
        Member testMember = createTestMember1();
        memberService.join(testMember);

        Long articleId = articleService.regisArticle(testMember.getId(), createArticleCreateDto("test title", "test des", "대전"),
                createItemCreateDto("플스", 170000L));
        //when
        Article article = articleService.findArticle(articleId);
        Trade trade = Trade.reservation(testMember, article.getItem());
        //then
        assertEquals(tradeService.findReservedByMember(testMember).get(0).getTradeStatus(), TradeStatus.RESERVATION);
    }

    @Test
    public void 구매_완료() throws Exception {
        //given
        Member testMember = createTestMember1();
        memberService.join(testMember);

        Long articleId = articleService.regisArticle(testMember.getId(), createArticleCreateDto("test title", "test des", "대전"),
                createItemCreateDto("플스", 170000L));

        Article article = articleService.findArticle(articleId);
        Long tradeId = tradeService.reservation(testMember.getId(), article.getItem().getId());
        //when
        tradeService.tradeComplete(tradeId);
        //then
        assertEquals(tradeService.findCompletedByMember(testMember).get(0).getTradeStatus(), TradeStatus.COMP);
    }

    @Test
    public void 구매_취소() throws Exception {
        //given
        Member testMember = createTestMember1();
        memberService.join(testMember);

        Long articleId = articleService.regisArticle(testMember.getId(), createArticleCreateDto("test title", "test des", "대전"),
                createItemCreateDto("플스", 170000L));

        Article article = articleService.findArticle(articleId);
        Long tradeId = tradeService.reservation(testMember.getId(), article.getItem().getId());

        //when
        tradeService.cancelTrade(tradeId);

        //then
        assertEquals(tradeService.findReservedByMember(testMember).size(), 0);
    }
}
```

이에 대한 테스트를 진행했다.
거래에 특별한 것으로는 예약, 완료, 취소라고 생각해서 크게 3가지를 테스트했다.

![](https://velog.velcdn.com/images/jckim22/post/33753df1-c40c-46a0-8511-fb3f71013a45/image.png)
