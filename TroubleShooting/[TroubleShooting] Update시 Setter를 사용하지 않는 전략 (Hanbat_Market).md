# 문제 발생

Hanbat_Market에서 Article 관련 CRUD를 개발하던 중 Update 처리에 대한 고민이 생겼다.
Setter를 사용하면 편하겠지만, 의도치 않은 값 변경이 우려가 되었다.

무슨 방법이 있을까 구글링하다가 Dto를 잘 활용하면 된다는 쉽고 보다 안정성있는 방법을 찾을 수 있었다.

# DTO

![](https://velog.velcdn.com/images/jckim22/post/159e4aa4-9b03-4dcc-9f34-fc0e8c1117ae/image.png)
![](https://velog.velcdn.com/images/jckim22/post/8e281d69-06d6-42db-b43e-a16633a1042b/image.png)

위처럼 변경에 필요한 컬럼만을 모아놓은 DTO를 만들어 개발을 진행했다.

```java
    @Transactional
    public Long updateArticle(Long articleId, ArticleUpdateDto articleUpdateDto) {
        Article article = articleRepository.findById(articleId).get();
        article.update(articleUpdateDto);
        return articleId;
    }
```

Update시 위처럼 ArticleUpdateDto로 받게 했고 
```java
    public void update(ArticleUpdateDto articleUpdateDto) {
        this.title = articleUpdateDto.getTitle();
        this.description = articleUpdateDto.getDescription();
        this.tradingPlace = articleUpdateDto.getDescription();
        this.item.updateItem(articleUpdateDto.getItemUpdateDto());
    }
```

해당 객체의 update 로직으로 update를 진행했다.
Article을 업데이트 하면서 item 또한 같이 업데이트 해주었다.

```java
    public void update(ItemUpdateDto itemUpdateDto){
        this.price = itemUpdateDto.getPrice();
        this.itemName = itemUpdateDto.getItemName();
    }
```

이로써 보다 더 안정적인 개발을 할 수 있게 되었다.
