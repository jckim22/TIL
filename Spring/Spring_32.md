배운 것들을 활용하여 간단한 웹 페이지를 만들어보자

# 요구사항 분석
상품을 관리할 수 있는 서비스를 만들어보자.
![](https://velog.velcdn.com/images/jckim22/post/fd7e92de-3a2f-40b8-ad54-cb5965033df5/image.png)


![](https://velog.velcdn.com/images/jckim22/post/d8422f3e-6e51-48b3-b1bf-c6d912999ddf/image.png)


**디자이너**: 요구사항에 맞도록 디자인하고, 디자인 결과물을 웹 퍼블리셔에게 넘겨준다.
**웹 퍼블리셔**: 다자이너에서 받은 디자인을 기반으로 HTML, CSS를 만들어 개발자에게 제공한다.
**백엔드 개발자**: 디자이너, 웹 퍼블리셔를 통해서 HTML 화면이 나오기 전까지 시스템을 설계하고, 핵심 비즈니스 모델을 개발한다. 이후 HTML이 나오면 이 HTML을 뷰 템플릿으로 변환해서 동적으로 화면을 그리고, 또 웹 화 면의 흐름을 제어한다.


>React, Vue.js 같은 웹 클라이언트 기술을 사용하고, 웹 프론트엔드 개발자가 별도로 있으면, 웹 프론트엔드 개발 자가 웹 퍼블리셔 역할까지 포함해서 하는 경우도 있다.
웹 클라이언트 기술을 사용하면, 웹 프론트엔드 개발자가 HTML을 동적으로 만드는 역할과 웹 화면의 흐름을 담 당한다. 이 경우 백엔드 개발자는 HTML 뷰 템플릿을 직접 만지는 대신에, HTTP API를 통해 웹 클라이언트가 필요로 하는 데이터와 기능을 제공하면 된다.



# 상품 도메인 개발

### Item
```java
@Getter @Setter
public class Item {

    private Long id;
    private String itemName;
    private Integer price;
    private Integer quantity;

    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}

```
### ItemRepository


```java
@Repository
public class ItemRepository {
    private static final Map<Long, Item> store = new HashMap<>(); //static
    private static long sequence = 0L; //static

    public Item save(Item item) {
        item.setId(++sequence);
        store.put(item.getId(), item);
        return item;
    }

    public Item findById(Long id) {
        return store.get(id);
    }

    public List<Item> findAll() {
        return new ArrayList<>(store.values());
    }


    //여기서 updateParam은 id를 사용하지도 않기도 해서 클래스를 하나 더 만드는게 좋다.
    //중복과 명확성 중에서는 명확성을 따르는게 맞다.
    public void update(Long itemId, Item updatePaarm) {
        Item findItem = findById(itemId);
        findItem.setItemName(updatePaarm.getItemName());
        findItem.setPrice(updatePaarm.getPrice());
        findItem.setQuantity(updatePaarm.getQuantity());
    }

    public void clearStroe(){
        store.clear();
    }
}

```

이렇게 Item class와 그 Repository를 만들었다.

```java
class ItemRepositoryTest {

    ItemRepository itemRepository = new ItemRepository();

    @AfterEach
    void afterEach(){
        itemRepository.clearStroe();
    }

    @Test
    void save() {
        //given
        Item item = new Item("itemA", 10000, 10);
        //when
        Item savedItem = itemRepository.save(item);
        //then
        Item findItem = itemRepository.findById(item.getId());
        assertThat(findItem).isEqualTo(savedItem);
    }

    @Test
    void findById() {
    }

    @Test
    void findAll() {
        //given
        Item item1 = new Item("item1", 10000, 10);
        Item item2 = new Item("item2", 20000, 20);

        itemRepository.save(item1);
        itemRepository.save(item2);
        //when
        List<Item> result = itemRepository.findAll();
        //then
        assertThat(result.size()).isEqualTo(2);
        assertThat(result).contains(item1,item2);
    }

    @Test
    void update() {
        //given
        Item item = new Item("item1", 10000, 10);

        Item savedItem = itemRepository.save(item);
        Long itemId = savedItem.getId();
        //when
        Item updateParam = new Item("item2", 20000, 20);
        itemRepository.update(itemId, updateParam);
        //then
        Item findItem = itemRepository.findById(itemId);

        assertThat(findItem.getItemName()).isEqualTo(updateParam.getItemName());
        assertThat(findItem.getPrice()).isEqualTo(updateParam.getPrice());
        assertThat(findItem.getQuantity()).isEqualTo(updateParam.getQuantity());
    }

    @Test
    void clearStroe() {
    }
}
```

테스트도 통과했으니 다음 단계로 가자


# 상품 서비스 HTML
핵심 비즈니스 로직을 개발하는 동안, 웹 퍼블리셔는 HTML 마크업을 완료했다.

https://getbootstrap.com/docs/5.0/getting-started/download/
위에서 부트스트랩을 다운 받자


![](https://velog.velcdn.com/images/jckim22/post/41102f6f-8723-4643-8f22-08f17983673c/image.png)
그리고 준비된 html과 css를 정적 디렉토리에 넣는다.

# 상품 목록 - 타임리프
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
            href="../css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
<div class="container" style="max-width: 600px">
    <div class="py-5 text-center">
        <h2>상품 목록</h2></div>
    <div class="row">
        <div class="col">
            <button class="btn btn-primary float-end"
                    onclick="location.href='addForm.html'"
                    th:onclick="|location.href='@{/basic/items/add}'|"
                   type="button">상품 등록
            </button>
        </div>
    </div>
    <hr class="my-4">
    <div>
        <table class="table">
            <thead>
            <tr>
                <th>ID</th>
                <th>상품명</th>
                <th>가격</th>
                <th>수량</th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="item : ${items}">
                <td><a href="item.html" th:href="@{/basic/items/{itemId}(itemId=${item.id})}" th:text="${item.id}">회원id</a></td>
                <td><a href="item.html" th:href="@{/basic/items/{itemId}(itemId=${item.id})}" th:text="${item.itemName}">상품명</a></td>
                <td th:text="${item.price}">10000</td>
                <td th:text="${item.quantity}">10</td>
            </tr>
            </tbody>
        </table>
    </div>
</div> <!-- /container -->
</body>
```
## 타임리프 알아보기

![](https://velog.velcdn.com/images/jckim22/post/f1ecec84-a4b2-4d13-83fb-df33b24a19da/image.png)
![](https://velog.velcdn.com/images/jckim22/post/9a8adae8-668b-426e-bfd5-920271f4d4a1/image.png)
![](https://velog.velcdn.com/images/jckim22/post/b46c07cf-92a3-4078-adaa-d60c056ed347/image.png)
![](https://velog.velcdn.com/images/jckim22/post/180f042f-bc83-4574-9c92-32d327b46853/image.png)
![](https://velog.velcdn.com/images/jckim22/post/870feaa0-7d15-44f6-a048-d3ae4e45e9ef/image.png)
![](https://velog.velcdn.com/images/jckim22/post/0ece4243-5f07-4f51-b583-3ecd0f9f6d5b/image.png)
![](https://velog.velcdn.com/images/jckim22/post/b803c29c-6b07-4958-ac03-accaac786ddf/image.png)


>타임리프는 순수 HTML 파일을 웹 브라우저에서 열어도 내용을 확인할 수 있고, 서버를 통해 뷰 템플릿을 거치면 동적으로 변경된 결과를 확인할 수 있다. JSP를 생각해보면, JSP 파일은 웹 브라우저에서 그냥 열면 JSP 소스코 드와 HTML이 뒤죽박죽 되어서 정상적인 확인이 불가능하다. 오직 서버를 통해서 JSP를 열어야 한다.
이렇게 **순수 HTML을 그대로 유지하면서 뷰 템플릿도 사용할 수 있는 타임리프의 특징을 네츄럴 템플릿**(natural templates)이라 한다.


# 상품 상세
```java
    @GetMapping("/{itemId}")
    public String item(@PathVariable("itemId") long itemId, Model model) {
        Item item = itemRepository.findById(itemId);
        model.addAttribute("item", item);
        return "basic/item";
    }
```

위처럼 상품 컨트롤러를 만든다.
>@PathVariable이 스프링 3.2 부터는 name을 생략할시 -parmeters 옵션이 필요하게 되었다.
그래서 웬만하면 name을 명시해주자

```
<!DOCTYPE HTML>
<html>
<head>
    <meta charset="utf-8">
    <link href="../css/bootstrap.min.css" rel="stylesheet">
    <style>
        .container {
            max-width: 560px;
        } </style>
</head>
<body>
<div class="container">
    <div class="py-5 text-center"><h2>상품 상세</h2>
    </div>
    <div>
        <label for="itemId">상품 ID</label>
        <input type="text" id="itemId" name="itemId" class="form-control"
               value="1" readonly>
    </div>
    <div>
        <label for="itemName">상품명</label>
        <input type="text" id="itemName" name="itemName" class="form-control"
               value="상품A" readonly></div>
    <div>
        <label for="price">가격</label>
        <input type="text" id="price" name="price" class="form-control"
               value="10000" readonly>
    </div>
    <div>
        <label for="quantity">수량</label>
        <input type="text" id="quantity" name="quantity" class="form-control"
               value="10" readonly>
    </div>
    <hr class="my-4">
    <div class="row">
        <div class="col">
            <button class="w-100 btn btn-primary btn-lg" onclick="location.href='editForm.html'" type="button">상품 수정
            </button>
        </div>
        <div class="col">
            <button class="w-100 btn btn-secondary btn-lg" onclick="location.href='items.html'" type="button">목록으로
            </button>
        </div>
    </div>
</div> <!-- /container -->
</body>
</html>
```
위처럼 타임리프로 상품 상세보기도 만들었다.

# 상품등록 폼

```java
    @GetMapping("/add")
    public String item(){
        return "basic/addForm";
    }

    @PostMapping("/add")
    public String addItemV2(@ModelAttribute("item") Item item, Model model){
        itemRepository.save(item);
//        model.addAttribute("item",item); //자동 추가 생략 가능
        return "basic/item";
    }
```
위처럼 메서드에 따라 다른 동작을 수행하게 한다.
get일 때는 그냥 폼을 던지고 post일 때는 @ModelAttritbute로 item을 바인딩하고 model에 담는다.

결국 @ModelAttritbute는 2가지 일을 한다.

**@ModelAttribute - 요청 파라미터 처리**
`@ModelAttribute` 는 `Item` 객체를 생성하고, 요청 파라미터의 값을 프로퍼티 접근법(setXxx)으로 입력해준다.
**@ModelAttribute - Model 추가**
`@ModelAttribute` 는 중요한 한가지 기능이 더 있는데, 바로 모델(Model)에 `@ModelAttribute` 로 지정한 객체
를 자동으로 넣어준다. 지금 코드를 보면 `model.addAttribute("item", item)` 가 주석처리 되어 있어도 잘 동작하는 것을 확인할 수 있다.



```java
<!DOCTYPE HTML>
<html>
<head>
    <meta charset="utf-8">
    <link href="../css/bootstrap.min.css" rel="stylesheet">
    <style>
        .container {
            max-width: 560px;
        } </style>
</head>
<body>
<div class="container">
    <div class="py-5 text-center"><h2>상품 등록 폼</h2>
    </div>
    <h4 class="mb-3">상품 입력</h4>
    <form action="item.html" method="post">
        <div>
            <label for="itemName">상품명</label>
            <input type="text" id="itemName" name="itemName" class="form- control" placeholder="이름을 입력하세요">
        </div>
        <div>
            <label for="price">가격</label>
            <input type="text" id="price" name="price" class="form-control" placeholder="가격을 입력하세요">
        </div>
        <div>
            <label for="quantity">수량</label>
            <input type="text" id="quantity" name="quantity" class="form- control" placeholder="수량을 입력하세요">
        </div>
        <hr class="my-4">
        <div class="row">
            <div class="col">
                <button class="w-100 btn btn-primary btn-lg" type="submit">상품 등
                    록
                </button>
            </div>
            <div class="col">
                <button class="w-100 btn btn-secondary btn-lg"
                        onclick="location.href='items.html'" type="button">취소
                </button>
            </div>
        </div>
    </form>
</div> <!-- /container -->
</body>
</html> 
```
타임이릎도 그에 맞게 수정한다.

# 상품 수정 개발

상품 수정 폼은 상품 등록과 유사하고, 특별한 내용이 없다.

```html
<!DOCTYPE HTML>
<html>
<head>
    <meta charset="utf-8">
    <link href="../css/bootstrap.min.css"
          rel="stylesheet">
    <style>
        .container {
            max-width: 560px;
        } </style>
</head>
<body>
<div class="container">
    <div class="py-5 text-center"><h2>상품 수정 폼</h2>
    </div>
    <form action="item.html" method="post">
        <div>
            <label for="id">상품 ID</label>
            <input type="text" id="id" name="id" class="form-control" value="1"
                   th:value="${item.id}"
                   readonly>
        </div>
        <div>
            <label for="itemName">상품명</label>
            <input type="text" id="itemName" name="itemName" class="form-
control" value="상품A"
                   th:value="${item.itemName}"
            ></div>
        <div>
            <label for="price">가격</label>
            <input type="text" id="price" name="price" class="form-control"
                   value="10000"
                   th:value="${item.itemPrice}"
            >
        </div>
        <div>
            <label for="quantity">수량</label>
            <input type="text" id="quantity" name="quantity" class="form-
control" value="10"
                   th:value="${item.quantity}"
            >
        </div>
        <hr class="my-4">
        <div class="row">
            <div class="col">
                <button class="w-100 btn btn-primary btn-lg" type="submit">저장</
                button>
            </div>
            <div class="col">
                <button class="w-100 btn btn-secondary btn-lg"
                        onclick="location.href='item.html'" type="button">취소
                </button>
            </div>
        </div>
    </form>
</div> <!-- /container -->
</body>
</html>
```

```java
    @GetMapping("/{itemId}/edit")
    public String editItem(Model model,
                           @PathVariable("itemId") long itemId) {
        model.addAttribute(itemRepository.findById(itemId));
        return "basic/editForm";
    }

    @PostMapping("/{itemId}/edit")
    public String editItem(@PathVariable("itemId") long itemId,
                           @ModelAttribute("item") Item item) {
        itemRepository.update(item.getId(), item);
        return "redirect:/basic/items/{itemId}";
    }
```

이번에도 메서드로 동작을 구분한다.
다만 Post일 때는 상세정보로 redirect를 한다.

>HTML Form 전송은 PUT, PATCH를 지원하지 않는다. GET, POST만 사용할 수 있다.
PUT, PATCH는 HTTP API 전송시에 사용
스프링에서 HTTP POST로 Form 요청할 때 히든 필드를 통해서 PUT, PATCH 매핑을 사용하는 방법이 있지 만, HTTP 요청상 POST 요청이다.

그런데 등록을 할 때는 왜 redirect를 하지 않고 수정할 때만 redirect를 했을까

그 비밀은 PRG Post/Redirect/Get 설명하기 위함에 있다.


# PRG Post/Redirect/Get


```java
    @GetMapping("/add")
    public String item() {
        return "basic/addForm";
    }

    @PostMapping("/add")
    public String addItemV2(@ModelAttribute("item") Item item, Model model) {
        itemRepository.save(item);
//        model.addAttribute("item",item); //자동 추가 생략 가능
        return "basic/item";
    }
```

![](https://velog.velcdn.com/images/jckim22/post/43654db4-bd53-4efe-9584-97cd33ea54f6/image.png)

지금 위에서는 리다이렉트로 GET으로 가지 않기 때문에 새로고침을 하게 되면 계속 POST 요청을 보내게 되어서 데이터가 계속 저장될 것이다.

![](https://velog.velcdn.com/images/jckim22/post/083c6e26-ffdb-4e16-ad94-1d2451988cc8/image.png)

위처럼 PRG로 해결할 수 있다.
웹 브라우저의 새로 고침은 마지막에 서버에 전송한 데이터를 다시 전송한다.
새로 고침 문제를 해결하려면 상품 저장 후에 뷰 템플릿으로 이동하는 것이 아니라, 상품 상세 화면으로 리다이렉트를 호출해주면 된다.
웹 브라우저는 리다이렉트의 영향으로 상품 저장 후에 실제 상품 상세 화면으로 다시 이동한다. 따라서 마지막에 호출한 내용이 상품 상세 화면인 `GET /items/{id}` 가 되는 것이다.
이후 새로고침을 해도 상품 상세 화면으로 이동하게 되므로 새로 고침 문제를 해결할 수 있다.

```java
    @PostMapping("/add")
    public String addItemV2(@ModelAttribute("item") Item item, Model model) {
        itemRepository.save(item);
//        model.addAttribute("item",item); //자동 추가 생략 가능
        return "redirect:/basic/items/"+item.getId();
    }
```

위처럼 수정한다.

근데 위에서는 +item.getId()에는 약간의 문제가 있다.
또 상품을 저장하고 상품 상세 화면으로 리다이렉트 한 것 까지는 좋았다. 그런데 고객 입장에서 저장이 잘 된 것인지 안 된 것인지 확신이 들지 않는다. 그래서 저장이 잘 되었으면 상품 상세 화면에 "저장되었습니다"라는 메시지를 보여달라는 요구사항이 왔다. 간단하게 해결해보자.

# RedirectAttributes

```java
    @PostMapping("/add")
    public String addItemV2(@ModelAttribute("item") Item item, RedirectAttributes redirectAttributes) {
        Item saveItem = itemRepository.save(item);
//        model.addAttribute("item",item); //자동 추가 생략 가능
        redirectAttributes.addAttribute("itemId", saveItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/basic/items/{itemId}";
    }
```
`RedirectAttributes` 를 사용하면 URL 인코딩도 해주고, `pathVariable` , 쿼리 파라미터까지 처리해준다.
-  `redirect:/basic/items/{itemId}`
-  pathVariable 바인딩: `{itemId}`
- 나머지는 쿼리 파라미터로 처리: `?status=true`

```
<h2 th:if="${param.status}" th:text="'저장 완료'"></h2>
```

그래서 위처럼 쿼리 파라미터의 status 값으로 조건을 걸어서 저장 완료라는 문구를 띄울 수 있게 되었다.
